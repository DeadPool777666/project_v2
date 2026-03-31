# Настройка NewsData.io API для автоматического обновления новостей

## Обзор

NewsHub использует [NewsData.io API](https://newsdata.io) для автоматического получения новостей каждый час. NewsData.io предоставляет **200 бесплатных запросов в день**, что более чем достаточно для регулярного обновления новостей.

## Получение API ключа

1. Перейдите на [https://newsdata.io](https://newsdata.io)
2. Нажмите "Get API Key" или "Sign Up Free"
3. Зарегистрируйтесь с помощью email или GitHub
4. После регистрации вы получите API ключ на панели управления
5. Скопируйте ваш API ключ (выглядит как: `pub_123456abcdefg...`)

## Настройка в Supabase

### Шаг 1: Добавьте секрет в Edge Function

1. Откройте ваш проект в [Supabase Dashboard](https://supabase.com/dashboard)
2. Перейдите в **Edge Functions** → **Settings** → **Secrets**
3. Нажмите "Add Secret"
4. Введите:
   - **Name:** `NEWSDATA_API_KEY`
   - **Value:** Ваш API ключ из NewsData.io
5. Нажмите "Save"

### Шаг 2: Деплой Edge Function (если необходимо)

Если вы вносили изменения в код Edge Function, выполните деплой:

```bash
supabase functions deploy make-server-e2f30d05
```

## Проверка работы

### Проверка через консоль браузера

1. Откройте ваш сайт NewsHub
2. Откройте Developer Tools (F12) → Console
3. Через 10 секунд после загрузки страницы вы увидите логи:
   ```
   Triggering news fetch from NewsData.io API...
   News fetch triggered: {...}
   Syncing news from backend to Firebase...
   Successfully synced X new articles to Firebase
   ```

### Ручная проверка через API

Вы можете вручную вызвать endpoint для получения новостей:

```bash
curl -X GET \
  "https://YOUR_PROJECT_ID.supabase.co/functions/v1/make-server-e2f30d05/fetch-news" \
  -H "Authorization: Bearer YOUR_ANON_KEY"
```

Замените:
- `YOUR_PROJECT_ID` на ID вашего проекта Supabase
- `YOUR_ANON_KEY` на публичный анонимный ключ

## Как работает автоматическое обновление

1. **При загрузке приложения** (через 10 секунд):
   - Frontend вызывает `fetchNewsFromBackend()`
   - Backend получает новости из NewsData.io API
   - Новости сохраняются в Supabase KV Store
   - Новости синхронизируются с Firebase Firestore

2. **Каждый час**:
   - Процесс повторяется автоматически
   - Проверяются дубликаты (новости не дублируются)
   - Только новые статьи добавляются в базу

## Лимиты бесплатного тарифа

NewsData.io Free Plan:
- ✅ 200 запросов в день
- ✅ Доступ к актуальным новостям
- ✅ Несколько категорий (technology, business, sports, science, general)
- ✅ Новости на английском и русском языках
- ❌ Исторические данные за последние 2 дня

При обновлении каждый час = 24 запроса в день, что значительно меньше лимита.

## Альтернативные API (если нужно)

Если вы хотите использовать другой API для новостей:

### The Guardian API
- **Бесплатно:** 5000 запросов/день
- **Регистрация:** [https://open-platform.theguardian.com/access/](https://open-platform.theguardian.com/access/)
- Измените код в `/supabase/functions/server/index.tsx`

### Currents API
- **Бесплатно:** 600 запросов/день
- **Регистрация:** [https://currentsapi.services/en/register](https://currentsapi.services/en/register)

## Устранение неполадок

### Ошибка "API key not configured"

**Решение:**
1. Убедитесь, что вы добавили `NEWSDATA_API_KEY` в Supabase Secrets
2. Проверьте правильность написания имени секрета
3. Выполните повторный деплой Edge Function

### Новости не обновляются

**Решение:**
1. Откройте консоль браузера и проверьте логи
2. Проверьте логи Edge Function в Supabase Dashboard
3. Убедитесь, что API ключ активен и не исчерпан лимит

### Ошибка "Failed to fetch news from backend"

**Решение:**
1. Проверьте, что Edge Function задеплоена
2. Проверьте настройки CORS в Edge Function
3. Убедитесь, что Firebase правильно настроен

## Мониторинг использования API

1. Войдите в [NewsData.io Dashboard](https://newsdata.io/dashboard)
2. Проверьте количество использованных запросов
3. Мониторьте лимиты и статистику

## Контакты и поддержка

- **NewsData.io Support:** [https://newsdata.io/contact](https://newsdata.io/contact)
- **Документация API:** [https://newsdata.io/documentation](https://newsdata.io/documentation)
