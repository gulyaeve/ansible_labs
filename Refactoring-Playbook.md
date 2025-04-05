## LAB: Refactoring / Improving Playbook

В этой лабораторной работе:
- как рефакторить и улучшать плейбуки

### Подготовка

- Проверьте выполнены ли у вас данные лабораторные работы:
  - [LAB: Multipass-SSH Configuration (Create Ansible Test Environment)](https://github.com/gulyaeve/ansible_labs/blob/main/Multipass-SSH-Configuration.md)
  - [LAB: Implement First Playbook](https://github.com/gulyaeve/ansible_labs/blob/main/Implement-First-Playbook.md)

### Шаги

- У нас есть плейбук, созданный в этой лабораторной работе: https://github.com/gulyaeve/ansible_labs/blob/main/Implement-First-Playbook.md

```
---

- hosts: all
  become: true
  tasks:

  - name: update repository index
    apt:
      update_cache: yes
    when: ansible_distribution in ["Debian", "Ubuntu"]

  - name: install apache2 package
    apt:
      name: apache2
      state: latest
    when: ansible_distribution == "Ubuntu"

  - name: add php support for apache
    apt:
      name: libapache2-mod-php
      state: latest
    when: ansible_distribution == "Ubuntu"

  - name: update repository index
    dnf:
      update_cache: yes
    when: ansible_distribution == "CentOS"

  - name: install apache2 package
    dnf:
      name: httpd
      state: latest
    when: ansible_distribution == "CentOS"

  - name: add php support for apache
    dnf:
      name: php
      state: latest
    when: ansible_distribution == "CentOS"
```

#### Консолидация задач у похожих модулей в одну задачу

- Консолидируем модули 'apt' и 'dnf' в одну группу, общую для Ubuntu и CentOS.
- Для обеих систем будем делать установку пакетов и обновление списка пакетов.
```
---

- hosts: all
  become: true
  tasks:

  - name: update repository index, install apache2 and php package for Ubuntu
    apt:
      name:
        - apache2
        - libapache2-mod-php
      state: latest
      update_cache: yes
    when: ansible_distribution == "Ubuntu"

  - name: update repository index, install apache2 and php package for CentOS
    dnf:
      name:
        - httpd
        - php
      state: latest
      update_cache: yes
    when: ansible_distribution == "CentOS"
```

#### Консолидация в одну задачу при помощи переменных

- Обновите файл install_apache.yml и добавьте переменные: apache_package, php_package
- Измените 'apt' и 'dnf' на 'package'

```
---

- hosts: all
  become: true
  tasks:

  - name: update repository index, install apache and php packages
    package:
      name:
        - "{{ apache_package }}"
        - "{{ php_package }}"
      state: latest
      update_cache: yes
```

- Обновите inventory, добавив туда переменные

```
172.21.79.85  apache_package=apache2 php_package=libapache2-mod-php
172.21.76.101 apache_package=apache2 php_package=libapache2-mod-php
172.21.76.102 apache_package=httpd php_package=php
```

- Запустите:

```
ansible-playbook --ask-become-pass install_apache.yml
```

![image](https://user-images.githubusercontent.com/10358317/201663202-00e9288e-9c9f-4c2b-95f8-d7146a590da7.png)
