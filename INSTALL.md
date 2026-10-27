# Install guide

How to get one of these custom images onto a machine, from scratch.

> **These images are not distributed as an ISO.** You install **stock Bazzite
> from a USB**, then **rebase** the installed system onto the custom image, then
> run the dotfiles **bootstrap**. Rebasing keeps upstream Bazzite updates
> flowing (see [Updating](README.md#updating)).

```
download stock Bazzite ISO  →  flash to USB  →  install  →  rebase onto custom image  →  bootstrap
```

## 1. Pick your variant

| Hardware / desktop | Use image |
| --- | --- |
| Desktop/laptop, AMD/Intel GPU, GNOME | `bazzite-gnome` |
| Desktop/laptop, NVIDIA GPU, GNOME | `bazzite-gnome-nvidia-open` |
| HTPC/handheld (Steam Gaming Mode), GNOME, no NVIDIA | `bazzite-deck-gnome` |
| HTPC/handheld (Steam Gaming Mode), GNOME, NVIDIA | `bazzite-deck-nvidia-gnome` |

Install the **stock** Bazzite that matches: same desktop environment (GNOME) and
same GPU vendor (NVIDIA or not), and `-deck` if you want Steam Gaming Mode.

## 2. Download the stock Bazzite ISO

1. Go to the [Bazzite download page](https://bazzite.gg/#image-picker).
2. Choose: hardware, primary GPU vendor, desktop environment (**GNOME**), and
   whether you want **Steam Gaming Mode** (the `-deck` variant, for HTPC/handheld).
3. Download the ISO. If the direct download is slow, use a download manager
   (e.g. [Motrix](https://motrix.app/)).
4. Optionally verify the ISO's SHA256 against the published checksum.

More detail: [official Bazzite download docs](https://docs.bazzite.gg/General/Installation_Guide/install-guide/#download-bazzite).

## 3. Flash the ISO to a USB

Use a **16 GB+** USB stick.

### Option A — Ventoy (multi-boot USB)

1. Install [Ventoy](https://www.ventoy.net/) to the USB (`Ventoy2Disk` on
   Windows, `VentoyGUI` / `Ventoy2Disk.sh` on Linux).
2. Copy the Bazzite `.iso` file onto the Ventoy partition (just drag-and-drop;
   no re-flashing needed per ISO).
3. Boot the USB and pick the ISO from the Ventoy menu.

> **Secure Boot:** Ventoy needs [extra steps](https://www.ventoy.net/en/doc_secure.html)
> for Secure Boot to work. If you use Secure Boot, you must also enroll the
> Universal Blue key (step 5).

### Option B — Fedora Media Writer (recommended by Bazzite)

[Fedora Media Writer](https://flathub.org/apps/org.fedoraproject.MediaWriter)
(`org.fedoraproject.MediaWriter` on Flathub, or the Windows/macOS builds) — flash
the ISO directly, then eject.

### Option C — Rufus (Windows)

Use [Rufus](https://rufus.ie/) in DD mode.

More detail: [official flash docs](https://docs.bazzite.gg/General/Installation_Guide/install-guide/#flash-iso).

## 4. Boot the installer

- Plug in the USB and boot from it (usually an F-key like **F9**/F12, or set the
  USB first in BIOS; don't leave that BIOS order set permanently).
- **Handhelds:** hold **Volume Down** + press **Power**; release both at the
  chime, then choose the USB.

## 5. Install Bazzite

Follow the on-screen installer:

- Language / region / keyboard.
- **Partitioning** — pick the target drive (this wipes it; back up first;
  unplug other drives). Dual-booting Windows: use automatic partitioning — see
  the [official dual-boot notes](https://docs.bazzite.gg/General/Installation_Guide/install-guide/#dual-booting-windows).
- **Disk encryption** (optional, LUKS) — needs a physical USB keyboard to unlock.
- **User account** — create your user (do **not** enable a root account).
- **Secure Boot** (only if enabled): after the installer, a blue MOK screen
  appears — choose **Enroll MOK**, password `universalblue`. Or, once installed,
  run `ujust enroll-secure-boot-key` (password `universalblue`) with Secure Boot
  temporarily disabled.
- Reboot and eject the USB.

Full walkthrough: [official Bazzite install guide](https://docs.bazzite.gg/General/Installation_Guide/install-guide/).

## 6. Rebase onto your custom image

On the freshly installed Bazzite host, open a terminal. Rebase **once** to the
unsigned image to pull in the signing keys/policy, reboot, then rebase to the
signed image:

```bash
# 1. unsigned image (installs the signing policy)
rpm-ostree rebase ostree-unverified-registry:ghcr.io/margeyshah/bazzite-gnome:latest
systemctl reboot

# 2. signed image
rpm-ostree rebase ostree-image-signed:docker://ghcr.io/margeyshah/bazzite-gnome:latest
systemctl reboot
```

Replace `bazzite-gnome` with the variant from step 1. Confirm with
`rpm-ostree status`.

## 7. Run the dotfiles bootstrap

The custom image bakes in the flatpaks and config, but the CLI toolchain
(Distrobox) and dotfiles are applied by the bootstrap:

```bash
git clone https://github.com/MargeyShah/dotfiles.git ~/.scripts
~/.scripts/setup/fresh_install_bazzite.sh
```

> The clone uses **HTTPS** (the repo is public) — there's no SSH key on a fresh
> install yet; the bootstrap generates one and configures git signing.

The flatpaks, GNOME extensions, Element keyring override and desktop defaults
are already baked into the image, so the bootstrap only adds what cannot live
there: Helium/Sidra as AppImages (via Gear Lever), the Flameshot screenshot
portal permission, the gaming extras (EmuDeck, Decky Loader + plugins), and the
system extras (Proton Plus, Sunshine, Wake-on-LAN, Boot-to-Windows,
Virtualization, input group, CoolerControl, LM Studio, VSCodium). The Ubuntu
Distrobox is provisioned with `fresh_install.sh --profile dotfiles`, which also
symlinks the dotfiles. See the
[dotfiles repo](https://github.com/MargeyShah/dotfiles) for details.

The bootstrap does **not** change your terminal. New terminals open a normal
host shell; enter the container when you want it with
`distrobox enter bazzite-ubuntu` (or run `box`).

## 8. Updating from here on

Nothing further to do — updates are automatic:

```bash
ujust update   # OS (this image) + flatpaks + distrobox containers
```

The image tracks upstream Bazzite automatically (daily + event-driven rebuilds).
See [Updating](README.md#updating).
