# SSH KEY GITHUB
### how to connect github with local machine

## 1. Check if SSH Key Exists
```sh
ls -al ~/.ssh
```

## 2. Generate a New SSH Key (if needed)
```sh
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```

## 3. Add the SSH Key to the SSH Agent
```sh
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_rsa
```

## 4. Add the SSH Key to Your GitHub Account

- Copy your SSH public key to the clipboard:
    ```sh
    cat ~/.ssh/id_rsa.pub
    ```
- Go to your [GitHub SSH keys page](https://github.com/settings/keys).
- Click "New SSH key", give it a title, and paste the public key you copied into the "Key" field.
- Click "Add SSH key".

## 5. Test the SSH Connection
```sh
ssh -T git@github.com
```