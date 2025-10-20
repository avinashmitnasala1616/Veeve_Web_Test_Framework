# Framework Architecture Diagrams

> GitHub can render **Mermaid** directly in Markdown. A PlantUML version is also provided for PNG generation.

---

## 1) High-Level System (Mermaid)

```mermaid
flowchart LR
  subgraph CP["core-product-tests"]
    CPF[Features]:::res
    CPS[Steps]:::code
    CPP[Pages]:::code
  end

  subgraph DP1["derived-product1-tests"]
    D1F[Features]:::res
    D1S[Steps]:::code
    D1TD[CSV testdata]:::res
  end

  subgraph DP2["derived-product2-tests"]
    D2F[Features]:::res
    D2S[Steps]:::code
    D2P[Pages]:::code
  end

  subgraph AF["automation-framework (shared)"]
    DF[DriverFactory]:::core
    WU[WaitUtils]:::core
    CFG[ConfigManager]:::core
    EX[ExtentManager]:::core
    U1[Dom/Scroll/Popup/Date/CSV Utils]:::core
    HK[Hooks]:::core
  end

  AUT[(Web App<br/>Under Test)]
  CI[CI/CD\nJenkins/GitHub Actions]
  RPT[(HTML/Extent\n+ Screenshots)]

  CPF --> CPS --> CPP --> DF
  D1F --> D1S --> DF
  D2F --> D2S --> D2P --> DF

  CPS -->|reuses| WU
  D1S -->|reads| D1TD
  D2S -->|reuses| WU
  CPS --> EX
  D1S --> EX
  D2S --> EX
  EX --> RPT

  DF --> AUT
  CI --> CP
  CI --> DP1
  CI --> DP2

  classDef core fill:#e8f0fe,stroke:#4a67d6,stroke-width:1px;
  classDef code fill:#e8f8e8,stroke:#3b7f3b,stroke-width:1px;
  classDef res fill:#fff8e1,stroke:#cc8f00,stroke-width:1px;
```

---

## 2) Dependency Direction (Mermaid)

```mermaid
graph TD
  CP[core-product-tests] --> AF[automation-framework]
  DP1[derived-product1-tests] --> AF
  DP2[derived-product2-tests] --> AF
  AF --> RPT[reports (Extent/HTML)]
```

---

## 3) PlantUML Component View

```plantuml
@startuml
skinparam componentStyle rectangle
skinparam shadowing false
skinparam packageStyle rectangle

package "Tests" {
  [core-product-tests] as CP
  [derived-product1-tests] as DP1
  [derived-product2-tests] as DP2
}

package "Shared Core" {
  [automation-framework] as AF
  [DriverFactory] as DF
  [WaitUtils] as WU
  [ConfigManager] as CFG
  [ExtentManager] as EX
  [Utils (Dom/Scroll/Popup/CSV/Date)] as UT
  [Hooks] as HK
}

[Web App Under Test] as AUT
[CI/CD (Jenkins/GitHub Actions)] as CI
[Reports (Extent/HTML + screenshots)] as RPT

CP --> AF
DP1 --> AF
DP2 --> AF

AF ..> DF
AF ..> WU
AF ..> CFG
AF ..> EX
AF ..> UT
AF ..> HK

DF --> AUT
EX --> RPT

CI --> CP
CI --> DP1
CI --> DP2
@enduml
```

**Render PlantUML to PNG (example via Docker):**
```bash
docker run --rm -v "$PWD":/work plantuml/plantuml -tpng ARCHITECTURE.puml
```
