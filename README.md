# FastAPI Parallel Fibonacci Backend

A small FastAPI service that accepts batches of integers, dispatches the calculation to a Python process pool, and exposes endpoints for polling job status and retrieving completed results.

> **Implementation note:** the current worker calculates `F(n % 35)` for each submitted integer. This keeps each individual calculation bounded; it is a multiprocessing demo, not an unrestricted large-integer Fibonacci service.

## Features

- Submit a batch of integers as a background job.
- Execute the batch in a `ProcessPoolExecutor`.
- Poll a job's status and fetch its result.
- Check service readiness and the configured worker count.
- Optional HTML/JavaScript frontend (if present in `frontend/`).

## Project structure

```text
fibonacci-project/
├── fastapi_parallel_backend.py
├── frontend/
└── README.md
```

## Run locally

Install the dependencies used by the backend:

```bash
python -m pip install fastapi uvicorn
```

Start the API:

```bash
python -m uvicorn fastapi_parallel_backend:app --reload
```

The API is available at `http://127.0.0.1:8000`. FastAPI's interactive API documentation is at `http://127.0.0.1:8000/docs`.

## API

| Method | Endpoint | Description |
|---|---|---|
| `POST` | `/submit` | Submit a non-empty list of integers; returns a `job_id`. |
| `GET` | `/status/{job_id}` | Read the current job record and status. |
| `GET` | `/result/{job_id}` | Read the result when finished, or the current status while it is pending. |
| `GET` | `/health` | Return service status and the configured process-worker count. |

The implementation does **not** currently define `/compute-sync` or `/compute-async`; those routes were removed from this README because documenting endpoints that do not exist sends API users to dead routes.

### Submit a job

```bash
curl -X POST http://127.0.0.1:8000/submit \
  -H 'Content-Type: application/json' \
  -d '{"numbers":[5,10,20,30]}'
```

A successful response contains a generated identifier:

```json
{"job_id":"<job-id>"}
```

### Poll status and retrieve the result

Replace `<job-id>` with the identifier returned by `/submit`:

```bash
curl http://127.0.0.1:8000/status/<job-id>
curl http://127.0.0.1:8000/result/<job-id>
```

A result is available after the job status becomes `finished`. Unknown identifiers return HTTP `404`.

## Frontend

If the optional frontend is included, serve its directory separately:

```bash
cd frontend
python -m http.server 5500
```

Then open `http://127.0.0.1:5500`. The frontend must send requests to the API origin shown above (or to the configured deployment origin).
