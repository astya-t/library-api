Library API

Контракт REST API сервиса «Библиотека» в формате OpenAPI 3.1. Домашнее задание к уроку 3 «Проектирование API-контрактов».

Файлы. openapi.yaml — контракт API, STYLE.md — правила дизайна API, REVIEW.md — саморевью по чек-листу, README.md — этот файл.

Ресурсы. Books: GET /books, POST /books, GET /books/{id}, PATCH /books/{id}, DELETE /books/{id}. Readers: GET /readers, POST /readers, GET /readers/{id}, PATCH /readers/{id}, DELETE /readers/{id}. Loans: GET /loans, POST /loans, GET /loans/{id}, POST /loans/{id}/return. Базовый адрес: http://localhost:8000/v1.

Основные решения. Выдача книги (POST /loans) требует заголовок Idempotency-Key, чтобы повтор запроса не создавал вторую выдачу. Выдача возможна только при available_copies > 0, у читателя может быть не больше 5 активных выдач, due_date должна быть позже текущей даты, книгу или читателя с активными выдачами удалить нельзя. Возврат сделан через POST /loans/{id}/return, а не DELETE, чтобы выдача осталась в истории. Просроченные выдачи можно получить запросом GET /loans?status=overdue. Все ошибки возвращаются в едином формате ErrorResponse.

Дополнительное задание. Добавлена авторизация Bearer JWT через components/securitySchemes. Все запросы передают заголовок Authorization: Bearer <токен>.

Как посмотреть и проверить. В VS Code установить расширение OpenAPI (Swagger) Editor, открыть openapi.yaml и нажать на иконку превью. Также можно вставить содержимое файла в Swagger Editor (https://editor.swagger.io/) или проверить его из терминала командой npx @redocly/cli lint openapi.yaml (нужен Node.js).