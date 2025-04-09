## LAB: Targeting Specific Nodes

В этой лабораторной работе:
- как указать определенные хосты в плейбуке 

### Подготовка

- Проверьте выполнены ли у вас данные лабораторные работы:
  - [LAB: Multipass-SSH Configuration (Create Ansible Test Environment)](https://github.com/gulyaeve/ansible_labs/blob/main/Multipass-SSH-Configuration.md)
  - [LAB: VirtualBOX-SSH Configuration (Create Ansible Test Environment)](https://github.com/gulyaeve/ansible_labs/blob/main/VirtualBOX-SSH-Configuration.md)

### Шаги

- Обновите файл инвентаризации.
- Создайте группы, указав имена в '[]'.

``` 
[web_servers]
172.21.79.85

[database_servers]
172.21.76.101
``` 

![image](https://user-images.githubusercontent.com/10358317/201671961-6eb2815e-67e1-43d5-9e36-49bcebe0dad4.png)


- Создайте файл 'site.yml' 
- Задачи можно упорядочить при помощи 'pre_tasks'
- Для разных хостов, теперь используются 3 группы: all, web_servers и database_servers
```
---

- hosts: all
  become: true
  pre_tasks:

  - name: install updates (CentOS)
    dnf:
      update_only: yes
      update_cache: yes
    when: ansible_distribution == "CentOS"

  - name: install updates (Ubuntu)
    apt:
      upgrade: dist
      update_cache: yes
    when: ansible_distribution == "Ubuntu"

- hosts: web_servers
  become: true
  tasks:

  - name: install apache and php (CentOS)
    dnf:
      name:
        - httpd
        - php
      state: latest
    when: ansible_distribution == "CentOS"

  - name: install apache and php (Ubuntu)
    apt:
      name:
        - apache2
        - libapache2-mod-php
      state: latest
    when: ansible_distribution == "Ubuntu"
    
- hosts: database_servers
  become: true
  tasks:

  - name: install MariaDB (CentOS)
    dnf:
      name: mariadb
      state: latest
    when: ansible_distribution == "CentOS"

  - name: install MariaDB (Ubuntu)
    apt:
      name: mariadb-server
      state: latest
    when: ansible_distribution == "Ubuntu"
```    

- Запустите 

```
ansible-playbook --ask-become-pass site.yml
```

![image](https://user-images.githubusercontent.com/10358317/201669957-5141bd02-b42a-4ceb-8750-d2aee3fb9716.png)

- Проверьте что база данных mariadb установлена или нет на группе database_servers (node2).

```
multipass shell node2
systemctl status mariadb
```

![image](https://user-images.githubusercontent.com/10358317/201671418-42d8734a-d55f-4247-8451-db45e7a889a4.png)

- Изменение параметров подключения


```
node1 ansible_port=5555 ansible_host=192.0.2.50
node2 ansible_port=5555 ansible_host=192.0.2.50
```

## Параметры подключения

- ansible_connection
Тип подключения. Это может быть названием плагина, который Ansible будет использовать для подключения. Протокол SSH может быть `ssh` или `paramiko`. По умолчанию ssh.

- ansible_host
A resolvable name or IP of the host to connect to, if different from the alias you wish to give to it. Never set it to depend on inventory_hostname, if you really need something like that use inventory_hostname_short, so it can work with delegation.

- ansible_port

    The connection port number, if not the default (22 for ssh)
ansible_user

    The username to use when connecting (logging in) to the host
ansible_password

    The password to use to authenticate to the host (never store this variable in plain text; always use a vault. See Keep vaulted variables safely visible)

Specific to the SSH connection plugin:

ansible_ssh_private_key_file

    Private key file used by SSH. Useful if using multiple keys and you do not want to use SSH agent.
ansible_ssh_common_args

    This setting is always appended to the default command line for sftp, scp, and ssh. Useful to configure a ProxyCommand for a certain host (or group).
ansible_sftp_extra_args

    This setting is always appended to the default sftp command line.
ansible_scp_extra_args

    This setting is always appended to the default scp command line.
ansible_ssh_extra_args

    This setting is always appended to the default ssh command line.
ansible_ssh_pipelining

    Determines whether or not to use SSH pipelining. This can override the pipelining setting in ansible.cfg.
ansible_ssh_executable (added in version 2.2)

    This setting overrides the default behavior to use the system ssh. This can override the ssh_executable setting in ansible.cfg under ssh_connection.

Privilege escalation (see Ansible Privilege Escalation for further details):

ansible_become

    Equivalent to ansible_sudo or ansible_su, allows to force privilege escalation
ansible_become_method

    Allows to set privilege escalation method to a matching become plugin
ansible_become_user

    Equivalent to ansible_sudo_user or ansible_su_user, allows you to set the user you become through privilege escalation
ansible_become_password

    Equivalent to ansible_sudo_password or ansible_su_password, allows you to set the privilege escalation password (never store this variable in plain text; always use a vault. See Keep vaulted variables safely visible)
ansible_become_exe

    Equivalent to ansible_sudo_exe or ansible_su_exe, allows you to set the executable for the escalation method selected
ansible_become_flags

    Equivalent to ansible_sudo_flags or ansible_su_flags, allows you to set the flags passed to the selected escalation method. This can be also set globally in ansible.cfg in the become_flags option under privilege_escalation.

Remote host environment parameters:

ansible_shell_type

    The shell type of the target system. You should not use this setting unless you have set the ansible_shell_executable to a non-Bourne (sh) compatible shell. By default, commands are formatted using sh-style syntax. Setting this to csh or fish will cause commands executed on target systems to follow those shell’s syntax instead.

ansible_python_interpreter

    The target host Python path. This is useful for systems with more than one Python or not located at /usr/bin/python such as *BSD, or where /usr/bin/python is not a 2.X series Python. We do not use the /usr/bin/env mechanism as that requires the remote user’s path to be set right and also assumes the python executable is named python, where the executable might be named something like python2.6.
ansible_*_interpreter

    Works for anything such as ruby or perl and works just like ansible_python_interpreter. This replaces shebang of modules that will run on that host.

New in version 2.1.

ansible_shell_executable

    This sets the shell the ansible control node will use on the target machine, overrides executable in ansible.cfg which defaults to /bin/sh. You should only change this value if it is not possible to use /bin/sh (in other words, if /bin/sh is not installed on the target machine or cannot be run from sudo.).

