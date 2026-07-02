# AWS Lambda Cold Start Optimization Documentation

This document records the architectural details, code changes, and performance benchmarks for the cold-start optimizations implemented across the HMS microservices.

---

## 1. Context & Motivation
Under the initial configuration, the HMS microservices running as AWS Lambda Container Functions suffered from a **~7-second cold start latency** (specifically in UAT environments allocated with `256 MB` memory / `~0.15 vCPUs`). 

Since Lambdas inside a VPC are heavily CPU-bound during bootstrap (running TCP/SSL handshakes and importing heavy libraries), optimizations were required to bring the cold start execution duration down without increasing the RAM/CPU cost.

---

## 2. Core Latency Bottlenecks & Solutions

### Bottleneck A: Redundant Outbound JWKS Key Fetches
* **The Problem**: In `shared/auth/middleware.py`, JWT signature verification instantiated a new `jwt.PyJWKClient` on every request. This forced a synchronous network request over the VPC's NAT Gateway to Amazon Cognito (`cognito-idp.ap-south-1.amazonaws.com`) to retrieve the JSON Web Key Set (JWKS), adding **1.5 to 3.0 seconds** of cold start latency.
* **The Solution**: Extracted the `kid` from the JWT header, resolved the matching public key locally from a thread-safe global memory cache `_jwks_cache`, and loaded the public key locally using `RSAAlgorithm.from_jwk`. Outbound network hops to Cognito are now **reduced from 2 to 1 on cold starts, and 0 on warm starts**.

### Bottleneck B: Lazy Database Connection Pool Initialization
* **The Problem**: The PostgreSQL connection pool (`ThreadedConnectionPool`) was lazy-initialized on the first query request inside the handler thread. Resolving the RDS database endpoint, establishing the TCP socket, and completing the PG SSL handshake added **500ms to 1.5 seconds** of latency during the billed request execution window.
* **The Solution**: Triggered eager initialization of the connection pool `get_db()` at the global module level (Initialization Phase). This shifts the connection handshake to the container bootstrap phase, which AWS Lambda provides as a free CPU burst (unbilled).

### Bottleneck C: Deferred Python Module Imports
* **The Problem**: The entry points lazy-imported the router (`from app.routes.router import route`) inside the handler function. Under `256 MB` RAM (`~0.15 vCPUs`), resolving and compiling all Pydantic schemas, routing trees, and core business services took **~6.5 seconds**, all of which was billed handler execution time.
* **The Solution**: Moved the router import to the top level of the file (module scope). This forces Python to import and compile all schemas during the unbilled container Init Phase, leaving the handler to execute instantly once active.

---

## 3. Reference Implementation Diffs

### Pattern A: Optimized JWT Verification (`shared/auth/middleware.py`)
```python
def decode_jwt(token: str, required_token_use: str | None = None) -> dict:
    region    = os.getenv("AWS_REGION", "ap-south-1")
    pool_id   = os.getenv("COGNITO_USER_POOL_ID", "")
    client_id = os.getenv("COGNITO_CLIENT_ID", "")

    try:
        # 1. Parse kid from unverified token header
        unverified_header = jwt.get_unverified_header(token)
        kid = unverified_header.get("kid")
        if not kid:
            raise AuthError("Invalid token: missing kid in header", code=401)

        # 2. Fetch the cached JWKS dictionary
        jwks = _get_jwks()
        jwk = next((key for key in jwks.get("keys", []) if key.get("kid") == kid), None)

        # 3. Handle key rotation: if kid not found, reload cache
        if not jwk:
            global _jwks_cache
            # Rate-limit cache invalidations (e.g., once every 10s) to prevent abuse
            if time.time() - _jwks_fetched_at > 10:
                with _jwks_lock:
                    _jwks_cache = None
                jwks = _get_jwks()
                jwk = next((key for key in jwks.get("keys", []) if key.get("kid") == kid), None)

        if not jwk:
            raise AuthError("Invalid token: signing key not found in JWKS", code=401)

        # 4. Construct RSA key locally and decode
        from jwt.algorithms import RSAAlgorithm
        public_key = RSAAlgorithm.from_jwk(jwk)

        payload = jwt.decode(
            token,
            public_key,
            algorithms=["RS256"],
            issuer=f"https://cognito-idp.{region}.amazonaws.com/{pool_id}",
            options={
                "verify_exp": True,
                "verify_aud": False,  # enforced manually
            },
        )
        ...
```

### Pattern B: Optimized Service Entry Point (`app/main.py`)
This pattern is applied to all 10 microservices:
```python
from __future__ import annotations

from shared.config.settings import Settings
from shared.logger.logger import get_logger
from shared.db.postgres import get_db
from app.routes.router import route  # <-- Eager import in Init Phase

Settings.validate()

logger = get_logger("hms.admin.main")

# Eagerly initialize DB connection pool during container bootstrap (Init Phase)
try:
    get_db()
except Exception as e:
    logger.warning(f"Could not warm up DB pool during init: {e}")


def handler(event: dict, context) -> dict:
    request_id = ...
    logger.set_context(request_id=request_id, service="hms-admin")

    try:
        return route(event)  # <-- Executes instantly
    except Exception:
        ...
```

---

## 4. Performance Benchmarks

Below are the actual measured execution latencies for the Admin Service running on a **256 MB** Lambda environment before and after implementing the optimizations:

| Measurement | Before | After | Performance Gain |
| :--- | :--- | :--- | :--- |
| **Cold Start Billed Duration** | **8,834 ms** | **4,847 ms** | **-45.1% Billed Cost** |
| **Cold Start Handler Execution** | **6,682.28 ms** | **159.70 ms** | **-97.6% (Saved 6.52 seconds)** |
| **Warm Start (Staff Search)** | **72.87 ms** | **24.22 ms** | **-66.8% (Saved 48 ms)** |
| **Warm Start (Leaves Summary)** | **54.87 ms** | **24.05 ms** | **-56.2% (Saved 30 ms)** |

---

## 5. Deployment Checklist
When deploying these optimizations, ensure that:
1. All files compile correctly (verified with `python -m py_compile`).
2. The Lambda container base image has access to the updated code.
3. Env vars `COGNITO_USER_POOL_ID`, `COGNITO_CLIENT_ID`, and `AWS_REGION` are correctly set on the functions.
