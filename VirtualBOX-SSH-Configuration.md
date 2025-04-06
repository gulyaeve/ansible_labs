## LAB: VirtualBOX-SSH-Configuration (Create Ansible Test Environment)

В этой лабораторной работе:
- настройка виртуальных машин в VirtualBOX
- создание control node, managed nodes
- настройка ssh между control node и managed nodes

### Шаги
- Получите у преподавателя шаблон виртуальной машины или установите Ubuntu Server самостоятельно в VirtualBOX
- Разверните ВМ из шаблона, которая будет являться узлом управления Ansible: controlnode (RAM 4GB)
- Разверните 2 управляемых узла из шаблона: node1 и node2 (RAM 2GB)
- Добавьте к каждой ВМ сетевой адаптер типа «Внутренняя сеть»
- Включите все ВМ и смените hostname:
```
sudo hostnamectl set-hostname …
```
- Настроить сетевые адаптеры (внутренняя сеть) вручную (можно использовать nmtui)
```
controlnode 10.0.1.1/24
node1 10.0.1.2/24
node2 10.0.1.3/24
gateway 10.0.1.1
```
- На ВМ master создайте ключевую пару для ssh (все параметры оставьте по умолчанию):
```
ssh-keygen -t rsa
```
- Скопируте публичный ключ на управляемые хосты:
```
ssh-copy-id -i ~/.ssh/id_rsa.pub <ip_addr>
```
- Проверьте доступ по ssh
ssh <ip_addr>
