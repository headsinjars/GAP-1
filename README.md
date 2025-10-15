Этот проект разворачивает CMS Wordpress с базой данных MySQL и веб-сервером Nginx, а также систему мониторинга на базе Prometheus, Grafana и связанных экспортеров (Alertmanager, blackbox_exporter, mysqld_exporter). Все сервисы объединены в одну инфраструктуру с использованием Docker Compose.

Описание сервисов

wordpress — CMS Wordpress, использующая MySQL как базу данных.

mysql — база данных MySQL для Wordpress.

nginx — веб-сервер для проксирования запросов к Wordpress.

prometheus — система сбора и хранения метрик.

grafana — система визуализации данных, подключённая к Prometheus.

alertmanager — система управления алертами Prometheus.

blackbox_exporter — экспортер для проверки доступности внешних ресурсов.

mysqld_exporter — экспортер метрик MySQL для Prometheus.

Требования

Docker ≥ 20.x

Docker Compose ≥ 1.29.x

Файл .env с паролями и конфигурацией (пример ниже).

Быстрый старт

Склонируйте репозиторий:

git clone <URL_репозитория>
cd <папка_проекта>
Создайте файл .env на основе шаблона .env.example и заполните необходимые переменные:

MYSQL_ROOT_PASSWORD=your_mysql_root_password
MYSQL_PASSWORD=wordpress_password
WORDPRESS_DB_PASSWORD=wordpress_password

Запустите все сервисы:

docker-compose up -d
Откройте в браузере:

Wordpress: http://localhost (или указанный порт)

Grafana: http://localhost:3000 (логин/пароль по умолчанию admin/admin)

Prometheus: http://localhost:9090

Структура проекта

docker-compose.yml — описание всех контейнеров и сетей.

.env — файл с переменными окружения и паролями.

password.txt - файл для аутентификации на blackbox_exporter (задание со *)

Конфиги Prometheus, Grafana, Alertmanager, Exporters (blackbox и mysqld) находятся в соответствующих папках.

Мониторинг и алерты
Prometheus собирает метрики с Wordpress, MySQL и других экспортеров. Grafana отображает эти данные в виде дашбордов. Alertmanager обрабатывает уведомления об инцидентах.

Безопасность
Пароли хранятся в .env файле, не сохраняйте его в публичных репозиториях.

Рекомендуется использовать Docker secrets или внешние менеджеры секретов для продакшн окружений.

Полезные команды
Запуск контейнеров:

docker-compose up -d
Остановка контейнеров:

docker-compose down
Просмотр логов:

docker-compose logs -f

В проекте использованы следующие Grafana dashboards:
ID 7362 - MySQL
ID 7587 - Blackbox
ID 23942 - Windows exporter

Задания со *
Т.к., в ДЗ используется контейнеризация, то пробрасываются только определенные порты, а не все. Для доступа к blackbox используется базовая аутентификация (логин/пароль). 

basic_auth:
      username: user1
      password_file: /etc/prometheus/password.txt

Логин и файл с паролем указаны в файле конфигурации prometheus.yml в job_name "blackbox". Пароль необходимо добавить в файл password.txt и расположить в корне проекта. Далее, с помощью утилиты bcrypt вычислить хэш пароля и внести в файл ./exporters/blackbox/web-config.yml. Пример:

basic_auth_users:
  user1: "$2a$12$BH0.8gdMblHTK1AgVC9KWOOBan08iY.wXyZkKfCdQa1gNTZcGk3U6"

Для дополнительной безопасности и защиты передачи данных предусмотрена возможность использование TLS шифрования и аутентификации. Для этого необходимо выпустить сертификат публичного ЦС и добавить его открытую и закрытую части по пути ./exporters/blackbox/certs. Далее, раскомментировать строки в файле конфигурации docker-compose.yml (создание сервиса blackbox_exporter) для проброса томов в контейнер:

#    - ./exporters/blackbox/certs:/certs:ro

В каталоге screenshots проекта приведены примеры скриншотов работающей системы.

Автор и лицензия
Автор: headsinjars
Проект распространяется под лицензией MIT.