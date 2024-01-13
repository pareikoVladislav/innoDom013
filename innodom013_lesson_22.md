<!-- TOC -->
* [**SQLalchemy, Alembic**](#sqlalchemy-alembic)
* [**Work with data**](#work-with-data-)
* [**query's object methods**](#querys-object-methods-)
* [**Alembic**](#alembic-)
* [**Main Components of Alembic**](#main-components-of-alembic-)
* [**Alembic Workflow**](#alembic-workflow)
* [**Home task**](#home-task-)
<!-- TOC -->
# **SQLalchemy, Alembic**

# **Work with data**                                               

Определённая часть запросов будет происходить через объект `Query`                                               

Объект `Query` в `SQLAlchemy` является основным инструментом для создания и                                                      
выполнения запросов к базе данных в стиле `ORM` (**Object-Relational Mapping**).                                                      

Этот объект используется для формирования запросов `SQL` с использованием                                                      
**Python-объектов** и их атрибутов, а не сырых **SQL-запросов**.                                                     

```python
with DBConnection(db_url=DB_URL) as session:
    users = session.query(User).all()

    for user in users:
        print(f"User name - {user.first_name}, user email - {user.email}")
```

Объект `Query` создается через вызов метода `query()` на объекте сессии `SQLAlchemy`                                                     
Где `Model` - это класс, отображаемый на таблицу в базе данных.                                                     

Если метод `all()` не будет указан, то ваш запрос будет возвращён в чистом                                                  
виде (мы так можем глянуть, какой запрос был составлен нашими методами.):                                               
```python
with DBConnection(db_url=DB_URL) as session:
    users = session.query(User)

    print(users)
```

# **query's object methods**                                                     

**Фильтрация (`Filtering`)**:                                                     

`filter()`: Принимает условия для фильтрации результатов, например,                                                      
`session.query(User).filter(User.name == 'Alice')`.                                                     
При всём этом этот метод используется для создания более мощных, гибких                                                 
условий фильтрации. В этом методе выражения записываются как утверждения                                                
SQL, используя при этом операторы сравнений:


```python
with DBConnection(db_url=DB_URL) as session:
    filtered_users = session.query(User).filter(
        User.email.like('%gmail.com')
    ).all()

    for user in filtered_users:
        print(f"User name - {user.first_name}")
```
`filter_by()`: Предоставляет более простой способ фильтрации, используя                                                      
ключевые слова, например, `session.query(User).filter_by(name='Alice')`.                                                     

```python
with DBConnection(db_url=DB_URL) as session:
    filtered_users = session.query(User).filter_by(
        first_name='Robert'
    ).all()

    print(filtered_users)
```

В `filter_by()` обычно используются аргументы с ключевыми словами для                                                  
фильтрации. Менее гибкий вариант фильтрации, но более простой в                                                      
использовании.                                                                         


Так же если вам необходимо проходиться сразу по нескольким условиям                                                     
вы можете использовать следующую конструкцию:  

```python
with DBConnection(db_url=DB_URL) as session:
    filtered_users = session.query(User).filter(
        or_(User.first_name.like('J%'), User.email.like('%yahoo.com'))
    ).all()

    print(filtered_users)
    for user in filtered_users:
        print(f"User name - {user.first_name}")
```

**Сортировка (Ordering)**:                                                     

`order_by()`: Упорядочивает результаты запроса, например,                                                      
`session.query(User).order_by(User.name)`.                                                     
```python
with DBConnection(db_url=DB_URL) as session:
    users = session.query(User).filter(
        and_(User.role_id == 3, User.rating >= 4)
    ).order_by(User.rating)

    for user in users:
        print(f"The user's ID - {user.id}. '{user.first_name}', {user.rating}, email is - {user.email}")
```
**Ограничение результатов и срезы (Limiting and Slicing)**:                                                     

`limit()`: Ограничивает количество возвращаемых результатов,                                                      
например, `query.limit(10)`.                                                     
`Срезы Python`: Можно использовать синтаксис срезов, например, `query[1:3]`.                                                     

```python
with DBConnection(db_url=DB_URL) as session:
    users = session.query(User).filter(
        and_(User.role_id == 3, User.rating >= 4)
    ).order_by(User.rating).limit(5)

    for user in users:
        print(f"The user's ID - {user.id}. '{user.first_name}', {user.rating}, email is - {user.email}")
```


**Агрегация (Aggregation)**:                                                                                                          

Методы, такие как `count()`, `sum()`, `avg()`, используются для агрегации                                                      
данных, например, `session.query(User).filter(User.age > 18).count()`.                                                     
```python
with DBConnection(db_url=DB_URL) as session:
    users = session.query(User).filter(
        User.rating >= 5
    ).count()

    print(users)

    print(f"Кол-во авторов с рейтингом выше пяти - {users}")
```

**Выполнение запроса:**                                                                                                          

`all()`: Возвращает список всех объектов, соответствующих запросу.                                                     
`first()`: Возвращает первый объект из результата или None.                                                     
`one()`: Возвращает ровно один объект или вызывает исключение.                                                     
`scalar()`: Возвращает одно значение (первый элемент первой строки результата).                                                     

```python
with DBConnection(db_url=DB_URL) as session:
    user = session.query(User).filter(
        User.rating >= 7
    ).first()
    print(f"The user's ID - {user.id}. '{user.first_name}', {user.rating}, email is - {user.email}")
```

```python
with DBConnection(db_url=DB_URL) as session:
    user = session.query(User).filter(
        User.id == 2
    ).one()
    print(f"The user's ID - {user.id}. '{user.first_name}', {user.rating}, email is - {user.email}")
```

**Присоединение (Joining)**                                                                                                          
`join()`: Используется для соединения таблиц                                                                                                                 


```python
with DBConnection(db_url=DB_URL) as session:
    users = session.query(User).options(
        joinedload(User.news)
    ).filter(
        User.rating >= 5
    ).all()

    users_data = []

    for user in users:
        user_news = [
            {
                "id": n.id,
                "title": n.title,
                "moderated": n.moderated,
                "deleted": n.deleted,
                "created_at": datetime.strftime(n.created_at, "%y-%m-%d %H:%M:%S")
            } for n in user.news
        ]

        user_data = {
            "id": user.id,
            "name": user.first_name,
            "surname": user.last_name,
            "rating": user.rating,
            "email": user.email,
            "news": user_news
        }

        users_data.append(user_data)

    data = json.dumps(users_data, indent=4)

    print(data)
```

`joinedload()` - это стратегия загрузки, которая позволяет выполнять                                                      
**«жадную»** загрузку **связанных объектов**. Это означает, что `SQLAlchemy` будет                                                    
**автоматически** загружать связанные объекты во время первоначального запроса,                                                    
что помогает избежать проблемы `N+1` запросов при обращении к связанным данным.                                                   

В нашем случае, используя `joinedload()` с моделью `User` и ее связью с `News`, мы                                                  
можем загрузить пользователей вместе с новостями, которые они создавали                                                 
в одном запросе. Это уменьшит количество обращений к базе данных, так как                                                  
все необходимые данные будут получены сразу.                                                                  

**Обычные же `join` работают следующим образом**:                                                                 

```python
with DBConnection(db_url=DB_URL) as session:

    specific_user_id = 2

    result = session.query(User, News).join(
        News, User.id == News.author_id
    ).filter(User.id == specific_user_id).all()

    if result:
        for user, news in result:
            print(f"User: {user.first_name} {user.last_name}")
            print(f"News: {news.title}")
            print("-" * 20)
    else:
        print([])
```

Что же тут происходит? На самом деле не всё так страшно, как могло бы                                                   
показаться. В объект запроса на самом деле мы можем передавать не только                                               
одну конкретную модель (мы даже можем передавать туда конкретные поля для                                               
выборки). В случае, когда передаётся более одной модели нам необходимо                                                 
испольховать метод `join()`, который и будет нам соединять наши таблицы.                                                   
Нам необходимо в нём указать куда и как мы хотим присоединиться.                                                    
Первым аргументом мы передаём название модели, которую хотим присоединить                                                
к основной, главной модели. Вторым аргументом мы прокидываем связь полей,                                                 
по которым мы хотим соединиться. На этом вся магия заканчивается :)                                                     

Мы так же можем выбирать конкретные поля, а не все. Для этого в методе                                                    
`query()` мы просто, обращаясь к определённой модели, через точку берём                                                   
только нужные нам поля:                                                             
```python
with DBConnection(db_url=DB_URL) as session:
    specific_user_id = 2

    result = session.query(
        User.id,
        User.first_name,
        User.email,
        User.rating,
        News.id.label('news_id'),
        News.title,
        News.moderated,
        News.deleted,
        News.created_at.label('news_created_at'),
        Comment.id.label('comment_id'),
        Comment.content,
        Comment.created_at.label('comment_created_at')
    ).join(
        News, User.id == News.author_id
    ).join(
        Comment, News.id == Comment.news_id
    ).filter(User.id == specific_user_id).all()

    for row in result:
        print(row)
```

**Группировка (Grouping)**                                                                                                          
`group_by()`: Используется для группировки результатов по                                                      
определенным критериям.                                                     
```python
with DBConnection(db_url=DB_URL) as session:
    specific_user_id = 2

    result = session.query(
        News, func.count(Comment.id).label('comments_count')).join(
        User, News.author_id == User.id).outerjoin(
        Comment, News.id == Comment.news_id).filter(
        News.author_id == specific_user_id).group_by(News.id).all()

    for news, comment_count in result:
        print(f"News: {news.title}, Comments: {comment_count}")
```

**Дополнительные функции**                                                     
`having()`: Применяется **после** `group_by()` для фильтрации                                                      
группированных результатов.                                                     
`distinct()`: Возвращает уникальные результаты.                                                     


**Обновление существующих записей**                                                  

```python
with DBConnection(db_url=DB_URL) as session:
    specific_user_id = 2

    user = session.query(User).filter_by(id=specific_user_id).one()

    user.email = 'eliz.hebert@icloud.com'
    user.phone = '+995 845 112 325'
    user.updated_at = datetime.now()

    session.commit()
```

```python
request_data = {
    "user": {"id": 21},
    "body": {
        "title": "New news from the sqlalchemy",
        "content": """
            This is a new news from the SQLALCHEMY! We're did this!!!!
        """,
        "moderated": False,
        "deleted": False,
    }

}


def update_user_info(request):
    user_id = request.get("user").get("id")
    with DBConnection(db_url=DB_URL) as session:
        try:
            user = session.query(User).filter_by(id=user_id).one()

            user.email = 'steph.wong@icloud.com'
            user.updated_at = datetime.now()

            session.commit()
        except Exception as err:
            print(err)


def create_new_news(request):
    user_id = request.get("user").get("id")

    with DBConnection(db_url=DB_URL) as session:
        try:
            news = News(
                id=41,
                title=request.get("body").get("title"),
                content=request.get("body").get("content"),
                author_id=user_id,
                moderated=request.get("body").get("moderated"),
                deleted=request.get("body").get("deleted"),
            )

            session.add(news)

            session.commit()
        except Exception as err:
            print(err)
            session.rollback()


def initiator():
    user_choice = input("Enter action: [update user info, create news]: ")

    match user_choice.strip().lower():
        case "update user info":
            update_user_info(request_data)
        case "create news":
            create_new_news(request_data)
        case _:
            print("Unknown command.")


initiator()
```

---

# **Alembic**                                                        

**Alembic** — это легковесная, базирующаяся на миграциях библиотека для                                                                                             
`SQLAlchemy`. Она создана для **управления изменениями в схеме базы данных**,                                                                                                      
позволяя отслеживать, модифицировать и создавать схему базы данных.                                                                                                       

Использование `Alembic` особенно полезно в средах, где работают **несколько**                                                                                                       
**разработчиков** и/или когда приложения развертываются на разных                                                                                                      
стадиях или окружениях.                                                                                                      

---
# **Main Components of Alembic**                                              

**Миграции**: Это основная функциональность `Alembic`. Миграции позволяют                                                                                                      
вносить изменения в схему базы данных, такие как добавление/удаление                                                                                                      
таблиц или столбцов, изменение типов данных столбцов и т.д.                                                                                                      

**Ревизии**: Каждая миграция в `Alembic` представляет собой ревизию.                                                                                                       
Ревизии обычно состоят из двух методов: `upgrade()` и `downgrade()`.                                                                                                       
`upgrade()` используется для применения изменений схемы, а                                                                                                       
`downgrade()` — для отката этих изменений.                                                                                                      

**Скрипты Миграции**: `Alembic` позволяет писать скрипты миграции на                                                                                                       
`Python`, что дает большую гибкость и контроль над процессом миграции.                                                                                                      

---
# **Alembic Workflow**
**Инициализация**: Сначала необходимо инициализировать `Alembic` в проекте,                                                                                                       
что создаст каталог миграций и файл конфигурации.                                                                                                      

**Создание Ревизии**: Для создания новой миграции используется команда                                                                                                       
`alembic revision -m "описание"`. Это создаст новый скрипт миграции.                                                                                                      

**Редактирование Скрипта Миграции**: После создания скрипта, его                                                                                                      
необходимо отредактировать, добавив нужные изменения в методы                                                                                                       
`upgrade()` и `downgrade()`.                                                                                                      

**Применение Миграции**: Чтобы применить миграцию к базе данных,                                                                                                      
используется команда `alembic upgrade head`.                                                                                                      

**Откат Миграции**: Если нужно отменить последние изменения,                                                                                                       
используется команда `alembic downgrade -1`.                                                                                                      

---

**установка alembic**                                              

Сперва нужно установить доп библиотеку алембик (`pip install alembic`)                                              

После чего нужно сынициализировать эту махину:                                                                                           
`alembic init alembic`(Это создаст директорию "alembic" в вашем проекте                                                                                         
с файлами конфигурации.)                                              
Потом нужно будет определить инициализацию скриптов в файле `alembic.ini`:                                                                                     

```python
[alembic]
# path to migration scripts
script_location = lesson_22/alembic
...
...
...
# the output encoding used when revision files
# are written from script.py.mako
# output_encoding = utf-8

sqlalchemy.url =
```

В Файле `alembic/env.py` добавить строчки:                                                                                        

```python
import os
from dotenv import load_dotenv

load_dotenv()

url_engine = os.getenv('DB_POS_URL')

context.config.set_main_option("sqlalchemy.url", url_engine)
...
...
...
# add your model's MetaData object here
# for 'autogenerate' support
# from myapp import mymodel
# target_metadata = mymodel.Base.metadata
target_metadata = Base.metadata
```

Создать первую миграцию:                                                                                             

`alembic revision --autogenerate -m "Initial migration"`                                              

Это создаст новый файл миграции в директории "alembic/versions"                                                                                       
соответствующий вашим изменениям в модели.                                                                                           

Откройте созданный файл миграции и троху поредачьте его.                                                                                             
Нас интересует именно функция `upgrade`                                                                                            

```python
def upgrade() -> None:
    # ### commands auto generated by Alembic - please adjust! ###
    pass
    # ### end Alembic commands ###


def downgrade() -> None:
    # ### commands auto generated by Alembic - please adjust! ###
    pass
    # ### end Alembic commands ###
```

Применить миграцию к Базе Данных можно след командой:                                                                                                

`alembic upgrade head`                                              

Это применит миграцию и создаст то, шо вы там прописали в нужном                                                                                             
вам методе.                                                                                             

После выполнения всех шагов, у вас будет успешно выполнена                                                                                       
миграция, и ваша база данных будет обновлена\создана.                                                                                        

---

**upgrade():**                                              
Метод **upgrade()** используется для применения к базе данных изменений,                                              
заданных в файле миграции. При запуске миграции `Alembic` выполняет метод                                              
`upgrade()` для приведения схемы базы данных в актуальное состояние.                                              
Этот метод обычно содержит операции **SQL** или код манипулирования                                              
схемой `SQLAlchemy` для создания новых таблиц, изменения существующих                                              
таблиц, добавления столбцов, модификации данных и т.д. Этот метод                                              
должен содержать операции, необходимые для возврата изменений                                              
схемы, внесенных в соответствующем методе `upgrade()`.                                              


**downgrade():**                                              
Метод `downgrade()` используется для отмены изменений, сделанных соответствующим                                              
методом `upgrade()`. Если необходимо откатить изменения, внесенные в                                              
результате определенной миграции, `Alembic` выполнит метод `downgrade()`                                              
для отмены изменений. Этот метод должен содержать операции, необходимые                                              
для отмены изменений схемы, внесенных в соответствующем методе `upgrade()`.                                              

`Alembic` позволяет создавать более сложные миграции, включая изменение                                                                                        
структуры таблиц, добавление новых таблиц и другие операции с базой данных.                                                                                                  
Он предоставляет мощные средства для управления схемой базы данных в проекте.                                                                                    

---

# **Home task**                                              

Написать **НЕБОЛЬШУЮ** систему с переводами деняк)))))                                              

Продумать необходимые для этого модели                                              

Необходимые секретные данные должны **хэшироваться**                                              

Можно будет                                              
**создавать** пользователя,                                              
**обновлять** нужную инфу,                                              
**просматривать** нужную инфу                                                                                           
Иметь возможность **закидывать** деньги на счёт,                                              
**снимать** их                                              
и **переводить**                                              


Все операции должны быть реализованы через базы данных, никаких больше файлов.                                                                                           
Все операции должны быть с соблюдением `ACID` свойств.                                             

подключение к базе данных(или MySQL, или PostgreSQL)                                             

работа принимается ТОЛЬКО в виде PR на меня от другой ветки.                                             

работа с БД через `SQLalchemy`                                                                                           
