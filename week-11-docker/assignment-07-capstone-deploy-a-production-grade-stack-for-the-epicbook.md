# Assignment 7 — Capstone: Deploy a Production-Grade Stack for The EpicBook

Part of the DevOps Micro Internship (DMI) with Agentic AI

---

## Purpose

In this assignment, you will deploy the EpicBook application as a production-oriented Docker Compose stack on a cloud VM. You will use optimized container images, isolated networks, health checks, persistent MySQL storage, a selected reverse proxy, logging, backup and restore testing, and reliability procedures.

---

# Task 0 — App Discovery and Architecture

## Goal

Review the EpicBook repository and design the intended application architecture.

### Evidence

#### Screenshot 1 — EpicBook Project Structure

Add a terminal screenshot showing the EpicBook project structure after cloning the repository.

![Screenshot](screenshots/Ass7.Task0.ss1.png)

---

#### Screenshot 2 — Architecture Diagram

Add a screenshot of your architecture diagram showing:

- Public user
- Reverse proxy
- Frontend
- Backend
- Database
- Docker networks
- Public and private ports
- Persistent database storage

Add your full name inside the diagram or as a clear caption below it.

![Screenshot](screenshots/Ass7.Task0.ss2.png)

---

#### Screenshot 3 — Environment Variables and Ports Document

Add a screenshot showing the contents of:

```text
docs/02-env-and-ports.md
```

It must document environment-variable names, internal ports, persistent-data details, and the health-check method. Do not expose real credentials or values.

![Screenshot](screenshots/Ass7.Task0.ss3.png)

---

# Task 1 — Create Production Docker Images

## Goal

Create optimized production images for the EpicBook backend and frontend.

### Evidence

#### Screenshot 4 — Backend Dockerfile

Add a screenshot showing `backend/Dockerfile`, including:

- Dependency stage
- Minimal runtime stage
- Production startup command
- Internal backend port
- Non-root user configuration

![Screenshot](screenshots/Ass7.Task1.ss4.png)

---

#### Screenshot 5 — Frontend Dockerfile

Add a screenshot showing `frontend/Dockerfile`, including:

- Nginx runtime image
- Static frontend files copied to the Nginx web root

![Screenshot](screenshots/Ass7.Task1.ss5.png)
---

#### Screenshot 6 — Docker Ignore Files

Add a screenshot showing both:

```text
backend/.dockerignore
frontend/.dockerignore
```

![Screenshot](screenshots/Ass7.Task1.ss6.png)

---

#### Screenshot 7 — Docker Image Builds and Size Comparison

Add a terminal screenshot showing successful builds of:

- Baseline backend image
- Optimized backend image
- Frontend image

The screenshot must also show the baseline and optimized backend image-size comparison.

![Screenshot](screenshots/Ass7.Task1.ss7.png)

---

#### Screenshot 8 — Backend Running as Non-Root User

Add a terminal screenshot showing the optimized backend container running as a non-root user.

![Screenshot](screenshots/Ass7.Task1.ss8.png)

---

### Notes

Write a short note covering:

- Baseline and optimized backend image sizes
- The image-size reduction achieved
- One Docker layer-caching optimization used
- The security benefit of running the backend as a non-root user

The baseline backend image was **1.73 GB**, while the optimized multi-stage backend image was **274 MB**, achieving an approximate **84.2% reduction** in image size.

One layer-caching optimization used was copying `package*.json` and installing production dependencies before copying the application source code. This allows Docker to reuse the dependency layer when only application files change, reducing rebuild time.

The optimized backend also runs as the non-root `node` user (`uid=1000`), which reduces the security impact of a potential container compromise by limiting the privileges available inside the container.

---

# Task 2 — Create the Docker Compose Stack and Networks

## Goal

Create one Docker Compose stack containing the reverse proxy, frontend, backend, and MySQL database.

### Evidence

#### Screenshot 9 — Docker Compose Services

Add a screenshot showing `docker-compose.yml` with all four services:

```text
reverse-proxy
frontend
backend
database
```

![Screenshot](screenshots/Ass7.Task2.ss9.png)

---

#### Screenshot 10 — Networks and Named Volume

Add a screenshot showing:

- `front-tier` network
- `back-tier` network
- `db_data` named volume

![Screenshot](screenshots/Ass7.Task2.ss10.png)

---

#### Screenshot 11 — Docker Compose Validation

Add a terminal screenshot showing successful Docker Compose validation without exposing environment-variable values or secrets.

![Screenshot](screenshots/Ass7.Task2.ss11.png)

---

# Task 3 — Configure Health Checks and Startup Dependencies

## Goal

Configure health checks and ensure services start only after their dependencies are healthy.

### Evidence

#### Screenshot 12 — Backend Health Endpoint

Add a screenshot showing the backend application configuration for the `/health` endpoint.

![Screenshot](screenshots/Ass7.Task3.ss12.png)

---

#### Screenshot 13 — MySQL and Backend Health Checks

Add a screenshot showing `docker-compose.yml` with health checks for MySQL and the backend.

![Screenshot](screenshots/Ass7.Task3.ss13.png)

---

#### Screenshot 14 — Frontend and Reverse-Proxy Health Checks

Add a screenshot showing:

- Frontend health check
- Reverse-proxy health check
- `depends_on` conditions using `service_healthy`

![Screenshot](screenshots/Ass7.Task3.ss14.png)

---

#### Screenshot 15 — Running Healthy Services

Add a terminal screenshot showing Docker Compose service status. The database, backend, frontend, and reverse proxy must be running successfully.

![Screenshot](screenshots/Ass7.Task3.ss15.png)

---

#### Screenshot 16 — Public Health Endpoint

Add a terminal screenshot showing a successful response from the public application health endpoint through the reverse proxy.

![Screenshot](screenshots/Ass7.Task3.ss16.png)

---

#### Screenshot 17 — Health-Check and Startup-Order Document

Add a screenshot showing the contents of:

```text
docs/03-healthchecks-and-depends-on.md
```

Explain the health-check method for each service and the startup dependency order.

![Screenshot](screenshots/Ass7.Task3.ss17.png)

![Screenshot](screenshots/Ass7.Task3.ss17i.png)

---

# Task 4 — Configure the Reverse Proxy and Same-Origin Routing

## Goal

Use either Nginx or Traefik as the only public entry point for the EpicBook application.

### Evidence

#### Screenshot 18 — Selected Reverse-Proxy Configuration

Add a screenshot showing the configuration for your selected reverse proxy.

It must show routes for:

- Static frontend assets
- Application pages
- API requests
- Health endpoint

![Screenshot](screenshots/Ass7.Task4.ss18.png)

---

#### Screenshot 19 — Only Reverse Proxy Publishes Port 80

Add a screenshot of `docker-compose.yml` showing that only the `reverse-proxy` service publishes port 80.

![Screenshot](screenshots/Ass7.Task4.ss19.png)

---

#### Screenshot 20 — Reverse-Proxy Route Testing

Add a terminal screenshot showing successful requests through the selected reverse proxy to:

- Application page
- One API endpoint
- One static asset
- Health endpoint

![Screenshot](screenshots/Ass7.Task4.ss20.png)

---

#### Screenshot 21 — EpicBook Application Through Public IP

Add a browser screenshot showing the EpicBook application loaded through the VM public IP address.

Add your full name as a clear caption below the screenshot.

![Screenshot](screenshots/Ass7.Task4.ss21.jpg)

---

#### Screenshot 22 — Proxy Routing and CORS Document

Add a screenshot showing the contents of:

```text
docs/04-proxy-routing-and-cors.md
```

Explain the proxy routes and state whether CORS was required and why.

![Screenshot](screenshots/Ass7.Task4.ss22.png)

![Screenshot](screenshots/Ass7.Task4.ss22i.png)

---

# Task 5 — Prove Data Persistence, Backup, and Restore

## Goal

Verify MySQL persistence and perform a controlled backup and restore drill.

### Evidence

#### Screenshot 23 — MySQL Volume Configuration

Add a terminal screenshot showing the `db_data` named volume and its MySQL mount configuration.

![Screenshot](screenshots/Ass7.Task5.ss23.png)

---

#### Screenshot 24 — Test Data Before Backup

Add a terminal screenshot showing the selected test data before the backup and restore drill.

![Screenshot](screenshots/Ass7.Task5.ss24.png)

---

#### Screenshot 25 — Successful Backup Creation

Add a terminal screenshot showing successful backup creation and the backup file stored in the host backup directory.

![Screenshot](screenshots/Ass7.Task5.ss25.png)

---

#### Screenshot 26 — Controlled Data-Loss Test

Add a terminal screenshot showing that the selected test record was removed during the controlled data-loss test.

![Screenshot](screenshots/Ass7.Task5.ss26.png)

---

#### Screenshot 27 — Restore Verification

Add a terminal screenshot showing successful restore and verification that the deleted test record is available again.

![Screenshot](screenshots/Ass7.Task5.ss27.png)

---

#### Screenshot 28 — Persistence After Down/Up Cycle

Add a terminal screenshot showing that database data remains available after a non-destructive Docker Compose down/up cycle.

Do not use `docker compose down -v`.

![Screenshot](screenshots/Ass7.Task5.ss28.png)

---

#### Screenshot 29 — Persistence and Backup Document

Add a screenshot showing the contents of:

```text
docs/05-persistence-and-backup.md
```

Include the backup plan and restore procedure.

![Screenshot](screenshots/Ass7.Task5.ss29.png)

![Screenshot](screenshots/Ass7.Task5.ss29i.png)

---

# Task 6 — Configure Logging and Observability

## Goal

Configure useful reverse-proxy and backend logs without exposing sensitive information.

### Evidence

#### Screenshot 30 — Logging Configuration

Add a screenshot showing:

- Configuration for the selected reverse proxy
- Proxy log format
- Docker Compose host log-directory bind mount

![Screenshot](screenshots/Ass7.Task6.ss30.png)

---

#### Screenshot 31 — Persistent Proxy Logs and Backend Logs

Add a terminal screenshot showing:

- Selected reverse-proxy logs available from the host directory after a proxy restart
- Backend logs displayed through Docker Compose

![Screenshot](screenshots/Ass7.Task6.ss31.png)

---

### Notes

Write a short note covering:

- The selected reverse proxy
- Host path used for reverse-proxy logs
- How backend logs are viewed
- Whether JSON or standard text logs were used
- Why passwords, tokens, headers, and database connection strings must not appear in logs

Nginx was selected as the reverse proxy. Its logs are persisted on the VM host in `logs/proxy/`, which is bind-mounted to `/var/log/nginx` inside the proxy container. Backend logs are written to container stdout/stderr and can be viewed using `docker compose logs backend`. Standard timestamped text logs were used instead of JSON. Passwords, tokens, authentication headers, and database connection strings must not appear in logs because logs can be accessed by administrators, monitoring systems, or other users and could expose sensitive credentials or enable unauthorized access.

---

# Task 7 — Deploy and Verify the Stack on a Cloud VM

## Goal

Deploy the completed Docker Compose stack on an AWS or Azure VM and verify public access.

### Evidence

#### Screenshot 32 — VM Public IP and Inbound Rules

Add a cloud-console screenshot showing:

- VM public IP address
- SSH port 22 restricted to your IP address
- HTTP port 80 allowed from Anywhere

![Screenshot](screenshots/Ass7.Task7.ss32.png)

---

#### Screenshot 33 — Cloud VM Stack Verification

Add a VM terminal screenshot showing:

- Docker Compose service status
- Successful public health or API response
- No published database, frontend, or backend ports

![Screenshot](screenshots/Ass7.Task7.ss33.png)

---

#### Screenshot 34 — EpicBook Application on Cloud VM

Add a browser screenshot showing the EpicBook application loaded through the VM public IP address.

Add your full name as a clear caption below the screenshot.

![Screenshot](screenshots/Ass7.Task7.ss34.png)

---

### Notes

Write a short note covering:

- Cloud provider used
- VM operating system
- Public port exposed
- Security rules configured
- Confirmation that the application and backend API worked through the reverse proxy

## Cloud VM Deployment Notes

- **Cloud provider:** AWS
- **Region:** eu-north-1
- **VM public IP:** 13.60.231.88
- **Public port:** 80
- **Application URL:** http://13.60.231.88
- **SSH:** Port 22 restricted to the administrator's IP address
- **HTTP:** Port 80 allowed from Anywhere
- **Database port:** 3306 remains private
- **Backend port:** 8080 remains private
- **Frontend port:** 80 is not published directly to the host
- **Reverse proxy:** Nginx is the only service publishing host port 80

All four Docker Compose services—database, backend, frontend, and reverse proxy—were verified as healthy. The public `/health` and `/api/cart` endpoints returned HTTP 200, and the EpicBook application loaded successfully through the VM public IP.

A non-destructive `docker compose down` followed by `docker compose up -d` was performed. The stack restarted successfully, the `db_data` Docker volume remained present, and the public health and API endpoints continued to return HTTP 200.
---

# Task 8 — Automate Deployment with CI/CD (Optional)

## Goal

Optionally automate image build, image push, and deployment through GitHub Actions or Azure Pipelines.

### Optional Evidence

#### Optional Screenshot — Successful CI/CD Pipeline Run

Add a screenshot showing a successful pipeline run with build, image push, deployment, and verification stages.

Add your screenshot here.

---

### Optional Notes

Write a short note covering:

- CI/CD platform used
- Image-tagging method
- Registry used
- Deployment trigger
- Manual approval or secret-handling approach

Write your note here.

---

# Task 9 — Perform Reliability Tests and Create an Operations Runbook

## Goal

Test controlled service failures and document safe operating procedures.

### Evidence

#### Screenshot 35 — Backend Failure and Recovery

Add a terminal screenshot showing:

- Backend failure test
- Expected unavailable response through the reverse proxy
- Backend restart
- Successful health-check recovery

![Screenshot](screenshots/Ass7.Task9.ss35.png)

---

#### Screenshot 36 — Database Failure and Recovery

Add a terminal screenshot showing:

- Database outage test
- Failed database-dependent request
- Database restart
- Successful application recovery

![Screenshot](screenshots/Ass7.Task9.ss36.png)

---

### Notes

Write a short operations runbook covering:

- Safe restart procedure for reverse proxy, frontend, backend, and database
- Backup and restore procedure
- Secret-rotation approach
- Database recovery procedure
- What to check when the application returns an error
- Results of backend and database reliability tests

## Reliability Tests and Operations Runbook

## Reliability Testing

Two controlled service-failure tests were performed on the deployed EpicBook Docker Compose stack.

### Backend Failure Test

The `epicbook-backend` container was stopped without removing it. While the backend was unavailable, the public `/health` request through the Nginx reverse proxy returned HTTP 504.

The backend container was then started again. It returned to a healthy state, and the public `/health` endpoint returned HTTP 200.

### Database Failure Test

The `epicbook-database` container was stopped without removing the container or the `db_data` named volume. While MySQL was unavailable, the database-dependent `/health` request returned HTTP 503.

The database container was started again and returned to a healthy state. The backend also returned to a healthy state. The public `/health` endpoint then returned HTTP 200, and `/api/cart` returned HTTP 200.

## Data Protection

No named volumes were removed during testing. `docker compose down -v` was not used. No database tables or application records were deleted, and no internal service ports were exposed.

The `db_data` volume remained present at:

`/var/lib/docker/volumes/db_data/_data`

## Operations Runbook

The operations runbook documents:

- Normal stack startup
- Non-destructive stack shutdown
- Service health checks
- Viewing service and proxy logs
- Safe individual service restarts
- Database backup
- Database restore
- High-level secret rotation
- Post-incident verification
- Reliability and data-protection principles

## Final Verification

After both reliability tests:

- All four Docker Compose services were healthy.
- `/health` returned HTTP 200.
- `/api/cart` returned HTTP 200.
- The `db_data` named volume remained intact.
- The EpicBook application was operational.

Task 9 reliability testing and operations documentation were completed successfully.

---

# Final Public Application URL

**EpicBook URL:** http://13.60.231.88/

Replace the placeholder with your working public URL.

---

# GitHub Repository URL

**Your Fork or Repository URL:** https://github.com/Tonia-onyeka/theepicbook.git
---

# LinkedIn Requirement

## Goal

Create a professional LinkedIn post of 6–10 lines about your EpicBook capstone deployment.

Your post must include:

- The architectural decision that most improved reliability
- Your biggest image-size reduction, with numbers
- Key production-hardening lessons
- A deployment verification image

### Evidence

**LinkedIn Post URL:** https://www.linkedin.com/posts/anthonia-akwuohia-5b00681b0_devops-docker-aws-share-7511815805773271040-AoIJ/?utm_source=share&utm_medium=member_desktop&rcm=ACoAADEhX1QBTHiW-kQPmKjn3MVixQzj4IzJO1Q

#### LinkedIn Post Screenshot

![Screenshot](screenshots/Ass7LinkedIn.png)

---

# Submission Checklist

- [ ] EpicBook repository reviewed and architecture diagram created
- [ ] Environment variables, ports, persistence, and health-check details documented
- [ ] Backend and frontend production Dockerfiles created
- [ ] Backend runs as a non-root user
- [ ] Docker image-size comparison completed
- [ ] Docker Compose stack includes reverse proxy, frontend, backend, and database
- [ ] `front-tier` and `back-tier` networks configured
- [ ] `db_data` named volume configured
- [ ] MySQL, backend, frontend, and reverse-proxy health checks configured
- [ ] Startup dependencies use `service_healthy`
- [ ] Nginx or Traefik selected as the only public reverse proxy
- [ ] Only reverse-proxy port 80 is publicly published
- [ ] Same-origin routing configured and CORS used only when required
- [ ] Backup, restore, and persistence testing completed
- [ ] Reverse-proxy and backend logs verified
- [ ] Cloud VM deployment verified through the public IP
- [ ] Backend and database reliability tests completed
- [ ] Screenshots 1–36 included
- [ ] Required notes completed
- [ ] LinkedIn post URL and screenshot included
- [ ] Full name visible in required screenshots or captions
- [ ] No passwords, tokens, private keys, account IDs, or other sensitive information exposed

---

## 📌 About DMI & CloudAdvisory

DevOps Micro Internship (DMI) is a project-based DevOps program run by Pravin Mishra (The CloudAdvisory) focused on real-world execution, systems thinking, and career readiness.

It helps learners build strong DevOps foundations with hands-on experience.

---

## 📌 Resources

- 🌐 DMI Official Website: https://dmi.pravinmishra.com?utm_source=github&utm_medium=readme  
- 🎓 University: https://university.pravinmishra.com?utm_source=github&utm_medium=readme  
- 💬 Discord Community: https://discord.pravinmishra.com?utm_source=github&utm_medium=readme  
- 📝 Blog: https://dmi.pravinmishra.com/blog?utm_source=github&utm_medium=readme  
- ▶️ YouTube Playlist: https://www.youtube.com/playlist?list=PLFeSNDtI4Cho  
- 🔗 Pravin Mishra (LinkedIn): https://www.linkedin.com/in/pravin-mishra-aws-trainer/  
- 🏢 CloudAdvisory (LinkedIn): https://www.linkedin.com/company/thecloudadvisory/

---

*This submission is part of DevOps Micro Internship (DMI) — Agentic AI Track.*
