
# 🔍 How to Know Which GitHub Account is Currently Connected to Your Device

Use the following methods to determine which GitHub account is currently active or configured on your system.

---

## ✅ 1. Check Git User Configuration

### 🔹 Global Git Identity
This applies to all repositories unless overridden locally.

```bash
git config --global user.name
git config --global user.email
```

### 🔹 Local Git Identity
This applies only to the current Git repository.

```bash
git config user.name
git config user.email
```

> 📝 This shows which name/email Git will use for commits in this repo.

---

## ✅ 2. Check Which SSH Key is Being Used

Run this to test your current GitHub SSH connection:

```bash
ssh -T git@github.com
```

You should see a response like:

```
Hi your-username! You've successfully authenticated, but GitHub does not provide shell access.
```

### If using multiple SSH keys with aliases:
Run using the alias you set in `~/.ssh/config`:

```bash
ssh -T git@github.com-personal
ssh -T git@github.com-work
```

---

## ✅ 3. Check Git Remote URL

Inside a Git repository, check the remote origin URL:

```bash
git remote -v
```

Example output:

```
origin  git@github.com-personal:your-username/repo.git (fetch)
origin  git@github.com-personal:your-username/repo.git (push)
```

You can tell:
- The GitHub alias (`github.com-personal`)
- The account or username in the URL

---

## ✅ 4. Check Stored Credentials (if using HTTPS)

If you're using HTTPS instead of SSH, Git may store credentials locally.

### Check credential helper:
```bash
git config --global credential.helper
```

### Check stored credentials:
- **Windows**: Open "Credential Manager" → "Windows Credentials" → Look for `github.com`
- **macOS**: Use Keychain Access
- **Linux**: Check `~/.git-credentials` or your credential manager

---

## ✅ 5. View SSH Keys on Your System

List existing SSH public keys:

```bash
ls ~/.ssh/*.pub
```

Print contents of each:

```bash
cat ~/.ssh/id_ed25519.pub
cat ~/.ssh/id_ed25519_personal.pub
cat ~/.ssh/id_ed25519_work.pub
```
