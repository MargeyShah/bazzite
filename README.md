# bazzite

Personal [BlueBuild](https://bluebuild.org) images derived from upstream
[Universal Blue / Bazzite](https://github.com/ublue-os/bazzite), one recipe per
variant. Each image is a thin layer on top of upstream Bazzite, so rebasing to
it keeps pulling Bazzite's own updates — you just get the extra baked-in config.

> **Installing from scratch?** See **[INSTALL.md](INSTALL.md)** (download stock
> Bazzite → Ventoy/USB → install → rebase onto this image → bootstrap).

## Variants

| Recipe | Base image | Image |
| --- | --- | --- |
| `recipes/bazzite-gnome.yml` | `ghcr.io/ublue-os/bazzite-gnome` | `ghcr.io/margeyshah/bazzite-gnome` |
| `recipes/bazzite-gnome-nvidia-open.yml` | `ghcr.io/ublue-os/bazzite-gnome-nvidia-open` | `ghcr.io/margeyshah/bazzite-gnome-nvidia-open` |
| `recipes/bazzite-deck-gnome.yml` | `ghcr.io/ublue-os/bazzite-deck-gnome` | `ghcr.io/margeyshah/bazzite-deck-gnome` |
| `recipes/bazzite-deck-nvidia-gnome.yml` | `ghcr.io/ublue-os/bazzite-deck-nvidia-gnome` | `ghcr.io/margeyshah/bazzite-deck-nvidia-gnome` |

## What the image adds

Flatpak-first, no host `dnf`/`copr`:

- **`default-flatpaks`** — system GUI apps from Flathub: Element, VS Code,
  **Signal Desktop**, Vesktop, Plexamp, osu! (lazer), Ludusavi, Heroic Games
  Launcher, Gear Lever (the AppImage manager), **Prism Launcher**, **Cemu**,
  **RPCS3**, **Dolphin**, and **XIVLauncher**. Apps without a Flathub
  package — **Helium browser** and **Sidra** — are installed as AppImages by the
  bootstrap script and integrated with Gear Lever. (**Playnite** is intentionally
  omitted: it has no Linux build; Steam Game Mode + Heroic + Ludusavi cover those
  roles.)
- **Gaming extras** (host, not baked in; installed automatically by the
  bootstrap, skip with `--no-gaming`):
  - **EmuDeck** — `ujust get-emudeck install`
  - **Decky Loader** — `ujust setup-decky install`
  - **Bazzite Buddy** — `ujust get-decky-bazzite-buddy install` (not in the
    store; installed via its ujust recipe)
  - **Decky plugins** (from the Decky store, hash-verified, into
    `~/homebrew/plugins`): CSS Loader, SteamGridDB, ProtonDB Badges, HLTB for
    Deck, TabMaster, PlayCount, IsThereAnyDeal for Deck, Decky LSFG-VK,
    AutoFlatpaks, Decky Terminal, Audio Loader, Game Theme Music, Quick Launch,
    Volume Mixer, EmuDecky, MusicControl. No config is synced between machines.
- **System extras** (host, not baked in; installed automatically by the
  bootstrap, skip with `--no-extras`) — applied via Bazzite's own targets:
  - **Proton Plus** — `ujust get-protonplus` (flatpak)
  - **Sunshine** — `ujust setup-sunshine enable-flatpak` (flatpak)
  - **Wake-on-LAN** — `ujust wol enable`
  - **Boot to Windows from Steam** — `ujust setup-boot-windows-steam`
  - **Virtualization** — `ujust setup-virtualization virt-on` (virt-manager + libvirtd; reboot)
  - **Input group** — `ujust add-user-to-input-group add`
  - **CoolerControl** — `ujust install-coolercontrol install` (**layers rpm**; reboot)
  - **LM Studio** + **VSCodium** — Homebrew casks (`ublue-os/tap`)
  - **Android Platform Tools** (adb/fastboot) — inside the Distrobox
- **`rpm-ostree`** — `podman-docker` only, baked at build time (Bazzite ships
  Podman; this makes the `docker` CLI target it — no Docker daemon). The only
  other rpm layering anywhere is CoolerControl (`ujust install-coolercontrol`,
  applied post-install, not baked).
- **`files`** — `files/system/*` copied into the image root.
- **`signing`** — cosign signing policy.

The apt-based CLI toolchain (rust/cargo, nvm, pyenv, starship, fzf, nvim, etc.)
is **not** baked in. On the Bazzite host it lives in an Ubuntu Distrobox created
by the dotfiles bootstrap
([`setup/fresh_install_bazzite.sh`](https://github.com/MargeyShah/dotfiles) in the
[dotfiles repo](https://github.com/MargeyShah/dotfiles)).

## Build

Built by GitHub Actions (`.github/workflows/build.yml`): daily, on push, and via
manual dispatch. Requires the repository secret **`SIGNING_SECRET`** (the cosign
private key). The public key is committed as `cosign.pub`.

## Rebase a machine onto your image

Rebase once to the unsigned image to pull in the signing keys/policy, reboot,
then rebase to the signed image (see [INSTALL.md](INSTALL.md) for the full
from-scratch walkthrough):

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

After rebasing, run the dotfiles bootstrap on the host
([repo](https://github.com/MargeyShah/dotfiles)):

```bash
git clone git@github.com:MargeyShah/dotfiles.git ~/.scripts
~/.scripts/setup/fresh_install_bazzite.sh
```

## Tags

BlueBuild emits this tag set on every build (this is BlueBuild's convention, not
Bazzite's `stable`/`testing`/`unstable` branch tags):

| Tag | Meaning | Use |
| --- | --- | --- |
| `latest` | newest build of this variant | **primary rebase target** |
| `YYYMMDD` | build date | pinning / rollback |
| `44` | Fedora major only | stream pin |
| `YYYMMDD-44` | build date + Fedora | rollback |
| `<sha>-44` | git commit + Fedora | exact reproducible pin |

`latest` plays the same role as Bazzite's `stable`/`latest` "rebase me" float.
There is intentionally no `stable`/`testing`/`unstable` tag here — those come
from Bazzite's own git branches, which this repo does not have.

`<sha256-*.sig>` entries you may see in `tags/list` are internal cosign
signature layers, not image tags.

## Verify

```bash
cosign verify --key cosign.pub ghcr.io/margeyshah/bazzite-gnome
```

## Updating

Updates happen in two hops, fully automatic:

```
Bazzite publishes a new stable  →  this repo rebuilds & re-tags :latest  →  your host runs `ujust update`
```

- **Keeping up with upstream.** Two mechanisms:
  1. **Scheduled build** (`.github/workflows/build.yml`, daily 06:00 UTC) —
     re-derives every variant from the floating `image-version: stable`.
  2. **Event-driven rebuild** (`.github/workflows/upstream-check.yml`, hourly) —
     compares upstream Bazzite's `:stable` *index digest* against the
     `org.opencontainers.image.base.digest` label on our published image; if
     upstream moved, it immediately dispatches a rebuild. So you track upstream
     within ~an hour instead of waiting for the next daily cron.
- **On the host.** `ujust update` (Bazzite's updater: OS + flatpaks + Distrobox
  containers) works unchanged against the rebased `:latest` image and picks up
  each newly published build. No host-side changes are needed after the initial
  rebase.

Because the recipes use a floating `image-version: stable`, every rebuild
re-tracks upstream Bazzite's newest stable — the same effect a source fork kept
in sync with the pull app would give, without maintaining a fork.

## Not a fork

This repo does **not** fork [`ublue-os/bazzite`](https://github.com/ublue-os/bazzite).
It is a [BlueBuild](https://bluebuild.org) image repo that pulls Bazzite as its
`base-image`. Bazzite's "fork me + use the pull app" advice targets forks of
their own buildah-based build system; a BlueBuild repo is the lighter-weight,
recommended way to derive a personal image and needs no source fork.
