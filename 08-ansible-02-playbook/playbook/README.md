### Что делает playbook?
 Данный playbook устанавливает на 3 сервера(redhat based os x86_64) следующие пакеты:
* clickhouse
* lighthouse и nginx 
* vector

### Параметры playbook-а:
 Параметры задаются в папке group_vars/имя_сервиса/vars.yml
  
## Параметры для clickhouse:
* clickhouse_version - в переменную записываем нужную версию
* clickhouse_packages - список необходимых пакетов кликхауса для установки

## Параметры для vector:
* vector_version - в переменную записываем нужную версию

## Параметры для lighthouse:
* lighthouse_vcs - путь к исходникам сервиса
* lighthouse_location_dir - путь установки сервиса

## какие у него есть теги:
* нет ниодного тега. 