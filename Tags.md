## LAB: Adding Tags

Tags are useful when we want to run specific part of the playbook. If we tagged with specific keywords, only tagged part would run.  

В этой лабораторной работе:
- как добавить теги и запускать определенные задачи из плейбука 

### Подготовка

- Проверьте выполнены ли у вас данные лабораторные работы:
  - [LAB: Multipass-SSH Configuration (Create Ansible Test Environment)](https://github.com/gulyaeve/ansible_labs/blob/main/Multipass-SSH-Configuration.md)

### Шаги

- Теги можно добавить при помощи ключевого слова "tags". Тег "Always" запускается всегда, другие теги необходимо задать по имени (apache, mariadb, итд.)

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

- Чтобы увидеть все теги в плейбуке, запустите:

```
ansible-playbook --list-tags site.yml
```

![image](https://user-images.githubusercontent.com/10358317/201675534-6773039e-b45e-4f1f-b473-3a1ab1059f69.png)

- Когда необходимо запустить только задачи, содержащие тег 'Ubuntu', запустите:

```
ansible-playbook --tags ubuntu --ask-become-pass site.yml
```

- Как можно увидеть ниже, были выполнены задачи с тегами 'ubuntu' и 'always'. "Gathering Facts" - Сбор данных запускается всегда для сбора данных с хоста.

![image](https://user-images.githubusercontent.com/10358317/201676220-0cb5dfc4-3a29-4de0-b9eb-96a3fcc275dd.png)

- Когда необходимо запустить только задачи, содержащие тег 'db', запустите:

```
ansible-playbook --tags db --ask-become-pass site.yml
```

- Как можно увидеть ниже, были выполнены задачи с тегами 'db' и 'always'.

![image](https://user-images.githubusercontent.com/10358317/201676636-7043e4e5-2277-4273-8274-e934e5ad1bb8.png)

- Когда необходимо запустить только задачи, с тегом 'always', запустите:

```
ansible-playbook --tags always --ask-become-pass site.yml
```

![image](https://user-images.githubusercontent.com/10358317/201677188-2df9f569-89ea-4691-952a-db812e47ffe3.png)
