# Домашнее задание к занятию 5 «Тестирование roles» - `Иншаков Владимир`

## Подготовка к выполнению

1. Установите molecule и его драйвера: `pip3 install "molecule molecule_docker molecule_podman`.
2. Выполните `docker pull aragast/netology:latest` —  это образ с podman, tox и несколькими пайтонами (3.7 и 3.9) внутри.

## Установка molecule была осуществлена в виртуальном окружении venv. Docker образ загрузился без проблем

----

## Основная часть

Ваша цель — настроить тестирование ваших ролей. 

Задача — сделать сценарии тестирования для vector. 

Ожидаемый результат — все сценарии успешно проходят тестирование ролей.

### Molecule

1. Запустите  `molecule test -s ubuntu_xenial` (или с любым другим сценарием, не имеет значения) внутри корневой директории clickhouse-role, посмотрите на вывод команды. Данная команда может отработать с ошибками или не отработать вовсе, это нормально. Наша цель - посмотреть как другие в реальном мире используют молекулу И из чего может состоять сценарий тестирования.
2. Перейдите в каталог с ролью vector-role и создайте сценарий тестирования по умолчанию при помощи `molecule init scenario --driver-name docker`.
3. Добавьте несколько разных дистрибутивов (oraclelinux:8, ubuntu:latest) для инстансов и протестируйте роль, исправьте найденные ошибки, если они есть.
4. Добавьте несколько assert в verify.yml-файл для проверки работоспособности vector-role (проверка, что конфиг валидный, проверка успешности запуска и др.). 
5. Запустите тестирование роли повторно и проверьте, что оно прошло успешно.
5. Добавьте новый тег на коммит с рабочим сценарием в соответствии с семантическим версионированием.

---

### Molecule

Вывод работы команды `molecule test default` после инициализации директории с ролью "vector-role"
![Screen01](https://github.com/MrVanG0gh/Netology_ansible_05_testing/blob/main/screens/Screen01.png)
![Screen02](https://github.com/MrVanG0gh/Netology_ansible_05_testing/blob/main/screens/Screen02.png)

Были добавлены несколько дополнительных дистрибутивов в файл molecule.yml:

```
# Dependency management (download roles/collections)
dependency:
  name: galaxy
  options:
    ignore-certs: false
    ignore-errors: false
    role-file: requirements.yml
    requirements-file: requirements.yml
driver:
  name: docker
platforms:
  - name: oraclelinux
    image: docker.io/pycontribs/oraclelinux:8
    pre_build_image: true
  - name: ubuntu
    image: dockeer.io/pycontribs/ubuntu:latest
    pre_build_image: true

ansible:
  cfg:
    defaults:
      host_key_checking: false
      verbosity: 1
    ssh_connection:
      pipelining: true
  env:
    ANSIBLE_FORCE_COLOR: "1"
    ANSIBLE_LOAD_CALLBACK_PLUGINS: "1"

  executor:
    backend: ansible-playbook
    args:
      ansible_playbook:
        - --diff
        - --force-handlers
        - --inventory=/path/to/inventory.yml
      ansible_navigator:
        - --mode stdout
        - --pull-policy missing
        - --execution-environment-image ghcr.io/ansible/community-ansible-dev-tools:latest

  playbooks:
    create: create.yml
    converge: converge.yml
    destroy: destroy.yml
    cleanup: cleanup.yml
    prepare: prepare.yml
    side_effect: side_effect.yml
    verify: verify.yml

scenario:
  name: default
  test_sequence:
    - dependency
    - cleanup
    - destroy
    - syntax
    - create

```
verify.yml:
```
---
# Purpose: assert that the instance really ended up in the expected state.
# Molecule calls this playbook with `molecule verify`.
- name: Verify
  hosts: instance
  gather_facts: false # Quicker, if you do not need facts
  tasks:
    - name: Assert something
      ansible.builtin.assert:
        that: true
    - name: Assert configuration is valid
      assert:
        that:
          - validation_result.rc == 0
        fail_msg: "Configuration validation failed!"
    - name: Validate the configuration syntax
      command: vector validate /etc/vector/config.toml
      register: validation_result
      failed_when: validation_result.rc != 0
      changed_when: False

```
Осуществил проверку работы роли Вывод команды `molecule test`

![Screen03](https://github.com/MrVanG0gh/Netology_ansible_05_testing/blob/main/screens/Screen03.png)
![Screen04](https://github.com/MrVanG0gh/Netology_ansible_05_testing/blob/main/screens/Screen04.png)

Оценив результаты, можно сказать, что в данном примере присутствуют предупреждения, но нет ошибок.

---

### Tox

1. Добавьте в директорию с vector-role файлы из [директории](./example).
2. Запустите `docker run --privileged=True -v <path_to_repo>:/opt/vector-role -w /opt/vector-role -it aragast/netology:latest /bin/bash`, где path_to_repo — путь до корня репозитория с vector-role на вашей файловой системе.
3. Внутри контейнера выполните команду `tox`, посмотрите на вывод.
5. Создайте облегчённый сценарий для `molecule` с драйвером `molecule_podman`. Проверьте его на исполнимость.
6. Пропишите правильную команду в `tox.ini`, чтобы запускался облегчённый сценарий.
8. Запустите команду `tox`. Убедитесь, что всё отработало успешно.
9. Добавьте новый тег на коммит с рабочим сценарием в соответствии с семантическим версионированием.

После выполнения у вас должно получится два сценария molecule и один tox.ini файл в репозитории. Не забудьте указать в ответе теги решений Tox и Molecule заданий. В качестве решения пришлите ссылку на  ваш репозиторий и скриншоты этапов выполнения задания. 

### Tox

Необходимые файлы tox.ini и tox-requirements.txt были добавлены в репозиторий.
Я обнаружил, что не смогу запустить контейнер в рамках ARM64 архитектуры, поэтому выбрал виртуалку на базе AMD64.

Выполнение tox:
![Screen05](https://github.com/MrVanG0gh/Netology_ansible_05_testing/blob/main/screens/Screen05.png)
![Screen06](https://github.com/MrVanG0gh/Netology_ansible_05_testing/blob/main/screens/Screen06.png)

Более облегченный сценарий:

![Screen07](https://github.com/MrVanG0gh/Netology_ansible_05_testing/blob/main/screens/Screen07.png)
![Screen08](https://github.com/MrVanG0gh/Netology_ansible_05_testing/blob/main/screens/Screen08.png)

