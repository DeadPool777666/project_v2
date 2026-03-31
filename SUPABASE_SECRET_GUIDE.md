# Добавление NEWSDATA_API_KEY в Supabase Edge Function

## Визуальная пошаговая инструкция

### Шаг 1: Откройте Supabase Dashboard
1. Перейдите на [https://supabase.com/dashboard](https://supabase.com/dashboard)
2. Войдите в свой аккаунт
3. Выберите ваш проект

### Шаг 2: Перейдите в настройки Edge Functions
1. В левом меню найдите **"Edge Functions"**
2. Нажмите на **"Edge Functions"**
3. Если вы видите список функций, отлично! Если нет, проверьте что функция задеплоена

### Шаг 3: Откройте настройки секретов
1. В верхней части страницы Edge Functions найдите кнопку **"Settings"** или **"Manage secrets"**
2. Или перейдите напрямую: Project Settings → Edge Functions → Secrets
3. Вы увидите список секретов (если есть)

### Шаг 4: Добавьте новый секрет
1. Нажмите кнопку **"Add secret"** или **"New secret"**
2. В поле **"Name"** введите точно: `NEWSDATA_API_KEY`
3. В поле **"Value"** вставьте ваш API ключ из NewsData.io (например: `pub_123456abcdefg...`)
4. Нажмите **"Save"** или **"Add"**

### Шаг 5: Проверка
1. Вернитесь на ваш сайт NewsHub
2. Откройте админ панель
3. Через несколько секунд статус должен измениться на "Автоматическое обновление новостей включено"
4. Проверьте консоль браузера (F12) - через 10 секунд должен появиться лог о загрузке новостей

## Альтернативный способ (через CLI)

Если вы используете Supabase CLI:

```bash
# Установите Supabase CLI (если еще не установлен)
npm install -g supabase

# Войдите в свой аккаунт
supabase login

# Добавьте секрет
supabase secrets set NEWSDATA_API_KEY=pub_your_api_key_here
```

## Получение API ключа NewsData.io

1. Перейдите на [https://newsdata.io](https://newsdata.io)
2. Нажмите **"Get API Key"** или **"Sign Up Free"**
3. Зарегистрируйтесь (можно через email или GitHub)
4. После регистрации вы увидите ваш API ключ на главной странице дашборда
5. Ключ выглядит примерно так: `pub_123456abcdefghijklmnop1234567890`
6. Скопируйте его и используйте в Supabase

## Важные замечания

⚠️ **Имя секрета должно быть точно:** `NEWSDATA_API_KEY`
⚠️ **Без кавычек:** Просто вставьте ключ, без `"` или `'`
⚠️ **Перезагрузка:** После добавления секрета может потребоваться несколько секунд для применения

## Проверка работы

### В консоли браузера вы должны увидеть:
```
Triggering news fetch from NewsData.io API...
News fetch result: {success: true, saved: 10, total: 20}
Syncing news from backend to Firebase...
Successfully synced 10 new articles to Firebase
```

### Если видите ошибку:
```
Backend news fetch info: {"success":false,"message":"API key not configured..."}
```

Значит:
- Секрет не добавлен
- Неправильное имя секрета (проверьте написание)
- Нужно подождать несколько минут после добавления

## Скриншоты путей в Supabase Dashboard

```
Supabase Dashboard
  └── Your Project
      └── Settings (⚙️)
          └── Edge Functions
              └── Secrets
                  └── [Add Secret]
                      ├── Name: NEWSDATA_API_KEY
                      └── Value: pub_your_key_here
```

ИЛИ

```
Supabase Dashboard
  └── Your Project
      └── Edge Functions
          └── [Manage Secrets] (кнопка сверху)
              └── [Add Secret]
                  ├── Name: NEWSDATA_API_KEY
                  └── Value: pub_your_key_here
```

## Лимиты бесплатного тарифа

✅ **NewsData.io Free Plan:**
- 200 запросов в день
- Текущие новости (без архива)
- Несколько категорий
- Поддержка английского и русского языков

✅ **Использование в NewsHub:**
- ~24 запроса в день (обновление раз в час)
- **Запас:** 176 запросов для тестирования

## Готово!

После выполнения этих шагов ваш сайт будет автоматически получать свежие новости каждый час! 🎉

---

**Нужна помощь?** Проверьте [NEWSDATA_API_SETUP.md](./NEWSDATA_API_SETUP.md) для более детальной информации.
