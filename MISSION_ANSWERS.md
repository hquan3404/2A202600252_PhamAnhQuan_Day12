# Day 12 Lab - Mission Answers

## Part 1: Localhost vs Production

### Exercise 1.1: Anti-patterns found
1. **API key hardcoded:** `OPENAI_API_KEY = "sk-hardcoded-fake-key-never-do-this"` - This exposes secrets if the code is committed.
2. **Missing Configuration Management:** Parameters like `DEBUG = True` and hardcoded `DATABASE_URL` limit flexibility between development and production environments.
3. **Improper Logging:** The application uses `print()` statements for debugging instead of a structured logging library, and even logs sensitive information (`OPENAI_API_KEY`).
4. **Missing Health Check Endpoint:** No `/health` or `/ready` endpoints are provided for cloud orchestration platforms to monitor container status.
5. **Fixed Port and Localhost Binding:** Running uvicorn with `host="localhost"` and `port=8000` hardcoded prevents cloud deployment. It should bind to `0.0.0.0` and read the port from environment variables.
6. **No Graceful Shutdown:** The process crashes completely upon receiving signals without giving active requests a chance to finish.

### Exercise 1.3: Comparison table
| Feature | Develop | Production | Why Important? |
|---------|---------|------------|----------------|
| Config  | Hardcoded | Environment Variables | Prevents committing secrets and makes changing configurations between stages (dev, test, prod) easier. |
| Health check | ❌      | ✅          | Allows orchestration platforms (Docker, Kubernetes) to monitor the application state and restart failed instances automatically. |
| Logging | `print()` | Structured JSON | Easily parsable by log aggregators (e.g., Datadog, ELK), provides trace information, and helps avoid logging sensitive variables. |
| Shutdown | Abrupt  | Graceful       | Prevents data loss and allows active HTTP requests/database transactions to finish gracefully before the process exits. |

## Part 2: Docker

### Exercise 2.1: Dockerfile questions
1. **Base image:** `python:3.11` (which is a full OS + Python interpreter, almost ~1GB).
2. **Working directory:** `/app` (code executes from here inside the container).
3. **Copy requirements first:** Copying `requirements.txt` before the rest of the code leverages Docker layer caching. If only source code changes but dependencies do not, the `pip install` layer doesn't need to be rebuilt.
4. **CMD vs ENTRYPOINT:** `CMD` supplies default arguments or commands which can be easily overridden from the command line (`docker run <image> <command>`). `ENTRYPOINT` configures a container that will run as an executable.

### Exercise 2.3: Image size comparison
- **Develop (Single-stage):** ~1.0 GB
- **Production (Multi-stage + Slim):** ~150-200 MB
- **Difference:** ~80% reduction
*(Note: To get exact exact numbers, run `docker images`. The multi-stage uses a `-slim` base and only brings site-packages to the runtime stage.)*
**Why Multi-stage:** The builder stage contains compilers and tools needed to compile dependencies, while the runtime stage contains only what is needed to strictly run the app plus the actual dependencies. This makes the resulting image much smaller and more secure.

## Part 3: Cloud Deployment

### Exercise 3.1 & 3.2: Railway/Render Configuration differences
- **`railway.toml`** uses simple build configurations, defaults to Nixpacks if no Dockerfile is provided, and automatically injects the `$PORT`.
- **`render.yaml`** uses continuous delivery features directly (`autoDeploy`), configures a specific `plan`, specifies the region explicitly, handles Redis add-ons natively inside the file, and relies on an explicit command instead of auto-detection.

## Part 4: API Security

### Exercise 4.3: Rate limiting details
- **Algorithm Used:** Sliding Window Counter using `deque` to store timestamps.
- **Limit:** 10 requests / 60 seconds (user), 100 requests / 60 seconds (admin).
- **Explanation:** Old timestamps outside the 60-second window are stripped. If the number of timestamps in the queue matches the limit, the request is blocked and it responds with `429 Too Many Requests`.

### Exercise 4.4: Cost guard implementation
*Implementation approach for tracking spend with Redis:*
```python
import redis
from datetime import datetime

r = redis.Redis()

def check_budget(user_id: str, estimated_cost: float) -> bool:
    """
    Return True if within budget, False if exceeded.
    Logic: Track monthly spending in Redis.
    """
    # Create key based on current month
    month_key = datetime.now().strftime("%Y-%m")
    key = f"budget:{user_id}:{month_key}"
    
    # Retrieve current spending, defaulting to 0
    current_spend = float(r.get(key) or 0)
    
    # Check limit ($10)
    if current_spend + estimated_cost > 10.0:
        return False
    
    # Increment spending
    r.incrbyfloat(key, estimated_cost)
    # Set expiration just over a month to clean up stale data
    r.expire(key, 32 * 24 * 3600)
    
    return True
```

## Part 5: Scaling & Reliability

### Exercise 5.1-5.5: Implementation notes
- **Health/Readiness probes:** The endpoint `/health` confirms the container is alive (uptime), while `/ready` confirms it's capable of taking requests. The system gracefully signals a `503 Unavailable` intentionally during startup.
- **Graceful Shutdown:** The `FastAPI` instance includes a `lifespan` manager and uses signal trapping (`signal.SIGTERM`). It blocks exiting while the `_in_flight_requests` counter is above 0, completing pending LLM requests before fully terminating.
- **Stateless Design:** `05-scaling-reliability/production/app.py` demonstrates replacing in-memory dictionaries for chat histories with Redis. This lets multiple agent containers load balance traffic since any worker can fetch any user's state from Redis.
