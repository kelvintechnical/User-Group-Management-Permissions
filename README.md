# User & Group Management / Permissions (RHEL 9)
### RHCSA EX200 Lab | Part of [linux-ops-mastery](https://github.com/kelvintechnical/linux-ops-mastery)
> Create a user, group, and directory, then configure ownership and permissions so access is precisely controlled per user, group, and others.

![RHCSA](https://img.shields.io/badge/RHCSA-EX200-EE0000?style=flat&logo=redhat&logoColor=white)
![Topic](https://img.shields.io/badge/Topic-User_and_Group_Management-blue)

---   

## 📋 Scenario

On **Node1**, create a directory `/data/engineers`, a group called `engineers`, and a user called `tom`. Configure permissions so that:

1. `tom` cannot read, write, or execute files in `/data/engineers` — but can change permissions on the directory (owner privilege).
2. Members of the `engineers` group have full access (`rwx`).
3. All other users have no access.

---

## 🎯 Requirements

1. Create the directory `/data/engineers`.
2. Create a group called `engineers`.
3. Create a user called `tom`.
4. Set `tom` as the owner of `/data/engineers`.
5. Set `engineers` as the group owner of `/data/engineers`.
6. Set permissions so owner=`---`, group=`rwx`, others=`---`.
7. Verify all settings are correct.

---

## ✅ Tasks

- Use `mkdir -p` to create the directory
- Use `groupadd` to create the group
- Use `useradd` to create the user
- Use `chown` to set owner and group
- Use `chmod` to set permissions
- Verify with `ls -ld` and `id`

---

## 📚 Command Decision Map

| Lab Phrase | Question Being Asked | Tool |
|------------|---------------------|------|
| "Create a directory" | How do I create a directory + parents? | `mkdir -p` |
| "Create a group" | How do I add a group to the system? | `groupadd` |
| "Create a user" | How do I add a user to the system? | `useradd` |
| "Set owner" | How do I change who owns a file/dir? | `chown user:group` |
| "Configure permissions" | How do I set rwx per owner/group/others? | `chmod` |
| "Verify" | How do I see permissions and ownership? | `ls -ld` |

---

## 🧠 Big Concept — Linux Permission Model

Every file and directory has **three permission sets**:

```
Owner   Group   Others
r w x   r w x   r w x
4 2 1   4 2 1   4 2 1
```

- **Owner** — the user who owns the file (controls with `chown`)
- **Group** — the group assigned to the file (controls with `chown :group`)
- **Others** — everyone else

**Key insight for this lab:** Even if `tom` is the owner, you can set owner permissions to `---` (000). Tom loses read/write/execute but retains the ability to `chmod` the directory — because **only the owner can change permissions**, regardless of what those permissions are.

---

## Step 1 — Create the directory

```bash
sudo mkdir -p /data/engineers
```

> `-p` creates parent directories as needed — if `/data` doesn't exist, it creates both `/data` and `/data/engineers` in one command.

---

## Step 2 — Create the group and user

```bash
sudo groupadd engineers
sudo useradd tom
```

> `useradd` creates the user with a home directory at `/home/tom` and a default private group also called `tom`. The `engineers` group is separate — we assign it via `chown`.

---

## Step 3 — Set ownership

```bash
sudo chown tom:engineers /data/engineers
```

> **Format:** `chown owner:group /path`
> - `tom` becomes the owning user
> - `engineers` becomes the owning group
> - Both set in one command

---

## Step 4 — Set permissions

```bash
sudo chmod 070 /data/engineers
```

> **Breaking down `070`:**

| Position | Who | Value | Permissions |
|----------|-----|-------|-------------|
| `0` | Owner (tom) | 0 | `---` no access |
| `7` | Group (engineers) | 4+2+1 | `rwx` full access |
| `0` | Others | 0 | `---` no access |

> `tom` gets `0` (no rwx) but still owns the directory — ownership is separate from permissions. The owner always retains the right to `chmod`, even with `---`.

---

## Step 5 — Verify everything

```bash
ls -ld /data/engineers
```

**Expected output:**
```
d---rwx--- 2 tom engineers 6 May 20 11:00 /data/engineers
```

**Reading the output:**

| Part | Meaning |
|------|---------|
| `d` | It's a directory |
| `---` | Owner (tom) has no rwx |
| `rwx` | Group (engineers) has full access |
| `---` | Others have no access |
| `tom` | Owning user |
| `engineers` | Owning group |

**Confirm user and group exist:**
```bash
id tom
getent group engineers
```

---

## 🧠 Key Concepts

| Concept | What it means |
|---------|--------------|
| `mkdir -p` | Creates directory + all missing parents in one shot |
| `groupadd` | Adds a new group to `/etc/group` |
| `useradd` | Adds a new user to `/etc/passwd` and creates home dir |
| `chown user:group` | Sets both owner and group in one command |
| `chmod 070` | Octal notation — owner=0, group=7, others=0 |
| Ownership vs permissions | Owner can always `chmod` even if permissions are `---` |
| `ls -ld` | `-l` = long format, `-d` = show the directory itself not its contents |

---

## ⚠️ Pitfalls

- **Forgetting `-p` on mkdir** → fails if `/data` doesn't exist yet
- **`chown tom engineers`** (space instead of colon) → sets owner to `tom` on a file called `engineers`, not what you want — always use `tom:engineers`
- **`chmod 770` instead of `070`** → gives tom rwx too, violating requirement 1
- **Confusing ownership and permissions** → tom owning the dir doesn't mean tom has rwx; they are independent settings
- **Not verifying with `ls -ld`** → always confirm before moving on; a wrong chmod is silent

---

## ✅ Lab Checklist

- `/data/engineers` directory created ✓
- Group `engineers` created ✓
- User `tom` created ✓
- `chown tom:engineers /data/engineers` ✓
- `chmod 070 /data/engineers` ✓
- `ls -ld` shows `d---rwx---` with `tom engineers` ✓

---

## 🔗 Related Labs

- [Configure a Static IP Address](https://github.com/kelvintechnical/linux-ops-mastery/blob/master/01-system-management/README.md)
- [Configure Repository Access](https://github.com/kelvintechnical/linux-ops-mastery/blob/master/01-system-management/configure-repo-access.md)
- [Configure Timezone and Time Synchronization](https://github.com/kelvintechnical/linux-ops-mastery/blob/master/01-system-management/configure-timezone.md)
- [Search for a String and Save Output](https://github.com/kelvintechnical/linux-ops-mastery/blob/master/01-system-management/search-string-save-output.md)
- [Find and Save Config Files](https://github.com/kelvintechnical/linux-ops-mastery/blob/master/01-system-management/find-save-config-files.md)
- [Full RHCSA/RHCE Study Guide →](https://github.com/kelvintechnical/linux-ops-mastery)

---

## 👤 Author

**Kelvin R. Tobias** — [kelvinintech.com](https://kelvinintech.com) | [GitHub](https://github.com/kelvintechnical) | [LinkedIn](https://www.linkedin.com/in/kelvin-r-tobias-211949219)
