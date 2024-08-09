# Introduction to SSH



## Shell, Telnet and SSH

### What is a Shell

A shell is a command-line interface that allows users to interact with an operating system. It interprets user commands and acts as an intermediary between the user and the OS kernel.

<details>
  <summary>Shells on Windows, Linux and macOS</summary>
  <ul>
    <li>Shells provide features like command execution, scripting, file manipulation, and process management.</li>
    <li>Windows provides the <b>cmd</b> shell and the <b>Power Shell</b>; <b>bash</b> is becoming available as well.</li>
    <li>Common shells on Linux and MacOS are <b>bash</b> and <b>zsh</b>.</li>
  </ul>
</details>

> [!TIP]  
>
> Use `echo $SHELL` command on Linux and macOS to find out the shell you are using.

### What is Telent

**Telnet** (short for "teletype network")  is a network protocol that allows a user to remotely access and control another computer over the Internet or local area network (LAN). It enables a user to establish a connection to a remote system and perform tasks as if they were sitting in front of that computer.

![telnet](./ssh.assets/telnet.webp) 

Credit: www.cloudns.net

<details>
  <summary>More details about Telent</summary>
  <ul>
    <li>It uses the Transmission Control Protocol (TCP) as its underlying transport protocol.</li>
    <li>It is platform-independent, which means that it can be used to connect to a variety of different operating systems and computers.</li>
    <li>Security concerns: no encryption, no server identity verification, no integrity checks</li>
  </ul>
</details>

### What is SSH

Excerpt from https://www.digitalocean.com/community/tutorials/ssh-essentials-working-with-ssh-servers-clients-and-keys

> The most common way of connecting to a remote Linux server is through SSH. SSH stands for Secure Shell and provides a safe and secure way of executing commands, making changes, and configuring services remotely. When you connect through SSH, you log in using an account that exists on the remote server.

![ssh-diagram](./ssh.assets/ssh-diagram.png) 

<details>
  <summary>Key differences from Telnet</summary>
  <ul>
    <li>Encryption: SSH encrypts all data, including login credentials, while Telnet transmits data in plain text.</li>
    <li>Authentication: SSH uses public-key cryptography for stronger authentication.</li>
    <li>Integrity: SSH ensures data integrity, detecting any tampering during transmission.</li>
    <li>SSH Host Key is a cryptographic key used for authenticating computers.</li>
  </ul>
</details>