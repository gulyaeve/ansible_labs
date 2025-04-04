## LAB: Install Ansible and Test Basic Ansible (Ad-Hoc) Commands

В этой лабораторной работе:
- установка ansible
- как использовать базовые команды

### Подготовка

- Проверьте выполнена ли у вас предыдущая лабораторная работа, созданные ВМ используются в этой работе
  - [LAB: Multipass-SSH Configuration (Create Ansible Test Environment)](https://github.com/gulyaeve/ansible_labs/blob/main/Multipass-SSH-Configuration.md)

### Шаги

- Установка Ansible

``` 
sudo apt install ansible
``` 

- Создайте директорию и инвентарный файл для хранения IP-адресов клиентов

![image](https://user-images.githubusercontent.com/10358317/201087999-7bbb7f0b-acdf-475a-b8c8-cf0f689bc29b.png)

- Запишите IP-адреса клиентов в файл инвентаризации

![image](https://user-images.githubusercontent.com/10358317/201088310-e7859682-dc0d-46f5-bac4-ba553e38be90.png)

- Проверьте доступность клиентов модулем Ping

``` 
ansible all -i inventory -m ping
``` 

![image](https://user-images.githubusercontent.com/10358317/201089266-84c032d5-7647-45ec-b44a-0323cf7f6274.png)

**NOTE:** При использовании SSH-ключа, при запуске команды необходимо указывать путь до файла с ключом "ansible all --key-file ~/.ssh -i inventory -m ping" 

- Создайте файл конфигурации (ansible.cfg)

``` 
touch ansible.cfg
# Скопируйте в него следующее:
[defaults]
inventory = inventory
# private_key_file = ~/.ssh/ansible
``` 

![image](https://user-images.githubusercontent.com/10358317/201090216-084d1328-88fc-462f-b307-d95c8d8b752d.png)

![image](https://user-images.githubusercontent.com/10358317/201090391-67057ecd-68a9-4aa6-af33-af5fcd099840.png)

- При использовании файла конфигурации (ansible.cfg), можно использовать более короткие команды

![image](https://user-images.githubusercontent.com/10358317/201090690-752feb31-9b42-42df-a89f-63e3092b4a32.png)

- Список всех хостов

```
ansible all --list-hosts
``` 
![image](https://user-images.githubusercontent.com/10358317/201090920-d5d2a294-698a-4e62-89e7-7df3f1d1834d.png)

- Сбор информации со всех хостов (cpu, ip, ssd, итд.)

```
# -m параметр указывает вызываемый модуль ansible
ansible all -m gather_facts
``` 

![image](https://user-images.githubusercontent.com/10358317/201091229-60ab2618-ba53-4460-96f8-7c69a4a9c6b1.png)

- Сбор информации с определенного хоста

```
ansible all -m gather_facts --limit 172.26.215.23
```

- Запустите "sudo apt update" на всех хостах используя ansible
- Как видите, на скриншоте, это не работает 

```
ansible all -m apt -a update_cache=true
```

![image](https://user-images.githubusercontent.com/10358317/201094159-89918be8-1d73-4a10-b346-4d54a1bc104f.png)

- Причина почему "sudo apt update" не работает, в том, что требуется пароль для "sudo".
- Сейчас установим одинаковый пароль на все хосты (node1, node2). Позже сделаем с разными паролями.

```
sudo passwd ubuntu
```

![image](https://user-images.githubusercontent.com/10358317/201094654-23381802-43a2-4261-892b-900244019bcc.png)
![image](https://user-images.githubusercontent.com/10358317/201094744-d8edfd82-9c5a-4bb8-9fc5-7e9f5f4567c1.png)
![image](https://user-images.githubusercontent.com/10358317/201094827-5ddd50dd-bd26-47b9-b266-5d997678774b.png)

- Запустите "sudo apt update" на всех хостах, используя "BECOME PASS", введите пароль во время запроса

```
ansible all -m apt -a update_cache=true --become --ask-become-pass
```

![image](https://user-images.githubusercontent.com/10358317/201095106-cfa74f25-9ae6-4ca2-b34a-061ed5d6622d.png)

- Установите определенный пакет на все хосты "sudo apt get snapd"

```
ansible all -m apt -a name=snapd --become --ask-become-pass
# последняя версия
ansible all -m apt -a "name=snapd state=latest" --become --ask-become-pass
```

![image](https://user-images.githubusercontent.com/10358317/201097511-9b0893f5-120c-4af1-be6d-a35fc15681a5.png)


### Задания для самостоятельного выполнения

- Выведите содержимое файла /etc/hosts со всех хостов
- Получите информаци о RAM на хостах:
```
ansible -m setup -a 'filter=ansible_memtotal_mb' all
```
- Изучите документацию по модулю setup
```
ansible-doc setup
```
- На основе документации найдите способ вывести информацию ТОЛЬКО о сетевых настройках
