## CI/CD Fundamentals & Workflows 

### **1. What is CI/CD and what core problems does it solve?**
**Answer:** 

CI/CD stands for **Continuous Integration** and **Continuous Delivery/Deployment**. It is **a software development practice that automates building, testing, and releasing code**. CI merges code changes frequently into a central repository with automated tests, while CD automatically prepares or releases those changes to production. 

#### Core Components of CI/CD

- **Continuous Integration (CI):** Developers merge code updates into a shared main branch often (multiple times a day). Each commit triggers automated builds and tests to spot bugs immediately. 
- **Continuous Delivery (CD):** Validated code from CI is automatically built, packaged, and prepped for a release. It requires a manual click or approval before going live to production. 
- **Continuous Deployment (CD):** An extension of continuous delivery where every change that passes all automated tests goes straight into the live production environment without human intervention. 

#### Core Problems CI/CD Solves

- **"Integration Hell" & Late-Stage Conflicts:** In traditional workflows, developers worked in isolation for weeks. Merging massive codebases all at once caused severe conflicts. CI solves this by forcing frequent, small merges that are easy to fix. 
- **Manual & Error-Prone Deployments:** Manually FTP-ing files or running shell scripts on servers leads to human error and downtime. CI/CD automates the build-and-release pipeline.
- **Slow Feedback Loops:** Waiting until the end of a multi-month project to test software means bugs are deeply embedded and costly to fix. Automated tests run on every single commit, offering instant feedback.
- **Code Freezes & Release Anxiety:** Big-bang releases make teams terrified of breaking production. Automated canary rollouts, blue/green deployments, and test suites make shipping updates routine and low-risk.

### **2. What is Continuous Integration (CI)?**
**Answer:** 

Continuous Integration (CI) is **a software development practice where team members regularly merge their code changes into a shared central repository**. Each merge triggers an automated build and a suite of tests to catch bugs and integration errors early. 

#### How Continuous Integration Works

- **Frequent Commits:** Developers write code in small pieces and push updates to a version control system like **GitHub** or **GitLab** multiple times a day.
- **Automated Builds:** A CI server compiles the code as soon as a new change is detected to make sure it builds cleanly. 
- **Automated Testing:** The pipeline runs unit tests, integration tests, and checks to verify that the new code does not break existing features. 
- **Immediate Feedback:** Developers receive quick alerts if the build fails, allowing them to fix problems right away. 

#### Key Benefits

- **Early Bug Detection:** Issues are spotted and fixed when they are small and easy to trace.
- **Less Integration Chaos:** Merging code continuously avoids massive, painful merge conflicts at the end of a project.
- **Time Savings:** Automation replaces tedious manual building and testing routines. 

#### Common CI Tools

- [**Jenkins**](https://www.jenkins.io/)
- [**GitHub Actions**](https://github.com/features/actions)
- [**GitLab CI/CD**](https://about.gitlab.com/stages-devops-lifecycle/continuous-integration/)
- [**CircleCI**](https://circleci.com/)

### **3. What is Continuous Delivery (CD)?**
**Answer:**

**Continuous delivery (CD)** is **a software development practice where code changes are automatically built, tested, and prepared for a release to production**.

It expands upon **Continuous Integration (CI)** by pushing every code change that passes the build stage into a testing or staging environment.

#### Key Characteristics

- **Always Releasable:** Your codebase is always in a deployment-ready state because every change goes through a standardized, automated test pipeline. 
- **Automated Pipeline:** Code commits trigger automated builds, unit tests, integration tests, and performance checks. 
- **Manual Production Gate:** Unlike continuous deployment, continuous delivery requires a **manual approval** from a developer or operator before pushing the update to the live production environment. 

#### Continuous Delivery vs. Continuous Deployment

- **Continuous Delivery:** Code changes are staged and ready, but a human must click a button or give final approval to release to production. 
- **Continuous Deployment:** Every code change that passes all automated tests goes straight to production automatically with **no human intervention**. 

#### Core Benefits

- **Faster Time-to-Market:** Reduces bottlenecks caused by manual handoffs between development and operations teams.
- **Early Bug Detection:** Automated tests catch regressions and errors early in the cycle before they reach customers.
- **Lower Risk:** Releasing smaller, incremental updates makes troubleshooting much simpler than deploying massive updates all at once.

### **4. What is Continuous Deployment (CD)?**
**Answer:** 

**Continuous deployment (CD)** is **a software development practice where code changes are automatically released into a live production environment after passing predefined automated tests**. 

#### How It Works

- **Code Commit:** Developers push and merge code changes into a shared version control system.
- **Automated Testing:** The code runs through a series of automated builds, unit tests, integration checks, and security scans.
- **Direct Release:** If all tests pass, the change moves straight to production automatically—**with zero human or manual approval required**. 

#### Continuous Deployment vs. Continuous Delivery

- **Continuous Delivery:** Code is automatically built, tested, and staged for a release, but a human must **manually click a button** to approve the final deployment to production.
- **Continuous Deployment:** The entire pipeline is fully automated from commit to production. There is no manual gatekeeper.

### **5. What are the key stages of a production CI/CD pipeline?**
**Answer:**

A production CI/CD (Continuous Integration and Continuous Delivery/Deployment) pipeline automates code progression from source control to live environments. Its key stages include **source/commit, build, automated testing, security scanning, artifact management, staging/approval, and production deployment followed by monitoring**.

#### Key Stages of a CI/CD Pipeline

- **Source / Code Commit:** Developers push code to a version control system like Git (GitHub or GitLab), which fires a webhook to trigger the pipeline. 
- **Build & Compile:** The system pulls the source code, installs dependencies, compiles binaries, or builds container images (e.g., via Docker). 
- **Automated Testing (CI):** Fast automated checks run, including unit tests, linting, and basic code-quality analysis (e.g., SonarQube) to catch regression issues early. 
- **Security & Vulnerability Scans:** Static Application Security Testing (SAST) and container/dependency scans (e.g., Trivy) check for open CVEs or secrets.
- **Artifact Storage:** Validated builds and packages are versioned and pushed to an artifact repository (e.g., JFrog Artifactory or Nexus). 
- **Staging & Acceptance Testing:** The artifact deploys to a pre-production or staging environment where integration, end-to-end (E2E), and user acceptance tests (UAT) execute. 
- **Approval Gate (Continuous Delivery) or Auto-Deploy (Continuous Deployment):** Either a manual sign-off is granted by an operator or the pipeline automatically pushes the release live.
- **Production Release & Strategies:** The app rolls out using zero-downtime methods like **blue/green deployments** or **canary releases**. 
- **Monitoring & Feedback:** Post-deployment tools (e.g., Prometheus, Grafana) track performance metrics, logs, and error budgets, triggering automatic rollbacks if anomalies occur.

### **6. What is an Artifact in CI/CD?**
**Answer:** 

In CI/CD, an **artifact** is **any file, binary, package, or document produced during the build, test, or compilation process of a pipeline**. It represents the tangible output of source code—such as a compiled `.jar` file, a Docker container image, or a test report—that moves between pipeline stages or gets deployed to production. 

#### Common Types of Artifacts

- **Compiled binaries and packages:** Java `.jar` or `.war` files, `.exe` installers, `.deb`/`.rpm` Linux packages, or Python `.whl` files.
- **Container images:** Docker or OCI images ready to run in Kubernetes or cloud servers.
- **Configuration and infrastructure templates:** Helm charts, Terraform scripts, or CloudFormation templates.
- **Reports and logs:** JUnit test results, code coverage files, or build logs used for debugging.

#### Why Artifacts Matter in CI/CD

- **Passing data between jobs:** An early CI job builds the code and saves the output as an artifact, which later testing or packaging jobs automatically download and use.
- **Immutability and consistency:** Using the exact same built artifact across testing, staging, and production guarantees that the code tested is identical to the code released. 
- **Traceability and rollbacks:** Every artifact ties back to a specific Git commit and build number. If a live release fails, teams can quickly roll back to a previous safe artifact version. 

#### Where Artifacts Live

- **Temporary CI storage:** Built-in storage inside tools like **GitLab CI/CD Job Artifacts** or Jenkins for short-term passing between pipeline steps. 
- **Dedicated artifact repositories:** Long-term, version-controlled systems like **JetBrains TeamCity Artifact Repositories**, JFrog Artifactory, Nexus, or cloud registries (like Google Artifact Registry) to store, scan, and manage release candidates.

### **7. What is the "Build Once, Deploy Anywhere" principle?**
**Answer:** 

The "Build Once, Deploy Anywhere" principle is **a DevOps and software engineering best practice**. It states that an application should be compiled and packaged into a single, immutable artifact (such as a Docker image, JAR file, or Helm chart) just once during the CI/CD pipeline, and that exact same binary or package should be promoted across all environments—from Development and QA to Staging and Production—without rebuilding. 

#### Core Concepts

- **Immutable Artifacts**: The code and its dependencies are locked into a single package. You never re-compile code as it moves closer to production. 
- **Environment Configuration Separation**: Instead of baking environment-specific settings (like database URLs or API keys) into the build, configuration is injected externally at runtime (via environment variables, secret managers, or external config files).
- **Parity Across Stages**: Testing happens on the exact same bits that will run in production, eliminating the dreaded "works on my machine" problem or build discrepancies between tiers.

#### Key Benefits

- **Reliability & Predictability**: Knowing that the staged artifact is bit-for-bit identical to production reduces unexpected bugs and deployment failures. 
- **Speed**: Skipping redundant compile and packaging steps at every single testing or deployment stage speeds up your pipeline. 
- **Traceability & Auditing**: Every artifact can be uniquely tagged (using Git commit hashes or build IDs), making rollbacks clean and tracking issues straightforward. 
- **Security**: Secrets and environment-specific credentials are kept out of the build and image registry, applied safely only when launched.

### **8. What is a Build Matrix in CI/CD?**
**Answer:** 

A build matrix in CI/CD is **a configuration strategy that automatically runs the same build, test, or deployment process across multiple combinations of environments, operating systems, language versions, or database engines from a single set of rules**.

#### How It Works

- **Parameter Definition:** You declare variables and values—such as `os: [ubuntu, windows]` or `node-version: [18, 20]`—inside your pipeline configuration file (like GitHub Actions YAML or GitLab CI). 
- **Automatic Expansion:** The CI/CD engine computes the Cartesian product (every possible combination) and dynamically generates a separate parallel job for each permutation. 
- **Parallel Execution:** All generated jobs run at the same time on independent runners, drastically cutting down total test feedback time. 

#### Common Use Cases

- **Multi-Platform Testing:** Verifying that a desktop or command-line tool works cleanly on Linux, macOS, and Windows.
- **Language & Runtime Compatibility:** Testing a package against multiple active versions of a language (e.g., Python 3.10, 3.11, and 3.12).
- **Service Integrations:** Running your test suite against different versions of a dependency, such as PostgreSQL 14 and 16.
- **Test Sharding:** Splitting a massive test suite across multiple parallel workers to speed up slow builds. 

#### Popular Tool Implementations

- **GitHub Actions:** Uses `strategy.matrix` blocks to expand workflow jobs.
- **GitLab CI/CD:** Uses `parallel: matrix:` syntax to instantiate job variations.
- **Travis CI:** Uses matrix expansion and allows specific row exclusions or experimental allowances (`jobs.exclude`, `jobs.allow_failures`).
- **Jenkins:** Uses Groovy-based scripted pipelines or declarative matrix projects to handle multi-axis builds.

### **9. What is Pipeline Caching vs Artifact Storage?**
**Answer:** 

Pipeline caching stores reusable dependency files (like `node_modules` or Maven packages) across multiple pipeline runs to speed up future builds. Artifact storage preserves specific build outputs (like compiled binaries, logs, or test reports) to pass between pipeline stages or keep for later inspection. 

#### Key Differences

- **Primary Purpose:**
  - **Pipeline Caching:** Boosts performance by avoiding repeated downloads.
  - **Artifact Storage:** Preserves and shares final or intermediate build results. 
- **Data Type:**
  - **Pipeline Caching:** External dependencies and tool libraries (e.g., npm cache, pip cache, m2 repository).
  - **Artifact Storage:** Generated code, Docker images, compiled binaries, and test reports. 
- **Lifecycle:**
  - **Pipeline Caching:** Shared and reused across distinct pipeline runs based on a lookup key (often a hash of a lockfile).
  - **Artifact Storage:** Tied directly to a specific pipeline run and passed to downstream jobs or saved for release. 
- **Guarantees:**
  - **Pipeline Caching:** Optional and non-critical; if a cache misses, the pipeline will still succeed by downloading the dependencies fresh.
  - **Artifact Storage:** Required and critical; if an artifact fails to generate or pass to a deployment stage, the pipeline usually fails. 

#### Quick Comparison

| **Feature** | **Pipeline Caching** | **Artifact Storage** |
|---|---|---|
| **Goal** | Speed optimization | Output preservation & sharing |
| **Typical Content** | Package manager caches (`node_modules`, `.m2`) | Binaries, zips, logs, test reports |
| **Scope** | Across future runs (via keys) | Within or after the current run |
| **Failure Impact** | Slower build (falls back to download) | Broken pipeline / failed deployment |

### **10. What is Semantic Versioning (SemVer)?**
**Answer:** 

Semantic Versioning (SemVer) is **a standard numbering system for software releases that uses a Major.Minor.Patch format** (such as `2.4.1`). Each number explains the scope of changes and helps users know if an upgrade is safe or will break compatibility. 

#### The Version Format (X.Y.Z)

- **Major (X):** Increased for **breaking changes** that are not backward-compatible.
- **Minor (Y):** Increased for **new features** added in a backward-compatible way.
- **Patch (Z):** Increased for **backward-compatible bug fixes**. 

#### Additional Tags

- **Pre-release:** Added with a hyphen (e.g., `1.0.0-alpha.1`) to show an unstable or test build.
- **Build metadata:** Added with a plus sign (e.g., `1.0.0+20261021`) to track build or commit info.

### **11. What are Conventional Commits?**
**Answer:** 

**Conventional Commits** is **a lightweight specification that adds a human- and machine-readable structure to Git commit messages**. It provides a standardized template so that anyone reading the history can quickly understand the intent of a code change, and automated tools can parse the history to manage versioning and changelogs.

#### Basic Structure of a Commit Message

A conventional commit message adheres to this exact layout: 

**text**

```text
<type>[optional scope]: <description>

[optional body]

[optional footer(s)]
```

Use code with caution.

- **Type:** Mandatory label indicating the category of change (e.g., `feat`, `fix`).
- **Scope**: Optional context enclosed in parentheses to specify the area of the codebase affected (e.g., `auth`, `api`, `ui`).
- **Description**: A short, clear summary of the code change.
- **Body**: Optional deeper explanation of the motivation or changes.
- **Footer**: Optional metadata, such as tracking issue numbers or signaling a breaking change.

#### Common Commit Types

Based on the **Conventional Commits Specification**, the standard types include: 

- **`feat`**: Introduces a new feature to the codebase.
- **`fix`**: Patches a bug in the code.
- **`docs`**: Changes or additions exclusively to documentation.
- **`style`**: Formatting changes (white-space, missing semicolons, linter fixes) that do not affect code logic.
- **`refactor`**: Rewriting or restructuring code without adding a feature or fixing a bug.
- **`perf`**: Code changes that specifically improve execution speed or memory performance.
- **`test`**: Adding missing tests or correcting existing ones.
- **`chore`**: Maintenance tasks, build script updates, or dependency version bumps that do not touch production code.
- **`build`**: Changes that affect the build system or external dependencies.
- **`ci`**: Changes to continuous integration configuration files and scripts.

#### Handling Breaking Changes

If a commit introduces an incompatible API change or breaks backward compatibility, it is marked as a **breaking change**: 

- Append an exclamation mark (`!`) directly before the colon in the type/scope: `feat(api)!: remove v1 endpoints`.
- Or include `BREAKING CHANGE: <description>` in the footer section. 

#### Why Use Conventional Commits?

- **Automated Semantic Versioning (SemVer)**: Tools like **semantic-release** read the commit types to automatically decide whether to bump a *major* (`BREAKING CHANGE` or `!`), *minor* (`feat`), or *patch* (`fix`) version.
- **Automatic Changelogs**: Scripts can automatically aggregate and format commit messages into release notes.
- **Tooling Integration**: Enforced via Git hooks using utilities like **Commitlint** and **Husky** to block improperly formatted commits.

### **12. What is a Linter and why must it run early?**
**Answer:** 

A linter is **a static code analysis tool that scans source code without running it to find syntax errors, bugs, and style violations**. It must run early—during local development or pre-commit hooks—because fixing defects immediately costs less time, prevents bad code from entering the main branch, and stops small style debates during code reviews. 

#### What a Linter Does

- **Analyzes static code:** Inspects code syntax and logic structures without executing the program.
- **Enforces style:** Ensures uniform indentation, naming conventions, and file formatting across the team.
- **Flags probable bugs:** Catches common mistakes like unused variables, missing imports, or unreachable code. 

#### Why It Must Run Early

- **Cost of fixing issues:** Bugs found during local coding take seconds to fix, whereas bugs discovered in production require hotfixes, testing, and deployments. 
- **Prevents merge friction:** Running linters in pre-commit hooks or CI/CD pipelines ensures broken or messy code never reaches the shared repository. 
- **Cleans up code reviews:** Automating style checks stops human reviewers from wasting time arguing over brackets, spaces, and commas. 
- **Immediate feedback:** Developers learn best practices in real time as the editor or terminal highlights mistakes immediately after typing.

### **13. What is Secret Masking in CI/CD?**
**Answer:** 

Secret masking in CI/CD is a security feature that automatically hides sensitive values—such as API keys, passwords, and tokens—by replacing them with placeholder characters (like `***`) in build logs and console outputs. It acts as a safety net against accidental exposure, though it is only a presentation layer control rather than a full secret management solution. 

#### How Secret Masking Works

- **Pattern & String Matching:** CI/CD platforms (like GitHub Actions or GitLab CI/CD) scan stdout/stderr streams for registered secret strings and swap them out before writing to logs.
- **Variable Configuration:** Users register secrets in project or organization settings and flag them as "masked" so the runner knows which values to scrub. 
- **UI Redaction:** Masked variables are hidden or obfuscated not just in real-time terminal output, but also in archived web UI job histories. 

#### Limitations and Risks

- **Exact String Dependency:** Masking typically fails if the secret is transformed, encoded (e.g., Base64), truncated, or split before it hits the log. 
- **Artifact Exfiltration:** Masking protects console logs, but it does not stop a poorly written script from writing the secret to a file and saving it as a downloadable build artifact. 
- **Not a True Vault:** It does not encrypt secrets at runtime or restrict which processes or containers can read the credential once the job starts. 

#### Best Practices for CI/CD Security

- **Use Dedicated Secret Managers:** Store credentials in external tools like HashiCorp Vault, AWS Secrets Manager, or Azure Key Vault rather than plain-text CI variables. 
- **Adopt Short-Lived Credentials:** Prefer dynamic tokens with short Time-To-Live (TTL) or OpenID Connect (OIDC) over static, long-lived access keys. 
- **Disable Verbose Debugging:** Turn off trace or debug logging (`set -x` or equivalent flags) in pipeline steps that handle sensitive parameters. 

### **14. What is a Webhook in CI/CD?**
**Answer:** 

A webhook in CI/CD is **an automated HTTP POST request sent in real time from a code repository (like GitHub or GitLab) to a CI/CD server (like Jenkins or CircleCI) the moment a specific event occurs**—such as a code push, pull request, or merge.

#### How Webhooks Work in CI/CD

- **Event Trigger:** A developer performs an action in the source control system, such as committing code to the `main` branch.
- **Instant Push:** Instead of the CI/CD tool continuously asking the repo *"did anything change?"* (polling), the repository immediately pushes an HTTP POST payload containing event data to a designated URL endpoint. 
- **Pipeline Execution:** The CI/CD system receives the payload, verifies it, and kicks off automated tasks like building, running tests, or deploying code. 

#### Key Benefits

- **Real-Time Speed:** Builds and tests start instantly without delay or scheduled polling gaps.
- **Lower Server Load:** Eliminates constant request overhead on repository and CI/CD servers.
- **Event-Driven Automation:** Links disparate tools (GitHub, Slack, Jira, Jenkins) into a seamless automated pipeline.

### **15. What are Ephemeral / Preview Environments?**
**Answer:** 

Ephemeral or preview environments are **short-lived, fully isolated application deployments automatically spun up for a specific Git branch or pull request (PR) and destroyed once that code is merged or closed**. They replace shared staging servers with dedicated, conflict-free test spaces. 

#### How They Work

- **Triggered on Demand:** Opening or pushing to a pull request automatically kicks off a CI/CD pipeline to provision the stack.
- **Complete Isolation:** Each environment has its own network, database, services, and unique URL so changes never collide.
- **Automatic Teardown:** Merging or closing the pull request triggers clean-up, destroying the infrastructure and stopping cloud costs.

#### Key Benefits

- **No Staging Bottlenecks:** Multiple team members can test different features at the same time without stepping on each other.
- **Faster Feedback Loops:** QA engineers, product managers, and automated tests can review live web or mobile builds right away via a unique link.
- **Lower Infrastructure Costs:** Resources run for minutes or hours instead of sitting idle in permanent staging clusters.

### **16. What is a Blue-Green Deployment?**
**Answer:** 

Blue-green deployment is **a software release strategy that uses two identical production environments (called Blue and Green) to achieve zero-downtime updates**. While Blue runs the live version serving users, Green hosts the new version for testing. Once verified, a router or load balancer shifts all traffic to Green. 

#### How It Works

- **Dual Environments:** You maintain two production setups (Blue and Green) that are as similar as possible in hardware, software, and configuration.
- **Active vs. Standby:** At any time, only one environment is live and handles user requests.
- **Safe Testing:** Developers deploy and test the new software version on the idle environment without affecting real users.
- **Traffic Switch:** A load balancer, proxy, or router switches incoming traffic from the old environment to the new one in an instant.
- **Instant Rollback:** If a bug appears after the switch, you can point the router right back to the previous environment with no delay. 

#### Key Benefits

- **No Downtime:** Users experience no service interruption while updates go live.
- **Easy Recovery:** If a critical error happens, rolling back is as simple as flipping the traffic switch back to the original environment.
- **Real-World Testing:** You can test the exact new build in a true production environment before exposing it to everyone.

#### Common Tools

Many modern cloud platforms and DevOps systems support this model natively or through automation extensions:

- **Cloud Providers:** Use AWS CodeDeploy or Google Cloud Run for built-in traffic shifting.
- **Container Orchestration:** Manage seamless rollouts in Kubernetes using tools like Argo Rollouts or native service selectors.
- **CI/CD Pipelines:** Automate testing and deployment handoffs using platforms like **Octopus Deploy**

### **17. What is a Canary Deployment?**
**Answer:** 

A canary deployment is **a software release strategy that rolls out a new version of an application to a small subset of users or servers first**. It monitors real-world performance and errors before gradually shifting 100% of the traffic over, minimizing the impact of potential bugs. 

#### How It Works

- **Traffic Splitting:** A load balancer, service mesh, or gateway routes a small percentage of requests (like 1% to 5%) to the new "canary" version while the rest go to the stable version.
- **Real-Time Monitoring:** Systems track error rates, latency, and CPU usage.
- **Gradual Rollout:** If metrics look healthy, traffic steps up in phases (e.g., 25%, 50%, 75%, then 100%).
- **Automated Rollback:** If errors spike, traffic quickly switches back to the stable version before most users notice.

#### Pros and Cons

- **Pros:** Tests code in a live production environment with real users, costs less than running two full duplicate environments (like blue-green deployments), and limits the blast radius of failures. 
- **Cons:** Requires strict monitoring and instrumentation, and routing traffic across two active versions can complicate database schemas and API compatibility.

### **18. What is Dark Launching?**
**Answer:** 

Dark launching is **a software deployment strategy where new features or back-end systems are rolled out into a live production environment silently**. Users interact with the application without knowing the new code is running behind the scenes, allowing teams to test performance and gather data safely. 

#### How It Works

- **Feature Flags:** Developers use toggles to turn new code on or off remotely without redeploying the app. 
- **Traffic Forking:** Requests can be duplicated in the background so a new service processes real data while throwing away the output response. 
- **Silent Rollout:** A tiny fraction of real users (like 1% to 5%) see a new UI element while the rest of the user base remains on the legacy version. 

#### Main Benefits

- **Real-World Testing:** Teams test system scaling and heavy loads using actual production traffic rather than fake staging environments.
- **Risk Reduction:** If bugs or high error rates pop up, engineers can flip a switch to disable the feature instantly.
- **Decoupled Release:** The act of deploying code to servers is separated from the act of releasing it to the public. 

#### Dark Launch vs. Similar Strategies

- **Canary Release:** Exposes a new feature or infrastructure change to a small group of *visible* users to monitor live metrics or errors.
- **Dark Launch:** Focuses on hidden background processing or invisible front-end testing where users do not realize they are part of a test.

### **19. What is a Feature Flag (Toggle)?**
**Answer:** 

A **feature flag** (or **feature toggle**) is **a software development technique that lets teams turn application features on or off remotely at runtime without deploying new code**. By wrapping code inside conditional statements tied to an external configuration, developers can decouple code deployment from feature release. 

#### Common Use Cases

- **Canary Releases:** Gradually roll out a new feature to a small percentage of users before a full launch.
- **A/B Testing:** Direct different user cohorts to distinct feature variations to measure performance or engagement.
- **Kill Switches:** Instantly disable a broken or high-load feature in production without rolling back the entire application.
- **Trunk-Based Development:** Safely merge incomplete or work-in-progress code into the main branch while keeping it hidden from end-users. 

#### Main Types of Flags

- **Release Toggles:** Short-term switches used to roll out a new feature, removed after the release is complete.
- **Experiment Toggles:** Used for A/B testing and tracking user behavior across different versions.
- **Ops (Operational) Toggles:** Used to control system behavior, performance thresholds, or emergency kill switches.
- **Permissioning Toggles:** Permanent or semi-permanent switches that gate features based on user tiers or subscription plans.

#### Benefits and Risks

- **Benefits:** Speeds up deployment cycles, lowers risk via instant rollbacks, and enables granular targeting by region, device, or user ID.
- **Risks:** Can introduce code complexity, technical debt, and "toggle clutter" if old, expired flags are not regularly cleaned up from the codebase.

### **20. What is Trunk-Based Development?**
**Answer:** # Trunk-Based Development (TBD)

**Trunk-based development (TBD)** is **a version control management practice where developers merge small, frequent code updates directly into a single central branch—known as the "trunk" or "main" branch—rather than working in isolated, long-lived feature branches**. 

It serves as a core foundation for Continuous Integration (CI) and Continuous Delivery (CD). 

#### Core Principles

- **Single Source of Truth:** All developers integrate their code into one main branch (frequently called `main`, `master`, or `trunk`).
- **Small, Frequent Commits:** Developers push small, incremental changes at least once a day (or multiple times daily).
- **Short-Lived Branches:** If branches are used at all, they exist only for a few hours or a couple of days maximum before being merged back. 
- **Feature Flags (Toggles):** Incomplete or work-in-progress features are merged into the trunk early, but hidden from end-users using **feature flags** until they are fully ready.

#### Key Benefits

- **Eliminates "Merge Hell":** Because code is integrated constantly, massive, painful merge conflicts that happen when long-lived branches diverge are largely avoided.
- **Always-Deployable Code:** The main branch stays in a clean, tested, and releasable state at all times. 
- **Faster Feedback Loops:** Automated testing pipelines run on every small commit, catching bugs and integration issues immediately.
- **Simpler Code Reviews:** Reviewing small, focused pull requests takes less time and yields higher bug-detection accuracy than reviewing thousands of lines of branched code. 

#### Trunk-Based Development vs. Alternatives

- **Gitflow:** Uses multiple long-lived primary branches (like `develop` and `release`) alongside separate, long-lived feature branches. It suits scheduled release cycles, whereas TBD suits rapid, continuous deployments. 
- **Strict TBD vs. Short-Lived Branches:** Smaller teams may commit directly to the trunk, while larger engineering organizations use short-lived pull-request branches merged daily via automated merge queues. As noted by a **consensus on Reddit**, many teams adopt short-lived feature branches paired with main-branch releases rather than raw direct-to-trunk commits.

### **21. What is GitFlow?**
**Answer:** 

Gitflow is **a legacy Git branching model introduced by Vincent Driessen in 2010 that organizes code around strict rules and multiple permanent and temporary branches**. It was designed to manage complex software release schedules and support multiple concurrent versions of a product, though it is largely superseded today by simpler strategies like trunk-based development or GitHub Flow. 

You can learn more about its core rules through the **Gitflow Workflow Guide** by Atlassian.

#### Core Branch Types in Gitflow

Gitflow divides your repository into two permanent main branches and several temporary auxiliary branches: 

- **`main` (or `master`)**: Permanent branch that holds the official, production-ready release history, with each commit typically tagged with a version number. 
- **`develop`**: Permanent integration branch where code for upcoming features converges and sits as a pre-production staging area. 
- **Feature branches**: Temporary branches created off `develop` to build new features; once finished, they merge back into `develop`. 
- **Release branches**: Temporary branches forked from `develop` when preparing a new production release, allowing final bug fixes and documentation updates before merging into both `main` and `develop`. 
- **Hotfix branches**: Temporary branches forked directly from `main` to address urgent production bugs, merging back into both `main` and `develop`.

#### When to Use vs. Alternatives

- **Best used for**: Projects with scheduled release cycles, software supporting multiple live public versions simultaneously (like mobile apps), or regulated environments needing strict audit trails. 
- **Drawbacks**: Adds branch-management overhead and can slow down continuous delivery due to merge conflicts from long-lived branches. 
- **Modern alternatives**: GitHub Flow (a single `main` branch with short-lived feature branches) or Trunk-Based Development for rapid, continuous deployment. 

### **22. What is Static Application Security Testing (SAST)?**
**Answer:** 

**Static Application Security Testing (SAST)** is **a white-box testing methodology that analyzes uncompiled source code, bytecode, or binaries to find security vulnerabilities without running the application**.

#### How SAST Works

- **White-box analysis:** Evaluates the application from the "inside out" by inspecting internal logic, architecture, and syntax. 
- **No execution required:** Scans code files at rest rather than launching a live system. 
- **Core analytical techniques:**
  - **Data flow analysis:** Tracks how user-controlled or untrusted data moves across the system.
  - **Control flow analysis:** Examines the sequence of operations, loops, and conditional logic.
  - **Syntactic pattern matching:** Compares code against known security signatures, rules, and bad practices. 

#### Common Vulnerabilities Detected

SAST tools are built to flag flaws outlined in standards like the OWASP Top 10 and SANS Top 25, including: 

- **SQL and Command Injections**
- **Cross-Site Scripting (XSS)**
- **Buffer Overflows**
- **XML External Entity (XXE) attacks**
- **Hardcoded secrets, API keys, and weak cryptographic practices** 

#### Advantages of SAST

- **Early detection ("Shift-Left"):** Finds vulnerabilities during the coding or build phase before software reaches production, making fixes far cheaper and faster.
- **High code coverage:** Can rapidly scan millions of lines of source code across 100% of the codebase. 
- **Exact pin-pointing:** Identifies the specific file, function, and line number where a security flaw lives. 
- **Pipeline integration:** Plugs directly into Integrated Development Environments (IDEs), version control (e.g., **GitHub Code Scanning**, and CI/CD pipelines.)

#### Limitations of SAST

- **False positives:** Can frequently flag safe code as a vulnerability, requiring manual developer triage.
- **No runtime visibility:** Cannot detect issues that only manifest when the application is running, such as environmental misconfigurations, weak network permissions, or broken live authentication logic.
- **Language and framework dependency:** Tools must specifically support the programming language and libraries being analyzed.

### **23. What is Dynamic Application Security Testing (DAST)?**
**Answer:** 

**Dynamic Application Security Testing (DAST)** is **a cybersecurity method that evaluates a running web application or API from the outside-in by simulating external hacker attacks**. 

#### How DAST Works

- **Black-Box Testing:** DAST tools test applications without access to the internal source code, architecture, or design documents. 
- **Automated Payload Injection:** Scanners send malicious inputs—such as SQL injection (SQLi), Cross-Site Scripting (XSS), and authentication bypass payloads—to the live application. 
- **Behavior Analysis:** The tool observes the application’s HTTP responses, error codes, and functional behavior to determine if an attack successfully exposed a vulnerability.

#### Common Vulnerabilities Found by DAST

- Input validation flaws (SQL injection, XSS, command injection)
- Authentication and session management issues
- Server, API, and database configuration mistakes
- Insecure redirects and improper error handling 

#### Key Pros and Cons

- **Pros:**
  - Language-agnostic and independent of the underlying framework.
  - Finds runtime and configuration-based flaws that static code analyzers miss.
  - Mimics a real-world attacker with low false-positive rates for exploitable paths. 
- **Cons:**
  - Cannot pinpoint the exact line of code responsible for a flaw.
  - Requires a working runtime or staging environment to test properly.
  - Cannot cover internal business logic or deep backend code paths as thoroughly as internal scans. 

#### Popular DAST Tools

- **Open Source:** OWASP ZAP (Zed Attack Proxy)
- **Commercial:** Burp Suite, Acunetix, Netsparker, IBM AppScan, and Veracode 

### **24. What is Software Composition Analysis (SCA)?**
**Answer:** 

**Software Composition Analysis (SCA)** is **an automated security and management process that identifies open-source software, third-party libraries, and dependencies embedded within an application's codebase**. 

Modern software relies heavily on external code, where up to 80% to 90% of an application's building blocks come from open-source repositories. SCA tools help development and security teams track and protect this supply chain.

#### Core Functions of SCA

- **Vulnerability Scanning:** Scans code, package manifests, and binaries against databases like the National Vulnerability Database (NVD) to flag known Common Vulnerabilities and Exposures (CVEs). 
- **License Compliance:** Identifies open-source licenses (such as GPL, MIT, or Apache) to ensure legal compliance and prevent intellectual property risks. 
- **Transitive Dependency Tracking:** Maps out "dependencies of dependencies" that are buried deep within an application's architecture.
- **Software Bill of Materials (SBOM) Generation:** Produces a comprehensive, machine-readable inventory of every component, library, and module present in the software. 

#### How SCA Works

1. **Discovery:** The tool scans project configuration files (like `package.json`, `pom.xml`, or `requirements.txt`), container images, or compiled binaries.
2. **Comparison:** It matches the discovered components and version numbers against public and proprietary vulnerability databases.
3. **Prioritization:** It scores and ranks risks using metrics like the Common Vulnerability Scoring System (CVSS).
4. **Remediation:** It suggests patch versions, automated pull requests, or policy blocks to fix the issue.

#### SCA vs. Other Security Testing

- **SCA vs. SAST (Static Application Security Testing):** SCA focuses strictly on open-source and third-party libraries. SAST analyzes your custom, proprietary source code for logic bugs and coding flaws. 
- **SCA vs. DAST (Dynamic Application Security Testing):** SCA evaluates code and manifests statically without running the app. DAST attacks a live, running application from the outside to find runtime weaknesses.

### **25. What is Mutation Testing in CI?**
**Answer:** 

Mutation testing in Continuous Integration (CI) is **an automated process that evaluates how well your unit tests catch bugs**. It injects small synthetic errors (mutants) into your source code, runs your test suite against them, and fails the build if too many mutants survive.

#### How It Works in a CI Pipeline

- **Mutant Generation:** Tools like **Stryker Mutator** or PIT modify operators, numbers, or true/false values in your code.
- **Test Execution:** The CI runner executes your test suite against each mutated file.
- **Killed vs. Survived:** If a test fails, the mutant is "killed" (good). If all tests pass, the mutant "survived," exposing a weak assertion or missing test case.
- **Score Calculation:** The pipeline generates a **mutation score** (killed mutants divided by total mutants). 

#### CI Integration Strategies

- **Diff-Mode / Scoped Runs:** Run mutation tests only on changed files or pull requests to save time.
- **Parallel Execution:** Distribute mutant runs across multiple worker nodes in CI (e.g., GitHub Actions or **CircleCI**)
- **Threshold Enforcement:** Set a minimum passing mutation score (e.g., 80%) that blocks merging if dropped.
- **Scheduled Full Runs:** Run deep, full-codebase mutation tests on a nightly or weekly schedule rather than every commit, due to high CPU and time costs.

### **26. What is Concurrency Control in CI/CD?**
**Answer:** 

**Concurrency control in CI/CD** is **the mechanism used to regulate, queue, or cancel simultaneously executing workflows, jobs, or deployments**. Because automated pipelines are frequently triggered by multiple developers pushing code at the same time, concurrency control ensures that these parallel execution streams do not cause race conditions, corrupt shared deployment environments, or exhaust limited build infrastructure. 

#### Core Objectives of Concurrency Control in CI/CD

- **Environment Protection:** It prevents multiple deployments from pushing to the same staging or production environment at the exact same moment, which would cause an unpredictable and broken infrastructure state. 
- **Sequential Execution:** It ensures that updates are deployed in the precise order they were committed, preventing older code versions from accidentally overwriting newer code. 
- **Resource Management:** It limits the number of parallel automated runners or containers operating simultaneously, preventing teams from exhausting their cloud infrastructure limits or monthly CI/CD build minutes. 

#### Common Strategies & Implementation Mechanics

Modern platforms—such as **GitHub Actions** and Bitbucket Deployments—handle concurrency using a few primary control models: 

| **Strategy** | **Behavior** | **Best Used For** |
|---|---|---|
| **Queuing (FIFO)** | Holds a newly triggered build in a "pending" status until the currently running build finishes completely. | **Critical release jobs** where every single code version must step through validation sequentially. |
| **Auto-Cancellation (Latest Wins)** | If Build A is running and a developer pushes Build B, the pipeline **cancels Build A mid-execution** and switches resources entirely to Build B. | **Pull Request validations** and feature branch test runs where intermediate commits no longer matter. |
| **Pessimistic Locking** | Strictly allows only one job per scoped environment at any time; explicitly rejects conflicting deployments with an error. | **Production deployments** or complex data migration workflows where collisions cause severe failures. |

#### Implementation Example (GitHub Actions)

In practice, engineers configure this directly within pipeline configuration files. In GitHub Actions, the `concurrency` block organizes how jobs are grouped and throttled: 

**YAML**

```yaml
name: Production Deployment

on:
  push:
    branches:
      - main

# Enforce concurrency control at the workflow level
concurrency:
  group: production-deploy-${{ github.ref }}
  cancel-in-progress: true  # Automatically drops previous runs to save build minutes
```

### **27. What is a Merge Queue in GitHub?**
**Answer:** 

A **GitHub merge queue** is **an automated workflow tool that stabilizes busy repositories by ensuring pull requests (PRs) pass all required checks when integrated with the latest code before they are formally merged into the target branch**. It acts as a branch traffic controller, sitting directly between the "PR approved" and "PR merged" stages.

#### The Core Problem It Solves

On high-performance, fast-moving teams, multiple engineers often commit to a single branch simultaneously. This introduces a "validation gap": 

1. Developer A and Developer B both branch off the `main` branch.
2. They both run tests on their independent PRs, and both pass.
3. Developer A merges their PR into `main`.
4. Developer B merges their PR immediately after. Even though Developer B’s PR passed tests earlier, it was never tested against Developer A's new changes. 

When combined, their changes might conflict or break the build, leading to a broken `main` branch.

#### How GitHub Merge Queue Works

Instead of allowing developers to merge approved PRs directly into a busy branch, GitHub serializes the process: 

- **Sequential Lineup:** Approved PRs are placed into a First-In, First-Out (FIFO) queue. 
- **Dynamic Testing Environment:** GitHub creates a temporary branch (using a `gh-readonly-queue/` prefix) that combines the latest `main` branch code, the PRs ahead of it in the queue, and the current PR. 
- **Pre-Merge Validation:** Continuous Integration (CI) tests run against this combined state.
- **Automatic Eviction:** If a PR's tests fail or code conflicts occur, GitHub automatically removes that PR from the queue. It then re-tests the subsequent PRs against the corrected branch state, protecting `main` from ever breaking.
- **Final Merge:** If all status checks pass, the PR is automatically merged into the target branch without manual intervention. 

#### Requirements & Availability

According to **GitHub Docs**, the feature requires specific configuration:
- **Account Tiers:** It is available for public repositories owned by organizations, or private repositories using GitHub Enterprise Cloud. 
- **Setup:** A repository administrator must enable the "Require merge queue" option inside the repository's **Branch protection settings**. 
- **CI Integration:** Your GitHub Actions or CI pipeline must be configured to trigger on the `merge_group` event to run checks while the PR is in the queue.

### **28. What is Self-Hosted Runner vs Cloud-Hosted Runner?**
**Answer:** 

**A cloud-hosted runner (managed by the platform provider like GitHub) is a pre-configured virtual machine that requires zero setup and is maintained automatically. A self-hosted runner is a machine or VM that you provision, run, and maintain on your own infrastructure to gain full environment control and network access**. 

#### Key Differences

| **Feature** | **Cloud-Hosted Runner** | **Self-Hosted Runner** |
|---|---|---|
| **Infrastructure** | Managed and hosted by the provider (e.g., GitHub/Azure) | Managed by you (on-premises, AWS, GCP, Azure) |
| **Setup & Maintenance** | Instant setup; automatic updates, security patches, and clean state per job | Requires manual installation, OS hardening, patching, and lifecycle management |
| **Hardware & OS** | Fixed standard choices (e.g., Ubuntu, Windows, macOS with default CPU/RAM) | Full control over custom hardware, GPUs, ARM chips, and specialized OS images |
| **Network & Access** | Isolated public cloud; cannot directly access internal corporate networks or private databases | Can connect directly to local private networks, internal APIs, and on-premise hardware |
| **Cost** | Billed per minute (or free for public repositories) | You pay for underlying compute/hardware plus engineering maintenance overhead |

#### When to Choose Cloud-Hosted Runners

- You want a fast, maintenance-free CI/CD setup.
- Your project uses standard build environments (Linux, Windows, macOS).
- You are building public/open-source code where standard runners are free.
- You prefer ephemeral, fresh virtual machines for every single job execution.

#### When to Choose Self-Hosted Runners

- You need access to private corporate networks, local servers, or restricted databases.
- You require custom hardware like heavy GPUs, specialized architecture, or pre-installed software tools.
- You have strict data residency, compliance, or security regulations that forbid public cloud execution.
- You want to persist state, caches, or files across multiple sequential workflow runs.

### **29. What is Actions Runner Controller (ARC)?**
**Answer:** 

**Actions Runner Controller (ARC)** is **an open-source Kubernetes operator that automatically orchestrates and scales self-hosted runners for GitHub Actions workflows**. Instead of manually maintaining fixed virtual machines or servers to run your CI/CD pipelines, ARC allows teams to dynamically spin up and tear down runner instances as Kubernetes pods inside their own cloud or on-premises infrastructure. 

#### Key Capabilities of ARC

- **Dynamic Autoscaling:** ARC creates **runner scale sets** that monitor your GitHub repository, organization, or enterprise workflow queues. It automatically scales the number of runner pods up during peak demand and scales them down to zero when idle. 
- **Ephemeral Runner Pods:** Runners can be configured as **short-lived (ephemeral) containers**. Each job runs in a clean, isolated environment, and the pod is immediately deleted upon completion. This enforces high security and prevents artifact pollution between builds. 
- **Cost Efficiency:** By hosting runners on your own Kubernetes cluster (such as AWS EKS, Google GKE, or Azure AKS), you bypass the build-minute limits and costs associated with GitHub-hosted runners. 
- **Enterprise Control:** ARC allows you to pass custom Docker images, volume mounts, and network policies to your runners. This is ideal for organizations with strict data residency, security compliance, or internal network dependency requirements. 

#### How It Fits into the GitHub Ecosystem

Originally started as a community-maintained project, ARC was officially **adopted by GitHub** as the recommended reference implementation for scaling self-hosted runners on Kubernetes. GitHub actively develops the modern version, known as **Actions Runner Controller for Autoscaling Runner Sets**, which communicates directly with GitHub's scale set APIs rather than relying on legacy webhook or polling mechanisms. 

Because running ARC requires advanced knowledge of container orchestration, network routing, and Kubernetes API pressure tuning, it is typically managed by dedicated platform or DevOps teams.

### **30. What is OIDC (OpenID Connect) in CI/CD?**
**Answer:** 

OpenID Connect (OIDC) in CI/CD is **a keyless authentication pattern**. It lets pipelines prove their identity to cloud providers (like AWS, Azure, or GCP) using short-lived, cryptographically signed tokens instead of storing long-lived static secrets or passwords. 

#### How OIDC Works in CI/CD

- **Job Request:** The CI/CD tool (such as **GitHub Actions** or GitLab) generates a JSON Web Token (JWT) for the running job.
- **Token Exchange:** The job presents this token to the cloud provider's Security Token Service (STS).
- **Claim Validation:** The cloud provider checks specific claims inside the token (like the repository name, branch, or environment).
- **Temporary Access:** If valid, the cloud provider issues temporary, short-lived credentials for that specific job session.

#### Key Benefits

- **No Stored Secrets:** You do not need to generate, rotate, or manually store cloud API keys inside your CI/CD settings.
- **Reduced Blast Radius:** Tokens expire automatically after a short time, limiting the risk if a token ever leaks.
- **Fine-Grained Trust:** Access can be locked down to specific branches, tags, or pull requests using claim conditions.

### **31. What is an SBOM (Software Bill of Materials)?**
**Answer:** 
An **SBOM (Software Bill of Materials)** is **a formal, machine-readable inventory list that details every component, open-source library, third-party module, and dependency used to build a software application**. Think of it as a detailed ingredient list for food packaging or a parts catalog for a manufactured vehicle.

#### What Information Does an SBOM Contain?

An SBOM tracks critical metadata for each element inside a software build: 

- **Supplier name:** Who created or published the component.
- **Component and product names:** The specific software modules or packages.
- **Version numbers:** Exact release versions being used.
- **Unique identifiers:** Universal IDs (such as package URLs or SWID tags).
- **Dependency relationships:** A tree showing how components rely on one another (including nested or transitive dependencies).
- **Cryptographic hashes:** Digital fingerprints to verify file integrity.
- **License information:** Open-source or commercial licenses governing each part.

#### Common SBOM Standards and Formats

SBOMs are typically generated, shared, and stored using standardized, machine-readable file formats: 

- **SPDX (Software Package Data Exchange):** Developed by the Linux Foundation, widely used for auditing and compliance.
- **CycloneDX:** Created by OWASP, built specifically for application security and DevSecOps workflows.
- **SWID Tags:** Software Identification tags standardized by ISO/IEC for asset tracking.
- **File Encodings:** Usually formatted in JSON, XML, or YAML.

#### Why Are SBOMs Important?

- **Vulnerability Management:** When a major zero-day flaw (like the Log4j incident) hits an open-source library, an SBOM helps security teams instantly check if and where that vulnerable component exists across their digital estate. 
- **Supply Chain Security:** It provides visibility into hidden or transitive dependencies that developers did not directly write or choose, protecting against supply chain attacks. 
- **Regulatory and Legal Compliance:** Government mandates (such as U.S. Executive Order 14028) and international standards (like the EU Cyber Resilience Act) increasingly require SBOMs for software sold to public or enterprise sectors.
- **License Auditing:** It prevents legal and intellectual property violations by making open-source license obligations transparent.

### **32. What is the SLSA Framework?**
**Answer:** 

The **SLSA Framework** (pronounced "salsa") stands for **Supply-chain Levels for Software Artifacts**. It is an industry-consensus, open-source security framework maintained by the Open Source Security Foundation (OpenSSF) to improve software supply chain integrity, prevent tampering, and establish trust in software builds.

#### Core Goals

- **Prevent Tampering:** Protect software from unauthorized modifications or injection attacks from source code all the way to deployment.
- **Ensure Provenance:** Generate and verify tamper-resistant metadata that shows *how*, *where*, and *by whom* a software artifact was built.
- **Provide a Common Language:** Give software producers, consumers, and security teams a standardized checklist and metric to evaluate supply chain safety.

#### SLSA Build Levels

The framework uses progressive levels of security assurance to help organizations adopt practices incrementally: 

- **Level 0 (No Protection / Ad Hoc):** No guaranteed security controls or automated provenance; builds typically happen on local or unmanaged machines. 
- **Level 1 (Basic Provenance):** The build platform automatically generates provenance metadata (such as source repository, build steps, and cryptographic hashes) to establish basic traceability. 
- **Level 2 (Hosted Build Platform & Signed Provenance):** Builds run on a hosted or dedicated CI/CD platform, and provenance is cryptographically signed to make post-build tampering detectable. 
- **Level 3 (Hardened Builds):** Requires isolated, non-falsifiable, and tamper-resistant build platforms that protect against privilege escalation, credential theft, and pipeline compromise. 

#### Who It Is For

- **Software Producers:** To harden build pipelines, prove artifact integrity, and build customer trust.
- **Software Consumers:** To cryptographically verify the origin and safety of third-party packages and dependencies before importing them.
- **Infrastructure & Platform Providers:** To build native security controls (like automated signing and isolation) directly into CI/CD environments.

### **33. What is Hermetic Build?**
**Answer:**

A **hermetic build** is **a software build process that runs in complete isolation from the host system and external network, using only explicitly declared, fixed inputs to produce identical, repeatable outputs**. 

#### Key Characteristics

- **Network Isolation:** The build cannot reach out to the internet or fetch unpinned resources during execution. 
- **Host Independence:** It ignores libraries, compilers, and utilities installed globally on the local machine, treating tools themselves as managed dependencies. 
- **Explicit Inputs:** Every source file, compiler, library, and tool version is explicitly declared and pinned—often using cryptographic hashes (like SHA). 
- **Determinism and Reproducibility:** Running the exact same commit or input set produces the exact same binary artifact regardless of where or when it is built. 

#### Why Hermetic Builds Matter

- **Security:** Blocks supply chain attacks, dependency confusion, and hidden code injections from stray network calls or hijacked host packages.
- **Reliability:** Eliminates "works on my machine" bugs caused by different local environment setups or floating dependency versions.
- **Caching & Speed:** Allows safe, reliable caching of intermediate build steps and distributed parallel builds.

#### Common Tools Supporting Hermetic Builds

- **Bazel:** Google's open-source build system designed around strict hermeticity and sandboxing.
- **Nix:** A functional package manager that builds packages in isolated environments.
- **Docker / Containers:** Frequently used to wrap build steps in an empty or fixed root filesystem.

### **34. What is Docker BuildKit layer caching?**
**Answer:** 

**Docker BuildKit layer caching** is **an advanced caching system built into Docker's next-generation build engine, BuildKit, that dramatically speeds up container image builds by reusing unchanged image layers from previous builds**.

Instead of rebuilding every single line in a `Dockerfile` from scratch, BuildKit treats the instructions as a concurrent dependency graph. If a step (and the steps before it) hasn't changed, BuildKit simply skips executing that command and pulls the pre-built layer directly from its cache.

#### How Layer Caching Works

Each instruction in a `Dockerfile` (like `RUN`, `COPY`, or `ADD`) creates a distinct image layer. 

1. **The Chain Effect:** BuildKit checks the cache sequentially from top to bottom. 
2. **Cache Invalidation:** If you modify a file that is pulled in by a `COPY` instruction, or change the text of a `RUN` command, BuildKit **invalidates the cache** for that specific layer. 
3. **Downstream Rebuilds:** Once a single layer is invalidated, **all subsequent layers** below it must also be rebuilt from scratch, even if their own code or commands didn't change.

Because of this behavior, it is a best practice to order your `Dockerfile` instructions by placing expensive, rarely-changed steps (like installing system dependencies) at the top, and frequently changed files (like your application source code) at the very bottom. 

#### Key Capabilities of BuildKit Caching

BuildKit significantly improves upon the legacy Docker builder with several advanced features:

- **Remote/External Caching:** Traditional Docker caching only works if the previous build occurred on the exact same machine. BuildKit allows you to export your build cache to remote registries (like Docker Hub or Amazon ECR) or CI providers (like GitHub Actions) using the `--cache-to` and `--cache-from` flags. This means ephemeral CI/CD pipelines can download existing cache layers on the fly rather than starting fresh every time. 
- **Concurrent Execution:** BuildKit can evaluate independent stages of a multi-stage build simultaneously, optimizing which layers are used or discarded in parallel.

#### Layer Cache vs. Cache Mounts

While related, BuildKit provides two completely different mechanisms to optimize speed, and they are often used together: 

| **Feature** | **Docker Layer Cache** | **BuildKit Cache Mount (`--mount=type=cache`)** |
|---|---|---|
| **What is reused** | The final, compiled filesystem result of an entire `Dockerfile` instruction. | Specific files or temporary directories inside a step (e.g., package manager downloads). |
| **Behavior on change** | If an input changes, the **entire step must re-run** and downstream steps break. | The step will re-run, but the command can **reuse the persisted directory** to skip downloading existing files. |
| **Primary Use Case** | Skipping entire unchanged processes. | Speeding up compiler caches (`GOCACHE`) or package restorations (`npm install`, `pip`, NuGet). |
| **Final Image Size** | Cached layers directly build the structure of the final image. | Mount contents are **never committed** to the final image, keeping it clean. |

### **35. What is SonarQube Quality Gate?**
**Answer:**

A **SonarQube Quality Gate** is **a configurable checkpoint that enforces a strict quality policy**, answering one fundamental question before code advances: **"Is my project ready for release?"** It consists of a set of Boolean conditions based on code metrics. If the code meets these thresholds, the project **Passes**; if even a single condition is missed, it **Fails**. 

Integrating Quality Gates into a CI/CD pipeline acts as a circuit breaker, allowing teams to automatically block pull requests or fail build deployments when non-compliant code is detected.

#### Core Components & Key Metrics

Quality gate conditions are evaluated against specific metrics calculated during a scan. These metrics can target either **overall code** or focus purely on **new code**: 

- **Reliability & Security:** Ratings (from A to E) based on the severity of detected bugs or vulnerabilities. 
- **Maintainability:** Ratings determined by technical debt ratios and code smells. 
- **Test Coverage:** Ensuring that a sufficient percentage of code lines are backed by automated tests. 
- **Code Duplication:** Limiting the density of copied or repeated code blocks. 
- **Security Hotspots:** Tracking the status of reviewed security-sensitive pieces of code. 

#### Built-in vs. Custom Quality Gates

Each project is assigned exactly one quality gate at a time. SonarQube organizes these into two main types: 

1. **Built-in Gates (Read-Only):**
   - **"Sonar way"**: The platform default. It champions the **"Clean As You Code"** philosophy, enforcing high standards on *new code* (e.g., ≥ 80% coverage, ≤ 3% duplication, and "A" ratings for bugs/vulnerabilities) so developers focus on keeping incremental changes clean rather than drowning in legacy tech debt.
   - **"Sonar way for AI Code"**: Tailored with specialized validation protocols for projects leveraging generative or agentic AI.
2. **Custom Quality Gates:**

   With appropriate administrator privileges, organizations can create custom configurations. This is helpful because different technologies have different demands; for example, an internal core Java library might require stricter code coverage criteria than a rapid front-end web project. 

#### Platform Feedback & Automation

- **Pull Request Decoration:** The quality status links back to repository hosting services (like GitHub or GitLab), visually flagging code health before a merge.
- **CI/CD Pipeline Failure:** Pipelines (like Jenkins or GitHub Actions) can intercept the quality gate output via Web APIs or plugins to halt broken code from hitting production.
- **The "Fudge Factor":** To prevent minor edits from disproportionately tanking a status, SonarQube includes a mechanism that ignores coverage and duplication requirements on small adjustments until a change contains at least 20 new lines.

### **36. What is Code Coverage?**
**Answer:** 

**Code coverage** is **a software testing metric that measures the percentage of source code executed when a test suite runs**. It is a white-box testing technique used to locate untested sections of a program and guide the creation of new tests. 

#### How It Is Calculated

Code coverage is calculated by dividing the number of code elements executed during testing by the total number of code elements, multiplied by 100: 

```math
Code Coverage = \left(\frac{\text{Executed Code Elements}}{\text{Total Code Elements}}\right) \times 100
```



#### Common Types of Metrics

Coverage can be tracked across several structural dimensions:

- **Statement Coverage:** The percentage of individual programming statements executed.
- **Line Coverage:** The percentage of physical source code lines run by the tests.
- **Branch Coverage:** The percentage of control flow branches (such as `if/else` paths or loops) tested for both true and false outcomes.
- **Function/Method Coverage:** The percentage of defined functions or methods called during the test run.
- **Condition Coverage:** The percentage of individual boolean sub-expressions evaluated as both true and false. 

#### Common Tools by Language

Developers use specialized tools and profilers to instrument code and generate coverage reports: 

- **Java:** JaCoCo, Cobertura, Clover
- **JavaScript / TypeScript:** Istanbul / NYC
- **Python:** Coverage.py
- **PHP:** PHPUnit
- **Ruby:** SimpleCov
- **C / C++:** Parasoft C/C++test, Coco 

#### Code Coverage vs. Test Coverage

Though often used interchangeably, they measure different things: 

- **Code Coverage:** Measures structural execution of the code (statements, branches, lines).
- **Test Coverage:** Measures functional behavior against business requirements, user scenarios, and product specifications. 

#### Limitations and Best Practices

- **High coverage does not guarantee bug-free software:** Code can execute without validating that the output or logic is actually correct. 
- **Diminishing returns:** Aiming for 100% coverage across a massive codebase can be costly and lead to brittle, low-value tests; a target of around **80%** is standard for general applications, while safety-critical industries (like avionics or automotive) may require strict 100% compliance. 
- **Consensus on forum discussions:** As noted by developers on **Stack Overflow**, relying solely on line counts is a trap; true safety requires evaluating condition, branch, and parameter edge cases rather than chasing a raw percentage.

### **37. What is Pipeline as Code?**
**Answer:** 

Pipeline as code is **the practice of defining continuous integration and continuous delivery (CI/CD) workflows—such as builds, tests, and deployments—through human-readable configuration files stored in a source control repository rather than manual point-and-click graphical user interfaces (GUIs)**. 

#### How It Works

- **Version Control:** Pipeline definitions (like a `Jenkinsfile`, `.gitlab-ci.yml`, or `.github/workflows/*.yml`) live side-by-side with your application code in Git.
- **Code Reviews:** Changes to the pipeline go through pull requests and code reviews just like regular software features.
- **Automation:** Triggered automatically by events like code pushes or pull requests to build, test, and deploy software across environments. 

#### Key Benefits

- **Audit Trails and History:** Every modification is tracked with a clear commit history showing who changed the pipeline and why.
- **Reproducibility and Rollbacks:** If a pipeline update breaks the build, you can easily roll back to a previous version of the pipeline code.
- **Branch-Specific Pipelines:** Teams can test experimental pipeline logic on separate branches before merging to production.
- **No GUI Lock-In:** Eliminates fragile, hand-clicked server configurations that are hard to replicate or back up.

#### Common Tools

- **Jenkins** Uses a Groovy-based DSL or Declarative syntax via a `Jenkinsfile`.
- **GitLab CI/CD** Uses `.gitlab-ci.yml` files stored inside the project repository.
- **GitHub Actions:** Uses YAML workflow files located in `.github/workflows/`.
- **Tekton / Pipelines as Code:** A Kubernetes-native framework for cloud-native pipelines.

### **38. What is a Monorepo CI Strategy?**
**Answer:**

A **Monorepo CI Strategy** is **a set of engineering practices and tooling used to run Continuous Integration (CI) efficiently when multiple projects, services, and shared libraries live inside a single version-controlled repository**.

#### Why It Matters

In a standard multi-repository setup, CI runs isolated tests per project. In a monorepo, a naive CI approach runs the entire test suite and rebuilds every single project on every commit. This causes:

- **Exploding CI costs:** Wasted compute time and money from rebuilding code that did not change.
- **Long developer wait times:** A single small fix can get blocked behind hours of unrelated tests. 

#### Core Pillars of a Monorepo CI Strategy

- **Affected-Projects Detection:** Using build graph analysis (via tools like **Nx**, Turborepo, or **Bazel** to inspect git diffs, trace dependency trees, and run tests **only** on the specific projects impacted by a change.
- **Aggressive Caching:** Storing and reusing previously built artifacts, test results, and compiler outputs so unchanged modules skip execution entirely. 
- **Parallel Execution & Matrix Jobs:** Splitting independent builds and tests across concurrent runner agents to maximize throughput.
- **Scope-Aware Merge Queues:** Using merge queue systems (such as **Mergify**) to run independent parallel merge lanes per project scope, preventing a frontend PR from waiting behind an unrelated backend migration. 
- **Path-Based Triggers:** Configuring CI platform rules (like path filters in **GitHub Actions**) to skip or trigger workflows based on the directory or file paths modified in a pull request.

### **39. What is a Pull-Through Cache?**
**Answer:** 

A **pull-through cache** is **an on-demand proxy mirror that automatically fetches, saves, and serves external resources—most commonly container images—from an upstream public or private registry**. 

When you request an image, the system checks the local cache first:

- **First request (Cache Miss):** The cache fetches the image directly from the upstream registry (such as Docker Hub, Quay, or registry.k8s.io), saves a local copy, and delivers it to you.
- **Subsequent requests (Cache Hit):** The image is served directly from your local mirror without reaching out to the external source. 

#### Key Benefits

- **Faster Pull Speeds & Lower Latency:** Images are stored close to your local or cloud workloads (such as AWS EC2, EKS, or Lambda), speeding up deployment and CI/CD build times. 
- **Mitigation of Rate Limits:** By caching frequently used external images locally, organizations avoid hitting strict rate limits imposed by public registries like Docker Hub.
- **High Availability & Reliability:** If an upstream public registry goes down or experiences network throttling, your builds can still pull previously cached images.
- **Enhanced Security & Compliance:** Local mirrors allow teams to scan cached images for vulnerabilities, enforce governance, and maintain control over software dependencies. 

#### Common Implementations

- **Amazon ECR Pull-Through Cache:** Automatically mirrors public or private registries (including Amazon ECR Public, Kubernetes, Docker Hub, and Quay) inside your private AWS ECR registry. 
- **Self-Hosted Registries:** Tools like **Docker Registry**, **Harbor**, and **Zot** can be configured as local proxy caches via daemon or runtime settings (e.g., updating `daemon.json` in Docker).

### **40. What is Argo Rollouts?**
**Answer:** 

**Argo Rollouts** is **an open-source Kubernetes controller and Custom Resource Definition (CRD) set that adds advanced progressive delivery and deployment capabilities to Kubernetes**. 

It acts as a drop-in replacement for native Kubernetes `Deployment` objects to provide fine-grained control over how software updates reach production. 

#### Core Deployment Strategies

- **Blue-Green Deployments:** Uses two separate environments or services (an active/blue and a preview/green version) to switch traffic all at once or after validation, keeping downtime to zero. 
- **Canary Releases:** Sends a small percentage of live production traffic to a new version (e.g., 10%), monitors it, and gradually scales traffic up while scaling the old version down. 
- **Experimentation (A/B Testing):** Routes traffic to different versions to evaluate metrics or user behavior before a full rollout. 

#### Key Features & Capabilities

- **Automated Rollbacks & Promotions:** Can query external metric providers (such as Prometheus, Datadog, New Relic, or Wavefront) to evaluate Key Performance Indicators (KPIs) in real time. If error rates spike or latency degrades, it automatically aborts and rolls back the release. 
- **Traffic Shaping:** Integrates with ingress controllers (like NGINX or ALB) and service meshes (like Istio or Linkerd) to precisely control traffic splitting. 
- **Manual Judgment Gates:** Allows teams to pause a rollout at a specific percentage and require human approval before proceeding. 
- **GitOps Friendly:** Designed to work smoothly with declarative configurations and GitOps tools like Argo CD.

### **41. What is GitOps?**
**Answer:**

**GitOps** is **an operational framework that applies software development practices—such as version control, collaboration, code review, and CI/CD—to infrastructure automation and application delivery**.

In a GitOps workflow, a **Git repository** serves as the single source of truth for the entire system's desired state.

#### Core Principles

- **Declarative Configuration:** System and infrastructure states are described declaratively (using tools like Kubernetes manifests or Terraform scripts) rather than through imperative scripts or manual commands. 
- **Versioned and Immutable History:** All configuration changes are stored in Git, creating a complete, auditable, and rollback-ready history of every modification. 
- **Pull Requests as Change Control:** Modifications happen via Pull Requests (PRs), allowing for peer reviews, automated validation, and team approvals before merging. 
- **Continuous Reconciliation:** Automated software agents or operators (such as Argo CD or Flux) constantly monitor the Git repository and automatically sync the live infrastructure to match the declared state in Git. 

#### How It Works

1. **Write Code:** Engineers define or update infrastructure and application configurations in code files. 
2. **Submit a Pull Request:** Changes are pushed to a Git branch and submitted via a Pull Request for review and testing. 
3. **Merge and Sync:** Once approved and merged into the main branch, a reconciliation agent detects the change and updates the live environment (e.g., a Kubernetes cluster) automatically.
4. **Drift Correction:** If someone makes a manual change directly in production, the automated agent detects the mismatch (configuration drift) and reverts or overrides it to match the state stored in Git. 

#### Key Benefits

- **Auditability & Security:** Every change is tracked to a specific user and commit, improving compliance and visibility.
- **Easy Rollbacks:** If a new deployment fails, rolling back is as simple as reverting the Git commit.
- **Consistency:** Eliminates "configuration drift" and manual "cowboy engineering" errors in production environments.

### **42. What is ArgoCD?**
**Answer:** 

**ArgoCD** is **an open-source, declarative GitOps continuous delivery (CD) tool specifically designed for Kubernetes**.

Instead of manually deploying applications or using traditional push-based CI/CD pipelines (like Jenkins or GitLab CI) to apply changes, ArgoCD automates the deployment of applications by using **Git repositories as the single source of truth**.

#### How ArgoCD Works (The GitOps Flow)

1. **The Source of Truth:** You define your desired Kubernetes cluster state using standard manifests (YAML files, Helm charts, Kustomize, or Jsonnet) and store them in a Git repository.
2. **Continuous Monitoring:** ArgoCD runs inside your Kubernetes cluster and constantly monitors both your Git repository and the live cluster state.
3. **Automated Synchronization:**
   - If someone updates the code in Git, ArgoCD detects that the live cluster is **Out of Sync** and automatically pulls the changes to update the cluster.
   - If someone manually changes a setting directly in the cluster (known as **configuration drift**), ArgoCD will detect it and can automatically overwrite it to match what is written in Git.

#### Core Features

- **Declarative Setup:** Everything—from application configurations to environments—is defined as code.
- **Web UI and CLI:** It provides a highly visual dashboard to see the health, status, and architecture of your Kubernetes resources in real time.
- **Multi-Cluster Support:** A single ArgoCD instance can manage deployments across multiple external Kubernetes clusters.
- **Role-Based Access Control (RBAC):** It integrates with SSO (Single Sign-On) providers to control who can view or sync specific applications.
- **Easy Rollbacks:** Because every change is tied to a Git commit, rolling back an application to a previous working state is as simple as reverting a Git commit.

### **43. What is FluxCD?**
**Answer:**

Flux CD is **an open-source, CNCF-graduated GitOps tool for Kubernetes**. It keeps your live cluster state automatically synchronized with configuration files stored in Git repositories, OCI artifacts, or Helm buckets. 

#### Key Components (GitOps Toolkit)

- **Source Controller**: Fetches manifests and artifacts from Git repositories, S3 buckets, or OCI registries.
- **Kustomize Controller**: Applies Kustomize overlays and plain Kubernetes YAML.
- **Helm Controller**: Manages Helm chart releases and automated upgrades.
- **Notification & Image Automation Controllers**: Sends chat alerts and automatically commits new container image tags back to Git.

#### Core Benefits

- **Pull-Based Architecture**: Flux runs inside the cluster and pulls changes, eliminating the need to expose cluster credentials to external CI servers.
- **Declarative & Auditable**: Every infrastructure or application change goes through a Git pull request history.
- **Lightweight & Modular**: Built on native Kubernetes custom resource definitions (CRDs), relying directly on standard Kubernetes RBAC.

### **44. What is Tekton?**
**Answer:**

**Tekton** is **an open-source, vendor-neutral framework for creating continuous integration and continuous deployment (CI/CD) systems that runs natively on Kubernetes**. Managed by the **Continuous Delivery Foundation (CDF)** and hosted as an incubating project by the Cloud Native Computing Foundation (CNCF), it allows developers to build, test, and deploy applications across multiple cloud providers and on-premises infrastructure.

#### Core Concepts of Tekton (CI/CD Framework)

Tekton defines its building blocks as Kubernetes Custom Resources (CRDs), meaning pipelines are written and managed as standard YAML files inside your cluster:

- **Step:** The smallest, atomic unit of work (e.g., compiling code, running tests, or building a container image). Each step runs inside its own isolated container.

- **Task:** A reusable sequence of one or more steps that perform a specific build job.

- **Pipeline:** A collection of tasks arranged in a specific order to define a full CI/CD workflow.

- **TaskRun & PipelineRun:** The runtime execution instances that trigger and monitor a specific Task or Pipeline.

- **Workspaces:** Shared storage (such as a PersistentVolumeClaim) that allows data to be passed safely between different tasks in a pipeline.

#### Key Ecosystem Components

- **Tekton Triggers:** Listens for external events (like a GitHub pull request or code push) and automatically kicks off pipeline runs.
- **Tekton CLI (`tkn`):** A command-line tool built on top of `kubectl` to manage Tekton resources.
- **Tekton Dashboard:** A web-based graphical interface for viewing pipeline runs and logs.
- **Tekton Hub & Catalog:** Repositories of pre-made, community-contributed tasks and pipelines.
- **Pipelines-as-Code:** Allows developers to define pipelines directly inside a `.tekton/` folder in their source code repository.

### **45. What is Spinnaker?**
**Answer:** 

**Spinnaker** is **an open-source, multi-cloud Continuous Delivery (CD) platform** originally created by Netflix and extended by Google (now part of the Continuous Delivery Foundation / Linux Foundation). It automates software releases and application deployments across multiple cloud providers. 

#### Core Capabilities

- **Multi-Cloud Deployments:** Works natively across Amazon Web Services (AWS EC2), Google Cloud Platform (GCP/GKE), Microsoft Azure, Kubernetes, Oracle Cloud, and Cloud Foundry. 
- **Safe Deployment Strategies:** Supports progressive delivery patterns like **blue/green** (red/black) and **canary** deployments, along with automated rollbacks if a release fails. 
- **Automated Pipelines:** Lets you build multi-stage workflows triggered by CI tools (such as Jenkins or Travis CI), Git events, Docker registries, or cron schedules. 
- **Immutable Infrastructure ("Baking"):** Integrates with Packer (Rosco) to create "baked" virtual machine images or container images before deployment. 
- **Application Management:** Gives you a centralized dashboard ("Deck") to view and manage server groups, clusters, load balancers, and security groups.

#### Key Microservices Architecture

Spinnaker operates as a collection of JVM- and service-based components: 
- **Deck:** The front-end user interface.
- **Gate:** The API gateway for UI and external requests.
- **Orca:** The orchestration engine that runs pipelines and stages.
- **Clouddriver:** The core module that talks to cloud providers and caches deployed resources.
- **Front50:** Stores metadata for applications, pipelines, and projects.
- **Rosco:** The "baker" service that creates machine/VM images.

#### Pros and Cons

- **Pros:** Unmatched multi-cloud flexibility, deep control over complex release guardrails, manual judgment gates, and enterprise security features (OAuth, SAML, RBAC).
- **Cons:** High resource consumption and a steep learning curve for initial setup and maintenance.

### **46. What is Gitleaks?**
**Answer:** 

**Gitleaks** is **a free, open-source Static Application Security Testing (SAST) tool used to detect and prevent hardcoded secrets—such as API keys, passwords, tokens, and private keys—in your code repositories**. 

Maintained by **Truffle Security**, it is widely relied upon by developers and security professionals to prevent accidental credential exposure before code is pushed to production or public repositories.

#### How it Works

Gitleaks evaluates code using three core detection methods: 

1. **Regex Pattern Matching:** It scans files against a predefined and highly comprehensive rule configuration targeting over 160 specific secret types (like AWS keys, Stripe keys, and JWTs). 
2. **Entropy Analysis:** It uses mathematical entropy validation to measure the randomness of text strings. This helps catch generic or custom keys that do not follow standard patterns, flagging strings that "look" like high-entropy secrets. 
3. **Allowlists:** It allows developers to configure custom exclusions to filter out low-entropy strings or known non-sensitive data, drastically reducing false positives.

#### Key Features

- **Git History Scanning:** Gitleaks does not just scan the active, front-facing codebase; it can scan an entire repository's timeline, uncovering credentials hidden inside historical commits. 
- **Flexible Scanning Modes:** It features a `git` command for traditional version-controlled repositories, a `dir` command to scan standard unmanaged folders or non-git environments, and an `stdin` mode for checking streaming inputs. 
- **Highly Customizable:** Teams can customize rules and thresholds by configuring a local `gitleaks.toml` file. 
- **Robust Reporting:** Findings can be exported into multiple structured developer formats, including JSON, CSV, JUnit, and SARIF.

#### Where is it Integrated?

To intercept secrets early in the software development lifecycle, Gitleaks is commonly inserted directly into modern developer workflows:

- **Local Pre-commit Hooks:** Set up locally via tools like `pre-commit`, Gitleaks automatically stops a `git commit` execution if a developer accidentally attempts to stage a secret.
- **CI/CD Pipelines:** Automated workflows can run **Gitleaks-Action** on pull requests or commits inside platforms like GitHub and GitLab. If a secret is uncovered, the pipeline fails loudly and blocks the deployment.

### **47. What is Trivy?**
**Answer:** 

Trivy is a comprehensive, open-source security scanner created and maintained by Aqua Security.

#### Key Capabilities

Trivy detects security risks across modern cloud-native environments:
- **Vulnerability Scanning (CVEs):** Checks OS packages (Alpine, Debian, Ubuntu, RHEL) and application dependencies (npm, PyPI, Maven, Go, NuGet) for known software glitches. 
- **Misconfiguration Detection:** Identifies security flaws in Infrastructure-as-Code (IaC) files like Terraform, Dockerfiles, and Kubernetes manifests. 
- **Secret Detection:** Uncovers hardcoded API keys, passwords, and tokens in code or configurations. 
- **License Analysis:** Checks software licenses for legal and corporate compliance. 
- **Software Bill of Materials (SBOM):** Generates and analyzes structured inventories of all software components and modules. 

#### Targets

Trivy can scan a wide range of artifacts and deployments: 

- Container images (Docker, Podman)
- Local filesystems and code repositories (GitHub, GitLab)
- Kubernetes clusters
- Virtual machine (VM) images and AWS environments 

#### How It Works

Trivy uses a simple Command Line Interface (CLI) that requires minimal setup. It downloads and caches an internal vulnerability database (`trivy-db`) to match installed packages against known security advisories. It can also be integrated directly into CI/CD pipelines like **GitHub Actions**, GitLab CI, and VS Code.

### **48. What is a GitHub Actions Workflow?**
**Answer:**

A **GitHub Actions workflow** is **a configurable, automated process that executes one or more jobs to automate software development tasks**. It is a core component of **GitHub Actions**, GitHub's built-in Continuous Integration and Continuous Delivery (CI/CD) platform.

Workflows are defined using **YAML files** and must be stored in a specific directory in your repository: `.github/workflows/`.

#### Core Components of a Workflow

To understand how a workflow operates, it helps to look at its core architecture: 

- **Events (Triggers):** The specific activity that kicks off the workflow. This can be an internal GitHub event (like a code `push` or a `pull_request` creation), a cron-based `schedule`, or a manual button click (`workflow_dispatch`).
- **Jobs:** A set of execution steps. By default, multiple jobs inside a workflow run in parallel, but you can configure them to run sequentially if one depends on another. 
- **Runners:** The server or virtual machine that executes the jobs. GitHub provides hosted runners (Linux, Windows, macOS), or you can host your own. 
- **Steps:** Individual tasks within a job that run sequentially. A step can either run a raw shell command or invoke an **Action**. 
- **Actions:** Pre-built, reusable applications or scripts that perform complex, repetitive tasks (e.g., setting up a specific version of Node.js or checking out code from your repository). You can find thousands of these in the **GitHub Marketplace**.

#### Basic Workflow Example

Below is a simple workflow configuration file (e.g., `ci.yml`) that automatically tests code every time a developer pushes changes to the `main` branch: 

**YAML**

```yaml
name: Node.js CI

# The event triggering the workflow
on:
  push:
    branches: [ main ]

# The automation tasks
jobs:
  test-code:
    runs-on: ubuntu-latest # The runner environment

    steps:
      - name: Check out repository code
        uses: actions/checkout@v4 # A reusable GitHub Action

      - name: Set up Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'

      - name: Install dependencies
        run: npm install # Raw command

      - name: Run unit tests
        run: npm test
```

#### Common Use Cases

Developers use workflows to move manual processes into autopilot: 

1. **CI/CD Pipelines:** Automatically compiling code, running tests, and deploying applications to production or staging environments upon code merges.
2. **Code Quality Controls:** Running security scanners (like SAST/DAST) or code linters against Pull Requests to catch errors early. 
3. **Repository Management:** Automating administrative tasks, such as adding specific labels to newly opened issues or sending Slack notifications when a build fails.

### **49. What is a GitHub Actions Job vs Step?**
**Answer:** 

In **GitHub Actions**, **a job is an execution environment (a virtual server), while a step is an individual task or command executed inside that environment**.

Think of a **job** as a **kitchen** where a meal is prepared, and **steps** as the **individual recipe instructions** carried out in that kitchen (like chopping onions or boiling water).

#### Direct Comparison

| **Feature** | **Job ⚙️** | **Step 📋** |
|---|---|---|
| **What it is** | A collection of steps running on a specific server. | The smallest unit of work (a script or action). |
| **Execution Order** | Runs in **parallel** by default (can be made sequential). | Runs strictly **sequentially** (one after the other). |
| **Environment** | Runs on its own **isolated runner** (e.g., Ubuntu, Windows). | Shares the **same runner and file system** as other steps in that job. |
| **Data Sharing** | Must use artifacts or outputs to pass data to other jobs. | Can natively pass data via the filesystem or variables. |

#### Deep Dive: Jobs

A job is a defined slice of your pipeline that gets assigned to a specific computer called a **runner**. 

- **Isolation:** Every job runs on a brand-new, clean virtual environment. Files created in "Job A" are not visible to "Job B" unless you explicitly save them. 
- **Parallelization:** If you have a `test` job and a `lint` job, GitHub will run them at the same time on different machines to speed up your pipeline. 
- **Dependencies:** You can force a job to wait for another using the `needs` keyword (e.g., `deploy` only runs after `build` succeeds).

#### Deep Dive: Steps

Steps are the granular tasks that make up a job. A step cannot exist outside of a job.

- **Sequence:** Steps execute in order, from top to bottom. If Step 2 fails, the job stops, and subsequent steps are skipped. 
- **Shared State:** Because all steps in a job run on the same machine, Step 2 can easily use a file created by Step 1. 
- **Types of Steps:** A step can either run a raw command-line script (using `run`) or call a pre-made, reusable block of code (using `uses`). 

#### YAML Example

Here is how they look inside a **GitHub Actions Workflow Syntax** file: 

```yaml
jobs:
  # 'build-and-test' is a JOB
  build-and-test:
    runs-on: ubuntu-latest

    # Inside this job are multiple STEPS
    steps:
      - name: Check out repository code
        uses: actions/checkout@v4  # Step 1: Uses a pre-made action

      - name: Install dependencies
        run: npm install            # Step 2: Runs a shell command

      - name: Run test suite
        run: npm test               # Step 3: Runs a shell command
```

### **50. What is a Reusable Workflow in GitHub Actions?**
**Answer:** 

A **Reusable Workflow** in GitHub Actions is **a pre-defined YAML configuration file that can be called and executed inside other workflows**. It functions as a modular component, allowing you to centralize repetitive pipeline configurations (like testing, security scans, or deployment scripts) in one location and invoke them across multiple repositories or projects. This design aligns directly with the **DRY (Don’t Repeat Yourself)** principle, dramatically reducing copy-pasted YAML code and simplifying organization-wide CI/CD maintenance. 

#### How It Works

1. **The Called (Reusable) Workflow:** To make a workflow reusable, you define it inside a repository's `.github/workflows` directory and configure it to trigger on the **`workflow_call`** event. You can also declare parameters (`inputs`) and `secrets` that it needs to run.
2. **The Caller Workflow:** To use the reusable workflow, another pipeline file references it using the **`uses`** keyword at the **job level** (not inside a step).

When triggered, the reusable workflow runs the jobs exactly as if they were natively written inside the caller workflow, executing inside the context of the repository that called it.

#### Key Features & Architecture

- **Nesting Capabilities:** A reusable workflow can call other reusable workflows, allowing up to **10 levels of nesting** (1 top-level caller + 9 called workflows). 
- **Inputs, Secrets, and Outputs:** You can seamlessly pass runtime variables via `with` and secure credentials via `secrets`. You can also map specific step outputs back up to the caller workflow to pass data downstream. 
- **Cross-Repository Access:** Depending on your configuration, reusable workflows can be shared locally within the same repository, across public repositories, or internally within a GitHub Organization. 
- **Real-time Logging:** Each job and individual step within a reusable workflow is logged separately in real-time in the GitHub Action UI, making troubleshooting highly transparent.

#### Reusable Workflows vs. Composite Actions

While both features aim to minimize redundancy, they serve distinct architectural purposes: 

| **Feature** | **Reusable Workflows** | **Composite Actions** |
|---|---|---|
| **Execution Level** | Runs at the **Job** level. | Runs as a **Step** inside an existing job. |
| **Structure** | Can contain **multiple jobs** and configure unique runners (`runs-on`). | Contains a **bundle of steps** running on the caller's pre-configured runner. |
| **Secrets Management** | Can natively pass or inherit secrets (`secrets: inherit`). | Cannot natively manage secrets; inputs must be explicitly mapped. |
| **Best Used For...** | Standardizing **entire CI/CD pipelines** across an organization. | Packaging **sequences of shell steps** or local scripts. |

#### Basic Syntax Example

#### 1. The Reusable Workflow (`.github/workflows/reusable-build.yml`)

```yaml
name: Reusable Build Pipeline
on:
  workflow_call:
    inputs:
      node-version:
        required: true
        type: string

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Setup Node
        uses: actions/setup-node@v4
        with:
          node-version: ${{ inputs.node-version }}
      - run: npm ci
      - run: npm test
```

#### 2. The Caller Workflow (`.github/workflows/main.yml`)

```yaml
name: Main Application CI
on:
  push:
    branches: [ main ]

jobs:
  run-global-build:
    # Syntax: {owner}/{repo}/.github/workflows/{file}@{ref}
    uses: my-organization/shared-workflows/.github/workflows/reusable-build.yml@main
    with:
      node-version: '20'
```

### **51. What is a Composite Action in GitHub Actions?**
**Answer:** 

A **Composite Action** in GitHub Actions is **a reusable package that groups multiple workflow steps into a single action**. Instead of copying and pasting the same sequence of shell commands or actions across different jobs or repositories, you bundle them together to keep your YAML files clean and dry. 

#### Key Characteristics & Mechanics

- **Step-Level Execution:** Composite actions run exclusively as a step inside a job. They cannot define their own jobs, select runners, or dictate target environments. 
- **The `using: "composite"` Requirement:** To declare a composite action, you must set the `using: "composite"` parameter within its metadata file. 
- **Mandatory Shell Declaration:** Unlike standard workflows, every `run` step inside a composite action must explicitly define a `shell` (e.g., `shell: bash` or `shell: pwsh`). 
- **Inputs & Outputs:** They support customized parameters, allowing you to pass variables (`inputs`) into the action and return generated data (`outputs`) back to the parent workflow. 

#### Structural Example (`action.yml`)

A composite action is managed via an `action.yml` or `action.yaml` metadata file. It looks like this: 

```yaml
name: 'Setup and Test Application'
description: 'Bundles environment setup and tests into one step'
inputs:
  node-version:
    description: 'Version of Node to use'
    required: true
    default: '20'
runs:
  using: 'composite'
  steps:
    - name: Set up Node.js
      uses: actions/setup-node@v4
      with:
        node-version: ${{ inputs.node-version }}

    - name: Install Dependencies
      run: npm ci
      shell: bash

    - name: Run Tests
      run: npm test
      shell: bash
```

#### Composite Actions vs. Reusable Workflows

While both features aim to reduce duplication, they target different scopes: 

| **Feature** | **Composite Actions** | **Reusable Workflows** |
|---|---|---|
| **Scope** | **Steps within a job** | **Entire jobs** (or multiple jobs) |
| **Invocation** | Called via `uses:` under a job's `steps` | Called directly as a job definition |
| **Secrets** | Cannot read standard secrets directly; must be passed as `inputs` or env vars | Can inherit or be explicitly passed secrets |
| **Logging** | Collapses multiple internal steps into a single log entry | Provides distinct, real-time logging for each job and step |

### **55. How do you share files between jobs in GitHub Actions?**
**Answer:** Since jobs run on independent virtual machines, files must be uploaded as artifacts using `actions/upload-artifact` in Job A and downloaded using `actions/download-artifact` in Job B.

### **56. How do you configure OIDC with AWS in GitHub Actions?**
**Answer:** Set `permissions: { id-token: write, contents: read }`, then use `aws-actions/configure-aws-credentials@v4` with `role-to-assume` to exchange the GitHub JWT for temporary AWS IAM credentials.

### **57. What is `needs` in GitHub Actions?**
**Answer:** An attribute that defines sequential job dependencies (e.g., `needs: [build, test]` ensures the deploy job runs only after both build and test succeed).

### **58. What is `fail-fast` in GitHub Actions Matrix builds?**
**Answer:** A boolean setting under `strategy:`. If `true` (default), GitHub cancels all running matrix jobs if any single job fails. Setting `fail-fast: false` allows all matrix variations to run to completion.

### **59. What are GitHub Actions Environments and Protection Rules?**
**Answer:** Deployment targets (e.g., `production`) configured with required reviewers, wait timers, and branch protection rules that must be approved before deployment jobs execute.

### **60. How do you securely pass secrets to Reusable Workflows?**
**Answer:** Use `secrets: inherit` in the calling workflow to pass all caller secrets, or explicitly pass specific secrets via `secrets: { DB_PASSWORD: ${{ secrets.DB_PASS }} }`.

### **61. What is the `.gitlab-ci.yml` architecture?**
**Answer:** A declarative configuration file defining stages, jobs, scripts, and artifact handling natively integrated into GitLab repositories and container registries.

### **62. What are GitLab CI Stages?**
**Answer:** Sequential execution blocks (e.g., `stages: [build, test, deploy]`). All jobs within the same stage run concurrently; the next stage starts only after all jobs in the current stage succeed.

### **63. What is GitLab CI `rules` keyword?**
**Answer:** A powerful conditional syntax determining whether a job is included in the pipeline based on branch names, commit messages, file changes, or pipeline variables.

### **64. What is a GitLab Runner?**
**Answer:** An open-source application that executes pipeline jobs defined in `.gitlab-ci.yml`, supporting Docker, Kubernetes, SSH, and Shell executors.

### **65. What is GitLab Auto DevOps?**
**Answer:** A pre-configured CI/CD pipeline template that automatically detects programming languages, builds container images, executes security tests, and deploys to Kubernetes without manual pipeline coding.

### **66. What is a Declarative Jenkinsfile?**
**Answer:** A structured, syntax-checked pipeline format (`pipeline { agent any; stages { ... } }`) with built-in directives for environments, parameters, and post-build actions.

### **67. What is a Scripted Jenkinsfile?**
**Answer:** A Groovy-based procedural pipeline format (`node { ... }`) offering unlimited programmatic flexibility at the cost of higher maintenance complexity.

### **68. What is Jenkins Configuration as Code (JCasC)?**
**Answer:** Defining the entire Jenkins controller configuration (plugins, security realms, credentials, node settings) in declarative YAML files stored in Git.

### **69. What is the Jenkins Kubernetes Plugin?**
**Answer:** A plugin that allows the Jenkins controller to dynamically spawn ephemeral agent pods in a Kubernetes cluster to execute build jobs, terminating pods immediately on job completion.

### **70. What is Jenkins Shared Libraries?**
**Answer:** A centralized Git repository of reusable Groovy code and pipeline steps that can be imported and executed across multiple independent Jenkinsfiles.

### **71. How do you prevent credentials leakage in Jenkins logs?**
**Answer:** Use `withCredentials([string(credentialsId: '...', variable: 'API_TOKEN')]) { ... }` which automatically masks the secret value in console logs.

### **72. What is Jenkins Multibranch Pipeline?**
**Answer:** A Jenkins job type that automatically scans a Git repository, creates pipeline jobs for every detected branch with a `Jenkinsfile`, and deletes jobs for merged branches.

### **73. What is Blue Ocean in Jenkins?**
**Answer:** A modern, visual user interface for Jenkins designed to visualize complex multi-stage pipeline executions and failure points.

### **74. What is GitLab CI `cache` vs `artifacts`?**
**Answer:** `cache` is used to speed up subsequent runs by caching project dependencies across pipelines; `artifacts` are files passed between sequential stages within the same pipeline.

### **75. What is GitHub Actions `hashFiles()` function?**
**Answer:** Computes an MD5/SHA256 hash of matching files (e.g., `hashFiles('**/package-lock.json')`) used as a dynamic cache key to invalidate dependencies when lockfiles change.

### **76. What is GitHub Actions `workflow_dispatch`?**
**Answer:** An event trigger allowing users to trigger workflows manually from the GitHub UI or API with customizable input parameters.

### **77. What is GitHub Actions `repository_dispatch`?**
**Answer:** An HTTP webhook trigger allowing external systems (e.g., third-party webhooks, custom microservices) to trigger a GitHub Actions workflow with a JSON payload.

### **78. How do you debug GitHub Actions workflows in real time?**
**Answer:** Enable runner diagnostic logging by setting repository secrets `ACTIONS_RUNNER_DEBUG=true` and `ACTIONS_STEP_DEBUG=true`, or use tools like `tmate` for interactive SSH debugging.

### **79. What is a Jenkins Blue-Green deployment plugin?**
**Answer:** Plugins or custom pipeline scripts that orchestrate swapping load balancer target groups or updating DNS records between Blue and Green environments.

### **80. What is GitLab CI DAG (Directed Acyclic Graph) Pipeline?**
**Answer:** Using the `needs` keyword in GitLab CI to allow jobs to start immediately once their specific prerequisites complete, regardless of stage ordering.

### **81. What is GitHub Actions Concurrency Group?**
**Answer:** A named grouping (`concurrency: ${{ github.workflow }}-${{ github.ref }}`) that limits concurrent execution of workflows, queuing or canceling redundant runs.

### **82. How do you implement automated semantic releases in GitLab CI?**
**Answer:** Run the `semantic-release` NPM package in the release stage, which analyzes commit messages, creates Git tags, generates release notes, and publishes packages.

### **83. What is the Jenkins Pipeline `post` section?**
**Answer:** Directives (`always`, `success`, `failure`, `unstable`, `cleanup`) executed at the completion of a pipeline or stage to send Slack alerts or clean workspace disks.

### **84. What is a Jenkins Agent vs Controller?**
**Answer:** The Controller manages the web UI, parses pipeline scripts, and schedules builds; Agents are worker instances that execute the actual build steps.

### **85. How do you secure Jenkins Controller from malicious agents?**
**Answer:** Enable Agent-to-Controller Access Control, disable CLI access over remoting, and run build agents in isolated ephemeral containers with minimal host permissions.

### **86. What is GitHub Actions `step-security/harden-runner`?**
**Answer:** A security action that monitors outbound network traffic from GitHub runners, blocks DNS exfiltration, and detects file tampering during CI execution.

### **87. What is GitHub Actions Composite Action `using: "composite"`?**
**Answer:** The declaration in `action.yml` indicating the action is built using composite steps rather than a Docker container or JavaScript runtime.

### **88. What is GitLab CI Include keyword?**
**Answer:** Directives (`include:local`, `include:file`, `include:remote`, `include:template`) allowing pipelines to modularize and import external YAML files.

### **89. What is GitHub Actions `runner.temp` vs `runner.workspace`?**
**Answer:** `runner.temp` is an isolated temporary directory wiped after job completion; `runner.workspace` is the directory where the repository is cloned.

### **90. How do you manage Docker-in-Docker (dind) securely in GitLab CI?**
**Answer:** Use rootless Docker or TLS-enabled Docker daemons (`DOCKER_TLS_CERTDIR="/certs"`), or switch to daemonless build tools like **Kaniko**.

### **91. What is Kaniko and why is it preferred for building containers in Kubernetes?**
**Answer:** A Google open-source tool that builds container images from a Dockerfile inside a Kubernetes pod **without requiring a Docker daemon or privileged host root access**.

### **92. What is Buildah?**
**Answer:** A command-line tool for building OCI container images without requiring a background container daemon or root privileges.

### **93. What is GitHub Actions Token (`GITHUB_TOKEN`)?**
**Answer:** An automatically generated, short-lived secret token provided to each workflow run with scoped permissions defined via the `permissions:` block.

### **94. What is GitHub Actions Least-Privilege Permissions?**
**Answer:** Explicitly defining `permissions: { contents: read, id-token: write }` at the top of workflows to override overly permissive default repository settings.

### **95. How do you run scheduled cron jobs in GitHub Actions?**
**Answer:** Using the `schedule` trigger: `on: schedule: - cron: '0 2 * * *'` (runs daily at 2:00 AM UTC).

### **96. What is GitLab CI Environment Variables Hierarchy?**
**Answer:** Variables are resolved with precedence: Project Variables $>$ Group Variables $>$ Instance Variables $>$ Pipeline Variables $>$ YAML Variables.

### **97. What is Jenkins Pipeline Syntax Generator?**
**Answer:** A built-in web tool (`/pipeline-syntax`) that generates accurate Groovy code snippets for specific plugins and steps.

### **98. What is SonarQube Scanner in CI?**
**Answer:** A CLI tool that executes static code analysis, uploads AST reports to the SonarQube server, and awaits Quality Gate webhook evaluation.

### **99. What is GitLab Review Apps?**
**Answer:** Ephemeral dynamic environments spun up automatically per branch in GitLab CI, integrated directly with merge request review pages.

### **100. What is GitHub Actions `continue-on-error`?**
**Answer:** A step-level attribute that prevents a job from failing if a non-critical step (e.g., an experimental linter) returns a non-zero exit code.

---

## 🔴 **Part 3: GitOps, Progressive Delivery & Supply Chain Security (Questions 101–200)**

### **101. What is GitOps?**
**Answer:** An operational model where Git repositories serve as the single source of truth for declarative infrastructure and application deployments, using automated pull-based reconciliation operators.

### **102. What are the Four Principles of GitOps (OpenGitOps)?**
**Answer:** 1. Declarative Desired State, 2. Versioned and Immutable Storage in Git, 3. Pulled Automatically by In-Cluster Agents, 4. Continuously Reconciled with Automated Drift Correction.

### **103. What is ArgoCD?**
**Answer:** A declarative GitOps continuous delivery operator for Kubernetes that continuously monitors Git repositories and synchronizes live cluster state with desired state.

### **104. What is the ArgoCD Application CRD?**
**Answer:** A Kubernetes Custom Resource defining the source repository, path, target cluster, target namespace, and sync policies for an application.

### **105. What is an ArgoCD ApplicationSet?**
**Answer:** A controller that automates the generation and multi-cluster deployment of multiple ArgoCD `Application` resources using List, Cluster, Git Directory, or Matrix generators.

### **106. What are ArgoCD Sync Waves?**
**Answer:** Annotations (`argocd.argoproj.io/sync-wave: "1"`) that control the exact numerical execution order of Kubernetes resources during synchronization (lower/negative numbers apply first).

### **107. What are ArgoCD Sync Phases?**
**Answer:** PreSync $\rightarrow$ Sync $\rightarrow$ PostSync $\rightarrow$ SyncFail. Allows running prerequisite jobs (database migrations) before updating application deployments.

### **108. What is ArgoCD Auto-Sync and Self-Healing?**
**Answer:** Auto-Sync automatically applies new Git commits to the cluster; Self-Healing detects manual out-of-band cluster edits and overwrites them back to the Git source of truth.

### **109. What is ArgoCD Prune?**
**Answer:** An automated synchronization setting that deletes Kubernetes resources from the cluster when their corresponding manifest files are removed from Git.

### **110. What is FluxCD?**
**Answer:** A set of continuous and progressive delivery controllers for Kubernetes (Source Controller, Kustomize Controller, Helm Controller) implementing GitOps.

### **111. What is FluxCD Kustomization CRD?**
**Answer:** A resource defining a pipeline for applying Kustomize overlays from a Git repository to a target Kubernetes cluster with health checking and automated rollback.

### **112. What is FluxCD HelmRelease?**
**Answer:** A declarative resource that manages the lifecycle of Helm chart releases, automatically pulling charts from OCI or HTTP repositories and reconciling values.

### **113. What is Progressive Delivery?**
**Answer:** Advanced continuous delivery combining canary deployments, traffic routing, feature flags, and automated metric analysis to minimize blast radius during releases.

### **114. What is Argo Rollouts?**
**Answer:** A Kubernetes operator replacing standard Deployments with a `Rollout` CRD that coordinates canary traffic splitting and automated metric analysis.

### **115. What is an AnalysisTemplate in Argo Rollouts?**
**Answer:** A declarative template defining Prometheus, Datadog, or CloudWatch queries used to evaluate canary health during progressive rollouts.

### **116. What is Flagger?**
**Answer:** A progressive delivery operator that automates canary releases, A/B testing, and blue-green deployments on Kubernetes using Service Meshes (Istio, Linkerd) and Ingress controllers.

### **117. What is Software Supply Chain Security?**
**Answer:** Protecting against unauthorized modifications, malicious dependency injections, and compromised build systems across the entire software development lifecycle.

### **118. What is the SLSA Framework (Levels 1–3)?**
**Answer:** Level 1: Scripted build generating provenance; Level 2: Hosted build service with source integrity; Level 3: Isolated, ephemeral, hermetic build platform with cryptographically signed, non-falsifiable provenance.

### **119. What is Keyless Signing with Sigstore Cosign?**
**Answer:** Signing container images using short-lived OIDC tokens exchanged for X.509 certificates from Fulcio, recording signatures in the Rekor transparency log.

### **120. What is Rekor in Sigstore?**
**Answer:** An immutable, tamper-evident, append-only transparency log that records signed artifact metadata and proof of provenance.

### **121. What is Fulcio in Sigstore?**
**Answer:** A free, public Certificate Authority that issues short-lived (10-minute) X.509 certificates bound to OpenID Connect identities (e.g., GitHub Actions workflows).

### **122. What is Syft?**
**Answer:** An open-source CLI tool and library for generating Software Bill of Materials (SBOMs) from container images, filesystems, and archives in CycloneDX and SPDX formats.

### **123. What is Grype?**
**Answer:** An open-source vulnerability scanner specifically designed to scan container images and SBOM files for known security vulnerabilities.

### **124. What is In-Toto?**
**Answer:** A framework for cryptographic verification of software supply chain integrity, ensuring every step from commit to build to packaging was performed by authorized actors.

### **125. What is Kyverno Image Verification?**
**Answer:** An admission policy rule that verifies container image cryptographic signatures against Sigstore Cosign before allowing pods to schedule in Kubernetes.

### **126. What is Bitnami Sealed Secrets?**
**Answer:** A GitOps secret management tool where secrets are encrypted client-side with a public key and committed to Git, decrypted inside the cluster by a controller possessing the private key.

### **127. What is External Secrets Operator (ESO)?**
**Answer:** A Kubernetes operator that synchronizes secrets from enterprise vaults (AWS Secrets Manager, HashiCorp Vault, Azure Key Vault) into native Kubernetes `Secret` resources.

### **128. What is HashiCorp Vault Agent Sidecar Injector?**
**Answer:** A Kubernetes mutating webhook that injects a Vault agent container into application pods to dynamically fetch and render secrets into an in-memory volume.

### **129. What is Atlantis for Terraform?**
**Answer:** An open-source application that executes `terraform plan` and `terraform apply` directly inside GitHub/GitLab Pull Request comments with automated state locking.

### **130. What is Spacelift?**
**Answer:** A specialized CI/CD management platform for Infrastructure as Code (Terraform, OpenTofu, Pulumi, CloudFormation, Kubernetes) with policy enforcement via OPA.

### **131. What is GitHub Actions Runner Controller (ARC) Autoscaling?**
**Answer:** Autoscaling runner pods dynamically based on GitHub API metrics (`workflow_job` queue depth) from 0 to hundreds of instances.

### **132. What is Ephemeral Runner Security?**
**Answer:** Destroying self-hosted runner pods/VMs immediately after completing a single job to prevent lateral movement, credential theft, and state pollution.

### **133. What is Snyk?**
**Answer:** A commercial developer security platform scanning source code (SAST), dependencies (SCA), container images, and IaC templates for security vulnerabilities.

### **134. What is SonarQube?**
**Answer:** A continuous code quality platform evaluating code coverage, duplication, complexity, code smells, and security vulnerabilities.

### **135. What is OWASP ZAP in CI/CD?**
**Answer:** An open-source web application security scanner used in CI pipelines to execute automated Dynamic Application Security Testing (DAST).

### **136. What is Checkov?**
**Answer:** A static analysis tool for IaC that scans Terraform, Kubernetes manifests, Helm charts, and Dockerfiles for security misconfigurations.

### **137. What is tfsec?**
**Answer:** A fast static analysis security scanner for Terraform code, now integrated into Trivy.

### **138. What is Infracost?**
**Answer:** A FinOps tool that parses Terraform code in pull requests to calculate monthly cloud cost impact before merging code.

### **139. What is Kustomize in GitOps?**
**Answer:** A template-free configuration manager that customizes Kubernetes YAML manifests using declarative overlays (dev, staging, prod) over a base configuration.

### **140. What is Helm in GitOps?**
**Answer:** A package manager for Kubernetes that bundles related manifests into reusable Charts, parameterized using `values.yaml` files.

### **141. What is Helmfile?**
**Answer:** A declarative spec for deploying multiple Helm charts across multiple Kubernetes clusters in dependency order.

### **142. What is Tekton Pipelines?**
**Answer:** A Kubernetes-native CRD defining a Directed Acyclic Graph (DAG) of Tasks executed sequentially or in parallel inside ephemeral pods.

### **143. What is Spinnaker Automated Canary Analysis (Kayenta)?**
**Answer:** An automated statistical analysis engine comparing canary metrics against baseline metrics over time to make automated promotion decisions.

### **144. What is Chaos Mesh in CI/CD?**
**Answer:** Injecting automated network delays, pod failures, and disk stress into staging environments during CI pipeline execution to validate resilience.

### **145. What is Semantic Release?**
**Answer:** An automated tool that analyzes commit messages to determine the next SemVer version, generates changelogs, creates Git tags, and publishes releases.

### **146. What is Release Please?**
**Answer:** A Google tool that generates Release PRs containing updated changelogs and version bumps based on conventional commit history.

### **147. What is Dependabot / Renovate?**
**Answer:** Automated dependency update bots that scan repository manifests, check for new package releases, and automatically open PRs with changelog summaries.

### **148. What is Renovate Bot?**
**Answer:** A highly configurable, multi-platform dependency update tool supporting automated merging of non-breaking security patches.

### **149. What is a Pull Request Builder Job?**
**Answer:** An automated CI job that triggers when a PR is created or updated to compile code, run tests, and report status checks back to the PR.

### **150. What is a GitOps Out-of-Sync State?**
**Answer:** A condition where the live state of Kubernetes cluster resources differs from the declared configuration stored in Git.

### **151. What is GitOps Drift Correction?**
**Answer:** The automatic overwrite of unauthorized out-of-band manual changes in a cluster back to the version declared in Git.

### **152. What is an ArgoCD PreSync Hook?**
**Answer:** A Kubernetes resource (Job) executed before any other deployment resources are applied, commonly used for database schema migrations.

### **153. What is an ArgoCD PostSync Hook?**
**Answer:** A script or notification Job executed only after all deployment resources have successfully become healthy in the cluster.

### **154. What is an ArgoCD SyncFail Hook?**
**Answer:** A remediation Job executed when an ArgoCD synchronization operation fails.

### **155. What is Secret Masking Bypass Risk?**
**Answer:** If secrets are base64-encoded or split across multiple strings, CI engines will fail to match the secret string and print it in plain text.

### **156. What is Git Credential Helper in CI?**
**Answer:** A utility allowing CI/CD runners to authenticate with remote Git repositories using short-lived OAuth tokens instead of hardcoded passwords.

### **157. What is GitHub Actions Reusable Workflow Inheritance?**
**Answer:** Passing all secrets and context from a caller workflow to a reusable workflow using `secrets: inherit`.

### **158. What is Container Layer Squashing?**
**Answer:** Merging all intermediate build layers of a container into a single layer to reduce image size and discard temporary files.

### **159. What is Multi-Arch Container Building?**
**Answer:** Using Docker `buildx` to compile container images for multiple CPU architectures (`linux/amd64`, `linux/arm64`) from a single Dockerfile.

### **160. What is OCI (Open Container Initiative)?**
**Answer:** An open governance industry standard defining specifications for container image formats (Image Spec) and runtimes (Runtime Spec).

### **161. What is Cosign Attestation?**
**Answer:** Cryptographically signing metadata predicates (SBOMs, test results, vulnerability scan reports) and attaching them to the container image in the registry.

### **162. What is Kyverno ClusterPolicy vs Policy?**
**Answer:** `ClusterPolicy` applies to all resources across the entire cluster; `Policy` is scoped strictly to a single namespace.

### **163. What is OPA Gatekeeper ConstraintTemplate?**
**Answer:** A Custom Resource defining the declarative Rego logic for a policy, instantiated by `Constraint` CRDs.

### **164. What is a Continuous Integration Feedback Loop?**
**Answer:** The time elapsed from a developer pushing a commit to receiving automated test results; must be $< 10$ minutes to maintain high velocity.

### **165. What is Monorepo Incremental Building?**
**Answer:** Using cached compilation outputs so that changing one file only recompiles that file and its direct dependents.

### **166. What is Bazel?**
**Answer:** A fast, scalable, multi-language build system developed by Google that enforces hermetic, reproducible builds and aggressive caching.

### **167. What is Turborepo?**
**Answer:** A high-performance build system for JavaScript/TypeScript monorepos that caches build and test execution outputs remotely.

### **168. What is Nx?**
**Answer:** A smart, extensible build framework with advanced dependency graph visualization and distributed task execution for monorepos.

### **169. What is a Git Pre-Commit Hook?**
**Answer:** A client-side script executed automatically before `git commit` runs, used to lint code, format files, and check for committed secrets locally.

### **170. What is `pre-commit` framework?**
**Answer:** A multi-language package manager for managing and maintaining pre-commit hooks via a declarative `.pre-commit-config.yaml` file.

### **171. What is Trunk-Based Feature Flag Lifecycle?**
**Answer:** 1. Create flag $\rightarrow$ 2. Implement logic behind flag $\rightarrow$ 3. Merge to `main` $\rightarrow$ 4. Enable flag in production $\rightarrow$ 5. Clean up flag conditional code.

### **172. What is Dark Launching vs Shadowing?**
**Answer:** Dark launching deploys backend code with no UI changes; Shadowing duplicates live incoming HTTP traffic and replays it against the new version.

### **173. What is an Artifact Provenance Document?**
**Answer:** Cryptographically signed metadata recording exactly who built the artifact, from which commit SHA, on which CI runner, and using which build parameters.

### **174. What is CycloneDX?**
**Answer:** An OWASP-backed, lightweight Software Bill of Materials (SBOM) standard designed for application security and vulnerability analysis.

### **175. What is SPDX?**
**Answer:** An open standard (ISO/IEC 5962:2021) for communicating Software Bill of Materials data, including components, licenses, and copyrights.

### **176. What is a Vulnerability Exploitability eXchange (VEX)?**
**Answer:** A machine-readable companion to an SBOM that declares whether a specific CVE in a dependency is actually exploitable in the context of the application.

### **177. What is Supply Chain Security SLSA Provenance?**
**Answer:** An in-toto attestation generated during the build step certifying the source repository, commit, and build environment.

### **178. What is Docker Content Trust (DCT)?**
**Answer:** A legacy Docker feature using Notary and digital signatures to verify the integrity and publisher of specific image tags.

### **179. What is a Pipeline Deadlock?**
**Answer:** A state where two or more pipeline jobs wait indefinitely on mutually dependent resources or locks (e.g., job A holds lock 1 waiting for job B, while job B holds lock 2 waiting for job A).

### **180. What is CI/CD Pipeline Blast Radius?**
**Answer:** The maximum potential damage caused if a CI/CD system is compromised (mitigated by using pull-based GitOps and ephemeral runners).

### **181. What is Automated Rollback in Argo Rollouts?**
**Answer:** Automatically aborting a canary deployment and reverting traffic to stable pods when an AnalysisTemplate metric breach occurs.

### **182. What is a Canary Step Weight?**
**Answer:** The percentage of total traffic allocated to the canary version during a specific phase of a progressive rollout (e.g., `setWeight: 20`).

### **183. What is a Flagger Metric Template?**
**Answer:** A Custom Resource defining Prometheus PromQL queries (e.g., error rate $< 1\%$, latency $< 500\text{ms}$) checked during canary analysis.

### **184. What is Blue-Green Cutover Latency?**
**Answer:** The time taken for load balancers or DNS to update routing rules from Blue to Green; with Kubernetes Services, this occurs in sub-seconds.

### **185. What is a Pipeline Concurrency Lock?**
**Answer:** Restricting deployment pipelines to execute one at a time per target environment to prevent concurrent overlapping state modifications.

### **186. What is a GitOps Sync Window?**
**Answer:** Scheduled maintenance windows in ArgoCD that allow or deny automated synchronizations during specific hours (e.g., blocking prod syncs on weekends).

### **187. What is ArgoCD SSO (Single Sign-On)?**
**Answer:** Integrating ArgoCD with enterprise identity providers (Okta, Azure AD, GitHub) via OIDC or SAML to enforce role-based access control.

### **188. What is FluxCD Source Controller?**
**Answer:** A dedicated Flux controller that watches Git repositories, Helm charts, and S3 buckets for changes and produces artifact archives for other controllers.

### **189. What is FluxCD Kustomize Controller?**
**Answer:** A Flux controller that takes artifacts from the Source Controller, generates Kubernetes manifests via Kustomize, and applies them to the cluster.

### **190. What is FluxCD Notification Controller?**
**Answer:** A Flux controller that handles inbound webhooks from Git providers and dispatches outbound event notifications to Slack, MS Teams, and Discord.

### **191. What is Tekton PipelineRun?**
**Answer:** A Custom Resource that instantiates and executes a `Pipeline` on a Kubernetes cluster, binding concrete parameters and workspaces.

### **192. What is Tekton TaskRun?**
**Answer:** A Custom Resource that instantiates and executes a single `Task` inside a Kubernetes pod composed of sequential container steps.

### **193. What is a Matrix Job in GitLab CI?**
**Answer:** Using the `parallel: matrix:` keyword in `.gitlab-ci.yml` to run multiple job variations across defined variables concurrently.

### **194. What is a Pipeline Secret Store CSI Driver?**
**Answer:** Mounting secrets stored in enterprise vaults directly into Kubernetes pods as in-memory files without persisting them as Kubernetes Secret objects.

### **195. What is a Zero-Downtime Database Migration sequence in CI/CD?**
**Answer:** 1. Add nullable columns (Expand), 2. Deploy app writing to both old and new columns, 3. Backfill data, 4. Deploy app reading from new column, 5. Drop old columns (Contract).

### **196. What is In-Tree vs Out-of-Tree CI Plugins?**
**Answer:** In-tree plugins are compiled directly into the core CI engine; Out-of-tree plugins are external community actions downloaded dynamically at runtime.

### **197. What is Action Pinning in GitHub Actions?**
**Answer:** Pinning third-party actions to full commit SHAs (`uses: actions/setup-node@60edb5...`) rather than mutable tags (`@v4`) to protect against supply chain attacks.

### **198. What is Hermetic Container Building with Kaniko?**
**Answer:** Executing container image builds inside an isolated pod with no host Docker socket mount, pushing directly to an OCI registry.

### **199. What is a Pipeline Flakiness Rate?**
**Answer:** The percentage of pipeline runs that fail due to unstable tests or network timeouts rather than actual code defects, eroding developer trust.

### **200. What is Progressive Delivery Automated Metric Gating?**
**Answer:** Evaluating real-time Prometheus golden signal telemetry during canary deployments to autonomously promote or abort releases without human intervention.