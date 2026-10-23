# Library — kutubxona axborot tizimi

PostgreSQL, Django va DRF asosidagi katalog, kitob nusxalari, o‘quvchilar va berish-qaytarish tizimi.

## Ishga tushirish (Windows PowerShell)

Loyiha papkasida `uv sync` bajaring. `.env.example` asosida `.env` yarating va haqiqiy baza parolini kiriting. `.env` Git orqali tarqatilmaydi.

```powershell
uv run python manage.py migrate
uv run python manage.py createsuperuser
uv run python manage.py runserver
```

Standart ulanish: `localhost:5433`, baza `library`, user `library_user`. Ularni `.env` orqali almashtirish mumkin. `PSYCOPG_IMPL=python` Windows'da PostgreSQL `bin` papkasi PATH'da bo‘lishini talab qiladi.

## Foydalanish

- `/` — ommaviy katalog (qidiruv va sahifalash).
- `/accounts/login/` — kirish. Superuser yoki `is_staff=True` foydalanuvchi kutubxonachi hisoblanadi.
- `/accounts/signup/` — ism-familiya, username va parol bilan ro‘yxatdan o‘tish; oddiy User va avtomatik raqamli Reader kartasi bir tranzaksiyada yaratiladi.
- `/my/` — shaxsiy kabinet: faqat kirgan o‘quvchining kitoblari, muddatlari, kechikishlari va tarixi. O‘quvchi kitob berish/qaytarish yozuvini o‘zgartira olmaydi.
- `/dashboard/` — statistika va eng ko‘p berilgan kitoblar.
- `/copies/` — kitob nusxalarini yaratish/tahrirlash.
- `/readers/` — o‘quvchilar; kartadagi «Foydalanish tarixi» orqali barcha berishlarni ko‘rish.
- `/loans/add/` — nusxani faol o‘quvchiga berish, qaytarish muddatini Toshkent vaqtida kiritish.
- `/loans/` — kitobni qaytarish, faol/qaytarilgan/muddati o‘tgan yozuvlarni filterlash.
- `/admin/` — ma’lumotnoma va foydalanuvchilarni boshqarish. Berish tarixi faqat o‘qiladi; berish/qaytarish uchun yuqoridagi sahifalarni ishlating.

Tarix o‘chirilmaydi. Nusxa, o‘quvchi va kitobga bog‘langan tarix `PROTECT` orqali himoyalanadi. Nusxaning `status`i jismoniy holat, berilganligi esa faol `Loan` orqali aniqlanadi. Limit `.env`dagi `MAX_ACTIVE_LOANS` orqali belgilanadi (standart 5).

## API

`/api/books/`, `/api/authors/`, `/api/copies/` — ommaviy GET, o‘zgartirish faqat kutubxonachi uchun.

`/api/readers/`, `/api/loans/` — faqat kutubxonachi. Ro‘yxatlarda javob formati `count`, `next`, `previous`, `results`; har sahifada 20 ta yozuv. `?search=...`, katalogda `?ordering=title` qo‘llanadi.

Berish: `POST /api/loans/`:

```json
{"copy": 1, "reader": 1, "due_at": "2026-12-01T18:00:00+05:00"}
```

Qaytarish: `POST /api/loans/1/return/`, tana `{}`. Tarix: `GET /api/loans/?reader=1`; holat: `?status=active`, `returned`, `overdue`. Loan yozuvlari uchun PUT/PATCH/DELETE mavjud emas.

`/api-auth/login/` browsable API uchun session login. Session bilan POST/PUT/PATCH/DELETE so‘rovlarida CSRF token kerak. Basic authentication faqat HTTPS orqali ishlatilishi kerak.

`GET /api/my/loans/` va `/api/my/loans/<id>/` — faqat kirgan hisobga bog‘langan o‘quvchining tarixi. Begona ID uchun 404; yozish amallari mavjud emas. Kabinet yoki API orqali boshqa reader ID berish o‘quvchini almashtirmaydi.

Avval qo‘lda yaratilgan o‘quvchilar yangi hisobga ismiga qarab avtomatik ulanmaydi. Kutubxonachi «O‘quvchilar → Tahrirlash → Saytga kirish hisobi» yoki admin orqali tekshirilgan hisobni bog‘laydi. Bir hisob faqat bitta o‘quvchi kartasiga bog‘lanadi. Ro‘yxatdan o‘tish hech qachon staff/superuser huquqini bermaydi.

## Tekshirish

```powershell
uv run python manage.py check
uv run python manage.py makemigrations --check --dry-run
uv run python manage.py test --noinput
```

Testlar alohida `test_library` bazasida ishlaydi. Lokal test useriga `CREATEDB` kerak; bu huquqni production ilova useriga bermang. Testlar asosiy bazadagi kitoblarni o‘zgartirmaydi.

## Serverga chiqarishdan oldin

`DEBUG=False`, yangi tasodifiy `SECRET_KEY`, domen uchun `ALLOWED_HOSTS`, HTTPS, PostgreSQL zaxirasi, static fayllarni xizmat qilish va production WSGI/ASGI serveri sozlanadi. `uv run python manage.py check --deploy` bilan tekshiring. `runserver` production server emas.

Hozircha bronlash, jarima, email/SMS xabarlari va ISBN bo‘yicha nashrlar katalogi kiritilmagan. O‘quvchi o‘z tarixini, kutubxonachi esa barcha o‘quvchilarning tarixini ko‘radi.
