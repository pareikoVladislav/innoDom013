<!-- TOC -->
* [**DRF class-views, serializers, swagger**](#drf-class-views-serializers-swagger)
* [**Serializers**](#serializers-)
* [**LIST all of subtasks + POST new subtask**](#list-all-of-subtasks--post-new-subtask-)
* [**GET a specific subtask + PUT OR DELETE it**](#get-a-specific-subtask--put-or-delete-it)
* [**DRF filtration**](#drf-filtration-)
* [**Swagger**](#swagger)
<!-- TOC -->
# **DRF class-views, serializers, swagger**


**Представление (`view`) в DRF** - это компонент, который обрабатывает\
**HTTP-запросы** и возвращает **HTTP-ответы**. Представление определяет, как\    
данные будут представлены в нашем `API`.

При обработке **HTTP-запроса**, представление может выполнять\
следующие задачи:

* **извлекать данные** из базы данных или других источников данных для\
дальнейшей обработки.
* **проверять и валидировать** полученные данные, чтобы убедиться, что они\
соответствуют ожидаемому формату и типу.
* **преобразовывать извлеченные** и валидированные данные в формат, который\
будет возвращен клиенту (например, `JSON`, `XML` и т.д.). **Сериализация**\
осуществляется с помощью сериализаторов `DRF`.
* **обрабатывать различные** типы `HTTP-запросов`, такие как `GET`, `POST`,\
`PUT`, `DELETE` и т.д., и выполнять соответствующие действия в\
зависимости от типа запроса.
* **после обработки данных**, формировать `HTTP-ответ`, который содержит\
данные, ошибки (если есть), статус код и другую информацию


**Django** предоставляет различные виды представлений:

1) **Функциональные представления**

2)  **Представления основанные на классах**:
    * **APIView** - базовый класс представления, который предоставляет обработку\
    **HTTP-методов** (`GET`, `POST`, `PUT`, `DELETE`) через методы\
    (`get()`, `post()`, `put()`, `delete()` и тд).

    * **GenericAPIView** - базовый класс представления, который предоставляет\
    общие методы для работы с моделями и сериализаторами. Он часто\
    используется с миксинами (`mixins`) для обработки различных типов запросов

    * **ViewSet** - это класс, который предоставляет более простой и декларативный\
    способ определения представлений для работы с моделями. **ViewSet**\
    объединяет логику для обработки различных **HTTP-методов** в одном классе,\
    таком как `list()`, `create()`, `retrieve()`, `update()`, `destroy()`, и т.д.


**Функциональные представления**

**Функциональные представления** - это один из способов определения\
представлений в **Django REST Framework (DRF)**. Вместо использования\
классов для определения представлений, функциональные представления\
представляют собой обычные функции `Python`, которые обрабатывают\
**HTTP-запросы** и возвращают **HTTP-ответы**, то есть принимают\
`Request` и возвращают `Response`. Обычно используются вместе\
со специальным декоратором `@api_view(['GET', '...'])`, в котором\
можно указывать нужные нам методы. Делается для того, чтобы определённая\
функция воспринималась системой именно как функция-отображение, а не обычная.


**APIView** - это базовый класс представления в **DRF**, который предоставляет\
более гибкий и детализированный подход к обработке запросов, чем функциональные\
представления. Он позволяет явно определять методы для каждого типа\
**HTTPзапроса** (`GET`, `POST`, `PUT`, `DELETE` и т.д.) и более гибко управлять\
логикой обработки запросов


**GenericAPIView** - это базовый класс представления в **DRF**, который\
предоставляет общий функционал для обработки различных типов\
запросов (`GET`, `POST`, `PUT`, `DELETE`) и операций с данными. Он\
предназначен для работы с одним объектом или списком объектов и\
является более гибким и кастомизируемым, чем `ModelViewSet`, но требует\
более явного определения логики для каждого метода запроса.


В DRF **ViewSet** представляет собой класс, который предоставляет\
удобный и декларативный способ определения представлений (**views**) для\
работы с моделями или другими данными вашего приложения. **ViewSet**\
сочетает в себе различные методы для обработки различных типов\
запросов и операций над данными.

---

Продолжим с того, на чём остановились на прошлом занятии.                                                                                                             

# **Serializers**                                                   


**Сериализаторы** помогают нам с вами указывать нашему `DRF` какие именно\
поля должны быть включены в процесс **сериализации**, или же\
**десереализации**. Далеко не редкость, когда необходимо подготавливать\
`JSON-ответы` для фронта в струкруре вложенной.

В нашем случае, допустим, при переходе по определённой задаче мы хотим\
так же отображать и список всех подзадач, если он есть.\
Это, как мы смотрели на прошлом занятии, вполне реально. Нужно добавить\
новое поле в классе-сериализаторе, значением которого будет другой\
сериализатор, отвечающий за те самые вложенные данные.

Но как писать код, если мы хотим, чтобы какие-то поля отображались не просто\
`ID` от первичного\вторичного ключа, а именно прям вот данные?

Для таких требований нам с вами можно использовать `serializers.StringRelatedField`\
класс, который поможет нам как раз преображать `related` поля вторичных ключей\
в нормальные, читабельные данные для нашего клиента:

```python
class SubTaskShortInfoSerializer(serializers.ModelSerializer):
    status = serializers.SlugRelatedField(
        slug_field='name',
        read_only=True
    )

    class Meta:
        model = SubTask
        fields = [
            'id',
            'title',
            'status',
            'deadline'
        ]
```

И сериализатор для самих задач:

```python
class TaskInfoSerializer(serializers.ModelSerializer):
    subtasks = SubTaskShortInfoSerializer(many=True, read_only=True)
    category = serializers.SlugRelatedField(
        slug_field='name',
        queryset=Category.objects.all()
    )
    status = serializers.SlugRelatedField(
        slug_field='name',
        queryset=Status.objects.all()
    )
    creator = serializers.SlugRelatedField(
        slug_field='email',
        read_only=True
    )

    class Meta:
        model = Task
        fields = [
            'id',
            'title',
            'description',
            'category',
            'status',
            'creator',
            'subtasks',
            'deadline'
        ]
```


Этот класс `StringRelatedField` берёт в возвращаемое значение то, что\
в главной, родительской модели, закреплено в репрезентации в магическом\
методе `__str__(self)`. Если вы прописали там отображение `name` - во\
вложенной структуре категорий, статусов и других полей будет отображаться,\
именно имя статуса\категории, или же имейл пользователя, вместо `ID`

Если мы пропишем в метод `__str__(self)` отображение:                                                                                                            

```python
class Category(models.Model):
    name = models.CharField(max_length=25)

    def __str__(self):
        return f"MY AWESOME {self.name}!"  # NEW
```

То в таком случае при построении нашего `JSON` ответа с данными в поле\
категории будет отображаться именно такой формат.

Но при этом следует помнить, что это поле - исключительно для чтения.\
Поля, которые вы переопределили с использованием `serializers.StringRelatedField()`\
будут исключены из форм на создание, или обновление записей.

Если вам необходимо И текстовое отображение, И возможность работать с этими\
полями в формах **Django** - вам может помочь класс `serializers.SlugRelatedField()`\
При работе с этим полем наш интерес будет окружён вокруг нескольких атрибутов:\
* `slug_field` - принимает строку в формате имени поля. Какое поле мы пропишем в этот\
атрибут, такое поле отображаться в ответе клиенту и будет
* `read_only` - принимает параметр `True`. Если этот флаг в состоянии `True` - данные будут\
только для чтения и так же не будут отображаться нам в формах `Django`. По умолчанию\
этот флаг в состоянии `False`
* `queryset` - если мы хотим наше личное отображение, но при этом чтобы и поле было доступно\
в `Django` формах - нам нужно указать какие значения должны будут доступны для\
работы в форме.


# **LIST all of subtasks + POST new subtask**                                                   

Добавим с вами возможность отображать список всех подзадач\
и возможность создавать новую подзадачу:

```python
from rest_framework.generics import ListCreateAPIView, get_object_or_404
from rest_framework.request import Request
from rest_framework.response import Response
from rest_framework import status

from apps.subtasks.serializers import ListSubTasksSerializer
from apps.subtasks.models import SubTask


class AllSubTasksGenericView(ListCreateAPIView):
    serializer_class = ListSubTasksSerializer

    def create_subtask(self, data):
        serializer = self.serializer_class(data=data)

        serializer.is_valid(raise_exception=True)
        serializer.save()

        return serializer.data

    def get_queryset(self):
        subtasks = SubTask.objects.filter(
            creator=self.request.user.id
        )

        return subtasks

    def get(self, request: Request, *args, **kwargs):
        subtasks = self.get_queryset()

        print(request.user.id)

        if not subtasks:
            return Response(
                status=status.HTTP_204_NO_CONTENT,
                data=[]
            )

        serializer = self.serializer_class(subtasks, many=True)

        return Response(
            status=status.HTTP_200_OK,
            data=serializer.data
        )

    def post(self, request: Request, *args, **kwargs):
        new_subtask = self.create_subtask(data=request.data)

        return Response(
            status=status.HTTP_201_CREATED,
            data=new_subtask
        )

```

И добавим сериализатор для работы с нашими данными на вход и выход:                                                                                                                          

```python
class ListSubTasksSerializer(serializers.ModelSerializer):
    category = serializers.SlugRelatedField(
        slug_field='name',
        queryset=Category.objects.all()
    )
    status = serializers.SlugRelatedField(
        slug_field='name',
        queryset=Status.objects.all()
    )
    creator = serializers.SlugRelatedField(
        slug_field='email',
        queryset=User.objects.all()
    )

    class Meta:
        model = SubTask
        fields = [
            'id',
            'title',
            'description',
            'category',
            'status',
            'task',
            'creator',
            'date_started',
            'deadline'
        ]

    def validate_title(self, value):
        if value is None:
            raise serializers.ValidationError(
                "This field is required"
            )

        if len(value) > 75:
            raise serializers.ValidationError(
                "The name of subtask must be at maximum 75 characters"
            )

        return value

    def validate(self, attrs: dict) -> dict:
        cur_datetime = timezone.now()
        date_started = attrs.get('date_started')
        deadline = attrs.get('deadline')

        if date_started is None or deadline is None:
            raise serializers.ValidationError(
                "You must provide both a date started and a deadline for the subtask you are creating."
            )

        if date_started < cur_datetime:
            raise serializers.ValidationError(
                "Date started can't be in the past."
            )

        if deadline < cur_datetime:
            raise serializers.ValidationError(
                "The deadline can't be earlier than today."
            )

        if deadline < date_started:
            raise serializers.ValidationError(
                "The deadline can't be earlier than the date started."
            )

        return attrs

```

Здесь мы переопределяем поля статуса и категории таким образом,\
чтобы мы получали в JSON формате не просто ID, а читабельные данные,\
при этом сохраняя возможность обновления этих полей, а не использование\
их только в режиме чтения, что делает класс `StringRelatedField`.

Атрибут `slug_field` в `SlugRelatedField` используется для определения
поля модели, **значение которого будет использоваться для представления**\
**связи в сериализаторе**. Это **не обязательно** должно быть поле типа\
`SlugField` в нашей модели `Django`; это может быть любое поле,\
значение которого **уникально** идентифицирует объекты,\
такое как `CharField`.

В контексте `SlugRelatedField`, `slug_field` указывает **DRF** использовать\
значение поля `name` модели `Status` или `Category` для сериализации и\
десериализации, вместо использования первичного ключа. Когда мы\
отправляем данные через `API`, мы будем использовать значение `name`\
для указания статуса или категории, и `DRF` будет искать\
соответствующий объект с таким `name` для связи.

**Вот как это работает:**

* При **сериализации** (отправке данных клиенту): **DRF** будет отображать значение\
поля `name` в `JSON` вместо `ID`.

* При **десериализации** (приеме данных от клиента): **DRF** ожидает, что в поле будет\
передано значение `name`, и использует это значение для поиска соответствующего\
объекта `Status` или `Category` в базе данных. Если объект найден, он будет\
связан с экземпляром `SubTask`.                                                                                                          

Если мы хотим, чтобы пользователи могли создавать и обновлять `SubTask` через\
`API`, используя имя статуса или категории, то `SlugRelatedField` с\
`slug_field='name'` - это правильный подход. Однако, если поля `name` в `Status`\
или `Category` **не уникальны**, это может привести к ошибкам, так как `DRF` не\
сможет точно определить, на какой объект ссылаться. В этом случае нам\
нужно будет гарантировать уникальность значений `name` в этих моделях.


# **GET a specific subtask + PUT OR DELETE it**

Теперь же добавим возможность просматривать определённую подзадачу, а так же\
добавим туда возможность обновлять её, или же и вовсе удалять:

```python
class SubTaskInfoGenericView(RetrieveUpdateDestroyAPIView):
    serializer_class = SubTaskInfoSerializer

    def partly_update(self, instance: SubTask) -> dict:
        serializer = self.serializer_class(
            instance=instance,
            data=self.request.data,
            partial=True
        )
        serializer.is_valid(raise_exception=True)
        serializer.save()

        return serializer.data

    def full_update(self, instance: SubTask) -> dict:
        serializer = self.serializer_class(
            instance=instance,
            data=self.request.data
        )

        serializer.is_valid(raise_exception=True)
        serializer.save()

        return serializer.data

    def get_object(self):
        subtask_id = self.kwargs.get("pk")

        subtask = get_object_or_404(SubTask, id=subtask_id)

        return subtask

    def get(self, request: Request, *args, **kwargs) -> Response:
        subtask = self.get_object()

        serializer = self.serializer_class(subtask)

        return Response(
            status=status.HTTP_200_OK,
            data=serializer.data
        )

    def put(self, request: Request, *args, **kwargs) -> Response:
        subtask = self.get_object()

        updated_subtask = self.full_update(
            instance=subtask
        )

        return Response(
            status=status.HTTP_205_RESET_CONTENT,
            data=updated_subtask
        )

    def patch(self, request: Request, *args, **kwargs) -> Response:
        subtask = self.get_object()

        updated_subtask = self.partly_update(
            instance=subtask
        )

        return Response(
            status=status.HTTP_205_RESET_CONTENT,
            data=updated_subtask
        )

    def delete(self, request: Request, *args, **kwargs) -> Response:
        subtask = self.get_object()

        subtask.delete()

        return Response(
            status=status.HTTP_204_NO_CONTENT,
            data=[]
        )

```

И так же отдельный сериализатор для работы с определённой задачей. Тут\
нужно будет учитывать такие аспекты, как получение информации о задаче,\
при этом валидацию полей как при полном обновлении, так и при частичном\
обновлении:

```python
class SubTaskInfoSerializer(serializers.ModelSerializer):
    category = serializers.SlugRelatedField(
        slug_field='name',
        queryset=Category.objects.all()
    )
    status = serializers.SlugRelatedField(
        slug_field='name',
        queryset=Status.objects.all()
    )
    creator = serializers.SlugRelatedField(
        slug_field='email',
        queryset=User.objects.all()
    )

    class Meta:
        model = SubTask
        fields = [
            'id',
            'title',
            'description',
            'category',
            'status',
            'creator',
            'date_started',
            'deadline'
        ]

    def validate_title(self, value):
        if self.instance and not value:
            value = self.instance.title

        if value is None:
            raise serializers.ValidationError(
                "This field is required"
            )

        if len(value) > 75:
            raise serializers.ValidationError(
                "The name of subtask must be at maximum 75 characters"
            )

        return value

    def validate_category(self, value):
        if self.instance and not value:
            value = self.instance.category

        if value is None:
            raise serializers.ValidationError(
                "You must specified the category of this subtask"
            )

        return value

    def validate_status(self, value):
        if self.instance and not value:
            value = self.instance.status

        if value is None:
            raise serializers.ValidationError(
                "You must specified the status of this subtask"
            )

        return value

    def validate_date_started(self, value):
        if self.instance and not value:
            value = self.instance.date_started

        if value < timezone.now():
            raise serializers.ValidationError(
                "Date started can't be in the past."
            )
        return value

    def validate_deadline(self, value):
        if self.instance and not value:
            value = self.instance.deadline

        if value < timezone.now():
            raise serializers.ValidationError(
                "The deadline can't be earlier than today."
            )

        return value

    def update(self, instance, validated_data):
        for attr, value in validated_data.items():
            setattr(instance, attr, value)
        instance.save()
        return instance

```

Мы можем как валидировать конкретное поле благодаря функции `validate_<field_name>(self, value)`,\
которую мы и будем использовать, так и общую функцию валидации всех полей, которая\
называется `validate(self, attrs)`.

При этом, как и описывалось чуть выше, нам нужно учитывать(если мы хотим это\
реализовывать) и полное, и частичное обновление. Для этого мы можем в наших\
валидаторах первично прописывать проверку, если наш инстанс уже есть(а мы\
передаём в наш сериализатор инстанс, поэтому доступ к нему есть), но при этом\
значения не было передано - значение будет браться из базы данных, то есть уже\
существующее.

# **DRF filtration**                                                                                                             

Как же при этом делать фильтрацию данных? Что, если нам хочется\
ещё и фильтровать наши данные по разным параметрам? При этом эти фильтры\
могут быть не обязательными, если мы не хотим фильтровать данные - возвращаться\
будут просто все данные, что есть.

Мы так же можем это сделать. Обычно фильтрация происходит посредством\
передачи `query_params` атрибутов, которые не являются обязательными,\
то есть можно обойтись и без них. Они указываются после символа `?`\
после чего указывается имя специального фильтрационного параметра запроса,\
и ему присваевается какое-то значение:

```127.0.0.1:8000/subtasks/?status=NEW``` - допустим так вот. Получить\
список всех подзадач, у которых статус - **NEW**


```python
class AllSubTasksGenericView(ListCreateAPIView):
    serializer_class = ListSubTasksSerializer

    def create_subtask(self, data):
        serializer = self.serializer_class(data=data)

        serializer.is_valid(raise_exception=True)
        serializer.save()

        return serializer.data

    def get_queryset(self):
        queryset = SubTask.objects.filter(
            creator=self.request.user.id
        )

        obj_status = self.request.query_params.get("status")
        category = self.request.query_params.get("category")
        date_from = self.request.query_params.get('date_from')
        date_to = self.request.query_params.get('date_to')
        deadline = self.request.query_params.get('deadline')

        if obj_status:
            queryset = queryset.filter(status__name=obj_status)

        if category:
            queryset = queryset.filter(category__name=category)

        if date_from and date_to:
            date_from = timezone.make_aware(
                datetime.datetime.strptime(
                    date_from,
                    "%Y-%m-%d"
                ),
                timezone.get_default_timezone()
            )

            date_to = timezone.make_aware(
                datetime.datetime.strptime(
                    date_to,
                    "%Y-%m-%d"
                ),
                timezone.get_default_timezone()
            )
            queryset = queryset.filter(date_started__range=[date_from, date_to])

        if deadline:
            deadline = timezone.make_aware(
                datetime.datetime.strptime(
                    deadline,
                    "%Y-%m-%d"
                ),
                timezone.get_default_timezone()
            )
            queryset = queryset.filter(deadline=deadline)

        return queryset

    def get(self, request: Request, *args, **kwargs):
        subtasks = self.get_queryset()

        if not subtasks:
            return Response(
                status=status.HTTP_204_NO_CONTENT,
                data=[]
            )

        serializer = self.serializer_class(subtasks, many=True)

        return Response(
            status=status.HTTP_200_OK,
            data=serializer.data
        )

    def post(self, request: Request, *args, **kwargs):
        new_subtask = self.create_subtask(data=request.data)

        return Response(
            status=status.HTTP_201_CREATED,
            data=new_subtask
        )
```

Тут логика следующая: Мы формируем первычный **queryset** и дальше,\
в зависимости от переданных фильтрационных параметров запроса,\
мы фильтруем наши задачи исходя из настроек. При этом, если не было\
передано ни единого фильтра - возвращаем просто первычный **queryset**,\
который получили изначально.

Дальше, в самом методе `get()`, мы получаем все эти данные и проверяем:\
если наш **queryset** не пустой - передаём его в сериализатор, не зыбывая\
указать ему флаг `many=True` для отображения списка всех задач. Если же\
ни одной задачи не будет - будем отображать пустой спикок.


Теперь давайте так же рассмотрим и похожую вариацию для обычных задач.\
Отличие дополнительно будет в том, что у каждой задачи может быть и сколько\
угодно дополнительно подзадач, поэтому запрос может отличаться. Сперва\
немного обновим сериализатор для всех задач:

```python
class ListTasksSerializer(serializers.ModelSerializer):
    subtasks = SubTaskShortInfoSerializer(many=True, read_only=True)
    category = serializers.SlugRelatedField(
        slug_field='name',
        queryset=Category.objects.all()
    )
    status = serializers.SlugRelatedField(
        slug_field='name',
        queryset=Status.objects.all()
    )
    creator = serializers.SlugRelatedField(
        slug_field='email',
        read_only=True
    )

    class Meta:
        model = Task
        fields = [
            'id',
            'title',
            'description',
            'category',
            'status',
            'creator',
            'date_started',
            'deadline',
            'subtasks'
        ]

```

После чего уже и перепишем наш класс-отображение:

```python
class TasksListAPIView(ListCreateAPIView):
    serializer_class = ListTasksSerializer

    def create_task(self, data):
        serializer = self.serializer_class(data=data)

        serializer.is_valid(raise_exception=True)
        serializer.save()

        return serializer.data

    def get_queryset(self):
        queryset = Task.objects.filter(
            creator=self.request.user.id
        ).prefetch_related('subtasks')

        return queryset

    def get(self, request: Request, *args, **kwargs) -> Response:
        tasks = self.get_queryset()

        if tasks.exists():
            serializer = ListTasksSerializer(tasks, many=True)

            return Response(
                status=status.HTTP_200_OK,
                data=serializer.data
            )
        else:
            return Response(
                status=status.HTTP_204_NO_CONTENT,
                data=[]
            )

    def post(self, request: Request, *args, **kwargs) -> Response:
        serializer = ListTasksSerializer(data=request.data)

        if serializer.is_valid(raise_exception=True):
            serializer.save()

            return Response(
                status=status.HTTP_201_CREATED,
                data=serializer.data
            )
        return Response(
            status=status.HTTP_400_BAD_REQUEST,
            data=serializer.errors
        )

```

В запросе мы дополнительно использовали метод `prefetch_related`. Это те самые\
`JOIN` в **Django ORM** и в этом фреймворке их есть два вида:

`select_related` используется для уменьшения количества запросов к базе\
данных путем "**соединения**" (`JOIN`) связанных объектов в одном **SQL-запросе**.\
Это хорошо подходит для обращений к "**одиночным**" связям, таким как\
`ForeignKey` или `OneToOneField`. Например, у нас есть модель `Task`
с `ForeignKey` на `Status`, использование `select_related` позволит избежать\
дополнительного запроса к базе данных для получения связанного\
`Status` для каждой `Task`.

`prefetch_related`, с другой стороны, используется для запроса\
"**множественных**" связей, таких как связи через `ManyToManyField` или\
от `ForeignKey`. Вместо выполнения отдельного запроса\
для каждого объекта, `prefetch_related` выполнит отдельный запрос для\
получения всех связанных объектов, а затем "**вручную**" свяжет их с\
основными объектами, с которыми они ассоциированы. Это эффективно\
для уменьшения количества запросов к базе данных при работе с\
большим числом связанных объектов.

Использование `select_related` и `prefetch_related` может значительно\
увеличить производительность вашего API за счет снижения количества\
запросов к базе данных, но следует учитывать, что они могут привести\
к большим объемам передаваемых данных, если связанные объекты\
сами по себе велики или их много.


---

# **Swagger**

Так же, как у нас может быть техническая документация к коду, мы можем\
составить и техническую документацию к `API`, используя дополнительные\
инструменты. Эта документация при этом позволит как нам, так и\
`front-end` разработчикам.

Такая техническая документация называется `Swagger`. Для того, чтобы добавить её\
в проект необходимо установить дополнительную библиатеку:

`pip install drf-yasg`

`drf-yasg` автоматически генерирует документацию из наших **представлений**\
и **сериализаторов**. Нужно убедиться, что наши `viewsets` и `views` правильно\
аннотированы и имеют соответствующие `docstrings`, чтобы улучшить\
сгенерированную документацию.

И, конечно же, добавить её в `INSTALLED_APPS`:

```python
INSTALLED_APPS = [
    ...
    'rest_framework',
    'drf_yasg',  # NEW
    ...
]

```

И после этого нужно будет добавить небольшой дополнительный код для генерации\
нешего сваггера. Этот код можем добавить прямо в главный файл `urls`:


```python
from django.urls import path, re_path
from drf_yasg.views import get_schema_view
from drf_yasg import openapi
from rest_framework import permissions


schema_view = get_schema_view(
   openapi.Info(
      title="API Documentation",
      default_version='v1',
      description="Test description",
      terms_of_service="https://www.google.com/policies/terms/",
      contact=openapi.Contact(email="contact@yourapi.com"),
      license=openapi.License(name="BSD License"),
   ),
   public=True,
   permission_classes=(permissions.AllowAny,),
)

urlpatterns = [
    path("admin/", admin.site.urls),
    path("", include('apps.router')),
    re_path(
        r'^swagger(?P<format>\.json|\.yaml)$',
        schema_view.without_ui(cache_timeout=0)
    ),
    path(
        'swagger/',
        schema_view.with_ui('swagger', cache_timeout=0)
    ),
    path(
        'redoc/',
        schema_view.with_ui('redoc', cache_timeout=0))

]
```

**Этот код создает три маршрута**: один для **JSON-спецификации** (`/swagger.json`),\
один для **YAML-спецификации** (`/swagger.yaml`), и два для визуализаций с\
помощью **Swagger UI** (`/swagger/`) и **ReDoc** (`/redoc/`).
