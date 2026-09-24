# Модуль Player

Описує сутності, пов'язані з користувачами, персонажами, їхніми характеристиками, закляттями та інвентарем.

---

## User

Зберігає облікові дані гравців платформи.

* **Атрибути:**
  * `id` — `int generated always as identity` (Primary Key).
  * `username` — `varchar[100] NOT NULL` — унікальне ім'я користувача для входу та ідентифікації.
  * `email` — `varchar[255] NOT NULL` — адреса електронної пошти.
  * `password` — `varbinary[128] NOT NULL` — хеш пароля користувача.
  * `ban_until` — `timestamp NULL default NULL` — часова мітка закінчення терміну блокування (якщо користувача заблоковано).
  * `ban_reason` — `varchar[420] NULL` — причина накладання блокування.

* **Зв'язки:**
  * Має зв'язок один до багатьох із сутністю `Character` (один акаунт може мати нуль або кілька створених персонажів, але кожен персонаж належить строго одному користувачу).

---

## Character

Центральна сутність ігрового персонажа, що об'єднує ігрові підсистеми, білд, характеристики та фізичну присутність у світі.

* **Атрибути:**
  * `id` — `int generated always as identity` (Primary Key).
  * `name` — `varchar[100] NOT NULL` — ігрове ім'я персонажа.
  * `user_id` — `int NOT NULL` — Foreign Key до таблиці `User`.
  * `worldentity_id` — `int NOT NULL` — Foreign Key до сутності `WorldEntity` (визначає координати у світі та базові параметри сутності).
  * `stats_id` — `int NOT NULL` — Foreign Key до сутності `Stats`.

* **Зв'язки:**
  * **User:** багато до одного (нуль або багато персонажів належать одному `User`).
  * **WorldEntity:** один до одного (персонаж є конкретним втіленням сутності світу).
  * **Stats:** один до одного (персонаж володіє власним набором базових характеристик).
  * **CharacterBuild:** один до одного (закріплює расу, стать, клас та зовнішність).
  * **Attributes:** один до одного (зберігає динамічні ресурси: HP, ману, досвід).
  * **Inventory:** один до одного (персональний інвентар та золото персонажа).
  * **CharacterSpell:** один до багатьох (персонаж може вивчити нуль або декілька заклять).

---

## CharacterBuild

Зберігає візуальні та рольові характеристики персонажа, задані під час створення або кастомізації.

* **Атрибути:**
  * `id` — `int generated always as identity` (Primary Key).
  * `character_id` — `int NOT NULL UNIQUE` — Foreign Key до `Character`. Зв'язок строго один до одного.
  * `race` — `Race NOT NULL` — раса персонажа (enum: `Human`, `Orc`, `Elf`, `Dwarf`, `Khajiit`).
  * `skin_color_hex` — `char[7] NOT NULL` — шістнадцятковий код кольору шкіри (наприклад, `#FFFFFF`).
  * `sex` — `SEX NOT NULL` — стать персонажа (enum: `Male`, `Female`, `Croissant`, `Non-binary`).
  * `class` — `CLASS NOT NULL` — ігровий клас персонажа (enum: `Cleric`, `Fighter`, `Rogue`, `Wizard`).

---

## Stats

Таблиця атрибутів базової бойової сили та фізичних/ментальних параметрів персонажа (або сутностей світу).

* **Атрибути:**
  * `id` — `int generated always as identity` (Primary Key).
  * `strength` — `int NOT NULL` — показник сили.
  * `dexterity` — `int NOT NULL` — показник спритності/влучності.
  * `intelligence` — `int NOT NULL` — показник інтелекту.
  * `defense` — `int NOT NULL` — показник захисту/броні.
  * `agility` — `int NOT NULL` — показник рухливості/швидкості реакції.

---

## Attributes

Зберігає динамічні ресурсні параметри персонажа, які змінюються в процесі гри.

* **Атрибути:**
  * `id` — `int generated always as identity` (Primary Key).
  * `character_id` — `int NOT NULL UNIQUE` — Foreign Key до `Character` (тип зв'язку один до одного).
  * `experience` — `int NOT NULL default 0` — поточна кількість накопиченого досвіду.
  * `max_hp` — `int NOT NULL` — максимальний запас здоров'я.
  * `current_hp` — `int NOT NULL` — поточний рівень здоров'я.
  * `max_mana` — `int NOT NULL` — максимальний запас мани.
  * `current_mana` — `int NOT NULL` — поточний рівень мани.

---

## Inventory

Описує персональне сховище предметів та фінанси конкретного персонажа.

* **Атрибути:**
  * `id` — `int generated always as identity` (Primary Key).
  * `character_id` — `int NOT NULL UNIQUE` — Foreign Key до `Character` (тип зв'язку один до одного).
  * `gold` — `int NOT NULL default 0` — кількість грошей/золота у персонажа.
  * `capacity` — `int NOT NULL DEFAULT 20` — місткість інвентарю (максимальна кількість слотів/вага).

---

## Spell

Каталог усіх доступних у грі заклять та магічних здібностей.

* **Атрибути:**
  * `id` — `varchar[128] NOT NULL` (Primary Key) — текстовий ідентифікатор закляття (наприклад, `fireball`, `heal`).
  * `name` — `varchar[128] NOT NULL` — назва закляття для відображення в інтерфейсі клієнта.
  * `description` — `varchar[512] NOT NULL` — повний опис дії та ефектів закляття.
  * `mana_cost` — `int NOT NULL DEFAULT 0` — вартість застосування у мані.

---

## CharacterSpell

Асоціативна сутність для реалізації зв'язку багато до багатьох між `Character` та `Spell` (відображає книгу вивчених заклять персонажа та їх прив'язку до панелі швидкого доступу).

* **Атрибути:**
  * `character_id` — `int NOT NULL` — Foreign Key до `Character`.
  * `spell_id` — `varchar[128] NOT NULL` — Foreign Key до `Spell`.
  * `slot_index` — `int NULL` — номер слота на панелі швидкого доступу/дії (якщо призначено).

* **Кардинальність:**
  * З боку `CharacterSpell` до кожної з пов'язаних сутностей — строго один.
  * Персонаж може мати нуль або багато вивчених заклять (`Character` до `CharacterSpell` — нуль або багато).
  * Одне й те саме закляття може бути вивчене багатьма персонажами (`Spell` до `CharacterSpell` — нуль або багато).