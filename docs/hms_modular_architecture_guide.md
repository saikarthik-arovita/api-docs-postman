# 🎨 HMS Visual Theme

| Component | Color   | Meaning                   |
| --------- | ------- | ------------------------- |
| 🟦 Blue   | #2563EB | Core HMS Platform         |
| 🟩 Green  | #16A34A | Active Modules            |
| 🟧 Orange | #EA580C | Clinical Operations       |
| 🟪 Purple | #7C3AED | Diagnostics               |
| 🟥 Red    | #DC2626 | Emergency / Critical Care |
| 🟨 Yellow | #CA8A04 | Administrative Functions  |

---

# 🏥 HMS Architecture

```mermaid
flowchart TD

    HMS["🏥 HMS Platform"]

    OPD["🟩 OPD"]
    IPD["🟩 IPD"]
    ICU["🟥 ICU"]
    LAB["🟪 Laboratory"]
    RAD["🟪 Radiology"]
    OT["🟧 Operation Theatre"]

    HMS --> OPD
    HMS --> IPD
    HMS --> ICU
    HMS --> LAB
    HMS --> RAD
    HMS --> OT

    classDef core fill:#2563EB,color:white,stroke:#1E3A8A,stroke-width:3px;
    classDef module fill:#16A34A,color:white;
    classDef critical fill:#DC2626,color:white;
    classDef diagnostics fill:#7C3AED,color:white;
    classDef clinical fill:#EA580C,color:white;

    class HMS core;
    class OPD,IPD module;
    class ICU critical;
    class LAB,RAD diagnostics;
    class OT clinical;
```

---

# 🎛️ Module Activation Flow

```mermaid
flowchart LR

A["👨‍💼 Admin"] --> B["🟩 Enable Module"]
B --> C["🏢 Department Activated"]
C --> D["👨‍⚕️ Doctors Available"]
D --> E["📋 Forms Available"]
E --> F["✅ Services Live"]

classDef start fill:#2563EB,color:white;
classDef active fill:#16A34A,color:white;
classDef output fill:#EA580C,color:white;

class A start;
class B,C,D active;
class E,F output;
```

---

# 🏥 Patient Journey

```mermaid
flowchart TD

A["👤 Patient Arrival"]
B["📝 Registration"]
C["👨‍⚕️ Consultation"]
D["🔍 Diagnosis"]

E["🟪 Laboratory"]
F["🟪 Radiology"]
G["🟩 OPD"]
H["🟧 Surgery"]
I["🟥 IPD"]

J["💰 Billing"]
K["🏠 Discharge"]

A --> B --> C --> D

D --> E
D --> F
D --> G
D --> H
D --> I

E --> J
F --> J
G --> J
H --> J
I --> J

J --> K

classDef patient fill:#2563EB,color:white;
classDef diagnostics fill:#7C3AED,color:white;
classDef clinical fill:#16A34A,color:white;
classDef surgery fill:#EA580C,color:white;
classDef critical fill:#DC2626,color:white;
classDef billing fill:#CA8A04,color:white;

class A,B,C,D patient;
class E,F diagnostics;
class G clinical;
class H surgery;
class I critical;
class J,K billing;
```

---

# 🚑 Emergency Flow

```mermaid
flowchart TD

A["🚑 Emergency Arrival"]
B["🩺 Triage"]

C["🟩 OPD"]
D["🟥 IPD"]
E["🟥 ICU"]

A --> B

B --> C
B --> D
B --> E

classDef emergency fill:#DC2626,color:white;
classDef opd fill:#16A34A,color:white;

class A,B emergency;
class C opd;
class D,E emergency;
```

---

# 🧠 Smart Dependency Logic

```mermaid
flowchart TD

IPD["🛏️ IPD"]

ICU["🟥 ICU"]
NURSE["👩‍⚕️ Nursing"]
OT["🔪 OT"]

IPD --> ICU
IPD --> NURSE
IPD --> OT

classDef parent fill:#2563EB,color:white;
classDef child fill:#16A34A,color:white;

class IPD parent;
class ICU,NURSE,OT child;
```

---

# 🌍 Multi-Branch Configuration

```mermaid
flowchart TD

GROUP["🏥 Hospital Group"]

A["🏥 Branch A<br/>OPD + Lab"]
B["🏥 Branch B<br/>OPD + IPD"]
C["🏥 Branch C<br/>Full Hospital"]

GROUP --> A
GROUP --> B
GROUP --> C

classDef group fill:#2563EB,color:white;
classDef branch fill:#16A34A,color:white;

class GROUP group;
class A,B,C branch;
```

---

# 🚀 Plug-and-Play Vision

```mermaid
flowchart LR

A["🟩 Enable Module"]
B["🏢 Department Ready"]
C["👨‍⚕️ Staff Ready"]
D["📋 Forms Ready"]
E["🏥 Patient Services Live"]

A --> B --> C --> D --> E

classDef step fill:#16A34A,color:white,stroke:#14532D,stroke-width:2px;

class A,B,C,D,E step;
```
