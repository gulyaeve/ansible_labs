## LAB: Adding User

В этой лабораторной работе:
- добавление пользователя.

### Подготовка

- Проверьте выполнены ли у вас данные лабораторные работы:
  - [LAB: Multipass-SSH Configuration (Create Ansible Test Environment)](https://github.com/gulyaeve/ansible_labs/blob/main/Multipass-SSH-Configuration.md)
  - [LAB: VirtualBOX-SSH Configuration (Create Ansible Test Environment)](https://github.com/gulyaeve/ansible_labs/blob/main/VirtualBOX-SSH-Configuration.md)

### Шаги

- Обновите файл site.yml, сделаем добавление пользователя 'newuser111':

```
 - name: create new user
    tags: always
    user:
      name: newuser111
      groups: root
```

- Обновите site.yml

```
---

- hosts: all
  become: true
  pre_tasks:

  - name: install updates (CentOS)
    tags: always
    dnf:
      update_only: yes
      update_cache: yes
    when: ansible_distribution == "CentOS"

  - name: install updates (Ubuntu)
    tags: always
    apt:
      upgrade: dist
      update_cache: yes
    when: ansible_distribution == "Ubuntu"

  - name: create new user
    tags: always
    user:
      name: newuser111
      groups: root

- hosts: web_servers
  become: true
  tasks:

  - name: install apache and php (CentOS)
    tags: centos,apache,httpd
    dnf:
      name:
        - httpd
        - php
      state: latest
    when: ansible_distribution == "CentOS"

  - name: install apache and php (Ubuntu)
    tags: ubuntu,apache,apache2
    apt:
      name:
        - apache2
        - libapache2-mod-php
      state: latest
    when: ansible_distribution == "Ubuntu"

  - name: start apache (Ubuntu)
    tags: ubuntu,apache,apache2
    service:
      name: apache2
      state: started
      enabled: yes
    when: ansible_distribution == "Ubuntu"

  - name: change email address for admin (Ubuntu)
    tags: ubuntu,apache,apache2
    lineinfile:
      path: /etc/apache2/sites-available/000-default.conf
      regexp: 'ServerAdmin webmaster@localhost'
      line: '       ServerAdmin somebody@somewhere.com'
    when: ansible_distribution == "Ubuntu"
    register: apache2_service

  - name: restart apache (Ubuntu)
    tags: ubuntu,apache,apache2
    service:
      name: apache2
      state: restarted
    when: apache2_service.changed

  - name: copy default (index) html file for site
    tags: apache,apache2,httpd
    copy:
      src: default_site.html
      dest: /var/www/html/index.html
      owner: root
      group: root
      mode: 0644

  - name: install unzip
    package:
      name: unzip

  - name: install terraform
    unarchive:
      src: https://releases.hashicorp.com/terraform/1.3.4/terraform_1.3.4_linux_amd64.zip
      dest: /usr/local/bin
      remote_src: yes
      owner: root
      group: root
      mode: 0755

- hosts: database_servers
  become: true
  tasks:

  - name: install MariaDB (CentOS)
    tags: centos,db,mariadb
    dnf:
      name: mariadb
      state: latest
    when: ansible_distribution == "CentOS"

  - name: install MariaDB (Ubuntu)
    tags: ubuntu,db,mariadb-server
    apt:
      name: mariadb-server
      state: latest
    when: ansible_distribution == "Ubuntu"
```

- Запустите плейбук:

```
ansible-playbook --ask-become-pass site.yml
```

![image](https://user-images.githubusercontent.com/10358317/201946359-c22d6265-12ef-4bbd-8e19-956bff8a70a9.png)

- Проверьте на хостах:

```
cat /etc/passwd
```

![image](https://user-images.githubusercontent.com/10358317/201946789-0a860635-910f-4239-8131-da7cea7af6b1.png)

- Добавьте SSH-ключ и создайте файл для sudo в плейбуке 'site.yml':

```
  - name: add ssh key for new user
    tags: always
    authorized_key:
      user: newuser111
      # Содержимое публичного ключа необходимо вставить своё:
      key: "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQCdEJ8xu1C0kJ4Y39x2bfptMQVdxnPDXrkjmDvB44oDV78yKWg/0B/kacMqiEaaiEyAedH>"

  - name: add sudoers file for newuser111
    tags: always
    copy:
      src: sudoer_newuser111
      dest: /etc/sudoers.d/newuser111
      owner: root
      group: root
      mode: 0440
```

- Создайте файл sudoer_newuser111 в директории files:

```
nano files/sudoer_newuser111
# copy following into it
newuser111 ALL=(ALL) NOPASSWD: ALL
```

![image](https://user-images.githubusercontent.com/10358317/201950100-3a9b10e3-3342-4dbe-b005-dab25be2a534.png)

```
ansible-playbook --ask-become-pass site.yml
```

![image](https://user-images.githubusercontent.com/10358317/201950757-5d496494-8d74-4513-80b4-2fa0f5a546ff.png)

- Проверьте что файл newuser111 доставлен на хост node1

```
sudo ls -l /etc/sudoers.d
```

![image](https://user-images.githubusercontent.com/10358317/201951036-f09de25b-bdf2-49fc-b54e-68f223b0e287.png)

![image](https://user-images.githubusercontent.com/10358317/201951673-28afbf0d-8992-4d53-8860-f23a7e2e694c.png)

![image](https://user-images.githubusercontent.com/10358317/201952021-c57d91fd-dddc-4c14-bf7b-2e8557bd9125.png)


- Обновите файл 'ansible.cfg'

```
[defaults]
inventory = inventory
# private_key_file = ~/.ssh/ansible
remote_user = newuser111
```
- Когда теперь мы запускаем плейбук без become (sudo), это должно работать:

```
ansible-playbook site.yml
```

![image](https://user-images.githubusercontent.com/10358317/201961697-79fee25d-f1f0-4894-81ce-8dfe30c6ff67.png)

- В итоге мы создали пользователя, добавили этого пользователя к sudoers и можем использовать плейбук без become password
