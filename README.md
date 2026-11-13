<h1 align="center">⛏️ miner</h1>

<p align="center">
  <b>Fast, concurrent secret &amp; credential miner for JavaScript files, HTTP responses and local files.</b><br>
  Built for bug bounty hunters and penetration testers who need to find leaked keys before attackers do.
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Go-1.21%2B-00ADD8?logo=go&logoColor=white" alt="Go">
  <img src="https://img.shields.io/badge/license-MIT-green" alt="License">
  <img src="https://img.shields.io/badge/platform-linux%20%7C%20macOS%20%7C%20windows-lightgrey" alt="Platform">
  <img src="https://img.shields.io/badge/dependencies-zero-brightgreen" alt="Zero dependencies">
</p>

<p align="center">
  <a href="#-features">Features</a> •
  <a href="#-installation">Installation</a> •
  <a href="#-usage">Usage</a> •
  <a href="#-recon-pipelines">Pipelines</a> •
  <a href="#-what-it-detects">Detections</a> •
  <a href="#-roadmap">Roadmap</a>
</p>

---

Modern web apps ship their secrets to the browser more often than you think: API keys hard-coded in JavaScript bundles,
`.env` files left on a web server, cloud bucket URLs, JWTs, private keys. **miner** pulls content from hundreds of URLs
in parallel and mines it for exactly these patterns, so you can turn a list of endpoints into a list of leads in seconds.

## ✨ Features

- **Three input modes:** URL lists, local files, or raw content over `stdin`. It drops straight into any Unix pipeline.
- **Concurrent by design:** goroutine worker pool with a configurable thread limit (`-t`).
- **40+ secret patterns:** cloud provider keys, SaaS tokens, database URIs, JWTs, private keys, `.env` variables and S3 bucket references.
- **Recon-friendly HTTP client:** rotating User-Agent, configurable timeout, works against self-signed and broken TLS.
- **Zero dependencies:** a single static binary built only on the Go standard library.
- **Grep-friendly output:** source on one line, findings indented underneath.

## 📦 Installation

**With Go (recommended):**

```bash
go install github.com/dedcrowd/miner@latest
```

**From source:**

```bash
git clone https://github.com/dedcrowd/miner.git
cd miner
go build -o miner .
sudo mv miner /usr/local/bin/
```

## 🚀 Usage

```
miner [flags] [file ...]

Flags:
  -f string        File containing a list of URLs to fetch and scan
  -t int           Maximum concurrent requests (default 20)
  -timeout int     HTTP timeout in seconds (default 20)
```

| Mode | Command |
|---|---|
| Scan a URL list | `miner -f urls.txt -t 50 -timeout 10` |
| URLs from stdin | `cat urls.txt \| miner -` |
| Scan local files | `miner app.js vendor.js .env` |
| Raw content from stdin | `curl -s https://target.tld/main.js \| miner` |

**Example**

```console
$ miner app.js
app.js
    api_key:"9f8e7d6c5b4a39281706
    AIzaSyA1b2C3d4E5f6G7h8I9j0K1l2M3n4O5p6Q
    MONGO_URI=mongodb+srv://admin:pass@cluster0.example.net/prod
    https://assets-prod.s3.us-east-1.amazonaws.com/logo.png
```

## 🔗 Recon Pipelines

miner reads from `stdin`, so it plugs into the tools you already use:

```bash
# Every JavaScript file from archived and crawled URLs
cat domains.txt | gau | grep -iE "\.js(\?|$)" | sort -u | miner - -t 50

# Hunt for exposed environment files on live hosts
cat live_hosts.txt | sed 's#/*$#/.env#' | miner - -timeout 8

# Save findings for triage
miner -f js_urls.txt -t 40 | tee miner_results.txt
```

## 🔍 What It Detects

| Category | Examples |
|---|---|
| **Cloud: AWS** | Access key IDs (`AKIA…`, `ASIA…`), `AWS_SECRET_ACCESS_KEY`, `AWS_SESSION_TOKEN`, S3 bucket URLs (virtual-hosted, path-style, `s3://`), `S3_BUCKET_*` |
| **Cloud: GCP / Azure** | Google API keys (`AIza…`), `GCS_BUCKET`, `AZURE_CLIENT_SECRET`, `AZURE_STORAGE_CONNECTION_STRING` |
| **Source control** | GitHub tokens (`ghp_`, `gho_`, `ghu_`, `ghs_`, `ghr_`), GitLab tokens (`glpat-`) |
| **SaaS &amp; payments** | Stripe live keys (`sk_live_`, `pk_live_`), Slack tokens (`xox…`), SendGrid keys (`SG.`), Contentful preview tokens |
| **Databases** | `MONGO_URI`, `REDIS_URL`, `DB_PASSWORD` |
| **Auth &amp; crypto** | JSON Web Tokens, `JWT_SECRET`, RSA/OpenSSH private key headers, UUID-style `apiKey` values |
| **Generic key/value** | `secret`, `token`, `password`, `api_key`, `access_token`, `client_id`, `client_secret`, `auth_key`, `session_token`, `ssh_key`, `encrypt_key` and more |
| **Misconfiguration** | `DEBUG=true`, `ADMIN_PASSWORD`, `SUPERUSER_TOKEN` |

> Regex-based detection produces candidates, not confirmed leaks. Always validate a finding (is the key live? what is its scope?) before reporting it.

## 🗺️ Roadmap

- [ ] De-duplicate overlapping matches per source
- [ ] JSON / JSONL output (`-json`) for automation
- [ ] Silent mode and per-pattern labels (`[aws-key]`, `[jwt]`, …)
- [ ] Custom pattern files (`-p patterns.yaml`)
- [ ] Optional live-key validation for common providers
- [ ] Custom headers and cookies for authenticated scanning

Contributions, pattern suggestions and issues are welcome. Open a PR or an issue.

## ⚠️ Legal Disclaimer

This tool is intended for **authorized security testing, bug bounty programs within their published scope, and educational
purposes only**. You are responsible for complying with all applicable laws and program rules. Report what you find
through responsible disclosure; never use discovered credentials beyond what is needed to demonstrate impact.

## 👤 Author

**DedCrowd** · Offensive security researcher · NASA VDP Letter of Appreciation · Deutsche Telekom Hall of Fame

[GitHub](https://github.com/dedcrowd) · [Bugcrowd](https://bugcrowd.com/h/DedCrowd) · [LinkedIn](https://linkedin.com/in/dedcrowd)

If miner helped you find something, consider leaving a ⭐. It helps others discover the project.
