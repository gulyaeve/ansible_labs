## LAB: Practice

В этой лабораторной работе:
- Практические задания для самостоятельного выполнения

### Подготовка

- Проверьте выполнены ли у вас данные лабораторные работы:
  - [LAB: Multipass-SSH Configuration (Create Ansible Test Environment)](https://github.com/gulyaeve/ansible_labs/blob/main/Multipass-SSH-Configuration.md)

### Задания

#### Запрет доступа к определенным ресурсам
- Сделайте плейбук, для запрета доступа к www.cisco.com
- Внесите изменения в /etc/hosts, добавив строку:
```
0.0.0.0 www.cisco.com
```
- Проверьте, что ресурс не доступен

#### Настройка Zabbix
- Самостоятельно создайте роль для установки и настройки zabbix-server на хосте [Инструкция](https://www.zabbix.com/download?zabbix=7.2&os_distribution=ubuntu&os_version=24.04&components=server_frontend_agent&db=mysql&ws=apache)
- Также сделайте роль для установки и настройки zabbix-agent на других хостах [Инструкция](https://www.zabbix.com/download?zabbix=7.2&os_distribution=ubuntu&os_version=24.04&components=agent&db=&ws=)
- Один из хостов сделайте zabbix-server, остальные zabbix-agent
- В роли должны использоваться шаблоны, переменные хостов, хэндлеры 
- Примените данные роли на тестовых хостах
