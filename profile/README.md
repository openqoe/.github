# OpenQoE

**Prometheus‑Native Video Quality of Experience (QoE) Analytics – Open, Extensible, Real‑Time**
`openqoe.dev` · Contact: **[dev@openqoe.dev](mailto:dev@openqoe.dev)** · GitHub: **[https://github.com/openqoe](https://github.com/openqoe)**

---

## TL;DR

OpenQoE is an **open‑source video player telemetry + analytics stack** that standardizes client & edge QoE metrics (startup, buffering, errors, ABR, network, ads, engagement) and ships them straight into **Prometheus / OpenMetrics** for **real‑time observability, alerting, SLOs, capacity planning, A/B testing, and root‑cause analysis**.

> **Goal:** Eliminate black‑box vendor lock‑in. Bring video QoE into the same first‑class SRE/DevOps tooling you already use (Prometheus, Grafana, Alertmanager, Tempo/Tracing, Loki, OpenTelemetry).

---

## Why OpenQoE?

| Problem                                      | Impact                                    | OpenQoE Answer                                                                            |
| -------------------------------------------- | ----------------------------------------- | ----------------------------------------------------------------------------------------- |
| Proprietary analytics silos                  | Fragmented view; slow MTTR                | Unified OSS pipeline into Prometheus                                                      |
| Delayed batch dashboards                     | Reactive firefighting                     | **<1 min** metric latency via streaming ingest                                            |
| Inconsistent player metric names             | Hard to do cross‑player / device analysis | Canonical **OpenQoE Metric Taxonomy**                                                     |
| Expensive per‑view licensing                 | Cost scales badly with growth             | Self‑hosted + efficient Prometheus exposition                                             |
| Limited correlation (CDN ↔ Player ↔ Backend) | Hidden multi‑layer issues                 | Common labels let you join across layers (session, asset\_id, edge\_pop, isp, experiment) |
| Manual SLO tracking                          | Unclear reliability posture               | Built‑in SLO templates & burn‑rate alerts                                                 |

---

## Core Principles

1. **Open Spec First** – Metric names, label schema, event model published & versioned.
2. **Prometheus Native** – No translation layer; direct exposition / remote‑write.
3. **Edge & Client Symmetry** – Correlate player ↔ CDN ↔ API ↔ encoder metrics.
4. **Minimal Overhead** – Lightweight SDK (<10 KB gzipped target), sampling & adaptive flush.
5. **Privacy Aware** – PII minimization, hashing strategies, opt‑out by design.
6. **Extensible** – Plugins for custom events, ABR heuristics, ad beacons, QoS probes.
7. **Deterministic Versioning** – `spec_semver` label for safe evolution.

---

## High‑Level Architecture

```
Player SDKs  -->  Local Buffer & Sampler  -->  Collector Gateway  -->  Prometheus
 (Web, MSE,     (interval & event flush)     (HTTP ingest,           (scrape / remote-write)
  iOS/tvOS,                                     multi-tenant, auth)
  Android, STB) --> Optional Edge Relays -->   Stream Processors -->  Long-term Storage (TSDB, Mimir, Thanos)
                                     |-->  Alertmanager / SLO Burn
                                     |-->  Grafana Dashboards
                                     |-->  Traces (OTel / Tempo)
```

**Data Flow:** Player events → normalized metric events → exposition (OpenMetrics / OTLP) → scrape or remote‑write → dashboards & alerts.

---

## Metric Domains (Taxonomy Extract)

| Domain         | Prefix               | Examples                                                                       |
| -------------- | -------------------- | ------------------------------------------------------------------------------ |
| Startup        | `openqoe_startup_*`  | `openqoe_startup_time_seconds`, `openqoe_startup_failed_total`                 |
| Playback QoE   | `openqoe_playback_*` | `openqoe_playback_buffer_time_seconds_sum`, `openqoe_playback_rebuffers_total` |
| ABR / Bitrate  | `openqoe_abr_*`      | `openqoe_abr_switch_total`, `openqoe_abr_bitrate_bits_current`                 |
| Network        | `openqoe_net_*`      | `openqoe_net_download_bytes_total`, `openqoe_net_segment_latency_seconds`      |
| Errors         | `openqoe_err_*`      | `openqoe_err_player_total{error_code="..."}`                                   |
| Engagement     | `openqoe_eng_*`      | `openqoe_eng_view_qualified_total`, `openqoe_eng_watch_time_seconds_sum`       |
| Ads (optional) | `openqoe_ad_*`       | `openqoe_ad_impression_total`, `openqoe_ad_startup_time_seconds`               |
| Experiments    | `openqoe_exp_*`      | `openqoe_exp_variant_watch_time_seconds_sum`                                   |
| SLO Helper     | `openqoe_slo_*`      | `openqoe_slo_rebuffer_ratio`, `openqoe_slo_startup_p95_seconds`                |

> Full taxonomy & semantic definitions will live under `/spec/metric-taxonomy.md` (coming soon).

---

## Standard Labels (Core)

`session_id`, `viewer_id_hash`, `asset_id`, `content_type`, `is_live`, `cdn`, `edge_pop`, `isp`, `country`, `device_type`, `os_family`, `player_version`, `sdk_version`, `experiment_id`, `experiment_variant`, `spec_version`.

**Design Notes:**

* **Cardinality Control:** Hashing + sampling for high‑throughput live events.
* **Experiment Support:** Native labels enable A/B deltas without extra ETL.

---

## Feature Highlights

* **Lightweight JavaScript SDK** (TypeScript source, tree‑shakable)
* **Mobile SDKs** (Kotlin, Swift) – parity in metric semantics
* **Edge Collector Gateway** (Go or Rust + optional WASM filters) with:

  * Multi‑tenant API keys
  * Rate & cardinality guards
  * Adaptive backpressure & loss accounting
  * OpenTelemetry export (metrics + traces)
* **Prometheus / OTLP Exporters** (pull & push)
* **Grafana Starter Dashboards** (`/deploy/grafana/` JSON)
* **SLO Library** – YAML recipes to define startup time / rebuffer / failure objectives
* **Alert Packs** – Prebuilt multi‑window burn‑rate alert rules
* **Sampling & Budgeting** – Dynamic sample rate based on session state (startup, stable, error burst)
* **Replay / Offline Upload** – Buffer offline sessions & flush when network recovers
* **Plugin Interface** – Add custom metric streams (e.g., DRM, latency probes, perceptual QoE ML)
* **Privacy Toolkit** – Hash / salt / rotate viewer identifiers; regional redact rules

---

## Quick Start (Preview)

```bash
# 1. Run the collector (example Docker Compose)
version: "3.9"
services:
  openqoe-collector:
    image: ghcr.io/openqoe/collector:latest
    ports: ["8080:8080"]
    environment:
      - OPENQOE_PROM_REMOTE_WRITE_URL=http://prometheus:9090/api/v1/write
  prometheus:
    image: prom/prometheus:latest
    ports: ["9090:9090"]
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml

# 2. Add the SDK to your web player build
npm install @openqoe/sdk

# 3. Initialize
import { OpenQoe } from '@openqoe/sdk';
const qoe = new OpenQoe({ endpoint: 'https://collector.example.com/v1/ingest', assetId: 'movie123', isLive: false });
qoe.startSession();

# 4. Hook player events
player.on('play', () => qoe.notePlaybackStart());
player.on('error', (e) => qoe.noteError(e.code));
```

> A full **Integration Guide** will ship once the initial spec solidifies.

---

## Observability & Tooling

| Component                   | Purpose                                          |
| --------------------------- | ------------------------------------------------ |
| Prometheus / Mimir / Thanos | Short & long‑term metrics storage                |
| Grafana                     | Dashboards, alert rule mgmt                      |
| Alertmanager                | SLO burn‑rate + anomaly alerts                   |
| Loki / ELK                  | Correlate logs (segment fetch, CDN edge)         |
| Tempo / Jaeger              | Trace segment request waterfall                  |
| OpenTelemetry               | Unified context propagation & exporter semantics |

---

## SLO Example

```yaml
# slo/rebuffer_ratio.yaml
apiVersion: openqoe.dev/v1
kind: PlaybackSLO
metadata:
  name: rebuffer-ratio
spec:
  objective: 0.02        # <=2% rebuffer ratio
  window: 30d
  alert:
    fast_burn: {factor: 14, windows: [5m, 1h]}
    slow_burn: {factor: 7, windows: [30m, 6h]}
```

---

## Roadmap (Public Draft)

| Phase               | Focus            | Key Items                                                                 |
| ------------------- | ---------------- | ------------------------------------------------------------------------- |
| **0.1 (Bootstrap)** | Spec + Prototype | Metric taxonomy draft, JS SDK minimal, Collector skeleton                 |
| **0.2**             | Stability & Docs | Sampling, batching, Grafana MVP dashboards, taxonomy v1.0 tag             |
| **0.3**             | Mobile & Edge    | iOS + Android SDK betas, CDN edge label enrichment, SLO pack v1           |
| **0.4**             | Scale & Perf     | High‑throughput ingest benchmarks, adaptive sampling, Mimir/Thanos guides |
| **0.5**             | Extensibility    | Plugin API, custom events, ABR heuristics adapters                        |
| **0.6**             | Advanced QoE     | ML‑aided perceptual metrics, anomaly detection experiments                |
| **1.0**             | GA               | Hardening, security review, versioned spec freeze, migration guide        |

> Roadmap subject to community discussion; issues & PRs welcome.

---

## Contributing

We *love* early contributors:

1. **Star & Watch** the repo(s) under `github.com/openqoe`.
2. Read **`CONTRIBUTING.md`** (draft pending) & open a **Proposal Issue** before large PRs.
3. Use **conventional commits** (`feat:`, `fix:`, `doc:`) to drive automated release notes.
4. Add / update **spec docs** when introducing or modifying metric names / labels.
5. Include **bench & overhead numbers** for performance‑sensitive changes.

**Good First Issues (planned):** SDK session lifecycle, collector auth middleware, Grafana startup dashboard, taxonomy doc scaffolding.

---

## Security & Privacy

* Principle of least data (no raw IP or direct user IDs stored in metrics; hash & salt).
* Transport security (HTTPS/TLS by default; optional mTLS for edge → core).
* API keys with scoped quotas & label cardinality guardrails.
* Coordinated disclosure: email **[security@openqoe.dev](mailto:security@openqoe.dev)** (planned alias) for vulnerabilities.

---

## Governance (Draft)

* **Spec Stewards:** Small rotating core maintainers approve taxonomy changes.
* **Release Cadence:** Minor release every 4 weeks until 1.0; patch as needed.
* **Enhancement Proposals:** `OQEP-####` markdown RFCs for material changes.

---

## FAQ (Early)

**Q:** Why not just use a commercial QoE vendor?
**A:** Openness, cost predictability, integration with existing SRE stack, and custom experimentation agility.

**Q:** Does this replace RUM / APM?
**A:** No—OpenQoE complements them with *domain‑specific* video playback metrics.

**Q:** How heavy is the SDK?
**A:** Target <10 KB gzipped core (without optional plugins) with adaptive event flush to minimize overhead.

**Q:** What about tracing?
**A:** Segment & manifest fetch spans exported via OpenTelemetry for deep waterfall analysis.

---

## Ecosystem Ideas

* **Browser Extension** for live session inspection.
* **Synthetic Prober** (headless players) for continuous baseline QoE checks.
* **Workbook Templates** (Jupyter / Observable) for advanced analysis.
* **Anomaly Detector** (PromQL + lightweight ML) to pre‑empt regressions.

---

## Badges (To Add)

```
[![License](https://img.shields.io/badge/license-Apache--2.0-blue)](#license)
[![Spec](https://img.shields.io/badge/spec-draft-orange)]()
[![CI](https://img.shields.io/badge/ci-pending-lightgrey)]()
[![SDK Size](https://img.shields.io/badge/sdk-%3C10KB-success)]()
```

---

## License

Planned: **Apache 2.0** (explicit patent grant, contribution clarity). Final confirmation coming with first tagged release.

---

## Contact & Links

* Website: **[https://openqoe.dev](https://openqoe.dev)** (coming online as docs portal)
* Email: **[dev@openqoe.dev](mailto:dev@openqoe.dev)**
* GitHub Org: **[https://github.com/openqoe](https://github.com/openqoe)**
* Issues: Use GitHub Issues for bugs & proposals

---

## Call to Action

> ⭐ **Star the repos.**
> 🐞 **File an issue** for any metric / label ambiguity.
> 🧪 **Try the prototype** in staging & share trace + metric snapshots.
> 🗣️ **Join early governance discussions** – help shape the open QoE standard.

Let’s make video QoE *observable by default*.
