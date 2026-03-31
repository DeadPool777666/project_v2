# Деплой NewsHub в продакшн

## 📋 Чек-лист перед деплоем

### 1. Firebase конфигурация
- [ ] Создан production Firebase проект
- [ ] Настроен Authentication (Email/Password)
- [ ] Создана Firestore Database
- [ ] Настроены правила безопасности Firestore (см. ниже)
- [ ] Созданы все необходимые индексы

### 2. Переменные окружения
- [ ] Файл `.env.production` создан
- [ ] Все Firebase переменные заполнены
- [ ] API ключи проверены

### 3. Безопасность
- [ ] Правила Firestore обновлены для продакшна
- [ ] Добавлен домен в Firebase Authentication
- [ ] CORS настроен правильно

### 4. Производительность
- [ ] Индексы Firestore созданы
- [ ] Изображения оптимизированы
- [ ] Bundle size проверен

## 🔐 Production Firestore Rules

Обновите правила в Firebase Console → Firestore → Rules:

```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    
    // Функция для проверки аутентификации
    function isSignedIn() {
      return request.auth != null;
    }
    
    // Функция для проверки роли админа
    function isAdmin() {
      return isSignedIn() && 
             get(/databases/$(database)/documents/users/$(request.auth.uid)).data.role == 'admin';
    }
    
    // Счетчик пользователей (только чтение)
    match /users/count {
      allow read: if true;
      allow write: if isAdmin();
    }
    
    // Пользователи
    match /users/{userId} {
      // Все авторизованные могут читать базовую информацию пользователей
      allow read: if isSignedIn();
      // Только владелец может обновлять свои данные
      allow write: if isSignedIn() && request.auth.uid == userId;
      // Создание только при регистрации
      allow create: if isSignedIn() && request.auth.uid == userId;
    }
    
    // Комментарии
    match /comments/{commentId} {
      // Все могут читать комментарии
      allow read: if true;
      
      // Создание комментариев
      allow create: if isSignedIn() 
                    && request.resource.data.authorId == request.auth.uid
                    && request.resource.data.newsId is string
                    && request.resource.data.text is string
                    && request.resource.data.text.size() > 0
                    && request.resource.data.text.size() <= 1000; // Макс 1000 символов
      
      // Удаление: автор или админ
      allow delete: if isSignedIn() && 
                    (resource.data.authorId == request.auth.uid || isAdmin());
      
      // Обновление запрещено (комментарии нельзя редактировать)
      allow update: if false;
    }
    
    // Избранные новости
    match /favorites/{favoriteId} {
      // Только владелец видит свои избранные
      allow read: if isSignedIn() && resource.data.userId == request.auth.uid;
      
      // Создание избранного
      allow create: if isSignedIn() 
                    && request.resource.data.userId == request.auth.uid
                    && request.resource.data.newsId is string;
      
      // Удаление избранного: только владелец
      allow delete: if isSignedIn() && resource.data.userId == request.auth.uid;
      
      // Обновление запрещено
      allow update: if false;
    }
  }
}
```

## 🌐 Настройка домена в Firebase

1. Firebase Console → Authentication → Settings
2. Authorized domains → Add domain
3. Добавьте ваш production домен

## 📦 Сборка

```bash
# Установка зависимостей
npm install

# Production build
npm run build

# Результат в папке dist/
```

## 🚀 Варианты деплоя

### Vercel (Рекомендуется)

1. Установите Vercel CLI:
```bash
npm i -g vercel
```

2. Деплой:
```bash
vercel --prod
```

3. Настройте Environment Variables в Vercel Dashboard:
- Settings → Environment Variables
- Добавьте все VITE_* переменные

### Netlify

1. Установите Netlify CLI:
```bash
npm i -g netlify-cli
```

2. Создайте `netlify.toml`:
```toml
[build]
  command = "npm run build"
  publish = "dist"

[[redirects]]
  from = "/*"
  to = "/index.html"
  status = 200
```

3. Деплой:
```bash
netlify deploy --prod
```

4. Настройте Environment Variables в Netlify Dashboard

### Firebase Hosting

1. Установите Firebase CLI:
```bash
npm i -g firebase-tools
```

2. Инициализация:
```bash
firebase init hosting
```

3. Выберите:
- Public directory: `dist`
- Single-page app: `yes`
- GitHub deploys: `optional`

4. Деплой:
```bash
npm run build
firebase deploy --only hosting
```

## 🔍 Тестирование production build локально

```bash
# Сборка
npm run build

# Запуск локального сервера
npx serve -s dist
```

Откройте http://localhost:3000

## ⚡ Оптимизация

### 1. Изображения
- Используйте WebP формат
- Настройте lazy loading (уже реализовано)
- CDN для статических файлов

### 2. Bundle Size
Проверьте размер бандла:
```bash
npm run build -- --stats
```

### 3. Caching
Настройте кэширование в хостинг-провайдере:
- HTML: no-cache
- JS/CSS: max-age=31536000 (1 год)
- Изображения: max-age=604800 (1 неделя)

## 📊 Мониторинг

### Firebase Console
- Authentication → Users (мониторинг пользователей)
- Firestore → Data (проверка данных)
- Analytics (опционально)

### Логи ошибок
Рекомендуется добавить:
- Sentry для JavaScript ошибок
- Firebase Crashlytics
- Google Analytics

## 🔒 Безопасность

### Дополнительные меры:
1. **Rate limiting** - ограничьте количество запросов
2. **reCAPTCHA** - защита от ботов при регистрации
3. **Email verification** - подтверждение email (настройка в Firebase)
4. **Password strength** - минимальные требования к паролю

## 🐛 Отладка в production

### Включение логов
В production логи Firebase по умолчанию отключены. Для временной отладки:

```javascript
// В firebase.ts (временно!)
import { setLogLevel } from "firebase/app";
setLogLevel("debug");
```

**Не забудьте отключить перед финальным деплоем!**

### Проверка правил Firestore
Firebase Console → Firestore → Rules → Rules Playground

## 📈 После деплоя

### Обязательно:
1. [ ] Протестируйте регистрацию
2. [ ] Протестируйте вход
3. [ ] Создайте первого пользователя (админа)
4. [ ] Проверьте создание новостей
5. [ ] Проверьте комментарии
6. [ ] Проверьте избранное
7. [ ] Протестируйте на мобильных устройствах

### Рекомендуется:
1. [ ] Настроить аналитику
2. [ ] Настроить мониторинг ошибок
3. [ ] Создать backup стратегию для Firestore
4. [ ] Настроить CI/CD pipeline

## 🆘 Частые проблемы

### "Firebase: Error (auth/unauthorized-domain)"
➡️ Добавьте домен в Firebase Authentication → Settings → Authorized domains

### "Firestore: Missing or insufficient permissions"
➡️ Проверьте правила Firestore и что пользователь авторизован

### "The query requires an index"
➡️ Перейдите по ссылке из ошибки для создания индекса

### Медленная загрузка
➡️ Проверьте:
- Индексы Firestore созданы?
- Изображения оптимизированы?
- CDN настроен?

## 📚 Полезные ссылки

- [Firebase Console](https://console.firebase.google.com/)
- [Vercel Dashboard](https://vercel.com/dashboard)
- [Netlify Dashboard](https://app.netlify.com/)
- [Firebase Hosting Docs](https://firebase.google.com/docs/hosting)

---

**Важно:** После деплоя сохраните копию всех конфигурационных файлов и переменных окружения в безопасном месте!
