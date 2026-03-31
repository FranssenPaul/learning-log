# Learning Log — Networking (Synthesis)

## 1. LAN, router, and basic architecture

- A **LAN (Local Area Network)** is defined by:
  - a router
  - an IP range (subnet)
- Ethernet and Wi-Fi are **two access methods to the same LAN**
- The **router defines the network**, not the cable

👉 As long as devices are in the same subnet, they can communicate.

---

## 2. IPv4, subnet, and CIDR

- An IPv4 address looks like: `192.168.129.33` (four octets, 0-255, 32 bits)
- The `/23`, `/24`, etc. suffix is **CIDR notation** (defines subnet size by e.g. 32-23 = 9 bits for hosts)
- Examples:
  - `/24` → 255.255.255.0 → 254 usable hosts
  - `/23` → 255.255.254.0 → 510 usable hosts

### Special addresses:
- **Network address** (e.g. `192.168.50.0`)  
  → identifies the network
- **Broadcast address** (e.g. `192.168.50.255`)  
  → sends traffic to all devices
- These addresses **can never be assigned to a device**

---

## 3. Router vs switch vs access point (AP)

### Router (TP-Link C6)
- Core of the network
- Provides:
  - DHCP
  - routing
  - firewall
- Defines the LAN
- Allows keeping **the same IP scheme everywhere (Belgium / Africa)**

### Switch (PoE or not)
- Extends Ethernet ports
- Does **not** route traffic
- PoE = power + network over one cable

### Access Point (EAP225)
- Provides Wi-Fi
- Does **not** do DHCP (in this setup)
- Broadcasts one or more **SSIDs**

---

## 4. SSID

- **SSID = Wi-Fi network name**
- Example: `POS-Network`
- One AP can broadcast multiple SSIDs
- Each SSID can serve a different purpose (POS, guests, etc.)

---

## 5. VLAN (concept learned, not yet used)

- **VLAN = logical network separation**
- Allows multiple isolated networks on the same infrastructure
- Useful for:
  - separating POS / cameras / guest Wi-Fi
- Not required at the beginning
- To be used later if needed

---

## 6. QoS (Quality of Service)

- QoS prioritizes traffic **when the network is congested**
- Useful mainly for:
  - guest Wi-Fi
  - cameras
  - streaming
- **Not required** for a well-designed local POS
- An offline-first POS does not depend on QoS

---

## 7. Separation of usages (key principle)

- POS = **critical network**
- Cameras = **separate network**
- Guest Wi-Fi = optional, often discouraged

👉 The POS must never depend on what can slow it down.

---

## 8. Internet ≠ LAN

- The POS must work **without Internet**
- Internet is:
  - optional
  - useful for sync, support, updates
- The LAN remains stable even if Internet fails

---

## 9. SSH, permissions, and sudo

- **SSH keys** are used only to:
  - authenticate
  - connect without a password
- They **do not grant admin rights**

### `sudo`
- Temporarily grants root privileges
- Always separate from SSH authentication
- Core security concept

---

## 10. Granting limited rights to a local operator

- Never share passwords
- Grant:
  - a specific action
  - nothing more
- Example:
  - allow only `poweroff` and `reboot`
- Principle: **least privilege**

---

## 11. Server power management

### Shutdown
- Clean shutdown via `sudo poweroff`
- Can be delegated to an operator with limited rights

### Power on
- A powered-off server cannot be started by software
- Possible options:
  - physical power button (acceptable)
  - Wake-on-LAN (optional)
  - BIOS settings

### Chosen approach
- **Stay OFF after power loss**
- Server does not auto-restart after outages
- Startup is a deliberate human action

---

## 12. Fundamental principle learned

> **Simplicity is a strength in production.**

- Clear architecture
- Few dependencies
- Offline-first
- Local control
- Adapted to African constraints

---

## Personal conclusion

- Networks are not “magic”
- They rely on simple, logical rules
- Understanding networking brings:
  - stability
  - control
  - peace of mind
- A POS is **infrastructure**, not just software


## 13. Printing from the command line (CUPS)

### What CUPS is
- **CUPS (Common UNIX Printing System)** is the Linux printing service
- Command-line tools (`lp`, `lpstat`, `lpinfo`) talk to CUPS
- CUPS handles:
  - printer discovery
  - queues
  - drivers / filters
  - network protocols (IPP, LPD)

---

### Core commands

#### List printers
```bash
lpstat -p
```

## 17. File permissions: chmod and chown

Linux protects files and directories using **ownership** and **permissions**. These mechanisms are fundamental for:

- SSH security
- system safety
- service reliability
- POS server hardening

Permissions are a **core security feature** in Linux, not an optional setting.

---

## Ownership: user and group

Every file and directory has:

- an **owner (user)**
- a **group**

Example:

```bash
ls -l report.txt
```

Output:

```text
-rw-r--r-- 1 alice developers 2048 Jan 16 10:00 report.txt
```

### How to read this output

| Element        | Meaning                |
| -------------- | ---------------------- |
| `-rw-r--r--`   | Permissions            |
| `alice`        | Owner (user)           |
| `developers`   | Group                  |
| `2048`         | File size (bytes)      |
| `Jan 16 10:00` | Last modification time |
| `report.txt`   | File name              |

- **Owner:**
  → The file belongs to the user *alice*
- **Group:**
  → Users in the *developers* group may have specific permissions

Ownership answers the question:

> **Who is responsible for this file?**

---

## Permission model: r / w / x

Linux defines three basic permissions:

- `r` → read (view file contents)
- `w` → write (modify or delete)
- `x` → execute (run a file or enter a directory)

Permissions are defined for three categories:

1. **User** (the owner)
2. **Group**
3. **Others** (everyone else)

Example:

```text
-rw-r--r--
```

Breakdown:

```text
rw-   r--   r--
 ^     ^     ^
user  group others
```

Meaning:

- user (alice): read and write
- group (developers): read only
- others: read only

---

## Numeric (octal) permissions

Permissions can also be expressed numerically:

- `r` = 4
- `w` = 2
- `x` = 1

Examples:

- `7` = 4+2+1 = rwx
- `6` = 4+2   = rw-
- `5` = 4+1   = r-x
- `0` = ---   = no access

Common permission values:

| Numeric | Symbolic  | Usage                   |
| ------- | --------- | ----------------------- |
| `700`   | rwx------ | private executable      |
| `600`   | rw------- | private file (SSH keys) |
| `755`   | rwxr-xr-x | public executable       |
| `644`   | rw-r--r-- | standard text file      |

---

## chmod — change permissions

`chmod` changes **what actions are allowed** on a file or directory.

### Example: secure a private SSH key

```bash
chmod 600 ~/.ssh/id_ed25519
```

Meaning:

- owner can read/write
- no access for group or others

SSH **requires this**; otherwise it refuses the key.

---

### Example: make a script executable

```bash
chmod 700 shutdown_pos.sh
```

Meaning:

- only the owner can execute the script
- no one else can read or modify it

---

### Recursive permissions

```bash
chmod -R 700 ~/.ssh
```

Applies permissions to the directory and all its contents.

---

## chown — change ownership

`chown` changes **who owns a file or directory**.

This is separate from permissions.

### Example

```bash
sudo chown root:root /usr/local/bin/shutdown_pos
```

Meaning:

- owner = root
- group = root

Only root can modify or replace this file.
"root" is the superuser account (accessible via sudo).
We can create a group in the system and assign it as the group owner.

---

### Change owner and group

```bash
sudo chown alice report.txt
sudo chown alice:developers report.txt
```

---

## chmod vs chown (important distinction)

| Command | Purpose                 |
| ------- | ----------------------- |
| `chmod` | Defines allowed actions |
| `chown` | Defines ownership       |

They solve different problems and are often used together.

---

## Files vs directories

Directories interpret permissions differently:

- `r` → list directory contents
- `w` → create or delete files
- `x` → enter the directory (it means "access": go inside)

Example:

```bash
chmod 700 ~/.ssh
```

Without `x`, the directory cannot be accessed at all.

---

## Why this matters for a POS server

Correct permissions:

- prevent accidental damage
- limit operator actions
- protect SSH access
- prevent privilege escalation

This is **defensive system design**, not over-engineering.

---

## Key principle

> **Ownership defines responsibility.**\
> **Permissions define allowed actions.**\
> **Secure systems rely on both.**
## 18. Vocabulary clarification (Security & Permissions)

Understanding the language used in system administration is important, because words are chosen **precisely**.

---

## “To grant”

### Meaning
**To grant** means:

> **To officially and deliberately give permission to do a specific action.**

It does **not** mean:
- giving a password
- giving full access
- giving ownership

It means allowing **one precise action**, under controlled conditions.

---

### Phonetic pronunciation
- **to grant** → /tuː ɡrɑːnt/

---

### Example in system administration
When we say:

> “Grant limited rights to a local operator”

We mean:
- allow the operator to perform **only** the required action
- nothing more

Example:
- grant permission to run `poweroff`
- do not grant permission to install software

This is usually done via `sudo` rules.

---

### Relation to security
Granting permissions is:
- explicit
- limited
- revocable

This follows the **principle of least privilege**:
> grant only what is strictly necessary.

---

### Simple analogy
- Password → keys to the whole building  
- Grant → badge that opens **one specific door**

---

## “Harmless”

### Meaning
**Harmless** means:

> **Unable to cause damage, danger, or negative consequences.**

In security, it refers to actions that are considered safe and non-destructive.

---

### Phonetic pronunciation
- **harmless** → /ˈhɑːrmləs/

---

### Example in a POS / server context

- Shutting down a server cleanly → **harmless**
- Reading logs → **harmless**
- Installing unknown software → **not harmless**
- Modifying system files → **not harmless**

When delegating actions, we only grant **harmless or controlled actions** to non-admin users.

---

### Why this matters
In production systems:
- harmless actions can be delegated
- dangerous actions must remain restricted

This reduces:
- human error
- security risks
- system damage

---

## Key takeaway

> **To grant** = to explicitly allow a specific action  
> **Harmless** = safe, non-destructive, controlled  

Clear language leads to clear security decisions.


