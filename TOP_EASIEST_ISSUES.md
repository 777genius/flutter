# 🎯 ТОП САМЫХ ЛЁГКИХ FLUTTER ISSUES

> **Критерии отбора:**
> - ⚡ Быстро делать (10-45 минут)
> - ✅ Легко проверить (без визуальных тестов)
> - 💯 Точно примут (90%+ уверенность, P1/P2)
>
> **Обновлено:** 18 ноября 2025

---

## 🥇 #1 - Issue #83630: image_picker Документация

**⏱️ 10 минут | 💯 100% уверенность | 📝 ТОЛЬКО документация**

### Почему это самое простое?
- ✅ Вообще ноль кода - только README и dartdoc
- ✅ Абсолютно безопасно - ничего не сломается
- ✅ P2 + "good first issue"
- ✅ Проверка: просто прочитать документацию
- ✅ Точно примут - это исправление misleading docs

### Что делать?
Добавить в документацию пояснение, что `maxDuration` работает **только для записи видео**, а не для фильтрации галереи.

**Файлы:**
- `packages/image_picker/image_picker/README.md`
- `packages/image_picker/image_picker/lib/src/image_picker.dart` (dartdoc)

**Добавить текст:**
> The `maxDuration` parameter sets the maximum recording duration for video capture from the camera. It does NOT filter videos when selecting from the gallery. Gallery videos of any duration can be selected regardless of this parameter.

📄 **[Полная инструкция](flutter_issues/issue_83630_image_picker_docs.md)**
🔗 **[Issue #83630](https://github.com/flutter/flutter/issues/83630)**

---

## 🥈 #2 - Issue #119484: Container Error Message

**⏱️ 15-20 минут | 💯 98% уверенность | 📝 Изменить одну строку**

### Почему это очень простое?
- ✅ Изменить только строку assertion - никакой логики
- ✅ P2, triaged-framework
- ✅ 100% безопасно - не влияет на функциональность
- ✅ Проверка: запустить код с обоими параметрами → увидеть новый текст ошибки
- ✅ Нужный всем - частая ошибка у новичков

### Что делать?
Улучшить текст ошибки когда передают и `color`, и `decoration` одновременно.

**Файл:** `packages/flutter/lib/src/widgets/container.dart:280`

**Текущий текст:**
```dart
'Cannot provide both a color and a decoration\n'
'The color argument is just a shorthand for "decoration: BoxDecoration(color: color)".',
```

**Новый текст:**
```dart
'Cannot provide both a color and a decoration to Container.\n'
'Use either Container.color OR Container.decoration (with BoxDecoration.color), not both.\n'
"The 'color' property is a convenience shorthand for 'decoration: BoxDecoration(color: color)'.",
```

**Проверка:**
```dart
// Этот код должен выдать улучшенную ошибку
Container(
  color: Colors.red,
  decoration: BoxDecoration(border: Border.all()),
)
```

📄 **[Полная инструкция](flutter_issues/issue_119484_container_error_message.md)**
🔗 **[Issue #119484](https://github.com/flutter/flutter/issues/119484)**

---

## 🥉 #3 - Issue #59462: SimpleDialog ContentTextStyle

**⏱️ 30-45 минут | 💯 95% уверенность | 🎨 Добавить theme property**

### Почему это простое?
- ✅ titleTextStyle **уже работает** - просто скопировать паттерн для contentTextStyle
- ✅ P2, triaged-design, **4.5 года issue!**
- ✅ Есть готовый пример в AlertDialog
- ✅ Проверка: создать SimpleDialog с DialogTheme → текст применит стиль
- ✅ Чёткое решение от команды Flutter

### Что делать?
Добавить поддержку `contentTextStyle` из `DialogTheme` в `SimpleDialog`.

**Файл:** `packages/flutter/lib/src/material/dialog.dart`

**Добавить:**
```dart
final DialogTheme dialogTheme = DialogTheme.of(context);
final DialogTheme defaults = DialogTheme.defaults(context);

final TextStyle? contentTextStyle =
  dialogTheme.contentTextStyle ?? defaults.contentTextStyle;
```

**Применить:**
```dart
DefaultTextStyle(
  style: contentTextStyle!,
  child: Column(children: children),
)
```

**Проверка:**
```dart
// Текст должен применить стиль из темы
MaterialApp(
  theme: ThemeData(
    dialogTheme: DialogTheme(
      contentTextStyle: TextStyle(fontSize: 14, color: Colors.grey),
    ),
  ),
  home: SimpleDialog(
    title: Text('Title'),
    children: [Text('Content')], // ← Этот текст должен быть grey, 14px
  ),
)
```

📄 **[Полная инструкция](flutter_issues/issue_59462_simpledialog_theme.md)**
🔗 **[Issue #59462](https://github.com/flutter/flutter/issues/59462)**

---

## 🎖️ #4 - Issue #117755: Divider Color M3 Spec

**⏱️ 30-45 минут | 💯 90% уверенность | 🎨 Theme consistency**

### Почему это относительно простое?
- ✅ P2, чёткая спецификация M2/M3
- ✅ Issue содержит точное решение
- ✅ Условная логика на useMaterial3
- ✅ Проверка: сравнить `Theme.dividerColor` с actual `Divider` цветом
- ✅ Важная проблема - влияет на все приложения

### Что делать?
Исправить несоответствие между `Theme.dividerColor` и фактическим цветом виджета `Divider`.

**Файл:** `packages/flutter/lib/src/material/theme_data.dart`

**Добавить в ThemeData конструкторы:**
```dart
dividerColor: dividerColor ?? (useMaterial3
  ? colorScheme.outlineVariant           // M3 spec
  : colorScheme.onSurface.withOpacity(0.12)),  // M2 spec
```

**Проверка:**
```dart
// M3 mode - должны совпадать
final theme = ThemeData(colorSchemeSeed: Colors.blue, useMaterial3: true);
assert(theme.dividerColor == theme.colorScheme.outlineVariant);

// M2 mode - должны совпадать
final theme2 = ThemeData(colorSchemeSeed: Colors.blue, useMaterial3: false);
assert(theme2.dividerColor == theme2.colorScheme.onSurface.withOpacity(0.12));
```

📄 **[Полная инструкция](flutter_issues/issue_117755_divider_color.md)**
🔗 **[Issue #117755](https://github.com/flutter/flutter/issues/117755)**

---

## 📊 Сравнение по критериям

| Issue | Время | Уверенность | Сложность проверки | Риск | Приоритет |
|-------|-------|-------------|-------------------|------|-----------|
| **#83630** docs | 10 мин | 100% | Очень легко | Ноль | P2 |
| **#119484** error | 15-20 мин | 98% | Очень легко | Ноль | P2 |
| **#59462** theme | 30-45 мин | 95% | Легко | Низкий | P2 |
| **#117755** divider | 30-45 мин | 90% | Средне | Низкий | P2 |

---

## 🚀 Рекомендации

### Начни с #83630 если:
- ✅ Хочешь максимально быстрый результат
- ✅ Первый раз контрибьютишь в Flutter
- ✅ Не хочешь трогать код вообще

### Начни с #119484 если:
- ✅ Хочешь легкий код (одна строка)
- ✅ Хочешь помочь всем Flutter разработчикам
- ✅ Нравится улучшать UX/DX

### Начни с #59462 если:
- ✅ Хочешь работать с Material Design
- ✅ Нравится consistency в API
- ✅ Готов написать несколько строк кода + тесты

### Начни с #117755 если:
- ✅ Интересуется Material 3
- ✅ Любишь спецификации
- ✅ Хочешь исправить важную проблему

---

## ✅ Чек-лист перед началом

Для каждого issue:

1. **Проверь что issue всё ещё открыт:**
   ```bash
   # Открой issue URL в браузере
   ```

2. **Убедись что никто не работает:**
   - Нет assignee
   - Нет открытых PR

3. **Прочитай все комменты:**
   - Особенно от команды Flutter
   - Проверь нет ли блокеров

4. **Настрой окружение:**
   ```bash
   git checkout -b fix/issue-XXXXX-description
   git config user.name "iliya"
   git config user.email "iliyazelenkog@gmail.com"
   ```

5. **Прочитай детальную инструкцию:**
   - Открой соответствующий MD файл
   - Изучи все шаги

---

## 📝 Шаблон коммита

```bash
git commit -m "Fix #XXXXX: Short description

<Detailed explanation of what was changed and why>

Fixes #XXXXX"
```

**Примеры:**

```bash
# #83630
git commit -m "Fix #83630: Clarify maxDuration only applies to recording

Updated image_picker documentation to explicitly state that maxDuration
parameter only restricts video recording duration from camera, and does
NOT filter gallery video selections.

Fixes #83630"

# #119484
git commit -m "Fix #119484: Improve Container color/decoration error message

Changed assertion message to clearly communicate using either color OR
decoration parameter, not both. Previous message was confusing about
the proper condition.

Fixes #119484"

# #59462
git commit -m "Fix #59462: Add contentTextStyle support to SimpleDialog

SimpleDialog now respects contentTextStyle from DialogTheme, similar
to how titleTextStyle already works. This provides consistent theming
for all SimpleDialog text.

Fixes #59462"

# #117755
git commit -m "Fix #117755: Fix divider color inconsistencies in M2/M3

Updated ThemeData constructors to use correct divider colors per
Material Design specifications:
- M2: onSurface.withOpacity(0.12)
- M3: outlineVariant

This ensures Theme.dividerColor matches actual Divider widget color.

Fixes #117755"
```

---

## 🎓 После завершения

1. **Запусти все тесты:**
   ```bash
   flutter test packages/flutter/test/...
   ```

2. **Коммит и push:**
   ```bash
   git add .
   git commit -m "..."
   git push -u origin claude/flutter-repo-issues-01HJhnUb93ktYD4fJFFqiYEV
   ```

3. **Создай Pull Request:**
   - Используй шаблон Flutter
   - Ссылка на issue
   - Опиши что изменил
   - Опиши как тестировал

4. **Отмечай в TOP_FLUTTER_ISSUES.md как ✅ IMPLEMENTED**

---

## 🔗 Полезные ссылки

- [Flutter Contributing Guide](https://github.com/flutter/flutter/blob/master/CONTRIBUTING.md)
- [Все детальные инструкции](flutter_issues/)
- [Material 3 Spec](https://m3.material.io/)
- [Good First Issues](https://github.com/flutter/flutter/labels/good%20first%20issue)

---

**Создано:** 18 ноября 2025
**Проверено:** Все issues открыты и доступны
**Общее время:** 1.5 - 3 часа на все 4 issues
**Средняя уверенность:** 95.75%
