## Ansible playbook: стек clickhouse, lighthouse, vector.
<b>Задача этого плейбука - автоматизировать установку пакетов clickhouse, lighthouse, vector.</b>

## Описание:
Данный плейбук устанавливает стек на 3 сервера.


## Требования:
* Сервер: redhat based OS x86_64;
* ansible 2.10.8 или старше(на более ранних версиях не тестировалось)

## Переменные:

|Имя           |Значение по умолчанию | Описание |
|:------------:|:--------------------|:--------|
|clickhouse_version |22.3.3.44|Версия clickhouse пакета |
|clickhouse_packages:| - clickhouse-client <br> - clickhouse-server<br> - clickhouse-common-static |Перечень устанавливаемых пакетов clickhouse|
|lighthouse_vcs|https://github.com/VKCOM/lighthouse.git|Ссылка на исходный код lighthouse|
|lighthouse_location_dir|/home/pixel/lighthouse|Назначение папки установки lighthouse|
|vector_version|0.30.0|Версия Vector пакета|

<b> inventory/prod.yml - необходимо записать адреса серверов в переменные ansible_host для каждого сервера: </b>
```
clickhouse:
  hosts:
    clickhouse-01:
      ansible_host: ip_server_clickhouse
vector:
  hosts:
    vector-01:
      ansible_host: ip_server_vector
lighthouse:
  hosts:
    lighthouse-01:
      ansible_host: ip_server_lighthouse
```
## Описание процесса установки:

<b>Site.yml</b> состоит из поочередной установки пакетов и дальнейшего подкидывания конфигов серверу  
* Для изменения пути скачивания какого-либо пакета - измените параметр <i> url:</i> после <i>ansible.builtin.get_url:</i>
* Для изменения дирректории назначения скачивания пакета - измените параметр <i> dest:</i> после <i> url:</i>
```
        ansible.builtin.get_url:
          url: "https:source_URL_to_file.rpm"
          dest: "./name_file.rpm"
```
* Обратите внимание, что также необходимо установить веб-сервер(nginx)
```
 tasks:
    - name: NGINX | Install epel-release
      become: true
      ansible.builtin.yum:
        name: epel-release
        state: present
    - name: nginx | Install nginx
      become: true
      ansible.builtin.yum:
        name: nginx
        state: present
      notify: start-nginx
```
* Lighthouse исходник берём с GitHub

```
tasks:
    - name: lighthouse | Copy from git
      git:
        repo: "{{ lighthouse_vcs }}"
        version: master
        dest: "{{ lighthouse_location_dir }}"
    - name: lighthouse | Create lighthouse config
```

* Для связки вебсервера с lighthouse необходимо подкинуть файлы конфигурации <b>templates/lighthouse.conf.j2</b> и <b>templates/nginx.conf.j2</b> после таски с установкой Lighthouse и nginx, пример:  

``` EXAMPLE:
 - name: lighthouse | Create lighthouse config
      become: true
      template:
        src: lighthouse.conf.j2
        dest: /etc/nginx/conf.d/default.conf
        mode: "0644"
      notify: reload-nginx
```
## Описание шаблонов(templates)
* Для lighthouse пишем простой конфиг, который слушает 80-й порт и задает дирректорию с index.html
```
server {
    listen    80;
    server_name localhost;

        location / {
        root    {{ lighthouse_location_dir }};
        index  index.html;
    }
}
```
* Для nginx берём стандартный конфиг и ставим путь к lighthouse

```
location / {
            root   {{ lighthouse_location_dir }};
            index  index.html index.htm;
        }
```