# Ansible Labs
Данный репозиторий содержит лабораторные работы по Ansible (используется Multipass: легковесные ВМ Ubuntu): Ad-Hoc команды, модули, плейбуки, тэги (Tags), управление файлами и серверами, пользователи, роли, хенлеры, переменные хостов, шаблоны и многое другое.

**Keywords:** Ansible, Multi-PC Configuration.

# Список лабораторных работ 
- [LAB: Multipass-SSH Configuration (Create Ansible Test Environment)](https://github.com/gulyaeve/ansible_labs/blob/main/Multipass-SSH-Configuration.md)
- [LAB: Install Ansible and Test Basic Ansible (Ad-Hoc) Commands](https://github.com/gulyaeve/ansible_labs/blob/main/Install-Ansible-Basic-Commands.md)
- [LAB: Implement First Playbook](https://github.com/gulyaeve/ansible_labs/blob/main/Implement-First-Playbook.md)
- [LAB: Playing Docker Module](https://github.com/gulyaeve/ansible_labs/blob/main/Docker-Module.md)
- [LAB: Important (Mostly Possible Used) Modules Sample Tasks](https://github.com/gulyaeve/ansible_labs/blob/main/Important-Modules-Sample-Tasks.md)
- [LAB: Refactoring / Improving Playbook](https://github.com/gulyaeve/ansible_labs/blob/main/Refactoring-Playbook.md)
- [LAB: Targeting Specific Nodes (Grouping)](https://github.com/gulyaeve/ansible_labs/blob/main/Targeting-Specific-Node.md)
- [LAB: Adding Tags](https://github.com/gulyaeve/ansible_labs/blob/main/Tags.md)
- [LAB: Managing Files](https://github.com/gulyaeve/ansible_labs/blob/main/Managing-Files.md)
- [LAB: Managing Services](https://github.com/gulyaeve/ansible_labs/blob/main/Managing-Services.md)
- [LAB: Adding Users](https://github.com/gulyaeve/ansible_labs/blob/main/Adding-User.md)
- [LAB: Roles](https://github.com/gulyaeve/ansible_labs/blob/main/Roles.md)
- [LAB: Host Variables](https://github.com/gulyaeve/ansible_labs/blob/main/Host-Variables.md)
- [LAB: Handlers](https://github.com/gulyaeve/ansible_labs/blob/main/Handlers.md)
- [LAB: Templates](https://github.com/gulyaeve/ansible_labs/blob/main/Templates.md)
- [LAB: Practice](https://github.com/gulyaeve/ansible_labs/blob/main/Practice.md)

# Содержание
- [Введение](#motivation)
- [Что такое Ansible?](#what_is_ansible)
- [Как работает Ansible?](#how_ansible_works)
- [Подготовка лабораторного стенда](#lab_environment)
- [Базовые команды Ansible (Ad-Hoc Commands)](#commands)    
- [Модули Ansible](#modules)
- [Плейбуки Ansible](#playbooks)
- [Файл инвентаризации - указание определенных хостов](#inventory)
- [Теги](#tags)
- [Управление файлами](#files)
- [Управление сервисами](#services)
- [Добавление пользователей](#users)
- [Роли](#roles)
- [Переменные хостов](#host_variables)
- [Хэндлеры](#handlers)
- [Шаблоны](#templates)
- [Дебаггинг](#debugging)
- [Дополнительные возможности](#details)
- [Задания для самостоятельного выполнения](#practice)
- [Другие полезные ресурсы по Ansible](#resource)
- [Ссылки](#references)

## Введение <a name="motivation"></a>

Для чего используется Ansible?

- Ansible позволяет автоматизировать задачи и команды на множество машин (серверы, ПК).
- Ansible это очень продвинутый инструмент для автоматизаций, используемый многими компаниями.
- Ansible можно использовать как на базе своей локальной инфраструктуры так и на облачной.
- Это бесплатно, с открытым исходным кодом (https://github.com/ansible/ansible) и с обширной поддержкой сообщества.
- Команды, задачи представлены в виде кода, согласно подходу "Инфраструктура как код" - Infrastructure As Code (IaC).
  - Благодаря IaC, задачи легко хранить, контролировать версии, повторять и тестировать.
  - Благодаря IaC, вся конфигурация определяется декларативным подходом.
- **Agentless:** - для запуска не требуется агент на рабочей машине.
- **Parallel Run:** Ansible позволяет запускать одну и ту же задачу на нескольких хостах параллельно.
- Задачи можно запускать как на Linux так и на Windows.
- Есть подробная документация (https://docs.ansible.com)
- Ansible использует SSH для связи с другими машинами.
- Ansible управляет большинством задач при помощи модулей.

  ![image](https://user-images.githubusercontent.com/10358317/202701707-b160e35c-7a05-43e8-93c7-b626c8054aa9.png) (ref: medium)

- Ansible лежит в основе других приложений (например, Open Source Gating CI Tools: [Zuul-CI](https://zuul-ci.org/))
- Ansible используется для управления конфигурациями на удаленных машинах и его можно легко интегрировать в другие решения (например, [Terraform](https://www.terraform.io/)).

## Что такое Ansible? <a name="what_is_ansible"></a>
- "Ansible это программый инструмент, предоставляющий простой и мощный способ автоматизировать кросс-платформенные удаленные машины." (ref: Opensource.com)

## Как работает Ansible? <a name="how_ansible_works"></a>

- При использовании Ansible, есть две категории компьютеров: управляющий хост (control node) и управляемые хосты - клиенты (managed nodes). Ansible запускается на управляющем компьютере. Необходим как минимум одни управляющий хост, может присутствовать резервный или конфигурация может быть доступна другим специалистам. Клиентом может быть любое устройство, к которому у управляющего хоста есть доступ. 
- Ansible осуществляет подключение к клиентам (серверы, ПК, сетевое оборудование, IoT и прочее) по сети, и отправляет на них небольшие программы-скрипты, называемые Ansible модулями. Ansible запускает данные модули посредством SSH и удаляет их после завершения.(ref: Opensource.com)
- Основное требование для использования Ansible - управляющий хост должен иметь доступ на клиенты. SSH ключи наиболее предпочтительный способ для предоставления доступа, однако поддерживаются и другие формы аутентификации. (ref: Opensource.com)
- Absible использует следующие файлы для настройки и управления клиентами:
  - **Inventory File**: файл инвентаризации - содержит группы клиентов, их IP-адреса или доменные имена. Ansible использует этот файл для отправки команд на клиенты, обычно располагается в /etc/ansible/hosts.
  - **Configuration (.cfg) File**: файл конфигурации - содержит настройки, такие как: расположение inventory file, расположение key_file, имена удаленных пользователей и другие.
  - **Playbook**: файл сценария (плейбук) - одна из ключевых функций Ansible. Файл может содержать несколько задач, которые необходмо выполнить на клиентах. Плейбуки пишутся на YAML. 
- Другие определения:  
  - **Control Machine / Node**: управляющий хост - система, где установлен Ansible и настроен на подключение и выполнение команд на клиентских машинах.
  - **Node (Worker node)**: клиент - хост, который настраивается при помощи Ansible.
  - **Role**: роль - коллекция плейбуков и других файлов, необходимых для достижения определенной цели на клиентах, например установка веб-сервера с определенными настройками и файлами.
  - **Play**: запуск Ansible. При помощи play можно запустить несколько плейбуков и ролей, включенных (include) в один плейбук - точку входа.

  ![image](https://user-images.githubusercontent.com/10358317/202699093-62fcc145-c023-43ed-af51-0866393f0701.png) (ref: kreyman.de)

## Подготовка лабораторного стенда <a name="lab_environment"></a>

- Для тестирования Ansible, Ansible модулей, потребуется несколько ПК или виртуальных машин. 
- В данном курсе будет использоваться Multipass - легковесное решение для быстрого запуска ВМ на базе Ubuntu.
- Установка на Linux, Windows и MacOs: https://multipass.run/install
- [LAB: Multipass-SSH Configuration (Create Ansible Test Environment)](https://github.com/gulyaeve/ansible_labs/blob/main/Multipass-SSH-Configuration.md)

## Базовые команды Ansible (Ad-Hoc Commands) <a name="commands"></a>

- Команды можно отправлять на все или на выбранные клиенты с управляющего хоста.  
- Структура команды: 
  ![image](https://user-images.githubusercontent.com/10358317/203532819-acf44653-de14-426b-9656-35fa82bd4721.png) 

- Примеры команд:
``` 
# сбор информации с определенного хоста по ip-адресу 
ansible all -m gather_facts --limit 172.26.215.23
# сбор информации со всех хостов
ansible all -m gather_facts
# обновить список пакетов на всех хостах (sudo apt-get update), 'become' используется для sudo (УЗ должна быть с одинаковым паролем на всех хостах)
ansible all -m apt -a update_cache=true --become --ask-become-pass
# установить snapd на всех ubuntu машинах (sudo apt-get install snapd)
ansible all -m apt -a "name=snapd state=latest" --become --ask-become-pass
# обновить все хосты (sudo apt-get upgrade)
ansible all -m apt -a upgrade=dist --become --ask-become-pass
# выполнить команду shell
ansible all -m shell -a "cat /proc/meminfo|head -2" 
ansible web_servers -m shell -a "cat /proc/meminfo|head -2" 
# проверить свободное место на дисках
ansible all -m command -a "df -h"
ansible all  -a "df -h"
# проверить статус сервиса httpd
ansible all -m service -a "name=httpd"
# узнать время работы хоста (uptime) при помощи модуля shell
ansible all -m shell -a uptime
# создать файл в '/tmp' с правами доступа mode=0755
ansible all -m file -a "path=/tmp/testfile state=touch mode=0755"
# список всех хостов
ansible all --list-hosts
``` 

- Если создать файл инвентаризации (nano inventory) и сгруппировать хосты по ключевым словам, то вы сможете использовать названия групп 'web_servers' и 'database_servers' в командах, представленных выше.
``` 
[web_servers]
172.21.67.249

[database_servers]
172.21.75.98
``` 
- Перейдите к лабораторной работе:
  - [LAB: Install Ansible and Test Basic Ansible (Ad-Hoc) Commands](https://github.com/gulyaeve/ansible_labs/blob/main/Install-Ansible-Basic-Commands.md)

## Плейбуки Ansible <a name="playbooks"></a>

- Вместо использования Adhoc команд, используются специальные файлы-плейбуки для хранения и управления задачами (декларативный подход)
  
  ![image](https://user-images.githubusercontent.com/10358317/203531052-f9fc2527-06cc-4503-b042-bca5997b5bd0.png) 

- Плейбуки используют формат YAML и включают в себя название, хосты (группы из файла инвентаризации), vars (переменные, которые используются в плейбуке) и задачи:

```
--- 
   name: install and configure DB
   hosts: testServer
   become: yes

   vars: 
     oracle_db_port_value : 1521
   
   tasks:
   -name: Install the Oracle DB
      yum: <code to install the DB>
    
   -name: Ensure the installed service is enabled and running
    service:
      name: <your service name>
```  

- Плейбуки могут включать имена хостов, информацию о пользователе и задачах (вызываемые модули):

  ![image](https://user-images.githubusercontent.com/10358317/203531873-cf746f02-67cd-4d2d-98f8-fd7b4900b614.png) 
    
- Перейдите к лабораторной работе:
  - [LAB: Implement First Playbook](https://github.com/gulyaeve/ansible_labs/blob/main/Implement-First-Playbook.md)

## Модули Ansible <a name="modules"></a>

- Модули: Небольшие программы для выполнения определенной задачи.
  - Некоторые примеры модулей:
    - Установка пакетов программ (sudo apt install),
    - Обновить пакеты, акутуализировать пакетную базу репозиториев (sudo apt upgrade, sudo apt update),
    - Создание и копирование файлов, запуск сервисов,
    - Скачивание Docker Образа, Запуск/Остановка Docker контейнеров,
    - Запуск/Остановка Nginx сервера, итд.     
- Управляющий хост отправляет модули на клиентские хосты.

- Перейдите к лабораторной работе:
  - [LAB: Playing Docker Module](https://github.com/gulyaeve/ansible_labs/blob/main/Docker-Module.md)

- Перейдите к лабораторной работе:
  - [LAB: Important (Mostly Possible Used) Modules Sample Tasks](https://github.com/gulyaeve/ansible_labs/blob/main/Important-Modules-Sample-Tasks.md)

- Перейдите к лабораторной работе:
  - [LAB: Refactoring / Improving Playbook](https://github.com/gulyaeve/ansible_labs/blob/main/Refactoring-Playbook.md)
  
- Все модули в Ansible: 
  - [Cloud Modules (AWS, Azure, Digital Ocean, Docker, Google Cloud, OpenStack, Vmware)](https://docs.ansible.com/ansible/2.9/modules/list_of_cloud_modules.html)
  - [Clustering Modules (Kubernetes, ETCD)](https://docs.ansible.com/ansible/2.9/modules/list_of_clustering_modules.html)
  - [Command Modules (Command, Expect, Shell, Script)](https://docs.ansible.com/ansible/2.9/modules/list_of_commands_modules.html)
  - [Crypto Modules (OpenSSL, ACME)](https://docs.ansible.com/ansible/2.9/modules/list_of_crypto_modules.html)
  - [Database Modules (MySql, PostgreSql, MongoDB, ElasticSearch, Redis, Kibana, InfluxDB)](https://docs.ansible.com/ansible/2.9/modules/list_of_database_modules.html)
  - [Files Modules (Copy, LineIn, Find, File, Unarchive, Read_CSV)](https://docs.ansible.com/ansible/2.9/modules/list_of_files_modules.html)
  - [Messaging Modules (RabbitMQ)](https://docs.ansible.com/ansible/2.9/modules/list_of_messaging_modules.html)
  - [Monitoring Modules (Datadog, Grafana, Nagios, Zabbix)](https://docs.ansible.com/ansible/2.9/modules/list_of_monitoring_modules.html)
  - [Network Modules](https://docs.ansible.com/ansible/2.9/modules/list_of_network_modules.html)
  - [Notification Modules (MQTT, RabbitMq, Mattermost, Slack, Telegram)](https://docs.ansible.com/ansible/2.9/modules/list_of_notification_modules.html)
  - [Packaging Modules (Apt, Dnf, Homebrew, Snap, Package)](https://docs.ansible.com/ansible/2.9/modules/list_of_packaging_modules.html)
  - [Source Code Modules (Github, Gitlab, Bitbucket)](https://docs.ansible.com/ansible/2.9/modules/list_of_source_control_modules.html)
  - [System Modules (Authorized Keys, Cron, IpTables, Make, Sysctl,)](https://docs.ansible.com/ansible/2.9/modules/list_of_system_modules.html)
  - [Web Infrastructure Modules (Apache, Django, Jenkins, Jira, Ansible Tower)](https://docs.ansible.com/ansible/2.9/modules/list_of_web_infrastructure_modules.html)
  - [Windows Modules (Command, Chocotaley, Environment, File, Find, Firewall, Ping, Powershell, Regedit, Service, Shell, User, SNMP)](https://docs.ansible.com/ansible/2.9/modules/list_of_windows_modules.html)
  - [All Modules (Alphabet Order)](https://docs.ansible.com/ansible/2.9/modules/list_of_all_modules.html)

## Файл инвентаризации - указание определенных хостов <a name="inventory"></a>

- Для группировки хостов (по имени группы), можно использовать файл инвентаризации (nano inventory): 
``` 
[web_servers]
172.21.67.249

[database_servers]
172.21.75.98
``` 
- Перейдите к лабораторной работе:
  - [LAB: Targeting Specific Nodes (Grouping)](https://github.com/gulyaeve/ansible_labs/blob/main/Targeting-Specific-Node.md)

## Теги <a name="tags"></a>
- При помощи тегов можно запускать определенную часть плейбука.

```
ansible-playbook --tags ubuntu --ask-become-pass site.yml
```

![image](https://user-images.githubusercontent.com/10358317/203546898-9dc1b0b5-929a-42ba-9cc4-7d919377aaba.png)

- Перейдите к лабораторной работе:
  - [LAB: Adding Tags](https://github.com/gulyaeve/ansible_labs/blob/main/Tags.md)

## Управление файлами <a name="files"></a>
- Есть возможность передавать файлы с управляющего хоста на клиентские хосты, также можно скачать файл из интернет, например zip-архив, и распаковать его при помощи плейбука.
  
  ![image](https://user-images.githubusercontent.com/10358317/203547001-fb106bbe-1171-4ca0-9d8d-5343907c6cf9.png)
  
- Перейдите к лабораторной работе:
  - [LAB: Managing Files](https://github.com/gulyaeve/ansible_labs/blob/main/Managing-Files.md)

## Управление сервисами <a name="services"></a>
- Есть возможность управлять состоянием сервисов (служб или демонов): create, start, stop, restart, изменение service file

```
- name: start apache (Ubuntu)
  tags: ubuntu,apache,apache2
  service:
    name: apache2
    state: started
    enabled: yes
  when: ansible_distribution == "Ubuntu"
```

- Перейдите к лабораторной работе:
  - [LAB: Managing Services](https://github.com/gulyaeve/ansible_labs/blob/main/Managing-Services.md)

## Добавление пользователей <a name="users"></a>
- Есть возможность управлять пользователями (добавление, создание SSH-ключей, редактирование sudoers)

```
- name: create new user
  user:
    name: newuser111
    groups: root
```

- Перейдите к лабораторной работе:
  - [LAB: Adding Users](https://github.com/gulyaeve/ansible_labs/blob/main/Adding-User.md)

## Роли <a name="roles"></a>
- Роли созданы для упрощения контроля над Ansible кодом, так как будто это код программного обеспечения. 
- Роли могут буть назначены группам хостов и роли помогают определить задачи для этих хостов. 

  ![image](https://user-images.githubusercontent.com/10358317/203546694-cb961a0b-e1e3-4e06-9f2d-0e70f1ef6cc8.png)

Для создания новой роли можно набрать команду для инициализации структуры каталогов и файлов:
```
ansible-galaxy role init "role_name"
```

- Перейдите к лабораторной работе:
  - [LAB: Roles](https://github.com/gulyaeve/ansible_labs/blob/main/Roles.md)

## Переменные хостов <a name="host_variables"></a>
- Есть возможность определить переменные для каждого из хостов. 

  ![image](https://user-images.githubusercontent.com/10358317/203547741-9a52592b-7385-4e77-89ad-d8128ecf16b7.png)
  
  ![image](https://user-images.githubusercontent.com/10358317/203547770-7eef113c-c9ce-4043-9b02-588ba86fd747.png)

- Перейдите к лабораторной работе:
  - [LAB: Host Variables](https://github.com/gulyaeve/ansible_labs/blob/main/Host-Variables.md)

## Хэндлеры <a name="handlers"></a>
- Хэндлеры (обработчики) используются для вызова (Notify) определённых Ansible задач.

  ![image](https://user-images.githubusercontent.com/10358317/203547977-c5ced352-d323-4be6-bcd0-31a6d3df2735.png)

- Перейдите к лабораторной работе:
  - [LAB: Handlers](https://github.com/gulyaeve/ansible_labs/blob/main/Handlers.md)

## Шаблоны <a name="templates"></a>
- Модуль template в Ansible предназначен для: 
  - Используя синтаксис шаблонзатора Jinja2 заменять переменные, записанные в ({{ }}) в файле шаблона, на их значения.
  - Копировать на удаленный хост изменённый файл используя scp.

- Как выглядит Jinja2 в плейбуке:
  
  ![image](https://user-images.githubusercontent.com/10358317/203617129-308c53fb-7720-4d12-a55e-b4aa71dc9957.png)
  
- В Ansible {{ }} используются также как и ${ } в shell-скриптах

- Вы можете запустить плейбук с измененными переменными.

```
ansible-playbook findtest.yaml -e "DIR=/apps/Tomcat FILEEXT=*.log DAYSOLD=30"
```

- Перейдите к лабораторной работе:
  - [LAB: Templates](https://github.com/gulyaeve/ansible_labs/blob/main/Templates.md)

## Дебаггинг <a name="debugging"></a>

- Для более подробного вывода используте опцию verbosity, можно менять уровень при помощи -v, -vv, -vvv.

```
ansible all -m shell -a uptime -v
ansible all -m shell -a uptime -vv
ansible all -m shell -a uptime -vvv
```

## Дополнительные возможности <a name="details"></a>

```
ansible-playbook <YAML> -f 10                       # Запустить параллельно на 10 хостах, по умолчанию f=5
ansible-playbook <YAML> -C                          # Тестовый запуск
ansible-playbook <YAML> -C -D                       # Dry run - запуск без изменений
ansible-playbook <YAML> --user <username>           # Использовать другое имя пользователя (или -u <username>)
ansible-playbook <YAML> --private-key <key>         # Указать SSH-ключ (обычно берётся из ~/.ssh) (или --key-file <key>)
ansible-playbook <YAML> --ssh-extra-args            # Передать дополнительные опции в SSH
ansible-playbook <YAML> --vault-id <id>             # Использовать хранилище с идентификатором ID
ansible-playbook <YAML> --vault-password-file <key> # Использовать хранилище паролей из файла
ansible-playbook <YAML> --ask-vault-pass            # Запросить ввод пароля от хранилища
ansible-playbook <YAML> --become                    # Повысить привелегии
ansible-playbook <YAML> --ask-become-pass           # Запросить ввод пароля для повышения привелегий
ansible-playbook <YAML> -l <host>                   # Запустить на одном хосте
ansible-playbook <YAML> --list-hosts                # Получить список хостов
ansible-playbook <YAML> --list-tasks                # Получить список задач
ansible-playbook <YAML> --syntax-check              # Проверка синтаксиса
ansible-playbook <YAML> --check                     # Проверка плейбука (запуск без изменений)
ansible-playbook <YAML> --diff                      # Показать изменения

```

### Захват вывода shell-команды
```
  tasks:
  - name: some shell
    register: sh_out
    ignore_errors: yes
    become_user: root
    shell: |
      find /

  - name: "Print stdout"
    debug:
      msg: "{{ sh_out.stdout_lines }}"
  - name: "Print stderr"
    debug:
      msg: "{{ sh_out.stderr.split('\n') }}"
```
- Использование секции Debug в плейбуке для вывода:

  ![image](https://user-images.githubusercontent.com/10358317/215115854-588292ce-1707-420a-bf2c-e77c40e5bb09.png)

- Выаод: 

  ![image](https://user-images.githubusercontent.com/10358317/215116236-af140eb4-ec7e-4584-a8cc-65cbe0575a66.png)

### Удаление файлов и директорий
```
tasks:
- name: rm
  file:
    path: <some path>
    state: absent
    recurse: yes        # optional
```

## Задания для самостоятельного выполнения <a name="practice"></a>

- Перейдите к лабораторной работе:
  - [LAB: Practice](https://github.com/gulyaeve/ansible_labs/blob/main/Practice.md)

## Другие полезные ресурсы по Ansible <a name="resource"></a>
- https://docs.ansible.com/ansible/2.9/
- Video Tutorial: https://www.youtube.com/watch?v=3RiVKs8GHYQ&list=PLT98CRl2KxKEUHie1m24-wkyHpEsa4Y70
- https://www.tutorialspoint.com/ansible/
- https://www.digitalocean.com/community/cheatsheets/how-to-use-ansible-cheat-sheet-guide
- https://www.middlewareinventory.com/blog/category/ansible/

## Ссылки <a name="references"></a>
- OpenSource: https://opensource.com/resources/what-ansible
- Video Tutorial: https://www.youtube.com/watch?v=3RiVKs8GHYQ&list=PLT98CRl2KxKEUHie1m24-wkyHpEsa4Y70
- https://www.kreyman.de/index.php/others/linux-kubernetes/213-ansible-verwendungsszenarien
- https://ibm.github.io/cloud-enterprise-examples/iac-conf-mgmt/ansible/
- Medium: https://medium.com/codex/automation-with-ansible-b39706ff777
- RedHat Presentation: https://speakerdeck.com/chrisshort/using-ansible-for-devops?slide=27
- https://www.middlewareinventory.com/blog/ansible-playbook-example/
- https://www.digitalocean.com/community/cheatsheets/how-to-use-ansible-cheat-sheet-guide
- https://www.middlewareinventory.com/blog/ansible-template-module-example/
