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

- `ansible_connection` - Тип подключения. Это может быть названием плагина, который Ansible будет использовать для подключения. Протокол SSH может быть `ssh` или `paramiko`. По умолчанию ssh.

- `ansible_host` - ip-адрес или имя хоста для подключения, если отличается от псевдонима.

- `ansible_port` - порт для подключения, если отличается от значения по умолчанию (22 для ssh)

- `ansible_user` - имя пользователя для подключения

- `ansible_password` - пароль для подключения (не храните в виде текста; используйте vault)

- `ansible_ssh_private_key_file` - файл с ключом для ssh

- `ansible_python_interpreter` - путь до интерпритатора python

## Reference

- https://docs.ansible.com/ansible/latest/inventory_guide/intro_inventory.html
