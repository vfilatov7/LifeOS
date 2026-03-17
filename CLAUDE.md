# Life OS — Project Context

## What is this?

**Life OS v10.2** — персональная операционная система жизни. Single-page web app на чистом HTML/CSS/JS (без фреймворков), геймифицирующая личное развитие через RPG-механики: XP, уровни, квесты, достижения.

Универсальное приложение для любого пользователя — трекинг продуктивности, здоровья, финансов, медиа и путешествий.

---

## Файловая структура

```
LifeOS/
├── life-os-v10.2.html      # Основной файл приложения (source of truth)
├── index.html               # Копия для GitHub Pages (всегда синхронизировать!)
├── manifest.json            # PWA манифест
├── sw.js                    # Service Worker (кэширование, офлайн)
├── icons/                   # PWA иконки (192, 512)
├── .github/workflows/
│   └── pages.yml            # GitHub Actions → GitHub Pages деплой
└── supabase/functions/
    └── gemini-proxy/
        └── index.ts         # Supabase Edge Function — прокси Gemini API
```

**ВАЖНО:** После правок в `life-os-v10.2.html` всегда копировать в `index.html`:
```bash
cp life-os-v10.2.html index.html
```

---

## Деплой

### GitHub Pages (фронтенд)
- **URL:** https://vfilatov7.github.io/LifeOS/
- **Репо:** https://github.com/vfilatov7/LifeOS
- **Деплой:** автоматически при `git push` в `main` (GitHub Actions)
- **Как деплоить:**
  ```bash
  cp life-os-v10.2.html index.html
  git add -A && git commit -m "описание" && git push
  ```

### Supabase (AI прокси)
- **Проект:** LifeOS
- **URL:** https://evylfgkftjkmtosvbmdd.supabase.co
- **Edge Function:** `gemini-proxy` — проксирует запросы к Gemini API
- **Секрет:** `GEMINI_API_KEY` — хранится в Supabase Secrets
- **Как деплоить функцию:**
  ```bash
  supabase functions deploy gemini-proxy --no-verify-jwt
  ```
- **Как обновить ключ:**
  ```bash
  supabase secrets set GEMINI_API_KEY=новый_ключ
  ```

### PWA
- `manifest.json` — метаданные приложения, иконки, standalone режим
- `sw.js` — Service Worker: кэширует HTML/CSS/шрифты, пропускает API-запросы
- Установка: Android → "Добавить на главный экран", iOS Safari → "Поделиться" → "На экран Домой"

---

## Модули и функциональность

| Вкладка | Содержание |
|---------|-----------|
| Задания | Дневные квесты по жизненным столпам, AI-генерация, завершение с таймером |
| Таймер | Pomodoro-таймер с пиксельным компаньоном и анимациями |
| Еда | Трекер КБЖУ, AI-логирование питания, сохранённые блюда |
| Деньги | Расходы/доходы по категориям, бюджеты, ежемесячная аналитика |
| Медиа | Библиотека фильмов/книг/игр с рейтингами и заметками |
| Мир | Интерактивная карта мира (Leaflet + CartoDB) с посещёнными и планируемыми странами |
| Отчёты | Комплексный отчёт: категории, питание (КБЖУ compliance grid), финансы, медиа, путешествия, AI-резюме |
| Настройки | Профиль, цели питания, управление спринтом |
| 🌙 | Screensaver-режим с анимированными градиентами |

---

## Архитектура

### AI
- **Модель:** Google Gemini 2.5 Flash
- **Прокси:** Supabase Edge Function (`gemini-proxy`) — ключ на сервере, клиент не знает ключ
- **Fallback:** если прокси URL не настроен (содержит `%%`), используется прямой вызов с локальным ключом
- **Вызовы AI:** квесты, еда (КБЖУ), деньги (категоризация), AI-отчёт

### Хранилище
Всё состояние в **localStorage** под ключом `lifeos3` как JSON-объект `st`.

```js
{
  name: string,           // имя пользователя (вводит сам)
  totalXP: number,        // суммарный опыт
  quests: [{id, name, cat, xp, mins, done, ts}],
  questDate: string,      // дата генерации квестов
  streak: number,         // дней подряд
  achievements: [...],    // достижения
  food: {
    entries: [{name, icon, kcal, protein, fat, carbs, fiber, sugar, grams}],
    goals: {kcal, protein, fat, carbs, fiber, sugar},
    menu: [...]           // сохранённые блюда
  },
  history: [...],         // история завершённых активностей
  foodHistory: [...],
  money: {
    entries: [{name, amount, cat, type, icon, ts}],
    budgets: {...}        // лимиты по категориям
  },
  chatHistory: [...],     // AI-чат квестов
  foodChatHistory: [...], // AI-чат еды
  moneyChatHistory: [...],// AI-чат денег
  media: [...],           // фильмы/книги/игры
  travel: [...]           // поездки
}
```

Прочие localStorage ключи:
- `lifeos_gemini_key` — локальный ключ Gemini (fallback, обычно не нужен)
- `lifeos_sprint_start` — дата начала спринта
- `lifeos_timer` — состояние таймера

---

## Жизненные столпы и категории

| Столп | Категории | XP множитель |
|-------|-----------|--------------|
| Карьера | work, code | work×0.5, code×1.2 |
| Навыки | learn, read | ×1.0 |
| Тело | sport, walk, cook | sport×1.5, walk×0.8, cook×0.6 |
| Отдых | rest, gaming, social, hobby | rest/gaming×0.3, social×0.4, hobby×0.5 |

**Прогрессия уровней:** 200 XP на уровень

---

## Технологии

- **Frontend:** Pure HTML5 + CSS3 + Vanilla JS (нет фреймворков/зависимостей)
- **Шрифты:** DM Sans, DM Mono (Google Fonts)
- **AI:** Google Gemini 2.5 Flash через Supabase Edge Function прокси
- **Хранилище:** localStorage (планируется миграция на Supabase DB)
- **Карта:** Leaflet + CartoDB Positron tiles + TopoJSON
- **Хостинг:** GitHub Pages (фронтенд) + Supabase (AI прокси)
- **PWA:** manifest.json + Service Worker

---

## UI особенности

- **Glassmorphism**: backdrop-blur, полупрозрачные карточки
- **CSS анимации**: XP pop-ups, breathing pulse таймера, анимации компаньона (смена каждые 700мс)
- **Адаптивность**: 4→2→1 колонки для столпов
- **Breathing ring timer**: SVG circle с анимированным stroke-dashoffset
- **Breathing companion**: контекстные позы под каждую категорию активности

---

## Отчёты (вкладка "Отчёты")

Единый комплексный отчёт за неделю/месяц:
1. **По категориям** — breakdown с барами + всего XP/заданий
2. **Лучший/тихий день** — дата + XP
3. **Столпы жизни** — карьера/скиллы/тело/отдых
4. **Питание** — 6 карточек средних КБЖУ/день (3 в ряд) + compliance grid (попадание в цели по дням с попапом при клике)
5. **Финансы** — расходы/доходы/% бюджета + категории
6. **Медиа** — количество, средний рейтинг, лучшее за период
7. **Путешествия** — поездки за период
8. **Достижения** — разблокированные за период
9. **AI-резюме** — Gemini анализирует данные и даёт рекомендации

---

## Важные детали реализации

- AI возвращает JSON, обёрнутый в markdown — есть парсер для извлечения
- Квесты автообновляются при смене даты (WEEK_FALLBACK по дню недели)
- Клик по квесту = завершить (открывает модал с временем); повторный клик = отменить
- Три отдельных AI-чата: квесты, еда, деньги
- `callAI()` — единая функция, сначала пробует Supabase прокси, fallback на прямой Gemini
- Отчёт питания: compliance grid использует `window._foodCG` для попапов
- Сравнение с предыдущим периодом через `delta()` helper (↑↓ стрелки)
