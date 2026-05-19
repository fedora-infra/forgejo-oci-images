# OCI Image Definitions

Image definitions for Forgejo deployment in Fedora Infrastructure.

This repository contains Dockerfiles for building OCI images used by the
Fedora Infrastructure team. Konflux builds images directly from this
[Codeberg repository](https://codeberg.org/fedora/oci-image-definitions) on the
[Fedora Konflux instance](https://konflux-ci.fedoraproject.org/).

## Branching and release strategy

This repository has 5 branches:

| Branch | Purpose |
|--------|---------|
| `main` | Regularly rebased on upstream Forgejo LTS releases with Fedora-specific commits cherry-picked on top |
| `forge-staging` | Forge staging image builds |
| `forge-production` | Forge production image builds |
| `src-staging` | Src staging image builds |
| `src-production` | Src production image builds |

We track only Forgejo **LTS releases** to ensure long-term stability and
security support. Pushing to any of these branches (except `main`) triggers a Konflux
image build and outputs the image to Quay. Only the image corresponding to
the target branch is built. Images are tagged with a composite version:
`{upstream forgejo version}-{fedora/forgejo last commit SHA}-{oci-image-definitions commit SHA}`.

Images must follow this deployment path:

1. **Build** — push to a staging branch to trigger a Konflux image build
2. **Local testing** — pull the built staging image from Quay and verify it locally before deploying
3. **Staging deployment** — deploy the verified image to the staging environment and test there
4. **Production** — only after staging is validated, push to the production branch for the final build and deployment

## Source images

Two websites are built from this repo, each in staging and production
variants, each for stable and rawhide Fedora:

| Website | Directory | Description |
|---------|-----------|-------------|
| **forge** | `forgejo/` | Forge instance |
| **src** | `distgit/` | Dist-git instance |

Each website directory contains:

```
<website>/
├── staging/
│   ├── Dockerfile.staging_rawhide
│   ├── Dockerfile.staging_stable
│   └── VERSION
└── production/
    ├── Dockerfile.prod_rawhide
    ├── Dockerfile.prod_stable
    └── VERSION
```

This produces **8 images** total (2 websites x 2 environments x 2 Fedora versions).

## Tekton pipelines

The `.tekton/` directory contains hand-crafted PipelineRun definitions for
Pipelines-as-Code (PaC). Each image has a push and pull-request pipeline,
triggered on relevant path changes.

## Konflux tenant configuration

This repo is onboarded to the `fedora-infra-tenant` namespace on the
[kflux-fedora-01](https://konflux-ci.fedoraproject.org/) Konflux cluster.
The tenant configuration lives in the
[tenants-config](https://gitlab.com/fedora/infrastructure/konflux/tenants-config)
GitLab repo at `clusters/kflux-fedora-01/tenants/fedora-infra-tenant/`.

The tenant uses Kustomize-based
[Configuration-as-Code](https://konflux-ci.dev/docs/building/configuration-as-code/)
with bases and overlays to define 4 Applications and 8 Components from a
single shared component template:

```
fedora-infra-tenant/
├── kustomization.yaml              # Top-level: ns, rbac, quota, applications
├── ns.yaml                         # Namespace definition
├── rbac.yaml                       # RBAC: admin users, viewer access
└── applications/
    ├── kustomization.yaml          # Aggregates forge + src
    ├── base/
    │   ├── component.yaml          # Shared component template (git URL,
    │   │                           #   provider, pipeline — common to all 8)
    │   └── kustomization.yaml
    ├── forge/                      # Forge website application
    │   ├── kustomization.yaml      # Aggregates staging + production
    │   ├── base/
    │   │   ├── application.yaml    # Base Application resource
    │   │   ├── rawhide/            # Component overlay: sets name to forge-rawhide
    │   │   └── stable/             # Component overlay: sets name to forge-stable
    │   ├── staging/                # Environment overlay → Application: forge-staging
    │   │   ├── application-patch.yaml
    │   │   ├── component-patch.yaml
    │   │   ├── rawhide-patch.yaml
    │   │   └── stable-patch.yaml
    │   └── production/             # Environment overlay → Application: forge-production
    │       └── ...                 # (same patch structure as staging)
    └── src/                        # Src website application (mirrors forge/)
        └── ...                     # (same structure, using distgit/ paths)
```

### Kustomize layering

The 8 components are produced through 3 layers of Kustomize overlays:

1. **Shared base** (`applications/base/component.yaml`) — git repository URL,
   provider annotations, pipeline, and revision common to all components.

2. **Per-website / per-component** (`forge/base/rawhide/`, `src/base/stable/`,
   etc.) — component name that distinguishes rawhide from stable.

3. **Per-environment** (`forge/staging/`, `src/production/`, etc.) — application
   name, build context path, and Dockerfile for each staging/production variant.

### Rendered resources

| Application | Component | Context | Dockerfile |
|---|---|---|---|
| `forge-staging` | `forge-rawhide-staging` | `forgejo/staging` | `Dockerfile.staging_rawhide` |
| `forge-staging` | `forge-stable-staging` | `forgejo/staging` | `Dockerfile.staging_stable` |
| `forge-production` | `forge-rawhide-production` | `forgejo/production` | `Dockerfile.prod_rawhide` |
| `forge-production` | `forge-stable-production` | `forgejo/production` | `Dockerfile.prod_stable` |
| `src-staging` | `src-rawhide-staging` | `distgit/staging` | `Dockerfile.staging_rawhide` |
| `src-staging` | `src-stable-staging` | `distgit/staging` | `Dockerfile.staging_stable` |
| `src-production` | `src-rawhide-production` | `distgit/production` | `Dockerfile.prod_rawhide` |
| `src-production` | `src-stable-production` | `distgit/production` | `Dockerfile.prod_stable` |

### References

- [Konflux CaC best practices](https://konflux-ci.dev/docs/building/configuration-as-code/)
- [Onboarding a Forgejo component](https://konflux-ci.dev/docs/building/creating-forgejo/)
- [Fedora tenants-config repo](https://gitlab.com/fedora/infrastructure/konflux/tenants-config)
- [ADR 52: GitOps Onboarding Redesign](https://konflux-ci.dev/architecture/ADR/0052-gitops-onboarding-redesign/)

---

*This README was authored by Claude (Anthropic), May 2026.*
