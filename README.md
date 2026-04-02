# PriceWatch AI

**SaaS Competitive Intelligence on Autopilot**

Automated daily monitoring of competitor pricing pages, changelogs, and feature updates. Get email alerts before your customers notice.

🔗 **Live Demo:** https://mmaliabbas.github.io/pricewatch-ai/

---

## What It Does

PriceWatch AI automatically:
1. **Fetches** competitor pricing pages daily
2. **Extracts** pricing data (plans, prices, features)
3. **Diffs** against previous snapshots
4. **Alerts** you via email when changes are detected

No dashboard. No login. Just daily intelligence reports in your inbox.

---

## How It Works

```
┌─────────────────┐     ┌──────────────┐     ┌─────────────┐
│  Landing Page   │────▶│  Sign-up     │────▶│  Customers  │
│  (GitHub Pages) │     │  Formspree   │     │  (SQLite)   │
└─────────────────┘     └──────────────┘     └─────────────┘
                              │
                              ▼
┌─────────────────┐     ┌──────────────┐     ┌─────────────┐
│  Email Reports  │◀────│  Pipeline    │────▶│  Targets    │
│  (SMTP/Gmail)   │     │  (Daily)     │     │  (SQLite)   │
└─────────────────┘     └──────────────┘     └─────────────┘
                              │
                              ▼
                        ┌──────────────┐
                        │  Stripe      │
                        │  (Billing)   │
                        └──────────────┘
```

### Daily Flow

1. **3 AM PKT** — Pipeline wakes up via cron
2. **Fetch** — Scrapes pricing pages for all tracked competitors
3. **Extract** — Parses HTML → structured JSON
4. **Diff** — Compares against yesterday's snapshot
5. **Report** — Generates markdown report with changes
6. **Email** — Sends report to customers (only if changes detected)
7. **Log** — Records delivery status in database

---

## Pricing

| Plan | Price | Competitors | Features |
|------|-------|-------------|----------|
| **Starter** | $49/mo | Up to 5 | Daily monitoring, email reports |
| **Growth** | $149/mo | Up to 20 | + Changelog tracking, Slack alerts |
| **Scale** | $299/mo | Unlimited | + Custom branding, priority support |

---

## Tech Stack

| Component | Technology |
|-----------|------------|
| **Database** | SQLite (WAL mode) |
| **Backend** | Python 3.12 |
| **Email** | SMTP (Gmail/Resend) |
| **Billing** | Stripe |
| **Hosting** | GitHub Pages (landing) |
| **Pipeline** | Cron + Python scripts |

### Key Scripts

| Script | Purpose |
|--------|---------|
| `db.py` | SQLite database layer with audit trail |
| `fetch.py` | Config-driven URL fetcher with retry logic |
| `extract_pricing.py` | HTML → JSON extraction |
| `diff_pricing.py` | Compare pricing snapshots |
| `diff_changelog.py` | Detect new changelog entries |
| `run_pipeline.py` | Full pipeline orchestration |
| `send_report.py` | Email delivery with HTML conversion |
| `add_customer.py` | Customer onboarding |
| `stripe_billing.py` | Stripe integration + webhooks |
| `test_db.py` | 21 database tests (in-memory) |

---

## Quick Start (Self-Host)

### Prerequisites

- Python 3.12+
- SQLite3 (built-in)
- Stripe account
- SMTP credentials (Gmail app password or Resend)

### Installation

```bash
# Clone the repo
git clone https://github.com/MMAliAbbas/aliai-config.git
cd aliai-config/workspace/projects/income-engine/opportunities/saas-ci-monitor

# Install dependencies
pip install stripe beautifulsoup4 python-dotenv

# Copy environment template
cp .env.example .env

# Edit .env with your credentials
nano .env
```

### Configure .env

```bash
# SMTP (email delivery)
SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_USER=your-email@gmail.com
SMTP_PASS=your-app-password

# Stripe (billing)
STRIPE_SECRET_KEY=sk_test_xxx
STRIPE_WEBHOOK_SECRET=whsec_xxx
```

### Create Stripe Products

```bash
python3 scripts/stripe_billing.py create-products
```

Save the price IDs to `.env`.

### Set Up Daily Cron

```bash
crontab -e

# Add this line (runs at 3 AM PKT / 22:00 UTC)
0 22 * * * cd /path/to/saas-ci-monitor && python3 scripts/run_pipeline.py >> logs/pipeline.log 2>&1
```

### Test the Pipeline

```bash
# Run manually
python3 scripts/run_pipeline.py --demo

# Send test report
python3 scripts/send_report.py data/reports/ci-report_*.md --to your@email.com
```

### Add First Customer

```bash
python3 scripts/add_customer.py \
  --email "customer@company.com" \
  --name "John Doe" \
  --plan growth \
  --competitors "https://vercel.com/pricing,https://linear.app/pricing"
```

---

## Database Schema

### customers
| Column | Type | Description |
|--------|------|-------------|
| id | TEXT | UUID primary key |
| email | TEXT | Unique customer email |
| name | TEXT | Customer name |
| plan | TEXT | starter/growth/scale |
| status | TEXT | active/paused/cancelled/trial |
| stripe_customer_id | TEXT | Stripe customer ID |
| stripe_subscription_id | TEXT | Stripe subscription ID |
| api_key | TEXT | 32-char API key |
| max_competitors | INTEGER | Plan limit (5/20/999) |
| created_at | TEXT | ISO timestamp |
| updated_at | TEXT | ISO timestamp |

### customer_targets
| Column | Type | Description |
|--------|------|-------------|
| id | TEXT | UUID primary key |
| customer_id | TEXT | FK to customers |
| target_name | TEXT | Display name |
| pricing_url | TEXT | Pricing page URL |
| changelog_url | TEXT | Changelog URL (optional) |
| enabled | INTEGER | 1=active, 0=disabled |

### reports
| Column | Type | Description |
|--------|------|-------------|
| id | TEXT | UUID primary key |
| customer_id | TEXT | FK to customers |
| report_path | TEXT | Path to markdown report |
| created_at | TEXT | ISO timestamp |
| delivered_at | TEXT | ISO timestamp (when sent) |
| delivery_status | TEXT | pending/sent/failed |

### audit_log
| Column | Type | Description |
|--------|------|-------------|
| id | INTEGER | Auto-increment |
| timestamp | TEXT | ISO timestamp |
| actor | TEXT | system/api/admin |
| action | TEXT | create/update/delete/etc |
| entity_type | TEXT | customer/target/report |
| entity_id | TEXT | Related entity ID |
| details | TEXT | JSON details |

---

## API (Future)

PriceWatch AI will expose a REST API for:
- Customer self-service (add/remove competitors)
- Report retrieval (JSON format)
- Webhook notifications (Slack, Discord)

**Planned endpoints:**
- `GET /api/reports/latest` — Latest report for customer
- `POST /api/targets` — Add competitor to track
- `DELETE /api/targets/:id` — Remove competitor
- `GET /api/stats` — Usage statistics

---

## Security

- **SQLite WAL mode** — Concurrent reads, crash-safe writes
- **Audit trail** — All operations logged with actor + timestamp
- **Webhook idempotency** — Stripe events processed exactly once
- **API keys** — 32-char SHA256 hashes per customer
- **No sensitive data in logs** — Credentials in .env only

---

## Development

### Run Tests

```bash
# Database tests (21 tests, in-memory SQLite)
python3 scripts/test_db.py --verbose

# Integration tests
python3 scripts/test_pipeline.py
```

### Project Structure

```
saas-ci-monitor/
├── config/
│   ├── targets.json       # Demo targets (not customer data)
│   └── stripe.json        # Stripe product/price IDs
├── scripts/
│   ├── db.py              # SQLite database layer
│   ├── fetch.py           # URL fetcher with retry
│   ├── extract_pricing.py # HTML → JSON
│   ├── diff_pricing.py    # Pricing snapshot diff
│   ├── diff_changelog.py  # Changelog entry diff
│   ├── run_pipeline.py    # Full pipeline orchestration
│   ├── send_report.py     # Email delivery
│   ├── add_customer.py    # Customer onboarding
│   ├── stripe_billing.py  # Stripe integration
│   ├── test_db.py         # 21 database tests
│   └── test_pipeline.py   # Integration tests
├── data/
│   ├── pricewatch.db      # SQLite database
│   ├── raw/               # HTML snapshots + JSON extracts
│   └── reports/           # Generated reports
├── site/
│   └── index.html         # Landing page (GitHub Pages)
├── .env.example           # Environment template
├── README.md              # This file
└── DEPLOYMENT_CHECKLIST.md # Launch checklist
```

---

## Roadmap

### Q2 2026
- [ ] Customer self-service portal (add/remove competitors)
- [ ] Slack/Discord webhook alerts
- [ ] API access for Growth/Scale plans
- [ ] Status page monitoring (uptime, incidents)

### Q3 2026
- [ ] Feature-level diff (not just pricing)
- [ ] Historical trend charts
- [ ] Competitor comparison reports
- [ ] White-label reports (Scale plan)

---

## License

MIT License — see LICENSE file.

---

## Contact

**Built by:** AliAI  
**Website:** https://mmaliabbas.github.io/pricewatch-ai/  
**Email:** support@pricewatch.ai (configure in .env)

---

*PriceWatch AI — Know when your competitors change their pricing.*
