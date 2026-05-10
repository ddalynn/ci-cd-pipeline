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


You are a senior DevOps engineer. Help me implement a GitHub Actions CI/CD pipeline with this branching strategy:

Branches:
- dev = active development / shared dev environment
- integration = UAT / staging / QA
- main = production

Main rules:
1. No direct push to dev, integration, or main.
2. Developers create feature branches from dev.
3. PR to dev must run CI: lint, test, build validation, security scan.
4. PR must be reviewed before merge.
5. After merge to dev:
   - build Docker image ONCE
   - tag image with commit SHA
   - push image to container registry
   - deploy automatically to DEV environment
6. DEV deployment uses concurrency:
   - cancel-in-progress: true
   - latest deployment wins
7. Auto-create PR from dev to integration.
8. Integration/UAT:
   - deploy one RC at a time
   - use concurrency cancel-in-progress: false
   - retag same image as x.y.z-rc.N
   - do not rebuild image
9. Auto-create PR from integration to main.
10. Main/production:
   - require manual approval
   - retag approved RC as official version vX.Y.Z
   - deploy same image to production
   - do not rebuild image
11. Use “build once, promote same artifact everywhere.”
12. Use good auto PR titles and bodies.

Please create the full implementation with GitHub Actions YAML files:
- .github/workflows/ci.yml
- .github/workflows/build-dev-image.yml
- .github/workflows/deploy-dev.yml
- .github/workflows/auto-pr-dev-to-integration.yml
- .github/workflows/promote-to-uat.yml
- .github/workflows/auto-pr-integration-to-main.yml
- .github/workflows/release-prod.yml

Also include:
- required repository secrets
- required environment variables
- branch protection rules
- GitHub environment protection settings
- example Docker image tagging strategy
- example auto PR message template
- explanation of each workflow
- security best practices for GITHUB_TOKEN permissions

Assume the app is containerized with Docker and deployed to Kubernetes.
Use placeholders for registry URL, Kubernetes config, namespace, app name, and image name.
Make the YAML production-quality and easy to adapt.

Important: Do not rebuild the image in UAT or production. The image built after merge to dev must be the exact same artifact promoted to integration and main. Use immutable SHA tags and only retag for RC/prod versions.


Do not make one giant workflow.
Do not make too many tiny workflows.
Use 5–6 workflows grouped by responsibility.



You are a senior DevOps engineer. Design and implement a GitHub Actions CI/CD system for a repository that supports TWO deployment/artifact types:

1. Package artifact:
   - build package
   - publish package to JFrog Artifactory

2. Docker/Kubernetes artifact:
   - build Docker image
   - push Docker image to registry
   - deploy to Kubernetes

Branch strategy:
- dev = active development / shared dev environment
- integration = UAT / staging / QA
- main = production

Core principles:
- Build once, promote the same artifact everywhere.
- Do not rebuild package/image in integration or production.
- Use immutable SHA tags for dev artifacts.
- Use RC tags for integration/UAT.
- Use official semantic version tags for production.
- No direct push to dev, integration, or main.
- PR required, CI required, review required.

Please create production-quality GitHub Actions workflows with clear naming that shows which workflows are automatic and which are manual.

Recommended workflow files:

1. `.github/workflows/auto-ci-pr.yml`
   Purpose: AUTOMATIC
   Trigger: pull_request to dev, integration, main
   Runs:
   - lint
   - tests
   - build validation
   - security scan
   Must not publish artifacts.

2. `.github/workflows/auto-build-publish-dev-package.yml`
   Purpose: AUTOMATIC
   Trigger: push to dev
   Runs:
   - build package once
   - version/tag package with commit SHA
   - publish package to JFrog
   - save artifact metadata for later promotion

3. `.github/workflows/auto-build-deploy-dev-kube.yml`
   Purpose: AUTOMATIC
   Trigger: push to dev
   Runs:
   - build Docker image once
   - tag image with commit SHA
   - push Docker image to registry
   - deploy latest image to DEV Kubernetes namespace
   Concurrency:
   - group: deploy-dev-kube
   - cancel-in-progress: true

4. `.github/workflows/auto-pr-dev-to-integration.yml`
   Purpose: AUTOMATIC
   Trigger: push to dev or workflow_dispatch
   Runs:
   - create or update PR from dev to integration
   - generate clear PR title and body
   - include commits, artifact SHA, Docker image tag, package version, and action required
   PR title:
   `chore(release): promote dev to integration`
   PR body should include:
   - source branch
   - target branch
   - included commits
   - package artifact reference
   - Docker image reference
   - action required: review CI and approve for UAT

5. `.github/workflows/manual-promote-package-to-uat.yml`
   Purpose: MANUAL
   Trigger: workflow_dispatch
   Inputs:
   - source_sha
   - rc_version, example: 1.4.0-rc.1
   Runs:
   - find package built from source_sha
   - retag/promote same package in JFrog as rc_version
   - do not rebuild package
   - optionally deploy/install package to UAT if applicable
   Concurrency:
   - group: promote-package-uat
   - cancel-in-progress: false

6. `.github/workflows/manual-promote-image-to-uat-kube.yml`
   Purpose: MANUAL
   Trigger: workflow_dispatch
   Inputs:
   - source_sha
   - rc_version, example: 1.4.0-rc.1
   Runs:
   - find Docker image built from source_sha
   - retag same Docker image as rc_version
   - do not rebuild image
   - deploy rc_version to UAT Kubernetes namespace
   Concurrency:
   - group: deploy-uat-kube
   - cancel-in-progress: false

7. `.github/workflows/auto-pr-integration-to-main.yml`
   Purpose: AUTOMATIC
   Trigger: push to integration or workflow_dispatch
   Runs:
   - create or update PR from integration to main
   - generate clear PR title and body
   PR title:
   `chore(release): prepare production release vX.Y.Z`
   PR body should include:
   - source branch
   - target branch
   - approved RC version
   - QA status placeholder
   - package artifact reference
   - Docker image reference
   - rollback plan
   - action required: final production approval

8. `.github/workflows/manual-release-package-prod.yml`
   Purpose: MANUAL
   Trigger: workflow_dispatch
   Inputs:
   - rc_version, example: 1.4.0-rc.1
   - prod_version, example: 1.4.0
   Runs:
   - promote same JFrog package from rc_version to prod_version
   - do not rebuild package
   - create Git tag vX.Y.Z if needed
   - create GitHub Release notes if needed
   Environment:
   - production
   Require GitHub Environment approval.

9. `.github/workflows/manual-release-image-prod-kube.yml`
   Purpose: MANUAL
   Trigger: workflow_dispatch
   Inputs:
   - rc_version, example: 1.4.0-rc.1
   - prod_version, example: 1.4.0
   Runs:
   - retag same Docker image from rc_version to prod_version
   - do not rebuild image
   - deploy prod_version to PROD Kubernetes namespace
   - create Git tag vX.Y.Z if needed
   - create GitHub Release notes if needed
   Concurrency:
   - group: deploy-prod-kube
   - cancel-in-progress: false
   Environment:
   - production
   Require GitHub Environment approval.

Naming rules:
- Prefix automatic workflows with `auto-`
- Prefix manual workflows with `manual-`
- Workflow display names should clearly show:
  - AUTO or MANUAL
  - artifact type: package or docker/kube
  - target environment: dev, uat, prod

Use these workflow display names:
- `AUTO - CI Pull Request`
- `AUTO - Build & Publish Dev Package to JFrog`
- `AUTO - Build & Deploy Dev Image to Kubernetes`
- `AUTO - Create PR Dev to Integration`
- `MANUAL - Promote Package to UAT`
- `MANUAL - Promote Image to UAT Kubernetes`
- `AUTO - Create PR Integration to Main`
- `MANUAL - Release Package to Production`
- `MANUAL - Release Image to Production Kubernetes`

Security requirements:
- Use least privilege `permissions`.
- Do not expose secrets in logs.
- Use GitHub environments for UAT and production.
- Production workflows must require manual approval.
- Use branch protection on dev, integration, and main.
- Use concurrency to prevent simultaneous deployments.
- Use immutable tags.
- Never use `latest` for production.
- Do not rebuild artifacts after dev build.

Please generate:
1. Full GitHub Actions YAML files.
2. Example reusable shell scripts for:
   - build package
   - publish to JFrog
   - build Docker image
   - push Docker image
   - retag/promote package
   - retag/promote Docker image
   - deploy to Kubernetes
3. Required GitHub secrets.
4. Required variables.
5. Branch protection settings.
6. GitHub environment settings.
7. Clear explanation of each workflow.
8. A Mermaid diagram showing the full pipeline.
Important: make all deployment commands provider-adaptable. For JFrog, use placeholders for Artifactory URL, repo name, username/token. For Kubernetes, use placeholders for kubeconfig, namespace, deployment name, container name, registry URL, and image name.


