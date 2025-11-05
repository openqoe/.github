# OpenQoE

**Open-Source Video Quality of Experience Monitoring**

We build production-ready tools to monitor and analyze video streaming quality across platforms.

---

## 🎯 What We Do

OpenQoE provides a complete, self-hosted solution for monitoring video Quality of Experience (QoE) metrics. Track video startup time, rebuffering, errors, and engagement in real-time with enterprise-grade observability.

### Key Projects

- **[openqoe-dev](https://github.com/openqoe/openqoe-dev)** - Complete QoE monitoring platform
  - JavaScript SDK for 5 video players
  - Cloudflare Worker for edge ingestion
  - Production Grafana dashboards
  - Self-hosted or cloud-ready

---

## 🚀 Quick Start

```bash
# Clone and run locally
git clone https://github.com/openqoe/openqoe-dev.git
cd openqoe-dev
docker compose up -d

# Access Grafana at http://localhost:3000
```

---

## 📊 What You Can Monitor

- **Video Startup Time** - P50, P95, P99 percentiles
- **Rebuffering** - Rate, duration, impact on completion
- **Errors** - Categorized by type, device, network
- **Engagement** - Watch time, completion rate, quartiles
- **ABR Performance** - Quality switches, bitrate distribution
- **Live Streaming** - Concurrent viewers, join time

---

## 🌍 Deployment Options

- **Self-Hosted** - Docker stack with Mimir, Loki, Grafana
- **Grafana Cloud** - Fully managed metrics and logs
- **Hybrid** - Edge ingestion with on-premise storage

---

## 🎨 Supported Players

- HTML5 Video
- Video.js
- HLS.js
- dash.js
- Shaka Player

---

## 📚 Resources

- **Website**: [openqoe.dev](https://openqoe.dev)
- **Documentation**: [Deployment Guide](https://github.com/openqoe/openqoe-dev/blob/main/docs/deployment-guide.md)
- **Issues**: [Report bugs or request features](https://github.com/openqoe/openqoe-dev/issues)
- **Discussions**: [Community Q&A](https://github.com/openqoe/openqoe-dev/discussions)

---

## 🤝 Contributing

We welcome contributions! Check out our [Contributing Guide](https://github.com/openqoe/openqoe-dev/blob/main/docs/contributing.md) to get started.

---

## 📄 License

All projects are open-source under the [Apache License 2.0](https://github.com/openqoe/openqoe-dev/blob/main/LICENSE).

---

## ⭐ Support the Project

If you find OpenQoE useful, please star our repositories and share with your network!

[![Star openqoe-dev](https://img.shields.io/github/stars/openqoe/openqoe-dev?style=social)](https://github.com/openqoe/openqoe-dev)

