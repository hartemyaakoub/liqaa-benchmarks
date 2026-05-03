# Methodology

Version `v1.0` · last updated 2026-05-03.

If you change the methodology, **bump the version** and document the change in [`CHANGELOG.md`](./CHANGELOG.md). Comparing results across methodology versions is meaningless — you need a single version to draw conclusions.

## Goals

This benchmark answers **one** question: *for the median user in each region, how does this provider feel?* It does **not** measure:

- Maximum throughput at scale
- Cost per minute (see [`liqaa-pricing-comparison`](https://github.com/hartemyaakoub/liqaa-roadmap) — TBD)
- Subjective video / audio quality past first frame
- Multi-publisher SFU performance under load

Those need separate, more involved benchmarks. We may publish them later.

## What we measure

### 1. Signaling latency

Time from "client opens a TCP socket" to "client receives the first message on the WebSocket signalling channel".

```
t0 = open TCP socket
t1 = TLS handshake complete
t2 = HTTP Upgrade: websocket sent
t3 = first WebSocket frame received from server
signaling = t3 - t0
```

We report **p50 / p95 / p99** of 100 probes per provider per region.

### 2. API latency

Time from "POST /room with a JSON body" to "201 Created received". This is the time until your backend has a join URL.

```
t0 = TCP connect
t1 = TLS handshake complete
t2 = POST request sent
t3 = response headers received
api = t3 - t0
```

### 3. SDK bundle size

`curl -sSL https://provider.com/sdk.js | gzip -9 | wc -c`. Compared brotli-11 too (`brotli --best`).

We use the production SDK URL each provider documents — **not** custom-bundled-and-tree-shaken estimates.

### 4. First-frame latency

Time from "client `join()` is called" to "client renders the first peer's video frame". Requires a real peer publishing — we run a Puppeteer pair on the same machine for this.

```
t0 = client.join()
t1 = onTrackPublished fires
t2 = onVideoElementHasFirstFrame fires
firstFrame = t2 - t0
```

### 5. Recovery on packet loss

Inject 5% UDP packet loss via `tc qdisc add … netem loss 5%`, run a 60-second 720p call, measure perceptual quality (PESQ for audio, MOS-based VMAF for video).

This is the most expensive metric — we run it once per release, not weekly.

## What we control for

- ✅ **Same machine** for all providers in a single run (eliminates network variance between providers)
- ✅ **Same time** — providers tested round-robin in 30-second windows
- ✅ **Same physical location** — bare metal in Hetzner FSN1 (Frankfurt) or Contabo Algeria
- ✅ **Same TLS library** — Node.js native (single OpenSSL implementation)
- ✅ **Cold connections** — `Connection: close`, fresh TCP socket per probe

## What we don't control for

- ❌ Provider-specific routing differences. Twilio may route us through their US edge while LIQAA goes to EU edge. **This is the point** — these are real-user numbers.
- ❌ Provider rate limits (we stay well under documented limits).
- ❌ Time of day. We run at a fixed UTC time to minimise this, but congestion still varies.

## What's reproducible

```bash
git clone https://github.com/hartemyaakoub/liqaa-benchmarks
cd liqaa-benchmarks
cp .env.example .env  # fill in API keys for each provider
npm install
node bench/run.mjs --region=eu-west --providers=all
```

Output lands in `results/<date>/<region>.csv` and `results/<date>/<region>.json`. Hash-chained — if anyone modifies a past result, the next run's hash chain breaks.

## When numbers contradict marketing

Several providers' marketing pages cite faster numbers than what we measure. Possible reasons:

1. **They measure a different thing.** "Latency" without specifying signaling vs API vs first-frame is a marketing term, not a metric.
2. **They measure from a single privileged region.** Their NYC datacentre to a probe in their NYC datacentre is naturally fast.
3. **The numbers are real but old.** Infrastructure changes; numbers from 2023 don't describe 2026.
4. **They include keep-alive reuse.** Real-world cold-connect numbers are slower than warm-pool numbers.

We publish the receipts; readers should verify with their own runs.
