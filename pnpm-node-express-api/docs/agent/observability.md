## 12. Logging & Observability — Metrics & Tracing

> Read this when adding logging, metrics, tracing, or health checks.

### Logging

- **winston** with `DailyRotateFile` for `logs/error-%DATE%.log` (level error, 14d, 20m, zipped) and `logs/combined-%DATE%.log` (all levels). Import as ESM.
- Console transport in non-production with colorized `HH:mm:ss [level] message`.
- Levels: `error, warn, info, success, debug`; default `info` in production, `debug` otherwise.
- **morgan** `combined` piped to `logger.info` via `stream.write`.
- Health endpoints: `GET /` (`{ status, message, version }`) and `GET /health` (`{ status, uptime, memory, timestamp }`). Mounted **before** `/api/v1`.
- DB `connectDB` / `closeDB` log success/failure; `server.js` handles `SIGTERM`/`SIGINT` graceful shutdown.
- **Structured logs:** always include `requestId` (from `X-Request-Id` header or generated `crypto.randomUUID()`), and `traceId` when tracing is enabled (see below). Never log `req.cookies`, tokens, or PII.

### Metrics — Prometheus via `prom-client`

- **Deps:** `prom-client`.
- **Setup** (`src/utils/metrics.js`, ESM):
  ```js
  import client from 'prom-client';
  export const register = new client.Registry();
  client.collectDefaultMetrics({ register }); // CPU, memory, event loop
  export const httpDuration = new client.Histogram({
    name: 'http_request_duration_seconds',
    help: 'Duration of HTTP requests in seconds',
    labelNames: ['method','route','status'],
    buckets: [0.05,0.1,0.3,0.5,1,2,5],
    registers: [register],
  });
  export const httpErrors = new client.Counter({
    name: 'http_errors_total',
    help: 'Total HTTP errors',
    labelNames: ['route','status'],
    registers: [register],
  });
  ```
- **Middleware** (`src/middlewares/metrics.middleware.js`):
  ```js
  import { httpDuration, httpErrors } from '../utils/metrics.js';
  export const metricsMiddleware = (req, res, next) => {
    const end = httpDuration.startTimer();
    res.on('finish', () => {
      end({ method:req.method, route:req.route?.path || req.path, status:res.statusCode });
      if (res.statusCode >= 400) httpErrors.inc({ route:req.route?.path || req.path, status:res.statusCode });
    });
    next();
  };
  ```
- **Endpoint** (`src/routes/v1/metrics.routes.js` — no auth, but restrict via firewall/VPC or `authorize('admin')` + internal network):
  ```js
  import { Router } from 'express';
  import { register } from '../../utils/metrics.js';
  const router = Router();
  router.get('/metrics', async (_req, res) => {
    res.set('Content-Type', register.contentType);
    res.end(await register.metrics());
  });
  export default router;
  // app.js: app.use(metricsMiddleware) before routes; app.use('/metrics', metricsRouter) before auth
  ```
- **Docker:** Prometheus scrapes `api:3000/metrics` (add `prometheus` service in compose if needed), Grafana dashboards for `p95 latency`, `error rate`, `DB pool`.

### Tracing — OpenTelemetry

- **Deps:** `@opentelemetry/sdk-node`, `@opentelemetry/auto-instrumentations-node`, `@opentelemetry/exporter-trace-otlp-http`.
- **Setup** (`src/config/tracing.js`, imported **first** in `src/server.js` before any other import):
  ```js
  import { NodeSDK } from '@opentelemetry/sdk-node';
  import { getNodeAutoInstrumentations } from '@opentelemetry/auto-instrumentations-node';
  import { OTLPTraceExporter } from '@opentelemetry/exporter-trace-otlp-http';
  export const initTracing = () => {
    const sdk = new NodeSDK({
      traceExporter: new OTLPTraceExporter({ url: process.env.OTEL_EXPORTER_OTLP_ENDPOINT || 'http://jaeger:4318/v1/traces' }),
      instrumentations: [getNodeAutoInstrumentations({ '@opentelemetry/instrumentation-fs': { enabled:false } })],
    });
    sdk.start();
  };
  ```
- **server.js:** `import './config/tracing.js'` as first line, then `initTracing()`.
- **Env:** `OTEL_EXPORTER_OTLP_ENDPOINT`, `OTEL_SERVICE_NAME=<project-name>`.
- **Compose:** add `jaeger` (or `otel-collector` + `tempo`) service for local tracing; Grafana/Tempo visualizes trace `traceId` -> spans (`GET /api/v1/users` -> `service.getUsers` -> `SELECT ...`).
- **Correlation:** winston format adds `traceId` from `trace.getActiveSpan()?.spanContext().traceId` so logs and traces join.

Rules: never use `console.log` in production code; use `logger`. Never log cookies, tokens, or PII. Keep `/metrics` off public internet (VPC or admin guard). Tracing is opt-in — if `OTEL_EXPORTER_OTLP_ENDPOINT` is unset, SDK no-ops with negligible overhead.

---
