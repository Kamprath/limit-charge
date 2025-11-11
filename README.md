# limit-charge

A lightweight Debian utility to **set and persist your laptop’s maximum battery charge limit**.  
It automatically detects your battery device and sysfs threshold file, so you don’t have to configure anything manually.

---

## ⚡ What It Does

Many laptops (Lenovo, Asus, Dell, etc.) support charge limiting via `/sys/class/power_supply/BAT*/charge_*_threshold`.  
`limit-charge` provides a simple CLI and background service to manage this cleanly.

- `limit-charge 80` → sets charge limit to 80% and saves it  
- Automatically re-applies your limit after boot and resume  
- Auto-detects correct sysfs path (`BAT0`/`BAT1`, and `charge_control_end_threshold` vs `charge_stop_threshold`)  

---

## 🧩 How It Works

1. **CLI:**  
   - `limit-charge <value>` writes the value to your system’s threshold file (via `sudo`)  
   - The same value is stored in `/var/lib/limit-charge/value`  

2. **Systemd service (`limit-charge.service`):**  
   - Runs automatically at boot  
   - Reads `/var/lib/limit-charge/value` and reapplies it  

3. **Sleep hook (`/lib/systemd/system-sleep/limit-charge`):**  
   - Reapplies the limit after suspend/resume  

4. **Detection logic (`/usr/lib/limit-charge/detect-sys-file`):**  
   - Finds your battery (`BAT0`, `BAT1`, etc.)  
   - Checks for threshold files:
     - `charge_control_end_threshold` (modern kernels)
     - `charge_stop_threshold` (legacy)
   - Stores the detected path in `/etc/limit-charge.conf` for later use  

---

## 💻 Installation

### From Release (.deb)

1. Download the latest `.deb` from the [Releases page](../../releases).
2. Install it:
   ```bash
   sudo dpkg -i limit-charge_<version>_amd64.deb
   ```

3. Confirm the service is active:

   ```bash
   sudo systemctl status limit-charge.service
   ```

That’s it — your saved charge limit will now persist across boots and resumes.

---

## 🚀 Usage

```bash
limit-charge 80     # Set max charge limit to 80%
limit-charge get    # Show currently stored limit
```

Once set, your chosen limit is saved and automatically restored on each boot.

To apply immediately:

```bash
sudo systemctl start limit-charge.service
```

---

## ⚙️ Configuration

After installation, a config file is generated at:

```
/etc/limit-charge.conf
```

Example:

```bash
# limit-charge configuration
# Auto-detected during installation
SYS_FILE="/sys/class/power_supply/BAT1/charge_control_end_threshold"
```

If detection failed or your hardware changes, you can manually edit this file to point to a valid sysfs path.

To check manually which files exist:

```bash
sudo ls /sys/class/power_supply/BAT*/charge_*_threshold
```

---

## 🛠 File Overview

| Path                                       | Purpose                                    |
| ------------------------------------------ | ------------------------------------------ |
| `/usr/bin/limit-charge`                    | CLI for setting & storing the charge limit |
| `/usr/bin/apply-limit-charge`              | Applies saved limit automatically          |
| `/usr/lib/limit-charge/detect-sys-file`    | Auto-detects correct sysfs path            |
| `/lib/systemd/system/limit-charge.service` | Runs once at boot                          |
| `/lib/systemd/system-sleep/limit-charge`   | Re-applies limit after resume              |
| `/etc/limit-charge.conf`                   | Stores detected sysfs path                 |
| `/var/lib/limit-charge/value`              | Stores user-set limit value                |

---

## 🧪 Tested On

* **Ubuntu 25.10**

### Laptops

* ASUS Zephyrus G16 → `BAT1`, `charge_control_end_threshold`

If your system uses a different filename or battery path, open an issue and include the output of:

```bash
ls /sys/class/power_supply/
ls /sys/class/power_supply/BAT*/charge_*_threshold
```

---

## 🧰 Building the `.deb` (for contributors)

To build locally:

```bash
./build.sh
dpkg-deb --build limit-charge_pkg
```

---

## 💡 Development Tips

* You can test detection manually:

  ```bash
  sudo /usr/lib/limit-charge/detect-sys-file
  ```
* You can disable the service for testing:

  ```bash
  sudo systemctl disable limit-charge.service
  ```
* Re-enable it:

  ```bash
  sudo systemctl enable --now limit-charge.service
  ```

---

## 🧑‍💻 Contributing

Contributions are welcome!
PRs that add new detection patterns, improve packaging, or enhance UX are appreciated.

---

## 🪪 License

MIT License © 2025 Johnny Kamprath
