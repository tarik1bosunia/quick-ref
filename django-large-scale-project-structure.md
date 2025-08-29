# Large-scale Django + DRF Project Guide

> A practical, opinionated guide and example layout for building and maintaining a **large-scale** Django + Django REST Framework application. Focuses on directory structure, architectural patterns, conventions, testing, deployment, and operational concerns.

---

## Table of Contents

1. Core Principles & Patterns
2. Recommended Directory Structure
3. App-Level Conventions

   * Selectors (read logic)
   * Services (business logic)
   * Repositories (optional abstraction)
   * Schemas / DTOs
4. Settings Management (dev / prod / staging)
5. Models, Serializers, Views, Routers — Detailed Example
6. Background Workers & Real-time (Celery, Channels)
7. Tests & Testing Strategy
8. API Versioning, Documentation & Standards
9. CI/CD, Docker, and Deployment
10. Observability, Logging, and Monitoring
11. Security, Performance, and Scaling Tips
12. Code Quality & Tooling
13. Example Snippets & Useful Patterns
14. Final Notes

---

## 1. Core Principles & Patterns

* **Separation of concerns**: Keep business logic out of views and serializers.
* **Read/Write separation**: Use *selectors* for queries and *services* for commands.
* **Consistency**: Follow common patterns across apps.
* **Testability**: Keep units small and test selectors/services independently.
* **API stability**: Version your APIs early (`/api/v1/...`).
* **Config as code**: No secrets in repo; use `.env`, Vault, or Secret Manager.
* **Resilience**: Assume external systems fail; design retries and idempotency.

---

## 2. Recommended Directory Structure

```
project_root/
├── apps/                         # all Django apps live here
│   ├── users/
│   │   ├── models.py
│   │   ├── selectors.py
│   │   ├── services.py
│   │   ├── repositories.py
│   │   ├── schemas.py           # optional: pydantic-style DTOs
│   │   ├── serializers.py
│   │   ├── views.py
│   │   ├── urls.py
│   │   ├── permissions.py
│   │   ├── signals.py
│   │   ├── tasks.py             # Celery tasks for this app
│   │   ├── consumers.py         # Channels consumers
│   │   └── tests/
│   │       ├── test_selectors.py
│   │       ├── test_services.py
│   │       ├── test_views.py
│   │       └── factories.py
│   └── posts/
├── config/                       # project-level configuration
│   ├── settings/
│   │   ├── base.py
│   │   ├── dev.py
│   │   ├── prod.py
│   │   └── staging.py
│   ├── urls.py
│   ├── wsgi.py
│   └── asgi.py
├── middleware/                   # custom middlewares
├── common/                       # shared utils (decorators, helpers)
├── scripts/                      # dev & ops scripts
├── docker/                       # docker configs, k8s manifests
├── docs/                         # API docs, ADRs
├── tests/                        # e2e / integration tests
├── requirements/                 # requirements split (base, dev, prod)
├── pyproject.toml / requirements.txt
├── manage.py
└── README.md
```

---

## 3. App-Level Conventions

### Selectors (read/query logic)

* Read-only, return QuerySets or domain objects.
* Should be composable and easily testable.

### Services (write/command logic)

* Business rules, transactions, side effects.
* Must be idempotent when possible.

### Repositories

* Optional abstraction over ORM.
* Useful for large teams or when DB might change.

### Schemas / DTOs

* Use [Pydantic](https://docs.pydantic.dev/) or DRF `Serializer` without `.save()` for data transfer objects.
* Keeps API contracts clear.

---

## 4. Settings Management

* Use layered approach: `base.py` + environment-specific overrides (`dev.py`, `prod.py`).
* Always load secrets from environment variables.
* Separate logging config, cache config, storage config.

---

## 5. Models, Serializers, Views, Routers — Example

(Users and Posts examples shown earlier)

**Additional best practices:**

* Always add indexes (`db_index=True` or `Meta.indexes`).
* Use UUIDs for public identifiers.
* Keep `choices` in enums (`models.TextChoices`).
* Serializers should validate and transform, but not contain business logic.

---

## 6. Background Workers & Real-time

* **Celery** for async tasks (emails, payments, image processing).
* **Django Channels** for websockets/notifications.
* **Signals** for small event hooks (but don’t overuse).
* Keep tasks **thin** — delegate real work to services.

---

## 7. Tests & Testing Strategy

* **pytest** + `pytest-django` recommended.
* Use `factory_boy` or `model_bakery` for test data.
* Follow test pyramid:

  * Unit: selectors/services (fast, majority)
  * Integration: API endpoints, permissions
  * E2E: few, critical paths
* Use `pytest-xdist` for parallel runs.

---

## 8. API Versioning, Documentation & Standards

* Prefix all API endpoints with `/api/v1/`.
* Use [drf-spectacular](https://drf-spectacular.readthedocs.io/) for OpenAPI schema generation.
* Maintain docs in `docs/` and auto-generate Swagger/Redoc.
* Standardize error responses (e.g., `{ "error": "message" }`).

---

## 9. CI/CD, Docker, and Deployment

* Use multi-stage Docker builds.
* Compose for local dev with Postgres, Redis, Celery, Flower.
* In production, run separate services:

  * Web (Gunicorn/Uvicorn)
  * Worker (Celery)
  * Scheduler (Celery beat)
  * Channels server (Daphne/Uvicorn)
* CI pipeline: Lint → Test → Build → Push → Deploy.
* Automate DB migrations on deploy.

---

## 10. Observability, Logging, and Monitoring

* Use JSON structured logging.
* Add correlation IDs in middleware.
* Capture metrics with Prometheus.
* Distributed tracing with OpenTelemetry.
* Alerting on error rates, slow queries, worker lag.

---

## 11. Security, Performance, and Scaling

* Use HTTPS, secure cookies, HSTS.
* Rate-limit login and sensitive endpoints.
* Avoid N+1 queries with `select_related` / `prefetch_related`.
* Cache expensive queries in Redis.
* Use CDN for media/static.
* Read replicas for DB scaling.

---

## 12. Code Quality & Tooling

* Linters: `flake8`, `ruff` or `pylint`.
* Formatter: `black`.
* Type checking: `mypy`.
* Security scans: `bandit`, `safety`.
* Pre-commit hooks for consistency.

---

## 13. Example Snippets & Useful Patterns

### Middleware: Correlation ID

```python
class CorrelationIdMiddleware(MiddlewareMixin):
    def process_request(self, request):
        request.correlation_id = request.headers.get('X-Correlation-ID') or str(uuid.uuid4())
    def process_response(self, request, response):
        if hasattr(request, 'correlation_id'):
            response['X-Correlation-ID'] = request.correlation_id
        return response
```

### Selector with Optimization

```python
def get_post_with_author(post_id: int):
    return Post.objects.select_related('author').filter(id=post_id).first()
```

### Service that Enqueues Task

```python
@transaction.atomic
def publish_post(post: Post):
    post.published = True
    post.save(update_fields=['published'])
    render_post_html.delay(post.id)
    return post
```

---

## 14. Final Notes

This guide is a blueprint for large-scale Django + DRF projects. Key takeaways:

* Keep views thin, move logic into services/selectors.
* Version and document your API.
* Enforce consistency across apps.
* Automate testing, CI/CD, and deployments.
* Monitor, log, and secure from day one.

With these patterns, your Django project will remain **maintainable, testable, and scalable** as it grows.
