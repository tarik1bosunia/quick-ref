# Restore "Open with Code" Context Menu in Windows

This guide will help you restore the **"Open with Code"** right-click menu option in Windows for **Visual Studio Code**.

---

## 1️⃣ Create the `.reg` file

1. Open **Notepad**.
2. Paste the following content, replacing `YOURUSERNAME` with your Windows username:

```reg
Windows Registry Editor Version 5.00

; Add "Open with Code" for folders
[HKEY_CLASSES_ROOT\Directory\Background\shell\VSCode]
@="Open with Code"
"Icon"="\"C:\\Users\\YOURUSERNAME\\AppData\\Local\\Programs\\Microsoft VS Code\\Code.exe\""

[HKEY_CLASSES_ROOT\Directory\Background\shell\VSCode\command]
@="\"C:\\Users\\YOURUSERNAME\\AppData\\Local\\Programs\\Microsoft VS Code\\Code.exe\" \"%V\""

; Add "Open with Code" for files
[HKEY_CLASSES_ROOT\*\shell\VSCode]
@="Open with Code"
"Icon"="\"C:\\Users\\YOURUSERNAME\\AppData\\Local\\Programs\\Microsoft VS Code\\Code.exe\""

[HKEY_CLASSES_ROOT\*\shell\VSCode\command]
@="\"C:\\Users\\YOURUSERNAME\\AppData\\Local\\Programs\\Microsoft VS Code\\Code.exe\" \"%1\""
```

---

## 2️⃣ Save & Import

1. In **Notepad**, go to **File → Save As**.
2. Change **Save as type** to **All Files**.
3. Name the file:
   ```
   open_with_code_fix.reg
   ```
4. Save it somewhere easy to find (e.g., Desktop).

---

## 3️⃣ Apply

1. **Double-click** the `.reg` file.
2. Accept all prompts (**Yes → Yes → OK**).
3. Restart File Explorer or **reboot your PC**.

---

✅ Now, when you right-click on a file or folder, you should see **"Open with Code"** in the context menu.
