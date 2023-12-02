# **Classmethod decorator, Encapsulation, @property**                                                  

# **Classmethod**                                     

**Что такое методы класса?**    

**Методы класса** - это функции, определенные внутри класса, которые                                         
имеют доступ к атрибутам и методам самого класса, а не к конкретным                                             
экземплярам этого класса. Эти методы могут быть вызваны на уровне класса,                                           
а не только на уровне экземпляра, и они обычно выполняют операции,                                          
связанные с классом в целом, а не с конкретными объектами.                                                   

**Определение методов класса**                                               
**Методы класса** определяются внутри класса и обозначаются декоратором `@classmethod`.                                                           

Декоратор `@classmethod` в **Python** применяется для методов класса и меняет поведение                                  
метода так, чтобы он принимал первым параметром сам класс (как правило, обозначается                                   
как `cls`), а не экземпляр класса. Это значит, что метод, объявленный как классовый,                                     
связан с самим классом, а не с каким-то одним объектом класса.                                               

В отличие от динамических методов классов (**методов экземпляра**), методы класса                                     
**не требуют создания экземпляра класса** для вызова. Они могут вызываться напрямую                                           
на самом классе.                                        

```python
class SuperClass:
    some_static_argument = "Hello from Class"

    @classmethod
    def get_class_argument(cls):
        return cls.some_static_argument

a = SuperClass()

print(f"Via class instance - {a.get_class_argument()}")
print(f"Via class - {SuperClass.get_class_argument()}")
```


# **How and when to use @classmethod?**                                                                     

1) **Фабричные методы:**                                            
Когда нужно предоставить дополнительные конструкторы.                                         
Методы класса могут использоваться для создания фабричных методов                        
и альтернативных конструкторов.                                                                  
Они облегчают доступ к классовым атрибутам и поведению, независимо от экземпляров.                                              

Дело в том, что наши классы позволяют в себе хранить только один метод `__init__()`                              
То есть для одного класса только один инициализатор.                          

**В чём может быть проблема?**                              

Иногда наши классы могут быть определены(инициализированы) по разному.                              
Может быть `n` способов вполне себе валидных способов создания объекта                               
класса. И для всех этих возможных способов **ВСЕГО ОДИН** инициализатор                                       
может не подойти - потому что это будет один инициализатор с кучей параметров,                               
большая часть из которых будет установлена по умолчанию.                             

Такие инициализаторы становятся перегруженными, непонятными, их становится крайне                             
трудно читать и понимать.                       

Помимо этого у нашего инициализатора нет имени. Когда мы создаём объект по классу                               
мы просто прописываем название класса и просто передаём параметры в                                   
скобках. Иногда такого синтаксиса недостаточно, чтобы писать "самодокументированный"                                
код (код выразительный, который понятен просто по названию метода, в котором он                                  
содержится, выражает намерения.)                                  

В таких случаях мы можем прибегнуть к нашим методам класса, которые позволят нам                             
обыграть это дело.                             

Плюс тут в том, что у методов класса есть имя, и оно может быть вполне описательным                               
Благодаря этому, создавая новый объект класса, используя такой метод, мы                              
можем писать более красивый, понятный код, ведь когда мы вызываем такой метод                          
нам сразу понятен смысл создания такого объекта класса.                              

```python
margherita_ingredients = ["dought", "tomatoes sause", "mozzarella", "tomatoes", "oregano", "olive oil"]
prosciutto_ingredients = ["dought", "tomatoes sause", "parmedjano", "olives", "oregano", "basil", "prosciutto", "olive oil"]

class Pizza:

    def __init__(self, *args):
        print("Creating a pizza!!")
        self.ingredients = args

    @classmethod
    def margherita(cls, ingredients):
        return cls(*ingredients)

    @classmethod
    def prosciutto(cls, ingredients):
        return cls(*ingredients)

margherita = Pizza.margherita(margherita_ingredients)
print(margherita.ingredients)

prosciutto = Pizza.prosciutto(prosciutto_ingredients)
print(prosciutto.ingredients)
```

или же, допустим, со словарём, чтобы сразу было видно и вес для каждого продукта:                                       

```python
margherita_ingredients = {
    "dought": 450,
    "tomatoes sause": 150,
    "mozzarella": 150,
    "tomatoes": 137,
    "oregano": 3,
    "olive oil": 10
}
prosciutto_ingredients = {
    "dought": 400,
    "tomatoes sause": 120,
    "parmedjano": 100,
    "olives": 25,
    "oregano": 3,
    "basil": 15,
    "prosciutto": 200,
    "olive oil": 10
}

class Pizza:
    def __init__(self, **kwargs):
        print("Creating a pizza!!!")
        self.ingredients = kwargs

    @classmethod
    def margherita(cls, ingredients):
        return cls(**ingredients)

    @classmethod
    def prosciutto(cls, ingredients):
        return cls(**ingredients)

    def show_ingredients(self):
        for key, value in self.ingredients.items():
            print(f"ingredient '{key}' - {value} gr.")

        print(f"gross weight = {sum(self.ingredients.values())}")


margherita = Pizza.margherita(margherita_ingredients)
print(margherita.ingredients)
margherita.show_ingredients()

prosciutto = Pizza.prosciutto(prosciutto_ingredients)
print(prosciutto.ingredients)
prosciutto.show_ingredients()
```

2) **Методы, зависящие от класса, но не от экземпляра:**                                       
К примеру, методы, которые могут изменять состояние класса, что отразится на                                     
всех экземплярах.                                          

```python
class User:
    active_users = 0

    @classmethod
    def display_active_users(cls):
        return f"Active users: {cls.active_users}"

    @classmethod
    def from_string(cls, data_string):
        first_name, last_name, age = data_string.split(', ')
        return cls(first_name, last_name, int(age))
    
    def __init__(self, first_name, last_name, age):
        self.first_name = first_name
        self.last_name = last_name
        self.age = age
        User.active_users += 1

    def logout(self):
        print("Logging out.")
        User.active_users -= 1


from_string_input = input("Enter your name, surname, and age: ")


user1 = User.from_string(from_string_input)
print(User.display_active_users())

user2 = User(
    input("Enter your name: "),
    input("Enter your surname: "),
    int(input("Enter your age: "))
)
print(User.display_active_users())

user1.logout()
print(User.display_active_users())
```

# **What do I need to keep in mind when working with @classmethod?**                            

* `cls` это ссылка на класс, а не на объект. Используйте это, чтобы обращаться                                         
к атрибутам класса или к другим методам класса.                                        
* Классовые методы не требуют создания экземпляра класса для их вызова.                                        
* Они имеют доступ к состоянию класса, что означает, что они могут читать и                                         
изменять атрибуты класса и другие классовые методы.                                                  


# **How else can @classmethod be used?**                                          

* **Наследование и расширение классов:**
Классовые методы могут быть удобны в наследовании, когда вы хотите, чтобы методы                                   
класса-потомка возвращали экземпляры этого потомка, а не базового класса.                              

* **Вспомогательные функции:**
Они могут использоваться для создания вспомогательных функций, которые могут                                 
выполнять операции, связанные с классом, но не с каким-либо конкретным экземпляром.                                  

* **Альтернативные конструкторы:**                                                
В случаях, когда вам нужно создавать экземпляры класса из разных источников или                                     
форматов данных, например, из файлов JSON, CSV или из результатов API.                                       

* **Модификация класса или его состояния:**
Если вам нужно изменить состояние всего класса, например, обновить кэш или изменить                                   
конфигурацию, которая должна быть общей для всех экземпляров.                                         

* **Полиморфизм без наследования:**
@classmethod позволяет классам использовать полиморфизм без строгой необходимости                                    
наследования, предоставляя различные реализации классовых методов.                                           

* **Логгирование или отслеживание создания экземпляров:**                                      
Если вы хотите отслеживать, сколько раз был создан экземпляр класса или                                   
логировать каждое создание.                               

```python
class User:
    count = 0

    def __init__(self, username):
        self.username = username
        User.count += 1
        self.log_creation()

    @classmethod
    def log_creation(cls):
        print(f"Created {cls.count} instance(s) of {cls.__name__}")


for i in range(1, 11):
    User('user_{i}')
```

* **Создание синглтонов:**                                   
Использование @classmethod может помочь в реализации паттерна синглтон,                                      
где @classmethod будет управлять созданием и доступом к единственному экземпляру класса.                                       

```python
import sqlite3

class Database:
    _instance = None

    def __init__(self):
        self.connection = sqlite3.connect(':memory:')
        self.cursor = self.connection.cursor()
        # Инициализация базы данных, например, создание таблиц.

    @classmethod
    def get_instance(cls):
        if cls._instance is None:
            cls._instance = cls()
        return cls._instance

    def execute(self, query, params=None):
        if params is None:
            self.cursor.execute(query)
        else:
            self.cursor.execute(query, params)
        return self.cursor

    def commit(self):
        self.connection.commit()

    def close(self):
        self.connection.close()

db_instance1 = Database.get_instance()
db_instance2 = Database.get_instance()

print(f"Обе переменные ссылаются на один и тот же объект класса? - {db_instance1 is db_instance2}")


def create_table_if_not_exists(table_name):
    return f"CREATE TABLE IF NOT EXISTS {table_name} (id INTEGER PRIMARY KEY, data TEXT)"


def insert_data_in_table(table_name, user_text):
    return f"INSERT INTO {table_name} (data) VALUES (?)", (user_text, )


def get_all_data_from_the_table(table_name):
    return f"SELECT * FROM {table_name}"


user_text = input("Enter some text: ")


db_instance1.execute(create_table_if_not_exists("example"))

for i in range(6):
    query, params = insert_data_in_table("example", f"{user_text}_{i}")
    db_instance1.execute(query, params)
    db_instance1.commit()

for row in db_instance2.execute(get_all_data_from_the_table("example")):
    print(row)
# HELLO FROM THE DATABASE IN MEMORY
```

# **Encapsulation**

**Инкапсуляция** – это один из четырех основных принципов ООП (наряду с наследованием,                               
полиморфизмом и абстракцией), который позволяет скрывать внутреннее состояние объекта                                         
от прямого доступа извне и контролировать способ взаимодействия с этим состоянием.                                       

# **Encapsulation basics**

В **Python** инкапсуляция достигается за счёт использования модификаторов доступа                                   
для методов и атрибутов класса. Их три:                                            


* `Public` **(публичные)**: могут быть доступны из любой точки программы.                                  
* `Protected` **(защищённые)**: предполагается, что они не будут доступны                                     
извне, за исключением класса-наследника.                                          
* `Private` **(приватные)**: не должны быть доступны извне класса ни при                                  
каких обстоятельствах.                                        

**Как это выглядит на практике???**                                      

```python
class Account:
    def __init__(self, name, balance):
        self._name = name          # Protected attribute
        self.__balance = balance   # Private attribute

    def cache_operation(self, operation, amount):
        return self.__update_balance(operation, amount)

    def __update_balance(self, operation, amount):    # Private method
        match operation.strip().lower():
            case "withdraw":
                if amount > 0 and amount <= self.__balance:
                    self.__balance -= amount
                else:
                    return "Недостаточно средств на счетe"
            case "deposit":
                self.__balance += amount
            case _:
                return "Операция не распознана."
        return f"Баланс обновлен. Новый баланс: {self.__balance}"


my_acc = Account("Card", 1000)

print(my_acc.cache_operation(
    operation=input("Enter the operation you needed [withdraw, deposit]: "),
    amount=int(input("Enter the amount of cash: "))
))
```

* `_name` – это защищенный атрибут, к нему можно обращаться внутри класса и в                                            
классах-наследниках. Соглашение о наименовании подразумевает, что этот атрибут                                      
не следует использовать снаружи класса.                                               
* `__balance` – это приватный атрибут, к нему можно получить доступ только                                 
внутри класса **Account**.                                      
* `__update_balance()` – это приватный метод, который нельзя вызвать на                                     
экземпляре класса за его пределами.                                       


## **Why it's important?**                                   

**Инкапсуляция позволяет:**

* Скрывать детали реализации.                                  
* Предотвратить непреднамеренное воздействие на внутренние                                  
процессы объекта.                                        
* Обеспечить точки расширения класса, где можно безопасно изменять                                
поведение без воздействия на клиентский код.                                        
* Обеспечить контроль над данными, например, проверять, что баланс                                   
аккаунта никогда не уходит в минус.                                      


# **How "private" are the private attributes?**                                

Так себе )))))))))))))))))))

Важно понимать, что в **Python** приватные атрибуты не являются действительно                                  
приватными в сравнении с некоторыми другими языками, такими как `Java` или `C#`                                     
В **Python**, если очень захотеть, можно получить доступ к приватным атрибутам через                                     
специальное именование `<object>._<ClassName>__<attribute_name>`. Это сделано для гибкости,                               
но требует от разработчика соблюдения соглашений о доступе к атрибутам.                                       


# **Is it ethical to use private attributes and methods outside a class?**                    

В сообществе **Python** существует прочно укоренившееся соглашение о том, что атрибуты                            
или методы, которые начинаются с одного или двух подчеркиваний, являются не                                   
предназначенными для внешнего использования. Использование приватных методов                             
класса напрямую считается нарушением этих негласных правил.                                  

Использовать приватные методы напрямую из другого класса или модуля – это                                
плохая практика по нескольким причинам:                                             

1) `Нарушение абстракции:` Приватные методы являются частью внутренней реализации                                     
класса и могут быть изменены или удалены разработчиком класса без предупреждения,                                          
поскольку предполагается, что они не используются напрямую.                                       

2) `Трудности поддержки`: Код, который зависит от внутренних методов других классов,                                    
сложнее поддерживать, поскольку любые изменения в приватных методах могут                                        
неожиданно сломать внешний код.                                           

3) `Отсутствие контракта`: Приватные методы не предназначены для взаимодействия с внешним                                            
миром, следовательно, они не обеспечивают никакого "контракта" или обещания стабильного                                           
поведения, на которое можно было бы полагаться.                                           

4) `Проблемы с безопасностью`: В случае с классами, предоставляющими доступ к ресурсам                                            
(как например, класс для работы с базой данных), прямой вызов приватных методов может                                           
привести к нарушению безопасности, поскольку эти методы могут обходить важные                                            
проверки доступа или валидацию данных.                                           


# **@property**


Декоратор `@property` в **Python** — это встроенный декоратор, который позволяет классу                                 
обрабатывать методы как атрибуты. Он используется для создания свойств объекта,                                 
через которые можно управлять доступом к атрибутам класса.                                             

**А навошта...?**                               

Когда мы создаем класс, иногда нам нужно, чтобы при чтении или изменении                                 
атрибута выполнялся определенный код.                                      
Например, проверять значение или модифицировать его каким-то образом. `@property` позволяет                              
нам это сделать, не изменяя интерфейс класса.


```python
class Publication:
    def __init__(self, title, score):
        self.title = title
        self.score = score


class Author:
    def __init__(self, name):
        self.name = name
        self.publications = []

    def add_publication(self, publication):
        self.publications.append(publication)

    @property
    def rating(self):
        total_score = sum(pub.score for pub in self.publications)
        result = round(total_score / len(self.publications) if self.publications else 0, 2)
        return result

author = Author("Джон Доу")

author.add_publication(Publication("Статья 1", 5))
author.add_publication(Publication("Статья 2", 8))
author.add_publication(Publication("Статья 3", 7))

print(author.rating)


new_author = Author("Adam S")

print(new_author.rating)
```


# **Home task**

### **Общее описание**
Разработать небольшую банковскую систему на **Python** с использованием парадигмы                                   
объектно-ориентированного программирования (ООП). Система должна позволять                                    
создавать пользователей, производить вход по логину и паролю, менять информацию                                           
пользователя в его аккаунте.                                           
Так же каждый пользователь может создать свой банковский счёт.                                              
Можно создать счёт, просматривать его баланс, пополнять счёт, снимать средства,                                         
переводить другому пользователю.                                                         

### **Регистрация и авторизация пользователей**

Система должна обеспечивать возможность регистрации пользователей с последующим                                   
входом в систему.                                                          

### **Реализация пользователей**                               

**Каждый пользователь должен иметь следующие поля:**

* `ID`
* `Имя`
* `Фамилия`
* `Телефон`
* `Email`
* `Пароль`
* `Повтор пароля`
* `Дата создания (created_at)`
* `Дата обновления (updated_at)`
* `Дата удаления (deleted_at)`


### **Реализация банк аккаунтов для пользователя**                               

**У аккаунта должны быть след поля:**

* `ID`
* `Номер счёта`
* `ID пользователя, которому пренадлежит счёт`
* `баланс`
* `Дата создания счёта`
* `Дата обновления счёта`
* `Поле, которое будет содержать среднее значение денежных затрат`

#### **Хранение данных**

**Все данные должны храниться в CSV файлах:**                                        

* `Пользователи (users.csv)`
* `ATM (atm.csv)`

#### **Операции с данными**                     


**Необходимо реализовать следующие операции:**

* **Сохранение новых данных**
* **Получение существующих данных**
* **Обновление данных**
* **Удаление данных**

#### **Tip menu**

Для каждого пользователя должно быть реализовано свое меню, которое                                       
должно содержать следующие пункты:                                    
1) User:                                             
* создать аккаунт                                        
* редактировать аккаунт                                        
* удалить аккаунт                                             
* посмотреть данные об аккаунте                                             
* создать счёт
* посмотреть баналс на счёте
* пополнить баланс
* снять деньги со счёта
* отправить деньги другому пользователю

##### **Ограничения и требования**

Все операции чтения/записи данных должны быть реализованы с использованием библиотеки `CSV`.                              
Для организации кода следует использовать классы и наследование, соответствующие принципам `ООП`.                           

* В системе должна быть реализована простая система  авторизации пользователей.                               
* Пароль не должен храниться в открытом виде, его необходимо хэшировать.                                     


##### **Тестирование**                                          
Должны быть предоставлены примеры использования системы, демонстрирующие                                        
работу всех функций для разных типов пользователей.                                  

##### **Документация**
Необходимо предоставить документацию по коду, включая описание классов и методов,                                  
а также инструкции по запуску и тестированию системы.                                   
