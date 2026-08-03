# Kora GitOps Manifests

Deployment manifests for the Kora ERP platform — **separate from the [Kora app monorepo](https://github.com/muretimiriti/Kora)**, per ADR-040. Engineers don't hand-edit this repo directly; the app monorepo's Tekton pipeline opens promotion PRs here automatically on a green, signed build (`open-promotion-pr` Task, `infra/tekton/tasks/tasks.yaml` in the app monorepo).

## Structure

- `charts/` — one Helm chart per deployable unit (19 backend services + 5 frontend apps), **flat** — not nested under `services/`/`apps/` subdirectories. This deviates from the app monorepo's `docs/Target_Repository_Structure.md` Section 9 as originally written: Kustomize's local Helm chart inflation (`helmGlobals.chartHome` + `helmCharts[].name`) breaks if a chart's `name` contains a path separator, verified directly (`kubectl kustomize --enable-helm`) while scaffolding this repo. Target_Repository_Structure.md Section 9 has been corrected to match.
- `environments/{dev,staging,uat,prod-africa}/kustomization.yaml` — one directory per environment (ADR-040: directories, not branches), each inflating every chart in `charts/` with environment-specific `replicaCount`/`resources`/`image.tag` via `valuesInline`. UAT and `prod-africa` are deliberately identical in sizing (ADR-032).

## Rendering locally

```bash
kubectl kustomize --enable-helm --load-restrictor LoadRestrictionsNone environments/dev
```

(`--load-restrictor LoadRestrictionsNone` is required — Kustomize's default load restrictor otherwise refuses to read `values.yaml` files outside the `environments/` directory it's invoked from.)

## Branching

One `main` branch only (ADR-040) — no `develop`, no `release/*`, no per-environment branches. Environment promotion happens via directory, not branch.

## Status

Scaffolded and locally verified (all 4 environments render cleanly, 24 Deployments each) 2026-08-03. **Not yet connected to anything real**: no GitHub remote, no ArgoCD `Application` pointed at it, no real container images published under `ghcr.io/muretimiriti/kora-*` yet. Full status: [CICD_Pipeline_Wiring_Status.md](https://github.com/muretimiriti/Kora/blob/main/docs/CICD_Pipeline_Wiring_Status.md) in the app monorepo.
