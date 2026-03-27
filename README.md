# Python dasturlash tilida Dekoratorlar mavzusi

Dekoratorlar Python dasturlash tilida funksiyalar va metodlarning funksionalligini kengaytirish yoki o'zgartirish uchun ishlatiladigan kuchli vositadir. Ushbu maqolada dekoratorlar nima ekanligi, ular qanday ishlaydi, qanday yaratiladi va real loyihalarda qanday qo'llanilishi haqida batafsil tushuncha beramiz. Maqola boshlang'ich va o'rta darajadagi dasturchilar uchun tushunarli bo'lishi uchun misollar bilan boyitilgan.

## Dekoratorlar nima?

Dekorator — bu boshqa bir funksiyani o'rab oladigan va uning xatti-harakatini o'zgartiradigan yoki kengaytiradigan maxsus funksiyadir. Python'da dekoratorlar odatda `@decorator_name` sintaksisi yordamida ishlatiladi. Ular funksiyalarni "o'rash" (wrapping) orqali ularga qo'shimcha funksionallik qo'shish imkonini beradi, masalan, kodni takrorlamasdan logging, vaqt o'lchash yoki ruxsatlarni tekshirish kabi vazifalarni bajarish uchun.

Dekoratorlar Python'da birinchi darajali funksiyalar (first-class functions) tufayli mumkin bo'ladi, chunki Python'da funksiyalar o'zgaruvchilarga yuklanishi, boshqa funksiyalarga argument sifatida uzatilishi yoki funksiyadan qaytarilishi mumkin.

### Dekoratorlarning asosiy xususiyatlari:

- **Funksiyani o'zgartirish**: Dekorator asl funksiyani o'zgartirmasdan uning xatti-harakatini kengaytiradi.
- **Qayta ishlatilish**: Bir dekoratorni bir nechta funksiyalarda qayta-qayta ishlatish mumkin.
- **Osonlik**: Kodni takrorlamasdan umumiy vazifalarni bir joyda boshqarish imkonini beradi.

## Dekoratorlar qanday ishlaydi?

Dekoratorning ishlash mantig'ini tushunish uchun avval Python'da funksiyalarning birinchi darajali ob'ekt ekanligini esga olish kerak. Dekorator odatda uchta qismdan iborat:

1. **Ichki funksiya (wrapper)**: Asl funksiyani o'rab oladigan va qo'shimcha logika qo'shadigan funksiya.
2. **Tashqi funksiya**: Dekoratorning o'zi, u asl funksiyani argument sifatida qabul qiladi.
3. **Qaytariladigan natija**: Tashqi funksiya ichki funksiyani qaytaradi.

### Oddiy misol:

Quyida oddiy dekoratorning tuzilishi keltirilgan:

```python
def my_decorator(func):
    def wrapper():
        print("Funksiya chaqirilishidan oldin biror narsa.")
        func()  # Asl funksiya chaqiriladi
        print("Funksiya chaqirilganidan keyin biror narsa.")
    return wrapper

@my_decorator
def say_hello():
    print("Salom, dunyo!")

say_hello()

```

**Chiqish:**

```
Funksiya chaqirilishidan oldin biror narsa.
Salom, dunyo!
Funksiya chaqirilganidan keyin biror narsa.

```

**Tushuntirish:**

- `@my_decorator` sintaksisi `say_hello = my_decorator(say_hello)` bilan bir xil ishlaydi.
- `my_decorator` funksiyasi `say_hello` funksiyasini argument sifatida qabul qiladi.
- `wrapper` funksiyasi `say_hello` ni o'rab oladi va qo'shimcha kod (oldin va keyin chop etish) qo'shadi.

## Argumentli funksiyalar uchun dekoratorlar

Yuqoridagi misolda `say_hello` funksiyasi hech qanday argument qabul qilmaydi. Ammo ko'pincha funksiyalar argumentlar bilan ishlaydi. Bunday hollarda `wrapper` funksiyasi `*args` va `**kwargs` dan foydalanib, har qanday argumentni qabul qila oladi.

```python
def my_decorator(func):
    def wrapper(*args, **kwargs):
        print("Funksiya chaqirilishidan oldin.")
        result = func(*args, **kwargs)  # Argumentlarni uzatamiz
        print("Funksiya chaqirilganidan keyin.")
        return result  # Natijani qaytaramiz
    return wrapper

@my_decorator
def greet(name):
    return f"Salom, {name}!"

print(greet("Ali"))

```

**Chiqish:**

```
Funksiya chaqirilishidan oldin.
Salom, Ali!
Funksiya chaqirilganidan keyin.
Salom, Ali!

```

**Tushuntirish:**

- `args` va `*kwargs` yordamida dekorator har qanday argumentlarni qabul qila oladi.
- `func(*args, **kwargs)` orqali asl funksiya o'z argumentlari bilan chaqiriladi.
- Natija `return` orqali qaytariladi, shunda dekorator funksiyaning o’zini o'zgartirmaydi.

## Dekoratorlarga parameter berish

Ba'zida dekoratorning o'ziga ham parameter berish kerak bo'ladi. Buning uchun dekoratorni yana bir funksiya bilan o'rash kerak, ya'ni uchtalik funksiya tuzilmasi hosil bo'ladi.

```python
def repeat(n):
    def decorator(func):
        def wrapper(*args, **kwargs):
            for _ in range(n):
                func(*args, **kwargs)
        return wrapper
    return decorator

@repeat(3)
def say_hello():
    print("Salom!")

say_hello()

```

**Chiqish:**

```
Salom!
Salom!
Salom!

```

**Tushuntirish:**

- `repeat(n)` funksiyasi dekoratorga qancha marta takrorlash kerakligini aytadi.
- `decorator` funksiyasi asl funksiyani qabul qiladi.
- `wrapper` funksiyasi `n` marta asl funksiyani chaqiradi.

## Dekoratorlarda funksiya metama'lumotlarini saqlash

Dekorator ishlatilganda, asl funksiyaning metama'lumotlari (masalan, `__name__`, `__doc__`) yo'qolishi mumkin, chunki `wrapper` funksiyasi asl funksiyani almashtiradi. Buni tuzatish uchun `functools.wraps` dan foydalanamiz.

```python
from functools import wraps

def my_decorator(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        print("Funksiya chaqirilishidan oldin.")
        return func(*args, **kwargs)
    return wrapper

@my_decorator
def greet(name):
    """Ismga salom beradi."""
    return f"Salom, {name}!"

print(greet.__name__)  # greet
print(greet.__doc__)   # Ismga salom beradi.

```

**Tushuntirish:**

- `@wraps(func)` yordamida `wrapper` funksiyasi asl funksiyaning metama'lumotlarini (masalan, `__name__`, `__doc__`) saqlab qoladi.
- Bu kod sifatini oshiradi va debuggingni osonlashtiradi.

## Real dunyoda dekoratorlardan foydalanish

Dekoratorlar ko'plab real loyihalarda qo'llaniladi. Quyida bir nechta misollar:

### Vaqt o'lchash dekoratori

Funksiyaning ishlash vaqtini o'lchash uchun dekorator:

```python
import time
from functools import wraps

def timer(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        start = time.time()
        result = func(*args, **kwargs)
        end = time.time()
        print(f"{func.__name__} {end - start:.4f} sekund ishladi.")
        return result
    return wrapper

@timer
def slow_function():
    time.sleep(2)
    return "Tugadi!"

print(slow_function())

```

**Chiqish:**

```
slow_function 2.0023 sekund ishladi.
Tugadi!

```

### Ruxsatlarni tekshirish dekoratori

Foydalanuvchi ruxsatlarini tekshirish uchun:

```python
from functools import wraps

def requires_permission(permission):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            user_permissions = ["read", "write"]  # Bu yerda haqiqiy ruxsatlar ro'yxati bo'ladi
            if permission in user_permissions:
                return func(*args, **kwargs)
            else:
                raise PermissionError(f"{permission} ruxsati talab qilinadi!")
        return wrapper
    return decorator

@requires_permission("admin")
def delete_user(user_id):
    return f"Foydalanuvchi {user_id} o'chirildi."

try:
    print(delete_user(123))
except PermissionError as e:
    print(e)

```

**Chiqish (agar "admin" ruxsati bo'lmasa):**

```
admin ruxsati talab qilinadi!

```

### Logging dekoratori

Funksiya chaqiruvlarini jurnalga yozish:

```python
from functools import wraps
import logging

logging.basicConfig(level=logging.INFO)

def log(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        logging.info(f"{func.__name__} chaqirildi: args={args}, kwargs={kwargs}")
        return func(*args, **kwargs)
    return wrapper

@log
def add(a, b):
    return a + b

print(add(3, 5))

```

**Chiqish:**

```
INFO:root:add chaqirildi: args=(3, 5), kwargs={}
8

```

## Klass dekoratorlari

Dekoratorlar nafaqat funksiyalar, balki klasslar uchun ham ishlatilishi mumkin. Klass dekoratorlari odatda klassning xatti-harakatini o'zgartirish yoki kengaytirish uchun ishlatiladi.

```python
def class_decorator(cls):
    cls.new_attribute = "Yangi atribut qo'shildi"
    return cls

@class_decorator
class MyClass:
    def __init__(self):
        self.value = 42

obj = MyClass()
print(obj.new_attribute)  # Yangi atribut qo'shildi
print(obj.value)          # 42

```

## Bir nechta dekoratorlarni qo'llash

Funksiyaga bir nechta dekorator qo'llanilishi mumkin. Bunday hollarda dekoratorlar yuqoridan pastga qarab ketma-ket qo'llaniladi.

```python
def decorator1(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        print("Birinchi dekorator")
        return func(*args, **kwargs)
    return wrapper

def decorator2(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        print("Ikkinchi dekorator")
        return func(*args, **kwargs)
    return wrapper

@decorator1
@decorator2
def say_hello():
    print("Salom!")

say_hello()

```

**Chiqish:**

```
Birinchi dekorator
Ikkinchi dekorator
Salom!

```

**Tushuntirish:**

- `@decorator1` birinchi bo'lib qo'llaniladi, keyin `@decorator2`.
- Natijada `decorator1(decorator2(say_hello))` kabi ishlaydi.

## Dekoratorlarning afzalliklari va kamchiliklari

### Afzalliklari:

- **Kodni qayta ishlatish**: Umumiy funksionallikni bir joyda yozib, ko'p joyda ishlatish mumkin.
- **Modullik**: Kodni toza va oson o'qiladigan qiladi.
- **Funksiyalarni kengaytirish**: Asl funksiyani o'zgartirmasdan yangi xususiyatlar qo'shish.

### Kamchiliklari:

- **Murakkablik**: Ko'p dekoratorlar ishlatilganda kodni tushunish qiyinlashishi mumkin.
- **Debug qilish qiyinligi**: Agar dekorator noto'g'ri yozilsa, xatolarni aniqlash qiyin bo'lishi mumkin.
- **Metama'lumotlarning yo'qolishi**: `functools.wraps` ishlatilmasa, funksiya metama'lumotlari yo'qoladi.

## Xulosa sifatida:

Python'dagi dekoratorlar kodni yanada moslashuvchan va qayta ishlatiladigan qilish uchun ajoyib vositadir. Ular yordamida logging, vaqt o'lchash, ruxsatlarni tekshirish kabi vazifalarni osonlikcha amalga oshirish mumkin. Dekoratorlarni to'g'ri ishlatish uchun `functools.wraps` dan foydalanish, argumentlarni boshqarish va ularning ichki tuzilishini tushunish muhimdir.

Ushbu dars orqali siz dekoratorlarni noldan boshlab tushunish va ularni o'z loyihalaringizda qo'llash imkoniyatiga ega bo'ldingiz. Amaliyot qilib, turli xil dekoratorlar yozishga harakat qiling va ularni loyihalaringizda sinab ko'ring!