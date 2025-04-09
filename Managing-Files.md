## LAB: Managing Files

В этой лабораторной работе:
- передача файла с управляющего хоста на удаленные.
- скачивание и распаковка архива.

### Подготовка

- Проверьте выполнены ли у вас данные лабораторные работы:
  - [LAB: Multipass-SSH Configuration (Create Ansible Test Environment)](https://github.com/gulyaeve/ansible_labs/blob/main/Multipass-SSH-Configuration.md)
  - [LAB: VirtualBOX-SSH Configuration (Create Ansible Test Environment)](https://github.com/gulyaeve/ansible_labs/blob/main/VirtualBOX-SSH-Configuration.md)
  - [LAB: Adding Tags](https://github.com/gulyaeve/ansible_labs/blob/main/Tags.md)

### Шаги

#### Передача файлов с управляющего хоста на клиенты

- В своей директории ansible на управляющем хосте создайте файл html.
- Создайте директорию и веб-страницу.

``` 
mkdir files
nano files/default_site.html
``` 

``` 
<html>
   <title>Ansible File Transfer Test</title>

   <body>
      <p>Ansible Great!</p>
   </body>
</html>
``` 

- Измените файл 'site.yml' (сделанный в работе: [LAB: Adding Tags](https://github.com/gulyaeve/ansible_labs/blob/main/Tags.md))

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
    
  - name: copy default (index) html file for site
    tags: apache,apache2,httpd
    copy:
      src: default_site.html
      dest: /var/www/html/index.html
      owner: root
      group: root
      mode: 0644    
    
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

![image](https://user-images.githubusercontent.com/10358317/201684361-7894105d-09b2-46ac-920b-b8702ce8f234.png)

- Запустите: 

```
ansible-playbook --ask-become-pass site.yml
``` 

![image](https://user-images.githubusercontent.com/10358317/201685754-26180f5c-04ee-4d8b-a0ee-38edde586312.png)

- Файл передан на node1: 

![image](https://user-images.githubusercontent.com/10358317/201686161-0ace79c4-0b99-40d2-8494-e5c7fb061df6.png)

![image](https://user-images.githubusercontent.com/10358317/201686403-b4909095-c1ae-4a52-90cf-9a6e2fdf09ab.png)

#### Скачивание и распаковка архива из интернета

- Измените файл 'site.yml' добавив установку пакета unzip (install unzip) и распаковку (unarchive) файла (install terraform).

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
    # ссылка может быть не доступна
    unarchive:
      src: https://mirror.selectel.ru/3rd-party/hashicorp-releases/terraform/1.3.4/terraform_1.3.4_linux_amd64.zip
      dest: /usr/local/bin
      validate_certs: false
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

![image](https://user-images.githubusercontent.com/10358317/201689373-a244449e-ca81-48d9-9947-dffaf6802e81.png)

- Это действие распакует содержимое архива с terraform на удаленном хосте в '/usr/local/bin'

```
ansible-playbook --ask-become-pass site.yml
``` 

![image](https://user-images.githubusercontent.com/10358317/201689863-cffbe7ac-2501-454d-9509-011886629d51.png)

- Установлено на node1

![image](https://user-images.githubusercontent.com/10358317/201690101-8620f86c-d27a-4f51-8f40-1be649efcf54.png)








