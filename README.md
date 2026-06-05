<div align="center">

# 🚀 Postiz on Kubernetes

### Production-ready Kustomize manifests to self-host [Postiz](https://github.com/gitroomhq/postiz-app) — open-source multi-channel social scheduling — on any K8s cluster.

[![License: MIT](https://img.shields.io/badge/manifests-MIT-green)](LICENSE)
[![Postiz](https://img.shields.io/badge/Postiz-AGPL--3.0-blue)](https://github.com/gitroomhq/postiz-app)
[![Kustomize](https://img.shields.io/badge/Kustomize-base%20%2B%20overlays-326ce5)](https://kustomize.io)

</div>

Self-hosted social posting → **no SaaS dependency**, EU-sovereign / BYOC-friendly.
Official image → **no build, deploy-only**. Secret-free generic `base/` + an example
overlay published **for transparency on a real setup** ([WebRobot](https://webrobot.eu)).
Reusable by anyone.

## Layout (kustomize base + overlays)
```
infra/postiz/
├── base/                     # GENERIC, SECRET-FREE — safe to publish / reuse
│   ├── deployment.yaml       #   official image, port 5000, /uploads PVC
│   ├── service.yaml  configmap.yaml  pvc.yaml  ingress.yaml(optional)
│   ├── secret.example.yaml   #   TEMPLATE → copy to secret.yaml (gitignored)
│   └── kustomization.yaml     #   no Secret included; bring your own
└── overlays/
    └── webrobot/             # OUR deployment (transparency): namespace + DB wiring
        ├── kustomization.yaml
        ├── patch-deployment.yaml   # admin DB password from postgres-nuvolaris-secret
        ├── patch-configmap.yaml    # our host + cluster Redis
        └── secret.yaml             # JWT (+OAuth) — GITIGNORED, not published
```
No real credentials live in git: `**/secret.yaml` is gitignored; only
`secret.example.yaml` is tracked.

## Deploy (generic)
```bash
cp base/secret.example.yaml base/secret.yaml   # fill JWT_SECRET + DATABASE_URL
# add `- secret.yaml` to base/kustomization.yaml resources, then:
kubectl apply -k infra/postiz/base
```

## Deploy (WebRobot overlay)
```bash
# 1. Create a dedicated DB on the nuvolaris Postgres (NOT webrobotdb):
#    CREATE DATABASE postiz;        (admin user `postgres`)
# 2. cp base/secret.example.yaml overlays/webrobot/secret.yaml  → set JWT_SECRET
#    (DATABASE_URL is injected by patch-deployment.yaml from postgres-nuvolaris-secret)
# 3. Apply (runs in the nuvolaris namespace):
kubectl apply -k infra/postiz/overlays/webrobot
# 4. Expose postiz.webrobot.eu — a Traefik ingress is included (overlays/webrobot/ingress.yaml).
```
Uses the existing **nuvolaris Redis** (`redis.nuvolaris.svc.cluster.local`, password
from its `redis-cm`, set inside the gitignored secret) and the **nuvolaris Postgres**
admin secret — no password copied for Postgres (secretKeyRef + `$(VAR)`). A **Temporal**
server (`temporalio/auto-setup`, base/temporal.yaml) is deployed too — recent Postiz
requires it for scheduling; it provisions its schema on the same Postgres on first boot.

## After first boot
- Create the first admin user, then set `DISABLE_REGISTRATION: "true"`.
- Add per-channel OAuth app credentials to the Secret as each platform app is approved
  (`X_*`, `LINKEDIN_*`, `FACEBOOK_*`, `MASTODON_*`, …). **The layer cannot post to a
  channel without that platform's approved app + OAuth tokens** — open-source does not
  bypass platform API access.

## Notes
- Image is `:latest` for the first deploy — **pin to a released tag** for reproducibility.
- All-in-one container serves on **port 5000** (frontend + backend + BullMQ workers + cron).
- AGPL-3.0 (Postiz): review copyleft before exposing a modified Postiz as a product feature.
- Integration: Postiz has a public API → call it from the Ray marketing/leadgen agents,
  or wrap it as an internal MCP (same HITL-gated pattern as the old Zernio MCP).
