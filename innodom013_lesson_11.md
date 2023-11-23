<!-- TOC -->
* [**Nested functions, Enclosures, Decorators**](#nested-functions-enclosures-decorators)
* [**Nested functions**](#nested-functions)
* [**Scope of variables**](#scope-of-variables-)
* [**Closures**](#closures-)
* [**Closures - examples**](#closures---examples-)
* [**Lambdas with closures**](#lambdas-with-closures)
* [**Closures in real code**](#closures-in-real-code)
* [**memoization through closures**](#memoization-through-closures)
* [**Decorators**](#decorators)
* [**Decorators factory**](#decorators-factory-)
* [**Stacking decorators**](#stacking-decorators-)
* [**Home Tasks**](#home-tasks-)
<!-- TOC -->

# **Nested functions, Enclosures, Decorators**

# **Nested functions**

**Вложенные функции** - это функции, которые находятся друг в друге.                                                 
Такие функции могут быть использованы только внутри родительской.                                                                               
Благодаря этому можно добиться взаимодействия между функциями                                                                               
вместе с их общими аргументами.                                                 

```python
def greeting_func(lang: str) -> None:
    def eng_greeting() -> None:
        print("Hello! I'm glad to meet with you! Wa're using English!")

    def rus_greeting() -> None:
        print("Привет! Рад приветствовать тебя! Мы используем русский язык!")

    match lang.strip().lower():
        case "eng":
            eng_greeting()
        case "rus":
            rus_greeting()
        case _:
            print("Совпадений нет.")


greeting_func("Rus")
```

# **Scope of variables**                                                 

Все переменные, которые мы объявляем в функции, включая её                                                                               
аргументы, являются локальными, а это значит, что обратиться                                                                                 
к ним можно только внутри функции, за функцией они не существуют.                                                 

Глобальные переменные объявляются внутри функции и работают, а                                                                                
также хранят значения полученные в функции за ней, учтите что                                                                                
использовать такие переменные не всегда безопасно.                                                 

**Всего есть 4 области видимости переменных:**                                                 

`Build-in` - уровень сборки вашего питона                                                                               
`Уровень файла` (глобальные переменные)                                                                                 
`Локальные переменные` (в блоке кода)                                                                                  
`Замкнутые переменные` переменные, закрытые внутри функции                                                                                        


```python
greeting = "Hello from the global scope"


def outer():
    greeting = "Hello from the local scope"
    print(f"local: {id(greeting)}")
    def inner():
        greeting = "Hello from the enclosing"
        print(f"enclosing: {id(greeting)}")
        print(greeting)
    return inner


print(greeting)
print(f"global: {id(greeting)}")
outer()()
```

**что тут происходит:**                                                                     

1) **лобальная область видимости**: Переменная `greeting` определена в глобальной области                                                                     
видимости. Когда вы вызываете `print(greeting)` вне каких-либо функций, `Python`                                                                      
использует значение этой глобальной переменной, и вывод будет                                                                      
**"Hello from the global scope"**.                                                                     

2) **Локальная область видимости**: Внутри функции `outer()`, переменная `greeting`                                                                      
**переопределяется**. Это новая переменная, которая существует только в локальной                                                                      
области видимости функции `outer()`. Однако, поскольку `print(greeting)` не                                                                     
вызывается внутри `outer()`, это переопределение не влияет на вывод программы.                                                                     

3) **Замыкающая область видимости**: Внутри вложенной функции `inner()`, переменная                                                                     
`greeting` снова **переопределяется**. Эта переменная находится в **замыкающей области**                                                                      
**видимости** относительно функции `inner()`. Когда `print(greeting)` вызывается внутри                                                                      
`inner()`, используется это значение, и вывод будет **"Hello from the enclosing"**.                                                                     

**Когда вы вызываете** `outer()()`, **происходит следующее**:                                                                     

`outer()` вызывается, создавая и возвращая функцию `inner`.                                                                     
Затем вызывается `inner()`, которая выводит значение `greeting` из своей                                                                      
замыкающей области видимости.                                                                     


---

# **Closures**                                                 

**Замыкание** - это функция, которая сохраняет доступ к переменным                                                                        
из окружающей ее области видимости, даже после того, как эта                                                                           
область видимости завершила свою работу.                                                 

Другими словами, **замыкание** - функция, которая имеет внутри себя другую,                                                                        
вложенную функцию, и возвращает её. Вложенная функция имеет доступ к                                                                      
атрибутам и переменным внешней функции.                                                                 


**Замыкания** (`closures`) в Python создаются путем **сохранения ссылки на переменные**                                                                        
из внешней функции (родителя) внутри вложенной функции. Это позволяет                                                                              
вложенной функции **сохранить доступ** к этим переменным **даже после завершения**                                                                             
выполнения внешней функции. Этот механизм позволяет вам создавать функции,                                                                           
которые "**запоминают**" значения переменных из окружающей области видимости.                                                                         


```python
def outer_function(x):
   # outer func with `x` argument

   def inner_function(y):
      # inner function with `y` argument
      # this function has access to the local
      # environment of the external function
      return x * y
   return inner_function

closure = outer_function(10)
result = closure(5)
print(result)
```

Суть заключается в том, что **Python** сохраняет окружение (включая переменные)                                                                            
внешней функции внутри замыкания, что позволяет замыканию обращаться к                                                                            
этим переменным даже после завершения выполнения внешней функции. Этот                                                                                
механизм очень полезен при создании функций, которые могут сохранять                                                                              
состояние и контекст для последующих вызовов.                                                                         

# **Closures - examples**                                                 

Замыкания можно использовать для создания счетчика функций, который будет                                                                      
отслеживать, сколько раз функция была вызвана. За счёт этого вы можете                                                                            
отследить количество вызовов ваших функций, что может помочь при                                                                                
оптимизации кода.                                                 

В таком подходе может понадобиться дополнительное ключевое слово в питоне,                                                                        
`nonlocal` для указания на то, что переменная не является локальной для                                                                           
вложенной функции, а должна быть взята из ближайшей области видимости,                                                                           
которая является областью видимости внешней функции.                                                                                          


**nonlocal** — это ключевое слово в **Python**, которое позволяет нам указать,                                            
что определенная переменная относится к ближайшей внешней (не глобальной)                                              
области видимости. Это особенно полезно в замыканиях, когда вам нужно                                               
изменить значение переменной из внешней функции.                                                  

**nonlocal** обычно требуется, когда вы работаете с **неизменяемыми** типами данных,                                               
такими как **числа**, **строки** и **кортежи**. Это потому, что **неизменяемые** типы **не могут**                                              
быть изменены напрямую, и чтобы изменить их значение, вам нужно присвоить                                               
им новое значение.                                              

Когда вы работаете с **изменяемыми** типами данных, такими как **списки** или **словари**,                                               
вы можете изменять их содержимое **без** использования **nonlocal**. Однако если бы вы                                               
хотели переопределить весь список или словарь внутри внутренней функции, вам                                               
бы потребовалось использовать **nonlocal**.                                              

```python
def counter():
   # Внешняя функция, инициализирует счетчик
   count = 0

   def inner_function():
      # Вложенная функция, увеличивает счетчик и возвращает его значение
      nonlocal count # Объявляем count как nonlocal, чтобы изменять значение из вложенной функции
      count += 1
      return count

   return inner_function

my_counter = counter()

print(my_counter)
print(my_counter)
print(my_counter)
print(my_counter)
```

Когда мы вызываем **counter()**, она возвращает `inner_function()`, которая                                                                           
замыкает (сохраняет) переменную `count` из внешней функции `counter()`. Таким                                                                         
образом, каждый раз, когда мы вызываем `my_counter()`, мы работаем с той                                                                          
же самой переменной `count`, которая сохраняется между вызовами функции.                                                                           

```python
def create_appender():
    lst = []

    def append_item(item):
        lst.append(item)
        return lst

    return append_item

appender = create_appender()
```

```python
def create_resetter():
    lst = [1, 2, 3]

    def reset(new_list):
        nonlocal lst
        lst = new_list
        return lst

    return reset

resetter = create_resetter()
```

```python
def modify_nested_dict():
    data = {
        'user': {
            'name': 'John',
            'age': 30
        },
        'location': 'USA'
    }

    def change_user_name(new_name):
        data['user']['name'] = new_name
        return data

    return change_user_name

changer = modify_nested_dict()
```


замыкания могут быть переданы как аргументы в другие функции                                                                          
или возвращены из них.                                                                        

```python
def multiplier(factor):
    def multiply_by_factor(number):
        return number * factor
    return multiply_by_factor

double = multiplier(2)
triple = multiplier(3)

numbers = [1, 2, 3, 4, 5]
doubled_numbers = list(map(double, numbers))
tripled_numbers = list(map(triple, numbers))
```

Замыкания и функции высшего порядка представляют мощную концепцию в языках                                                                                
программирования, позволяя передавать функции как аргументы в другие функции                                                                                 
или возвращать их из функций. Это делает код более гибким и позволяет                                                                             
работать с функциями как с данными.                                                                                   

таким образом сторонние функции могут использовать замыкание или его внутреннюю                                                                               
функцию для выполнения определенных операций. Это особенно полезно, когда                                                                          
вы хотите передать логику выполнения функции как параметр.


# **Lambdas with closures**

**Лямбда-функции** также могут быть использованы внутри замыканий, что делает                                                                             
замыкания более гибкими и позволяет создавать функции на месте с                                                                    
помощью лямбда-функций.                                                                           

```python
def generate_multiplier(factor):
    return lambda x: x * factor

double = generate_multiplier(2)
triple = generate_multiplier(3)

print(double(5))
print(triple(5))
```

`generate_multiplier` - это функция, которая возвращает лямбда-функцию.                                                                               
**Лямбда-функция** умножает свой аргумент `x` на значение `factor`, которое                                                                             
передается внутрь `generate_multiplier`. Это позволяет создавать разные                                                                                  
функции на месте на основе разных значений `factor`.                                                                                 

Использование лямбда-функций в замыканиях может быть полезным, когда вам                                                                               
**нужно динамически создавать функции с разной логикой** на основе                                                                               
параметров или контекста.                                                                                


# **Closures in real code**

Замыкания могут быть использованы **для обеспечения безопасности данных**                                                                               
путем скрытия переменных и ограничения доступа к ним извне. Этот механизм                                                                               
называется **инкапсуляцией** данных и представляет собой одну из основных                                                                                  
концепций **объектно-ориентированного программирования**.                                                                                   

В **Python**, однако, нет явной поддержки приватных переменных (которые                                                                                
не могут быть доступны извне объекта), но мы можем использовать                                                                             
замыкания для имитации этой функциональности.                                                                                    


```python
def create_secure_account(balance, password):
    # Внешняя функция, скрывающая баланс и пароль

    def get_balance(pwd):
        # Вложенная функция, проверяющая пароль перед возвращением баланса
        if pwd == password:
            return balance
        else:
            return "Неверный пароль."

    def deposit(pwd, amount):
        # Вложенная функция для внесения средств
        nonlocal balance
        if pwd == password:
            balance += amount
            return f"Внесено {amount}. Новый баланс: {balance}"
        else:
            return "Неверный пароль."

    def withdraw(pwd, amount):
        # Вложенная функция для снятия средств
        nonlocal balance
        if pwd == password:
            if amount <= balance:
                balance -= amount
                return f"Снято {amount}. Новый баланс: {balance}"
            else:
                return "Недостаточно средств"
        else:
            return "Неверный пароль."

    return get_balance, deposit, withdraw

# Создаем безопасный счет
get_balance, deposit, withdraw = create_secure_account(1000, "password123")

# Попытка доступа к балансу с правильным паролем
print(get_balance("password123"))  # Выведет 1000

# Попытка доступа к балансу с неправильным паролем
print(get_balance("wrong_password"))  # Выведет "Недопустимый пароль"

# Внесение и снятие средств
print(deposit("password13", 500))
print(deposit("password123", 500))  # Выведет "Внесено 500. Новый баланс: 1500"
print(withdraw("password123", 200))  # Выведет "Снято 200. Новый баланс: 1300"
print(withdraw("password123", 2000))  # Выведет "Недостаточно средств"
```

В этом примере переменные **balance** и **password** инкапсулированы внутри внешней                                                                              
функции `create_secure_account`, и доступ к ним возможен только через                                                                             
вложенные функции `get_balance`, `deposit` и `withdraw`, которые имеют доступ                                                                            
к этим переменным благодаря замыканию. Таким образом, данные остаются                                                                              
скрытыми и безопасными от прямого доступа извне, и доступ к ним                                                                              
осуществляется только через управляемый интерфейс функций.                                                                                         


# **memoization through closures**

**Мемоизация** - это оптимизационная техника в программировании, которая                                                                               
заключается в сохранении результатов выполнения функции для заданных входных                                                                             
данных и последующем использовании сохраненных результатов при повторных                                                                             
вызовах функции с теми же входными данными, вместо выполнения вычислений                                                                             
заново. Это позволяет улучшить производительность функции, особенно в                                                                             
случаях, когда функция вызывается с одними и теми же аргументами                                                                               
несколько раз.                                                                          

Замыкания играют важную роль в реализации мемоизации. С помощью замыканий                                                                                 
можно создать функцию-обертку, которая будет хранить результаты выполнения                                                                            
функции и возвращать их, если входные данные уже были обработаны.                                                                                


# **Decorators**

**Декораторы** - это мощный инструмент в `Python`, который позволяет изменять                                                                                     
поведение функций или методов класса, не изменяя их собственный код. Они                                                                              
представляют собой функции, которые принимают другую функцию или метод в                                                                              
качестве аргумента и возвращают новую функцию или метод, обогащенную                                                                            
дополнительной логикой.                                                 

Декораторы обычно используются для добавления                                                                                
функциональности, логирования, аутентификации, проверки прав доступа и                                                                             
других аспектов, не изменяя исходный код функции или метода.                                                 

Декораторы позволяют разделять код функций и дополнительную логику,                                                                                      
что делает код более читаемым и поддерживаемым.                                                 

**Декораторы предоставляют несколько преимуществ:**                                                 

`Разделение функциональности:` Декораторы позволяют разделить различные                                                                                   
аспекты функциональности, такие как логирование, аутентификация и валидация,                                                                             
от основной логики функции. Это делает код более структурированным и понятным.                                                 

`Многократное использование:` Декораторы могут быть использованы на нескольких                                                                                      
функциях или методах, что упрощает повторное использование кода.                                                 

`Упрощение чтения кода:` Декораторы позволяют добавлять дополнительную                                                                                   
логику к функциям, не засоряя их собственный код. Это улучшает читаемость                                                                                    
кода и делает его более понятным.                                                 

Для определения декоратора, вы должны создать функцию, которая принимает                                                                            
другую функцию в качестве аргумента и возвращает новую функцию, которая                                                                                  
может быть вызвана вместо оригинальной функции. Эта новая функция обычно                                                                             
содержит дополнительную логику перед или после вызова оригинальной функции.                                                 

**Так же декораторы могут работать с функциями с аргументами:**                                                 

```python
def greet(name):
    return f"Привет, {name}"
```

```python
def exclamation_decorator(func):
    def wrapper(name):
        result = func(name)
        return result + ". Nice to meet with you!!!!!"

    return wrapper
```

Декораторы мы можем вызывать не только через спец символ `@`, но и при определении                                                                          
переменной:                                                 

```python
my_dec = exclamation_decorator(greet)
print(my_dec("Vlad"))
```

---

# **Decorators factory**                                                              

Фабрика декораторов в Python - это функция, которая возвращает декоратор.                                                               
Это полезно, когда нам нужно создать декоратор с параметрами, позволяя настраивать                                                               
его поведение. Фабрика декораторов добавляет еще один уровень функций,                                                               
позволяя передавать аргументы в декоратор.                                                              


```python
def decorator_factory(enter_message, exit_message):
    # Это фабрика декораторов
    def decorator(func):
        # Это сам декоратор
        def wrapper(*args, **kwargs):
            # Это обертка, добавляющая дополнительное поведение
            print(enter_message)
            result = func(*args, **kwargs)
            print(exit_message)
            return result
        return wrapper
    return decorator

# Использование фабрики декораторов
@decorator_factory("Начало выполнения функции", "Конец выполнения функции")
def my_function(x):
    print(f"Функция выполняется с аргументом {x}")

my_function(5)
```

```python
from functools import wraps

def role_required(*roles):
    def decorator(view_func):
        @wraps(view_func)
        def _wrapped_view(request, *args, **kwargs):
            if not request.get("user").get("is_authenticated"):
                return {
                    "status_code": 403,
                    "message": "FORBIDDEN. NOT AUTHENTICATED"
                }
            if not request.get("user").get("role") in roles:
                return {
                    "status_code": 403,
                    "message": "FORBIDDEN. DON'T HAVE ACCESS ROLE."
                }
            return view_func(request, *args, **kwargs)
        return _wrapped_view
    return decorator

# Пример использования
@role_required('admin', 'manager')
def some_view(request):
    user_name = request.get("user").get("Name")
    return {
        "status_code": 200,
        "response": f"Hello, {user_name}"
    }


admin_user = {
    "user": {
        "Name": "Vladislav",
        "Surname": "Pareiko",
        "Age": 26,
        "Phone": "45678912654",
        "is_authenticated": True,
        "role": "admin"
    }
}

anon_user = {
    "user": {
        "Name": "Anon",
        "Surname": "",
        "Age": 26,
        "Phone": "111111111111",
        "is_authenticated": False,
        "role": "admin"
    }
}


not_admin_or_manager = {
    "user": {
        "Name": "Anon",
        "Surname": "",
        "Age": 26,
        "Phone": "111111111111",
        "is_authenticated": True,
        "role": "user"
    }
}

# print(some_view(admin_user))
# print(some_view(anon_user))
print(some_view(not_admin_or_manager))
```
---

# **Stacking decorators**                                                 

**Стекирование декораторов** - это процесс применения нескольких декораторов                                                                             
к одной функции или методу. Вы можете создавать сложные цепочки декораторов,                                                                             
чтобы комбинировать различные аспекты функциональности. Порядок,                                                                                
в котором вы стекируете декораторы, важен, и он определяет,                                                                              
какие преобразования будут применены к функции.                                                 

Вложенные декораторы предоставляют мощный способ добавлять функциональность                                                                           
и изменять поведение функций и методов в Python, позволяя создавать                                                                             
гибкий и читаемый код. Они особенно полезны при работе с различными                                                                              
аспектами приложения, такими как логирование, валидация, кеширование                                                                               
и многое другое.                                                 

```python
from functools import wraps

admin_user = {
    "user": {
        "Name": "Vladislav",
        "Surname": "Pareiko",
        "Age": 26,
        "Phone": "45678912654",
        "is_authenticated": True,
        "role": "admin"
    }
}

anon_user = {
    "user": {
        "Name": "Anon",
        "Surname": "",
        "Age": 26,
        "Phone": "111111111111",
        "is_authenticated": False,
        "role": "admin"
    }
}


not_admin_or_manager = {
    "user": {
        "Name": "Anon",
        "Surname": "",
        "Age": 26,
        "Phone": "111111111111",
        "is_authenticated": True,
        "role": "user"
    }
}

# Декоратор для проверки авторизации
def login_required(func):
    def wrapper(request, *args, **kwargs):
        if not request.get("user").get("is_authenticated"):
                return {
                    "status_code": 403,
                    "message": "FORBIDDEN. NOT AUTHENTICATED"
                }
        return func(request, *args, **kwargs)
    return wrapper

# Декоратор для проверки роли
def role_required(role):
    def decorator(view_func):
        @wraps(view_func)
        def _wrapped_view(request, *args, **kwargs):
            if not request.get("user").get("role") == role:
                return {
                    "status_code": 403,
                    "message": "FORBIDDEN. DON'T HAVE ACCESS ROLE."
                }
            return view_func(request, *args, **kwargs)
        return _wrapped_view
    return decorator

# Декоратор для логирования
def log_access(view_func):
    @wraps(view_func)
    def _wrapped_view(request, *args, **kwargs):
        print(f"Accessing view: {view_func.__name__}")
        return view_func(request, *args, **kwargs)
    return _wrapped_view

# Применение стекированных декораторов
@login_required
@role_required('admin')
@log_access
def my_view(request):
    user_name = request.get("user").get("Name")
    status_code = 200
    message = f"Hello, {user_name}!! I'm glad to greeting you!"
    return {
        "status": status_code,
        "message": message
    }

my_view(not_admin_or_manager)
```
---

# **Home Tasks**                                                 

1) Написать функцию, которая возвращает замыкание для                           
вычисления факториала.                      

2) Создать функцию для генерации случайных паролей                         
с заданной длиной, используя замыкания.

3) Написать декоратор на валидацию пароля. Этот декоратор должен                           
применяться к функции, которая пароль запрашивает.                           
Если всё хорошо - выводить сообщение о том, что пароль валиден,                           
в противном случае вывести, что что-то не так. (или кто захочет -                                 
вывести конкретное сообщение, что в пароле не так.)

4) Когда-то давно была задача на расчёт индекса массы-тела.                           
Перепишите эту задачу с использованием замыканий.

5) Напишите декоратор, который будет применяться к функции,                         
выводить информацию типа: какая функция была вызвана, какие                           
у неё были параметры, сколько раз она была вызвана.                                    

6) Напишите декоратор, который замеряет время выполнения вашей                                
функции. Если время выполнения в пределах двух секунд - выводите                          
что всё хорошо. В противном случае сообщение о том, что функция                                  
(её имя) работает слишком медленно, её необходимо оптимизировать.                                           

