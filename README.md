##### Этот проект разворачивает CMS Wordpress с базой данных MySQL и веб-сервером Nginx, а также систему мониторинга на базе Prometheus, Grafana и связанных экспортеров (Alertmanager, blackbox_exporter, mysqld_exporter). Все сервисы объединены в одну инфраструктуру с использованием Docker Compose.

##### Описание сервисов

- __wordpress__ — CMS Wordpress, использующая MySQL как базу данных.

- __mysql__ — база данных MySQL для Wordpress.

- __nginx__ — веб-сервер для проксирования запросов к Wordpress.

* __prometheus__ — система сбора и хранения метрик.

* __grafana__ — система визуализации данных, подключённая к Prometheus.

* __alertmanager__ — система управления алертами Prometheus.

* __blackbox_exporter__ — экспортер для проверки доступности внешних ресурсов.

* __mysqld_exporter__ — экспортер метрик MySQL для Prometheus.

##### Требования

1. Docker ≥ 20.x

2. Docker Compose ≥ 1.29.x

3. Файл .env с паролями и конфигурацией (пример ниже).

##### Быстрый старт

1. Склонируйте репозиторий:

`git clone https://github.com/headsinjars/GAP-1.git`
`cd <папка_проекта>`

2. Создайте файл .env на основе шаблона .env.example и заполните необходимые переменные:

MYSQL_ROOT_PASSWORD=your_mysql_root_password
MYSQL_PASSWORD=wordpress_password
WORDPRESS_DB_PASSWORD=wordpress_password```

3. Запустите все сервисы:

`docker-compose up -d`

4. Откройте в браузере:

__Wordpress:__ [http://localhost](http://localhost/) (или указанный порт)

__Grafana:__ [http://localhost:3000](http://localhost:3000/) (логин/пароль по умолчанию admin/admin)

__Prometheus:__ [http://localhost:9090](http://localhost:9090/)

##### Структура проекта

__docker-compose.yml__ — описание всех контейнеров и сетей.

__.env__ — файл с переменными окружения и паролями.

__password.txt__ - файл для аутентификации на blackbox_exporter (задание со *)

Конфиги Prometheus, Grafana, Alertmanager, Exporters (blackbox и mysqld) находятся в соответствующих папках.
Мониторинг и алерты Prometheus собирает метрики с Wordpress, MySQL и других экспортеров. Grafana отображает эти данные в виде дашбордов. Alertmanager обрабатывает уведомления об инцидентах.
Безопасность Пароли хранятся в .env файле, не сохраняйте его в публичных репозиториях.
Рекомендуется использовать Docker secrets или внешние менеджеры секретов для продакшн окружений.

##### Полезные команды 
Запуск контейнеров:
`docker-compose up -d`

Остановка контейнеров:
`docker-compose down`

Просмотр логов:
`docker-compose logs -f`

В проекте использованы следующие Grafana dashboards:
* ID 7362 - MySQL
* ID 7587 - Blackbox
* ID 23942 - Windows exporter

Задания со * Т.к., в ДЗ используется контейнеризация, то пробрасываются только определенные порты, а не все. Для доступа к blackbox используется базовая аутентификация (логин/пароль).


```
basic_auth:
	username: user1
	password_file: /etc/prometheus/password.txt
```

Логин и файл с паролем указаны в файле конфигурации _prometheus.yml_ в job_name "blackbox". Пароль необходимо добавить в файл _password.txt_ и расположить в корне проекта. Далее, с помощью утилиты bcrypt вычислить хэш пароля и внести в файл ./exporters/blackbox/web-config.yml. Пример:
```
basic_auth_users: user1: "$2a$12$BH0.8gdMblHTK1AgVC9KWOOBan08iY.wXyZkKfCdQa1gNTZcGk3U6"
```
Для дополнительной безопасности и защиты передачи данных предусмотрена возможность использование TLS шифрования и аутентификации. Для этого необходимо выпустить сертификат публичного ЦС и добавить его открытую и закрытую части по пути ./exporters/blackbox/certs. Далее, раскомментировать строки в файле конфигурации docker-compose.yml (создание сервиса blackbox_exporter) для проброса томов в контейнер:

`- ./exporters/blackbox/certs:/certs:ro

В каталоге screenshots проекта приведены примеры скриншотов работающей системы.

### ДЗ №2

1. Добавлен сервис для установки долгосрочного хранилища метрик - VictoriaMetrics

```
victoriametrics:
	image: victoriametrics/victoria-metrics:v1.127.0
	container_name: victoriametrics
	volumes:
		- ./victoriametrics/data:/victoria-metrics-data
	ports:
		- '8428:8428'
	command:
		- '--retentionPeriod=2w'
	depends_on: - prometheus
```

2. Установлен период хранения метрик - 2 недели:

`command: - '--retentionPeriod=2w'

3. В файле конфигурации Prometheus добавлен job для чтения метрик из хранилища:

```
job_name: "victoriametrics" static_configs:
    - targets: ['victoriametrics:8428']
```

4. В файле конфигурации Prometheus добавлена интеграция с хранилищем VictoriaMetrics:

```
remote_write:
	- url: [http://victoriametrics:8428/api/v1/write]
```

5. В файле конфигурации Prometheus в секции global добавлен label "site: prod" для записи в хранилище VictoriaMetrics:
```
global:
	scrape_interval: 5s
	external_labels:
		site: prod
```


### ДЗ №3

[](https://github.com/headsinjars/GAP-1#%D0%B4%D0%B7-3)

1. В папке alertmanager добавлен файл конфигурации alertmanager.yml Настроены ресиверы, маршруты в зависимости от значения severity (отправка в телеграм в разные чаты)
    
2. В папке prometheus добавлен файл с правилами алертинга и установкой меток severity
    
3. В конфиге prometheus.yml добавлена ссылка на файл с правилами:
    

```
rule_files:
  - "rules.yml"
```

4. docker-compose.yml - добавлен том для сервиса alertmanager с файлом конфигурации для alertmanager'а

```
    volumes:
      - ./alertmanager/alertmanager.yml:/etc/alertmanager/alertmanager.yml
```

### ДЗ №4

1. Созданы 2 папки __infra__ и __app__
2. В папке __infra__ создан dashboard infra status. Созданы 2 переменные: $myjob и $myhostname для возможности выбора инстанса. Дэшборд разделен на несколько секций (панелей). Сверху дэшборда представлена базовая статистика по инстансам инфраструктуры, далее, в раскрывающихся панелях представлена более подробная статистика по элементам инфраструктуры (статистика загрузки ЦПУ по ядрам, сетевая статистика по количеству установленных сессий, сетевая статистика по входящему/исходящему трафику, дисковая статистика по очередям на чтение и запись)
3. В папке __app__ создан dashboard по статистике метрик приложений и базы данных. Представлены графики статистики по времени http запросов, по кодам полученных ответов, статусы доступности фронтенда и базы данных.
4. Создан алерт средствами Grafana. В качестве примера взято значение свободного дискового пространства (алерт в статусе firing приведен на скриншоте). Период опроса и срабатывания алерта в 10 сек взят для примера и оперативной отладки.
5. Скриншоты дэшбордов и алерта приведены в папке GAP-2.


Автор и лицензия Автор: headsinjars
Проект распространяется под лицензией MIT.