## 3.2 Contact Harvesting

Outline a module to collect and validate contact information associated with discovered organizations.

- **Crawling approach**: Use headless browsers and APIs to crawl LinkedIn company pages, employee directories, and public corporate sites. Respect robots.txt and rate-limit requests. Support both keyword-based discovery and domain-focused crawling to expand organization profiles.
- **Email permutation + SMTP probing**: Generate likely email patterns (e.g., `{first}.{last}@domain`, `{f}{last}@domain`) and verify deliverability via SMTP VRFY and RCPT checks without sending actual messages. Capture successful permutations for enrichment.
- **Data-cleansing rules**: Normalize names, deduplicate entries, and reject addresses failing MX lookup or SMTP probes. Flag disposable or role-based emails (info@, support@) to reduce false positives.
- **Enrichment integration**: Plug into services like Clearbit and Hunter.io to append job titles, social links, and confidence scores to verified contacts.

## 3.3 Vulnerability Aggregation

Describe a pipeline that consolidates vulnerability intelligence for discovered assets.

- **Active scans vs. passive feeds**: Schedule Shodan and Censys queries for exposed services while ingesting passive CVE feeds (NVD, vendor advisories). Balance frequency to avoid rate limits and redundant scanning.
- **CVSS-based scoring**: For each vulnerability, compute CVSS v3 base scores and adjust with environmental factors (asset criticality, exposure). Store normalized risk scores per asset.
- **Scan scheduling**: Use cron-based jobs for periodic Shodan/Censys lookups and webhooks or daily pulls for passive feeds. Allow configurable intervals per asset group.
- **Normalization & storage**: Standardize fields (CVE ID, service, version, score) and persist in a relational table or Elasticsearch index optimized for filtering and correlation.

## 3.4 Enrichment & Correlation

Propose a service that correlates vulnerabilities, indicators of compromise (IoCs), and organizational context.

- **ATT&CK mapping**: Map each CVE or IoC to relevant MITRE ATT&CK techniques using maintained reference tables.
- **Per-asset risk scoring**: Combine vulnerability scores, threat intel, and business impact to compute a composite risk score per asset.
- **High-risk alerting**: Trigger immediate alerts (email, webhook, or queue) when assets exceed a configurable risk threshold.
- **Correlation schema**: Maintain tables linking assets, CVEs, IoCs, ATT&CK techniques, and enrichment metadata with timestamps for auditability.

## 3.5 Dashboard & Reporting

Sketch UX and API features for the platform's web dashboard.

- **Inventory & search**: Interactive tables with filters, sorting, and heatmaps showing asset exposure and risk levels.
- **Report generation**: REST endpoints to produce PDF and CSV reports for selected assets or time ranges.
- **Scheduled summaries**: Background jobs that email weekly or monthly summaries to stakeholders.
- **Auth & RBAC**: OAuth2/SAML authentication with role-based access to views (admin, analyst, read-only).

## 4. Architecture & Tech Stack

A high-level view of platform components and technologies.

```
[React + Tailwind]  -->  [Node.js/Express API]  -->  [PostgreSQL]
                                   |              [Elasticsearch]
                            [Python microservices]
                                   |              [Redis]
                             [RQ/Celery workers]
                                   |
                             [K8s CronJobs]
```

- **Frontend**: React with Tailwind CSS for fast UI development.
- **API**: Node.js/Express serving REST and report generation endpoints.
- **Microservices**: Python services for discovery, enrichment, and vulnerability scanning, using shared libraries and message queues.
- **Datastores**: PostgreSQL for relational data, Elasticsearch for search/analytics, Redis for caching and task queues.
- **Orchestration**: Kubernetes CronJobs for scheduled tasks and RQ/Celery for asynchronous workloads.

## 5. Security & Compliance Checklist

- TLS for all inbound/outbound connections; encrypt sensitive data at rest with AES-256.
- Centralized secrets management via Vault or Kubernetes Secrets with tight RBAC.
- Immutable audit logs stored in append-only buckets or WORM storage.
- User privacy controls allowing per-source opt-out and data deletion on request.
- Document handling of harvested personal data to satisfy GDPR/CCPA requirements.

## 6. Phased Roadmap

```yaml
Phase 1:
  timeline: Q1
  deliverables:
    - Domain & email discovery
    - Vulnerability feed ingestion
    - Basic web UI
Phase 2:
  timeline: Q2
  deliverables:
    - Dark-web leak monitoring
    - Risk scoring engine
    - MITRE ATT&CK mapping
    - PDF report generation
Phase 3:
  timeline: Q3
  deliverables:
    - Multi-tenant scalability
    - Alerting engine
    - Role-based access control
Phase 4:
  timeline: Q4
  deliverables:
    - Social-media OSINT modules
    - Anomaly detection models
    - Public API for integrations
```

## 7. Kick-off & Next Steps

1. Draft and circulate a requirements workshop agenda covering target users, data sources, and compliance constraints.
2. Acquire API keys for Shodan, Censys, Clearbit, Hunter.io, and other planned data sources.
3. Build a proof-of-concept pipeline for Phase 1, ingesting domains and enriching emails end-to-end.
4. Review and iterate on UX wireframes with stakeholders.
5. Set up development environments, repositories, and CI/CD pipelines.
