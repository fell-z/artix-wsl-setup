# Setup of Artix over WSL2

## Mentions & Sources

- [sp4d1n0's Repo about Artix over WSL2](https://github.com/sp4d1n0/Artix-WSL) -> Provided the base.
- [Artix's Official Wiki about migration](https://wiki.artixlinux.org/Main/Migration) -> The source above uses this to migrate an Arch installation to an Artix installation.
- [Article about OpenRC in WSL2](https://wsl.dev/wsl2init/) -> Gave me the simple command to make WSL2 start OpenRC.

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
  3.2. If necessary, set the `SigLevel` in `/etc/pacman.conf` to `Never`.

    ```conf
    # /etc/pacman.conf
    ...
    #SigLevel = Required DatabaseOptional
    SigLevel = Never
    ...
    ```

4. Still inside the `root.x86_64` folder, compress the whole folder to a tar.gz file.

    ```bash
    sudo tar -czpf ../root.tar.gz *
    ```

5. Move the compressed file to a folder in Windows. e.g. `C:\Users\<username>\Documents\`.

6. Import the distro using the `--import` option of the `wsl.exe` command.

    ```powershell
    wsl --import <YourDistroName> <InstallLocation> <DistroFile>
    ```

    - `<YourDistroName>` is simply the name you give the distro in WSL.
    - `<InstallLocation>` is the path where the filesystem file is installed (normally I set in a folder alongside the `root.tar.gz` file)
    - `<DistroFile>` is the path to the `root.tar.gz` file.

7. Log in to the now imported distro.

    ```powershell
    wsl ~ -d <YourDistroName>
    ```

### Convert from Arch to Artix

1. Install a text editor for use later.

    ```bash
    pacman -Sy vim
    ```

2. Replace `/etc/pacman.conf` and `/etc/pacman.d/mirrorlist` with Artix's ones.

    ```bash
    mv -vf /etc/pacman.conf /etc/pacman.conf.arch
    curl https://gitea.artixlinux.org/packages/pacman/raw/branch/master/pacman.conf -o /etc/pacman.conf
    mv -vf /etc/pacman.d/mirrorlist /etc/pacman.d/mirrorlist-arch
    curl https://gitea.artixlinux.org/packages/artix-mirrorlist/raw/branch/master/mirrorlist -o /etc/pacman.d/mirrorlist
    cp -vf /etc/pacman.d/mirrorlist /etc/pacman.d/mirrorlist.artix
    ```

3. Clean up the packages cache and force-update the pacman database.

    ```bash
    pacman -Scc && pacman -Syy
    ```

4. Change the `SigLevel` in `/etc/pacman.conf` to `Never` using the text editor of choice.

    ```conf
    ...
    #SigLevel = Required DatabaseOptional
    SigLevel = Never
    ...
    ```

5. Init the keyring using `pacman-key` and install the Artix's keyring.

    ```bash
    pacman-key --init
    pacman -S artix-keyring
    pacman-key --populate artix
    pacman-key --lsign-key 95AEC5D0C1E294FC9F82B253573A673A53C01BC2
    ```

6. Change the `SigLevel` in `/etc/pacman.conf` back to `Required DatabaseOptional`.

    ```conf
    SigLevel = Required DatabaseOptional
    #SigLevel = Never
    ```

7. Download the base packages and an init.

    ```bash
    # This downloads openrc as the init.
    pacman -Sw base base-devel lsb-release connman esysusers etmpfiles artix-branding-base openrc-system elogind-openrc openrc
    ```

8. Remove systemd.

    ```bash
    pacman -Rdd --noconfirm systemd systemd-libs systemd-sysvcompat pacman-mirrorlist dbus
    ```

9. Copy the Artix's mirrorlist as the previous command deleted.

    ```bash
    cp -vf /etc/pacman.d/mirrorlist.artix /etc/pacman.d/mirrorlist
    ```

10. Install the packages downloaded with `pacman -Sw`.

    ```bash
    pacman -S base base-devel lsb-release connman esysusers etmpfiles artix-branding-base openrc-system elogind-openrc openrc
    ```

11. Define the locale to `C` using `export LC_ALL=C` in the shell.

12. Now replace all Arch packages with Artix's ones.

    ```bash
    pacman -Sl system | grep installed | cut -d" " -f2 | pacman -S -
    pacman -Sl world | grep installed | cut -d" " -f2 | pacman -S -
    pacman -Sl galaxy | grep installed | cut -d" " -f2 | pacman -S -
    ```

13. Remove the rest of systemd junk.

    ```bash
    for user in journal journal-gateway timesync network bus-proxy journal-remote journal-upload resolve coredump; do
      userdel systemd-$user
    done
    rm -vfr /{etc,var/lib}/systemd
    ```

14. Tell WSL2 to run OpenRC on boot in `/etc/wsl.conf`.

    ```conf
    ...
    [boot]
    command = "/usr/bin/env -i /usr/bin/unshare --pid --mount-proc --fork --propagation private -- sh -c 'exec /sbin/init'"
    ...
    ```

By now, the migration from Arch to Artix is done and the distro can be restarted to apply any leftover changes.

```powershell
wsl --terminate <YourDistroName>

wsl ~ -d <YourDistroName>
```

### Post-install

#### Init scripts (OpenRC)

1. Install some services.

    ```bash
    pacman -S --needed cronie-openrc fuse-openrc haveged-openrc hdparm-openrc openssh-openrc samba-openrc syslog-ng-openrc
    ```

2. Enable them.

    ```bash
    for daemon in cronie fuse haveged hdparm sshd smb syslog-ng; do rc-update add $daemon default; done
    ```

    2.1. Also enable `udev` if not already.
        ```bash
        rc-update add udev sysinit
        ```
