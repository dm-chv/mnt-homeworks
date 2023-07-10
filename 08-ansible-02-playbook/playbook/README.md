### PlayBook for Ansible 
## Данный плейбук для ансибла устанавливает clickhouse, lighthouse, nginx и vector на redhat based os x86_64.

1. Определение переменных в дирректории group_vars
* В vector/vars.yml  указывается версия приложения Vector
* В clickhouse/vars.yml указывается необходимая версия и какие пакеты clickhouse будут установлены(клиент, сервер, common)
* В lighthouse/vars.yml указываем исходник лайтхауса, директорию для установки
2. В prod.yml указываем хосты для установки
3. В site.yml происходит вся магия:
* Скачиваем пакет вектора нужной нам разрядности в указанную папку(dest:)
```
- name: Get Vector distrib
        ansible.builtin.get_url:
          url: https://packages.timber.io/vector/{{ vector_version }}/vector-{{ vector_version }}-1.x86_64.rpm
          dest: "./vector-{{ vector_version }}.rpm"
```
* Устанавливаем скаченный пакет
```
- name: install vector packages
      become: true
      ansible.builtin.yum:
        name:
          - vector-{{ vector_version }}.rpm
```
* Переходим к clickhouse
* ansible.builtin.service: - Запускает сервис clickhouse  по имени демона "clickhouse-server"
```
name: Install Clickhouse
  hosts: clickhouse
  handlers:
    - name: Start clickhouse service
      become: true
      ansible.builtin.service:
        name: clickhouse-server
        state: restarted
```
* Данная задача скачивает определенные пакеты кликхаус(clickhouse_packages) указанной версии(clickhouse_version) в определенное место(dist:)
```
- name: Get clickhouse distrib
          ansible.builtin.get_url:
            url: "https://packages.clickhouse.com/rpm/stable/{{ item }}-{{ clickhouse_version }}.noarch.rpm"
            dest: "./{{ item }}-{{ clickhouse_version }}.rpm"
          with_items: "{{ clickhouse_packages }}"
```
*  Отдельно скачиваем пакет clickhouse-common-static, т.к. он не подходит под предыдущий шаблон
```
- name: Get clickhouse distrib
          ansible.builtin.get_url:
            url: "https://packages.clickhouse.com/rpm/stable/clickhouse-common-static-{{ clickhouse_version }}.x86_64.rpm"
            dest: "./clickhouse-common-static-{{ clickhouse_version }}.rpm"
```
* Устанавливаем Скаченные пакеты кликхауса
```
 - name: Install clickhouse packages
      become: true
      ansible.builtin.yum:
        name:
          - clickhouse-common-static-{{ clickhouse_version }}.rpm
          - clickhouse-client-{{ clickhouse_version }}.rpm
          - clickhouse-server-{{ clickhouse_version }}.rpm
```
* Создаем БД для логов
```
 ansible.builtin.command: "clickhouse-client -q 'create database logs;'"
      register: create_db
      failed_when: create_db.rc != 0 and create_db.rc !=82
      changed_when: create_db.rc == 0
```
* Переходим к установке nginx
```
nginx мы устанавливаем из репозитория по умолчанию с помощью команды ansible.builtin.yum:
далее с помощью шаблона подкидываем ему дефолтный конфиг с измененным путем к страничке lighthouse.
```

* Переходим к установке lighthouse
```
Для начала ставим git
      ansible.builtin.yum:
        name: git
далее клонируем гит ветку лайтхауса, перезапускаем nginx

```