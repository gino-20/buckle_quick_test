# Тестовое задание

## *Практическая часть*

 - Подготовлены образы Nginx, PHP с установленным composer и тестовым приложением Laravel (Докерфайлы находятся в соответсвующих папках репозитория).
 - Деплой образов на сервер происходит из публичного реджистри Докер с помощью Ansible. Для запуска роли необходимо выполнить команду "ansible-playbook playbook.yml" в папке ansible репозитория.
 - Также Ansible выполняет добавление пользователя и настройку правил iptables.

## *Теоритическая часть*

 1. На мой взгляд, архитектура получилась достаточно отказоустойчивой: все сервисы размещены в отдельных root-less контейнерах, взаимодействие происходит через внутреннюю сеть Докер. Сервисы запущены с restart_policy=always. Однако не использованы механизмы healthchecks, что может привести к ситуации, когда сервис внутри контейнера будет неработоспособен, но внешне живой, поэтому не будет перезапущен. Также можно использовать Nginx в качестве round-robin балансировщика и запустить несколько экземпляров сервиса PHP. В том, что касается самой системы, я бы запретил доступ по SSH по паролю и навесил fail2ban.
2. Я бы выделил мастер-мастер и мастер-слейв. В первом случае возможна запись в несколько мастеров, которые синхронизируются между собой, однако такой способ требует очень аккуратного подхода в реализации логики сервисов, так как возможны коллизии, когда одни и те же данные меняются на разных мастерах. Второй способ более более простой в реализации и позволяет добиться существенного прироста скорости в чтении данных разными сервисами, в том числе за счет расположения слейвов по принципу географицеской доступности. Я бы включил механизм репликации следующим образом (допустим, что на данный момент существует одна база MySQL): на мастере нужно создать учетную запись пользователя, который будет использоваться при репликации, назначить уникальный номер сервера (server_id), включить двоичный журнал (по нему слейвы будут отслеживать текущее состояние строк) и сделать бэкап базы в файл; на слейве (или слейвах) нужно также указать идентификатор сервера, расположение журнала ретрансляции и собственного двоичного журнала, а также активировать режим "только чтение", затем загрузить дамп базы мастера из файла, указать адрес мастера и координаты по двоичному журналу в момент создания дампа, а затем запустить слейв и проверить его статус. Здесь необходимо отметить, что для работающей базы необходимо будет выполнить её перезагрузку после активации двоичного журнала, а также блокировку на момент создания дампа.
3. Преимущества и недостатки разных Ingress имеет смысл рассматривать только в разрезе конкретных задач и набора сервисов проекта. В проде я работал только с ingress-nginx. Среди его преимуществ применительно к тем сервисам, которые были развернуты, я бы отметил быстрый старт. Среди недостатков, наверное, сложность кастомных настроек под разные сервисы, например, вебсокеты. Если бы я сейчас приступил к разворачиванию масштабного проекта с нуля, я бы выбрал traefik. С этим балансировщиком сталкивался не в k8s, а в рамках деплоя микросервисов через Docker. Мне очень понравилось, как там реализовано динамическое получение SSL-сертификатов и функции балансировки.


