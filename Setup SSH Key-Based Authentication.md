## Setup SSH Key-Based Authentication

> [!IMPORTANT]
>
> Before you begin, take your time and read the entire document at least twice with full attention so you truly understand it. Do not just copy and paste chunks hoping everything will work out of the box. Careful reading and real comprehension beat blind trial-and-error every single time.

### What and Why

[What is SSH Public Key Authentication?](https://www.ssh.com/academy/ssh/public-key-authentication)

>The motivation for using public key authentication over simple passwords is security. Public key authentication provides cryptographic strength that even extremely long passwords can not offer. With [SSH](https://www.ssh.com/ssh/), public key authentication improves security considerably as it frees the users from remembering complicated passwords (or worse yet, writing them down).
>
>In addition to security public key authentication also offers usability benefits - it allows users to implement single sign-on across the [SSH servers](https://www.ssh.com/ssh/server) they connect to. Public key authentication also allows automated, passwordless login that is a key enabler for the countless secure automation processes that execute within enterprise networks globally.

### Terms

- **OpenSSH**: OpenSSH is a connectivity tool for remote sign-in that uses the SSH protocol. It encrypts all traffic between client and server to eliminate eavesdropping, connection hijacking, and other attacks.
- **SSH Target**: SSH server, the Linux you want to log in.
- **`ssh-keygen`**: a standard component of the [Secure Shell](https://en.wikipedia.org/wiki/Secure_Shell) (SSH) protocol suite. The ssh-keygen utility is used to generate, manage, and convert authentication keys.
- **SSH config file**: OpenSSH client-side configuration file is named `config`, and it is stored in the `.ssh` directory under the user’s home directory.
- **`ssh` command**: OpenSSH remote login client.
- **SSH client**: The program (like `ssh` command in Windows Terminal, PuTTY, OpenSSH client, or MobaXterm) that you run on your local computer to securely connect to and control the remote SSH server.
- **`.ssh` folder**: `%USERPROFILE%\.ssh\` on Windows, `~/.ssh/` on Linux and macOS.

### Steps

https://github.com/sait-lab/devops/assets/81775267/df082d3e-b538-4cce-8c14-780dc3072481

> [!IMPORTANT]
> If you are generating ssh keys in Windows as SSH client, the following demo code works in **Command Prompt** instead of PowerShell.
>
> Always **generate** the key pair on the SSH client machine, **never directly on the remote** server.


1. On SSH client, verify that you can use username and password to connect to the SSH target. Once you successfully verify that, log out.
   ```shell
   ssh USERNAME_OF_SSH_TARGET@FQDN_OR_IP_ADDR_OF_SSH_TARGET
   ```

   ![verify-username-pwd-log-in](./Setup%20SSH%20Key-Based%20Authentication.assets/verify-username-pwd-log-in.jpeg) 

2. Generate an ed25519 key pair and save it to client's `.ssh` folder. If the folder does not exist, create it. `- C` specifies the comment. You can skip the `- C` part, but it's recommended when you have multiple keys to manage. Bruh… you skipped the giant purple warning paragraph right above step 1, didn’t you? Go back. Right now. Read every cursed word like it’s the group chat rules after someone already got kicked. Take your sweet time… we both know you were about to yeet yourself into the copy/paste anyway.

   ```text
   : On Microsoft Windows as SSH client. Do NOT do this on the SSH target.
   mkdir "%USERPROFILE%/.ssh"
   cd "%USERPROFILE%/.ssh"
   ssh-keygen -t ed25519 -f "%USERPROFILE%/.ssh/YOUR_KEY_NAME" -C "John Doe"
   
   : If your Windows does not take forward slash, try backward slash
   mkdir "%USERPROFILE%\.ssh"
   cd "%USERPROFILE%\.ssh"
   ssh-keygen -t ed25519 -f "%USERPROFILE%\.ssh\YOUR_KEY_NAME" -C "John Doe"
   
   : If you got an error, read important note above step 1.
   ```

   ```text
   # On Linux or macOS as SSH client. Do NOT do this on the SSH target.
   mkdir -p ~/.ssh
   cd ~/.ssh
   ssh-keygen -t ed25519 -f "~/.ssh/YOUR_KEY_NAME" -C "John Doe"
   ```

   [Using Ed25519 for OpenSSH keys (instead of DSA/RSA/ECDSA) (linux-audit.com)](https://linux-audit.com/using-ed25519-openssh-keys-instead-of-dsa-rsa-ecdsa/)

   This guide does **NOT** cover using a **passphrase** with your SSH key or how to manage it with `ssh-agent` / `ssh-add`.

   The examples below assume you create the key pair **without a passphrase** (just press Enter twice when asked for a passphrase during key generation).

   If you want to protect your private key with a [passphrase](https://www.ssh.com/academy/ssh/passphrase) (which is strongly recommended for security on shared or portable devices), you should do your own research: 

   - generate keys with a passphrase
   - use `ssh-agent` to cache the decrypted key
   - add the key to the agent (`ssh-add`)

   For most beginners and demo purposes, a passphrase-less key is simpler — but understand the security trade-off.

   The screenshot below shows using `ssh-keygen` to generate a key pair (`devops_vm_id.pub` and `devops_vm_id`). 
   ![ssh-keygen-demo](./Setup%20SSH%20Key-Based%20Authentication.assets/ssh-keygen-demo.jpeg) 

3. Copy the content of public key generated (the file with `.pub` filename extention) to the `~/.ssh/authorized_keys` file on the SSH target. If the `~/.ssh/` directory does not exist on the SSH target, create it.

   Display the public key on the client:

   ```cmd
   : On Microsoft Windows as SSH client.
   type "%USERPROFILE%\.ssh\YOUR_KEY_NAME.pub"
   
   : If you got an error, read important note above step 1.
   ```

   ```shell
   # On Linux or macOS as SSH client.
   cat ~/.ssh/YOUR_KEY_NAME.pub
   ```

   The following demo shows copying the content of public key `devops_vm_id` to the `~/.ssh/authorized_keys` file on the SSH target at `192.168.47.143`. The username of the SSH target is `student`.

   Replace `student` with your SSH target username. Replace `192.168.47.143` with your SSH target IP address.

   ```text
   ssh student@192.168.47.143 mkdir -p ~/.ssh
   ssh student@192.168.47.143 "echo 'CONTENT_YOUR_PUB_KEY' >> ~/.ssh/authorized_keys"
   ```

   ![ssh-copy-pub-id](./Setup%20SSH%20Key-Based%20Authentication.assets/ssh-copy-pub-id.jpeg) 
   **Note**: You can use `ssh-copy-id` command on Linux client. [What is ssh-copy-id? How ssh-copy-id works?](https://www.ssh.com/academy/ssh/copy-id)

4. `ssh` into the SSH target using key-based authentication using `-i` argument followed by the location of the private key file. If you can connect to the SSH target **without typing password**, you made it! Once you successfully verify that, log out.

   ```cmd
   : On Microsoft Windows as SSH client.
   ssh -i "%USERPROFILE%/.ssh/YOUR_KEY_NAME" USERNAME@FQDN_OR_IP_ADDR_OF_SSH_TARGET
   
   : If your Windows does not take forward slash, try backward slash
   ssh -i "%USERPROFILE%\.ssh\YOUR_KEY_NAME" USERNAME@FQDN_OR_IP_ADDR_OF_SSH_TARGET
   
   : If your Windows still complains about file not found
   cd "%USERPROFILE%\.ssh\"
   ssh -i YOUR_KEY_NAME USERNAME@FQDN_OR_IP_ADDR_OF_SSH_TARGET
   
   : If you got an error, read important note above step 1.
   ```

   ```shell
   # On Linux or macOS as SSH client.
   ssh -i ~/.ssh/YOUR_KEY_NAME USERNAME@FQDN_OR_IP_ADDR_OF_SSH_TARGET
   ```

   ![ssh-key-log-in](./Setup%20SSH%20Key-Based%20Authentication.assets/ssh-key-log-in.jpeg) 

5. OpenSSH allows you to set up a per-user configuration file where you can store different SSH options for each remote machine you connect to. Edit the SSH config file on your OpenSSH **client**.
   
   If you don't know how to create a `.ssh` subdirectory under your home folder, do your own research. If you don't have `"%USERPROFILE%\.ssh\config"` file on Microsoft Windows or `~/.ssh/config` file on Linux or macOS, create one. **If the SSH config already exists, append** the Host section for the SSH target to the config file.

   When creating the `config` file, make sure to save it as `config` without the `.txt` extension. Some text editors may automatically append `.txt` filename extension to the filename, so **double-check the file name before saving**.
   ![save-as](./Setup%20SSH%20Key-Based%20Authentication.assets/save-as.webp)
   
   After saving the `config` file, double-check if the filename has `.txt` extension.
   
   ```shell
   cd "%USERPROFILE%/.ssh/"
   dir config
   ```
   
   If you see 'File Not Found' in the output, most likely it's been saved as `config.txt`. In your **Command Prompt**, rename it by running the following command in **Command Prompt** instead of PowerShell:
   
   ```shell
   cd "%USERPROFILE%/.ssh/"
   ren config.txt config
   ```
   
   [SSH config file syntax and how-tos for configuring the OpenSSH client](https://www.ssh.com/academy/ssh/config)
   
   ```
   # Append the following section to your SSH config file on SSH client.
   # You can use "/" on Windows as path delimiter.
   # StrictHostKeyChecking and UserKnownHostsFile settings are recommended.
   
   Host NAME_FOR_SSH_TARGET
     HostName                FQDN_OR_IP_OF_SSH_TARGET
     User                    USERNAME_OF_SSH_TARGET
     IdentityFile            ~/.ssh/PRIVATE_KEY_FILENAME
     ServerAliveInterval     5
     ExitOnForwardFailure    yes
     StrictHostKeyChecking   no
     UserKnownHostsFile      /dev/null
   ```
   
   The screenshot below shows a `Host` section with name `devops-vm`.
   
   ![ssh-config-file](./Setup%20SSH%20Key-Based%20Authentication.assets/ssh-config-file.png)  
   
6. `ssh` into the SSH target using SSH configuration defined in SSH config file.

   ```shell
   ssh NAME_OF_HOST_SECTION
   ```

   **Note**: Use name of `Host` section (`devops-vm` in the screenshot below) instead of `HostName` (`192.168.47.143` in the screenshot below) of Host section.

   If you can connect to the SSH target **without typing password** by using the name of host section in your SSH profile, congratulations!

   ![ssh-use-config-file](./Setup%20SSH%20Key-Based%20Authentication.assets/ssh-use-config-file.jpeg) 
   
   
