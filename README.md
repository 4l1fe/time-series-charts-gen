### ЗАПУСК ПРОЕКТА
В корне проекта запустить команду `docker-compose up -d --build`. Необходимо дождаться создания таблицы БД, 
которая будет запущена через 4 секунды(по умолчанию) после успешного запуска зависящих контейнеров.
Случай получения ошибки 500 в веб интерфейсе означает, что контейнер с БД не успел проинициализироваться и 
попытка создания табллицы была преждевременной, для запуска повторно необходимо запустить команду `docker-compose up admin_db_gen`
### ОПИСАНИЕ СЕРВИСОВ
#### Админка
Сервис является составным, т.е. с единого образ запускается три контейнера:
1. Веб интерфейс. При обновлении графиков происходит проверка на задачи в обработке для них. В случае существования обновление по найденным игнорируется. 
2. Изначальное создание таблицы БД.
3. Селери воркер, отвечающий за последовательность генерации данных, графика и обновления БД. При получении задачи получает данные для графика, передает в след сервис для генерации изображения, в случае успеха обновляет его в БД и удаляет ошибку, иначе обновляет ошибку.
#### Генерация данных
Сервис генерирует и возвращает массив пар данных для графика.
API метод:
```
   POST /generate
   Request(application/json):
       - function(string, required)
       - inteval(int, required)
       - step(int, required)
   Response(application/json)
       - array[timestmap, func_result]
```
#### Генерация изображения графика
Доп логика отсутствует.
### ПРОВЕРКА
Все настраиваемые параметры описаны в файле `.env` в корне проекта(кроме `RABBIT_PORT`).
Все три сервиса слушают публичные порты `ADMIN_PUB_PORT, DATA_GEN_PUB_PORT, HIGHCHARTS_PUB_PORT`. Соответственно при необходимости можно обратиться к ним напрямую.
Админку надо проверять через веб интерфейс.
#### Пример проверки генерации данных
```
$ curl -H "Content-Type: application/json" -X POST -d '{"function":"t/2","interval":4, "step":1}' http://127.0.0.1:8081/generate
[
  [
    1522864580.89806, 
    761432290.449028
  ], 
```
