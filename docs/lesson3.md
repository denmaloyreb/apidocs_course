# Урок 3. Остальные методы

В этом уроке формируются описания для остальных методов для реализации необходимых действий.

## Формирование адресов конечных точек (методов)

В начале курса мы условились, что в проектируемом API будут реализован определенный [набор действий](main).

Исходя из этого давайте сформируем адреса конечных точек API (методы) и выберем для них HTTP-глаголы:

|Выполняемое действие|Глагол и конечная точка (метод)|
|--------------------|-------------------------------|
|Получение списка технических писателей|`GET /techwriters`|
|Получение списка скиллов|`GET /skills`|
|Получение информации о техническом писателе по идентификатору|`GET /techwriters/{techwriterId}`|
|Получение информации о навыке|`GET /skills/{skillId}`|
|Добавление нового технического писателя|`POST /techwriters/`|
|Редактирование сведений о техническом писателе|`PATCH /techwriters/{techwriterId}`|
|Удаление технического писателя|`DELETE /techwriters/{techwriterId}`|
|Добавление нового навыка|`POST /skills`|
|Редактирование навыка|`PATCH /skills/{skillId}`|
|Удаление навыка|`DELETE /skills/{skillId}`|

## Итоговая спецификация

После описания этих методов в формате OpenAPI (пример — в [Уроке 2](lesson2)) итоговая спецификация получается такой (вы можете скопировать ее в любой из указанных в [Уроке 2](lesson2) инструментов и посмотреть, как это рендерится):

```yaml
openapi: 3.0.0
info:
  description: |
    <b>API для работы с базой данных о технических писателях и их навыками.</b></br>
    API позволяет получать:<ul>
    <li>список технических писателей,</li>
    <li>список навыков каждого из них,</li>
    <li>информацию о конкретном техническом писателе по идентификатору.</li></ul>
    Также здесь можно получать информацию о навыках, добавлять новых технических писателей, редактировать сведения об имеющихся в базе техписах.</br>
    В версии API 1.0.0 реализовано добавление нового навыка, редактирование навыка, удаление навыка.
  version: 1.0.0
  title: База технических писателей
tags:
  - name: techwriters
    description: Методы для работы с техническими писателями
  - name: skills
    description: Методы для работы с навыками
paths:
  /techwriters:
    get:
      tags:
        - techwriters
      summary: Получение списка технических писателей
      description: Метод возвращает список технических писателей из базы.
      operationId: getAllТws
      parameters:
        - name: limit
          in: query
          description: |
            Количество технических писателей в выдаче (для пагинации).
            Значение по умолчанию `20`
          required: false
          schema:
            type: integer
            format: int64
            example: 10
            default: 20
      responses:
        '200':
          description: ОК
          content:
            application/json:
              schema:
                type: object
                properties:
                    items:
                      type: array
                      description: Список технических писателей
                      items:
                        $ref: '#/components/schemas/Techwriter'
                    limit:
                      type: integer
                      example: 10
                required:
                  - items
                  - limit
        '400':
          description: Неверный формат limit
        '404':
          description: В базе не найдено технических писателей
    post:
      tags:
        - techwriters
      summary: Добавление технического писателя в базу
      description: Метод добавляет нового технического писателя в базу
      operationId: addТw
      requestBody:
        description: Данные технического писателя
        required: true
        content:
          'application/json':
            schema:
              $ref: "#/components/schemas/NewTechwriter"
      responses:
        '201':
          description: Created
          content:
            application/json:
              schema: 
                $ref: '#/components/schemas/Techwriter'
        '405':
          description: Неверный формат ввода
  /techwriters/{techwriterId}:
    get:
      tags:
        - techwriters
      summary: Получение информации о техническом писателе по идентификатору
      description: Метод возвращает информацию о техническом писателе по его идентификатору
      operationId: findТwByID
      parameters:
        - name: techwriterId
          in: path
          description: Идентификатор технического писателя, о котором нужно получить информацию
          required: true
          schema:
            type: integer
            format: int64
            example: 123
      responses:
        '200':
          description: ОК
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Techwriter'
        '400':
          description: Задан неверный id
        '404':
          description: Технический писатель не найден
    patch:
      tags:
        - techwriters
      summary: Редактирование сведений о техническом писателе
      description: |
        Метод изменяет сведения о техническом писателе. Метод изменяет сведения частично: можно передать только имя, фамилию, отчество, опыт, номер телефона, адрес электронной почты или идентификатор навыка.
        Обязательно должен быть передан хотя бы один параметр
      operationId: updateTwInfo
      parameters:
        - name: techwriterId
          in: path
          description: Идентификатор технического писателя, сведения о которым нужно изменить
          required: true
          schema:
            type: integer
            format: int64
            example: 25
      requestBody:
        description: Изменяемые сведения о техническом писателе
        required: true
        content:
          'application/json':
            schema:
              $ref: "#/components/schemas/UpdTechwriter"
      responses:
        '200':
          description: ОК
          content:
            application/json:
              schema: 
                $ref: '#/components/schemas/Techwriter'
        '405':
          description: Неверный формат ввода
        '404':
          description: Технического писателя с таким идентификатором нет в базе
    delete:
      tags:
        - techwriters
      summary: Удаление технического писателя из базы
      description: Метод для удаления технического писателя из базы по его идентификатору
      operationId: deleteTechwriter
      parameters:
        - name: techwriterId
          in: path
          description: Идентификатор технического писателя, которого нужно удалить из базы
          required: true
          schema:
            type: integer
            format: int64
            example: 25
      responses:
        '200':
          description: Технический писатель удален из базы
        '404':
          description: Технического писателя с таким идентификатором нет в базе
  /techwriters/{techwriterId}/skills/{skillId}:
    get:
      tags:
        - techwriters
      summary: Получение информации о конкретном навыке конкретного технического писателя
      description: |
        Метод возвращает информацию о конкретном навыке конкретного технического писателя.
        Возвращается общая информация плюс опыт и подтверждающий документ для конкретного технического писателя
      operationId: getТwSkillInfo
      parameters:
        - name: techwriterId
          in: path
          description: Идентификатор технического писателя, о навыке которого нужно получить информацию
          required: true
          schema:
            type: integer
            format: int64
            example: 123
        - name: skillId
          in: path
          description: Идентификатор навыка конкретного технического писателя, о котором нужно получить информацию
          required: true
          schema:
            type: integer
            format: int64
            example: 25
      responses:
        '200':
          description: ОК
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/TechwriterSkill'
        '400':
          description: Соотношение технического писателя и навыка не найдено
        '404':
          description: Технического писателя нет в базе
    post:
      tags:
        - techwriters
      summary: Добавление навыка для определенного технического писателя
      description: Метод добавляет конкретный навык для определенного технического писателя
      operationId: addTWSkill
      parameters:
        - name: techwriterId
          in: path
          description: Идентификатор технического писателя, которому добавляется навык
          required: true
          schema:
            type: integer
            format: int64
            example: 123
        - name: skillId
          required: true
          in: path
          description: Идентификатор навыка
          schema:
            type: integer
            format: int64
            example: 25
      requestBody:
        description: Подтверждение навыка конкретного техписателя
        required: true
        content:
          'application/json':
            schema:
              $ref: "#/components/schemas/TechwriterSkillInfo"
      responses:
        '200':
          description: ОК
          content:
            application/json:
              schema: 
                $ref: '#/components/schemas/TechwriterSkill'
        '405':
          description: Неверный формат ввода
        '404':
          description: Технического писателя с таким идентификатором нет в базе
  /skills:
    get:
      tags:
        - skills
      summary: Получение информации о доступных в системе навыках
      description: Метод возвращает информацию обо всех доступных в системе навыках
      operationId: getAllSkillsInfo
      parameters:
        - name: limit
          in: query
          description: |
            Количество навыков в выдаче (для пагинации).
            Значение по умолчанию `20`
          required: false
          schema:
            type: integer
            format: int64
            example: 10
            default: 20
      responses:
        '200':
          description: ОК
          content:
            application/json:
              schema:
                type: object
                properties:
                    items:
                      type: array
                      description: Список навыков, доступных в системе
                      items:
                        $ref: '#/components/schemas/Skill'
                    limit:
                      type: integer
                      example: 10
                required:
                  - items
                  - limit
        '400':
          description: Неверный формат limit
        '404':
          description: В базе не найдено навыков
    post:
      tags:
        - skills
      summary: Добавление навыка в базу
      description: Метод добавляет новый навык в базу
      operationId: addSkill
      requestBody:
        description: Описание навыка
        required: true
        content:
          'application/json':
            schema:
              $ref: "#/components/schemas/NewSkill"
      responses:
        '201':
          description: Created
          content:
            application/json:
              schema: 
                $ref: '#/components/schemas/Skill'
        '405':
          description: Неверный формат ввода
  /skills/{skillId}:
    get:
      tags:
        - skills
      summary: Получение информации о навыке по идентификатору
      description: Метод возвращает информацию о навыке по его идентификатору
      operationId: findSkillByID
      parameters:
        - name: skillId
          in: path
          description: Идентификатор навыка, о котором нужно получить информацию
          required: true
          schema:
            type: integer
            format: int64
            example: 25
      responses:
        '200':
          description: ОК
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Skill'
        '400':
          description: Задан неверный id навыка
        '404':
          description: Навык не найден
    patch:
      tags:
        - skills
      summary: Редактирование навыка
      description: |
        Метод изменяет описание навыка. Метод изменяет навык частично: можно передать только название, описание или алиас для изменения.
        Обязательно должен быть передан хотя бы один параметр
      operationId: updateSkill
      parameters:
        - name: skillId
          in: path
          description: Идентификатор навыка, который нужно изменить
          required: true
          schema:
            type: integer
            format: int64
            example: 25
      requestBody:
        description: Изменяемые свойства навыка
        required: true
        content:
          'application/json':
            schema:
              $ref: "#/components/schemas/UpdSkill"
      responses:
        '200':
          description: ОК
          content:
            application/json:
              schema: 
                $ref: '#/components/schemas/Skill'
        '405':
          description: Неверный формат ввода
        '404':
          description: Навык не найден
    delete:
      tags:
        - skills
      summary: Удаление навыка из базы
      description: Метод удаляет навык из базы по его идентификатору
      operationId: deleteSkill
      parameters:
        - name: skillId
          in: path
          description: Идентификатор навыка, который нужно удалить
          required: true
          schema:
            type: integer
            format: int64
            example: 25
      responses:
        '200':
          description: ОК
        '404':
          description: Навык не найден
components:
  schemas:
    NewSkill:
      type: object
      properties:
        skillName:
          type: string
          description: Название навыка
          example: Ручное документирование API
        skillDescription:
          type: string
          description: Описание навыка
          example: Создание документации на API, тестирование, создание описаний вручную
        alias:
          type: string
          description: Алиас навыка (на английском языке в camelCase)
          example: manualApiDocs
      required:
        - skillName
        - skillDescription
        - alias
    UpdSkill:
      type: object
      properties:
        skillName:
          type: string
          description: Название навыка
          example: Ручное документирование API
        skillDescription:
          type: string
          description: Описание навыка
          example: Создание документации на API, тестирование, создание описаний вручную
        alias:
          type: string
          description: Алиас навыка (на английском языке в camelCase)
          example: manualApiDocs
    NewTechwriter:
      type: object
      properties:
        firstName:
          type: string
          description: Имя технического писателя
          example: Вася
        secondName:
          type: string
          description: Фамилия технического писателя
          example: Пупкин
        patronymic:
          type: string
          description: Отчество технического писателя. Необязательное поле
          example: Иванович
        experience:
          type: integer
          format: int64
          description: Опыт работы (в месяцах)
          example: 12
        phoneNumber:
          type: string
          format: phone
          description: Номер телефона для связи
          example: +71234567890
        email:
          type: string
          format: email
          description: Адрес электронной почты технического писателя
          example: pupkin@mail.pup
        skills:
          type: array
          description: Список навыков (массив идентификаторов навыков)
          items:
            type: integer
            format: int64
            example: 1
          example: [1, 2, 3]
      required:
        - firstName
        - secondName
        - experience
        - skills
        - email
    Techwriter:
      type: object
      properties:
        id:
          type: integer
          format: int64
          description: Идентификатор технического писателя
          example: 123
        firstName:
          type: string
          description: Имя технического писателя
          example: Вася
        secondName:
          type: string
          description: Фамилия технического писателя
          example: Пупкин
        patronymic:
          type: string
          description: Отчество технического писателя. Необязательное поле
          example: Иванович
        experience:
          type: integer
          format: int64
          description: Опыт работы (в месяцах)
          example: 12
        phoneNumber:
          type: string
          format: phone
          description: Номер телефона для связи
          example: +71234567890
        email:
          type: string
          format: email
          description: Адрес электронной почты технического писателя
          example: pupkin@mail.pup
        skills:
          type: array
          description: Список навыков (массив идентификаторов навыков)
          items:
            type: integer
            format: int64
            example: 1
          example: [1, 2, 3]
      required:
        - id
        - firstName
        - secondName
        - experience
        - skills
        - email
    UpdTechwriter:
      type: object
      properties:
        firstName:
          type: string
          description: Имя технического писателя
          example: Вася
        secondName:
          type: string
          description: Фамилия технического писателя
          example: Пупкин
        patronymic:
          type: string
          description: Отчество технического писателя. Необязательное поле
          example: Иванович
        experience:
          type: integer
          format: int64
          description: Опыт работы (в месяцах)
          example: 12
        phoneNumber:
          type: string
          format: phone
          description: Номер телефона для связи
          example: +71234567890
        email:
          type: string
          format: email
          description: Адрес электронной почты технического писателя
          example: pupkin@mail.pup
        skills:
          type: array
          description: Список навыков (массив идентификаторов навыков)
          items:
            type: integer
            format: int64
            example: 1
          example: [1, 2, 3]
    Skill:
      type: object
      properties:
        id:
          type: integer
          format: int64
          description: Идентификатор навыка
          example: 25
        skillName:
          type: string
          description: Название навыка
          example: Ручное документирование API
        skillDescription:
          type: string
          description: Описание навыка
          example: Создание документации на API, тестирование, создание описаний вручную
        alias:
          type: string
          description: Алиас навыка
          example: manualApiDocs
      required:
        - id
        - skillName
        - skillDescription
        - alias
    TechwriterSkill:
      type: object
      properties:
        techwriterId:
          type: integer
          format: int64
          description: Идентификатор технического писателя
          example: 123
        skillCommonInfo:
          $ref: '#/components/schemas/Skill'
        confirmingDocument:
          type: string
          description: Подтверждающий документ
          example: mydocuments.com/API.pdf
        linksExamples:
          type: array
          description: Ссылки на примеры работ
          items:
            type: string
            example: ["myexamples.com/example_1", "myapidocs.com/exampleApiDoc.png"]
      required:
        - techwriterId
        - skillCommonInfo
    TechwriterSkillInfo:
      type: object
      properties:
        confirmingDocument:
          type: string
          description: Подтверждающий документ
          example: mydocuments.com/API.pdf
        linksExamples:
          type: array
          description: Ссылки на примеры работ
          items:
            type: string
            example: ["myexamples.com/example_1", "myapidocs.com/exampleApiDoc.png"]
      required:
        - techwriterId
        - skillCommonInfo
```

Также можете скачать готовую спецификацию на [GitLab](https://gitlab.com/denmaloyreb/apidocs_course/-/blob/main/OAS-specs/oas-3.00_iteration_3.yaml?ref_type=heads).

## Некоторые пояснения по итоговой спецификации

У кого-то может возникнуть вопрос, **«почему в `Schemas` гораздо больше схем, чем ресурсов, которые мы определили/спроектировали ранее?»**. Так вот, здесь размещаются не только описания ресурсов, но и модели данных, которыми оперируют методы API: структуры запросов, ответов и т.д. — все, что отправляется и передается, переиспользуется и пр.

Также стоит пояснить, почему для изменения ресурсов мы используем метод с глаголом `PATCH`, если есть еще `PUT`? Действительно, изменять ресурс можно `PATCH` и `PUT`-методами. Но стандарты HTTP рекомендуют использовать `PUT` для полной замены или перемещения ресурса ([подробнее](https://www.ietf.org/rfc/rfc7231.txt)). Т.е. если бы мы использовали `PUT`, то каждый раз при изменении данных, например, техписа или навыка, нужно было бы передавать полный набор полей. При использовании `PATCH` же можно передать только те поля, которые нужно перезаписать.

Теперь у нас есть полноценная спецификация в формате OpenAPI.

## Что дальше

А дальше будем собирать статический справочник из нашей спецификации с помощью [Foliant](https://foliant-docs.github.io/docs/). Перед этим рекомендую вам форкнуть [репозиторий с заготовкой](https://gitlab.com/denmaloyreb/foliant_slate_starterpack), установить на свой компьютер `Docker`, прочесть `Readme` в форкнутом репозитории (там информации не очень много) и, по желанию, выполнить локальную сборку (а может даже и деплой), как там описано.
