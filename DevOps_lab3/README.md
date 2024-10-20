# Лабораторная работа 2

В этой лабе создадим "хорошие" и "плохие" CI/CD файлы используя Gitlab, опишем плохие практики, пофиксим и расскажем что и почему.

## Пример "плохого" CI/CD

```
stages:
  - build
  - test
  - deploy

build_job:
  script:
    - apt-get update
    - apt-get install -y some-package
    - echo "Building the project"
    - make
  only:
    - master

test_job:
  script:
    - echo "Running tests"
    - exit 1
  only:
    - master

deploy_job:
  script:
    - echo "Deploying to production"
    - scp ./build lol@production:/var/www/project
  only:
    - master

```

### А минусы где?
1. **Установка пакетов внутри пайплайна вместо подготовленного образа**: 
  Для установки инструментов мы используем `apt-get install`, что только замедляет выполнение.

2. **Использование сборки без кэширования**:
  Мы использовали `make` для сборки без проверки кэшированных результатов, что приводит к повторной полной сборке при каждом запуске.

3. **Запуск пайплайна только для `master`**:
  Все задания выполняются только для ветки `master`, это ограничит возможность проверить код на других ветках, следовательно ошибки будет сложнее обнаружить.

4. **Заданные ошибки в тесте**:
  Тестовое задание завершает работу сошибкой `exit 1`, то есть тест в любом случае неудачный.

5. **Использование устаревшей команды для деплоя**:
  Для деплоя используется `scp`, это вероятно замедлит(при передаче большого кол-ва файлов) и понизит безопасность выполнения.

## Пример "хорошего"(относительно) CI/CD

```
image: node:16

stages:
  - build
  - test
  - deploy

variables:
  PACKAGE_VERSION: "1.0.0"

build_job:
  script:
    - npm install
    - npm run build
  only:
    - main
  artifacts:
    paths:
      - build/ 
  cache:
    paths:
      - node_modules


test_job:
  script:
    - npm run test
  only:
    - merge_requests

deploy_job:
  script:
    - echo "Deploying to production"
    - rsync -avz ./build lol@production:/var/www/project
  only:
    - main
  environment:
    name: production
    url: https://blablabla.com

```
### Разница

1. Здесь же мы используем контейнерный образ `node:16`, в котором все необходимые зависимости уже есть.

2. Сейчас мы использовали кеширование зависимостей(директива `cache`), благодаря этому можно повторно пользовать скачанные пакеты.

3. В хорошем примере сборка и деплой выполнены для ветки `main`, а тестики запускаются для `merge request`.

4. Теперь тесты запускаются через `npm run test` и мы получаем реальные результаты.

5. Заменили на `rsync`, он синхронизирует только изменённые файлы.

##  Вывод

  Васю жалко.
