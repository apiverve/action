# APIVerve GitHub Action

> Access 350+ production-ready REST APIs directly in your GitHub workflows.

> **Beta Release** - This action is in beta. We'd love your feedback! [Open an issue](https://github.com/apiverve/action/issues) if you encounter any problems.

[![GitHub Marketplace](https://img.shields.io/badge/Marketplace-APIVerve-blue?logo=github)](https://github.com/marketplace/actions/apiverve)
[![APIs](https://img.shields.io/badge/APIs-350+-blue.svg)](https://apiverve.com/marketplace?utm_source=github&utm_medium=action&utm_campaign=readme)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

**[Browse All APIs](https://apiverve.com/marketplace?utm_source=github&utm_medium=action&utm_campaign=readme)** | **[Get Free API Key](https://dashboard.apiverve.com/signup?utm_source=github&utm_medium=action&utm_campaign=readme)** | **[Documentation](https://docs.apiverve.com?utm_source=github&utm_medium=action&utm_campaign=readme)**

---

## What is APIVerve?

[APIVerve](https://apiverve.com?utm_source=github&utm_medium=action&utm_campaign=readme) is a platform that provides 350+ production-ready REST APIs for developers. Instead of building and maintaining common functionality yourself, you can call our APIs to handle tasks like:

- **Generating QR codes and barcodes** for your releases and documentation
- **Capturing website screenshots** for visual testing and previews
- **Looking up DNS records, SSL certificates, and domain info** for monitoring
- **Validating emails, phone numbers, and other data** before processing
- **Converting documents** (HTML to PDF, Markdown to PDF, etc.)
- **Checking IP geolocation and reputation** for security workflows

All APIs are fast, reliable, and designed for developer workflows - including CI/CD pipelines.

---

## What does this GitHub Action do?

This action lets you call any of APIVerve's 350+ APIs directly from your GitHub workflows. With a single action, you can:

- **Generate assets** - Create QR codes, screenshots, PDFs, and more as part of your build
- **Validate and verify** - Check DNS propagation, SSL certificates, email addresses
- **Monitor infrastructure** - Track domain expiration, DNS records, SSL status
- **Enrich data** - Look up IP locations, parse phone numbers, detect languages
- **Automate checks** - Spell check docs, validate content, run security checks

One action. 350+ APIs. No separate integrations needed.

---

## Quick Start

```yaml
- name: Generate QR Code
  uses: apiverve/action@v1
  with:
    api_key: ${{ secrets.APIVERVE_KEY }}
    api: qrcodegenerator
    params: '{"value": "https://github.com/${{ github.repository }}", "size": 300}'
    output_file: ./qrcode.png
```

---

## Setup

### 1. Get Your API Key

Sign up for a free account at [dashboard.apiverve.com/signup](https://dashboard.apiverve.com/signup?utm_source=github&utm_medium=action&utm_campaign=readme) and create an API key.

### 2. Add Secret to Repository

Go to your repository **Settings** → **Secrets and variables** → **Actions** → **New repository secret**

- Name: `APIVERVE_KEY`
- Value: Your API key from the dashboard

### 3. Use in Workflow

**Option A: Pass API key per step**
```yaml
- name: Call APIVerve
  uses: apiverve/action@v0
  with:
    api_key: ${{ secrets.APIVERVE_KEY }}
    api: dnslookup
    params: '{"domain": "example.com"}'
```

**Option B: Set API key once via environment variable**
```yaml
jobs:
  example:
    runs-on: ubuntu-latest
    env:
      APIVERVE_API_KEY: ${{ secrets.APIVERVE_KEY }}
    steps:
      - name: Check DNS
        uses: apiverve/action@v0
        with:
          api: dnslookup
          params: '{"domain": "example.com"}'

      - name: Check SSL
        uses: apiverve/action@v0
        with:
          api: sslchecker
          params: '{"domain": "example.com"}'
```

---

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `api_key` | Your APIVerve API key (or set `APIVERVE_API_KEY` env var) | Yes* | - |
| `api` | API to call (e.g., `qrcodegenerator`, `dnslookup`) | Yes | - |
| `params` | JSON parameters for the API | No | `{}` |
| `output_file` | Path to save binary output (images, PDFs) | No | - |
| `format` | Response format: `json`, `yaml`, or `xml` | No | `json` |
| `fail_on_error` | Fail workflow if API returns error | No | `true` |

*\*API key is required but can be provided via input OR `APIVERVE_API_KEY` / `APIVERVE_KEY` environment variable.*

## Outputs

| Output | Description |
|--------|-------------|
| `result` | Full API response as JSON |
| `data` | The `data` field from response as JSON |
| `status` | API status (`ok` or `error`) |
| `file` | Path to downloaded file (if `output_file` was used) |

---

## Examples

### Generate QR Code for Releases

Automatically add a QR code linking to each release:

```yaml
name: Release with QR
on:
  release:
    types: [published]

jobs:
  add-qr:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Generate release QR code
        uses: apiverve/action@v1
        with:
          api_key: ${{ secrets.APIVERVE_KEY }}
          api: qrcodegenerator
          params: '{"value": "https://github.com/${{ github.repository }}/releases/tag/${{ github.ref_name }}"}'
          output_file: ./release-qr.png

      - name: Upload QR to release
        uses: softprops/action-gh-release@v1
        with:
          files: ./release-qr.png
```

### Monitor Domain Expiration

Get notified before your domains expire:

```yaml
name: Domain Monitor
on:
  schedule:
    - cron: '0 9 * * 1'  # Every Monday at 9am

jobs:
  check:
    runs-on: ubuntu-latest
    steps:
      - name: Check domain expiration
        id: domain
        uses: apiverve/action@v1
        with:
          api_key: ${{ secrets.APIVERVE_KEY }}
          api: domainexpiration
          params: '{"domain": "mycompany.com"}'

      - name: Alert if expiring soon
        if: fromJson(steps.domain.outputs.data).daysUntilExpiration < 30
        run: |
          echo "::warning::Domain expires in ${{ fromJson(steps.domain.outputs.data).daysUntilExpiration }} days!"
```

### Validate DNS After Deployment

Verify your DNS is configured correctly after infrastructure changes:

```yaml
- name: Check DNS propagation
  id: dns
  uses: apiverve/action@v1
  with:
    api_key: ${{ secrets.APIVERVE_KEY }}
    api: dnspropagation
    params: '{"domain": "api.mycompany.com", "type": "A"}'

- name: Verify propagation
  run: echo "Propagation: ${{ steps.dns.outputs.data }}"
```

### Screenshot Documentation Site

Capture screenshots for visual regression testing or previews:

```yaml
- name: Capture screenshot
  uses: apiverve/action@v1
  with:
    api_key: ${{ secrets.APIVERVE_KEY }}
    api: webscreenshots
    params: '{"url": "https://docs.example.com", "width": 1280, "height": 800}'
    output_file: ./docs-screenshot.png
```

### Check SSL Certificate Status

Monitor SSL certificate expiration:

```yaml
- name: Check SSL
  id: ssl
  uses: apiverve/action@v1
  with:
    api_key: ${{ secrets.APIVERVE_KEY }}
    api: sslchecker
    params: '{"domain": "mycompany.com"}'

- name: Alert if expiring
  if: fromJson(steps.ssl.outputs.data).daysUntilExpiration < 14
  run: echo "::error::SSL certificate expires soon!"
```

### Generate PDF Reports

Convert webpages or markdown to PDF:

```yaml
- name: Generate PDF
  uses: apiverve/action@v1
  with:
    api_key: ${{ secrets.APIVERVE_KEY }}
    api: websitetopdf
    params: '{"url": "https://example.com/report"}'
    output_file: ./report.pdf
```

---

## Popular APIs for CI/CD

| Category | APIs |
|----------|------|
| **QR & Media** | `qrcodegenerator`, `qrcodereader`, `webscreenshots`, `websitetopdf`, `barcodegenerator` |
| **DNS & Domain** | `dnslookup`, `dnspropagation`, `domainexpiration`, `domainavailability`, `whoislookup` |
| **SSL & Security** | `sslchecker`, `dnsseccheck`, `botdetector`, `ipblacklistlookup` |
| **Email Validation** | `emailvalidator`, `emaildisposablechecker`, `spfvalidator`, `dkimvalidator`, `dmarcvalidator` |
| **IP & Network** | `iplookup`, `ipdemographics`, `reversednslookup`, `asnlookup` |
| **Documents** | `htmltopdf`, `markdowntopdf`, `websitetopdf` |
| **Text & Content** | `spellchecker`, `grammarcheck`, `sentimentanalysis`, `contentfilter` |

**[Browse all 350+ APIs →](https://apiverve.com/marketplace?utm_source=github&utm_medium=action&utm_campaign=readme)**

---

## Pricing

- **Free tier** - Get started with generous free limits
- **Pro plans** - Higher rate limits and priority support for production use

Check out [pricing details](https://apiverve.com/pricing?utm_source=github&utm_medium=action&utm_campaign=readme).

---

## Resources

- **API Documentation**: [docs.apiverve.com](https://docs.apiverve.com?utm_source=github&utm_medium=action&utm_campaign=readme)
- **API Marketplace**: [apiverve.com/marketplace](https://apiverve.com/marketplace?utm_source=github&utm_medium=action&utm_campaign=readme)
- **Issues & Support**: [GitHub Issues](https://github.com/apiverve/action/issues)
- **Email**: support@apiverve.com

---

## License

MIT - see [LICENSE](LICENSE)

---

Built by [APIVerve](https://apiverve.com?utm_source=github&utm_medium=action&utm_campaign=readme) - 350+ APIs for developers
