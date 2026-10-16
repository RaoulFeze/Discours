# Home Network Setup — Resume Checkpoint

Saved: 2026-10-13, 23:32 (Mountain Time)

## Overall goal

Build a home network with:

- **Dell computer** — Ubuntu Server for testing web applications and websites, plus a music server.
- **UGREEN NASync DXP4800 Plus** — photo backups from mobile devices, shared storage, and backups.
- **Mac mini and Windows computer (Fenrir)** — clients for development, administration, and playback.
- **Browser access:** Navidrome at <https://music.xprojects.ca>, using Cloudflare Tunnel and Cloudflare Access. Working for Raoul; family enrollment pending. No router port forwarding.

## Where we stopped

- All mobile backups (iPad, Endy's iPhone, Samsung) are complete and verified.
- Independent NAS backup is deferred until a USB drive is available.
- Dell drives reconnected; Ubuntu boots normally. The Seagate was erased, formatted as ext4 (`storage`), and auto-mounts at `/srv/storage`. The Kingston was erased, formatted as ext4 (`fast`), and auto-mounts at `/srv/fast`.
- Navidrome deployed; administrator account created, test music plays, and automatic recovery after a Dell reboot verified.
- **Remote address working:** Raoul confirmed "working perfectly" at <https://music.xprojects.ca> at 23:31 on 2026-10-13. On 2026-10-15, Raoul confirmed that the cloudflared service checks, Dell reboot recovery, and off-home access/playback all work.
- Printer deferred by Raoul. Next: family access.
- Windows can be shut down without stopping Navidrome or Cloudflare Tunnel: both run on the Dell. The Dell, NAS, and router must remain on for music access. If shutting down the Dell, use `sudo shutdown -h now` over SSH; music, test website, and SSH will be unavailable until it is turned on again.

## Start here next time

1. Navidrome: **Deployed and reboot-tested 2026-10-13.** Compose configuration at `~/apps/navidrome/compose.yaml`: image `deluan/navidrome:0.64.2`, user `1000:1000`, localhost port `4533`, data at `/srv/fast/navidrome`, music read-only from `/mnt/nas-music`, restart `unless-stopped`. Container `navidrome-navidrome-1` started; web interface `/app/` independently verified HTTP 200, listener verified at `127.0.0.1:4533`, NAS mount still read-only. **Administrator account created:** confirmed by Raoul on 2026-10-13. **Test music scanning and playback working:** confirmed by Raoul after copying test songs from Windows to NAS `Music`. **Reboot recovery verified:** supplied output shows container automatically running, localhost port preserved, NAS CIFS mount read-only, and `/srv/fast` mounted; Raoul confirmed playback still works after the reboot.
   - From Windows PowerShell: `ssh -N -o ExitOnForwardFailure=yes -L 127.0.0.1:4533:127.0.0.1:4533 home-server`, then open <http://127.0.0.1:4533>. Keep that terminal open; Ctrl+C stops the tunnel, not Navidrome.
2. ~~Verify cloudflared is enabled and recovers after a Dell reboot.~~ Done 2026-10-15; Raoul confirmed the service checks and reboot recovery work.
3. ~~Test <https://music.xprojects.ca> from outside the home network.~~ Done 2026-10-15; Raoul confirmed everything works, including off-home access/playback.
4. Add family members' explicitly approved exact email addresses to Cloudflare Access policy `Invited family emails`, then create separate **non-admin** Navidrome accounts for them. Do not share the administrator account. Browser access is the current setup; native Subsonic clients may not support the Access browser login, so assess separately without bypassing Access.
5. Then resume web application Docker Compose projects and their data/database backups. Printer remains deferred; independent NAS backup still needs an external drive.

**Safety rules:** keep the originals on the phones and the administrator's original NAS photos. Do **not** enable "free up space" or delete-after-backup.

### Cloudflare remote access — working

- Domain: `xprojects.ca`, registered with Cloudflare; DNS managed there.
- Access application: **Family Music**, destination `music.xprojects.ca`, 24-hour session. One Allow policy **Invited family emails**, initially restricted to Raoul's approved Outlook email (exact address stored in dashboard). Email one-time PIN enabled; application accepts configured identity providers. No public bypass policy was added.
- Tunnel: **home-server**, ID `e03131d9-d666-4e9f-8351-960b0d075dac`. Installed directly on Dell Ubuntu using Debian/64-bit instructions and **Install as service**, not a Docker container or manual foreground process. Dashboard Healthy with one replica, cloudflared `2026.10.0`. Enabled/active status and recovery after a Dell reboot confirmed 2026-10-15.
- Published application route: `music.xprojects.ca` -> **`http://127.0.0.1:4533`**, path blank. No other services published. Navidrome remains bound to Dell localhost; no router port forwarding or new inbound firewall ports.
- DNS: automatically created tunnel CNAME to `e03131d9-d666-4e9f-8351-960b0d075dac.cfargotunnel.com`. Public DNS resolution verified. Independent unauthenticated HTTPS request returned **302 to Cloudflare Access login**. Raoul confirmed the resulting site works at 23:31.
- Earlier failures: hostname did not resolve because the tunnel had no published route. A form typo `127.0.1` was corrected to `127.0.0.1` before route creation. Windows briefly cached the missing DNS result; `ipconfig /flushdns` and reopening the browser can help if it recurs.
- Never put tunnel tokens, private keys, passwords, or email login codes in this checkpoint or chat. Use Cloudflare dashboard to manage Access users and routes.

## Network

| Item | Value |
|---|---|
| Router | Rogers gateway, `10.0.0.1` |
| Subnet | `10.0.0.0/24` (`255.255.255.0`) |
| DHCP pool | `10.0.0.2` – `10.0.0.253` (unchanged) |
| Dell / `home-server` | Reserved IP **`10.0.0.20`** (Ethernet `enp4s0`, MAC `20:04:0F:FF:F8:00`) |
| UGREEN NAS | Reserved IP **`10.0.0.21`** |
| Printer | Not reserved yet (`10.0.0.22` is only a candidate; check first) |
| Windows PC (Fenrir) | DHCP over Wi-Fi (was `10.0.0.131`) |

Reservations are configured on the router (**Connected Devices → Devices → Edit → Reserved IP**). The devices still use DHCP.

## Dell server — done

- **Hardware:** Intel Core i7-6700 (4 cores, 8 threads), about 16 GB RAM.
- **OS:** Ubuntu Server 24.04 LTS, installed on the **Fanxiang S101 512 GB SSD** only.
  - LVM root `/` ext4 about 473.9 GB, `/boot` 2 GB, `/boot/efi` 1.05 GB (EFI is on the Fanxiang SSD).
  - Linux Mint was removed.
- **Hostname:** `home-server`. **User:** `fenrir`.
- System updated, `intel-microcode` installed, reboot verified.
- **SSH:**
  - Key-only login from Windows with the key `C:\Users\jraou\.ssh\home_server_ed25519`, which has a passphrase.
  - Windows `~\.ssh\config` has a `Host home-server` entry, so connect with `ssh home-server`.
  - Windows `ssh-agent` enabled with Manual startup on 2026-10-13; the key was successfully loaded using `ssh-add`. If unavailable after a Windows restart, start the service in administrator PowerShell and load the key again in regular PowerShell. Dell Docker commands still require interactive `sudo` authentication.
  - Password SSH login and root login are disabled (`/etc/ssh/sshd_config.d/00-home-server.conf`).
- **Firewall (UFW):** active. Denies incoming, allows outgoing, and allows SSH (22/tcp) only from `10.0.0.0/24`.
- **Docker Engine 29.9.0 and Docker Compose v5.6.0** are installed from Docker's official repository. `hello-world` succeeded.
  - Use `sudo docker` for now. `fenrir` was not added to the `docker` group.
  - Ports published by Docker can bypass UFW. Bind test services to `127.0.0.1` unless we deliberately expose them.
- **NAS music mount:** `//10.0.0.21/Music` mounted **read-only** (CIFS, SMB 3.1.1) at `/mnt/nas-music` via `/etc/fstab` (`nofail,_netdev,x-systemd.automount`; backup `/etc/fstab.bak3`). Credentials in `/etc/nas-music.cred` (root-only, `chmod 600`). Write test correctly fails with "Read-only file system".
- **Test website:** `~/apps/test-website` on the Dell runs Nginx with `restart: unless-stopped` and is bound to `127.0.0.1:8080`.
  - To view it from Windows, run:
    `ssh -N -o ExitOnForwardFailure=yes -L 127.0.0.1:8080:127.0.0.1:8080 home-server`
    Then open <http://127.0.0.1:8080>. The window looks idle while the tunnel is running.

### Dell drives

| Drive | Status | Plan |
|---|---|---|
| Fanxiang S101 512 GB SSD | Ubuntu system disk | Keep |
| Seagate ST1000DM010 1 TB HDD | **Reconnected 2026-10-13.** Checked (effectively empty), then erased: GPT, one ext4 partition, label `storage`, UUID `1e4037f5-5dc1-4d9e-b00e-8161c7ae2ca1`. ID `ata-ST1000DM010-2EP102_ZN120X99` | **Done:** auto-mounts at `/srv/storage` (fstab by UUID, `nofail`; backup `/etc/fstab.bak`), owned by `fenrir`, reserve 1%, ~907 GB free. Verified after reboot. |
| Kingston SA400 960 GB SSD | **Erased 2026-10-13** (personal files already backed up by Raoul). GPT, one ext4 partition, label `fast`, reserve 1%, UUID `94938d4c-0c07-4803-befe-3df15edc7f67`. ID `ata-KINGSTON_SA400S37960G_50026B7785A72CD3` | **Done:** auto-mounts at `/srv/fast` (fstab by UUID, `nofail`; backup `/etc/fstab.bak2`), owned by `fenrir`, ~871 GB free. Verified after reboot. Use for Docker app data and databases |

Ubuntu still boots from the Fanxiang SSD with both drives connected (verified 2026-10-13).

## UGREEN NAS — done

- **Storage:** Storage Pool 1 is **RAID 1** (drives 1 and 2), normal, about 3.6 TB. Volume 1 is ext4, about 3.5 TB, normal. No changes made.
- **Accounts:**
  - `Raoul F` — administrator. Use only for NAS management.
  - `raoul-personal` — standard user, personal folder enabled. Use for Raoul's phone and iPad.
  - `Endy` — standard user, personal folder enabled.
  - `navidrome` — standard user (created 2026-10-13) used only by the Dell to read `Music`. No personal folder or apps. Password is in Raoul's password manager (never write it in this file).
  - Shared folders `docker` and `Photos`: **access denied** for both standard users (intended).
  - Shared folder `Music` (Volume 1, recycle bin admin-only): `raoul-personal` read/write, `navidrome` read-only, `Endy` read-only. Music library will live here.
- **Photos app:** each user's Personal Library indexes `Personal Folder/Photos`.
- **Migration:** the administrator's `Personal Folder/Photos` (`CA_IMAGES`, `Camera`, `MobileBackup`, and loose files) was **copied, not moved**, into `raoul-personal`'s Photos. The checked files open correctly.
  - A `Migration-test` folder with 3 test files remains in `raoul-personal`.
  - The administrator's originals are kept until an independent backup exists.

## Mobile backups

| Device | NAS account | Status |
|---|---|---|
| Samsung Galaxy S24 (SM-S921W) | `raoul-personal` | Full backup started. All 12 albums go to `Photos/MobileBackup/samsung SM-S921W`, sorted by month and year. Time range: All. Mobile data backup **off**. Energy-saving backup **on**. Background settings need verification. |
| iPad | `raoul-personal` | Complete and verified (2026-10-13) |
| Endy's iPhone | `Endy` | Complete and verified (2026-10-13) |

Samsung backup verified complete, and background backup over home Wi-Fi confirmed working (2026-10-13).

If a browser preview fails, refresh the browser. That fixed the earlier preview problem.

## Remaining tasks (in order)

1. ~~Finish and verify the iPhone, iPad, and Samsung backups.~~ Done 2026-10-13.
2. **Independent NAS backup:** an external USB drive and/or an encrypted off-site copy. RAID 1 is not a backup. **Deferred:** no USB drive available yet. Size it to the NAS's used space × 1.5–2 (8 TB covers nearly the whole volume).
   - Only after that is verified, consider cleaning up the administrator's duplicate photos and `Migration-test`.
3. **Printer (deferred by Raoul):** identify it on the router and reserve its current IP or `10.0.0.22` if that address is free.
4. ~~**Dell drives:**~~ Done 2026-10-13. Seagate at `/srv/storage`, Kingston at `/srv/fast`; both verified after reboot.
5. ~~**Music server:**~~ Done 2026-10-13. Navidrome in Docker on the Dell, administrator account created, scanning/playback and reboot recovery verified. Music stays on the NAS with read-only access; data stays at `/srv/fast/navidrome` on the Dell.
6. **Web applications:** Docker Compose projects on the Dell. Back up databases to the NAS.
7. **Optional:** UPS for the NAS, Dell, router, and switch; Tailscale for private admin access.
8. **Remote access:** working for Raoul at <https://music.xprojects.ca>, using Cloudflare Tunnel and email-restricted Cloudflare Access. cloudflared reboot recovery and off-home access/playback confirmed 2026-10-15. Remaining: family email enrollment and non-admin Navidrome accounts. Never publish NAS management, infrastructure admin pages, databases, or SSH.
