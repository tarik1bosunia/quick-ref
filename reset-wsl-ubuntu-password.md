# Reset Forgotten WSL Ubuntu Password

## Step 1: Find Your WSL Distribution Name
Open **PowerShell** as Administrator and run:

```powershell
wsl -l -v
```

This will list all installed WSL distributions. Note the name of the one you want to reset the password for.

## Step 2: Start WSL as the Root User
Replace `<distro_name>` with your actual distribution name (e.g., `Ubuntu-22.04`):

```powershell
wsl -d <distro_name> --user root
```

This starts WSL as the root user, which has full control.

## Step 3: Reset the User Password
Once inside WSL as root, run:

```bash
passwd <your_username>
```

You'll be prompted to enter a new password.

## Step 4: Exit and Restart WSL
Exit the root session:

```bash
exit
```

Then restart WSL normally:

```powershell
wsl
```

Now, log in with your username and the new password.

---

