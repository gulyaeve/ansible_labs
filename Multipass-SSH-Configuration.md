## LAB: Multipass-SSH-Configuration (Create Ansible Test Environment)

В этой лабораторной работе:
- установка multipass
- создание control node, managed nodes
- настройка ssh между control node и managed nodes



### Шаги

- "Multipass представляет собой мини-облако (mini-cloud) на вашей рабочей станции, использующее нативный гипервизор в разных ОС (Windows, macOS and Linux)"
- Быстрая установка и использование.
- **Link:** https://multipass.run/
- Установка на Linux, Windows и MacOs: https://multipass.run/install
- После установки будет возможность создавать ВМ на локальной машине.

``` 
# creating controlnode, managed nodes (node1, node2, etc.)
# -c => cpu, -m => memory, -d => disk space
multipass launch --name controlnode -c 2 -m 2G -d 10G   
multipass launch --name node1 -c 2 -m 2G -d 10G
multipass launch --name node2 -c 2 -m 2G -d 10G
multipass list
``` 

![image](https://user-images.githubusercontent.com/10358317/201082045-b90b645b-c6b5-4d8f-9bd6-4bb09c0f0ea7.png)

- Подключение к ВМ через открытие командной оболочки

``` 
# get shell on controlnode
multipass shell controlnode
# get shell on node1, on different terminal
multipass shell node1
# get shell on node2, on different terminal
multipass shell node2
``` 

![image](https://user-images.githubusercontent.com/10358317/201082458-97b41058-5389-4301-b1e2-06a1b9d3a4ba.png)

``` 
sudo apt update
sudo apt install net-tools
# to see IPs
ifconfig
``` 

- Создайте ssh public key на controlnode
- Копируйте public key с controlnode

``` 
> on controlnode
ssh-keygen (no password, enter 3 times)
cat ~/.ssh/id_rsa.pub (copy the value)
``` 

![image](https://user-images.githubusercontent.com/10358317/201083201-8e0a9bfb-8001-429e-881f-d38a7c970015.png)

- Вставьте скопированный public key (controlnode) в файл authorized_keys на каждой managed nodes.

``` 
cd .ssh (on each workers)
nano authorized_keys 
> Paste keys from controlnode (hence managed nodes know the controlnode IP and public key, controlnode can connect it)
``` 

![image](https://user-images.githubusercontent.com/10358317/201083610-d4141690-d5d7-4f9c-90ba-dbfb6743b2d1.png)

- Список всех ВМ с ip-адресами

```
multipass list
```

![image](https://user-images.githubusercontent.com/10358317/201084356-c34f3629-7e86-4e15-9cad-3361b5a49f34.png)

- SSH-подключение с controlnode в node1

```
ssh <IP>
or 
ssh <username>@<IP>
```

![image](https://user-images.githubusercontent.com/10358317/201084577-1028dc59-be04-4cb4-982b-f3dca1ea6251.png)

### References
- https://techsparx.com/linux/multipass/enable-ssh.html
