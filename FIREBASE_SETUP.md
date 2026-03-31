# Инструкция по настройке Firebase для NewsHub

## Шаг 1: Создание проекта Firebase

1. Перейдите на [Firebase Console](https://console.firebase.google.com/)
2. Нажмите "Add project" (Добавить проект)
3. Введите название проекта (например, "NewsHub")
4. Следуйте инструкциям для завершения создания проекта

## Шаг 2: Регистрация веб-приложения

1. В консоли Firebase выберите свой проект
2. Нажмите на значок веб-приложения (</>) в разделе "Get started"
3. Введите название приложения
4. Скопируйте конфигурацию Firebase (firebaseConfig)

## Шаг 3: Настройка переменных окружения

Создайте файл `.env` в корне проекта и добавьте следующие переменные:

```env
VITE_FIREBASE_API_KEY=your_api_key_here
VITE_FIREBASE_AUTH_DOMAIN=your_project.firebaseapp.com
VITE_FIREBASE_PROJECT_ID=your_project_id
VITE_FIREBASE_STORAGE_BUCKET=your_project.appspot.com
VITE_FIREBASE_MESSAGING_SENDER_ID=your_sender_id
VITE_FIREBASE_APP_ID=your_app_id

# Опционально: для автоматического обновления новостей из News API
VITE_NEWS_API_KEY=your_news_api_key_here
```

## Шаг 4: Включение Authentication

1. В Firebase Console перейдите в раздел **Authentication**
2. Нажмите "Get started"
3. Во вкладке "Sign-in method" включите **Email/Password**
4. Сохраните изменения

## Шаг 5: Создание Firestore Database

1. В Firebase Console перейдите в раздел **Firestore Database**
2. Нажмите "Create database"
3. Выберите режим **Start in test mode** (для разработки)
4. Выберите регион сервера (ближайший к вам)

## Шаг 6: Настройка правил безопасности Firestore

В разделе **Firestore Database → Rules** замените правила на следующие:

```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    // Пользователи могут читать свои данные
    match /users/{userId} {
      allow read: if request.auth != null;
      allow write: if request.auth != null && request.auth.uid == userId;
    }
    
    // Счетчик пользователей
    match /users/count {
      allow read: if true;
      allow write: if request.auth != null;
    }
    
    // Комментарии
    match /comments/{commentId} {
      // Все могут читать комментарии
      allow read: if true;
      // Аутентифицированные пользователи могут создавать
      allow create: if request.auth != null;
      // Пользователь может удалять свои комментарии, админ - любые
      allow delete: if request.auth != null && 
        (resource.data.authorId == request.auth.uid || 
         get(/databases/$(database)/documents/users/$(request.auth.uid)).data.role == 'admin');
    }
    
    // Избранные новости
    match /favorites/{favoriteId} {
      // Пользователи могут читать только свои избранные
      allow read: if request.auth != null && resource.data.userId == request.auth.uid;
      // Пользователи могут добавлять и удалять свои избранные
      allow create, delete: if request.auth != null && request.resource.data.userId == request.auth.uid;
    }
  }
}
```

**Примечание:** Эти правила предназначены для разработки. Для продакшена рекомендуется более строгая настройка.

## Шаг 6.5: Создание индексов Firestore

После первого использования приложения Firestore может запросить создание индексов для оптимизации запросов. Это нормально!

**Автоматический способ:**
1. Откройте консоль браузера (F12)
2. При выполнении действий (просмотр комментариев, избранного) появится ошибка с ссылкой
3. Перейдите по ссылке - индекс будет создан автоматически
4. Подождите несколько минут

**Подробнее:** см. [FIRESTORE_INDEXES.md](./FIRESTORE_INDEXES.md)

## Шаг 7: (Опционально) Получение API ключа News API

Для автоматического обновления новостей:

1. Перейдите на [newsapi.org](https://newsapi.org/)
2. Зарегистрируйтесь и получите бесплатный API ключ
3. Добавьте ключ в `.env` файл как `VITE_NEWS_API_KEY`

## Шаг 8: Запуск приложения

```bash
npm install
npm run dev
```

## Важные заметки:

- **Первый пользователь** автоматически получает права администратора
- Администра��ор может создавать, редактировать и удалять новости
- Все авторизованные пользователи могут оставлять комментарии
- Все авторизованные пользователи могут добавлять новости в избранное
- Новости автоматически обновляются каждый час (если настроен News API)
- В режиме разработки используйте тестовые правила Firestore
- Перед деплоем в продакшн настройте более строгие правила безопасности

## Структура данных в Firestore:

### Коллекция `users`:
```javascript
{
  uid: string,
  email: string,
  displayName: string,
  role: "admin" | "user",
  createdAt: string
}
```

### Коллекция `comments`:
```javascript
{
  newsId: string,
  author: string,
  authorId: string,
  text: string,
  createdAt: string
}
```

### Коллекция `favorites`:
```javascript
{
  userId: string,
  newsId: string,
  addedAt: string
}
```

### Примечание о новостях:
Новости хранятся в localStorage для упрощения демонстрации. Если вы хотите перенести их в Firestore, следуйте аналогичному подходу с коллекцией `news`.