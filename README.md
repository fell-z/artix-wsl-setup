# Setup of Artix over WSL2

---

## Mentions & Sources

- [sp4d1n0's Repo about Artix over WSL2](https://github.com/sp4d1n0/Artix-WSL) -> Provided the base.
- [Artix's Official Wiki about migration](https://wiki.artixlinux.org/Main/Migration) -> The source above uses this to migrate an Arch installation to an Artix installation.
- [Article about OpenRC in WSL2](https://wsl.dev/wsl2init/) -> Gave me the simple command to make WSL2 start OpenRC.

---

## Step-by-step

### RootFS

*Obs: Normally I do this section in Ubuntu over WSL, so mind some commands and instructions which contain Linux-specific commands (e.g. `sudo`, `nano`, etc). But it should be possible in Windows too.*

1. We need to download the Arch bootstrap (Artix doesn't have one) from a mirror listed in [Arch Linux Downloads](https://archlinux.org/download/). Select the closest mirror to you for better downloads.

2. Extract the file (mind the extension of the tar file).

    ```bash
    tar -xzf archlinux-bootstrap-<YOUR_VERSION_HERE>-x86_64.tar.gz # for tar-gzip files
    ```

    ```bash
    tar --zstd -xf archlinux-bootstrap-<YOUR_VERSION_HERE>-x86_64.tar.zst # for tar-zstd files
    ```

3. Enter the extracted folder (typically named `root.x86_64`), it's good to do some things beforehand.
  3.1. Make some mirrors available in `/etc/pacman.d/mirrorlist` as they come all commented out. Use an editor of your choice, e.g. vim, nano, etc.
  3.2. Set the `SigLevel` in `/etc/pacman.conf` to `Never`.

    ```conf
    # /etc/pacman.conf
    ...
    #SigLevel = Required DatabaseOptional
    SigLevel = Never
    ```

4. Still inside the `root.x86_64` folder, compress the whole folder to a tar.gz file.

    ```bash
    sudo tar -czpf ../root.tar.gz *
    ```

5. Move the compressed file to a folder in Windows. e.g. `C:\Users\<username>\Documents\`

6. Import the distro using the `--import` option of the `wsl.exe` command.

    ```powershell
    wsl --import <YourDistroName> <InstallLocation> <DistroFile>
    ```

    - `<YourDistroName>` is simply the name you give the distro in WSL.
    - `<InstallLocation>` is the path where the filesystem file is installed (normally I set in a folder alongside the `root.tar.gz` file)
    - `<DistroFile>` is the path to the `root.tar.gz` file.

7. Log in to the now imported distro.

    ```powershell
    wsl -d <YourDistroName>
    ```

---

### Convert from Arch to Artix
