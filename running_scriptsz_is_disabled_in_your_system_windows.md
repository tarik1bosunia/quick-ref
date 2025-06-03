# How to Fix PowerShell Error: "Running Scripts is Disabled"

When running the command `npx create-next-app@latest .`, you might encounter the following error:

```plaintext
npx : File C:\Program Files\nodejs\npx.ps1 cannot be loaded because running scripts is disabled on this system. 
For more information, see about_Execution_Policies at https:/go.microsoft.com/fwlink/?LinkID=135170.
At line:1 char:1
+ npx create-next-app@latest .
+ ~~~
    + CategoryInfo          : SecurityError: (:) [], PSSecurityException
    + FullyQualifiedErrorId : UnauthorizedAccess
```

## Summary of Commands

| Command                                             | Purpose                                             |
|-----------------------------------------------------|-----------------------------------------------------|
| `Get-ExecutionPolicy`                               | Check the current execution policy.                |
| `Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass` | Temporarily allow scripts to run.                  |
| `Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser` | Permanently allow signed scripts for the user.     |
| `Set-ExecutionPolicy Restricted -Scope CurrentUser` | Restore the default script restriction.            |
