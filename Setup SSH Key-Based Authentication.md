## Setup SSH Key-Based Authentication

### Terms

- **OpenSSH**: OpenSSH is a connectivity tool for remote sign-in that uses the SSH protocol. It encrypts all traffic between client and server to eliminate eavesdropping, connection hijacking, and other attacks.
- **SSH Target**: SSH server, the Linux you want to log in.
- **`ssh-keygen`**: a standard component of the [Secure Shell](https://en.wikipedia.org/wiki/Secure_Shell) (SSH) protocol suite. The ssh-keygen utility is used to generate, manage, and convert authentication keys.
- **SSH config file**: OpenSSH client-side configuration file is named `config`, and it is stored in the `.ssh` directory under the user’s home directory.
- **`ssh` command/client**: OpenSSH remote login client.
- **`.ssh` folder**: `%USERPROFILE%\.ssh\` on Windows, `~/.ssh/` on Linux and macOS.


https://github.com/sait-lab/devops/assets/81775267/df082d3e-b538-4cce-8c14-780dc3072481


### Steps

1. Verify that you can use username and password to connect to the SSH target.
   ```shell
   ssh USERNAME_OF_SSH_TARGET@FQDN_OR_IP_ADDR_OF_SSH_TARGET
   ```

   ![verify-username-pwd-log-in](./Setup%20SSH%20Key-Based%20Authentication.assets/verify-username-pwd-log-in.jpeg) 

2. Generate an ed25519 key pair and save it to client's `.ssh` folder. If the folder does not exist, create it. `- C` specifies the comment.

   ```cmd
   : On Microsoft Windows
   mkdir "%USERPROFILE%/.ssh"
   cd "%USERPROFILE%/.ssh"
   ssh-keygen -t ed25519 -f "%USERPROFILE%/.ssh/YOUR_KEY_NAME" -C "John Doe"
   ```

   ```shell
   # On Linux or macOS
   mkdir -p ~/.ssh
   cd ~/.ssh
   ssh-keygen -t ed25519 -f ~/.ssh/YOUR_KEY_NAME" -C "John Doe"
   ```

   [Using Ed25519 for OpenSSH keys (instead of DSA/RSA/ECDSA) (linux-audit.com)](https://linux-audit.com/using-ed25519-openssh-keys-instead-of-dsa-rsa-ecdsa/)

   The screenshot below shows using `ssh-keygen` to generate a key pair (`devops_vm_id.pub` and `devops_vm_id`).
   ![ssh-keygen-demo](./Setup%20SSH%20Key-Based%20Authentication.assets/ssh-keygen-demo.jpeg) 

3. Copy the content of public key generated (the file with `.pub` filename extention) to the `~/.ssh/authorized_keys` file on the SSH target. If the `~/.ssh/` directory does not exist on the SSH target, create it.

   Display the public key on the client:

   ```cmd
   : On Microsoft Windows
   type "%USERPROFILE%\.ssh\YOUR_KEY_NAME.pub"
   ```

   ```shell
   # On Linux or macOS
   cat ~/.ssh/YOUR_KEY_NAME.pub
   ```

   The following demo shows copying the content of public key `devos_vm_id` to the `~/.ssh/authorized_keys` file on the SSH target at `192.168.47.143`. The username of the SSH target is `student`.
   ```cmd
   ssh student@192.168.47.143 mkdir -p ~/.ssh
   ssh student@192.168.47.143 "echo 'CONTENT_YOUR_PUB_KEY' >> ~/.ssh/authorized_keys"
   ```

   ![ssh-copy-pub-id](./Setup%20SSH%20Key-Based%20Authentication.assets/ssh-copy-pub-id.jpeg) 
   **Note**: You can use `ssh-copy-id` command on Linux client. [What is ssh-copy-id? How ssh-copy-id works?](https://www.ssh.com/academy/ssh/copy-id)

4. `ssh` into the SSH target using key-based authentication using `-i` argument followed by the location of the private key file. If you can connect to the SSH target **without typing password**, you made it!

   ```cmd
   : On Microsoft Windows
   ssh -i "%USERPROFILE%/.ssh/YOUR_KEY_NAME" USERNAME@FQDN_OR_IP_ADDR_OF_SSH_TARGET
   ```

   ```shell
   # On Linux or macOS
   ssh -i ~/.ssh/YOUR_KEY_NAME USERNAME@FQDN_OR_IP_ADDR_OF_SSH_TARGET
   ```

   ![ssh-key-log-in](./Setup%20SSH%20Key-Based%20Authentication.assets/ssh-key-log-in.jpeg) 

5. OpenSSH allows you to set up a per-user configuration file where you can store different SSH options for each remote machine you connect to. Edit the SSH config file on your OpenSSH **client**.
   
   If you don't have `"%USERPROFILE%\.ssh\config"` file on Microsoft Windows or `~/.ssh/config` file on Linux or macOS, create one. If the SSH config already exists, append the config for the SSH target to it.

   [SSH config file syntax and how-tos for configuring the OpenSSH client](https://www.ssh.com/academy/ssh/config)
   
   ```
   # Append the following section to your SSH config file on SSH client.
   # You can "/" on Windows as path delimiter.
   Host NAME_FOR_SSH_TARGET
     HostName                FQDN_OR_IP_OF_YOUR_SSH_TARGET
     User                    USERNAME_OF_SSH_TARGET
     IdentityFile            ~/.ssh/YOUR_PRIVATE_KEY_FILENAME
     ServerAliveInterval     5
     ExitOnForwardFailure    yes
   ```
   
   ![ssh-config-file](./Setup%20SSH%20Key-Based%20Authentication.assets/ssh-config-file.png)  
   
6. `ssh` into the SSH target using SSH options defined in SSH config file.

   ```shell
   ssh NAME_OF_HOST_SECTION
   ```

   **Note**: Use name of `Host` section (`devops-vm` in the screenshot below) instead of `HostName` (`192.168.47.143` in the screenshot below) of Host section.

   If you can connect to the SSH target **without typing password**, congratulations!

   ![ssh-use-config-file](./Setup%20SSH%20Key-Based%20Authentication.assets/ssh-use-config-file.jpeg) 
   
   
