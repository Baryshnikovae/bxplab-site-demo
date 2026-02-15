# BxPLAB — One-screen hero (Vue 3 + Vite)

Один экран (100vh) без скролла: “огоньки/частицы” на Canvas, мягкий spotlight под курсором, премиальный стеклянный блок и один CTA в Telegram.

## Запуск (2 команды)
```bash
npm i
npm run dev
```

Открой адрес из консоли (обычно http://localhost:5173).

## Сборка и предпросмотр (опционально)
```bash
npm run build
npm run preview
```

## Как заменить логотип
1) Замени файл:
`src/assets/logo.png`

2) Рекомендации:
- PNG с прозрачным фоном (идеально).
- Размер 512x512 или 1024x1024 (чтобы было чётко).
- Пусть у логотипа будут поля внутри файла (не вплотную к краям).

3) Если у тебя SVG:
- положи `src/assets/logo.svg`
- в `src/components/HeroScreen.vue` замени импорт:
  `import logoUrl from '../assets/logo.png'` → `import logoUrl from '../assets/logo.svg'`

## Где менять тексты/контакт
`src/components/HeroScreen.vue`
- `tgLink` — ссылка на Telegram
- текст бейджа/описания/подписи
- текст кнопки

## Где настраивать анимацию частиц
`src/components/ParticleBackground.vue`
- количество частиц (target)
- скорость (vx/vy)
- радиусы (r)
- “мерцание” (tw, baseA)
- “вспышки” (flash) и их вероятность
- реакция на курсор (maxD / r в блоке pointer interaction)

## Структура
- `src/App.vue` — подключает экран
- `src/components/HeroScreen.vue` — контент/анимации UI
- `src/components/ParticleBackground.vue` — Canvas фон (частицы)
- `src/styles/base.css` — базовые стили + фон/сетка
- `src/assets/logo.png` — заглушка логотипа
