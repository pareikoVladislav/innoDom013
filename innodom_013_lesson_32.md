# **DRF filtration**                                                                                                             

До этого момента мы в основном смотрели с вами возможности получения\
информации, как она есть: получить список всех объектов, получить какой-то\
конкретный объект и так далее.

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

# **Custom User model**                                                

При всех наших механиках, что мы уже реализовали, мы использовали модель\
пользователя, которая предлагалась системой Django по умолчанию. Но что,\
если нам нужна какие-то свои поля, мы хотим работать с нашими конкретными\
полями, используя так же какую-то иную, кастомную логику работы с ним?

Мы можем создать своего пользователя, овновываясь на базовых моделях.

```python
from django.db import models
from django.contrib.auth.models import (
    AbstractBaseUser,
    PermissionsMixin,
)
from django.utils.translation import gettext_lazy

from apps.user.manager import UserManager


class User(AbstractBaseUser, PermissionsMixin):
    email = models.EmailField(
        max_length=120,
        unique=True,
        verbose_name=gettext_lazy('Email address')
    )
    first_name = models.CharField(
        max_length=50,
        verbose_name=gettext_lazy('First name')
    )
    last_name = models.CharField(
        max_length=50,
        verbose_name=gettext_lazy('Last name')
    )
    username = models.CharField(
        max_length=30,
        blank=True,
        null=True
    )
    phone = models.CharField(
        max_length=75,
        blank=True,
        null=True
    )
    is_staff = models.BooleanField(default=False)
    is_superuser = models.BooleanField(default=False)
    is_verified = models.BooleanField(default=False)
    is_active = models.BooleanField(default=True)
    date_joined = models.DateTimeField(auto_now_add=True)
    last_login = models.DateTimeField(auto_now=True)

    USERNAME_FIELD = "email"

    REQUIRED_FIELDS = ['first_name', 'last_name']

    objects = UserManager()

    def __str__(self):
        return self.email

    @property
    def get_full_name(self):
        return f"{self.first_name} {self.last_name}"

```

Здесь мы переопределили нашу базовую модель пользователя, прописав свои\
настройки и так же, как вы могли обратить внимание, дополнительно указали\
настройки и для админа, так как модель будет общая. Просто для админа\
поля по умолчанию будут в состоянии **False**

Дополнительно мы тут указываем настройку поля `username`, которое после\
нашего переопределения будет именно `email` за это отвечает атрибут `USERNAME_FIELD`.\
Так же указываем обязательные поля при регистрации, помимо наших всяких\
`email` и `password`. За это отвечает атрибут `REQUIRED_FIELDS`. Его основная\
цель - указать нам какие дополнительные поля должны быть включены в выборку\
при создании пользователя через коммандрую строку командой `createsuperuser`\
Эти поля идут дополнительно, такие поля, как `username` и `password` воспринимаются\
системой **Django**, как само собой разумеющееся.

Так же, так как мы переопределили пользователя, написав своего, необходимо                                                  
еще и менеджера для него определить. Как именно будут создаваться обычные\
пользователи, и как - администраторы. За это будет отвечать будущий класс\
`UserManager`, в котором мы опишем как у нас должен будет создаваться\
пользователь (админ, или же обычный, не важно.)

---

# **Create custom UserManager**                                                 

Для написания своего Мэнеджера нам понадобится базовый мэнеджер, от которого\
мы будем наследоваться, прописывая что-то конкретное. На выходе получится так,\
что для нашей модели пользователя мы будем иметь возможность работать со всеми\
внутренними методами (благодаря `BaseUserManager`), такими как получение данных,\
создание, обновление и прочее. Но при этом дополним всю эту логику своими кастомными\
методами на создание обычного пользователя и пользователя - администратора.

```python
from django.contrib.auth.models import BaseUserManager
from django.core.exceptions import ValidationError
from django.core.validators import validate_email
from django.utils.translation import gettext_lazy


class UserManager(BaseUserManager):
    def email_validator(self, email):
        try:
            validate_email(email)
        except ValidationError as err:
            raise ValueError(
                gettext_lazy(f"{err.message}.\nPlease, enter a valid email")
            )

    def create_user(self, email, first_name, last_name, password, **extra_fields):
        if email:
            email = self.normalize_email(email)
            self.email_validator(email=email)
        else:
            raise ValueError(gettext_lazy(
                "Email is required."
            ))

        if not first_name:
            raise ValueError(gettext_lazy(
                "First name is required."
            ))

        if not last_name:
            raise ValueError(gettext_lazy(
                "Last name is required."
            ))

        user = self.model(
            email=email,
            first_name=first_name,
            last_name=last_name,
            **extra_fields
        )

        user.set_password(password)
        user.save(using=self._db)

        return user

    def create_superuser(self, email, first_name, last_name, password, **extra_fields):
        extra_fields.setdefault("is_staff", True)
        extra_fields.setdefault("is_superuser", True)
        extra_fields.setdefault("is_verified", True)

        if not extra_fields.get("is_staff"):
            raise ValueError(
                gettext_lazy("Admin must be 'is staff'")
            )
        if not extra_fields.get("is_superuser"):
            raise ValueError(
                gettext_lazy("Admin must be a superuser")
            )

        user = self.create_user(
            email=email,
            first_name=first_name,
            last_name=last_name,
            password=password,
            **extra_fields
        )
        user.save(using=self._db)

        return user

```

Тут мы создаём класс будущего мэнеджера, который будет наследоваться\
от `BaseUserManager` класса. у нас будет два основных метода:\
создание обычного пользователя и создание администратора. Так же мы\
добавим валидацию **email**.

для валидатора имэйла ничего особо не выдумываем. Мы принимаем значение,\
пытаемся провалидировать его уже написанными инструментами **Django**,\
если что-то не так - поднимаем ошибку с нужным нам значением.

При создании пользователя мы будем принимать ряд аргументов для регистрации\
и проверять их на валидность

если всё хорошо - создаём объект будущего пользователя, прокидывая в него\
наши поля. Так как мы работаем с написанием своего, кастомного менеджера\
мы не будем прописывать здесь никаких `User` напрямую, так как мы создаём\
менеджера, нашим `User` для него будет - `self.model`

после создания объекта пользователя мы у этого объекта вызываем метод\
который отвечает за установление пароля, после чего мы у этого объекта вызываем\
метод `save(using=self._db)`, который данные и сохранит. Здесь `self._db` - наша\
база данных. Нужно это всё для того, что мы работаем здесь на чуть более глубинном\
уровне и нет никакой конкретики: что именно за модель, где уже внутри итак\
есть связь - в рамках какой базы данных эта модель написана.

в конце возвращаем созданного пользователя.

Крайне похоже будет работать и метод на создание суперюзера, только\
полей немного больше.

Так же необходимо в настройках Джанго показать ему, что модель пользователя\
должна быть не по умолчанию базовой, а именно той, что написали мы:

```python
AUTH_USER_MODEL = "user.User"
```

---

# **Custom User's Serializer**

Кастомная модель есть, мэнеджер так же есть. Необходимо создать сериализатор,\
в котором будет происходить валидация данных при создании.

```python
from rest_framework import serializers

from apps.user.models import User


class UserRegisterSerializer(serializers.ModelSerializer):
    password = serializers.CharField(max_length=68,min_length=8, write_only=True)
    password2 = serializers.CharField(max_length=68, min_length=8, write_only=True)

    class Meta:
        model = User
        fields = ['email', 'first_name', 'last_name', 'password', 'password2']

    def validate(self, attrs):
        password = attrs.get("password", "")
        password2 = attrs.get("password2", "")

        if password != password2:
            raise serializers.ValidationError(
                "Passwords must match"
            )

        return attrs

    def create(self, validated_data):
        user = User.objects.create_user(
            email=validated_data.get("email"),
            first_name=validated_data.get("first_name"),
            last_name=validated_data.get("last_name"),
            password=validated_data.get("password")
        )

        return user

```


Для валидации данных на создание пользователя и будущее сохранение напишем\
сериализатор. Тут допом определим поля для пароля, что они должны быть только\
для записи. Никакого чтения, так как это пароль, быть не должно.

Метод `validate` будет обрабатывать данные по паролю. Они должны совпадать.

Метод `create`, собственно, будет отвечать за создание пользователя.

Остаётся написать класс-отображение, который позволит нам создавать
нового пользователя:


```python
class UserRegistrationGenericView(CreateAPIView):
    serializer_class = UserRegisterSerializer

    def post(self, request: Request, *args, **kwargs):
        serializer = self.serializer_class(data=request.data)

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

Тут ничего особенного. Мы принимаем в форме данные для регистрации. Эти\
данные улетают в сериализатор для валидации. Если всё хорошо, ошибок нет и\
данные валидные - сохраняем нового пользователя и возвращаем ответ с успешным\
успехом. В противном же случае возвращаем ответ с ошибками из сериализатора.


Подвязываем этот класс-отображение к `urls`:

```python
urlpatterns = [
    ...
    ...
    ...
    path("auth/register/", UserRegistrationGenericView.as_view()),  # NEW
] + router.urls
```

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
