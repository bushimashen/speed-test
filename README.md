# Speed Test · 网速测试

> Measure latency, download and upload speed straight in your browser. No app, no account, no third-party middleman UI.

**English** | [简体中文](README.zh-CN.md)

<a href="https://bushimashen.github.io/speed-test/"><b>▶ Live demo</b></a>

## Features

- **Latency** — 6 round trips, median taken so a single hiccup doesn't skew the number
- **Download** — 4 parallel workers pulling chunks for 9 seconds, chunk size doubling itself when the link has headroom
- **Upload** — 2 parallel workers POSTing **cryptographically random** blobs for 8 seconds (zero-filled buffers get compressed by the transport and would report a fake-fast result)
- **Live gauge** — a logarithmic 1–1000 Mbps scale, so a 5 Mbps link still shows visible movement
- **Grade** — plain-language verdict from the measured download speed

## Why it uses a public endpoint

A speed test needs something to talk to. This tool deliberately has **no backend**: it talks to
Cloudflare's public speed endpoints (`speed.cloudflare.com/__down` and `/__up`), which serve random
bytes with `Access-Control-Allow-Origin: *` and need no key.

That keeps the whole tool a single HTML file, but it also means the numbers describe *your link to a
nearby Cloudflare PoP*, not to any particular server. That is the right thing to measure for "how fast
is my connection", and the wrong thing if you wanted to benchmark one specific host.

**Pointing it at your own server** — edit three constants at the top of the script:

```js
const NODE     = 'https://speed.cloudflare.com'
const DOWN_URL = (bytes) => `${NODE}/__down?bytes=${bytes}`   // GET  -> N random bytes
const UP_URL   = `${NODE}/__up`                               // POST -> any 2xx
```

Your endpoints must send `Access-Control-Allow-Origin`, otherwise the browser refuses to read the
response and the test silently reports 0.

## Usage

Download `index.html` and double-click it, or serve it locally:

```bash
python -m http.server 8000
```

> Under `file://` some browsers refuse cross-origin `fetch`. If the test errors out immediately,
> use the local server above.

## How the numbers are computed

```
latency  = median(rtt[0..5])
download = totalBytes * 8 / 1e6 / 9      (Mbps)
upload   = totalBytes * 8 / 1e6 / 8      (Mbps)
```

Both throughput phases run for a fixed wall-clock window rather than a fixed byte count — that's what
keeps the result stable on slow links without over-transferring on fast ones.

## Notes

- Uploading really does consume traffic roughly equal to the reported figure × 8 seconds.
- Results vary with WiFi, time of day and your ISP's peering; a single run is a snapshot, not a verdict.
- No analytics, no storage, nothing leaves the page except the speed probes themselves.

## License

MIT — see [LICENSE](LICENSE).
