# Backend Engineer Guide

**Owns**: `src/api/`
**Deliverable**: a working, testable inference API.

[← Back to main README](../../README.md)

---

## Objective

Expose the trained model as a service that accepts a location and date, fetches the corresponding live satellite tile, runs inference, and returns a segmentation mask — fast enough for an interactive frontend.

## Scope

- FastAPI application and routing
- Async handling for slow inference on large tiles
- Caching to avoid redundant work
- Input validation and error handling

## Responsibilities

### 1. API design
- `POST /predict` — accepts `{ lat, lon, date }`, returns a mask (as base64 PNG or GeoTIFF) plus metadata (tile date, cloud %, inference latency)
- `GET /health` — basic liveness check for deployment monitoring
- Validate inputs: reject out-of-range coordinates, malformed dates, and dates with no available imagery, with clear error messages

### 2. Model integration
- Load the exported ONNX/TorchScript model once at startup, not per-request
- Use the ML Engineer's `infer.py` interface as the integration contract — the backend should not need to know about training internals

### 3. Async processing
- Large-tile inference can take several seconds; use FastAPI background tasks or Celery + Redis so requests don't block
- Return a job ID immediately for long-running requests, with a `GET /jobs/{id}` polling endpoint, or use WebSockets if the frontend needs push updates

### 4. Caching
- Cache by `(tile_id, date, model_version)` — identical requests should not re-fetch or re-infer
- Redis or a simple on-disk cache is sufficient at this scale

### 5. Reliability
- Handle Sentinel Hub API failures gracefully (rate limits, no imagery available for requested date/cloud cover too high)
- Log request volume, latency, and error rate

## File structure

```
src/api/
├── main.py          # FastAPI app entrypoint
├── routes.py         # /predict, /health, /jobs endpoints
├── queue.py           # async job handling (Celery/background tasks)
├── cache.py            # request/result caching
└── schemas.py           # Pydantic request/response models
```

## Interfaces provided to other roles

| Consumer | Interface |
|---|---|
| Frontend Engineer | REST API — `POST /predict`, `GET /jobs/{id}`, `GET /health` |
| MLOps Engineer | Dockerized service exposing a configurable port, `/health` for readiness checks |

## Definition of done

- [ ] `/predict` returns a correct mask for a known test location, verified against the ML Engineer's notebook output
- [ ] Invalid inputs return clear 4xx errors, not stack traces
- [ ] Long-running inference doesn't block the event loop
- [ ] Repeated identical requests are served from cache
- [ ] API is documented via FastAPI's auto-generated OpenAPI docs (`/docs`)

## Suggested tools

`fastapi`, `uvicorn`, `pydantic`, `celery` + `redis` (or `fastapi.BackgroundTasks` for a simpler start), `onnxruntime`
