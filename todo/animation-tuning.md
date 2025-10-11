# Настройка анимации интро

## Текущая последовательность анимации

1. Menu items (6 элементов) - снизу вверх (Links → AI Play → Story 3 → Story 2 → Story 1 → Intro)
2. H1 Title - появление
3. Subtitle - появление
4. Rectangle 1 - появление
5. Rectangle 2 - появление
6. Rectangle 3 - появление
7. Rectangle 4 - появление
8. Scroll hint ("Scroll to Story One") - появление

## Timing эффект

Используется **ease-in-out** эффект (медленно → быстро → медленно):
- Задержка в начале и конце: **400ms**
- Задержка в середине: **250ms**

## Где менять настройки

### Файл: `/src/pages/main-page/intro/Intro.vue`

#### Изменить скорость анимации

Найти строки (примерно 138-140):

```javascript
// Базовые задержки: медленный старт и конец (400ms), середина (250ms)
const minDelay = 250;  // Задержка в середине последовательности
const maxDelay = 400;  // Задержка в начале и конце последовательности
```

**Примеры:**
- Сделать быстрее: уменьшить значения (например, `minDelay = 150`, `maxDelay = 250`)
- Сделать медленнее: увеличить значения (например, `minDelay = 300`, `maxDelay = 500`)

#### Изменить easing функцию

Найти функцию (примерно строка 123):

```javascript
function easeInOutCubic(t) {
  return t < 0.5 ? 4 * t * t * t : 1 - Math.pow(-2 * t + 2, 3) / 2;
}
```

**Варианты замены:**
- Линейная (без easing): `return t;`
- Ease-in (ускорение): `return t * t * t;`
- Ease-out (замедление): `return 1 - Math.pow(1 - t, 3);`

#### Изменить длительность fade transitions

Найти строки с transition конфигами (примерно 103-114):

```javascript
const titleTransition = {
  ...baseTransition,
  duration: 0.6,  // Длительность появления заголовка
};

const subtitleTransition = {
  ...baseTransition,
  duration: 0.5,  // Длительность появления подзаголовка
};

const scrollHintTransition = {
  ...baseTransition,
  duration: 0.4,  // Длительность появления scroll hint
};
```

### Файл: `/src/pages/main-page/intro/IntroRectangle.vue`

#### Изменить скорость появления прямоугольников

Найти строку (примерно 170):

```css
transition: opacity 0.4s cubic-bezier(0.4, 0, 0.2, 1);
```

Можно изменить `0.4s` на другое значение для более быстрого/медленного появления.

### Файл: `/src/components/page-navigation/PageNavigation.vue`

#### Изменить анимацию меню (снизу вверх)

Найти функцию `startIntroAnimation()` (примерно строка 91-161).

Ключевые параметры:
- `baseDelay` (строка 138) - базовая скорость анимации меню
- Начальная задержка перед стартом (строка 168): `setTimeout(..., 250)`

## Структура файлов

```
/src/pages/main-page/
  ├── MainPage.vue          # Главная страница, управляет состоянием
  ├── intro/
  │   ├── Intro.vue         # 🔧 ОСНОВНОЙ ФАЙЛ - логика анимации интро
  │   ├── IntroRectangle.vue # 🔧 Анимация отдельных прямоугольников
  │   └── variants.js       # Варианты анимации для motion-v

/src/components/page-navigation/
  └── PageNavigation.vue    # 🔧 Анимация навигационного меню
```

## Будущие улучшения

- [ ] Добавить настройки анимации через props/конфиг
- [ ] Сделать адаптивную скорость в зависимости от размера экрана
- [ ] Добавить возможность пропуска анимации
- [ ] Синхронизировать с анимацией меню для единого timing
