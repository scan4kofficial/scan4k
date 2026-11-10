# Как запустить Scan4k на GitHub с рабочими кнопками

Кнопки управления видишь только ты. Остальные видят сайт и могут голосовать. Данные хранятся в Firebase (бесплатно). GitHub отдаёт только саму страницу.

## 1. Создай проект Firebase
1. Открой https://console.firebase.google.com и войди через Google.
2. «Add project» → любое имя (например scan4k) → Google Analytics можно выключить.

## 2. Включи вход
Build → Authentication → Get started → Sign-in method:
- включи **Anonymous** (для зрителей, чтобы голосовать);
- включи **Google** (для тебя, чтобы менять игры).

Потом Authentication → Settings → Authorized domains → Add domain:
`ТВОЙ_ЛОГИН.github.io`

## 3. Создай базу
Build → Firestore Database → Create database → любой регион → **Production mode**.

Вкладка **Rules** → замени всё на это (поменяй почту на свою) → Publish:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /config/games {
      allow read: if true;
      allow write: if request.auth != null
        && request.auth.token.email == 'ТВОЯ_ПОЧТА@gmail.com'
        && request.auth.token.email_verified == true;
    }
    match /votes/{uid} {
      allow read: if true;
      allow write: if request.auth != null
        && request.auth.uid == uid
        && request.resource.data.keys().hasOnly(['g', 'ver'])
        && request.resource.data.g is int
        && request.resource.data.g >= 0
        && request.resource.data.g < 3
        && request.resource.data.ver is int;
    }
  }
}
```

## 4. Возьми ключи
Project settings (шестерёнка) → Your apps → значок `</>` (Web) → зарегистрируй приложение → скопируй `firebaseConfig`.

Открой `index.html`, найди блок «НАСТРОЙКИ» и вставь:
- apiKey, authDomain, projectId, appId из firebaseConfig;
- OWNER_EMAIL — твоя Gmail, та же, что в правилах.

(apiKey в Firebase не секрет, защита идёт через правила из шага 3.)

## 5. Выложи на GitHub
Создай репозиторий → загрузи `index.html` и `scene.webp` → Settings → Pages → Branch: main, папка /root → Save.
Сайт будет на `https://ТВОЙ_ЛОГИН.github.io/ИМЯ_РЕПО/`.

## Как пользоваться
Открой сайт со словом #owner в конце адреса (например https://ТВОЙ_ЛОГИН.github.io/ИМЯ_РЕПО/#owner) → внизу блока с играми появится кнопка «Вход для владельца» → войди своей Gmail → появятся «Изменить игры», «Сохранить», «Сбросить голоса».

## Время стримов
После входа владельца под расписанием появляется кнопка «Изменить время»: у каждого дня можно поставить время или отметить «выходной». Таймер до следующего стрима подстраивается сам. Правила базы менять не нужно.

## Метка «Сейчас в эфире»
После входа владельца под расписанием есть кнопка «🔴 Начать эфир». Пока она включена, вверху сайта горит метка «Сейчас в эфире» со ссылкой на Twitch, а таймер показывает «Сейчас в эфире». Кнопка «⏹ Закончить эфир» выключает метку. Если забыть выключить, метка погаснет сама через 12 часов.

## Победитель голосования
У игры, за которую отдано больше всего голосов, появляется золотая метка «👑 Побеждает». При равенстве у лидеров метка «⚖ Ничья». Кнопка владельца «👑 Победителя на сегодня» записывает эту игру в блок «Сегодня на стриме».
