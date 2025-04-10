## LAB: Create Module for Ansible

### Подготовка окружения

Для тестирования своих модулей необходим только пакет `ansible-core`. Модуль может быть написан на любом языке, но чаще всего используют `Python`.

Для создания своего отдельного модуля:

- Создайте директорию `library` в своём окружении:
```
mkdir library
```
- Создайте в этой директории файл для нового модуля:
```
touch library/my_test.py
```
- Скопируйте содержимое следующего шаблона в ваш файл:
```python
#!/usr/bin/python

# Copyright: (c) 2018, Terry Jones <terry.jones@example.org>
# GNU General Public License v3.0+ (see COPYING or https://www.gnu.org/licenses/gpl-3.0.txt)
from __future__ import (absolute_import, division, print_function)
__metaclass__ = type

DOCUMENTATION = r'''
---
module: my_test

short_description: This is my test module

# If this is part of a collection, you need to use semantic versioning,
# i.e. the version is of the form "2.5.0" and not "2.4".
version_added: "1.0.0"

description: This is my longer description explaining my test module.

options:
    name:
        description: This is the message to send to the test module.
        required: true
        type: str
    new:
        description:
            - Control to demo if the result of this module is changed or not.
            - Parameter description can be a list as well.
        required: false
        type: bool
# Specify this value according to your collection
# in format of namespace.collection.doc_fragment_name
# extends_documentation_fragment:
#     - my_namespace.my_collection.my_doc_fragment_name

author:
    - Your Name (@yourGitHubHandle)
'''

EXAMPLES = r'''
# Pass in a message
- name: Test with a message
  my_namespace.my_collection.my_test:
    name: hello world

# pass in a message and have changed true
- name: Test with a message and changed output
  my_namespace.my_collection.my_test:
    name: hello world
    new: true

# fail the module
- name: Test failure of the module
  my_namespace.my_collection.my_test:
    name: fail me
'''

RETURN = r'''
# These are examples of possible return values, and in general should use other names for return values.
original_message:
    description: The original name param that was passed in.
    type: str
    returned: always
    sample: 'hello world'
message:
    description: The output message that the test module generates.
    type: str
    returned: always
    sample: 'goodbye'
'''

from ansible.module_utils.basic import AnsibleModule


def run_module():
    # define available arguments/parameters a user can pass to the module
    module_args = dict(
        name=dict(type='str', required=True),
        new=dict(type='bool', required=False, default=False)
    )

    # seed the result dict in the object
    # we primarily care about changed and state
    # changed is if this module effectively modified the target
    # state will include any data that you want your module to pass back
    # for consumption, for example, in a subsequent task
    result = dict(
        changed=False,
        original_message='',
        message=''
    )

    # the AnsibleModule object will be our abstraction working with Ansible
    # this includes instantiation, a couple of common attr would be the
    # args/params passed to the execution, as well as if the module
    # supports check mode
    module = AnsibleModule(
        argument_spec=module_args,
        supports_check_mode=True
    )

    # if the user is working with this module in only check mode we do not
    # want to make any changes to the environment, just return the current
    # state with no modifications
    if module.check_mode:
        module.exit_json(**result)

    # manipulate or modify the state as needed (this is going to be the
    # part where your module will do what it needs to do)
    result['original_message'] = module.params['name']
    result['message'] = 'goodbye'

    # use whatever logic you need to determine whether or not this module
    # made any modifications to your target
    if module.params['new']:
        result['changed'] = True

    # during the execution of the module, if there is an exception or a
    # conditional state that effectively causes a failure, run
    # AnsibleModule.fail_json() to pass in the message and the result
    if module.params['name'] == 'fail me':
        module.fail_json(msg='You requested this to fail', **result)

    # in the event of a successful module execution, you will want to
    # simple AnsibleModule.exit_json(), passing the key/value results
    module.exit_json(**result)


def main():
    run_module()


if __name__ == '__main__':
    main()
```

- Для использования своего модуля в Ad-Hoc команде:
```
ANSIBLE_LIBRARY=./library ansible -m my_test -a 'name=hello new=true' all
```
    Здесь путь к расположению модуля задаётся при помощи переменной окружения
- Использование в плейбуке (переменная окружения также нужна)
```
- name: test my new module
  hosts: localhost
  tasks:
  - name: run the new module
    my_test:
      name: 'hello'
      new: true
    register: testout
  - name: dump test output
    debug:
      msg: '{{ testout }}'
```

### Пример модуля

Далее представлен пример модуля для создания указанного количества файлов (`count`) по указанному пути (`path`):
```python
#!/usr/bin/python

from __future__ import (absolute_import, division, print_function)
__metaclass__ = type

DOCUMENTATION = r'''
---
module: create_files

short_description: create files in path

# If this is part of a collection, you need to use semantic versioning,
# i.e. the version is of the form "2.5.0" and not "2.4".
version_added: "1.0.0"

description: This is my longer description explaining my test module.

options:
    count:
        description: count of files.
        required: true
        type: str
    path:
        description:
            - Filesystem path.
        required: true
        type: str

author:
    - Your Name (@yourGitHubHandle)
'''

EXAMPLES = r'''
# Pass in a message
- name: Test with a message
  my_namespace.my_collection.my_test:
    count: 10
    path: /tmp
'''

RETURN = r'''
# These are examples of possible return values, and in general should use other names for return values.
test_echo:
    type: str
    returned: always
test:
    type: str
    returned: always
'''

import os

from ansible.module_utils.basic import AnsibleModule


def run_module():
    module_args = dict(
        path=dict(type='str', required=True),
        count=dict(type='int', required=False, default=1)
    )

    result = dict(
        changed=False,
        files='',
        test_echo=''
    )

    module = AnsibleModule(
        argument_spec=module_args,
        supports_check_mode=True
    )

    if module.check_mode:
        module.exit_json(**result)

    # --------
    files = []

    for file_number in range(module.params['count']):
        file_name = os.path.join(
            module.params['path'],
            f"file_{file_number}.txt"
        )
        if not os.path.exists(file_name):
            with open(file_name, 'w') as new_file:
                new_file.write('hello')
                files.append(file_name)

    result['files'] = files

    if len(files) > 0:
        result['changed'] = True

    out = module.run_command('/usr/bin/echo "hello"')
    result['test_echo'] = out[1]
    result['test'] = 123

    module.exit_json(**result)


def main():
    run_module()


if __name__ == '__main__':
    main()
```

### Задание для самостоятельного выполнения

- Напишите модуль для замены командного интерпритатора у всех пользователей
- На вход модуль получает значение для замены и на какой менять
- Изменения не записываются в /etc/passwd, а выполняются командой
- Пример: заменить /bin/bash на /usr/bin/zsh
