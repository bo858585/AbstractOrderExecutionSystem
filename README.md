# AbstractBooking

***Техническое задание на систему выполнения абстрактных заказов***

###Сущности:
1. **Пользователи.** Пользователь - либо администратор, либо принадлежит к одной из двух групп - исполнители, заказчики. Эти группы обладают различными правами.
2. **Расширенный профиль** пользователя.
3. **Заказ**.
4. **Счет системы**.


###Счет пользователя:
У любого заказчика, исполнителя может быть только по одному счету - поле в расширенном профиле модели пользователя. Профиль содержит поле "Счет":
DecimalField, NOT NULL, default = 100.

###Две группы пользователей с правами:
1. **customers** - booking.add_booking (создавать заказ).
2. **performers** - booking.perform_perm (брать заказ на исполнение).  


###Настройка и использование сущностей в админке:
1. **System accounts** - создать один счет системы, указать текущие денежные средства и комиссию системы.
2. **User profiles**	- создать профили для этих двух пользователей, указать их денежные средства.
3. **Группы** - создать две группы: customers, performers, назначить им права (см. ниже).
4. **Пользователи** - тестовых пользователей после создания групп можно завести в админке (custuser, perfuser) и внести их в соответствующую группу.
Также это можно сделать через форму регистрации, но после создания групп в админке. В этом случае расширенные профили пользователей будут созданы автоматически.

**Добавить группам в админке права:**

1. performers - booking | booking | Ability to perform created booking
2. customers - booking | booking | Can add booking
3. customers - booking | booking | Can change booking
4. customers - booking | booking | Can delete booking

**Добавить пользователям группы в админке:**
1. custuser - customers,
2. perfuser - performers

Введенные в админке данные должны быть валидными.

###Элементы управления заказами в пользовательском интерфейсе списка заказов:
- **Для исполнителя:** исполняет заказ (кнопка в элементе списка в BookingListView).
- **Для заказчика:** завершает заказ (кнопка в элементе списка в BookingListView).

###Модель заказа:

######Статусы заказа (CHOICES в модели заказа):
1. **"Ожидает выполнения"** - начальный статус.
2. **"Взят на исполнение"** - статус назначается заказу после клика на кнопку
“Взять заказ” исполнителем.
3. **"Завершен"** - статус назначается заказу после клика на кнопку “Завершить заказ” исполнителем.

**CHOICES** = [ “pending”, “running”, “completed” ]

######Схема модели заказа:
- **Название** - CharField NOT NULL.
- **Описание** - TextField NOT NULL.
- **Стоимость** - IntegerField NOT NULL, default=100.
- **Статус** - CharField с choices - варианты внутри модели, один из них default.
- **Заказчик** - Foreign Key, NOT NULL.
- **Исполнитель** - Foreign Key (can be NULL).
- **Дата и время создания заказа** - стандартное поле.


###User stories.

1. ######Форма создания заказа: CreateView.
Когда заказчик формирует заказ через форму, он указывает стоимость заказа.
Заказу назначается введенная цена. Затем заказ сохраняется в таблицу-ленту
заказов, заказу автоматически назначается статуc - “Ожидает выполнения”,
заказу назначается исполнитель - "NULL". Доступна только заказчикам. В ней можно заполнить поля для модели заказа. Для отправки формы используется POST request.

2. ######Лента заказов.
В ней отображаются заказы. Невзятые на исполнение заказы доступны для исполнения исполнителям. Заказчики могут их просматривать. Завершенные заказы не отображаются. После взятия заказа он становится доступен на чтение. Для создавшенго его заказчика появляется кнопка завершения.
Для каждого заказа выводится его название, описание, стоимость, статус,
заказчик, при наличии - исполнитель, соответствующая статусу кнопка, дата и
время создания. В начале на заказе есть кнопка “Взять заказ”, которая
отображается только исполнителям. (Нереализовано: для пользователя, создавшего заказ, на месте этой кнопки располагается кнопка "Удалить заказ").

**Соответствие кнопок на заказе из ленты и текущих статусов этих заказов
(Надпись на кнопке - статус заказа):**
- "Взять заказ("/"Удалить заказ") - "Ожидает выполнения"("pending").
- "Завершить заказ" - "Взят на исполнение"("running").
- Заказ не отображается в ленте - “Завершен”("completed").

**Схема отображения кнопок/действий в столбце таблицы:**
- Взять заказ можно, если его статус "pending" и у заказчика достаточно   средств для оплаты. Если средств недостаточно, заказ неактивен.
- Завершить заказ можно, если его статус "running" и пользователь,  создавший заказ, совпадает с текущим пользователем.
- Строка таблицы не отображается, если статус заказа "completed".

3. ######При клике на кнопку “Взять заказ”:
Производится проверка, хватит ли заказчику этого заказа денег на оплату заказа. Цена введенного заказа должна быть меньше (либо равна) сумме, которая у него есть на счету. Со счета заказчика списывается цена заказа (она присутствует в модели заказа). Заказу назначается  статус “Выполняется”. Заказ сохраняется. Это значит, что системе в ленту заказов - виртуально, согласно статусу заказа - перешли деньги со счета заказчика, то есть система выступает посредником между заказчиком и исполнителем). Страница обновляется. После этого кнопка “Взять заказ”, доступная исполнителям, на заказе исчезает, появляется кнопка “Завершить заказ”, доступная только заказчику, сделавшему этот заказ.

4. ######При нажатии на кнопку “Завершить заказ”:
Сумма заказа в ленте в зависимости от комиссии (целое число от 0 до 100 процентов, указывается в админке в соответсвующей модели (уточнить)) делится на две части - на счет исполнителя заказа и системы поступают две эти части суммы. Заказу назначается статус “Завершен”. Заказчику выводятся сообщения с указанием этих сумм. Страница обновляется. Заказ исчезает из ленты как выполнившийся.

4. ######Список заказов, связанных с пользователем:
Страница, в которой отображается список заказов самого пользователя - заказы, в которых он является заказчиком или исполнителем. Этот список полностью аналогичен списку всех заказов системы за тем исключением, что в нем отображаются и завершенные заказы (сделано, чтобы пользователь видел историю своих заказов).

###Обработка нажатия кнопок в последнем столбце таблицы при включенном ajax:
По нажатию кнопки в строке таблицы вызывается AJAX POST request по обработке заказа с последующим изменением его статуса и его отображением на странице.

При регистрации пользователю начисляется 1000. В реальной системе средства пользователь себе начисляет через отдельную соответсвующую форму ввода, связанную с платежной системой (например, django-robokassa).

###Установка библиотек и остальных компонентов, необходимых для работы:

1. virtualenvwrapper
https://virtualenvwrapper.readthedocs.org/en/latest/

2. pip install python

3. pip install Django

4. postgresql http://paintincode.blogspot.ru/2012/08/install-postgresql-for-django-and.html
(проверить pg_conf с django https://stackoverflow.com/questions/7695962/postgresql-password-authentication-failed-for-user-postgres)

**Установить django-registration:**
```sh
cd project_dir
$ git clone https://github.com/macropin/django-registration
$ cd django-registration
$ python setup.py install
```
###Тесты

```sh
$ cd /home/user/work/Booking/AbstractBooking/Booking
$ python manage.py test booking.tests.BookingModelTestCase
$ python manage.py test booking.tests.BookingViewsTestCase
```

###Развертывание

```sh
$ apt-get install nginx
$ sudo apt-get install uwsgi uwsgi-plugin-python
$ cp /etc/nginx/sites-available/default /etc/nginx/sites-available/_default
$ nano /etc/nginx/sites-available/default
$
$# Вставить в default конфигурацию ниже:
$# (Пути к проекту заменить на локальные)
$server {
  $  listen   80;
  $  server_name localhost;
  $  
  $  location /static/  {
    $    alias /home/user/work/Booking/AbstractBooking/Booking/static_for_deploy/;
    $  }
    $  
    $  location / {
      $    root            /home/user/work/Booking/AbstractBooking/Booking/Booking;
      $    uwsgi_pass      127.0.0.1:3031;
      $    include         uwsgi_params;
      $  }
      $}
      $ cd /home/user/work/Booking/AbstractBooking/Booking
      $ mkdir static_for_deploy
      $ python manage.py collectstatic
      $
      $ uwsgi --socket :3031 --chdir ./ --env DJANGO_SETTINGS_MODULE=Booking.settings --module "django.core.wsgi:get_wsgi_application()"
      $
      $ sudo /etc/init.d/nginx reload
      ```
      
