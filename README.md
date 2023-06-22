# standart-blog
[Лабораторный проект по видео Archakov Blog: Полный Full Stack курс Node.js + React.js для начинающих за 4 часа (MongoDB, Express)](https://www.youtube.com/watch?v=GQ_pTmcXNrQ).

Задача – сайт-блог.

Модель: а) статьи, б) пользователи, г) дополнительные данные для статьи: комментарии пользователей, тэги.

Backend и Frontend разделены на отдельные приложения.

Содержание:
- [Backend](#backend-section)
  - [Платформа и зависимости](#backend-section-1)
  - [Структура](#backend-section-2)
  - [Корневой файл](#backend-section-3)
  - [Контроллеры](#backend-section-4)
  - [Модель](#backend-section-5)
  - [Авторизация](#backend-section-6)
  - [Валидация данных](#backend-section-7)
- [Frontend](#frontend-section)

## Backend <a name="backend-section"></a>
### Платформа и зависимости <a name="backend-section-1"></a>
- Среда исполнения – node.js
- Хостинг и обработка HTTP сообщений – express
- БД –mongoose (MongoDB)
- Аутентификация и Авторизация – jsonwebtoken.

- express validator – валидация модели, читать ниже
- bcrypt – для кэширования пароля
- cors – для CORS валидации сайта
- multer – создание хранилища для ресурсов (картинок)
- nodemon – dev режим под node.js, для автоматического перезапуска

### Структура <a name="backend-section-2"></a>
- Базовые папки и файлы:
  - папка node_modules
  - файл package.json – зависимости, параметры, команды/скрипты
  - файл package-lock.json – зависимости
  - файл index.js – основной файл проекта
- Добавляем папки: model – модель
  - controllers – для обработки HTTP
  - PostController.js
  - UserController.js
  - CommentController.js - домашнее задание от автора видео.
  - index.js – объединяет экспорт файлов выше
- Добавляем папку utils – дополнительный функционал
  - checkAuth.js – авторизация, проверка JWT ключа
  - handleValidationErrors.js – код пакета express-validator
  - index.js – объединяет экспорт файлов выше
- Добавляем папку uploads – ресурсы, типа картинок
- Добавляем файл: validations.js

Работа начиналась с базового файла index.js и папки и файлов модели.
Остальные папки и файлы подключались по мере подключения функционала и рефакторинга кода (файлы экспорта модулей index.js появились последними). 

### Корневой файл [index.js](https://github.com/barbado-vl/standart-blog/blob/master/standart_website_backend/index.js) и Основной функционал <a name="backend-section-3"></a>
const app = express(); -- подключение модуля express, который будет внедрять в себя все остальное.
Далее по зонам. Порядок зон важен. Зоны:
  - подключение хранилища (подключаем multer)
  - подключение дополнительных объектов (express.json(), папки хранилища uploads, cors)
  - подключение MongoDB (используем mongoose, которой передаем строку подключения, сама БД создается отдельно, читать ниже)
  - обработчики действий (CRUD запросы)
    > app.post('/posts/:id', checkAuth, postCreateValidation, handleValidationErrors, PostController.create);
  - сервер (задаем порт для localhost)

В обработчик действия передаем параметры:
  - (обязательно) адрес 
  - функция проверки авторизации (checkAuth из модуля checkAuth.js)
  - функция валидации (одна из функций модуля validation.js)
  - (обязательно) функция контроллера (можно стрелочной, можно отдельно, можно из модуля)

### [Контроллеры](https://github.com/barbado-vl/standart-blog/tree/master/standart_website_backend/controllers) (функции CRUD) <a name="backend-section-4"></a>
Их вынесли в отдельный файл, модуль, и отдельную папку.
Функция задается с 2 параметрами req (запрос) и res (ответ), app передаст в них нужные объекты.
Внутри try catch.
Внутри обращение к модели с целью получить, добавить или изменить данные. Важно, функционал работы с моделью определяется внутри файла модели, где подключаем библиотеку mongoose, которая дает методы работы с схемами БД. Эти методы тут вызываются.
Запрос дает данные по которым обращаемся к модели.
Ответ возвращает ответ в виде json.

В контроллере User, в методах register и login генерируем JWT token, который добавляем к Ответу.

### [Модель](https://github.com/barbado-vl/standart-blog/tree/master/standart_website_backend/models) <a name="backend-section-5"></a>
Создание модели с помощью Схемы из библиотеки mongoose, которая прикрутит схему к БД.
Так же mongoose даст методы работы со схемой, которые потребуются для обращения к данным из Контроллера.

### Авторизация ([checkAuth.js](https://github.com/barbado-vl/standart-blog/blob/master/standart_website_backend/utils/checkAuth.js)) <a name="backend-section-6"></a>
Метод импортируется в базовый файл index.js и вызывается в обработчиках действия CRUD модели.

Сам метод checkAut реализован в модуле checkAuth.js, папка /utils.

В метод, как и в метод контролера, передаются параметры res (запрос) и req (ответ), плюс 3-ий параметр, функция callback next().

Из запроса достается JWT token.

Если проверка проходит, то вызывается функция next().

Если проверка не проходится обработка блокируется, т.к. next() не вызывается, идем в catch где пишем Нет доступа.

### Валидация данных. <a name="backend-section-7"></a>
Для реализация валидации используем библиотеку express validator.
- Создаем требования валидации, файл [validation.js](https://github.com/barbado-vl/standart-blog/blob/master/standart_website_backend/validations.js) в корневой папке. Импортируем функцию body из express-validator. Прописываем массивы из функций, в которых указываем параметры модели и требования к ним. Массив для каждой модели объекта, по функции для каждого параметра объекта модели.
- Создаем обработчик валидации, файл [handleValidationErors.js](https://github.com/barbado-vl/standart-blog/blob/master/standart_website_backend/utils/handleValidationErrors.js) в папке utils с помощью функции validationResult, которую импортируем из express-validator. 

## Frontend <a name="frontend-section"></a>

