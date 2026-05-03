<div align="center">

# LIQAA Benchmarks

**Public, reproducible performance benchmarks for video infrastructure providers.**

We benchmark LIQAA against Daily.co, Twilio Programmable Video, LiveKit Cloud, and Whereby — and we publish the methodology, the data, and the receipts.

[![runs](https://img.shields.io/badge/runs-weekly-1d4ed8?style=flat-square)](./results)
[![license](https://img.shields.io/badge/data-CC--BY--4.0-475569?style=flat-square)](./LICENSE)

</div>

---

## Why open benchmarks?

The video API space is full of "we're 10x faster" marketing claims with **zero published methodology**. We think the customer deserves better.

This repo:

- 📊 **Publishes raw probe data** (CSV + JSON) — you can verify our claims byte-by-byte
- 🔁 **Is reproducible** — clone the repo, run `node bench/run.mjs`, get your own numbers
- 🌍 **Probes from multiple regions** — Frankfurt, Algiers, Singapore, São Paulo
- ⏱ **Runs continuously** — weekly cron via GitHub Actions
- 📋 **Documents every change** — methodology versioned in `methodology.md`

## What we measure

| Metric | Method | Why it matters |
| --- | --- | --- |
| **Signaling latency** (p50 / p95 / p99) | TLS handshake → first WebSocket message | Time until peer can join a room |
| **API latency** (p50 / p95 / p99) | POST /room → 201 Created | Time until your backend has a join URL |
| **SDK bundle size** | gzipped + brotli | What ships to your customers |
| **First-frame latency** | join → first peer video frame | The "did it actually work?" moment |
| **Recovery on packet loss** | iperf-injected 5% loss → quality MOS | What happens on a bad mobile connection |

## Latest results (snapshot)

> Last full run: **2026-05-03** · Frankfurt (EU)
> Each provider: 100 sequential probes · ~2 min per provider.

| Provider              | Signaling p50 | API p50  | SDK gz | First-frame p50 |
| --------------------- | ------------- | -------- | ------ | --------------- |
| **LIQAA**             | **41 ms**     | **89 ms** | **6.4 KB** | **920 ms**     |
| Daily.co              | 53 ms         | 142 ms   | 18 KB  | 1.2 s           |
| Twilio Programmable   | 78 ms         | 188 ms   | 24 KB  | 1.8 s           |
| LiveKit Cloud         | 38 ms         | 95 ms    | 12 KB  | 880 ms          |
| Whereby               | 64 ms         | 167 ms   | 31 KB  | 1.4 s           |

> ⚠️ **Numbers will move.** Re-run the benchmark after major releases. The whole point of this repo is reproducibility — don't trust this table; trust your own run.

[See full results](./results) · [See methodology](./methodology.md)

## Reproduce

```bash
git clone https://github.com/hartemyaakoub/liqaa-benchmarks.git
cd liqaa-benchmarks
npm install
node bench/run.mjs --region=eu-west --providers=liqaa,daily,twilio
# results saved to ./results/<date>/<region>.csv
```

You'll need API keys for each provider (free tiers are sufficient for the probe volume).

## Methodology — TL;DR

- **No authenticated traffic in benchmarks** — only TLS connect + handshake + first packet. We're measuring **infra**, not your code.
- **Cold connections** — every probe is a fresh TCP socket (no keep-alive reuse).
- **Same IP, same time** — providers are probed in tight loops to minimize transient variation.
- **Outliers reported, not removed** — we publish min/max alongside p50/p95/p99.
- **Public probe code** — `bench/` is the entire test harness (~200 lines).

Read the [full methodology](./methodology.md) before drawing conclusions.

## Disagree with the results?

Open a PR. We'll merge it if your methodology fix is correct, even if it makes us look worse.

## License

- **Data** (everything in `results/`) — [CC-BY-4.0](./LICENSE-DATA): use freely with attribution.
- **Code** (everything in `bench/`) — [MIT](./LICENSE).
