# 🪟 Windows RDP on GitHub Actions (Cloud Backup Edition)

A full Windows desktop environment hosted on a GitHub Actions runner with secure remote access via Cloudflare Tunnel. Includes automated cloud backup for your browser profiles (Firefox, Chrome, Brave, Edge) and a dedicated `SavedFiles` desktop folder — all persisted across session restarts.

---

## ✨ Features

| Feature | Details |
| :--- | :--- |
| 🪟 **Full Windows Desktop** | Real Windows runner via GitHub Actions — not emulated. |
| 🔒 **Secure Tunnel Access** | Access via Cloudflare Tunnel — no port forwarding required. |
| 🔊 **Audio Support** | VB-Cable virtual audio device pre-installed. |
| 📹 **Screen Recording** | Captura pre-installed with a desktop shortcut. |
| 🌐 **Browsers Pre-installed** | Google Chrome + Brave Browser ready to use. |
| 💾 **Browser Profile Backup** | Saves Firefox, Chrome, Brave, and Edge logins/history/bookmarks to the cloud. |
| 📁 **SavedFiles Folder** | A dedicated `SavedFiles` folder on the Desktop gets synced to the cloud. |
| ⏰ **Hourly Incremental Sync** | Only changed or new files are uploaded every hour. Deleted files are removed from the cloud too. |
| 🔄 **Auto-Restart Loop** | Automatically triggers a new run when the current session ends. |

---

## 🔑 GitHub Secrets Configuration

Go to **Settings ➡️ Secrets and variables ➡️ Actions ➡️ New repository secret** and add:

| Secret Name | Required | Description |
| :--- | :---: | :--- |
| `PASS` | ✅ Yes | Password for the `RDP` user account on the Windows desktop. |
| `CF_TOKEN` | ✅ Yes | Your Cloudflare Tunnel token to securely expose the session. |
| `GDRIVE_CONF` | ⭐ Optional | Rclone config block for Google Drive. **Takes priority over MEGA if both are set.** |
| `MEGA_USER` | ⭐ Optional | Your MEGA email address. Used only if `GDRIVE_CONF` is not set. |
| `MEGA_PASS` | ⭐ Optional | Your MEGA account password. Required alongside `MEGA_USER`. |

> **Backup Priority:** Google Drive is tried first. If `GDRIVE_CONF` is not set, MEGA is used. If neither is configured, the session runs without backups (no data will be saved between restarts).

---

## 💾 What Gets Backed Up?

| Data | Cloud Path |
| :--- | :--- |
| 🦊 Firefox profile (logins, history, cookies) | `winrdp-backup/Mozilla` |
| 🌐 Chrome profile (logins, history, cookies) | `winrdp-backup/ChromeUserData` |
| 🦁 Brave profile (logins, history, cookies) | `winrdp-backup/BraveUserData` |
| 🔷 Edge profile (logins, history, cookies) | `winrdp-backup/EdgeUserData` |
| 📁 Desktop `SavedFiles` folder | `winrdp-backup/SavedFiles` |

> Browser **cache** folders are excluded to keep backups fast and small.

---

## ☁️ Cloud Backup Setup

### Option A — Google Drive *(Recommended — Takes Priority)*

#### 🖥️ On Desktop (Windows/macOS/Linux)

1. Download and install **[Rclone](https://rclone.org/downloads/)**.
2. Open a terminal and run:
   ```bash
   rclone config
   ```
3. Create a new remote — name it exactly **`gdrive`**, select **`drive`** (Google Drive), and complete the authentication flow.
4. Open your `rclone.conf` file:
   - **Windows:** `C:\Users\<YourName>\AppData\Roaming\rclone\rclone.conf`
   - **macOS/Linux:** `~/.config/rclone/rclone.conf`
5. Copy the entire `[gdrive]` block and paste it as the **`GDRIVE_CONF`** GitHub secret.

#### 📋 Example `GDRIVE_CONF` Secret Value:
```ini
[gdrive]
type = drive
client_id = your_client_id.apps.googleusercontent.com
client_secret = your_client_secret_here
scope = drive
token = {"access_token":"ya29.xxxxxxxxxxxx","token_type":"Bearer","refresh_token":"1//xxxxxxxxxxxx","expiry":"2026-07-05T12:00:00.000Z"}
```

> ⚠️ The remote **must** be named `[gdrive]` exactly for the workflow to detect it!

---

#### 📱 On Mobile (Android via Termux)

> [!TIP]
> You can set up Google Drive backup entirely from your Android phone — no PC needed!

**Step A: Get the App**

1. Download **Termux** from the [F-Droid store](https://f-droid.org/en/packages/com.termux/) — ⚠️ **do NOT use the Google Play version**, it is outdated and broken.
2. Open Termux and run:
   ```bash
   pkg update && pkg upgrade -y
   pkg install rclone -y
   ```

**Step B: Link Your Google Account**

3. Start the rclone setup wizard:
   ```bash
   rclone config
   ```
4. Type `n` (New remote) and name it exactly **`gdrive`**.
5. A large list of cloud providers appears. Find **Google Drive** (usually option `18` or `19`) and type that number.
6. Leave `client_id` **blank** → press Enter.
7. Leave `client_secret` **blank** → press Enter.
8. For `scope`, type `1` (Full access to all files).
9. Leave `root_folder_id` and `service_account_file` **blank** → press Enter both times.
10. `Edit advanced config?` → type `n`.
11. `Use web browser to authenticate?` → type `y`. It will output a URL starting with `http://127.0.0.1:53682/auth...`.
12. Copy that URL and open it in **Chrome** on your phone.
13. Log into your Google account and click **Allow**.
14. Back in Termux, when asked `Configure as Shared Drive?` → type `n`.
15. When asked if the config is OK → type `y`, then `q` to quit.

**Step C: Extract and Copy the Config**

16. Print your config to the screen:
    ```bash
    cat ~/.config/rclone/rclone.conf
    ```
17. Long-press to **select and copy** the entire `[gdrive]` block.
18. Paste it as the value of the **`GDRIVE_CONF`** GitHub secret.

---

### Option B — MEGA *(Easiest Setup)*

No local software required. Just add your MEGA login credentials:

1. Add your MEGA email as the **`MEGA_USER`** secret.
2. Add your MEGA password as the **`MEGA_PASS`** secret.

The workflow automatically configures `rclone` for MEGA on the fly.

---

## 🔌 How to Connect

1. Trigger the workflow via **Actions → Windows RDP → Run workflow**.
2. Wait for the setup steps to complete (~3–5 minutes).
3. Open your **Cloudflare Zero Trust** dashboard to find your tunnel's public hostname.
4. Use **Windows Remote Desktop Connection** (or any RDP client) to connect using:
   - **Username:** `RDP`
   - **Password:** *(the value of your `PASS` secret)*
