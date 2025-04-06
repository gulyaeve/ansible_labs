## LAB: Templates

В этой лабораторной работе:
- создание шаблонов.

### Подготовка

- Проверьте выполнены ли у вас данные лабораторные работы:
  - [LAB: Multipass-SSH Configuration (Create Ansible Test Environment)](https://github.com/gulyaeve/ansible_labs/blob/main/Multipass-SSH-Configuration.md)
  - [LAB: VirtualBOX-SSH Configuration (Create Ansible Test Environment)](https://github.com/gulyaeve/ansible_labs/blob/main/VirtualBOX-SSH-Configuration.md)

### Шаги

- Создайте каталог 'templates' в '/roles/base/'
- Копируте файл 'sshd_config' в 'templates' с добавлением суффикса файла jinja2 (.j2)

```
mkdir templates
cd templates
cp /etc/ssh/sshd_config sshd_config_ubuntu.j2
 ```

![image](https://user-images.githubusercontent.com/10358317/202679439-45eceefc-0b29-418c-8b75-5558e722fd87.png)


- Откройте файл 'sshd_config_ubuntu.j2' и добавьте 'AllowUsers'.

```
nano sshd_config_ubuntu.j2
# add following
AllowUsers {{ ssh_users }}
```

![image](https://user-images.githubusercontent.com/10358317/202685736-463d1e46-2f00-4852-a8c3-f6f62e861399.png)

- Перейдите в каталог 'host_vars',
- Добавьте переменные хостов в файлы host_vars

```
nano 172.21.69.156.yml
# add following
ssh_users: "newuser2022"
ssh_template_file: sshd_config_ubuntu.j2
```

![image](https://user-images.githubusercontent.com/10358317/202680949-69c1f1dd-b51f-4900-8bdd-992955707293.png)

- Содержимое файла:

```
apache_package_name: apache2
apache_service: apache2
php_package_name: libapache2-mod-php
ssh_users: "newuser2022"
ssh_template_file: sshd_config_ubuntu.j2
```

![image](https://user-images.githubusercontent.com/10358317/202682564-b6ed46b2-b155-4366-9acc-6158236e5643.png)

- Добавьте новую задачу на применение шаблонов

![image](https://user-images.githubusercontent.com/10358317/202683263-66005498-8f3a-43ac-9745-ec63f6788ddc.png)

- Откройте и добавьте ('nano main.yml')
- После изменения файла sshh_config file, должен сработать хэндлер (restart_ssh)

```
- name: generate sshd_config file using templates
  tags: ssh
  template:
    src: "{{ ssh_template_file }}"
    dest: /etc/ssh/sshd_config
    owner: root
    group: root
    mode: 0644
  notify: restart_sshd
```  

![image](https://user-images.githubusercontent.com/10358317/202692366-c606995e-0eb5-4f7b-82ec-7a398750df1b.png)

- Создайте данный хэндлер в 'main.yml'

```
- name: restart_sshd
  service:
    name: sshd
    state: restarted
```

![image](https://user-images.githubusercontent.com/10358317/202687675-fcc12387-9f4f-4f70-a357-797bf3ad1738.png)

- Запустите:

```
ansible-playbook site.yml
```
![image](https://user-images.githubusercontent.com/10358317/202692817-02c63dd3-86e4-47da-bec9-864cbc3a3510.png)

- После запуска плейбука, перейдите на 'node1' и проверьте содержимое файла sshd_config ('nano /etc/ssh/sshd_config').
- Должно быть добавлено 'AllowUsers newuser2022'.

![image](https://user-images.githubusercontent.com/10358317/202694177-b202a08c-ff59-4739-b0e9-922c7e5d8d41.png)

## Reference

- https://www.youtube.com/watch?v=s8F_YWGHeDM&list=PLT98CRl2KxKEUHie1m24-wkyHpEsa4Y70&index=17

