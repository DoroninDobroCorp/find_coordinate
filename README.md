# Color Pipette - Точный захват координат на macOS

Инструмент для точного определения координат и цветов пикселей на экране с учетом особенностей macOS (Retina-дисплеи, масштабирование).

## Ключевые возможности

- ✅ **Retina-aware**: корректно работает с Retina-масштабированием
- 🎯 **Точные координаты**: использует native `screencapture` для максимальной точности
- 🔬 **Magnifier**: увеличенный preview области под курсором (16x16 → 120x120)
- 📊 **Усреднение**: сглаживание шума через kxk kernel (1/3/5/7/9 пикселей)
- 🔄 **Переключение методов**: `capture` (native) vs `direct` (pyautogui)
- 💾 **Сохранение**: экспорт координат и RGB в формате `.env`
- 🐛 **Диагностика**: инструменты для сравнения разных методов захвата

## Установка

```bash
# Создать venv и установить зависимости
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

## Использование

### Базовый запуск
```bash
python3 color_pipette.py
```

### С параметрами
```bash
# Высокая частота обновления, усреднение 5x5, follow режим
python3 color_pipette.py --rate 60 --avg 5 --follow

# Режим direct (без screencapture)
python3 color_pipette.py --info-backend direct --save-backend direct
```

## Горячие клавиши

| Клавиша | Действие |
|---------|----------|
| `Space` | Пауза/возобновление |
| `Alt/Option` (hold) | Заморозить позицию |
| `F` | Включить/выключить follow mode (окно следует за курсором) |
| `B` | Переключить info backend (capture ↔ direct) |
| `T` | Переключить save backend (crop → capture → direct → status) |
| `S` | Сохранить скриншот и координаты |
| `C` | Копировать координаты в буфер обмена (.env формат) |
| `E` | Вывести координаты в консоль |
| `V` | Verify - диагностика методов захвата |
| `Q` | Выход |

## Параметры командной строки

```
--rate RATE              Частота обновления в Гц (по умолчанию 30)
--save-dir SAVE_DIR      Директория для сохранения (по умолчанию "debug")
--follow                 Окно следует за курсором
--avg AVG                Размер kernel усреднения: 1/3/5/7/9 (по умолчанию 3)
--info-backend           capture или direct (по умолчанию capture)
--save-backend           crop/capture/direct/status (по умолчанию status)
--info-hz INFO_HZ        Частота обновления статуса в Гц (по умолчанию 10)
--auto-quit AUTO_QUIT    Авто-выход через N секунд (для тестов)
```

## Особенности macOS Retina

### Проблема
На Retina-дисплеях "логические пиксели" отличаются от физических. Например:
- Логическое разрешение: 1440x900
- Физическое разрешение: 2880x1800
- Scale factor: 2.0

При использовании `pyautogui.position()` или `pyautogui.pixel()` координаты могут быть неточными.

### Решение
Этот скрипт использует:
1. **Native screencapture** (`screencapture -R`) - корректно учитывает scaling
2. **Автоматический расчет scale** - анализирует размер захваченного изображения
3. **Усреднение** - снижает погрешность на границах пикселей

## Backends

### Info Backend (отображение в UI)
- `capture` (рекомендуется) - native screencapture, точнее на Retina
- `direct` - pyautogui.pixel(), быстрее но может быть неточным

### Save Backend (что сохраняется при нажатии S)
- `status` (рекомендуется) - использует тот же метод что и info
- `capture` - всегда native screencapture
- `direct` - всегда pyautogui.pixel()
- `crop` - из захваченного crop-изображения (наиболее точно)

## Формат вывода

При сохранении (клавиша `S`) создаются:
1. `pipette_crop_*.png` - увеличенная область с крестом и диагностикой
2. `pipette_full_*.png` - полный экран с отметкой
3. Координаты в формате `.env` (копируются в буфер обмена):

```env
READY_PIXEL_X=1234
READY_PIXEL_Y=567
READY_PIXEL_R=255
READY_PIXEL_G=128
READY_PIXEL_B=64
```

## Диагностика

Клавиша `V` запускает verify режим - сравнивает 3 метода:
- `Direct` - pyautogui.pixel()
- `Img` - из crop с учетом Retina
- `R1x1` - screenshot region 1x1

Результат сохраняется в `pipette_verify_*.png` с подробной информацией.

## Примеры использования

### Для UI тестирования
```python
# После захвата координат клавишей C:
import pyautogui
pyautogui.click(x=READY_PIXEL_X, y=READY_PIXEL_Y)
```

### Для проверки цвета пикселя
```python
import pyautogui
rgb = pyautogui.pixel(READY_PIXEL_X, READY_PIXEL_Y)
assert rgb == (READY_PIXEL_R, READY_PIXEL_G, READY_PIXEL_B)
```

## Troubleshooting

**Координаты неточные?**
- Используйте `--info-backend capture` (по умолчанию)
- Увеличьте усреднение: `--avg 5`
- Проверьте через `V` (verify)

**UI тормозит?**
- Уменьшите частоту: `--rate 15 --info-hz 5`
- Отключите follow: не используйте `--follow`

**Неправильные цвета?**
- Переключите backend клавишей `B`
- Используйте `--save-backend crop` для максимальной точности

## Лицензия

Free to use
