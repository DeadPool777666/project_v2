# 🚀 Быстрый старт - NewsHub

## За 5 минут до запуска

### 1️⃣ Установка (30 сек)
```bash
npm install
```

### 2️⃣ Создание Firebase проекта (2 мин)
1. Перейдите на https://console.firebase.google.com/
2. Нажмите "Add project"
3. Дайте название проекту
4. Дождитесь создания

### 3️⃣ Настройка Firebase (2 мин)

#### Authentication:
1. Откройте **Authentication** → Get started
2. Выберите **Email/Password** → Enable → Save

#### Firestore:
1. Откройте **Firestore Database** → Create database
2. Выберите **Start in test mode** → Next
3. Выберите регион → Enable

#### Получите конфигурацию:
1. Откройте **Project Settings** (шестеренка сверху)
2. Прокрутите вниз до "Your apps"
3. Нажмите на иконку веб-приложения `</>`
4. Скопируйте конфигурацию

### 4️⃣ Создайте файл .env (30 сек)
Создайте файл `.env` в корне проекта:

```env
VITE_FIREBASE_API_KEY=AIzaSy...
VITE_FIREBASE_AUTH_DOMAIN=your-project.firebaseapp.com
VITE_FIREBASE_PROJECT_ID=your-project
VITE_FIREBASE_STORAGE_BUCKET=your-project.appspot.com
VITE_FIREBASE_MESSAGING_SENDER_ID=123456789
VITE_FIREBASE_APP_ID=1:123456789:web:abc123
```

### 5️⃣ Запуск (10 сек)
```bash
npm run dev
```

## ✅ Готово! 

Откройте http://localhost:5173

## 🎯 Первые шаги

1. **Зарегистрируйтесь** (используйте любой email/пароль)
   - Первый пользователь автоматически станет администратором

2. **Посмотрите демо-новости** на главной странице

3. **Попробуйте функции:**
   - 💬 Оставьте комментарий под новостью
   - ❤️ Добавьте новость в избранное
   - 🛡️ (Админ) Создайте свою новость через "Админ панель"

## 📚 Полная документация

- [README.md](./README.md) - Полная документация
- [FIREBASE_SETUP.md](./FIREBASE_SETUP.md) - Детальная настройка Firebase
- [FIRESTORE_INDEXES.md](./FIRESTORE_INDEXES.md) - Настройка индексов

## ⚠️ Важно!

### Индексы Firestore
При первом использовании комментариев/избранного вы увидите ошибку в консоли с ссылкой для создания индекса:
1. Перейдите по ссылке из ошибки
2. Дождитесь создания индекса (1-5 минут)
3. Перезагрузите страницу

### Правила безопасности
Для продакшена настройте правила в **Firestore → Rules**:
```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /users/{userId} {
      allow read: if request.auth != null;
      allow write: if request.auth != null && request.auth.uid == userId;
    }
    
    match /comments/{commentId} {
      allow read: if true;
      allow create: if request.auth != null;
      allow delete: if request.auth != null && 
        (resource.data.authorId == request.auth.uid || 
         get(/databases/$(database)/documents/users/$(request.auth.uid)).data.role == 'admin');
    }
    
    match /favorites/{favoriteId} {
      allow read: if request.auth != null && resource.data.userId == request.auth.uid;
      allow create, delete: if request.auth != null && request.resource.data.userId == request.auth.uid;
    }
  }
}
```

## 🆘 Проблемы?

### Firebase не инициализируется
- ✅ Проверьте `.env` файл
- ✅ Убедитесь, что включены Authentication и Firestore

### Ошибки при ��омментариях
- ✅ Создайте индексы (см. выше)
- ✅ Проверьте правила Firestore

### Не работает вход/регистрация
- ✅ Убедитесь, что Email/Password включен в Authentication

## 🎨 Функции

✅ Firebase Authentication - регистрация и вход
✅ Firestore - комментарии и избранное
✅ Система ролей (admin/user)
✅ CRUD новостей для админов
✅ Комментарии для всех пользователей
✅ Избранные новости ❤️
✅ Фильтрация по категориям
✅ Адаптивный дизайн

---

**Совет:** Сначала зарегистрируйтесь, чтобы стать админом и получить полный доступ!
