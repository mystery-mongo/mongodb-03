# Домашняя работа по теме "Введение в Docker. MongoDB в docker & docker-compose"

## Создание виртуальной машины в YandexCloud
Была создана виртуальная машина с ОС Ubuntu через веб-интерфейс. Ключ для ssh добавлен на этапе создания.
![YC](./01.png?raw=true)

## Установка и настройка Docker

### Обновим список пакетов и установим пакеты для импорта ключей
```
sudo apt-get update && sudo apt-get install ca-certificates curl gnupg && sudo install -m 0755 -d /etc/apt/keyrings && sudo install -m 0755 -d /etc/apt/keyrings
```
### Импорт ключей
```
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg && sudo chmod a+r /etc/apt/keyrings/docker.gpg
```
### Добавление сведений о репозитории с docker в sources.list
```
echo \
  "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```
### Обновление списка пакетов
```
sudo apt-get update
```
### Установка docker
```
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```
### Проверка docker
```
sudo docker run hello-world
```
![Docker-Check](./02.png?raw=true)

## Установка MongoDB через docker
### Создадим директорию для хранения баз данных в каталоге пользователя
```
mkdir ~/mongo-test && mkdir ~/mongo-test/db1
```
### Загрузка образа с mongodb
```
sudo docker pull mongo:7.0.3
```
### Создание yml-файл для docker-compose с сервером mongo `docker-compose.mongo.yml`. Тут указывается volume с ссылкой на ранее созданную папку:
```
version: '3.9'
services:
  mongodb:
    image: mongo:7.0.3
    container_name: mongodb-server
    volumes:
      - mongodb:/data/db
    ports:
      - 48658:27017
volumes:
  mongodb:
    driver: local
    driver_opts:
      type: none
      device: '/home/mystery/mongo-test/db1/'
      o: bind
```
### Запуск контейнера в качестве демона через docker compose, и пробуем "провалиться" в него через клманду `mongosh`
```
sudo docker compose -f docker-compose.mongo.yml up -d
sudo docker exec -it mongodb-server mongosh
```
![Docker-Mongosh](./03.png?raw=true)

## Создание контейнера с клиентом

### Получить образ и запустить контейнер
```
sudo docker pull rtsp/mongosh
sudo docker run -d --name mongosh rtsp/mongosh
```
### Выполнить подключение через новый контейнер к созданному вначале через mongosh
```
sudo docker exec -it mongosh mongosh -- mongodb://172.17.0.1:48658
```
![Docker-Mongosh](./04.png?raw=true)

![Docker-Mongosh](./05.png?raw=true)

## Проверка внешнего подключения через MongoDB Compass

![Docker-Compass](./06.png?raw=true)

## Удалить все контейнеры, создать заново и проверить наличие данных:

```
sudo docker stop mongosh && sudo docker stop mongodb-server && sudo docker remove mongosh && sudo docker remove mongodb-server
sudo docker compose -f docker-compose.mongo.yml up -d && sudo docker run -d --name mongosh rtsp/mongosh && sudo docker exec -it mongosh mongosh -- mongodb://172.17.0.1:48658
```
![Docker-New](./07.png?raw=true)
