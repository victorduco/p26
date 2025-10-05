# E2E Performance Testing

Автоматизированное тестирование производительности для проекта.

## 🚀 Быстрый старт

### 1. Запуск dev сервера
```bash
npm run dev
# Сервер запустится на http://localhost:5173
```

### 2. Запуск performance теста
```bash
# С открытым браузером (для отладки)
npm run test:perf -- --comment="Описание теста"

# Headless режим (для CI/CD)
npm run test:perf:headless -- --comment="Описание теста"

# С CPU throttling (эмуляция слабого устройства)
npm run test:perf -- --cpu=4 --comment="CPU 4x slowdown"
npm run test:perf -- --cpu=6 --comment="CPU 6x slowdown"
```

### 3. Сравнение результатов
```bash
# Сравнить два последних теста
npm run test:compare

# Вывести в JSON формате
npm run test:compare:json
```

## 📁 Структура

```
tests/e2e/
├── scenarios/          # Тестовые сценарии
│   └── resize-performance.js
├── helpers/           # Вспомогательные функции
│   ├── performance.js  # FPS, memory tracking
│   ├── resize.js       # Resize утилиты
│   ├── reporter.js     # Сохранение результатов
│   ├── analyzer.js     # Анализ метрик
│   └── comparator.js   # Сравнение результатов
├── fixtures/          # Тестовые данные
├── results/           # Результаты (JSON + logs)
├── run.js            # Главный раннер
├── compare.js        # Утилита сравнения
└── README.md         # Эта документация
```

## 📊 Доступные метрики

### Resize метрики
- **Среднее время** - среднее время одного ресайза
- **Медиана** - медианное значение
- **Min/Max** - минимальное и максимальное время
- **Стандартное отклонение** - разброс значений
- **Общее время** - суммарное время всех ресайзов

### FPS метрики
- **Средний FPS** - frames per second
- **Min/Max FPS** - диапазон FPS
- **Состояние:**
  - Excellent: 60+ FPS
  - Good: 30-60 FPS
  - Poor: 15-30 FPS
  - Critical: <15 FPS

### Деградация производительности
- **Первая половина** - avg время первых 50% тестов
- **Вторая половина** - avg время последних 50% тестов
- **Изменение %** - процент деградации
- **Деградация** - есть ли деградация (>20% = ДА)

### Память
- **Использовано** - MB используемой памяти
- **Процент** - % от общей памяти
- **Доступно** - свободная память

## ⚙️ CPU Throttling

CPU throttling позволяет эмулировать работу на менее производительных устройствах:

```bash
# Без throttling (по умолчанию)
npm run test:perf -- --comment="Baseline"

# 4x замедление - имитация среднего ноутбука
npm run test:perf -- --cpu=4 --comment="Mid-range laptop"

# 6x замедление - имитация слабого устройства/мобильного
npm run test:perf -- --cpu=6 --comment="Low-end device"
```

**Рекомендуемые значения:**
- `--cpu=1` - без throttling (мощный десктоп)
- `--cpu=4` - средний ноутбук, старый десктоп
- `--cpu=6` - слабый ноутбук, планшет
- `--cpu=8` - очень слабое устройство, старый смартфон

## 🔄 Типичные сценарии использования

### Сценарий 1: Baseline тест
Создание базового теста перед изменениями:

```bash
# 1. Убедись что dev сервер запущен
npm run dev

# 2. Запусти baseline тест
npm run test:perf -- --comment="Baseline: до оптимизации"

# Результат сохранится в tests/e2e/results/
```

### Сценарий 2: Тест после изменений
После внесения изменений в код:

```bash
# 1. Внеси изменения в код
# Например, закомментируй компонент в App.vue

# 2. Запусти тест
npm run test:perf -- --comment="После удаления Keypad"

# 3. Сравни с предыдущим тестом
npm run test:compare
```

### Сценарий 3: Поиск проблемы
Если обнаружена деградация:

```bash
# 1. Запусти baseline
npm run test:perf -- --comment="Baseline"

# 2. Постепенно отключай компоненты/функции
# Например: закомментируй glass-effect

# 3. Тестируй после каждого изменения
npm run test:perf -- --comment="Без glass-effect"
npm run test:compare

# 4. Найди компонент с проблемой
# Результат покажет где деградация исчезла
```

### Сценарий 4: Тестирование на слабых устройствах
Проверка производительности на разных типах устройств:

```bash
# 1. Тест на мощном устройстве (baseline)
npm run test:perf -- --comment="Desktop - no throttling"

# 2. Тест на среднем ноутбуке
npm run test:perf -- --cpu=4 --comment="Mid laptop - 4x slowdown"

# 3. Сравнение результатов
npm run test:compare
```

### Сценарий 5: CI/CD интеграция
Автоматическая проверка перед деплоем:

```bash
# В вашем pipeline:
npm run dev &          # Запустить dev
sleep 5               # Подождать старт
npm run test:perf:headless -- --cpu=4 --comment="CI: Pre-deploy check"

# Проверить exit code
if [ $? -ne 0 ]; then
  echo "Performance test failed!"
  exit 1
fi
```

## 📈 Интерпретация результатов

### ✅ Хорошие показатели
- Среднее время ресайза: **<100ms**
- FPS: **>30**
- Деградация: **<20%**
- Память: **<80%**

### ⚠️ Нужно внимание
- Среднее время: **100-500ms**
- FPS: **15-30**
- Деградация: **20-100%**
- Память: **80-90%**

### 🚨 Критично
- Среднее время: **>500ms**
- FPS: **<15**
- Деградация: **>100%**
- Память: **>90%**

## 🔍 Как читать сравнение

При запуске `npm run test:compare` вы увидите:

```
📊 СРАВНЕНИЕ: Test A vs Test B

⏱️  ОБЩЕЕ ВРЕМЯ:
  Было:    12.38s
  Стало:   1.92s
  Разница: ✅ -10.46s (84.5% быстрее)

📏 СРЕДНЕЕ ВРЕМЯ РЕСАЙЗА:
  Было:    552.75ms
  Стало:   39.75ms
  Разница: ✅ -513ms (92.8% быстрее)

📉 ДЕГРАДАЦИЯ:
  Было:    +339% (205ms → 900ms) ⚠️
  Стало:   +5.96% (38ms → 40ms) ✅

💡 ВЫВОД: Производительность значительно улучшилась!
```

### Легенда:
- ✅ **Зеленый** - улучшение
- ⚠️ **Желтый** - небольшая регрессия (<20%)
- 🚨 **Красный** - критическая регрессия (>20%)

## 🛠️ Разработка новых тестов

### Создание нового сценария

1. Создай файл `scenarios/your-test.js`:

```javascript
import { setupPerformanceTracking, getPerformanceMetrics } from '../helpers/performance.js';
import { saveResults, printSummary } from '../helpers/reporter.js';
import { analyzeResults, printDetailedReport } from '../helpers/analyzer.js';

export async function testYourScenario({ comment = '' } = {}) {
  const testStartTime = Date.now();
  // ... ваша логика теста

  const results = {
    totalTime: { ms: Date.now() - testStartTime },
    // ... ваши метрики
  };

  const analysis = analyzeResults(results);
  printDetailedReport(analysis, results);
  saveResults('your-test', results, comment);

  return results;
}
```

2. Добавь в `run.js`:

```javascript
import { testYourScenario } from './scenarios/your-test.js';

// В switch:
case 'your-test':
  await testYourScenario({ comment });
  break;
```

3. Добавь в `package.json`:

```json
"test:your-scenario": "node tests/e2e/run.js your-test"
```

## 🐛 Troubleshooting

### Тест не запускается
```bash
# Проверь что dev сервер запущен
curl http://localhost:5173

# Если нет - запусти:
npm run dev
```

### Браузер не открывается
```bash
# Установи браузеры Playwright:
npx playwright install chromium
```

### FPS = 0 или N/A
Это нормально для очень быстрых тестов (<1s). FPS начинает считаться после ~1 секунды.

### Деградация есть всегда
Проверь что между тестами не накапливаются изменения:
1. Откати все изменения
2. Запусти 2 теста подряд без изменений
3. Они должны быть похожи

## 📚 Полезные команды

```bash
# Посмотреть последние результаты
ls -lt tests/e2e/results/*.json | head -5

# Прочитать последний результат
cat tests/e2e/results/resize-performance-*.json | tail -1 | jq

# Очистить старые результаты
rm tests/e2e/results/*.json
rm tests/e2e/results/*.txt

# Запустить с другими параметрами
node tests/e2e/run.js resize-performance --headless --comment="Custom test"

# Запустить с CPU throttling
node tests/e2e/run.js resize-performance --cpu=6 --comment="Slow device"
```

## 🎯 Best Practices

1. **Всегда добавляй комментарий** к тестам (`--comment`)
2. **Запускай baseline** перед изменениями
3. **Сравнивай сразу** после изменений
4. **Чисти results/** периодически
5. **Используй headless** для автоматизации
6. **Проверяй деградацию** - главный индикатор проблем

## 🔗 Ссылки

- [Playwright Docs](https://playwright.dev)
- [Performance API](https://developer.mozilla.org/en-US/docs/Web/API/Performance)
- [Chrome DevTools Protocol](https://chromedevtools.github.io/devtools-protocol/)
