# bazzite

Personal [BlueBuild](https://bluebuild.org) images derived from upstream
[Universal Blue / Bazzite](https://github.com/ublue-os/bazzite), one recipe per
variant. Each image is a thin layer on top of upstream Bazzite, so rebasing to
it keeps pulling Bazzite's own updates — you just get the extra baked-in config.

## Variants

| Recipe | Base image | Image | GPU |
| --- | --- | --- | --- |
| `recipes/bazzite-gnome.yml` | `ghcr.io/ublue-os/bazzite-gnome` | `ghcr.io/margeyshah/bazzite-gnome` | AMD / Intel |
| `recipes/bazzite-gnome-nvidia.yml` | `ghcr.io/ublue-os/bazzite-gnome-nvidia` | `ghcr.io/margeyshah/bazzite-gnome-nvidia` | NVIDIA 7XX–10XX |
| `recipes/bazzite-gnome-nvidia-open.yml` | `ghcr.io/ublue-os/bazzite-gnome-nvidia-open` | `ghcr.io/margeyshah/bazzite-gnome-nvidia-open` | NVIDIA 16XX or newer |
| `recipes/bazzite-deck-gnome.yml` | `ghcr.io/ublue-os/bazzite-deck-gnome` | `ghcr.io/margeyshah/bazzite-deck-gnome` | AMD / Intel |
| `recipes/bazzite-deck-nvidia-gnome.yml` | `ghcr.io/ublue-os/bazzite-deck-nvidia-gnome` | `ghcr.io/margeyshah/bazzite-deck-nvidia-gnome` | NVIDIA 16XX or newer |

## Setup

> **Installing from scratch?** See **[INSTALL.md](INSTALL.md)** first: it covers
> downloading stock Bazzite, flashing a USB and installing it.

This is a **rebase of an existing Bazzite install** — upstream Bazzite stays
underneath, so keep the variant you already run (same desktop, same GPU driver,
`-deck` if you use Steam Gaming Mode). Find out which one that is:

```bash
rpm-ostree status | grep -m1 -oE 'ghcr\.io/[^:]+'
```

For example `ghcr.io/ublue-os/bazzite-gnome-nvidia-open` → rebase to
`ghcr.io/margeyshah/bazzite-gnome-nvidia-open`. If the command already prints a
`margeyshah/` image you are on one of these already — just keep that variant.
The commands below use `bazzite-gnome`; substitute your variant.

1. **Rebase onto this image**, in two steps: the first (unsigned) image installs
   the signing policy, the second (signed) one is what you keep updating.

   ```bash
   # 1. unsigned image: installs the signing keys/policy
   rpm-ostree rebase ostree-unverified-registry:ghcr.io/margeyshah/bazzite-gnome:latest
   systemctl reboot

   # 2. signed image: the one you actually run
   rpm-ostree rebase ostree-image-signed:docker://ghcr.io/margeyshah/bazzite-gnome:latest
   systemctl reboot
   ```

2. **Verify the image signature** (optional). `cosign.pub` lives in this repo,
   so either run it from a clone, or download just the key:

   ```bash
   # from a clone of this repo
   cosign verify --key cosign.pub ghcr.io/margeyshah/bazzite-gnome

   # or, without cloning
   curl -fsSL https://raw.githubusercontent.com/MargeyShah/bazzite/main/cosign.pub -o cosign.pub
   cosign verify --key cosign.pub ghcr.io/margeyshah/bazzite-gnome
   ```

### Applied by [dotfiles](https://github.com/MargeyShah/dotfiles)

```bash
git clone https://github.com/MargeyShah/dotfiles.git ~/.scripts
~/.scripts/setup/fresh_install.sh
```

Clone over **HTTPS** — a fresh install has no SSH key yet (the bootstrap makes
one). The menu offers **1. Unix PC** (host extras + Distrobox toolchain) or
**2. dotfiles only** (toolchain + dotfiles, no host extras). Both do the same
host work: verify `podman-docker`, install `zsh` with Homebrew and make it the
login shell (`usermod -s`; Bazzite has no `chsh`), grant Flameshot the
screenshot portal, and create/provision the Ubuntu Distrobox.

- **AppImages** (Unix PC tier) — Helium, Sidra, Ghostty and Vesktop, installed
  into `~/AppImages` and integrated with Gear Lever.
- **Gaming extras** (skip with `--no-gaming`):
  - EmuDeck — `ujust get-emudeck install`
  - Decky Loader — `ujust setup-decky install`
  - Bazzite Buddy — `ujust get-decky-bazzite-buddy install` (not in the store;
    installed via its ujust recipe)
  - Decky plugins, from the Decky store, hash-verified into
    `~/homebrew/plugins` (no config is synced between machines):
    - CSS Loader
    - SteamGridDB
    - ProtonDB Badges
    - HLTB for Deck
    - TabMaster
    - PlayCount
    - IsThereAnyDeal for Deck
    - Decky LSFG-VK
    - AutoFlatpaks
    - Decky Terminal
    - Audio Loader
    - Game Theme Music
    - Quick Launch
    - Volume Mixer
    - EmuDecky
    - MusicControl
- **System extras** (skip with `--no-extras`):
  - Wake-on-LAN — `ujust wol enable`
  - Boot to Windows from Steam — `ujust setup-boot-windows-steam`
  - Virtualization — `ujust setup-virtualization virt-on` (virt-manager +
    libvirtd; reboot)
  - Input group — `ujust add-user-to-input-group add`
  - LM Studio + VSCodium — Homebrew casks (`ublue-os/tap`)
  - Android Platform Tools (adb/fastboot) — inside the Distrobox

The apt-based CLI toolchain (rust/cargo, nvm, pyenv, starship, fzf, nvim, etc.)
lives in an Ubuntu Distrobox on the host.

**Entering the toolchain:**

- `distrobox enter bazzite-ubuntu` — straight into the container.
- `box` — an fzf picker over every Distrobox container. It enters the only
  container directly when there is just one, so it is the everyday command.
- `exit` inside the container returns you to the host shell.

That's it. `ujust update` keeps the machine current afterwards — see
[Updating](#updating).

## What the image adds

Flatpak-first: GUI apps come from Flathub, and RPMs are baked at build time.

- **`default-flatpaks`** — system GUI apps from Flathub:
  - Element
  - VS Code
  - Signal Desktop
  - Plexamp
  - Hidamari (video wallpaper)
  - osu! (lazer)
  - Ludusavi (save backups)
  - Heroic Games Launcher
  - Gear Lever (AppImage manager)
  - Flameshot (screenshots)
  - OBS Studio (recording/streaming)
  - GPU Screen Recorder (ShadowPlay-like)
  - Proton Plus (Proton/Wine manager)
  - Sunshine (game streaming host)
  - Prism Launcher (Minecraft)
  - Cemu (Wii U)
  - RPCS3 (PS3)
  - Dolphin (GameCube/Wii)
  - XIVLauncher (FFXIV)
- **`gnome-extensions`** — installs the extensions Bazzite does not ship, pinned
  by e.g.o PK ID:
  - Deja Window
  - Hide Top Bar
  - Just Perfection
  - Mute All Microphones
  - Native Dock
  - Quick Settings Audio Devices Hider
  - Quick Settings Audio Devices Renamer
- **`gschema-overrides`** — two drop-ins:
  - `zz1-margey-extensions.gschema.override` — default-enables the full extension
    set (Bazzite's own plus the seven above, 17 in total), so a fresh install
    needs no manual toggling.
  - `zz1-margey-desktop.gschema.override` — window-manager defaults
    (`focus-new-windows='smart'`, `auto-raise=true`).
- **`rpm-ostree`** — baked at build time:
  - `podman-docker` — makes the `docker` CLI target Podman (no Docker daemon).
  - `liquidctl` + `coolercontrol` — the CoolerControl stack, from the Terra repo
    that `files/rpm-ostree/terra.repo` enables for the build (Bazzite ships it
    disabled and only turns it on at runtime for `ujust`).
- **`files`** — `files/system/*` copied into the image root:
  - `/var/lib/flatpak/overrides/im.riot.Riot` — Element keyring access
    (`talk-name=org.freedesktop.secrets`), so no per-user `flatpak override` is
    needed.
  - `/usr/share/margey-image/README` — marker file.

  (`files/gschema-overrides/` and `files/rpm-ostree/` are module inputs, not
  copied.)
- **`signing`** — cosign signing policy.

## Build

Built by GitHub Actions (`.github/workflows/build.yml`): on push, on pull
requests, via manual dispatch, and on a daily cron that skips itself when the
commit is already built *and* upstream is unchanged (see [Updating](#updating)).
Requires the repository secret **`SIGNING_SECRET`** (the cosign private key). The
public key is committed as `cosign.pub`.

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

## Updating

Updates happen in two hops, fully automatic:

```
Bazzite publishes a new stable  →  this repo rebuilds & re-tags :latest  →  your host runs `ujust update`
```

- **Keeping up with upstream.** Two mechanisms:
  1. **Scheduled build** (`.github/workflows/build.yml`, daily 06:00 UTC) —
     re-derives every variant from the floating `image-version: stable`. A
     `guard` job skips the scheduled run only when **both** the current commit
     has already been built *and* every published variant was built from the
     current upstream `stable` digest, so a daily run never masks upstream drift
     (push and dispatched builds are never skipped).
  2. **Event-driven rebuild** (`.github/workflows/upstream-check.yml`, hourly) —
     compares upstream Bazzite's `:stable` *index digest* against the
     `org.opencontainers.image.base.digest` label on our published image; if
     upstream moved, it immediately dispatches a rebuild. So you track upstream
     within ~an hour instead of waiting for the next daily cron.
- **On the host.** `ujust update` (the `uupd` updater) updates the OS, flatpaks,
  Homebrew and Distrobox containers, and picks up each newly published build; no
  host-side changes are needed after the initial rebase. It only **stages** the
  OS deployment — reboot to switch to it (`rpm-ostree status` shows the pending
  one). If the `distrobox` module has been disabled in `/etc/uupd/config.json`,
  refresh containers with `distrobox upgrade --all` instead.

Because the recipes use a floating `image-version: stable`, every rebuild
re-tracks upstream Bazzite's newest stable — the same effect a source fork kept
in sync with the pull app would give, without maintaining a fork.
