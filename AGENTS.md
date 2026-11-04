# AGENTS.md — pod-crabbox-gpu

The virtual-GPU lease image for the Crabbox `containerd` provider: the `pod-crabbox-gpu` image
(`charly.yml`) and the variant layer it composes. The lease contract itself belongs
to the base image's repo; this repo owns only the variant delta.

Canonical files:

- `charly.yml` — the image: its pinned base image, user/entrypoint, and candy list.
- `candy/<layer>/charly.yml` — the variant's packages, plan steps and `check:` steps.
- `.github/workflows/image.yml` — the image build; it triggers on
  `tag-on-merge` **completion** (`workflow_run`), never on `push: tags`
  (`opencharly/.github#277`).
- `.github/workflows/tag-on-merge.yml` — the org's CalVer tag + `CHANGELOG/`
  dispatcher, installed by the org bootstrap.
- `README.md` — user overview only; never agent guidance.

## Load these skills first (R0)

- `/charly-image:image` — the `candy:` image entry (base, user, entrypoint).
- `/charly-image:layer` — the candy body: `distro:` packages, plan step verbs,
  and the `check:` surface.
- `/charly-distros:cachyos` — the CachyOS base and Arch package semantics.
- `/charly-check:check` — the check verbs, the ADE requirement, and the R10 gate.
- `/charly-internals:git-workflow` — before any git/PR action.

## What is unique here

- **No NVIDIA userspace here.** On krun the host's Vulkan loader runs in the render
  server; on crun/runc the NVIDIA userspace is injected by CDI. Adding `nvidia-utils`
  would pin the image to one host driver.
- `vulkan-virtio` is Arch's Venus ICD (`/usr/share/vulkan/icd.d/virtio_icd.json`);
  `vulkan-swrast` is the CPU fallback. Mesa no longer ships Vulkan ICDs, which is why
  they are separate packages.
- **Not claimed:** a mesa build carrying `amdgpu-virtio` for DRM native context. It is
  not a named package in the Arch/CachyOS repos, so this image does not claim it.

## Build / validate / test

- `charly box validate` at the repo root — the structural check on the image and
  its candy (including the pulled base image at its pinned tag).
- `charly box generate pod-crabbox-gpu` — inspect the emitted Containerfile and the baked
  `ai.opencharly.description` plan.
- `charly check box pod-crabbox-gpu` — the build-scope `check:` steps.
- The PR image build (`.github/workflows/image.yml`, `pull_request` path) builds
  the image and publishes nothing; the merge-time `workflow_run` path publishes
  `ghcr.io/opencharly/pod-crabbox-gpu:<git-tag>`.

## Modify this repo

- Keep the base tag in step: a base-contract fix lands in `opencharly/pod-crabbox` first,
  then this repo's `base:` tag advances (producer-first). The base is a pinned OCI
  artifact, not an `import:`; `charly box reconcile` does not apply to it.
- A package may be added only from a named source (the base candy, an existing
  `layer-*`/`pod-*` repo's Arch section, the provider's docs, or the Arch/AUR
  package index). Record the source in a comment.
- Every claim in the plan needs a deterministic `check:`. A package that merely
  installs without a probe is not proof.
- Never weaken a sandbox boundary: no `seccomp=unconfined`, no `--no-sandbox`.

## Landing

- PR-only. Every change lands through a pull request; the org-required
  `charly/pr-validator` validates the diff and body and arms native auto-merge on
  PASS. Direct pushes to `main` are blocked.
- History lives in `CHANGELOG/` (written by `tag-on-merge` at merge time); the PR
  body IS the changelog.
- The authoritative rulebook is the umbrella `AGENTS.md` in
  `opencharly/opencharly`. Do not restate its rules here.
