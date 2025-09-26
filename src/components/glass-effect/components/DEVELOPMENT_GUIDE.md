# Glass Effect - Миграция на новую архитектуру

## Задача

Перенести функциональность из старого монолитного `GlassEffect.vue` в новую модульную архитектуру с отдельными компонентами для каждого слоя.

## Расположение файлов

- **Старый проект:** `/src/components/glass-effect/GlassEffect.vue` + вспомогательные файлы
- **Новый проект:** `/src/components/glass-effect/components/` - временная папка с новыми компонентами, в будущем будет перенесено вместо старого проекта

## Ожидаемый результат

После завершения всех этапов разработки новый `GlassEffect.vue` должен работать идентично старому, но с улучшенной архитектурой и лучшей поддерживаемостью кода.

## Что нужно доделать

### Этап 1. Сделать энд ту энд с одним слоем - ВЫПОЛНЕНО

- **GlassEffectDefaults.js** - перенести и адаптировать логику из `effect-options.js`
- Имплементировать GeLight - на основе стаорого компонента и описания туду в файле
- Проверить работу основного файла
- Добавить компонент в переименовать пропсы 1. **glassConfig → userOptions** (переименование пропа)
- Убедиться что новый компонент с одним слоем GeLight работает идентично старому по стилям

### Этап 2. Добавление 3 других слоев - ВЫПОЛНЕНО

**Базируясь на опыте Этапа 1, учесть следующее:**

#### 2.1. Паттерн именования переменных - ВЫПОЛНЕНО

- Использовать префикс компонента: `highlight*`, `noise*`, `outline*` (НЕ legacy названия)
- В `GlassEffectDefaults.js` добавить параметры с правильными префиксами
- В деструктуризации `GlassEffect.vue` использовать shorthand синтаксис
- В самих компонентах маппить на legacy API для `layer-*.js` функций

#### 2.2. Структура для каждого компонента - ВЫПОЛНЕНО

**GeHighlight.vue:**

```javascript
// В GlassEffectDefaults.js добавить:
highlightReflection: 0.45, // surfaceReflection в новой архитектуре

// В GlassEffect.vue:
const { highlightReflection } = opts;
<GeHighlight :options="{ highlightReflection }" :intensity />
```

**GeNoise.vue:**

```javascript
// В GlassEffectDefaults.js добавить:
noiseStrength: 0.22,
noiseRefractionDepth: 2.0, // refractionDepth в новой архитектуре

// В GlassEffect.vue:
const { noiseStrength, noiseRefractionDepth } = opts;
<GeNoise :options="{ noiseStrength, noiseRefractionDepth }" :intensity />
```

**GeOutline.vue:**

```javascript
// В GlassEffectDefaults.js добавить:
outlineIntensity: 0.4, // shadowDepth в новой архитектуре
outlineGlassTintHue: 210, // glassTintHue в новой архитектуре

// В GlassEffect.vue:
const { outlineIntensity, outlineGlassTintHue, surfaceReflection, shadowDepth } = opts;
<GeOutline :options="{ outlineIntensity, outlineGlassTintHue, surfaceReflection, shadowDepth }" :intensity />
```

#### 2.3. Обязательные файлы для реализации - ВЫПОЛНЕНО

- Изучить `layer-highlight.js` для GeHighlight
- Изучить `layer-noise.js` для GeNoise
- Изучить `layer-outline.js` + логику `liquid-glass__card` из старого GlassEffect.vue для GeOutline

#### 2.4. CSS классы - ИЗОЛИРОВАННЫЕ В КОМПОНЕНТАХ - ВЫПОЛНЕНО

- Каждый компонент содержит свои стили в `<style scoped>`
- `.glass-highlight` в GeHighlight.vue (аналогично `.glass-light` в GeLight.vue)
- `.glass-noise` в GeNoise.vue
- `.glass-outline` в GeOutline.vue
- НЕ добавлять в общий `/index.css` - каждый компонент самодостаточен
- Базовые стили: `position: absolute, inset: 0, pointer-events: none, border-radius: inherit`

#### 2.5. Тестирование поэтапно - ВЫПОЛНЕНО

- После каждого компонента раскомментировать в `GlassEffect.vue`
- Обновить `IntroRectangle.vue` glassConfig добавив параметры для тестируемого слоя
- Проверить визуальную идентичность со старым компонентом

### Этап 3. Сделать реализацию GeFilter

ФОН УЖЕ ГОТОВ! Директива v-mask-element создает ::before псевдоэлемент с фоном. Нужно только
применить к нему SVG фильтр!

---

📝 УПРОЩЕННАЯ СТРАТЕГИЯ (8 ШАГОВ)

ШАГ 1: Создать упрощенный GeFilter только с SVG фильтром

ЧТО ДЕЛАТЬ:

- Удалить всю логику DOM клонирования
- Оставить только SVG фильтр из SvgFilter.vue
- GeFilter станет просто контейнером для <svg><filter>

ЧТО НАПИСАТЬ:
<template>
<div class="glass-filter">
<!-- Только SVG фильтр, БЕЗ DOM клонирования -->
<svg v-if="filterProps.filterReady" class="glass-filter__svg" aria-hidden="true">
<defs>
<filter :id="filterProps.filterId">
<!-- Весь SVG фильтр из SvgFilter.vue -->
</filter>
</defs>
</svg>
</div>
</template>

ШАГ 2: Применить SVG фильтр к ::before псевдоэлементу через CSS

ЧТО ДЕЛАТЬ:

- GeFilter создает CSS переменную --glass-filter: url(#filterId)
- Директива v-mask-element применяет эту переменную к ::before

ЧТО НАПИСАТЬ В GeFilter:
// GeFilter.vue
const glassFilterCss = computed(() => `url(#${filterProps.filterId})`)

onMounted(() => {
// Найти родительский элемент с v-mask-element и установить CSS переменную
const maskElement = document.querySelector('.mask-element')
if (maskElement) {
maskElement.style.setProperty('--glass-filter', glassFilterCss.value)
}
})

ШАГ 3: Интегрировать полный SVG фильтр из SvgFilter.vue

ЧТО ПЕРЕНЕСТИ:

- Весь <filter> блок из SvgFilter.vue (строки 4-133)
- Заменить props: filterId → filterProps.filterId и т.д.

ШАГ 4: Создать CSS переменную --glass-filter для применения фильтра

ЧТО ДОБАВИТЬ В GeFilter:
watch(
() => filterProps.filterId,
(newFilterId) => {
if (newFilterId) {
const maskElement = document.querySelector('.mask-element')
if (maskElement) {
maskElement.style.setProperty('--glass-filter', `url(#${newFilterId})`)
}
}
}
)

ШАГ 5: Раскомментировать в новом GlassEffect.vue

ЧТО ИЗМЕНИТЬ:

  <!-- БЫЛО: -->
  <!-- <GeFilter ... /> -->

  <!-- СТАНЕТ: -->
  <GeFilter :filterProps="filterProps" />

ШАГ 6: Передать filterProps через createFilterProps

ЧТО ДОБАВИТЬ в новый GlassEffect.vue:
// Импортировать функции
import { createFilterProps } from './path-to-filter-props'

// Создать filterProps
const filterProps = createFilterProps(opts, props.intensity, filterState)

ШАГ 7: Обновить директиву maskElement для поддержки --glass-filter

ЧТО ДОБАВИТЬ в maskElement.js:126:
.mask-element::before {
/_ ...существующие стили _/
filter: var(--glass-filter, none); /_ ← ПРИМЕНЯЕМ ФИЛЬТР _/
}

ШАГ 8: Протестировать интеграцию всех 5 компонентов

ЧТО ПРОВЕРИТЬ:

- ::before псевдоэлемент получает SVG фильтр
- GeHighlight, GeNoise, GeLight, GeOutline работают поверх
- Фильтр корректно применяется к фону

---

🔄 ИТОГОВАЯ АРХИТЕКТУРА

ПОТОК ДАННЫХ:
v-mask-element создает ::before с фоном
↓
GeFilter создает SVG фильтр + CSS переменную --glass-filter
↓
::before получает filter: var(--glass-filter) и применяет к фону
↓
GeHighlight/GeNoise/GeLight/GeOutline рендерятся поверх

ПРЕИМУЩЕСТВА:

- ✅ Нет DOM клонирования - используем готовый псевдоэлемент
- ✅ Нет RAF обновлений - директива сама синхронизирует
- ✅ Простая архитектура - GeFilter только для SVG фильтра
- ✅ CSS переменные - чистый способ передачи фильтра
