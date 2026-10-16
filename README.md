# bazzite

Personal [BlueBuild](https://bluebuild.org) images derived from upstream
[Universal Blue / Bazzite](https://github.com/ublue-os/bazzite), one recipe per
variant. Each image is a thin layer on top of upstream Bazzite, so rebasing to
it keeps pulling Bazzite's own updates — you just get the extra baked-in config.

## Variants

| Recipe | Base image | Image |
| --- | --- | --- |
| `recipes/bazzite-gnome.yml` | `ghcr.io/ublue-os/bazzite-gnome` | `ghcr.io/margeyshah/bazzite-gnome` |
| `recipes/bazzite-gnome-nvidia-open.yml` | `ghcr.io/ublue-os/bazzite-gnome-nvidia-open` | `ghcr.io/margeyshah/bazzite-gnome-nvidia-open` |
| `recipes/bazzite-deck-gnome.yml` | `ghcr.io/ublue-os/bazzite-deck-gnome` | `ghcr.io/margeyshah/bazzite-deck-gnome` |
| `recipes/bazzite-deck-nvidia-gnome.yml` | `ghcr.io/ublue-os/bazzite-deck-nvidia-gnome` | `ghcr.io/margeyshah/bazzite-deck-nvidia-gnome` |

## What the image adds

Flatpak-first, no host `dnf`/`copr`:

- **`default-flatpaks`** — system GUI apps from Flathub (Element, VS Code).
- **`rpm-ostree`** — `podman-docker` only (the single rpm exception; Bazzite
  ships Podman, this makes the `docker` CLI target it — no Docker daemon).
- **`files`** — `files/system/*` copied into the image root.
- **`signing`** — cosign signing policy.

The apt-based CLI toolchain (rust/cargo, nvm, pyenv, starship, fzf, nvim, etc.)
is **not** baked in. On the Bazzite host it lives in an Ubuntu Distrobox created
by the dotfiles bootstrap (`setup/fresh_install_bazzite.sh` in the dotfiles repo).

## Build

Built by GitHub Actions (`.github/workflows/build.yml`): daily, on push, and via
manual dispatch. Requires the repository secret **`SIGNING_SECRET`** (the cosign
private key). The public key is committed as `cosign.pub`.

## Rebase a machine onto your image

Rebase once to the unsigned image to pull in the signing keys/policy, reboot,
then rebase to the signed image:

```bash
# 1. unsigned image (installs the signing policy)
rpm-ostree rebase ostree-unverified-registry:ghcr.io/margeyshah/bazzite-gnome:latest
systemctl reboot

# 2. signed image
rpm-ostree rebase ostree-image-signed:docker://ghcr.io/margeyshah/bazzite-gnome:latest
systemctl reboot
```

Swap `bazzite-gnome` for `bazzite-gnome-nvidia-open`, `bazzite-deck-gnome`, or
`bazzite-deck-nvidia-gnome` as appropriate.

After rebasing, run the dotfiles bootstrap on the host:

```bash
~/.scripts/setup/fresh_install_bazzite.sh
```

## Verify

```bash
cosign verify --key cosign.pub ghcr.io/margeyshah/bazzite-gnome
```

## Updating

Updates flow automatically: each daily build re-derives from the latest upstream
Bazzite, and your machine picks it up through the normal `rpm-ostree`/`uupd`
update path (`ujust update`).
