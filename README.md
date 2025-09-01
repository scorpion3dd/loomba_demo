# PROJECT - LOOMBA ACS: AUTOMATED DOCKER PROJECT MANAGEMENT SYSTEM

## STATUS AND CONTACTS

<table>
    <tr>
        <th>
            <p>This project is under active support.</p>
            <p>Actual version: 1.1.1</p>
            <p>The repository is available in read/write mode.</p>
            <h3>(c) Denis Puzik <b>scorpion3dd@gmail.com</b></h3>
            [docs](https://myerp.com.ua)
        </th>
        <th>
            <img src="/images/3ds.jpg" alt="3ds Logo">
        </th>
        <th>
            <img src="/images/proxy.png" alt="Proxy Logo" height="100px">
        </th>
        <th>
            <img src="/images/loomba.png" alt="Loomba Logo" height="150px">
        </th>
  </tr>
</table>

---
---

# INTRODUCTION

## BRIEF DEFINITION

**Loomba** — is a cross-platform (Linux/Windows) automated Docker project management system for the full lifecycle of web systems:  
**Dev → CI / CD → Staging → Prod.**  
It consists of a set of coordinated scripts, templates, and CLI wrappers that transform infrastructure operations into declarative and reproducible processes.

## VALUE FOR THE TEAM

- **Environment Standardization** — all environments (Dev, Test, Preprod, Prod) are built from the same declarative files.
- **Repeatability and Traceability** — all actions are codified (IaC/EaC); any environment can be reproduced from Git.
- **Infrastructure Transparency** — centralized reverse-proxy, clear service structure, built-in metrics and logging.
- **Performance** — maximum automation of routine operations (deployment, testing, updates, monitoring).
- **Security** — automatic generation and installation of SSL certificates at the RP level; all sites run on HTTPS/2.
- **Flexible Access Control** — rights segregation by subdomains, user roles, and networks.
- **Rapid Onboarding** — a new developer launches the project with a standard command and starts working immediately.
- **Low Operating Cost** — unified scripts, built-in health-checks, diagnostics.
- **Error Reduction and Predictable Delivery Times** — via automatic checks, standards, and process transparency.

## GOALS

* Manage multiple Docker projects on a single host/cluster through a unified reverse-proxy.
* Support standard stacks (PHP/Node.js, MySQL/MariaDB, PostgreSQL, Redis, Elasticsearch, MailHog).
* Unified CLI (`stop_all`, `restart_all`, `rebuild_selected`, `check_server`, etc.).
* Built-in service projects: Portainer, SonarQube, Jenkins, JMeter, Netdata.
* Cross-platform: Linux (bash/Make) and Windows (PowerShell/Make/WSL2).

**Not included in scope:**
* Full Kubernetes operator (integration possible via adapters).
* Secrets storage “as-is” (integration with external secret stores available).

---
---

# GETTING STARTED

## SYSTEM REQUIREMENTS

1. **Docker**
2. **Docker Compose** — `docker-compose.yml` is used for building images and running containers.
3. **Service projects** — proxy, portainer, sonar, jenkins, jmeter, netdata.

## INSTALLATION

1. Clone the project repository:
```bash
git clone https://github.com/scorpion3dd/loomba_demo.git
```


# ARCHITECTURE OVERVIEW

## COMPONENTS

* **CLI** — thin wrapper over Docker Compose/Builds/Make/PowerShell.
* **Reverse Proxy** — centralized Nginx, TLS, SNI routing, domain → container mapping.
* **Service Projects** — Portainer, SonarQube, Jenkins, JMeter, Netdata.
* **Configuration** — conf.yml, .env, Compose profiles.
* **Observability** — health-checks, metrics, logs, dashboards.

## NETWORK ORGANIZATION

* Unified external network gateway for RP and projects.
* Service isolation via internal networks (project_net).
* TLS support: dev (mkcert), staging/prod (Let’s Encrypt/Certbot).

# LIFECYCLE: Dev → CI / CD → Staging → Prod

## 1. DEV (DEVELOPMENT)

**DEV environment** is intended for **rapid development**, local debugging, and ensuring full environment 
consistency within the team.

### Workflow
1. Developer clones the ERP repository.
2. Configures the environment (select PHP version, database, web server).
3. Runs `start` — environment automatically spins up (PHP + DB + Redis, etc.) in Docker containers.
4. Uses built-in services for testing and debugging.

### Built-in DEV Services
- 📧 **MailHog** — test outgoing ERP emails without external SMTP.
- 🐞 **Xdebug** — PHP code debugging (PHPStorm integration).
- 🗄 **PhpMyAdmin / Adminer** — database management.
- 📦 **Redis UI**  — cache inspection.

### Team Benefits
* Unified, reproducible environment for all developers.
* Eliminates “works on my machine” issues.
* Accelerated onboarding of new team members.
* Minimizes manual errors and configuration differences.

## 2. CI / CD (CONTINUOUS INTEGRATION & DELIVERY)

**CI/CD environment** automates code building, testing, and delivery, ensuring predictability between 
DEV and subsequent environments.

### Workflow
1. Environment configuration is stored in Git.
2. Jenkins CI/CD server deploys an identical test environment.
3. Jenkins Pipeline performs:
    - 📦 Project build.
    - 🐳 Container startup (`start --skip-build`).
    - ✅ Test execution:
        - **Unit tests** (PHPUnit).
        - **Static code analysis** (PHPStan, Psalm).
    - 📊 Artifact generation:
        - Docker images.
        - Code quality reports (SonarQube).

### Team Benefits
* Maximum alignment of CI environment with DEV.
* Reliable tests and predictable results.
* Automatic defect detection, reducing risk of errors reaching Staging and Prod.
* Standardized code delivery process.

## 3. STAGING (PRE-RELEASE / ACCEPTANCE TESTING)

**STAGING environment** is used for **final product quality checks** before production.
Both automated and manual acceptance tests are performed.

### Workflow
1. Jenkins takes the built Docker ERP image and deploys it on the staging server.
2. Full reproduction of PROD-equivalent environment.
3. QA management capabilities:
    - Deploy staging with different PHP or DB versions.
    - Import a copy of any database version.
    - Enable SSL (via **mkcert** or **Traefik**) to test ERP under near-production conditions.
4. Comprehensive testing:
    - ✅ Functional tests.
    - 🚀 Performance/load tests (**Apache JMeter**).
    - 🔥 Smoke tests.
    - ♻️ Regression tests.
    - 🎨 UI tests (**Selenium**).
    - 🔐 Security tests (**OWASP ZAP**).
    - 📊 Reports in **SonarQube**.
5. Acceptance testing by stakeholders.
6. Client demo.

### Team Benefits
* Full confidence that PROD deployment will behave predictably.
* Identification of integration bugs and vulnerabilities before PROD.
* Environment accessible to QA, analysts, and business stakeholders.
* Reduced risk of incidents and failures in production.
* Client demonstration before release.


## 4. PROD (PRODUCTION)

**PROD environment** is the main working environment where the application is used by end-users.
Critical factors: stability, predictability, and security.

### Workflow
1. **Artifact promotion** — only images that passed STAGING are allowed in PROD.
2. **Zero-downtime deployment** via reverse-proxy (containers update without downtime).
3. **Database migrations** executed automatically with rollback on failure.
4. **Automatic TLS certificate management** (Let’s Encrypt / Certbot).
5. **Backup and DR**: regular backups of databases, configurations, and artifacts.

### Monitoring and Security
- 📊 **Netdata** — performance dashboards.
- 🔎 **SonarQube** — code quality analysis.
- 🛠 **Portainer** — container and stack management.
- ⚙ **Jenkins** — CI/CD automation.
- 🔔 Alerts on failures and threshold breaches.
- 🔐 Resource limitations and access control for containers.

### Business Benefits
- **Reliability** — minimize downtime and operational risk.
- **Transparency** — all changes go through CI/CD and are tracked in Git.
- **Confidence in Quality** — PROD uses verified artifacts.
- **SLA Compliance** — predictable application performance.
- **Scalability Ready** — unified service management approach.


# CLI COMMAND EXAMPLES

## LINUX/MACOS

```bash
loomba up --env dev
loomba restart_selected myerp3:api myerp3:web
loomba rebuild_all --no-cache
loomba check_server
loomba status --wide
```

## WINDOWS POWERSHELL

```powershell
./loomba.ps1 up --env dev
./loomba.ps1 restart_selected myerp3:api myerp3:web
./loomba.ps1 rebuild_all --no-cache
./loomba.ps1 check_server
./loomba.ps1 status --wide
```


# SECURITY AND MONITORING

* Automatic TLS certificate updates.
* Portainer, Netdata, SonarQube, and Jenkins for observability and CI/CD.
* Container resource limits, centralized logs, and alerts.


# SUMMARY

**Loomba ACS** is a cross-platform automated Docker project management system that:
* standardizes and makes environments transparent,
* minimizes manual errors,
* accelerates product delivery and reduces DevOps/QA/developer workload.


# LICENSING

**Loomba ACS** is distributed under a **proprietary (closed) license**.
Usage, copying, and modification are **only permitted under purchased licenses**.

## TERMS OF USE
- 📜 The right to install and operate is granted only with a valid license agreement.
- 🔒 Unauthorized distribution, publication, or reverse engineering is prohibited.
- ⚖️ All rights to code, documentation, and project materials belong to the author:
  **(c) Denis Puzik <scorpion3dd@gmail.com>**.

## COMMERCIAL MODEL
- 💼 **License provided on a paid basis**:
    - Individual licenses for developer teams.
    - Corporate licenses with extended support.
- 📦 Updates and security patches available only to licensed users.

## UPDATES AND SECURITY PATCHES AVAILABLE ONLY TO LICENSED USERS
- Official project repository:  
  👉 [Bitbucket: docker-gateway-proxy](https://bitbucket.org/3dscorpion7/docker-gateway-proxy)
- Documentation and materials:
  👉 [docs](https://myerp.com.ua)

## DOCUMENTATION AND MATERIALS
For pricing and license options, contact:
✉️ **scorpion3dd@gmail.com**  
