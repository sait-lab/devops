## GitHub Key-Based SSH Authentication

https://github.com/sait-lab/devops/assets/81775267/9268b544-0b87-4dd7-a7a7-40e061061be8

1. Generate an ed25519 key pair on the client. Use a meaningful file name instead of default one.
   [Generating a new SSH key and adding it to the ssh-agent - GitHub Docs](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent)

   ```shell
   mkdir -p ~/.ssh
   cd ~/.ssh
   ssh-keygen -t ed25519 -C "YOUR_EMAIL_ADDRESS"
   ```

   <img src="./GitHub%20Key-Based%20SSH%20Authentication.assets/create-ed25519-key-pair.jpg" alt="create-ed25519-key-pair" style="zoom:50%;" /> 

2. Add the created public key to your GitHub Account.
   [Adding a new SSH key to your GitHub account - GitHub Docs](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account)


   In the upper-right corner of any page, click your profile photo, then click **Settings**. In the "Access" section of the sidebar, click **SSH and GPG keys**.

   <img src="./GitHub%20Key-Based%20SSH%20Authentication.assets/github-user-profile.jpg" alt="github-user-profile" style="zoom: 50%;" />
   <img src="./GitHub%20Key-Based%20SSH%20Authentication.assets/select-ssh-n-gpg-keys.jpg" alt="select-ssh-n-gpg-keys" style="zoom: 33%;" /> 

Click on green "New SSH key" button to add your public key (the content of .pub file)
   <img src="./GitHub%20Key-Based%20SSH%20Authentication.assets/paste-pub-key.jpg" alt="paste-pub-key" style="zoom:50%;" />
   Verify the key fingerprint
   <img src="./GitHub%20Key-Based%20SSH%20Authentication.assets/match-fingerprints.jpg" alt="match-fingerprints" style="zoom:50%;" /> 

3. Create/Modify ssh config file on the client.
   Google "ssh config example github" [ssh config example github - Google Search](https://www.google.com/search?q=ssh+config+example+github&newwindow=1&sca_esv=6cca4d99aac7a0e4&sxsrf=ADLYWIIMbo8W3ODlv-a_KHnywMFtw-DrsQ%3A1716953637887&ei=JaJWZv3qNazw0PEPz5yEkAo&ved=0ahUKEwi9kbrd9rGGAxUsODQIHU8OAaIQ4dUDCBA&oq=ssh+config+example+github&gs_lp=Egxnd3Mtd2l6LXNlcnAiGXNzaCBjb25maWcgZXhhbXBsZSBnaXRodWIyChAAGLADGNYEGEcyChAAGLADGNYEGEcyChAAGLADGNYEGEcyChAAGLADGNYEGEcyChAAGLADGNYEGEcyChAAGLADGNYEGEcyChAAGLADGNYEGEcyChAAGLADGNYEGEdIrAVQAFgAcAF4AZABAJgBAKABAKoBALgBDMgBAJgCAaACBpgDAIgGAZAGCJIHATGgBwA&sclient=gws-wiz-serp)

   <img src="./GitHub%20Key-Based%20SSH%20Authentication.assets/ssh-config-example.jpg" alt="ssh-config-example" style="zoom:50%;" />
   Content of `~/.ssh/config` file:

   ```
   Host github.com
           User git
           Hostname github.com
           PreferredAuthentications publickey
           IdentityFile /home/student/.ssh/YOUR_PRIVE_KEY_FILE
   ```

4. Test the ssh configuration.
   ```
   ssh github.com
   ```

   You should see something like this
   <img src="./GitHub%20Key-Based%20SSH%20Authentication.assets/github-authn-success.jpg" alt="github-authn-success" style="zoom:50%;" /> 

5. Add the private key to the ssh-agent on the client.
   [Generating a new SSH key and adding it to the ssh-agent - GitHub Docs](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent?platform=linux)

   ```
   eval "$(ssh-agent -s)"
   ssh-add /home/YOUR_USERNAME/.ssh/YOUR_PRIVE_KEY_FILE
   ```

   <img src="./GitHub%20Key-Based%20SSH%20Authentication.assets/ssh-id-added.jpg" alt="ssh-id-added" style="zoom:50%;" /> 

6. Set your user name and email address for Git commit on the client.
   https://git-scm.com/book/en/v2/Getting-Started-First-Time-Git-Setup

   ```shell
   git config --global user.email "you@example.com"
   git config --global user.name "Your Name"
   ```

7. Create a **private** repository and add a README file using web browser: [Quickstart for repositories - GitHub Docs](https://docs.github.com/en/repositories/creating-and-managing-repositories/quickstart-for-repositories)

   <img src="./GitHub%20Key-Based%20SSH%20Authentication.assets/new-repo-1.jpg" alt="new-repo-1" style="zoom:50%;" /> 


   ![new-repo-2](./GitHub%20Key-Based%20SSH%20Authentication.assets/new-repo-2.jpg) 

8. Verify that you can `git clone YOUR_REPO` to your client.
   Copy the repository's ssh url
   <img src="./GitHub%20Key-Based%20SSH%20Authentication.assets/copy-ssh-url.jpg" alt="copy-ssh-url" style="zoom:50%;" /> 
   Clone your repository using the ssh url copied to `~/demo` foler.


   ```
   git clone YOUR_REPO_SSH_URL ~/demo
   ```

   If the clone is successful, **congratulations**!

   <img src="./GitHub%20Key-Based%20SSH%20Authentication.assets/clone-private-repo-ssh-url.jpg" alt="clone-private-repo-ssh-url" style="zoom:50%;" /> 

9. To avoid entering ssh-agent commands each time you log into your Linux client, add ssh-agent commands to your shell rc file (`~/.bashrc` for bash, `~/.zshrc` for zsh)
   [Shell initialization files (tldp.org)](https://tldp.org/LDP/Bash-Beginners-Guide/html/sect_03_01.html)

   ```
   echo 'eval "$(ssh-agent -s)"' >> ~/.bashrc
   echo 'ssh-add /home/YOUR_USERNAME/.ssh/YOUR_PRIVE_KEY_FILE' >> ~/.bashrc
   ```

   <img src="./GitHub%20Key-Based%20SSH%20Authentication.assets/add-ssh-agent-rc.jpg" alt="add-ssh-agent-rc" style="zoom:50%;" /> 
