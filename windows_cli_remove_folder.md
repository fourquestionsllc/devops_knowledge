To remove a folder in Windows using the Command-Line Interface (CLI), you can use the `rmdir` or `del` command. Here's how:

---

### Remove an Empty Folder
If the folder is empty:
```cmd
rmdir FolderName
```

---

### Remove a Folder and Its Contents
If the folder contains files or subfolders, use the `/s` flag:
```cmd
rmdir /s FolderName
```

- **Example:**
  ```cmd
  rmdir /s MyFolder
  ```
  This will prompt you to confirm the deletion.

#### To bypass confirmation, use the `/q` flag:
```cmd
rmdir /s /q FolderName
```

- **Explanation:**
  - `/s`: Removes all files and subfolders within the folder.
  - `/q`: Quiet mode; does not ask for confirmation.

---

### Additional Tips
1. Ensure you have the necessary permissions to delete the folder.
2. If the folder path contains spaces, enclose the path in quotes:
   ```cmd
   rmdir /s /q "C:\Path To\Folder Name"
   ```
3. Run the Command Prompt as Administrator if you encounter permission issues.

---

If you need to remove files individually, use:
```cmd
del /s /q "C:\Path To\Folder\*"
rmdir /q "C:\Path To\Folder"
```

This ensures all contents are deleted before removing the folder.
