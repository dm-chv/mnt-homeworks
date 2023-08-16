## Ansible playbook: стек clickhouse, lighthouse, vector.
<b>Задача этого плейбука - автоматизировать установку пакетов clickhouse, lighthouse, vector с использованием ролей.</b>

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

<b>Site.yml</b> определяет на каких серверах какие роли мы устанавливаем  
* В каждом блоке описывается имя сервера на который будут ставиться пакеты, описанный в одноименно названной роле

<b>requirements.yml</b> Описаны ссылки на репозитории ролей

## Запуск:

```
#скачиваем роли
ansible-galaxy install -r requirements.yml -p roles
#запускаем плейбук
ansible-playbook -i inventory/prod.yml site.yml
```