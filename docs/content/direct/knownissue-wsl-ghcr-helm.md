Common WSL-related issue with KubeStellar setup, especially one tied to file permission issues, container platform mismatches, and installation hiccups.

---

## Common WSL Issues for Windows Users

## Issue: File permission errors when using Git or setting up the KubeStellar environment via WSL

**Type:** File System Permission Error (WSL on `/mnt/c/`)

## Root Cause

When WSL is used to access Windows file paths (e.g., `/mnt/c/...`), the mounted NTFS file system does not support Unix-style file permissions or ownership models. This causes commands like `chmod`, `git clone`, `kubectl`, or `kind` to fail silently or behave unexpectedly — especially when Git tries to create or lock config files.

## Test/Check

Run this command to confirm whether you're in a mounted Windows path:

```bash
pwd
```

If the path starts with `/mnt/c/`, `/mnt/d/`, etc., you're inside the Windows file system.

Check whether file ownership and permission commands fail:

```bash
touch testfile && chmod +x testfile
```

If this throws `Operation not permitted`, you're affected.

Also, try:

```bash
git status
```

If it returns "dubious ownership" or config lock errors, you're working in a non-Linux-native directory.

---

## Resolution (Recommended)

Use the **native Linux filesystem** within WSL to avoid permission issues:

1. Change directory to your WSL user home:

   ```bash
   cd ~
   ```

2. Clone the repo in a native Linux path (not `/mnt/c/`):

   ```bash
   git clone git@github.com:YourUsername/kubestellar.git
   ```

3. All development (building, editing, running scripts, kubectl, kind) should happen inside that folder in your WSL filesystem.

---

## Workaround (If you must use `/mnt/c`)

If you have to work on `/mnt/c` (e.g., to open files in VS Code for editing):

* Set Git to treat the directory as "safe":

  ```bash
  git config --global --add safe.directory /mnt/c/Users/YourName/Downloads/YourProject
  ```

* Add the following Git setting to avoid filemode issues:

  ```bash
  git config --global core.filemode false
  ```

> This won't fix chmod-related errors or other system-level permission issues — only mitigates Git behavior.

---

## Issue: Helm or container image pull errors with `ghcr.io` on WSL

**Type:** Network/Image Authentication Error

**Cause:** Docker on Windows might not authenticate properly with GitHub Container Registry from WSL, or Helm may fail to install due to permission errors when used in mounted drives.

### Suggested Test:

Try pulling an image manually:

```bash
docker pull ghcr.io/kubestellar/your-image
```

If this fails with a 403 or unauthorized error, it’s likely tied to auth or WSL integration.

### Resolution:

* Ensure Docker Desktop has WSL integration enabled.
* Run Docker or Helm commands as `root` within WSL:

  ```bash
  sudo helm install ...
  ```

## Workaround:

Manually pre-pull the container image from PowerShell using Docker Desktop (native Windows Docker):

```powershell
docker pull ghcr.io/kubestellar/your-image
```

Then switch back to WSL and use it.

---

## Other Helpful Tips

**VS Code integration:** Use the “Remote - WSL” extension to open your Linux-side project in VS Code cleanly.
**Path management:** Ensure `$HOME` paths (like `~/kubestellar`) are used consistently in your setup scripts.
**Testing tools:** Use `kind` and `kubectl` from WSL only if installed natively via `wget` or Scoop inside WSL.

---
