## LAB: Managing Services

В этой лабораторной работе:
- управление службами (сервисами) 

### Подготовка

- Проверьте выполнены ли у вас данные лабораторные работы:
  - [LAB: Multipass-SSH Configuration (Create Ansible Test Environment)](https://github.com/gulyaeve/ansible_labs/blob/main/Multipass-SSH-Configuration.md)
  - [LAB: VirtualBOX-SSH Configuration (Create Ansible Test Environment)](https://github.com/gulyaeve/ansible_labs/blob/main/VirtualBOX-SSH-Configuration.md)

### Шаги

#### Запуск службы

- Измените файл site.yml, добавив:
```
  - name: start apache (Ubuntu)
    tags: ubuntu,apache,apache2
    service:
      name: apache2
      state: started
      enabled: yes
    when: ansible_distribution == "Ubuntu"
```
- Измените site.yml 

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
    
  - name: start apache (Ubuntu)
    tags: ubuntu,apache,apache2
    service:
      name: apache2
      state: started
      enabled: yes
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
    unarchive:
      src: https://mirror.selectel.ru/3rd-party/hashicorp-releases/terraform/1.3.4/terraform_1.3.4_linux_amd64.zip
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
- Остановите службу apache2 и проверьте статус.

``` 
systemctl stop apache2
systemctl status apache2
``` 

![image](https://user-images.githubusercontent.com/10358317/201748361-1423d4e9-f880-44f6-9f3c-c7c3c0afe712.png)

- Запустите 

``` 
ansible-playbook --ask-become-pass site.yml
``` 

- Убедитесь что служба apache2 запущена: 

![image](https://user-images.githubusercontent.com/10358317/201749062-dcf7cd41-4133-4cc9-8f6b-a3e0b35a4cda.png)

#### Change Service Config File

- Перейдите на node1 и запустите:

```
sudo nano /etc/apache2/sites-available/000-default.conf
```

![image](https://user-images.githubusercontent.com/10358317/201751273-ed32c787-f360-48a2-aa66-2fedfd4e9391.png)

- Добавьте следующую задачу на веб-сервере в файл site.yml.

```
  - name: change email address for admin (Ubuntu)
    tags: ubuntu,apache,apache2
    lineinfile:
      path: /etc/apache2/sites-available/000-default.conf
      regexp: '^ServerAdmin'
      line: ServerAdmin somebody@somewhere.com
    when: ansible_distribution == "Ubuntu"
    register: apache2_service

  - name: restart apache (Ubuntu)
    tags: ubuntu,apache,apache2
    service:
      name: apache2
      state: restarted
    when: apache2_service.changed
```

- Как видно на скриншоте, когда ключевое слово записывается в регистр (apache2_service), происходит его изменение, в этом случае запускается задача на перезапуск службы. Задача становится зависимой от переменной регистра.

![image](https://user-images.githubusercontent.com/10358317/201753745-29e9e879-770a-46e5-a315-1da40f0cd1fa.png)

- **ВАЖНО:** Если переменная в регистре определена последовательно, то если во второй задаче она не изменит значение (потому что первая изменила), то задача на перезапуск выполнена не будет.   

![image](https://user-images.githubusercontent.com/10358317/201755326-44284c95-db0d-46c3-8f22-f224d845bd5b.png)

- Измените site.yml:

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
      src: https://mirror.selectel.ru/3rd-party/hashicorp-releases/terraform/1.3.4/terraform_1.3.4_linux_amd64.zip
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

- Как видите файл изменился:

![image](https://user-images.githubusercontent.com/10358317/201753411-ed5e38cd-1877-4261-b63f-72e413cd4a7c.png)

## Reference

- https://www.youtube.com/watch?v=soeBHGAMkoQ&list=PLT98CRl2KxKEUHie1m24-wkyHpEsa4Y70&index=12

### Задания для самостоятельного выполнения

- создайте файл для изменения порта ssh
- сделайте плейбук, который копирует данный файл на хосты (/etc/ssh/sshd_config.d/*.conf)
- добавьте перезапуск сервиса при измении порта
- проверьте чтобы порт был в файле инвентаризации

