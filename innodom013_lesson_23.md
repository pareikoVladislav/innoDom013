<!-- TOC -->
* [**SQLAlchemy, Pydantic**](#sqlalchemy-pydantic)
* [**JOIN's variants**](#joins-variants-)
* [**Pydantic**](#pydantic-)
<!-- TOC -->

# **SQLAlchemy, Pydantic**


В наших моделях для классов мы определяли такие поля, значения у которых                                                  
были представлены методом `relationship()`.                                                      

`relationship` - Обеспечивает связь между двумя сопоставленными классами.                                                    
Это соответствует отношениям "родитель-ребенок" или ассоциативным таблицам.                                               
Созданный класс является экземпляром `.Relationship`.                                                               

Вот некоторые из самых распространенных аргументов:                                                              

* `back_populates` или `backref`: Указывает на обратную связь между моделями. Если вы                                                               
используете `back_populates`, вам нужно явно определить эту связь в обеих                                                              
моделях. `backref` автоматически добавляет обратную связь к связанной модели.                                                              

* `secondary`: Используется для определения ассоциативной таблицы в отношениях                                                               
**многие-ко-многим**. Это позволяет создать связь, где одна запись в таблице                                                               
может быть связана с множеством записей в другой таблице.                                                              

* `lazy`: Определяет, как `SQLAlchemy` будет загружать связанные объекты.                                                              
Например, `select` (загрузка при первом обращении), `joined` (загрузка вместе                                                              
с запросом к родительской таблице) или `dynamic` (загрузка элементов                                                              
по требованию).                                                              

* `uselist`: По умолчанию `SQLAlchemy` рассматривает отношения как коллекцию объектов.                                                               
Если установлено значение `False`, связь будет восприниматься как связь **один-к-одному**.                                                              
В противном случае (по умолчанию `True`) связь будет считаться **один-ко-многим**.                                                              

* `cascade`: Управляет поведением каскадных операций, таких как сохранение, обновление                                                              
и удаление. Например, если вы установите `cascade="all, delete"`, то при удалении                                                               
объекта будут автоматически удалены все связанные с ним объекты.                                                              

* `primaryjoin`: Используется для указания условия соединения для отношения. Это                                                               
особенно полезно в сложных сценариях, где автоматически определенное условие                                                              
соединения не подходит.                                                              

* `order_by`: Позволяет определить порядок, в котором элементы будут загружаться                                                              
из связанной таблицы.                                                              

* `foreign_keys`: Указывает на конкретные столбцы, которые должны использоваться                                                               
как внешние ключи в отношении. Это может быть необходимо в случаях, когда                                                              
`SQLAlchemy` не может автоматически определить, какие внешние ключи использовать.                                                              

* `remote_side`: Используется в отношениях **один-к-одному** или **один-ко-многим** для                                                               
указания, какая сторона отношения считается "удаленной".                                                              



`joinedload()` - это стратегия загрузки, которая позволяет выполнять                                                      
**«жадную»** загрузку **связанных объектов**. Это означает, что `SQLAlchemy` будет                                                    
**автоматически** загружать связанные объекты во время первоначального запроса,                                                    
что помогает избежать проблемы `N+1` запросов при обращении к связанным данным.                                                   

В нашем случае, используя `joinedload()` с моделью `User` и ее связью с `News`, мы                                                  
можем загрузить пользователей вместе с новостями, которые они создавали                                                 
в одном запросе. Это уменьшит количество обращений к базе данных, так как                                                  
все необходимые данные будут получены сразу.                                                                  

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
        News.id.label('news_id'),  # Спец метод, который позваляет нам дать другое название полю. Более удобное нам
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

# **JOIN's variants**                                                          

**Как вообще можно делать объединения таблиц?**                                                     

* **Явное соединение:** Вы можете явно указать, какие таблицы                                                             
и как должны быть соединены.                                                           

```python
with DBConnection(db_url=DB_URL) as session:
    user = session.query(
        User.id,
        User.first_name,
        User.last_name,
        User.email,
        User.phone,
        User.rating,
        Role.name.label('user_role')
    ).join(Role, User.role_id == Role.id).filter(
        User.id == 16
    ).one()

    user_data = {
        "id": user.id,
        "first_name": user.first_name,
        "last_name": user.last_name,
        "email": user.email,
        "phone": user.phone,
        "rating": user.rating,
        "role": user.user_role
    }

    json_data = json.dumps(user_data, indent=4)

    print(json_data)
```

* **Неявное соединение:** Если между таблицами определена связь, можно опустить                                                            
условие соединения.                                                           

```python
with DBConnection(db_url=DB_URL) as session:
    users = session.query(User).join(News).all()

    users_data = []

    for user in users:
        news_data = [
            {
                "id": n.id,
                "title": n.title,
                "moderated": n.moderated
            } for n in user.news
        ]

        user_data = {
            "id": user.id,
            "first_name": user.first_name,
            "last_name": user.last_name,
            "email": user.email,
            "news": news_data
        }

        users_data.append(user_data)

    json_data = json.dumps(users_data, indent=4)

    print(json_data)
```

* **Использование `outerjoin` для внешних соединений**:                                                           
Подобно `join`, но для создания `LEFT OUTER JOIN`.                                                           

```python
with DBConnection(db_url=DB_URL) as session:
    result = session.query(News).outerjoin(
        Comment, News.comments
    ).all()

    all_news_data = []

    for news in result:
        comments = [
            {
                "id": c.id,
                "content": c.content,
                "created_at": datetime.strftime(c.created_at, "%y-%m-%d %H:%M:%S")
            } for c in news.comments
        ]

        news_data = {
            "id": news.id,
            "title": news.title,
            "content": news.content,
            "moderated": news.moderated,
            "author": news.author_id,
            "comments": comments,
        }

        all_news_data.append(news_data)

    json_data = json.dumps(all_news_data, indent=4)

    print(json_data)
```

* **Использование `subquery` для сложных запросов**:                                                           
Вложенные запросы могут использоваться для создания сложных соединений.                                                           

```python
with DBConnection(db_url=DB_URL) as session:
    subq = session.query(User.id).filter(
        User.rating > 5
    ).subquery()
    result = session.query(News).join(
        subq,
        News.author_id == subq.c.id
    ).all()

    all_news_data = []

    for news in result:
        comments = [
            {
                "id": c.id,
                "content": c.content,
                "created_at": datetime.strftime(c.created_at, "%y-%m-%d %H:%M:%S")
            } for c in news.comments
        ]

        news_data = {
            "id": news.id,
            "title": news.title,
            "content": news.content,
            "moderated": news.moderated,
            "author": news.author_id,
            "comments": comments,
        }

        all_news_data.append(news_data)

    json_data = json.dumps(all_news_data, indent=4)

    print(json_data)
```

* **Использование `contains_eager` для оптимизации загрузки связанных объектов**:                                                           
Это используется, когда вы хотите явно управлять запросами `Eager Loading`.                                                           

```python
session.query(Table1).join(Table2).options(contains_eager(Table1.table2)).all()
```

* **Использование `aliased` для работы с одной и той же таблицей в разных ролях**:                                                           
Когда одна таблица участвует в запросе несколько раз в различных ролях или в                                                           
сложных запросах с несколькими соединениями.                                                           

```python
with DBConnection(db_url=DB_URL) as session:
    start_date = datetime(2023, 1, 1)
    end_date = datetime(2023, 12, 31)

    CommentUserAlias = aliased(User)

    result = session.query(News) \
        .join(User, News.user) \
        .outerjoin(Comment, News.comments) \
        .outerjoin(CommentUserAlias, Comment.user) \
        .filter(News.created_at.between(start_date, end_date)) \
        .options(contains_eager(News.user)) \
        .options(joinedload(News.comments).contains_eager(Comment.user, alias=CommentUserAlias)) \
        .all()

    print(result)

news_data_list = []

for news in result:
    author_data = {
        "id": news.user.id,
        "first_name": news.user.first_name,
        "last_name": news.user.last_name
    }

    comments_data = [
        {
            "id": comment.id,
            "content": comment.content,
            "author": {
                "id": comment.user.id,
                "first_name": comment.user.first_name,
                "last_name": comment.user.last_name
            },
            "created_at": datetime.strftime(comment.created_at, "%Y-%m-%d %H:%M:%S")
        } for comment in news.comments
    ]

    news_data = {
        "id": news.id,
        "title": news.title,
        "moderated": news.moderated,
        "deleted": news.deleted,
        "created_at": datetime.strftime(news.created_at, "%Y-%m-%d %H:%M:%S"),
        "author": author_data,
        "comments": comments_data
    }

    news_data_list.append(news_data)

json_data = json.dumps(news_data_list, indent=4)

print(json_data)
```

* **Использование `select_from` для явного указания основной таблицы в соединении**:                                                           
Это полезно, когда необходимо явно указать, с какой таблицы должен начинаться запрос.                                                           

```python
session.query(Table1).select_from(Table2).join(Table1).all()
```

**Использование `with_polymorphic` для соединения с полиморфными моделями**:
При работе с наследованием и полиморфизмом, это позволяет соединять базовые                                                           
и производные таблицы.                                                           

```python
session.query(BaseClass).join(SubClass).all()
```

---

# **Pydantic**                                                                 

На проектах вам так же может понадобиться возможность валидации данных,                                                  
для, записи в Базу Данных и для отправки их из БД на фронт часть.                                                  

Один из таких способов валидации данных - использование библиатеки. В нашем                                             
случае - это библиатека **Pydantic**.                                                                         

**Pydantic** — это полезный инструмент для пользователей Python, особенно                                                                         
при работе с данными                                                                         

**Pydantic** помогает проверять, соответствуют ли данные определённым требованиям                                                                          
или форматам. Например, мы можем проверить, является ли строка действительным                                                                         
адресом электронной почты или целым числом.                                                                         

С помощью Pydantic можно создавать **модели данных**, определяя классы со **строгими**                                                                         
**типами**. Это улучшает **читаемость** и **поддерживаемость** кода, а также помогает                                                                         
**в обнаружении ошибок**.                                                                         

**Pydantic** часто используется вместе с **FastAPI**, популярным фреймворком для                                                                         
создания веб-приложений. В этом контексте **Pydantic** обеспечивает валидацию                                                                          
входящих запросов и сериализацию ответов.                                                                         

Она позволяет легко преобразовывать сложные структуры данных, такие как **JSON**,                                                                          
в модели **Python** с чётко определёнными типами.                                                                         

**Pydantic** может автоматически генерировать схемы **JSON** для                                                                          
моделей, что упрощает документирование **API** и валидацию клиентских данных.                                                                         


Важно при этом не забывать, что Эта библиатека может применяться в целом для                                            
валидации данных, а не только конкретно под базы данных.                                                             

**Pydantic** используется для валидации и управления данными, особенно                                                             
при переводе данных из/в форматы вроде JSON, и обеспечении соответствия                                                             
данным определённым схемам (контрактам).                                                                             

В контексте базы данных, **Pydantic** может быть использован                                                                             
для валидации данных перед их записью в базу данных или после извлечения из неё.                                                                              
Это особенно полезно при работе с **веб-API**, где мы хотим убедиться, что данные,                                                                             
отправляемые пользователями, соответствуют нашим требованиям.                                                                             

Для определения типов данных для полей таблицы базы данных можно использовать                                          
подход написания через классы. Мы должны создать класс с названиями полей                                                  
определённой таблицы, в качестве значений у таких полей будет ожидаемый                                                    
тип данных \ значение для каждого конкретного поля. Такой механизм называется                                                   
**контрактами**.                                                                  

Благодаря этим контрактам мы можем валидировать входящие и выходящие                                                       
данные для более надёжной работы с **JSON** форматом.                                                          


Часто можно увидеть такие "контракты" на одну и ту же модель, но на разные                                              
операции с данными. Начнём по порядку с наших ролей. Создадим такие контракты                                              
для модели Ролей на получение данных, создание и обновление:

```python
from pydantic import (
    BaseModel
)
from pydantic.types import StringConstraints

from typing import Annotated


class RoleCreate(BaseModel):
    name: Annotated[str, StringConstraints(max_length=20)]


class RoleRead(BaseModel):
    id: int
    name: str


class RoleUpdate(BaseModel):
    name: Annotated[str, StringConstraints(max_length=20)]

```

Как вы можете обнаружить, для чтения данных мы используем обычные                                                       
типы данных прям из питона. Так как данные из базы данных мы получаем,                                                     
после чего они преобразовываются в питон объекты, с которыми мы можем                                                      
работать.                                                                

В рамках же создания и обновления данных, конкретно здесь, мы используем                                                 
более углубленный, точный, более гибкий в перспективе формат валидации.                                                  
Нам понадобится класс `Annotaded` из `pydantic.types`.                                                             
`Annotaded` позволяет добавлять дополнительные ограничения к типам данных,                                           
сохраняя при этом чистоту и читаемость аннотаций типов.                                                                 
`StringConstraints` класс позволяет нам накидывать дополнительные мета                                                 
настройки на тип данных, чтобы валидировать его более точно, как нужно                                                   
именно нам.                                                                               


И после попытаемся создать наши роли:                                                           

```python
from pydantic_core import ValidationError

from lesson_23.db_conn import DBConnection
from lesson_23.db_url_getter import DB_URL
from lesson_23.models import (
    Role,
)
from lesson_23.contracts.role import RoleCreate


def create_new_role(data: dict):
    with DBConnection(db_url=DB_URL) as session:
        try:
            new_role_data = RoleCreate(**data)
            
            new_role = Role(name=new_role_data.name)
            session.add(new_role)
            session.commit()
            session.refresh(new_role)

            return new_role

        except ValidationError as err:
            print(err)
            session.rollback()

            
fake_role_form_data = {
    "name": "supperDupperMegaAdminRole"
}

role_form_data = {
    "name": "newRole"
}
            
create_new_role(data=fake_role_form_data)  # сработают валидаторы
create_new_role(data=role_form_data)  # запрос пройдёт
```

Так же с получением данных:                                                       

```python
def get_role(role_id: int):
    with DBConnection(db_url=DB_URL) as session:
        try:
            required_role = session.query(Role).filter(
                Role.id == role_id
            ).one()

            role_data = {key: getattr(required_role, key) for key in RoleRead.model_fields.keys() if hasattr(required_role, key)}

            validated_data = RoleRead(**role_data)

            return validated_data

        except NoResultFound as err:
            return {
                "error": err
            }

        except ValidationError as err:
            return {
                "error": err
            }


role = get_role(role_id=4)

json_data = json.dumps(
    {
        "id": role.id,
        "name": role.name
    },
    indent=4
)

print(json_data)
```

И обновлением:                                                   

```python
def update_role(role_id: int, data: dict):
    with DBConnection(db_url=DB_URL) as session:
        try:
            role_to_update = session.query(Role).filter(
                Role.id == role_id
            ).one()
            
            new_role_data = RoleUpdate(**data)

            role_to_update.name = new_role_data.name
            session.commit()
            session.refresh(role_to_update)

            return role_to_update

        except NoResultFound as err:
            return str(err)

        except ValidationError as err:
            return str(err)


fake_role_form_data = {
    "name": 4
}
        
role_form_data = {
    "name": 'JustRole'
}

print(update_role(role_id=4, data=fake_role_form_data))  # сработает валидация, запрос не пройдёт
print(update_role(role_id=4, data=role_form_data))  # всё сработает
```


Теперь давайте разберёмся с пользователем:                                                      

Контракты будут так же на получение данных о пользователе, создании и обновлении.                                           

```python
from datetime import datetime

from pydantic import (
    BaseModel,
    EmailStr,
)
from pydantic.types import StringConstraints

from typing import Annotated, Optional


class UserCreate(BaseModel):
    first_name: Annotated[str, StringConstraints(max_length=25)]
    last_name: Optional[Annotated[str, StringConstraints(max_length=30)]] = None
    phone: Optional[Annotated[str, StringConstraints(max_length=45, pattern=r"^\+?1?[\d\s-]{9,15}$")]] = None
    email: EmailStr
    password: Annotated[str, StringConstraints(min_length=8)]
    repeat_password: Annotated[str, StringConstraints(min_length=8)]
    role_id: int

    @classmethod
    def passwords_match(cls, values, field):
        password = cls.__getattribute__(values.instance, 'password')
        repeat_password = values.get('repeat_password')
        if repeat_password is not None and repeat_password != password:
            raise ValueError("Passwords don't match!")
        return repeat_password


class UserRead(BaseModel):
    id: int
    first_name: str
    last_name: Optional[str] = None
    phone: Optional[str] = None
    email: str
    role_id: int
    rating: float
    created_at: datetime
    updated_at: Optional[datetime] = None
    deleted_at: Optional[datetime] = None


class UserUpdate(BaseModel):
    first_name: Optional[Annotated[str, StringConstraints(max_length=25)]] = None
    last_name: Optional[Annotated[str, StringConstraints(max_length=30)]] = None
    phone: Optional[Annotated[str, StringConstraints(max_length=45)]] = None
    email: Optional[EmailStr] = None
    role_id: Optional[int] = None
    rating: Optional[float] = None
    updated_at: datetime

```

```python
def create_new_user(data: dict):
    with DBConnection(db_url=DB_URL) as session:
        try:
            user_creation_data = UserCreate(**data)
            
            # user = User(
            #     last_name=user_creation_data.last_name,
            #     phone=user_creation_data.phone,
            #     email=user_creation_data.email,
            #     password=user_creation_data.password,
            #     repeat_password=user_creation_data.repeat_password,
            #     role_id=user_creation_data.role_id
            # )
            
            user = User(
                first_name=user_creation_data.first_name,
                phone=user_creation_data.phone,
                email=user_creation_data.email,
                password=user_creation_data.password,
                repeat_password=user_creation_data.repeat_password,
                role_id=user_creation_data.role_id
            )

            session.add(user)
            session.commit()
            session.refresh(user)

            return user

        except ValidationError as err:
            session.rollback()
            return str(err)

        
user_fake_form_data = {
    "last_name": 'Anquaria',
    "email": 'yana.s@gmail.com',
    "password": 'aF9a$5-s76Df5)5sd@7',
    "repeat_password": 'aF9a$5-7',
    "role_id": "1",
}

user_form_data = {
    "first_name": 'Yana',
        "phone": '+ 1 331-852-446',
        "email": 'yana.s@gmail.com',
        "password": 'aF9a$5-s76Df5)5sd@7',
        "repeat_password": 'aF9a$5-s76Df5)5sd@7',
        "role_id": 1,
}

print(  # не пройдёт
    create_new_user(data=user_fake_form_data)
)

print(
    create_new_user(data=user_form_data)
)

```

```python
def update_user(user_id: int, data: dict):
    with DBConnection(db_url=DB_URL) as session:
        try:
            user = session.query(User).filter(
                User.id == user_id
            ).one()
            
            new_user_data = UserUpdate(**data)

            user.last_name = new_user_data.last_name
            user.rating = new_user_data.rating
            user.updated_at = datetime.now()

            session.commit()
            session.refresh(user)

            return user

        except NoResultFound as err:
            return str(err)

        except ValidationError as err:
            session.rollback()
            return str(err)

        
user_update_form_data = {
            # "last_name": 'NikiforNikiforNikiforNikiforNikiforNikiforNikiforNikifor',  # не пройдёт валидация по длинне фамилии
            # "last_name": True,  # не пройдёт валидация из-за типа данных
            "last_name": 'Nikifor',
            # "rating": "one"  # не пройдёт валидация из-за типа данных
            "rating": 8.1
}

print(update_user(
    user_id=2,
    data=user_update_form_data
))
```

```python
def get_user_by_id(user_id: int):
    with DBConnection(db_url=DB_URL) as session:
        try:
            user = session.query(User).filter(
                User.id == user_id
            ).one()

            user_data = {key: getattr(user, key) for key in UserRead.model_fields.keys() if hasattr(user, key)}

            validated_user = UserRead(**user_data)

            return validated_user

        except NoResultFound as err:
            return str(err)

        except ValidationError as err:
            session.rollback()
            return str(err)


def represent_json_user(user: UserRead):
    user_data = {
        "id": user.id,
        "first_name": user.first_name,
        "last_name": user.last_name,
        "phone": user.phone,
        "email": user.email,
        "role_id": user.role_id,
        "rating": user.rating,
        "created_at": datetime.strftime(
            user.created_at,
            "%y-%m-%d %H:%M:%S"
        ),
        "updated_at": datetime.strftime(
            user.updated_at,
            "%y-%m-%d %H:%M:%S"
        ),
    }

    user_json = json.dumps(user_data, indent=4)

    return user_json


print(
    represent_json_user(
        get_user_by_id(user_id=2)
    )
)
```

Похожим образом будет проходить и структура с новостями:                                                        

Сперва напишем контракты:                                                       

```python
from datetime import datetime

from pydantic import (
    BaseModel
)

from pydantic.types import StringConstraints

from typing import (
    Annotated,
    Optional,
)


class NewsCreate(BaseModel):
    title: Annotated[str, StringConstraints(max_length=100)]
    content: Annotated[str, StringConstraints(max_length=1500)]
    author_id: int


class NewsRead(BaseModel):
    id: int
    title: str
    content: str
    author_id: int
    moderated: bool
    deleted: bool
    created_at: datetime
    updated_at: Optional[datetime] = None
    deleted_at: Optional[datetime] = None


class NewsUpdate(BaseModel):
    title: Optional[Annotated[str, StringConstraints(max_length=100)]] = None
    content: Optional[Annotated[str, StringConstraints(max_length=1500)]] = None
    moderated: Optional[bool] = None
    deleted: Optional[bool] = None
    updated_at: datetime

```


```python
def create_news(data: dict):
    with DBConnection(db_url=DB_URL) as session:
        try:
            news_form_data = NewsCreate(**data)

            news = News(
                title=news_form_data.title,
                content=news_form_data.content,
                author_id=news_form_data.author_id
            )

            session.add(news)
            session.commit()
            session.refresh(news)

            return news

        except ValidationError as err:
            session.rollback()
            return str(err)


news_data = {
    # "title": 4545454654,  # Неверный тип
    "title": "TEST TITLE222",
    # "content": 464646464644646,  # Неверный тип
    "content": "TEST CONTENT222",
    "author_id": 2
}

print(
    create_news(news_data)
)
```

```python
def update_news_by_id(news_id: int, data: dict):
    with DBConnection(db_url=DB_URL) as session:
        try:
            news_to_update = session.query(News).filter(
                News.id == news_id
            ).one()

            news_form_data = NewsUpdate(**data)

            news_to_update.moderated = news_form_data.moderated
            news_to_update.updated_at = datetime.now()

            session.commit()
            session.refresh(news_to_update)

            return news_to_update

        except NoResultFound as err:
            return str(err)

        except ValidationError as err:
            session.rollback()
            return str(err)


news_data = {
    # "moderated": "NEW TEST TITLE FOR UPDATED NEWS",
    "moderated": True,
}

print(
    update_news_by_id(news_id=1, data=news_data)
)
```

```python
def get_news_by_id(news_id: int):
    with DBConnection(db_url=DB_URL) as session:
        try:
            news = session.query(News).filter(
                News.id == news_id
            ).one()

            news_form_data = {
                key: getattr(
                    news,
                    key
                ) for key in NewsRead.model_fields.keys() if hasattr(
                    news,
                    key
                )
            }

            validated_news = NewsRead(**news_form_data)

            return validated_news

        except NoResultFound as err:
            return str(err)

        except ValidationError as err:
            session.rollback()
            return str(err)


def represent_news_data_to_json(data: NewsRead):
    news_data = {
        "id": data.id,
        "title": data.title,
        "content": data.content,
        "author_id": data.author_id,
        "moderated": data.moderated,
        "deleted": data.deleted,
        "created_at": datetime.strftime(
            data.created_at,
            "%y-%m-%d %H:%M:%S"
        ),
    }

    json_data = json.dumps(news_data, indent=4)

    return json_data


print(
    represent_news_data_to_json(
        data=get_news_by_id(
            news_id=1
        )
    )
)
```

И остаются комментарии, сперва контракты:                                                                     

```python
from datetime import datetime

from pydantic import (
    BaseModel
)

from pydantic.types import StringConstraints

from typing import (
    Annotated,
    Optional,
)


class CommentCreate(BaseModel):
    content: Annotated[str, StringConstraints(max_length=1500)]
    author_id: int
    news_id: int


class CommentRead(BaseModel):
    id: int
    content: str
    author_id: int
    news_id: int
    deleted: bool
    created_at: datetime
    updated_at: Optional[datetime] = None
    deleted_at: Optional[datetime] = None


class CommentUpdate(BaseModel):
    content: Optional[Annotated[str, StringConstraints(max_length=1500)]] = None
    deleted: Optional[bool] = None

```

```python
def create_comment(data: dict):
    with DBConnection(db_url=DB_URL) as session:
        try:
            comment_form_data = CommentCreate(**data)

            comment = Comment(
                content=comment_form_data.content,
                author_id=comment_form_data.author_id,
                news_id=comment_form_data.news_id
            )

            session.add(comment)
            session.commit()
            session.refresh(comment)

            return comment

        except ValidationError as err:
            session.rollback()
            return str(err)


comment_data = {
    # "content": 464646464644646,  # Неверный тип
    "content": "TEST CONTENT222",
    "author_id": 2,
    # "news_id": None
    "news_id": 1
}

print(
    create_comment(comment_data)
)
```

```python
def update_comment_by_id(comment_id: int, data: dict):
    with DBConnection(db_url=DB_URL) as session:
        try:
            comment_to_update = session.query(Comment).filter(
                Comment.id == comment_id
            ).one()

            comment_form_data = CommentUpdate(**data)

            comment_to_update.content = comment_form_data.content
            comment_to_update.updated_at = datetime.now()

            session.commit()
            session.refresh(comment_to_update)

            return comment_to_update

        except NoResultFound as err:
            return str(err)

        except ValidationError as err:
            session.rollback()
            return str(err)

        except IntegrityError as err:
            session.rollback()
            return str(err)


comment_data = {
    "content": "NEW TEST COMMENT FOR MODERATED NEWS",
    # "content": None,  # алярма
}

print(
    update_comment_by_id(comment_id=1, data=comment_data)
)
```

```python
def get_comment_by_id(comment_id: int):
    with DBConnection(db_url=DB_URL) as session:
        try:
            comment = session.query(Comment).filter(
                Comment.id == comment_id
            ).one()

            comment_form_data = {
                key: getattr(
                    comment,
                    key
                ) for key in CommentRead.model_fields.keys() if hasattr(
                    comment,
                    key
                )
            }

            validated_comment = CommentRead(**comment_form_data)

            return validated_comment

        except NoResultFound as err:
            return str(err)

        except ValidationError as err:
            session.rollback()
            return str(err)


def represent_comment_data_to_json(data: CommentRead):
    comment_data = {
        "id": data.id,
        "content": data.content,
        "author_id": data.author_id,
        "news_id": data.news_id,
        "deleted": data.deleted,
        "created_at": datetime.strftime(
            data.created_at,
            "%y-%m-%d %H:%M:%S"
        ),
    }

    json_data = json.dumps(comment_data, indent=4)

    return json_data


print(
    represent_comment_data_to_json(
        data=get_comment_by_id(
            comment_id=1
        )
    )
)
```

Для того, чтобы получить возможность отображать определённые данные из таблице                                                  
вместе со смежными полями, которые ссылаются на другие таблицы, необъодимо                                                      
логику переписать немного иначе.                                                         


Контракт на получение информации о пользователе может быть обновлён следующим                                                         
образом:                                                             

Мы создадим новый контракт, для возможности отображать и доп данные. Это можно делать                                                    
для того, чтобы оставить возможность получать информацию только о пользователе,                                                     
а так же и дополнительно доп информацию, разграничивая при этом получение различных                                                   
форматов данных от одной и той же сущности.                                                           

```python
class UserNewsRead(BaseModel):
    id: int
    first_name: str
    last_name: Optional[str] = None
    phone: Optional[str] = None
    email: str
    role_id: int
    rating: float
    created_at: datetime
    updated_at: Optional[datetime] = None
    deleted_at: Optional[datetime] = None

    news: Optional[List[NewsRead]] = []
```

Дальше напишем функцию для получения объекта пользователя и его новостей:                                                 

```python
def get_user_by_id(user_id: int):
    with DBConnection(db_url=DB_URL) as session:
        try:
            user = session.query(User).options(
                joinedload(User.news),
            ).filter(User.id == user_id).one()

            user_data = {
                **{
                    key: getattr(
                        user, key
                    ) for key in UserNewsRead.model_fields if hasattr(user, key)
                },
            }

            validated_user = UserNewsRead(**user_data)

            return validated_user

        except NoResultFound:
            return None

        except ValidationError as err:
            session.rollback()
            return str(err)
```

Дальше нам нужно допом выгрузить ещё и новости все, но так прсото мы это                                                   
не сделаем, мало будет просто указать что-то, так как у нас будет ожидаемый                                                      
тип данных - или словарь данных, или объект типа `NewsRead()`, поэтому данные                                                    
мы должны преобразовать. Можем разделить эту логику более дробно по функциям:                                                      


```python
def convert_news_to_news_read(news):
    news_read = NewsRead(
        id=news.id,
        title=news.title,
        content=news.content,
        author_id=news.author_id,
        moderated=news.moderated,
        deleted=news.deleted,
        created_at=news.created_at,
        updated_at=news.updated_at,
    )

    return news_read
```

И уже потом вызывать эту функцию в нужном нам месте. В нашем случае это                                                       
момент сбора и валидации полного пакета данных о пользователе:                                                                  


```python
def get_user_by_id(user_id: int):
    with DBConnection(db_url=DB_URL) as session:
        try:
            user = session.query(User).options(
                joinedload(User.news),
            ).filter(User.id == user_id).one()

            news_data = [convert_news_to_news_read(n) for n in user.news]  # NEW

            user_data = {
                **{
                    key: getattr(
                        user, key
                    ) for key in UserNewsRead.model_fields if hasattr(user, key)
                },
                "news": news_data,  #  NEW
            }

            validated_user = UserNewsRead(**user_data)

            return validated_user

        except NoResultFound:
            return None

        except ValidationError as err:
            session.rollback()
            return str(err)
```

И после этого мы можем отдельно написать такую функцию, которая будет                                                            
сереализовать (конвертировать python объекты в `json` строку для передачи                                                         
фронту) полученные данные:                                                                     

```python
def represent_json_user(user: UserNewsRead):
    if user is None:
        return "User not found or validation error occurred"

    user_data = {
        "id": user.id,
        "first_name": user.first_name,
        "last_name": user.last_name,
        "phone": user.phone,
        "email": user.email,
        "role_id": user.role_id,
        "rating": user.rating,
        "news": [
            {
                "id": n.id,
                "title": n.title,
                "content": n.content,
                "moderated": n.moderated,
                "created_at": datetime.strftime(
                    n.created_at,
                    "%y-%m-%d %H:%M:%S"
                ),
                "updated_at": datetime.strftime(
                    n.updated_at,
                    "%y-%m-%d %H:%M:%S"
                ) if n.updated_at else None,
            } for n in user.news
        ],
        "created_at": datetime.strftime(
            user.created_at,
            "%y-%m-%d %H:%M:%S"
        ),
        "updated_at": datetime.strftime(
            user.updated_at,
            "%y-%m-%d %H:%M:%S"
        ) if user.updated_at else None,
    }

    user_json = json.dumps(user_data, indent=4)

    return user_json


print(
    represent_json_user(
        get_user_by_id(user_id=2)
    )
)
```


Валидация данных - достаточно полезный инструмент в разработке. С ним                                                         
вы можете точно настроить отправку и получения данных, чтобы утвердить                                                       
конкретный формат, стиль и типы данных, которые должны как-то учавствовать                                                 
в процессе разработки (всё, что присылает пользователь, отправляете ему вы и т.д.)                                                 
