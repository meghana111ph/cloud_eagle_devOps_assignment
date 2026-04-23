# sync-service CI/CD Pipeline Design

## 1. Branching & Environment Strategy
To ensure stability and a clear path to production, we follow a **GitFlow-inspired** branching model:

| Branch | Environment | Purpose |
| :--- | :--- | :--- |
| `main` | **Production** | Stable, production-ready code. |
| `release/*` | **Staging** | Pre-production testing and final QA. |
| `develop` | **QA** | Integration branch for features. |
| `feature/*` / `PR` | **CI-Only** | Development work; runs tests but **no deployment**. |

*   **Accidental Deployment Prevention:** 
    *   **Logic-based restriction:** The Jenkinsfile resolves the `TARGET_ENV` based on the branch name. If the branch doesn't match `main`, `release/`, or `develop`, it defaults to `ci-only`.
    *   **Approval Gates:** Deployment to `prod` (from `main`) requires a manual approval from the `devops-leads` or `release-managers` group via a Jenkins Input step.

## 2. Jenkins Pipeline Architecture
The pipeline is designed as a linear sequence of stages with built-in safety checks:

*   **Continuous Integration (PR/Merge):** 
    *   Every commit triggers `Build & Unit Test` and `Code Quality (SonarQube)`.
    *   PRs stop after Quality checks (they are not packaged or deployed).
*   **Continuous Delivery:**
    *   Merged code is packaged into a Docker image and pushed to **GCP Artifact Registry**.
*   **Rollback Strategy:**
    *   **Automatic:** If the `Smoke Test` fails immediately after deployment, the pipeline automatically fetches the `last_stable_tag` from a GCS bucket and triggers a redeploy.
    *   **Manual:** A `ROLLBACK` parameter allows engineers to manually trigger a redeploy of a specific known-good tag.

## 3. Configuration & Secrets Management
*   **Environment Configs:** We use **Spring Profiles** (`qa`, `staging`, `prod`). The pipeline injects the correct profile into the instance metadata, which the application reads at runtime.
*   **Secrets Handling (MongoDB & API Keys):** 
    *   Secrets are stored in the **Jenkins Credentials Store**.
    *   During the `Deploy` stage, the pipeline pulls the `MONGO_URI` and injects it securely into the GCP Instance Template as an environment variable (`SPRING_DATA_MONGODB_URI`).
    *   This ensures that no sensitive credentials are ever committed to the source code.

## 4. Deployment Strategy: Rolling Update
We have chosen a **Rolling Update** strategy via GCP Managed Instance Groups (MIGs).

*   **Why Rolling Update (Justification):**
    1.  **Zero Downtime:** By setting `max-unavailable=0`, we ensure the current capacity is maintained until new instances are healthy.
    2.  **Cost Efficiency:** Unlike Blue/Green, which requires 200% resource allocation during deployment, Rolling Updates only requires a small "surge" (e.g., 1 extra instance), significantly reducing cloud costs.
    3.  **Automation:** Native integration with GCP health checks allows the MIG to automatically detect a failed rollout and halt the update.
*   **Downtime Approach:** We achieve **minimal-to-zero downtime** by using a `max-surge=1` and `max-unavailable=0` configuration, ensuring users never experience service interruption.
