
# How to Use Two GitHub Accounts on a Single Device

You can use both personal and work GitHub accounts on the same computer by configuring SSH keys and Git settings separately.

---

## Step 1: Generate SSH Keys for Both Accounts

### Personal Account
```bash
ssh-keygen -t ed25519 -C "your_personal_email@example.com"
# Save as: /c/Users/YourName/.ssh/id_ed25519_personal
```

### Work Account
```bash
ssh-keygen -t ed25519 -C "your_work_email@example.com"
# Save as: /c/Users/YourName/.ssh/id_ed25519_work
```

---

## Step 2: Add SSH Keys to the SSH Agent

```bash
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519_personal
ssh-add ~/.ssh/id_ed25519_work
```

---

## Step 3: Add Public Keys to GitHub

Copy each public key:

```bash
cat ~/.ssh/id_ed25519_personal.pub
cat ~/.ssh/id_ed25519_work.pub
```

Paste them into your GitHub accounts under **Settings → SSH and GPG keys**.

---

## Step 4: Configure the SSH Config File

Edit or create `~/.ssh/config`:

```ssh
# Personal GitHub
Host github.com-personal
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_ed25519_personal

# Work GitHub
Host github.com-work
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_ed25519_work
```

---

## Step 5: Clone Repositories Using Aliased Host

### Personal
```bash
git clone git@github.com-personal:username/repo.git
```

### Work
```bash
git clone git@github.com-work:workusername/repo.git
```

---

## Step 6: Set Git User per Repository

Inside each repo directory:

### Personal Project
```bash
git config user.name "Your Name"
git config user.email "your_personal_email@example.com"
```

### Work Project
```bash
git config user.name "Your Work Name"
git config user.email "your_work_email@example.com"
```

---

## Optional: Avoid HTTPS Credential Conflicts

If using HTTPS and you want to disable cached credentials:

```bash
git config --global credential.helper ""
```

---

## Summary Table

| Purpose        | SSH Host Alias         | Email Used                        |
|----------------|------------------------|-----------------------------------|
| Personal       | `github.com-personal`  | `your_personal_email@example.com` |
| Work           | `github.com-work`      | `your_work_email@example.com`     |
