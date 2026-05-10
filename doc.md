flowchart TD

    A[Developer creates feature branch from dev] --> B[Code locally]

    B --> C[Open PR to dev]

    C --> D[CI runs on PR]
    D --> D1[Lint]
    D --> D2[Test]
    D --> D3[Build]
    D --> D4[Security scan]

    D --> E{CI passed?}

    E -- No --> F[Fix code in feature branch]
    F --> D

    E -- Yes --> G[Merge into dev]

    G --> H[Build Docker image once]
    H --> I[Tag image with Git SHA]
    I --> J[Push image to registry]

    J --> K[Deploy latest dev to DEV environment]

    K --> L[Ready for integration testing?]

    L --> M[Create PR dev to integration]

    M --> N[CI runs again on integration]
    N --> O[Deploy to Integration/UAT environment]

    O --> P{QA approved?}

    P -- No --> Q[Fix in feature branch]
    Q --> C

    P -- Yes --> R[Create PR integration to main]

    R --> S[Final CI checks]

    S --> T[Release approval]

    T --> U[Tag official version v1.2.0]

    U --> V[Deploy same image to PROD]







    flowchart TD

    A[Developer creates feature branch from dev] --> B[Open PR to dev]

    B --> C["AUTOMATIC CI
    - lint
    - tests
    - build check
    - security scan"]

    C --> D{CI passed?}

    D -- No --> E[Developer fixes code]
    E --> C

    D -- Yes --> F["MANUAL
    Code review + approve"]

    F --> G["MANUAL
    Merge to dev"]

    G --> H["AUTOMATIC
    Build Docker image/package ONCE"]

    H --> I["AUTOMATIC
    Tag image with commit SHA
    example: abc123"]

    I --> J["AUTOMATIC
    Push image to registry"]

    J --> K["AUTOMATIC
    Deploy latest image to DEV"]

    K --> L["MANUAL
    Decide release candidate"]

    L --> M["MANUAL
    Promote same image to integration/staging"]

    M --> N["AUTOMATIC
    Retag image as RC
    example: 1.2.0-rc.1"]

    N --> O["AUTOMATIC
    Deploy RC to staging"]

    O --> P{QA passed?}

    P -- No --> Q[Fix bug and create new RC]
    Q --> B

    P -- Yes --> R["MANUAL
    Approve production release"]

    R --> S["MANUAL
    Merge integration to main"]

    S --> T["AUTOMATIC
    Retag SAME image as official version
    example: v1.2.0"]

    T --> U["AUTOMATIC
    Create GitHub Release + changelog"]

    U --> V["MANUAL
    Production deployment approval"]

    V --> W["AUTOMATIC
    Deploy SAME image to PROD"]





    Simple Mental Model
Branches
Branch	Purpose
dev	developers combine features
integration	QA/staging/testing
main	production
CI Behavior
During PR to dev

GitHub temporarily tests:

current dev + new feature

CI runs:

lint
tests
build validation
security scan

NO official image/version yet.

Build Stage
After merge to dev

We:

build Docker image/package ONCE
tag with unique SHA

Example:

myapp:abc123

This is the real reusable artifact.

Deployment Rules
DEV environment

Purpose:

fast developer integration

Best practice:

cancel-in-progress: true

Meaning:

newest deployment wins
old deployment jobs cancelled

Example:

aaa111 cancelled
bbb222 deployed

DEV always runs:

latest shared dev state
Important

Code is NOT replaced.

dev branch grows:

Feature A
+ Feature B
+ Feature C

New deploys include previous features.

STAGING / INTEGRATION

Purpose:

stable QA testing

Best practice:

cancel-in-progress: false

Meaning:

deployments queue
only one RC deployed at a time
Release Candidate (RC)

Example:

1.2.0-rc.1
1.2.0-rc.2
1.2.0-rc.3

If QA fails rc1:

create rc2
deploy rc2
rc2 replaces rc1 in staging

Registry stores all RCs, but staging usually runs only ONE active RC.

Production

Only approved RC becomes production version.

Example:

1.2.0-rc.3
↓
v1.2.0

Best practice:

manual approval
protected environment
queue deployments
rollback strategy
Most Important Senior DevOps Rule
Build once.
Promote same artifact everywhere.

Meaning:

DO NOT rebuild in staging
DO NOT rebuild in production

Reuse SAME image:

myapp:abc123

across:

DEV
STAGING
PROD
Automation vs Manual
AUTOMATIC 🤖
CI checks
build image
tag SHA
push registry
deploy DEV
create RC tags
create release tags
generate changelog
MANUAL 👨‍💻
code review
PR approval
choosing RC
QA approval
production approval
Final Best-Practice Rules
DEV
fast
latest wins
shared environment
STAGING
stable
QA-safe
one RC at a time
PROD
controlled
auditable
safe
