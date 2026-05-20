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

- **Owner** — the user who owns the file (set with `chown`)
- **Group** — the group assigned to the file (set with `chown :group`)
- **Others** — everyone else on the system

**Key insight for this lab:** Even if `tom` is the owner, you can set owner permissions to `---` (0). Tom loses read/write/execute but retains the ability to `chmod` the directory — because **only the owner can change permissions**, regardless of what those permissions currently are.

---

## Step 1 — Create the directory

```bash
sudo mkdir -p /data/engineers
```

`/data` doesn't exist on your system yet — `-p` handles that by creating both `/data` and `/data/engineers` in one shot. Without `-p`, the command fails if the parent directory is missing.

**Verify it was created:**
```bash
ls -ld /data/engineers
```

**Expected output:**
```
drwxr-xr-x. 2 root root 6 May 20 11:14 /data/engineers
```

Right now `root` owns everything. By the end of the lab it will show `tom engineers` with `---rwx---` permissions.

---

## Step 2 — Create the group and user

```bash
sudo groupadd engineers
sudo useradd tom
```

`groupadd` adds `engineers` to `/etc/group`. `useradd` creates `tom` with a home directory at `/home/tom`.

**Verify both exist:**
```bash
getent group engineers
id tom
```

**Expected output:**
```
engineers:x:1001:
uid=1001(tom) gid=1002(tom) groups=1002(tom)
```

---

### 🧠 Understanding the output

**`getent group engineers` — reads left to right as a colon-separated record:**

| Field | Value | Meaning |
|-------|-------|---------|
| `engineers` | engineers | Group name |
| `x` | x | Password placeholder — always `x`, ignore it |
| `1001` | 1001 | Group ID (GID) — the number Linux uses internally |
| *(empty)* | | Member list — blank means no users added yet |

**`id tom` — three fields:**

| Field | Value | Meaning |
|-------|-------|---------|
| `uid=1001(tom)` | 1001 | Tom's User ID — his personal identity number |
| `gid=1002(tom)` | 1002 | Tom's primary group — Linux auto-creates a private group named `tom` for every new user |
| `groups=1002(tom)` | 1002 | Every group Tom belongs to — right now only his own private group |

---

### 🧠 Why does tom have uid=1001 and engineers have GID=1001?

That's a coincidence of creation order — UIDs and GIDs are assigned from the same incrementing counter. They look the same but live in completely separate namespaces:

- `uid=1001` = Tom the **user**
- `GID=1001` = engineers the **group**

They don't conflict. Linux tracks users and groups in separate databases.

**Tom is NOT a member of engineers** — his `groups=` line only shows `1002(tom)`. If he were in engineers it would show `groups=1002(tom),1001(engineers)`. That's fine — we don't need him in the group. We just need `engineers` as the **group owner** of the directory.

---

### 🧠 User Private Group (UPG)

When you run `useradd tom`, Linux automatically creates:
- A user named `tom` with `uid=1001`
- A group named `tom` with `gid=1002` (next available GID)

This is called a **User Private Group (UPG)** — a RHEL convention so files a user creates aren't accidentally shared with others. You can override it at creation time with `useradd -g engineers tom` to assign a different primary group instead.

---

### 🧠 `getent` vs `groupadd` — completely different jobs

| Command | Job | Analogy |
|---------|-----|---------|
| `groupadd` | **Creates** a group — writes a new entry to `/etc/group` | INSERT into a database |
| `getent group` | **Reads** group entries — queries the system's group database | SELECT from a database |

`getent` stands for **"get entries"** — a lookup tool that queries system databases:

- `getent group` → reads `/etc/group`
- `getent passwd` → reads `/etc/passwd` (user accounts)
- `getent hosts` → reads hostname/DNS entries

Simple rule: `groupadd` = **write**, `getent` = **read**. After you create something, use `getent` to confirm it worked — same reason you run `cat` after writing a file.

---

## Step 3 — Set ownership

```bash
sudo chown tom:engineers /data/engineers
```

Sets `tom` as the owning user and `engineers` as the owning group in one command. The colon separates user from group — no spaces around it.

**Verify:**
```bash
ls -ld /data/engineers
```

**Expected output:**
drwxr-xr-x. 2 tom engineers 6 May 20 11:14 /data/engineers

Notice what changed from Step 1:

| Before | After |
|--------|-------|
| `root root` | `tom engineers` |
| `rwxr-xr-x` | unchanged — permissions come next |

---

## Step 4 — Set permissions

```bash
sudo chmod 070 /data/engineers
```

**Breaking down `070`:**

| Position | Who | Value | Permissions | Math |
|----------|-----|-------|-------------|------|
| `0` | Owner (tom) | 0 | `---` no access | 0 |
| `7` | Group (engineers) | 7 | `rwx` full access | 4+2+1 |
| `0` | Others | 0 | `---` no access | 0 |

> **The key trick:** `tom` gets `0` (no rwx) but still owns the directory. Ownership and permissions are separate — the owner always retains the right to `chmod`, even when their own permissions say `---`.

**Verify:**
```bash
ls -ld /data/engineers
```

**Expected output:**
d---rwx---. 2 tom engineers 6 May 20 11:14 /data/engineers

---

## Step 5 — Full verification

```bash
ls -ld /data/engineers
id tom
getent group engineers
```

**Reading the final `ls -ld` output:**

| Part | Requirement met |
|------|----------------|
| `d---` | Owner (tom) has no rwx — but can still `chmod` ✓ |
| `rwx` | Group (engineers) has full access ✓ |
| `---` | Others have no access ✓ |
| `tom` | Tom is the owning user ✓ |
| `engineers` | Engineers is the owning group ✓ |

All 3 requirements satisfied:
- Requirement 1 — `tom` = `---` ✓
- Requirement 2 — `engineers` = `rwx` ✓
- Requirement 3 — others = `---` ✓

## 🧠 Key Concepts

| Concept | What it means |
|---------|--------------|
| `mkdir -p` | Creates directory + all missing parents in one shot |
| `groupadd` | Adds a new group to `/etc/group` |
| `useradd` | Adds a new user to `/etc/passwd` and creates home dir |
| `chown user:group` | Sets both owner and group in one command |
| `chmod 070` | Octal notation — owner=0, group=7, others=0 |
| User Private Group | Every new user gets a private group of the same name by default |
| Ownership vs permissions | Owner can always `chmod` even if permissions are `---` |
| `getent` | Read-only lookup tool for system databases |
| `ls -ld` | `-l` = long format, `-d` = show the directory itself not its contents |

---

## ⚠️ Pitfalls

- **Forgetting `-p` on `mkdir`** → fails if `/data` doesn't exist yet
- **`chown tom engineers`** (space instead of colon) → tries to set owner to `tom` on a file called `engineers` — always use `tom:engineers`
- **`chmod 770` instead of `070`** → gives tom rwx too, violating requirement 1
- **Confusing ownership and permissions** → tom owning the dir doesn't mean tom has rwx; they are independent
- **Not verifying with `ls -ld`** → a wrong `chmod` is silent; always confirm

---

## ✅ Lab Checklist

- `/data/engineers` directory created with `mkdir -p` ✓
- Group `engineers` created with `groupadd` ✓
- User `tom` created with `useradd` ✓
- `chown tom:engineers /data/engineers` sets correct ownership ✓
- `chmod 070 /data/engineers` sets correct permissions ✓
- `ls -ld` shows `d---rwx--- tom engineers` ✓

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
