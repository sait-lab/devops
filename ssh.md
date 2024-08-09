# Introduction to SSH



## Shell, Telnet and SSH

### What is a Shell

A shell is a command-line interface that allows users to interact with an operating system. It interprets user commands and acts as an intermediary between the user and the OS kernel.

<details>
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

<details>
  <ul>
    <li>It uses the Transmission Control Protocol (TCP) as its underlying transport protocol.</li>
    <li>It is platform-independent, which means that it can be used to connect to a variety of different operating systems and computers.</li>
    <li>Security concerns: no encryption, no server identity verification, no integrity checks</li>
    <li>SSH is now preferred for secure remote management due to security concerns.</li>
  </ul>
</details>

### What is SSH