# MLOps Engineer Guide

**Owns**: `Dockerfile`, `docker-compose.yml`, CI/CD configuration
**Deliverable**: a live, publicly accessible deployment.

[← Back to main README](../../README.md)

---

## Objective

Take the backend, model, and frontend from "runs on my machine" to a live, shareable deployment with basic reliability and observability — the piece that turns this from a local project into something you can put a URL on your resume.

## Scope

- Containerization
- Deployment to a hosting platform
- Logging and monitoring
- Basic CI (optional but recommended)

## Responsibilities

### 1. Containerization
- Write a `Dockerfile` for the backend that bundles the FastAPI app and the exported model
- Keep the image lean — use a slim Python base image, avoid bundling training dependencies (PyTorch training extras) that aren't needed at inference time
- Use `docker-compose.yml` to wire up backend + Redis (if used for queue/cache) for local integration testing

### 2. Deployment
- Choose a free/low-cost host suited to ML demos: Render, Railway, or Hugging Face Spaces (Spaces is particularly well-suited for ML demo apps)
- Configure environment variables (Sentinel Hub credentials) securely via the platform's secrets manager — never in the image
- Deploy the frontend separately (Vercel/Netlify for static hosting) or serve it from the same backend if kept simple

### 3. Monitoring & logging
- Log request count, latency, and error rate at minimum
- Expose `/health` for platform readiness/liveness checks
- Consider a lightweight uptime check (e.g. UptimeRobot free tier) so you know if the demo goes down

### 4. CI (optional, recommended for portfolio credibility)
- GitHub Actions workflow: run tests on push, build the Docker image, optionally auto-deploy on merge to `main`
- Keep it simple — lint + test + build is enough to demonstrate the practice

## File structure

```
├── Dockerfile
├── docker-compose.yml
├── .github/
│   └── workflows/
│       └── ci.yml
└── .env.example
```

## Interfaces consumed

| Provider | Interface used |
|---|---|
| Backend Engineer | containerized FastAPI app, `/health` endpoint |
| Frontend Engineer | static build artifact for hosting |

## Definition of done

- [ ] Backend runs correctly inside Docker locally before deploying
- [ ] Live URL is publicly reachable and returns correct predictions
- [ ] Secrets are not committed anywhere in the repo or image
- [ ] Basic logging/monitoring is in place
- [ ] README is updated with the live demo link

## Suggested tools

`Docker`, `docker-compose`, `Render` / `Railway` / `Hugging Face Spaces`, `GitHub Actions`, `UptimeRobot` (optional)
