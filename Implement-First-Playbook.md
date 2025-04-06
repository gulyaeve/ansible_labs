## LAB: Implement First Playbook

В этой лабораторной работе:
- как создавать плейбуки в ansible
- запуск команды "apt install" в плейбуке 
- запуск команды "update" в плейбуке 
- запуск команды "apt remove" в плейбуке 
- запуск команды в зависимости от дистрибутива на хосте (when)

### Подготовка

- Проверьте выполнены ли у вас данные лабораторные работы:
  - [LAB: Multipass-SSH Configuration (Create Ansible Test Environment)](https://github.com/gulyaeve/ansible_labs/blob/main/Multipass-SSH-Configuration.md)
  - [LAB: VirtualBOX-SSH Configuration (Create Ansible Test Environment)](https://github.com/gulyaeve/ansible_labs/blob/main/VirtualBOX-SSH-Configuration.md)

### Шаги

- Создайте плейбук для устрановка веб-сервера apache2

#### Команда "Apt Install" в плейбуке

``` 
nano install_apache.yml
# copy followings:
---

- hosts: all
  become: true
  tasks:

  - name: install apache2 package
    apt:
      name: apache2
``` 

![image](https://user-images.githubusercontent.com/10358317/201101204-780321bf-57b7-450b-89e9-1c5d6deb5e39.png)

![image](https://user-images.githubusercontent.com/10358317/201100926-480de972-078e-4541-9f06-30b625c71585.png)

- Запустите плейбук 

``` 
ansible-playbook --ask-become-pass install_apache.yml
``` 

- Для задачи "install apache2 package" изменился статус, это значит что пакет apache2 был установлен успешно 

![image](https://user-images.githubusercontent.com/10358317/201101843-efc4262d-5506-404e-b505-0d91131154df.png)

- Если запустить повторно, то статус будет "ok" у задачи "install apache2 package", это значит что установка повторно не выполнялась

![image](https://user-images.githubusercontent.com/10358317/201102379-69fe5e2b-7793-421d-add0-25d88b19b969.png)

- Если ввести адрес хоста в браузере, вы сможете увидеть, что веб-сервер apache2 установлен

![image](https://user-images.githubusercontent.com/10358317/201103096-a62b7d08-1208-485f-8bd4-de5b5c7b1e06.png)

- Если ввести имя несуществующего пакета, вы увидите что его невозможно установить.

![image](https://user-images.githubusercontent.com/10358317/201104092-7a38235c-1a48-4f16-8c2e-9269be6d7faa.png)

![image](https://user-images.githubusercontent.com/10358317/201103961-7a10a711-d6e4-4aac-b05c-f5d5172f25ad.png)

#### Команда "Update" в плеубуке 

- Добавьте "apt update" и установите другие пакеты

``` 
---

- hosts: all
  become: true
  tasks:

  - name: update repository index
    apt:
      update_cache: yes

  - name: install apache2 package
    apt:
      name: apache2

  - name: add php support for apache
    apt:
      name: libapache2-mod-php
``` 

![image](https://user-images.githubusercontent.com/10358317/201105158-4b3e598b-0726-444f-8844-ee99fc8f8d82.png)

``` 
ansible-playbook --ask-become-pass install_apache.yml
``` 

![image](https://user-images.githubusercontent.com/10358317/201105473-2697a57a-4334-484f-97d9-501452007259.png)

- Добавьте "state: latest" чтобы установить последнюю версию пакета

``` 
---

- hosts: all
  become: true
  tasks:

  - name: update repository index
    apt:
      update_cache: yes

  - name: install apache2 package
    apt:
      name: apache2
      state: latest

  - name: add php support for apache
    apt:
      name: libapache2-mod-php
      state: latest
```

![image](https://user-images.githubusercontent.com/10358317/201106278-537221bf-878a-4ee3-b211-83e9e52e252f.png)

#### Команда "Apt remove" в плейбуке

- Создайте "remove_apache.yml" для удаления пакетов.
- Используйте "state:absent" для удаления.

``` 
---

- hosts: all
  become: true
  tasks:

  - name: remove apache2 package
    apt:
      name: apache2
      state: absent

  - name: add php support for apache
    apt:
      name: libapache2-mod-php
      state: absent
```

![image](https://user-images.githubusercontent.com/10358317/201107052-4e2c2d50-d7e7-44dd-8352-bd91f7e7be6b.png)

``` 
ansible-playbook --ask-become-pass remove_apache.yml
``` 

![image](https://user-images.githubusercontent.com/10358317/201107516-ff2b2337-a01c-401d-af0a-177ac38c58c7.png)

- При попытке открыть адрес в браузере вы увидите что веб-сервер apache2 удален.

![image](https://user-images.githubusercontent.com/10358317/201107771-df10bf6e-367c-4235-a779-2703958b8774.png)

#### Команды для определенных дистрибутивов ("when")

- Добавьте 'when: ansible-distribution == "Ubuntu"' в файл install_apache.yaml
- Используя 'when', появляется возможность выполнять отдельные команды для разных систем (например, Ubuntu, Centos)

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
```
- Чтобы получить информацию о хосте. 

```
ansible all -m gather_facts --limit 172.21.79.85 | grep ansible_distribution
```

![image](https://user-images.githubusercontent.com/10358317/201655623-98733d68-3624-48a1-8589-d6ec62bbf7aa.png)

- Команду 'when' можно использовать и с другими данными (например, "ansible_distribution_version": "22.04")

- Добавьте новую задачу для 'CentOS'

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
Запустите: 
``` 
ansible-playbook --ask-become-pass install_apache.yml
``` 
- Задача для 'CentOS' была пропущена

![image](https://user-images.githubusercontent.com/10358317/201657035-511cf7aa-8b17-4f87-95f9-4ea8d2772b1d.png)


### Reference

- https://www.youtube.com/watch?v=VANub3AhZpI&list=PLT98CRl2KxKEUHie1m24-wkyHpEsa4Y70&index=6
