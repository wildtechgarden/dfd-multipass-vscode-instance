# Creating a Multipass instance for use with VSCode

## Prerequisites

* [Multipass](https://multipass.run/) has been installed.
* [VSCode (Visual Studio Code)](https://code.visualstudio.com/) has been
installed.
* [Git](https://git-scm.com/download) has been installed.
* [Remote - SSH extension for
VSCode](https://code.visualstudio.com/docs/remote/ssh) has been installed.
* [GPG-Indicator extension for
VSCode](https://marketplace.visualstudio.com/items?itemName=wdhongtw.gpg-indicator)
has been installed (if you use GnuPG to sign your commits).
* You know how to use VSCode with a Git provider such Gitlab, GitHub,
BitBucket, or SourceHut.

### For systems using Oracle VirtualBox as the driver for Multipass

For example because your system does not support Hyper-V (for Windows host),
or because you need VirtualBox for other reasons and this conflicts with the
default hypervisor.

#### For Windows systems

* `PsExec` from SysInternals installed. This can be installed from the Microsoft Store
(<https://apps.microsoft.com/store/detail/sysinternals-suite/9P7KNL5RWT25>) or via [Chocolatey](https://chocolatey.org).

## Daniel's Method

### Base instance creation

1. Create a [cloud-init YAML file to provision the instance (e.g.
`vscode-dev.yaml`)](vscode-dev.yaml) [\[Note 1\]](#note-1).
2. Launch Multipass using the `cloud-init` YAML file (e.g. `vscode-dev.yaml`)
   1. Change to directory containing `vscode-dev.yaml`
   2. Execute the following command:

      * For Windows, in a PowerShell:

        ```powershell
         multipass launch -c 4 -d 40G -m 4G -n hugo-dev --cloud-init .\vscode-dev.yaml --timeout 2400
         ```

      * For Linux, in a bash shell:

        ```bash
         multipass launch -c 4 -d 40G -m 4G -n hugo-dev --cloud-init ./vscode-dev.yaml --timeout 2400
        ```

      See [\[Note 2\]](#note-2)

### Additional steps when using Oracle VirtualBox as the driver for Multipass on Windows

1. Change to the location containing the PsExec command (unless you
have added it to your path). For example:

   ```powershell
   cd 'C:\Users\DanielDickinson\AppData\Local\Microsoft\WindowsApps'
   ```

2. Forward a port (in this case `22222`) on the local machine to Multipass
instance's SSH

   ```powershell
   PsExec -s $Env:VBOX_MSI_INSTALL_PATH\VBoxManage.exe controlvm "primary" natpf1 "sshvs,tcp,,22222,,22"
   ```

### Steps for accessing the Multipass instance from VSCode

1. Launch VSCode
2. From the 'Command Palette' (Window: Ctrl+Shift+P), select
"Remote-SSH: Connect to Host…" (alternatively use the 'remote' icon in the
bottom-left corner of the full VSCode window)
3. Scroll to the bottom of the list, and select "Configure SSH Hosts…"
4. Select the `.ssh/config` file in your home directory (Linux) or your
user profile directory (Windows).
5. Add a section such as:

   ```plaintext
   Host mp-dev
       Hostname 172.44.32.88
       Port 22
       User ubuntu
       IdentityFile ~/.ssh/your-ssh-private-key
       AddKeysToAgent yes
   ```

   Where `172.44.32.88` is the IP address of the instances (for
   non-VirtualBox-based Multipass), or `127.0.0.1`
   for VirtualBox-based Multipass AND change `Port 22` to the port
   you used above (`Port 22222` in the example above).
6. Repeat Step `2.` (only)
7. Select the host you defined (`mp-dev` in the example)
8. Wait while VSCode adds and launches the VS Code server on the remote.
9. You should now have VSCode connected to your Multipass instance. You can
use it mostly the way you would a local instance (but see
[Remote:Overview](https://code.visualstudio.com/docs/remote/remote-overview),
and [Remote:SSH](https://code.visualstudio.com/docs/remote/ssh) for
more details.

## Recommended additional steps

### Create a git-backed workspace

1. Create a new git repository on your hosting service of choice
(e.g. Gitlab, GitHub, BitBucket, SourceHut, etc).
2. Clone the project repository into your home directory (e.g.
as `mp-dev-workspace`)
3. In VSCode select `File|Open Folder`
4. Create a new folder named `.vscode` in the folder created by step `2.`
5. Select `File|Save Workspace As…`
6. Save the workspace into the `.vscode` folder you created.

### Add extensions you wish to use to the remote, where possible

1. Add the extensions you want to use locally (see [Extension
Marketplace](https://code.visualstudio.com/docs/editor/extension-marketplace)).
2. For the extensions you want in your remote workspace, on the
'Install in SSH: mp-dev-workspace` button in the `Extensions`
sidebar windows (where `mp-dev-workspace` is the name of your
actual workspace).
3. In addition, for those same extensions, click on the settings icon (gear)
for each extension and select `Add to Workspace Recommendations`.
4. Select the `mp-dev-workspace` workspace from the list (for example), and
click `Ok`.
5. Commit the resulting changes to the `.vscode` directory and push
you new commits to your hosting service.
6. Select `File|Close Workspace`.
7. Reopen the workspace.
8. You extensions should now be present on the remote.

## Recommendation for users with multiple devices

For any repositories you wish to include in your workspace, use them as
Git submodules. This makes it much easier to sync difference devices (e.g
a laptop and a desktop with both your extensions and your code/content).

Since VSCode does not automate the addition and use of submodules, you will
need to note the following:

1. Do not add a repository to the workspace directory by cloning to the
workspace directory. Instead use a command such as:

   ```bash
   git submodule add --name cool-submodule \
   -- https://gitlab.com/useryou/a-cool-submodule.git a-cool-submodule
   ```

2. Make sure you keep your submodules and the base repo commits up to
date and pushed to your hosting service so that you can move your work to
your other device(s) at any time.
3. If you clone the workspace repository to a new device using the
VSCode GUI, you will so need to do the following:
   1. Open a terminal in your workspace on the new device
   2. Execute `git submodule update --init --recursive -- .`

   You should now have your full set of code on the new device.
4. If you clone the repository to a new device using the command line, use:
`git clone --recursive …`

## Colophon

* [Copyright and licensing](LICENSE)

-------

## Notes

### Note 1

#### Outline of contents in sample `cloud-init` YAML file

1. Packages to add (and related configuration) using `apt`:
   This is arguably the most important part as without it one only gets a
   bare-bones instance.
2. SSH public key for the private key you will use to access the instance.
   This helps secure your connection and makes it more convenient by using
   a public/private keypair instead of entering a password for every connection
   to the remote host.
3. Files to add to the instance:
   This should be kept to a minimum due to size limitations (see
   [\[Note 1\]](#note-1)). The author uses this for configuration on which he
   relies.
4. Commands to run for additional provisioning:
   In the example there are two main sections:
   1. Copying files from step `3.` to their final destination and setting
   permissions and owner:group correctly. The author uses this order
   because `write_files` copies the files to the instance before users
   are created, and most of the files he adds are intended for the default
   user.
   2. Adding software for which the default `apt` repositories do not have
   the desired version (usually `apt` is fairly old versions of packages
   like `Go` and `Node.js`) or for which there is no version in the default
   `apt` repositories.

#### Base64 gzipped content

##### Example Base64 gzipped content

In the example cloud-init YAML file you will notice sections like:

```yaml
content: |
  H4sIAAAAAAAAA5WTXU+DMBSG7/srTnAZV9B7TS+WuYgXc2aZ3phlKR9bG+lHaCEas/9uUeaYEJTL
  vu+Th3OaQnNODSSa+ImGgPuIfgWiIr6oWkEhiF+IVlBx4ksqlY9Q9qZVYeHhebm7vV8TbxKtlgsc
  ykp4CL1AYMCbNCV2YWiYB1uYTiHsKeAKNowbyBVNDbj0ZPhoyCOOqWG7RAmdZ5Yr2ZINMB0v/EIQ
  Eq8pLyDQtaheoWMJU7cQ34ObKAVcmgLnKqE5PigccwnbG7AskwjgcbaJSAe4dt66OXoOaS6tPqM9
  b2krWuCUWor1u2VK9pp7mH/Jf/bqOlvVKFX4veGA8UyMEvdf6mU7blKpRXDIVfzHuJfY4Ceac/R0
  t9rNZ/NoUb9/zJTIcBmX0paYlQcVJDRh2Ql2zGa1JvW/gz4BNjw2aX4DAAA=
encoding: gz+b64
```

##### Reasons for using Base64 gzipped content

There two main reasons for use Base64 gzipped content in your `cloud-init`
YAML file:

1. The file you want to include contains binary, or text data that does not
'play well' with YAML encoding. For example a `tar` file of a subdirectory.
2. To keep the size of the `cloud-init` file smaller. Many cloud providers
impose a size limit (e.g. AWS was (is?) 16KB, OVH (OpenStack-based) was (is?)
32KB) on the userdata (cloud-init YAML in this case) one can pass to the
instance being created. Using Gzip reduces the size of the file, and using
base64 encoding ensures the resulting binary data is not mangled by the
YAML parser.

**Note**: Be very careful not to include private information ('secrets') in
this file in public repositories, as Base64 is just a different encoding,
it is not encryption.

##### Placeholders in the sample YAML for private data

A final note for the Base64 gzipped content. In some cases the example does
not provide actual data because the data would contain private information
('secrets') which do not belong in a public repository.

For example there is this placeholder:

```plaintext
Linux: Create file from existing ~/.gnupg with the following command:

tar -czf - ~/.gnupg | base64

and copy the output into this content section (with correct indentation)
instead of this instructional text.
```

I recommend copy `vscode-dev.yaml` to `vscode-dev-secrets.yaml` so that
the `.gitignore` in this repo will prevent storing the version that includes
secrets.

### Note 2

* For `-c 4` you may wish to adjust the 4 (number of CPU cores to give the
instance) depending on your hardware.
* Likewise for `-m 3G` the `3G` memory size may need to be adjusted for
your use case and hardware.
* Similarly, `-d 40G` the `40 G` for the virtual hard drive may be too large
for the storage you have, or too small for your development use case.
* The timeout (`--timeout 2400`) means that the instance is allowed to take up
to 40 minutes to finish initializing (including `cloud-init`). The default is
`300` (seconds = 5 minutes) which this author finds too short a time when using
`cloud-init` to provision the instance, on his hardware.
