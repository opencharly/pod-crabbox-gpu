# pod-crabbox-gpu

The **GPU lease image** for the Crabbox `containerd` provider — the base
[`pod-crabbox`](https://github.com/opencharly/pod-crabbox) lease contract with the
Venus Vulkan ICD, the software fallback ICD, and `vulkan-tools` for the krun render
path.

## What it provides

| Property | Value |
|---|---|
| Base | CachyOS base image `ghcr.io/opencharly/pod-crabbox:2026.307.1828` |
| Vulkan | `vulkan-virtio` (Venus), `vulkan-swrast`, `vulkan-tools` |
| krun | the host's Vulkan stack renders venus through the render server |
| crun/runc | no driver in the image — NVIDIA userspace arrives by CDI |

## How to use it

```sh
ssh crabbox@<lease> vulkaninfo --summary
# under krun: the GPU is the virtual venus device; no NVIDIA userspace is in the image
```

## Layout

- `charly.yml` — the `pod-crabbox-gpu` image (pinned base image + entrypoint).
- `candy/` — the variant's own layer: packages, files and `check:` steps.
- `.github/workflows/tag-on-merge.yml` — CalVer tag + `CHANGELOG/` on merge.
- `.github/workflows/image.yml` — builds on `pull_request` and publishes at the merge
  tag, so the image tag equals the git tag.
- `CHANGELOG/` — per-CalVer history.

## Related

- [`opencharly/pod-crabbox`](https://github.com/opencharly/pod-crabbox) — the lease
  base image this family builds on.
- [`CONTAINERD-IMAGES.md`](https://github.com/opencharly/opencharly/blob/main/CONTAINERD-IMAGES.md)
  §2 — the lease-image contract.
