# Qamshy Made — Инструкция по запуску

## Структура проекта

```
qamshy-made/
├── index.html      ← Магазин (витрина)
├── admin.html      ← Админ панель
├── vercel.json     ← Конфиг Vercel
└── README.md       ← Эта инструкция
```

---

## Шаг 1 — Настройка Supabase (база данных)

### 1.1 Создать проект
1. Зайдите на **https://supabase.com** и зарегистрируйтесь
2. Нажмите **"New project"**
3. Введите название: `qamshy-made`, выберите пароль, регион — `Frankfurt (eu-central-1)`
4. Нажмите **"Create new project"** (подождите ~2 минуты)

### 1.2 Создать таблицы
1. В левом меню перейдите: **SQL Editor** → **New query**
2. Вставьте весь SQL ниже и нажмите **Run**:

```sql
-- Товары
CREATE TABLE products (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  name TEXT NOT NULL,
  cat TEXT,
  price INTEGER DEFAULT 0,
  old_price INTEGER DEFAULT 0,
  label TEXT,
  desc TEXT,
  img TEXT,
  in_stock BOOLEAN DEFAULT true,
  created_at TIMESTAMPTZ DEFAULT now(),
  updated_at TIMESTAMPTZ DEFAULT now()
);

-- Заказы
CREATE TABLE orders (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  order_num TEXT,
  user_name TEXT,
  phone TEXT,
  city TEXT,
  address TEXT,
  delivery TEXT,
  payment TEXT,
  items JSONB DEFAULT '[]',
  total INTEGER DEFAULT 0,
  status TEXT DEFAULT 'new',
  comment TEXT,
  created_at TIMESTAMPTZ DEFAULT now()
);

-- Профили пользователей
CREATE TABLE profiles (
  id UUID REFERENCES auth.users PRIMARY KEY,
  full_name TEXT,
  phone TEXT,
  email TEXT,
  role TEXT DEFAULT 'customer',
  created_at TIMESTAMPTZ DEFAULT now()
);

-- Администраторы
CREATE TABLE admins (
  id UUID REFERENCES auth.users PRIMARY KEY,
  granted_at TIMESTAMPTZ DEFAULT now()
);

-- Заявки с сайта (из формы контактов)
CREATE TABLE leads (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  name TEXT,
  phone TEXT,
  interest TEXT,
  comment TEXT,
  created_at TIMESTAMPTZ DEFAULT now()
);

-- Включить Row Level Security
ALTER TABLE products ENABLE ROW LEVEL SECURITY;
ALTER TABLE orders ENABLE ROW LEVEL SECURITY;
ALTER TABLE profiles ENABLE ROW LEVEL SECURITY;
ALTER TABLE admins ENABLE ROW LEVEL SECURITY;
ALTER TABLE leads ENABLE ROW LEVEL SECURITY;

-- Товары: читать всем, писать только через anon ключ (фронтенд)
CREATE POLICY "products_public_read" ON products FOR SELECT USING (true);
CREATE POLICY "products_admin_write" ON products FOR ALL
  USING (EXISTS (SELECT 1 FROM admins WHERE id = auth.uid()));

-- Заказы: создавать всем, управлять только админам
CREATE POLICY "orders_insert_public" ON orders FOR INSERT WITH CHECK (true);
CREATE POLICY "orders_admin_all" ON orders FOR ALL
  USING (EXISTS (SELECT 1 FROM admins WHERE id = auth.uid()));

-- Заявки: создавать всем
CREATE POLICY "leads_insert_public" ON leads FOR INSERT WITH CHECK (true);
CREATE POLICY "leads_admin_read" ON leads FOR SELECT
  USING (EXISTS (SELECT 1 FROM admins WHERE id = auth.uid()));

-- Профили: свой профиль
CREATE POLICY "profiles_own" ON profiles FOR ALL USING (id = auth.uid());
CREATE POLICY "profiles_admin_read" ON profiles FOR SELECT
  USING (EXISTS (SELECT 1 FROM admins WHERE id = auth.uid()));

-- Администраторы: читать авторизованным
CREATE POLICY "admins_read" ON admins FOR SELECT USING (auth.uid() IS NOT NULL);
CREATE POLICY "admins_write" ON admins FOR ALL
  USING (EXISTS (SELECT 1 FROM admins WHERE id = auth.uid()));
```

### 1.3 Включить авторизацию по Email
1. Перейдите: **Authentication** → **Providers**
2. Убедитесь что **Email** включён

### 1.4 Создать первого администратора
1. Перейдите: **Authentication** → **Users** → **Add user**
2. Введите email и пароль (например: `admin@qamshy.kz` / `Admin123!`)
3. После создания скопируйте **User UID** (длинная строка вида `xxxxxxxx-xxxx-...`)
4. Перейдите: **SQL Editor** → **New query**, выполните:

```sql
INSERT INTO admins (id) VALUES ('07758d05-e53b-49e5-953b-42a4553adb71');
```

### 1.5 Получить ключи API
1. Перейдите: **Project Settings** → **API**
2. Скопируйте:
   - **Project URL** → вставьте в `SUPABASE_URL`
   - **anon public** (ключ) → вставьте в `SUPABASE_ANON_KEY`

---

## Шаг 2 — Вставить ключи в файлы

### В `index.html` (строки ~1487–1492):
```javascript
const SUPABASE_URL = 'https://xxxxx.supabase.co';  // ← ваш URL
const SUPABASE_ANON_KEY = 'eyJhbGciO...';           // ← ваш anon ключ
const WA_NUMBER = '77012345678';                     // ← ваш WhatsApp
const KASPI_PHONE = '+7 (701) 234-56-78';           // ← ваш Kaspi номер
```

### В `admin.html` (строки ~2–3):
```javascript
const SUPABASE_URL = 'https://xxxxx.supabase.co';  // ← те же данные
const SUPABASE_ANON_KEY = 'eyJhbGciO...';
```

---

## Шаг 3 — Деплой на Vercel

### Способ A — через GitHub (рекомендуется)

1. Создайте аккаунт на **https://github.com**
2. Создайте новый репозиторий (кнопка `+` → "New repository"), назовите `qamshy-made`
3. Загрузите все 4 файла: `index.html`, `admin.html`, `vercel.json`, `README.md`
   - На странице репозитория нажмите **"Add file"** → **"Upload files"**
4. Зайдите на **https://vercel.com** → "Sign in with GitHub"
5. Нажмите **"New Project"** → выберите репозиторий `qamshy-made`
6. Нажмите **"Deploy"** — готово!

### Способ B — через Vercel CLI

```bash
npm install -g vercel
cd папка-с-файлами
vercel
# Следуйте инструкциям в терминале
```

---

## Готово! Ваш сайт будет доступен по адресам:
- **Магазин:** `https://qamshy-made.vercel.app`
- **Админка:** `https://qamshy-made.vercel.app/admin`

---

## Как добавить товары

1. Откройте `/admin` → войдите с email/паролем из Шага 1.4
2. Перейдите в **Товары** → нажмите **+ Добавить товар**
3. Заполните: название, категория, цена, фото (URL)
4. Товар сразу появится на витрине

## Как работает оплата Kaspi

При оформлении заказа через сайт:
1. Заказ сохраняется в базе Supabase
2. Автоматически открывается WhatsApp с деталями заказа
3. В админке вы видите заказ со статусом "Новый"
4. Отправляете клиенту реквизиты Kaspi в WhatsApp вручную
5. После оплаты меняете статус заказа в админке

---

## Вопросы?

Если что-то не работает — проверьте:
1. Правильно ли вставлены ключи Supabase в оба файла
2. Выполнен ли SQL-скрипт в Supabase
3. Создан ли первый admin в таблице `admins`
