```Саморевью по чек-листу (слайды 32–35)```

```РЕСУРСЫ И URL```

Пути — существительные во множественном числе, kebab-case. Выполнено: /books, /readers, /loans.

Нет глагольных URL, кроме осознанных действий. Выполнено: единственное действие — POST /loans/{id}/return. В description операции объяснено, почему не DELETE (выдача не удаляется, а остаётся в истории) и не PATCH (сервер сам ставит returned_at и освобождает экземпляр книги).

id конкретного ресурса — в path, фильтры — в query. Выполнено: /books/{id}, /readers/{id}, /loans/{id}; фильтры author, available, email, status, reader_id, book_id передаются в query.

Версия API указана в servers. Выполнено: http://localhost:8000/v1.

```МЕТОДЫ И КОДЫ```

POST создания возвращает 201 + Location. Выполнено для POST /books, POST /readers, POST /loans.

DELETE возвращает 204 без тела. Выполнено для DELETE /books/{id} и DELETE /readers/{id}.

Пустой список — 200 + пустой data, не 404. Выполнено, указано в описании GET /books.

Бизнес-конфликт — 409. Выполнено: нет свободных экземпляров и лимит 5 активных выдач (POST /loans), активные выдачи при удалении книги или читателя, повторный возврат (POST /loans/{id}/return), занятый email (POST /readers), уменьшение total_copies ниже числа активных выдач (PATCH /books/{id}).

Ошибка валидации — 422. Выполнено: пустые строки, год меньше 1450, некорректный email, due_date не позже текущей даты, несуществующие book_id или reader_id в теле запроса. Несуществующий id в теле — это 422, а не 404, потому что адрес /loans существует, а неверно поле в JSON.

```СХЕМЫ```

У каждого request body есть отдельная схема. Выполнено: CreateBookRequest, UpdateBookRequest, CreateReaderRequest, UpdateReaderRequest, CreateLoanRequest.

У каждого успешного response есть схема. Выполнено: Book, Reader, Loan, для списков — обёртка data + Pagination.

required, enum, format, nullable и ограничения описаны явно. Выполнено: форматы email, date, date-time, uuid; ограничения minLength, minimum, maximum; enum для status, sort и error.code; returned_at — nullable (type: [string, "null"]).

Серверные поля не принимаются от клиента. Выполнено: id, available_copies, registered_at, loaned_at, returned_at помечены readOnly и отсутствуют в схемах запросов. available_copies вычисляется сервером из total_copies и активных выдач. Во всех схемах запросов additionalProperties: false, поэтому неизвестные поля дают 422.

Список и карточка не обязаны быть одной схемой. Выполнено: списки возвращают обёртку с data и pagination, а не голый массив.

```ОШИБКИ И СПИСКИ```

Все ошибки используют один ErrorResponse. Выполнено: все ответы 400, 404, 409, 422 ссылаются на одну схему через $ref. В error.code есть все коды из задания: INVALID_REQUEST, NOT_FOUND, VALIDATION_ERROR, CONFLICT, BUSINESS_RULE_VIOLATION, IDEMPOTENCY_MISMATCH.

Для 422 есть example с details. Выполнено: POST /books, ошибки по полям title (minLength) и published_year (minimum).

На коллекциях есть фильтры, сортировка и пагинация. Выполнено для GET /books, GET /readers, GET /loans. Сортировка через sort, минус в начале — по убыванию. Просроченные выдачи — GET /loans?status=overdue.

limit имеет default и maximum. Выполнено: default 20, maximum 100, minimum 1.

```ИДЕМПОТЕНТНОСТЬ```

Idempotency-Key на POST /loans. Выполнено: обязательный заголовок формата uuid. В description описаны полный replay (тот же ключ и тело — сохранённый ответ 201 с тем же id), другое тело с тем же ключом — 409 IDEMPOTENCY_MISMATCH, конкурентный повтор, пока первый запрос обрабатывается, — 409, срок хранения ключа — 24 часа.

```ДОКУМЕНТАЦИЯ```

Operations имеют summary, description, tags. Выполнено: теги Books, Readers, Loans; summary и operationId у всех 14 операций; description у операций с бизнес-правилами.

Есть examples успеха и типичной ошибки. Выполнено: 5 примеров — созданная выдача (201), ошибка валидации (422), книга не найдена (404), лимит выдач и IDEMPOTENCY_MISMATCH (409).

Общие схемы подключены через $ref, а не скопированы. Выполнено: схемы ресурсов, ErrorResponse, Pagination и параметры Id, Limit, Offset описаны один раз в components.

```ЧТО МОЖНО УЛУЧШИТЬ```

Ответ 401 при отсутствии токена описан в общем описании API, но не в каждой операции.

Пагинация через offset на больших и часто меняющихся списках может пропускать или дублировать записи; для них лучше cursor.

Нет защиты от одновременного изменения записи двумя клиентами (ETag и If-Match для PATCH).