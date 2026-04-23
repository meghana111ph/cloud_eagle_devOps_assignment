# cloud_eagle_devOps_assignment

## CI/CD Pipeline Design (Part 1)

This project implements a fully automated CI/CD pipeline for a Spring Boot service (`sync-service`) deployed to GCP VMs.

### Branching Strategy
- **main**: Deploys to **Production**.
- **release/***: Deploys to **Staging**.
- **develop**: Deploys to **QA**.
- **Others**: CI only (build & test).

### Key Features
- **Rolling Updates**: Zero-downtime deployments via GCP Managed Instance Groups.
- **Automated Rollbacks**: Automatic reversion to the last stable tag if smoke tests fail.
- **Secret Management**: Secure injection of MongoDB credentials via Jenkins & GCP Metadata.
- **Approval Gates**: Production deployments require manual sign-off from designated leads.

---
*For full technical details, see [DESIGN_DOC.md](./DESIGN_DOC.md)*

## Infrastructure Design (Part 2)

A proposed modern infrastructure setup for scaling and cost-efficiency.

### Architecture Highlights
- **Compute**: Cloud Run (Serverless) for automatic scaling and cost-to-zero.
- **Database**: MongoDB Atlas for managed high-availability.
- **Security**: Cloud Armor (WAF) and Secret Manager.
- **Networking**: Private VPC connectivity.

---
*For full infrastructure details, see [INFRA_DESIGN.md](./INFRA_DESIGN.md)*