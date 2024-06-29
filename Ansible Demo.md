## Ansible Demo

### Table of Contents

   * [Ansible Demo](#ansible-demo)
      * [Introduction](#introduction)
      * [Ansible installation guide](#ansible-installation-guide)
         * [Installing Ansible](#installing-ansible)
         * [Upgrading Ansible](#upgrading-ansible)
      * [Building Ansible inventories](#building-ansible-inventories)
         * [Adding ranges of hosts](#adding-ranges-of-hosts)
      * [YAML](#yaml)
      * [Using Ansible playbooks](#using-ansible-playbooks)
         * [Running playbooks in check mode](#running-playbooks-in-check-mode)
      * [Using Ansible modules and plugins](#using-ansible-modules-and-plugins)
         * [command – Execute commands on targets](#command--execute-commands-on-targets)
         * [debug – Print statements during execution](#debug--print-statements-during-execution)
      * [Using Variables](#using-variables)
         * [Defining variables in included files and roles](#defining-variables-in-included-files-and-roles)
      * [Loops](#loops)
         * [Iterating over a simple list](#iterating-over-a-simple-list)
         * [Registering variables with a loop](#registering-variables-with-a-loop)
      * [Organizing host and group variables](#organizing-host-and-group-variables)
      * [ansible.cfg](#ansiblecfg)
         * [The configuration file](#the-configuration-file)
      * [ansible.builtin.copy module – Copy files to remote locations](#ansiblebuiltincopy-module--copy-files-to-remote-locations)
      * [Understanding privilege escalation: become](#understanding-privilege-escalation-become)
            * [ansible.builtin.apt module](#ansiblebuiltinapt-module)



---



### Introduction

This demo setup includes one AlmaLinux 9.4 serving as the Ansible control node and two Ubuntu 24.04 LTS instances as the managed nodes.

> [!IMPORTANT]  
> This demonstration assumes you are familiar with SSH key-based authentication.
> [Setup SSH Key-Based Authentication](https://github.com/sait-lab/devops/blob/main/Setup%20SSH%20Key-Based%20Authentication.md)

### Ansible installation guide

https://docs.ansible.com/ansible/latest/installation_guide/index.html

https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html#installing-and-upgrading-ansible-with-pip

```shell
# Verify Python version
python3 -V
```

```
Python 3.9.18
```

```shell
# Install Python package installer on RHEL/AlmaLinux/Rocky Linux
sudo dnf install python3-pip
```
```shell
# Install Python package installer on Ubuntu/Debian
sudo apt-get install -y python3-pip
```

```shell
# Verify pip version
python3 -m pip -V

# Or

pip -V
```

```
pip 21.2.3 from /usr/lib/python3.9/site-packages/pip (python 3.9)
```

#### Installing Ansible

Use `pip` in your selected Python environment (https://docs.python.org/3/library/venv.html, https://github.com/pyenv/pyenv, https://conda.io/projects/conda/en/latest/index.html, https://github.com/mamba-org/mamba) to install the full Ansible package for the current user:

```shell
# Install Ansible on RHEL/AlmaLinux/Rocky Linux
python3 -m pip install --user ansible
```

#### Upgrading Ansible

To upgrade an existing Ansible installation in this Python environment to the latest released version, simply add `--upgrade` to the command above:

```shell
# Install Ansible on RHEL/AlmaLinux/Rocky Linux
python3 -m pip install --upgrade --user ansible
```
```shell
# Install Ansible on Debian/Ubuntu
# Create a work directory and enter it
mkdir ~/ansible && cd ~/ansible

# Create a virtual environment
python3 -m venv venv-ansible

# Activate virtual environment
. venv-ansible/bin/activate
# Or
. ~/ansible/venv-ansible/bin/activate
# Or
source venv-ansible/bin/activate
# Or
source ~/ansible/venv-ansible/bin/activate

# Upgrade pip
python -m pip install --upgrade pip

# Install Ansible
python -m pip install ansible
```
```shell
# Show Ansible's version number, config file location, configured module search path,
# module location, executable location and exit
ansible --version
```

```
ansible [core 2.17.1]
  config file = None
  configured module search path = ['/home/alma/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /home/alma/.local/lib/python3.9/site-packages/ansible
  ansible collection location = /home/alma/.ansible/collections:/usr/share/ansible/collections
  executable location = /home/alma/.local/bin/ansible
  python version = 3.x.x (main, Jan 24 2024, 00:00:00) [GCC 11.4.1 20231218 (Red Hat 11.4.1-3)] (/usr/bin/python3)
  jinja version = 3.1.4
  libyaml = True
```



---

Add `ansible-node1`, `ansible-node2` to `/etc/hosts`

```
IP_ADDRESS_OF_CONTROLNODE ansible-controlnode
IP_ADDRESS_OF_NODE1 ansible-node1
IP_ADDRESS_OF_NODE2 ansible-node2
```

`ssh` into both nodes.



---



### Building Ansible inventories

https://docs.ansible.com/ansible/latest/inventory_guide/index.html

```shell
mkdir -p ~/ansible-demo/inventory
cd ~/ansible-demo/inventory
touch ~/ansible-demo/inventory/inventory.ini
```

`inventory.ini`:

```ini
ansible-node1 ansible_connection=ssh ansible_user=ubuntu ansible_ssh_private_key_file=~/.ssh/YOUR_SSH_PRIVATE_KEY_FILE
ansible-node2 ansible_connection=ssh ansible_user=ubuntu ansible_ssh_private_key_file=~/.ssh/YOUR_SSH_PRIVATE_KEY_FILE
```

```shell
# ansible ping all nodes
ansible all -m ping -i inventory.ini
```

```
ansible-node1 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
ansible-node2 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
```
```shell
# ansible ping one node
ansible ansible-node1 -m ping -i inventory.ini
```

```
ansible-node1 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
```

https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_variables.html#valid-variable-names

Note: The headings in brackets are group names. **Dash ("-") is invalid**.

`inventory.ini` with groups:

```ini
[web_srv_1]
ansible-node1 ansible_connection=ssh ansible_user=ubuntu ansible_ssh_private_key_file=~/.ssh/YOUR_SSH_PRIVATE_KEY_FILE

[web_srv_2]
ansible-node2 ansible_connection=ssh ansible_user=ubuntu ansible_ssh_private_key_file=~/.ssh/YOUR_SSH_PRIVATE_KEY_FILE
```
```shell
# ping managed nodes in web_srv_1 group
ansible web_srv_1 -m ping -i inventory.ini
```
```
ansible-node1 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
```

#### Adding ranges of hosts

If you have a lot of hosts with a similar pattern, you can add them as a range rather than listing each hostname separately:

```ini
[web_srv]
ansible-node[1:2] ansible_connection=ssh ansible_user=ubuntu ansible_ssh_private_key_file=~/.ssh/YOUR_SSH_PRIVATE_KEY_FILE

#ansible-node2 ansible_connection=ssh ansible_user=ubuntu ansible_ssh_private_key_file=~/.ssh/YOUR_SSH_PRIVATE_KEY_FILE
```



---



### YAML

YAML guide https://circleci.com/blog/what-is-yaml-a-beginner-s-guide/

YAML Syntax https://docs.ansible.com/ansible/latest/reference_appendices/YAMLSyntax.html



---



### Using Ansible playbooks

https://docs.ansible.com/ansible/latest/playbook_guide/index.html

```shell
mkdir -p ~/ansible-demo/playbook && cd ~/ansible-demo/playbook
touch ~/ansible-demo/playbook/playbook1.yaml
```

Create `~/.ssh/config`

```
Host ansible-node1
  HostName                  IP_ADDRESS_OR_FQDN_OF_NODE1
  Port                      22
  User                      ubuntu
  IdentityFile              ~/.ssh/YOUR_SSH_PRIVATE_KEY_FILE
  ServerAliveInterval       5
  ExitOnForwardFailure      yes

Host ansible-node2
  HostName                  IP_ADDRESS_OR_FQDN_OF_NODE2
  Port                      22
  User                      ubuntu
  IdentityFile              ~/.ssh/YOUR_SSH_PRIVATE_KEY_FILE
  ServerAliveInterval       5
  ExitOnForwardFailure      yes
```

Create `inventory.ini` under `~/ansible-demo/playbook/`:

```ini
[web_srv]
ansible-node1 ansible_connection=ssh
ansible-node2 ansible_connection=ssh
```

```
$ ls -l ~/ansible-demo/playbook/
total 4
-rw-r--r--. 1 alma alma 84 Jun 18 00:56 inventory.ini
-rw-r--r--. 1 alma alma  0 Jun 18 00:53 playbook1.yaml
```

`playbook1.yaml`

```yaml
- hosts: web_srv
  name: play-demo
  tasks:
  - name: verify host connectivity
    ping:
```
```shell
# Use ping in playbook
ansible-playbook playbook1.yaml -i inventory.ini --verbose
```
```
No config file found; using defaults

PLAY [play-demo] ************************************************************

TASK [Gathering Facts] ******************************************************
ok: [ansible-node1]
ok: [ansible-node2]

TASK [verify host connectivity] *****************************************************************************
ok: [ansible-node2] => {"changed": false, "ping": "pong"}
ok: [ansible-node1] => {"changed": false, "ping": "pong"}

PLAY RECAP ******************************************************************
ansible-node1              : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
ansible-node2              : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

#### Running playbooks in check mode

https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_checkmode.html

Check mode is just a simulation. It will not generate output for tasks that use [conditionals based on registered variables](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_conditionals.html#conditionals-registered-vars) (results of prior tasks). However, it is great for validating configuration management playbooks that run on one node at a time.

```shell
ansible-playbook --check playbook1.yaml -i inventory.ini
```



---



### Using Ansible modules and plugins

https://docs.ansible.com/ansible/latest/module_plugin_guide/index.html

https://docs.ansible.com/ansible/latest/collections/index_module.html



You can execute modules from the command line.

#### command – Execute commands on targets

`ansible.builtin.command` module: https://docs.ansible.com/ansible/latest/collections/ansible/builtin/command_module.html

This module is part of `ansible-core` and included in all Ansible installations. In most cases, you can use the short module name `command` even without specifying the [collections keyword](https://docs.ansible.com/ansible/latest/collections_guide/collections_using_playbooks.html#collections-keyword). However, we recommend you use the [Fully Qualified Collection Name (FQCN)](https://docs.ansible.com/ansible/latest/reference_appendices/glossary.html#term-Fully-Qualified-Collection-Name-FQCN) `ansible.builtin.command` for easy linking to the module documentation and to avoid conflicting with other collections that may have the same module name.

```shell
# Recommended
ansible web_srv -m ansible.builtin.command -a "hostnamectl" -i inventory.ini

# Not recommended 
ansible web_srv -m command -a "hostnamectl" -i inventory.ini
```

```
ansible-node1 | CHANGED | rc=0 >>
 Static hostname: ansible-node1
       Icon name: computer-vm
         Chassis: vm
      Machine ID: 3e486d594e7bb2048e09d8e066712705
         Boot ID: 761d97b73559476bb2626e97362e32c9
  Virtualization: vmware
Operating System: Ubuntu 24.04 LTS
          Kernel: Linux 6.8.0-35-generic
    Architecture: x86-64
 Hardware Vendor: VMware, Inc.
  Hardware Model: VMware Virtual Platform
Firmware Version: 6.00
   Firmware Date: Thu 2020-11-12
    Firmware Age: 3y 7month 5d
ansible-node2 | CHANGED | rc=0 >>
 Static hostname: ansible-node2
       Icon name: computer-vm
         Chassis: vm
      Machine ID: 208764583c135fadff15af416671270f
         Boot ID: 3592de635eea4ef199972ccccfddee4f
  Virtualization: vmware
Operating System: Ubuntu 24.04 LTS
          Kernel: Linux 6.8.0-35-generic
    Architecture: x86-64
 Hardware Vendor: VMware, Inc.
  Hardware Model: VMware Virtual Platform
Firmware Version: 6.00
   Firmware Date: Thu 2020-11-12
    Firmware Age: 3y 7month 5d
```

From playbooks, Ansible modules are executed in a very similar way. `playbook2.yaml`

```yaml
- name: module-demo
  hosts: web_srv
  tasks:
  - name: query hostname and related settings
    ansible.builtin.command: hostnamectl
```

```shell
# Use ansible.builtin.command module in a playbook
ansible-playbook playbook2.yaml -i inventory.ini --verbose
```

```
No config file found; using defaults

PLAY [module-demo] **********************************************************************************************************

TASK [Gathering Facts] ******************************************************************************************************
ok: [ansible-node1]
ok: [ansible-node2]

TASK [query hostname and related settings] **********************************************************************************
changed: [ansible-node2] => {"changed": true, "cmd": ["hostnamectl"], "delta": "0:00:00.156681", "end": "2024-06-18 01:09:38.058298", "msg": "", "rc": 0, "start": "2024-06-18 01:09:37.901617", "stderr": "", "stderr_lines": [], "stdout": " Static hostname: ansible-node2\n       Icon name: computer-vm\n         Chassis: vm \n      Machine ID: 208764583c135fadff15af416671270f\n         Boot ID: 3592de635eea4ef199972ccccfddee4f\n  Virtualization: vmware\nOperating System: Ubuntu 24.04 LTS\n          Kernel: Linux 6.8.0-35-generic\n    Architecture: x86-64\n Hardware Vendor: VMware, Inc.\n  Hardware Model: VMware Virtual Platform\nFirmware Version: 6.00\n   Firmware Date: Thu 2020-11-12\n    Firmware Age: 3y 7month 5d", "stdout_lines": [" Static hostname: ansible-node2", "       Icon name: computer-vm", "         Chassis: vm", "      Machine ID: 208764583c135fadff15af416671270f", "         Boot ID: 3592de635eea4ef199972ccccfddee4f", "  Virtualization: vmware", "Operating System: Ubuntu 24.04 LTS", "          Kernel: Linux 6.8.0-35-generic", "    Architecture: x86-64", " Hardware Vendor: VMware, Inc.", "  Hardware Model: VMware Virtual Platform", "Firmware Version: 6.00", "   Firmware Date: Thu 2020-11-12", "    Firmware Age: 3y 7month 5d"]}
changed: [ansible-node1] => {"changed": true, "cmd": ["hostnamectl"], "delta": "0:00:00.153781", "end": "2024-06-18 01:09:38.086830", "msg": "", "rc": 0, "start": "2024-06-18 01:09:37.933049", "stderr": "", "stderr_lines": [], "stdout": " Static hostname: ansible-node1\n       Icon name: computer-vm\n         Chassis: vm \n      Machine ID: 3e486d594e7bb2048e09d8e066712705\n         Boot ID: 761d97b73559476bb2626e97362e32c9\n  Virtualization: vmware\nOperating System: Ubuntu 24.04 LTS\n          Kernel: Linux 6.8.0-35-generic\n    Architecture: x86-64\n Hardware Vendor: VMware, Inc.\n  Hardware Model: VMware Virtual Platform\nFirmware Version: 6.00\n   Firmware Date: Thu 2020-11-12\n    Firmware Age: 3y 7month 5d", "stdout_lines": [" Static hostname: ansible-node1", "       Icon name: computer-vm", "         Chassis: vm", "      Machine ID: 3e486d594e7bb2048e09d8e066712705", "         Boot ID: 761d97b73559476bb2626e97362e32c9", "  Virtualization: vmware", "Operating System: Ubuntu 24.04 LTS", "          Kernel: Linux 6.8.0-35-generic", "    Architecture: x86-64", " Hardware Vendor: VMware, Inc.", "  Hardware Model: VMware Virtual Platform", "Firmware Version: 6.00", "   Firmware Date: Thu 2020-11-12", "    Firmware Age: 3y 7month 5d"]}

PLAY RECAP ******************************************************************************************************************
ansible-node1              : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
ansible-node2              : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

You can access the documentation for each module from the command line with the ansible-doc tool.

```shell
ansible-doc command
```

```
> ANSIBLE.BUILTIN.COMMAND    (/home/hong/.local/lib/python3.9/site-packages/ansible/modules/command.py)

        The `command' module takes the command name followed by a list of space-delimited arguments.
        The given command will be executed on all selected nodes. The command(s) will not be
        processed through the shell, so variables like `$HOSTNAME' and operations like `"*"', `"<"',
        `">"', `"|"', `";"' and `"&"' will not work. Use the [ansible.builtin.shell] module if you
        need these features. To create `command' tasks that are easier to read than the ones using
        space-delimited arguments, pass parameters using the `args' task keyword
        <https://docs.ansible.com/ansible/latest/reference_appendices/playbooks_keywords.html#task>
        or use `cmd' parameter. Either a free form command or `cmd' parameter is required, see the
        examples. For Windows targets, use the [ansible.windows.win_command] module instead.

ADDED IN: historical
......
```



#### debug – Print statements during execution

https://docs.ansible.com/ansible/latest/collections/ansible/builtin/debug_module.html

- This module prints statements during execution and can be useful for debugging variables or expressions without necessarily halting the playbook.
- Useful for debugging together with the `when:` directive.
- This module is also supported for Windows targets.

`playbook3.yaml`

```yaml
- name: Hello World
  hosts: localhost

  tasks:
    - name: Hello World
      ansible.builtin.debug:
        msg: "Hello World from debug-module"
```
```shell
# Print debug statements
ansible-playbook playbook3.yaml
```
```
[WARNING]: No inventory was parsed, only implicit localhost is available
[WARNING]: provided hosts list is empty, only localhost is available. Note that the implicit localhost does not match 'all'

PLAY [Hello World] *************************************************************

TASK [Gathering Facts] ********************************************************************************
ok: [localhost]

TASK [Hello World] ********************************************************************************
ok: [localhost] => {
    "msg": "Hello World from debug-module"
}

PLAY RECAP ********************************************************************************
localhost                  : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```



---



### Using Variables

https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_variables.html

`playbook4.yaml`

```yaml
- name: Hello World
  hosts: localhost

  vars:
    greetings: "Hello from vars"

  tasks:
    - name: Hello World
      debug:
        msg: "{{ greetings }}"
```

```shell
# Use greetings variable
ansible-playbook playbook4.yaml
```

```
[WARNING]: No inventory was parsed, only implicit localhost is available
[WARNING]: provided hosts list is empty, only localhost is available. Note that the implicit localhost does not match 'all'

PLAY [Hello World] **********************************************************************************************************

TASK [Gathering Facts] ******************************************************************************************************
ok: [localhost]

TASK [Hello World] **********************************************************************************************************
ok: [localhost] => {
    "msg": "Hello from vars"
}

PLAY RECAP ******************************************************************************************************************
localhost                  : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```



#### Defining variables in included files and roles

https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_variables.html#defining-variables-in-included-files-and-roles

```shell
mkdir ~/ansible-demo/playbook/vars
touch ~/ansible-demo/playbook/vars/greeting_vars.yaml
```

`vars/greeting_vars.yaml`

```yaml
---
ext_greetings: "Hello from ext vars"
```

`playbook5.yaml`

```yaml
- name: Hello World
  hosts: localhost

  vars:
    greetings: "Hello from vars"

  vars_files:
    - "vars/greeting_vars.yaml"

  tasks:
    - name: Hello World
      debug:
        msg: "{{ ext_greetings }}"
```

```shell
# Use variables defined in included files
ansible-playbook playbook5.yaml
```

```
[WARNING]: No inventory was parsed, only implicit localhost is available
[WARNING]: provided hosts list is empty, only localhost is available. Note that the implicit localhost does not match 'all'

PLAY [Hello World] **********************************************************************************************************

TASK [Gathering Facts] ******************************************************************************************************
ok: [localhost]

TASK [Hello World] **********************************************************************************************************
ok: [localhost] => {
    "msg": "Hello from ext vars"
}

PLAY RECAP ******************************************************************************************************************
localhost                  : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```



---



### Loops

https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_loops.html

```shell
touch ~/ansible-demo/playbook/vars/esxi_hosts.yaml
```

`vars/esxi_hosts.yaml`

```yaml
---
esxi_hosts:
  - "10.1.0.11"
  - "10.1.0.21"
  - "10.1.0.31"
```

`playbook6.yaml`

```yaml
- name: Hello World
  hosts: localhost
  gather_facts: no

  vars:
    test:
    - test1
    - test2
    - test3

  vars_files:
    - "vars/esxi_hosts.yaml"

  tasks:
    - name: Loop demo
      debug:
        msg:
        - "{{ test }}"
        - "{{ esxi_hosts }}"
```

```shell
# Use loops
ansible-playbook playbook6.yaml
```

```
[WARNING]: No inventory was parsed, only implicit localhost is available
[WARNING]: provided hosts list is empty, only localhost is available. Note that the implicit localhost does not match 'all'

PLAY [Hello World] **********************************************************************************************************

TASK [Loop demo] ************************************************************************************************************
ok: [localhost] => {
    "msg": [
        [
            "test1",
            "test2",
            "test3"
        ],
        [
            "10.1.0.11",
            "10.1.0.21",
            "10.1.0.31"
        ]
    ]
}

PLAY RECAP ******************************************************************************************************************
localhost                  : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```



#### Iterating over a simple list

You can define the list in a variables file, or in the `vars` section of your play, then refer to the name of the list in the task.

```
loop: "{{ somelist }}"
```

`playbook7.yaml`

```yaml
- name: Hello World
  hosts: localhost
  gather_facts: no

  vars:
    test:
    - test1
    - test2
    - test3

  vars_files:
    - "vars/esxi_hosts.yaml"

  tasks:
    - name: Loop demo
      debug:
        msg: "{{ item }}"
      # with_items: "{{ test }}" # Old syntax
      loop: "{{ test }}"
```

```shell
# Iterate over a simple list
ansible-playbook playbook7.yaml
```

```
[WARNING]: No inventory was parsed, only implicit localhost is available
[WARNING]: provided hosts list is empty, only localhost is available. Note that the implicit localhost does not match 'all'

PLAY [Hello World] **********************************************************************************************************

TASK [Loop demo] ************************************************************************************************************
ok: [localhost] => (item=test1) => {
    "msg": "test1"
}
ok: [localhost] => (item=test2) => {
    "msg": "test2"
}
ok: [localhost] => (item=test3) => {
    "msg": "test3"
}

PLAY RECAP ******************************************************************************************************************
localhost                  : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```



#### Registering variables with a loop

You can register the output of a loop as a variable.

When you use `register` with a loop, the data structure placed in the variable will contain a `results` attribute that is a list of all responses from the module. This differs from the data structure returned when using `register` without a loop.

`playbook8.yaml`

```
- name: Hello World
  hosts: localhost
  gather_facts: no

  vars:
    test:
    - test1
    - test2
    - test3

  vars_files:
    - "vars/esxi_hosts.yaml"

  tasks:
    - name: Loop demo
      debug:
        msg: "{{ item }}"
      loop: "{{ test }}"
      register:	test_result

    - name: Registering var with a loop demo
      debug:
        msg: "{{ item }}"
      loop: "{{ test_result.results }}"
```

```shell
# Register the output of a loop as a variable
ansible-playbook playbook8.yaml
```

```
PLAY [Hello World] **********************************************************************************************************

TASK [Loop demo] ************************************************************************************************************
ok: [localhost] => (item=test1) => {
    "msg": "test1"
}
ok: [localhost] => (item=test2) => {
    "msg": "test2"
}
ok: [localhost] => (item=test3) => {
    "msg": "test3"
}

TASK [Registering var with a loop demo] *************************************************************************************
ok: [localhost] => (item={'msg': 'test1', 'failed': False, 'changed': False, 'item': 'test1', 'ansible_loop_var': 'item'}) => {
    "msg": {
        "ansible_loop_var": "item",
        "changed": false,
        "failed": false,
        "item": "test1",
        "msg": "test1"
    }
}
ok: [localhost] => (item={'msg': 'test2', 'failed': False, 'changed': False, 'item': 'test2', 'ansible_loop_var': 'item'}) => {
    "msg": {
        "ansible_loop_var": "item",
        "changed": false,
        "failed": false,
        "item": "test2",
        "msg": "test2"
    }
}
ok: [localhost] => (item={'msg': 'test3', 'failed': False, 'changed': False, 'item': 'test3', 'ansible_loop_var': 'item'}) => {
    "msg": {
        "ansible_loop_var": "item",
        "changed": false,
        "failed": false,
        "item": "test3",
        "msg": "test3"
    }
}

PLAY RECAP ******************************************************************************************************************
localhost                  : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```



### Organizing host and group variables

https://docs.ansible.com/ansible/latest/inventory_guide/intro_inventory.html#organizing-host-and-group-variables

Content of `inventory.ini` file:

```ini
[web_srv]
ansible-node1 ansible_connection=ssh port=443
ansible-node2 ansible_connection=ssh port=443
```

Store variables in the main inventory file: `inventory9.ini`. Extract common vars out:

```ini
[web_srv]
ansible-node1
ansible-node2

[web_srv:vars]
ansible_connection=ssh
port=443
```

`playbook9.yaml`

```yaml
---

- name: Hello World
  hosts: web_srv
  gather_facts: no

  tasks:
    - name: test vars
      debug:
        msg: "ansible_user={{ ansible_connection }} port={{ port }}"
```

```shell
# Extract common vars under [web_srv:vars]
ansible-playbook -i inventory9.ini playbook9.yaml
```

```
PLAY [Hello World] **********************************************************************************************************

TASK [test vars] ************************************************************************************************************
ok: [ansible-node1] => {
    "msg": "ansible_user=ssh port=443"
}
ok: [ansible-node2] => {
    "msg": "ansible_user=ssh port=443"
}

PLAY RECAP ******************************************************************************************************************
ansible-node1              : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
ansible-node2              : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

Node specific vars (port=80 of ansible-node1) overwrite group vars.

`inventory9-2.ini`

```ini
[web_srv]
ansible-node1 port=80
ansible-node2

[web_srv:vars]
ansible_connection=ssh
port=443
```

```shell
# Node specific vars (port=80 of ansible-node1) overwrite group vars.
ansible-playbook -i inventory9-2.ini playbook9.yaml
```

```
PLAY [Hello World] **********************************************************************************************************

TASK [test vars] ************************************************************************************************************
ok: [ansible-node1] => {
    "msg": "ansible_user=ssh port=80"
}
ok: [ansible-node2] => {
    "msg": "ansible_user=ssh port=443"
}

PLAY RECAP ******************************************************************************************************************
ansible-node1              : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
ansible-node2              : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```



Although you can store variables in the main inventory file, storing separate host and group variables files may help you organize your variable values more easily. You can also use lists and hash data in host and group variables files, which you cannot do in your main inventory file.

```shell
mkdir -p ~/ansible-demo/playbook/inventory/group_vars
mkdir -p ~/ansible-demo/playbook/inventory/host_vars
touch ~/ansible-demo/playbook/inventory/host
```

```
tree .
.
├── inventory
│   ├── group_vars
│   │   └── web_srv.yaml
│   ├── host
│   └── host_vars
│       └── ansible-node1.yaml
......
├── playbook9.yaml
```

Content of `~/ansible-demo/playbook/inventory/group_vars/web_srv.yaml`

```yaml
ansible_connection: ssh
port: 443
```

Content of `~/ansible-demo/playbook/inventory/host_vars/ansible-node1.yaml`

```yaml
port: 80
```

Content of `~/ansible-demo/playbook/inventory/host`

```
[web_srv]
ansible-node1
ansible-node2
```

```shell
# Store separate host and group variables files
ansible-playbook -i inventory/host playbook9.yaml
```

```
PLAY [Hello World] **********************************************************************************************************

TASK [test vars] ************************************************************************************************************
ok: [ansible-node1] => {
    "msg": "ansible_user=ssh port=80"
}
ok: [ansible-node2] => {
    "msg": "ansible_user=ssh port=443"
}

PLAY RECAP ******************************************************************************************************************
ansible-node1              : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
ansible-node2              : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```



---



### ansible.cfg

https://docs.ansible.com/ansible/latest/reference_appendices/config.html

Ansible supports several sources for configuring its behavior, including an ini file named `ansible.cfg`, environment variables, command-line options, playbook keywords, and variables. See [Controlling how Ansible behaves: precedence rules](https://docs.ansible.com/ansible/latest/reference_appendices/general_precedence.html#general-precedence-rules) for details on the relative precedence of each source.

The `ansible-config` utility allows users to see all the configuration settings available, their defaults, how to set them and where their current value comes from. See [ansible-config](https://docs.ansible.com/ansible/latest/cli/ansible-config.html#ansible-config) for more information.

#### The configuration file

Changes can be made and used in a configuration file which will be searched for in the following order:

> - `ANSIBLE_CONFIG` (environment variable if set)
> - `ansible.cfg` (in the current directory)
> - `~/.ansible.cfg` (in the home directory)
> - `/etc/ansible/ansible.cfg`

Ansible will process the above list and use the first file found, all others are ignored.

```shell
touch ~/ansible-demo/playbook/ansible.cfg
```

Content of `~/ansible-demo/playbook/ansible.cfg`

```
[defaults]
inventory = inventory/host
host_key_checking = False
```

Now the command line arg `-i inventory/host` can be omitted:

```yaml
# Use configuration file
ansible-playbook playbook9.yaml
```



---



### ansible.builtin.copy module – Copy files to remote locations

https://docs.ansible.com/ansible/latest/collections/ansible/builtin/copy_module.html

The [ansible.builtin.copy](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/copy_module.html#ansible-collections-ansible-builtin-copy-module) module copies a file or a directory structure from the local or remote machine to a location on the remote machine. File system meta-information (permissions, ownership, etc.) may be set, even when the file or directory already exists on the target system. Some meta-information may be copied on request.

```shell
echo "demo" > ~/ansible-demo/playbook/demo.conf
```

`playbook10.yaml`

```yaml
---

- name: ansible.builtin.copy module
  hosts: web_srv
  gather_facts: no

  tasks:
    - name: copy file
      ansible.builtin.copy:
        src: demo.conf
        dest: /home/ubuntu/demo.conf
        owner: ubuntu
        group: ubuntu
        mode: '0644'
```

```shell
# Use ansible.builtin.copy module to copy files to managed nodes
ansible-playbook playbook10.yaml
```

```
PLAY [ansible.builtin.copy module] ******************************************************************************************

TASK [copy file] ************************************************************************************************************
changed: [ansible-node2]
changed: [ansible-node1]

PLAY RECAP ******************************************************************************************************************
ansible-node1              : ok=1    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
ansible-node2              : ok=1    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

```shell
# Verifys the copied files on managed nodes
ssh ansible-node1 cat /home/ubuntu/demo.conf
ssh ansible-node2 cat /home/ubuntu/demo.conf
```

```
demo
demo
```



---



### Understanding privilege escalation: become

https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_privilege_escalation.html

Ansible uses existing privilege escalation systems to execute tasks with root privileges or with another user’s permissions. Because this feature allows you to ‘become’ another user, different from the user that logged into the machine (remote user), we call it `become`. The `become` keyword uses existing privilege escalation tools like sudo, su, pfexec, doas, pbrun, dzdo, ksu, runas, machinectl and others.

##### ansible.builtin.apt module

https://docs.ansible.com/ansible/latest/collections/ansible/builtin/apt_module.html

Manages apt packages (such as for Debian/Ubuntu).

lighttpd is not installed on both servers under web_srv group:

```shell
ssh ansible-node1 systemctl status lighttpd
```

```
Unit lighttpd.service could not be found.
```

```shell
ssh ansible-node2 systemctl status lighttpd
```

```
Unit lighttpd.service could not be found.
```

`playbook11.yaml`

```yaml
---

- name: Install lighttpd package on Ubuntu Linux servers
  hosts: web_srv
  become: true
  become_method: sudo
  gather_facts: no

  tasks:
    - name: Update repositories and install lighttpd package
      ansible.builtin.apt:
        name: lighttpd
        update_cache: true
        state: present
```
```shell
# Install lighttpd package on Ubuntu Linux servers
ansible-playbook playbook11.yaml
```
```
PLAY [Install lighttpd package on Ubuntu Linux servers] *********************************************************************

TASK [Update repositories and install lighttpd package] *********************************************************************
changed: [ansible-node1]
changed: [ansible-node2]

PLAY RECAP ******************************************************************************************************************
ansible-node1              : ok=1    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
ansible-node2              : ok=1    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```
```shell
# Verify that lighttpd has been installed on the managed node
ssh ansible-node1 systemctl status lighttpd
```
```
● lighttpd.service - Lighttpd Daemon
     Loaded: loaded (/usr/lib/systemd/system/lighttpd.service; enabled; preset: enabled)
     Active: active (running) since Tue 2024-06-18 01:49:06 MDT; 1min 1s ago
    Process: 4273 ExecStartPre=/usr/sbin/lighttpd -tt -f /etc/lighttpd/lighttpd.conf (code=exited, status=0/SUCCESS)
   Main PID: 4278 (lighttpd)
      Tasks: 1 (limit: 4613)
     Memory: 928.0K (peak: 3.1M)
        CPU: 320ms
     CGroup: /system.slice/lighttpd.service
             └─4278 /usr/sbin/lighttpd -D -f /etc/lighttpd/lighttpd.conf

Jun 18 01:49:06 ansible-node1 systemd[1]: Starting lighttpd.service - Lighttpd Daemon...
Jun 18 01:49:06 ansible-node1 systemd[1]: Started lighttpd.service - Lighttpd Daemon.
```

```shell
# Visit the web servers to see the placeholder page
curl http://ansible-node1
```

```
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
...
```



Create a local `index.html`:

```shell
echo "Peter Parker was here" > index.html
```

`playbook12.yaml`

```shell
---

- name: Install lighttpd package on Ubuntu Linux servers
  hosts: web_srv
  become: true
  become_method: sudo
  gather_facts: no

  tasks:
    - name: Update repositories and install lighttpd package
      ansible.builtin.apt:
        name: lighttpd
        update_cache: true
        state: present

    - name: Copy local index.html file to web servers
      ansible.builtin.copy:
        src: index.html
        dest: /var/www/html/index.html
        owner: ubuntu
        group: ubuntu
        mode: '0644'
```

```shell
# Install lighttpd package and copy index.html file to Ubuntu Linux servers
ansible-playbook playbook12.yaml
```

```
PLAY [Install lighttpd package on Ubuntu Linux servers] *********************************************************************

TASK [Update repositories and install lighttpd package] *********************************************************************
ok: [ansible-node2]
ok: [ansible-node1]

TASK [Copy local index.html file to web servers] ****************************************************************************
changed: [ansible-node2]
changed: [ansible-node1]

PLAY RECAP ******************************************************************************************************************
ansible-node1              : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
ansible-node2              : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

```shell
# Visit the web servers to see the customized page
curl http://ansible-node1
```

```
Peter Parker was here
```











