# **Create relations between models **                                                 

Связи между моделями создаются объявлением в них полей, формируемых                                                            
особыми классами из того же модуля `django.db.models`                                                   


Существует несколько способов связывания:                                                       
* **ForeignKey** (Один ко многим)                                                                                                  
* **ManyToManyField** (Многие ко многим)                                                 
* **OneToOneField** (Один к одному)                                                 


1) **Связь "один-со-многими"**                                                 

Связь "один-со-многими" связывает одну запись первичной модели с                                                           
произвольным числом записей вторичной модели. Это наиболее часто                                                           
применяемый на практике вид связей. Для создания связи такого типа                                                             
в классе вторичной модели следует объявить поле типа ForeignКey.                                                 

```python
FоrеingКеу(<связываемая первичная модель>,
    оn_dеlеtе=<поведение при удалении записи>, [<остальные параметры>])
```

Первым, позиционным, параметром указывается связываемая первичная модель в виде:                                                          
* непосредственно ссылки на класс модели, если объявление первичной модели                                                           
находится перед объявлением вторичной модели (в которой и создается поле внешнего ключа):                                                        
```python
from django.db import models
from django.contrib.auth.models import User


class Status(models.Model):
    name = models.CharField(max_length=30, unique=True)

    def __str__(self):
        return self.name

    class Meta:
        verbose_name = 'Status'
        verbose_name_plural = 'Statuses'


class Category(models.Model):
    name = models.CharField(
        max_length=25,
        unique=True
    )

    def __str__(self):
        return self.name

    class Meta:
        verbose_name = 'Category'
        verbose_name_plural = 'Categories'


class Task(models.Model):
    title = models.CharField(
        max_length=75,
        default="DEFAULT TITLE",
        unique_for_date='date_started'
    )
    description = models.TextField(
        max_length=1500,
        verbose_name="task details",
        default="Here you can add your description..."
    )
    creator = models.ForeignKey(
        User,
        on_delete=models.CASCADE,
        blank=True,
        null=True
    )
    category = models.ForeignKey(
        Category,
        null=True,
        blank=True,
        on_delete=models.SET_NULL
    )
    status = models.ForeignKey(
        Status,
        on_delete=models.SET(1),
        blank=True,
        null=True
    )
    date_started = models.DateField(
        help_text="День, когда задача должна начаться",
        blank=True,
        null=True
    )
    deadline = models.DateField(
        help_text="День, когда задача должна быть выполнена",
        blank=True,
        null=True
    )
    created_at = models.DateTimeField(
        auto_now_add=True
    )
    updated_at = models.DateTimeField(auto_now=True)
    deleted_at = models.DateTimeField(null=True, blank=True)

    def __str__(self):
        return f"{self.title[:6]}..."

    def count_by_status(self, status_name):
        try:
            status = Status.objects.get(name=status_name)
        except Status.DoesNotExist:
            return 0

        count = Task.objects.filter(status=status).count()

        return count

    def count_by_category(self, category_name):
        try:
            category = Category.objects.get(name=category_name)
        except Category.DoesNotExist:
            return 0

        count = Task.objects.filter(category=category).count()

        return count

    class Meta:
        verbose_name = 'Task'
        verbose_name_plural = 'Tasks'


class SubTask(models.Model):
    title = models.CharField(max_length=75, blank=True)
    description = models.CharField(max_length=1500, blank=True)
    category = models.ForeignKey(
        Category,
        null=True,
        blank=True,
        on_delete=models.SET_NULL
    )
    task = models.ForeignKey(
        Task,
        on_delete=models.CASCADE,
        related_name='subtasks',
        blank=True,
        null=True
    )
    status = models.ForeignKey(
        Status,
        on_delete=models.SET(1),
        blank=True,
        null=True
    )
    creator = models.ForeignKey(
        User,
        on_delete=models.CASCADE,
        blank=True,
        null=True
    )
    date_started = models.DateField(blank=True)
    deadline = models.DateField(blank=True)
    created_at = models.DateTimeField(
        auto_now_add=True
    )
    updated_at = models.DateTimeField(auto_now=True)
    deleted_at = models.DateTimeField(null=True, blank=True)

    def __str__(self):
        return f"{self.title[:6]}..."

    def count_by_status(self, status_name):
        try:
            status = Status.objects.get(name=status_name)
        except Status.DoesNotExist:
            return 0

        count = SubTask.objects.filter(status=status).count()

        return count

    def count_by_category(self, category_name):
        try:
            category = Category.objects.get(name=category_name)
        except Category.DoesNotExist:
            return 0

        count = SubTask.objects.filter(category=category).count()

        return count

    class Meta:
        verbose_name = 'SubTask'
        verbose_name_plural = 'SubTasks'
```

* строки с именем класса, если первичная модель объявлена после вторичной:                                                        
```python
class SubTask(models.Model):
    ...
    task = models.ForeignKey(
    'Task',
    on_delete=models.CASCADE,
    ...
    )
    ...


class Task(models.Model):
    ...
    ...
    ...

```
Ссылка на модель из другого приложения проекта записывается в виде строки формата                                                           
`<имя прwюжения>.<имя класса модели>`                                                 


Вторым параметром `on_delete` указывается поведение фреймворка в случае,                                                           
если будет выполнена попытка удалить запись первичной модели, на которую                                                           
ссылаются какие-либо записи вторичной модели. Параметру присваивается                                                         
значение одной из переменных, объявленных в модуле `django.db.models`:                                                 

* `CASCADE` - удаляет все связанные записи вторичной модели (каскадное удаление).                                                 


* `PROTECT` - возбуждает исключение `ProtectedError` из модуля `django.db.models`,                                                          
тем самым предотвращая удаление записи первичной модели.                                                 


* `SET_NULL` - заносит в поле внешнего ключа всех связанных записей вторичной                                                           
модели значение null. Сработает только в том случае, если поле внешнего ключа                                                           
объявлено необязательным к заполнению на уровне базы данных                                                            
(параметр`null` конструктора поля имеет значение True).                                                 


* `SET_DEFAULT` - заносит в поле внешнего ключа всех связанных                                                            
записей вторичной модели заданное для него значение по умолчанию.                                                           
Сработает только в том случае, если у поля внешнего ключа было                                                          
указано значение по умолчанию (оно задается параметром                                                         
`default` конструктора поля).                                                 


* `SET(<значение>)` - заносит в поле внешнего ключа указанное значение                                                           

```python
status = models.ForeingKey(Status, on_delete=models.SET(1))
```
Также можно указать ссылку на функцию, не принимающую                                                           
параметров и возвращающую значение, которое будет записано в поле:                                                 


```python
def set_default_val():
  return "default value"

...

rubric = models.ForeingKey(Status, on_delete=models.SET(set_default_val)) 
```

* `DO_NOTНING` - ничего не делает.                                                 


```python
"""
ВНИМАНИЕ!
Если СУБД подцерживает межтабличные связи с сохранением ссылочной      
целостности, то попытка удаления записи первичной модели, с которой        
связаны записи вторичной модели, в этом случае все равно не увенчается         
успехом, и будет возбуждено исключение IntegrityError из модуля django.dЬ.models.
"""
```

Полю внешнего ключа рекомендуется давать имя, обозначающее связываемую сущность                                                           
и записанное в единственном числе. Например, для представления рубрики в модели                                                           
`Task` мы объявили много полей, одно из которых - `sub_task`. На                                                                                                           
уровне базы данных поле внешнего ключа модели представляется полем таблицы,                                                                                             
имеющим имя вида `<имя поля внешнего ключа>_id`.                                                           
В веб-форме такое поле будет представляться раскрывающимся списком,                                                           
содержащим строковые представления записей первичной модели.                                                  


Класс `ForeignКey` поддерживает следующие дополнительные необязательные параметры:                                                 

* `limit_choices_to` - позволяет вывести в раскрывающемся списке записей первичной                                                         
модели, отображаемом в веб-форме, только записи, удовлетворяющие заданным                                                       
критериям фильтрации. Критерии фильтрации записываются в виде словаря **Python**,                                                          
имена элементов которого совпадают с именами полей первичной модели,                                                       
по которым должна выполняться фильтрация, а значения элементов укажут значения                                                         
для этих полей. Выведены будут записи, удовлетворяющие всем критериям, заданным в                                                        
таком словаре (т. е. критерии объединяются по правилу логического И). Для примера                                                          
укажем Django выводить только задачи, поле `status` которых содержит значение True:                                                 


```python
task = models.ForeignКey(
    Task,
    on_delete=models.CASCADE,
    null=True,
    blank=True,
    limit_choices_to={'status': 1}
)
```

Если параметр не указан, то список связываемых записей будет включать все записи первичной модели                                                 

* `related_name` - имя атрибута записи первичной модели, предназначенного для доступа к                                                           
связанным записям вторичной модели, в виде строки:                                                 

```python
class SubTask(models.Model):
    ...
    task = models.ForeignКey(
        Task,
        on_delete=models.CASCADE,
        null=True,
        blank=True,
        limit_choices_to={'status': 1},
        related_name='subtasks'
)
# Получаем первую задачу
first_task = Task.objects.first()
# Получаем доступ к связанным подзадачам через атрибут subtasks ,
# указанный в параметре related_name
subtasks = first_task.subtasks.all() 
```

Если доступ из записи первичной модели к связанным записям вторичной модели не требуется,                                                         
можно указать Django не создавать такой атрибут и тем самым немного сэкономить системные                                                            
ресурсы. Для этого достаточно присвоить параметру `related_name` символ "плюс".                                                             
Если параметр не указан, то атрибут такого рода получит стандартное                                                           
имя вида `<имя связанной вторичной модели>_sеt`                                                 


* `to_field` - имя поля первичной модели, по которому будет выполнена связь, в виде строки.                                                          
Такое поле должно быть помечено как уникальное (параметр `unique` конструктора должен                                                         
иметь значение `True`). Если параметр не указан, связывание выполняется по ключевому                                                            
полю первичной модели - неважно, созданному явно или неявно                                                 


* `db_constraint` - если `True`, то в таблице базы данных будет создана связь, позволяющая                                                         
сохранять ссылочную целостность; если `False`, ссылочная целостность будет поддерживаться                                                             
только на уровне `Django`. Значение по умолчанию - `True`. Менять его на `False` имеет смысл,                                                           
только если модель создается на основе уже существующей базы с некорректными данными.                                                 

2) **Связь "многие-со-многими"**                                                 

Связь "многие-со-многими" соединяет произвольное число записей одной модели                                                               
с произвольным числом записей другой (обе модели здесь выступают как равноправные,                                                               
и определить, какая из них первичная, а какая вторичная, не представляется возможным).                                                              
Для создания такой связи нужно объявить в одной из моделей (но не в обеих сразу!)                                                                                                             
поле внешнего ключа типа `ManyToManyField`. Вот формат его конструктора:                                                 

```python
ManyТoManyField(<вторая связываемая модель>, [<остальные параметры>])
```

Первый параметр задается в таком же формате, что и в конструкторах классов                                                         
`ForeignField` и `OneToOneField`.                                                 

Модель, в которой было объявлено поле внешнего ключа, назовем ведущей,                                                       
а вторую модель - ведомой. Для примера создадим модели `book` и `reader`,                                                            
из которых первая, ведущая, будет хранить готовые машины, а вторая, ведомая,                                                        
- отдельные детали для них.                                                 

```python
from django.contrib.auth.models import User
from django.db import models


class Book(models.Model):
    name = models.CharField(max_length=150)
    content = models.FileField(
        upload_to='documents/',
        null=True,
        blank=True
    )
    creator = models.ForeignKey(
        Reader,
        on_delete=models.CASCADE
    )
    deleted = models.BooleanField(default=False)
    created_at = models.DateTimeField(auto_now_add=True)
    deleted_at = models.DateTimeField()


class Reader(models.Model):
  name = models.CharField(max_length=ЗO)
  books = models.ManyТoManyField(Book) 
```

В отличие от связей описанных ранее типов, имя поля, образующего связь                                                           
"многие-со-многими", рекомендуется записывать во множественном числе.                                                            
Что и логично - ведь такая связь позволяет связать произвольное Число                                                           
записей, что называется, с обеих сторон. На уровне базы данных для                                                           
представления связи такого типа создается таблица, по умолчанию                                                          
имеющая имя вида `<псевдоним приложения>_<имя класса ведущей модели>_<имя класса ведомой модели>`                                                         
(связующая таблица). Она будет иметь ключевое поле `id` и по одному полю                                                       
с именем вида `<имя класса связываемой модели>_id` на каждую из связываемых                                                          
моделей. Так, в нашем случае будет создана связующая таблица с именем                                                        
`samplesite_machine_spare`, содержащая поля `id`, `machine_id` и `spare_id`.                                                           

Если создается связь с той же самой моделью, связующая таблица будет                                                            
иметь поля `id`, `from _<имя класса модели>_id` И `to_<имя класса модели>_id`.                                                           
Конструктор класса `ManyТoManyField` поддерживает дополнительные необязательные                                                          
параметры `limit_choices_to`, `related_name`, `related_query_name` и `db_constraint`, а также:                                                 

* `symmetrical` - используется только в тех случаях, когда модель связывается сама с собой.                                                       
Если `True`, **Django** создаст симметричную связь, действующую в обоих направлениях                                                       
Если `False`, то связь будет асимметричной (чисто гипотетически: Иванов любит колбасу, однако                                                          
колбаса не любит Иванова ((((( ). Значение по умолчанию - `True`. Для асимметричной связи                                                          
**Django** создаст в классе модели атрибут для доступа к записям связанной модели в                                                        
обратном направлении                                                 


* `through` - класс модели, которая представляет связующую таблицу (связующая модель) либо в                                                        
виде ссылки, либо в виде имени, представленном строкой. Если класс не указан, то связующая                                                        
таблица будет создана самим Django. При использовании связующей модели нужно иметь в виду следующее:                                                 

  1) поле внешнего ключа для связи объявляется и в ведущей, и в ведомой моделях. При создании                                                        
  этих полей следует указать как саму связующую модель (параметр `through`), так и поля внешних                                                           
  ключей, по которым будет установлена связь (параметр `through_fields`, описанный далее);                                                 
  2) в связующей модели следует явно объявить поля внешних ключей для установления связи с                                                            
  обеими связываемыми моделями: и ведущей, и ведомой                                                 


* `through_fields` - используется, если связь устанавливается через связующую модель,                                                         
записанную в параметре `through` конструктора. Указывает поля внешних ключей, по которым                                                       
будет создаваться связь. Значение параметра должно представлять собой кортеж из двух                                                        
элементов: имени поля ведущей модели и имени поля ведомой модели, записанных в виде строк.                                                          
Если параметр не указан, то поля будут созданы самим фреймворком.                                                 


* `db_table` - имя связующей таблицы. Обычно применяется, если связующая модель не используется.                                                         
Если оно не указано, то связующая таблица получит имя по умолчанию                                                 


---

# **The model's parameters**                                                 

Параметры самой модели описываются различными атрибутами класса `Meta`, вложенного в класс                                                         
модели и не являющегося производным ни от какого класса.                                                 

* `verbose_name` - название сущности, хранящейся в модели, которое будет выводиться на экран.                                                        
Если не указано, используется имя класса модели.                                                 


* `verbose_name_plural` - название набора сущностей, хранящихся в модели, которая будет выводиться                                                        
на экран. Если не указано, используется имя класса модели во множественном числе.                                                 


* `ordering` - параметры сортировки записей модели по умолчанию. Задаются в виде последовательности                                                         
имен полей, по которым должна выполняться сортировка, представленных строками. Если перед именем                                                          
поля поставить символ "минус", то порядок сортировки будет обратным. Пример:                                                 

```python
class Task(models.Model):
    class Meta:
        ordering =['-created_at', 'title'] 
```

Сортируем записи модели сначала по убыванию значения поля `created_at`, а потом по                                                        
возрастанию значения поля `title`.                                                 

* `unique_together` - последовательность имен полей, представленных в виде строк, которые                                                           
должны хранить уникальные в пределах таблицы комбинации значений. При попытке занести в них                                                         
уже имеющуюся в таблице комбинацию значений будет возбуждено исключение `ValidationError`                                                                                                           
из модуля `django.core.exceptions`. Пример:                                                 

```python
class Task(models.Model):
    class Meta:
        unique_together = ('title' , 'created_at') 
```

Теперь комбинация названия задачи и временной отметки создания должна быть уникальной                                                           
в пределах модели. Добавить в тот же день еще одну такую же задачу не получится.                                                            
Можно указать несколько подобных групп полей, объединив их в последовательность:                                                 

```python
class Task(models.Model):
    class Meta:
        unique_together = (('title' , 'created_at'), ( 'title', 'creator', 'category'))
```

Теперь уникальными должны быть и комбинация названия задачи и временной отметки                                                         
создания, и комбинация названия задачи, её создателя и категории.                                                 

* `get_latest_by` - имя поля типа `DateField`, или `DateTirneField`, которое будет                                                          
ВЗЯТО в расчет при получении наиболее поздней или наиболее ранней записи с                                                          
помощью метода `latest()` или `earliest()` соответственно, вызванного                                                        
без параметров. Можно задать:                                                 
  1) имя поля в виде строки - тогда в расчет будет взято только это поле:                                                 
    ```python
      class Task(models.Model):
        ...
        created_at = models.DateTimeField() 
        class Meta:
            get_latest_by = 'created_at'
    ```
    Теперь метод `latest()` вернет запись с наиболее поздним значением временной отметки,                                                     
    хранящемся в поле `created_at`. Если имя поля предварить символом "минус", то порядок                                                        
    сортировки окажется обратным, и при вызове `latest()` мы получим, напротив, самую                                                           
    раннюю запись, а при вызове метода `earliest()` - самую позднюю.                                                 

  2) последовательность имен полей - тогда в расчет будут взяты значения всех этих                                                        
  полей, и, если у каких-то записей первое поле хранит одинаковые значения,                                                        
  будет проверяться значение второго поля и т. д.                                                        
  
  ```python
    class Bb(models.Model):
      ...
      added = models.DateTirneField() 
      published = models.DateTirneField() 
      
      class Meta:
          get_latest_by =  ['edited', 'published'] 
  ```

* `indexes` - последовательность индексов, включающих в себя несколько полей. Каждый                                                           
элемент такой последовательности должен представлять собой экземпляр класса `Index`                                                         
из модуля `django.db.models`                                                 

```python
class Task(models.Model):
       ...
       class Meta:
         indexes = [
           models.Index(fields=['-created_at', 'title'],
                name='task_main'),
           models.Index(fields=['title', 'status', 'category']), 
         ]
```

В параметре `fields` указывается список или кортеж строк с именами полей, которые                                                           
должны быть включены в индекс. По умолчанию сортировка значений поля выполняется                                                              
по их возрастанию, а чтобы отсортировать по убыванию, нужно предварить имя поля                                                             
знаком "минус". Параметр `name` задает имя индекса - если он не указан, то имя                                                          
будет создано самим фреймворком.                                                 

* `index_together` - предлагает другой способ создания индексов, содержащих несколько полей.                                                         
Строки с именами полей указываются в виде последовательности, а набор таких                                                           
последовательностей также объединяется в последовательность.                                                 

```python
class Task(models.Model):
      ...
      class Meta:
        index_together = [
          ['-published', 'title'],
          ['title', 'price', 'rubric']
        ]
```

* `default_related_name` - имя атрибута записи первичной модели,                                                          
предназначенного для доступа к связанным записям вторичной модели, в виде строки.                                                       
Соответствует параметру `related_name` конструкторов классов полей, предназначенных                                                         
для установления связей между моделями. Неявно задает значения параметрам                                                         
`related_name` и `related_query_name` конструкторов                                                 


* `db_tablе` - имя таблицы, в которой хранятся данные модели.                                                           
Если оно не указано, то таблица получит имя вида `<псевдоним приложения>_<имя класса модели>`.                                                          
Например, для модели `Task` приложения `test_models` в базе данных будет создана                                                
таблица `test_models_task`                                                  


* `constraints` - условия, которым должны удовлетворять данные, заносимые в запись.                                                           
В качестве значения указывается список или кортеж экземпляров классов,                                                           
каждый из которых задает одно условие.                                                 

---

# **Models methods**                                                 


Модели могут включать в себя различные методы:                                                 

*  `__str__()` определяет строковое представление объекта модели.                                                      
Он возвращает строку, которая будет использоваться при приведении                                                        
объекта к строке, например, при выводе в административном                                                       
интерфейсе Django или в консоли.                                                      


* `__repr__()` определяет представление объекта модели в виде                                                      
строки, которое будет использоваться для отладки и внутренних целей.                                                       
Обычно он возвращает строку, содержащую имя класса модели и                                                         
некоторую информацию о состоянии объекта                                                         


*  `__init__()` выполняется при создании нового экземпляра модели.                                                      
Он позволяет инициализировать поля модели и выполнить другие                                                        
необходимые действия перед сохранением объекта.                                                        


* `save()` вызывается при сохранении объекта модели. Он                                                        
выполняет операцию создания или обновления записи в базе данных.                                                      
Можно переопределить этот метод для выполнения дополнительных                                                           
действий перед сохранением или после сохранения объекта                                                       


* `delete()` вызывается при удалении объекта модели. Он удаляет                                                        
соответствующую запись из базы данных. Можно так же переопределить                                                        
этот метод для выполнения дополнительных действий перед удалением                                                        
объекта или после удаления                                                          


* `get_absolute_url()` возвращает URL-адрес объекта модели. Этот                                                       
метод часто используется в шаблонах или при редиректах для                                                        
получения URL-адреса объекта                                                        


* `clean()` выполняется при валидации объекта модели. Он                                                         
позволяет выполнять пользовательскую валидацию полей модели и                                                     
вызывать ошибки валидации при необходимости.                                                          


* Если у поля модели есть аргумент `choices`, то автоматически создается                                                          
метод `get_<field>_display()`, который возвращает текстовое                                                         
представление выбранного значения из списка `choices`.                                                     

---

# **Migrations**                                                 

Очень хитрая, крутая и мощная штука в Django!                                                 

После создания модели, её следует мигрировать из рабочего                                                      
пространства **Django** в подключённую БД. Когда происходит любое                                                          
изменение моделей, следует проводить миграции.                                                    


Для создания миграций используется команда `makemigrations`, а для их                                                           
применения `migrate`. При выполнении миграций **Django** создаёт                                                          
дополнительные модели для правильной работы приложения, а также                                                        
автоматически создаёт стандартную таблицу **Users**, и в следствии                                                        
создания кастомной таблицы пользователей её следует переопределить,                                                       
чтобы избежать конфликтов БД.                                                  

Для этого в настройках проекта указывается строка:                                                       

```python
AUTH_USER_MODEL = '<app_name>.<model_name>'
```

В кавычках указывается ссылка до модели (название приложения.модель)                                                 


Для создания миграций используем `python manage.py makemigrations`                                                 
Для применения миграций `python manage.py migrate`                                                 
Для того, чтобы посмотреть применённые миграции, требуется в консоли                                                      
выполнить `python manage.py showmigrations`                                                 
В случае ошибки с миграциями в приложении, их можно откатить                                                 
используя `python manage.py migrate <app_name> zero`                                                  

Для того, чтобы откатить полностью `python manage.py migrate                                                 
<app_name> <номер_миграции_до_которой_нужно_откатить>`                                                 

# !!!! при откате миграции, которая затрагивает другие таблицы,                                                 
(например, в случае связи между таблицами) связные миграции откатятся                                                 
тоже.                                                 

---

# **Django Admin Panel**                                                 

Django автоматически создаёт админ панель вашего веб-приложения, в                                                         
которой можно администрировать БД. За панель администратора                                                       
отвечает файл `admin.py` в приложении. Админ-панель доступна по `адресу                                                          
вашей страницы + /admin`                                                        

```python
from django.contrib import admin
from apps.todos.models.category import Category
from apps.todos.models.statuses import Status
from apps.todos.models.todo import Task
from apps.todos.models.subtask import SubTask


@admin.register(Category)
class CategoryAdmin(admin.ModelAdmin):
    pass


@admin.register(Status)
class StatusAdmin(admin.ModelAdmin):
    pass


@admin.register(Task)
class TaskAdmin(admin.ModelAdmin):
    pass


@admin.register(SubTask)
class SubTaskAdmin(admin.ModelAdmin):
    pass

```

Для регистрации используется декоратор **register** (это один из вариантов регистрации)                                                 

**Для получения доступа входа в админ панель требуется**                                                 
создать суперпользователя: `python manage.py createsuperuser`                                                 


**Чем удобен вариант регистрирования моделей через декоратор?**                                                 

Он позволяет нам добавлять доп функционал при работе с нашими моделями:                                                 
добавить какие-то действия, фильтры, отображения определённых полей и т.д.                                                 

```python
from django.contrib import admin

from apps.todos.models.category import Category
from apps.todos.models.statuses import Status
from apps.todos.models.todo import Task
from apps.todos.models.subtask import SubTask


@admin.register(Category)
class CategoryAdmin(admin.ModelAdmin):
    list_display = ('name',)
    list_filter = ('name',)
    search_fields = ('name',)


@admin.register(Status)
class StatusAdmin(admin.ModelAdmin):
    list_display = ('name',)
    list_filter = ('name',)
    search_fields = ('name',)


@admin.register(Task)
class TaskAdmin(admin.ModelAdmin):
    list_display = ('title', 'category', 'status', 'creator', 'created_at')
    list_filter = ('category', 'status', 'creator', 'created_at')
    search_fields = ('title',)


@admin.register(SubTask)
class SubTaskAdmin(admin.ModelAdmin):
    list_display = ('title', 'category', 'task', 'status', 'creator', 'created_at')
    list_filter = ('task', 'category', 'status', 'creator', 'created_at')
    search_fields = ('title',)
```


Так же там мы можем создавать свои, кастомные действия для объектов, которые                                                                                   
хотим выбрать. Эта возможность - admin actions:                                                                                    

```python
@admin.register(SubTask)
class SubTaskAdmin(admin.ModelAdmin):
    def change_subtasks_register(self, request, queryset):
        for obj in queryset:
            obj.title.upper()
            obj.save()

    change_subtasks_register.short_description = 'Up register'

    actions = [
        'change_subtasks_register'
    ]
...
...
```

# **Changing themes**

Если вдруг вас не устраивает то, как по умолчанию выглядит тема админ                                                                                            
панели Django - вы всегда можете создать свою!                                                                                       

Для того, чтобы это было вам доступно - необходимо сделать следующее:                                                                                            

1) Установить доп пакет для работы с кастомными темами:                                                                                            
`pip install django-admin-interface`                                                                                              
2) После его установки в настройках проекта в `INSTALLED_APPS`                                                                                         
добавить два новых пакета:                                                                                                  

```python
INSTALLED_APPS = [
    "admin_interface",
    "colorfield",
    ...
    ...
```
3) Провести миграции, так как будет добавлена новая модель:                                                                                                
`python manage.py makemigrations`                                                 
`python manage.py migrate`                                                 

4) Наслаждаться :)                                                 
