# limit-charge

A simple Linux utility to **set and persist your laptop’s max battery charge limit**.  
Supports most modern kernels that expose `/sys/class/power_supply/BAT*/charge_*_threshold` controls.

---

## ⚡ Overview

Many laptops (Lenovo, Dell, Asus, etc.) expose a system file that controls how much the battery can charge before stopping.  
This helps **extend battery lifespan** if you often keep your laptop plugged in.

`limit-charge` lets you:

- Set a charge limit: `limit-charge 80`
- Reapply it automatically at boot and after suspend/resume
- Store and reuse your preferred limit

---

## 🧩 How It Works

1. `limit-charge <value>`:
   - Writes the value to your system’s battery threshold file  
   - Saves the value in `/var/lib/limit-charge/value`

2. A systemd service (`limit-charge.service`):
   - Runs on boot (and resume)
   - Reads the saved value
   - Applies it again automatically

3. Detection logic:
   - Automatically finds your battery directory (`/sys/class/power_supply/BAT0`, `BAT1`, etc.)
   - Detects which threshold file your kernel uses:
     - `charge_control_end_threshold` (modern)
     - `charge_stop_threshold` (legacy)
   - Stores that path in `/etc/limit-charge.conf` after install

---

## 💻 Installation

Download the latest `.deb` from the [Releases page](../../releases).

Then install:

```bash
sudo dpkg -i limit-charge_<version>_amd64.deb
````

After install:

```bash
sudo systemctl status limit-charge.service
```

You should see it enabled and ready to apply your saved limit at boot.

---

## 🚀 Usage

```bash
limit-charge 80     # Set max charge to 80%
limit-charge get    # Show current stored limit
```

The limit will be stored in `/var/lib/limit-charge/value`
and reapplied on boot or resume.

To apply immediately (without reboot):

```bash
sudo systemctl start limit-charge.service
```

---

## ⚙️ Configuration

A config file is created at:

```
/etc/limit-charge.conf
```

It looks like this:

```bash
# limit-charge configuration
SYS_FILE="/sys/class/power_supply/BAT1/charge_control_end_threshold"
```

If detection picked the wrong path, you can edit this manually.

To test which file works on your system:

```bash
sudo cat /sys/class/power_supply/BAT*/charge_*_threshold
```

---

## 🛠 Development

To build the `.deb` manually:

```bash
./build.sh
dpkg-deb --build limit-charge_pkg
```

You’ll find the output in the project root.

---

## 🔁 Systemd Details

Installed components:

| File                                       | Purpose                               |
| ------------------------------------------ | ------------------------------------- |
| `/usr/bin/limit-charge`                    | CLI for setting and storing the limit |
| `/usr/bin/apply-limit-charge`              | Reads stored value and applies it     |
| `/usr/lib/limit-charge/detect-sys-file`    | Auto-detects correct sysfs path       |
| `/lib/systemd/system/limit-charge.service` | Runs at boot                          |
| `/lib/systemd/system-sleep/limit-charge`   | Runs after resume                     |
| `/etc/limit-charge.conf`                   | Persistent config override            |

---

## 🧪 Tested On

* Ubuntu 24.04 / 25.10
* Fedora 41
* Arch Linux

Laptops tested:

* ThinkPad X1 Carbon Gen 8 (BAT0 + `charge_control_end_threshold`)
* ASUS Zenbook UX425 (BAT1 + `charge_stop_threshold`)

If your laptop uses a different filename, open an issue and we’ll add it to detection.

---

## 🧑‍💻 Contributing

PRs welcome — especially for:

* Additional filename or vendor detection
* Better error messages
* Packaging improvements

---

## 🪪 License

MIT License © [Your Name]

---

## ❤️ Credits

Inspired by ThinkPad battery utilities and kernel `power_supply` sysfs interface.
