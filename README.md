# B9121-02.03.03-Streltsov-Grats-Machkasov
Система контроля знаний студентов.  
ФИО разработчиков: Стрельцов Алексей Романович, Грац Валентин Сергеевич, Мачкасов Дмитрий Анатольевич.  
ФИО научного руководителя: Остроухова Светлана Николаевна.  

Необходимо создать 3 проекта visual studio. Необходимые для работы репозитории:  
1) Интерфейс системы (В visual studio сделать клонирование репозитория по ссылке "https://github.com/1DimBeam1/SKAT.git", "Вид" -> "Репозитории Git" -> выбрать ветку merge - "pre-demo fix" коммит вытянуть) https://github.com/1DimBeam1/SKAT/tree/merge  
2) Модуль выполнения кода (В visual studio сделать клонирование репозитория по ссылке "https://github.com/Lexaky/InterpretatorService.git", "Вид" -> "Репозитории Git" -> выбрать ветку workvariantextended - "added new code checks" коммит вытянуть) https://github.com/Lexaky/InterpretatorService/tree/workvariantextended  
3) Модуль оценки (В visual studio сделать клонирование репозитория по ссылке "https://github.com/Lexaky/MarkSubsystem.git", "Вид" -> "Репозитории Git" -> выбрать ветку master - "fixed bug with clear db" коммит вытянуть НЕ ПОСЛЕДНИЙ КОММИТ !!! ОБРАТИЛ ВНИМАНИЕ) https://github.com/Lexaky/MarkSubsystem  

Сборку необходимо выбрать через Container (docker), если по-умолчанию стоял https.  

Далее необходимо создать 2 базы данных для работы. Мы собирали их через docker-compose пустыми, а потом выполняли миграции из проекта 2 (InterpretatorService) и проекта 3 (MarkSubsystem).  
Для первой бд: необходимо создать папку postgresql, внутри создать файл docker-compose.yml с содержимым:  
version: '3.8'  

services:  
  postgres:  
    image: postgres:15  
    container_name: mypostgresql  
    environment:  
      POSTGRES_USER: postgres  
      POSTGRES_PASSWORD: 12345  
      POSTGRES_DB: usersandalgos  
    ports:  
      - "5433:5432"  
    volumes:  
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql  
      - pgdata:/var/lib/postgresql/data  
    restart: unless-stopped  

volumes:  
  pgdata:  


Для второй бд: необходимо создать папку postgresqlusers, внутри создать файл docker-compose.yml с содержимым:  
version: '3.8'  
  
services:  
  postgres:  
    image: postgres:15  
    container_name: postgres-users  
    environment:  
      POSTGRES_USER: postgres  
      POSTGRES_PASSWORD: 12345  
      POSTGRES_DB: users  
    ports:  
      - "5435:5432"  
    volumes:  
      - users_pgdata:/var/lib/postgresql/data  
    restart: unless-stopped  

volumes:  
  users_pgdata:  

В обоих папках необходимо зайти в консоли и выполнить "docker-compose up --build" для деплоя двух бд.  

На данном этапе у вас должно появиться в docker-desktop (или через docker ps) 4 следующих контейнера:  
postgresqlusers  
postgresql  
InterpretatorService  
MarkSubsystem  

Далее необходимо настроить сеть докера. В консоли ввести:  
docker network connect postgresqlusers_default MarkSubsystem  
docker network connect postgresql_default InterpretatorService  
docker network connect postgresqlusers_default InterpretatorService  

Далее необходимо выполнить миграции в базы данных из проектов MarkSubsystem и InterpretatorService.  
Для InterpretatorService:  
в файле appsettings.json строка подключения должна стать:  
"TestsDb": "Host=localhost;Port=5433;Database=usersandalgos;Username=postgres;Password=12345"  
Сохранить appsettings и выполнить в консоли диспетчера пакетов: dotnet ef database update (если нету инструментов ef, то надо их в этой же консоли скачать через dotnet tool install --global dotnet-ef).  
Здесь должна выполниться сборка проекта и миграция в бд postgresql.  
Потом необходимо вернуть строку подключения на:  
"TestsDb": "Host=postgres;Port=5432;Database=usersandalgos;Username=postgres;Password=12345"  
и сохранить appsettings.  

Для MarkSubsystem всё сделать то же самое, но строки подключения:  
Изменить на 1) "UsersDb": "Host=localhost;Port=5435;Database=users;Username=postgres;Password=12345"  
Вернуть на 2) "UsersDb": "Host=postgres-users;Port=5432;Database=users;Username=postgres;Password=12345"  

Для проекта SKAT необходимо настроить в appsetings.json строки подключения к сервисам, конкретно нужно назначить правильные порты, такиеже как в docker desktop, для http.
ApiSettings для подключения к интерпритатору.
EvaluationService для подключения к сервису оценки.

Очень важно очистить директорию /code_files/ от файлов в InterpretatorSerivce. Должны остаться только: "debug.txt", "logs.txt", папка TestData.  

Можно запускать все 3 проекта.  

Регистрация и создание учебных групп в системе не реализованы (и это верно), поэтому необходимо вручную заполнить информацию о пользователях. Для тестирования программного средства вам будет достаточно   пользователя с ролью "Обучающийся", и пользователя с ролью "Преподаватель".  
В таблицу Users вставить записи о людях:  
INSERT INTO Users (user_id, group_id, role, name, login, password, icon_path) VALUES  
(1, 1, 'Обучающийся', 'Сергеев Сергей Сергеевич', 'sergeev.ss', '12345', ' '),  
(2, 0, 'Преподаватель', 'Иванов Иван Иванович', 'ivanov.ii', '12345', ' ');  

В таблицу Groups вставить записи о группах:  
INSERT INTO Groups (group_id, name) VALUES  
(0, 'Преподаватели'),  
(1, 'Б9121-02.03.03тп'),  
(2, 'Б9122-01.01.01'),  
(3, 'М1234-05.06.07');  

Далее можно заходить в систему через преподавателя и наполнять базу данных алгоритмами через "Редактор тестовых заданий". В папке TestData содержатся папки с алгоритмами. Изображения можно изначально не вставлять, но если есть картинки псевдокодов, то можно вставить при добавлении алгоритма.  

Чтобы пройти сессию тестирования нужно:  
1) Создать алгоритм  
2) Создать тестовое задание на основе алгоритма  
3) Создать сессию тестирования и включить в неё тестовое задание алгоритма  
4) Начать сессию тестирования  
  
