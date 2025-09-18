## Генерация RTF-файла (Рекомендуемый)

Формат RTF (Rich Text Format) — старый, но прекрасно поддерживаемый всеми версиями Word текстовый формат.

**План действий:**

1.  **Создадим новый класс или набор функций** для экспорта данных.
2.  **Будем записывать данные** в текстовый файл с расширением `.rtf`, следуя синтаксису RTF.
3.  **Добавим в существующую структуру `AircraftParameters`** метод для экспорта.

**Код модуля сохранения\вывода на печать:**

Создадим новый заголовочный файл, например, `Exporter.h`:

```cpp
#ifndef EXPORTER_H
#define EXPORTER_H

#include <string>
#include <fstream>
#include "AircraftParameters.h" // Подключаем ваш основной класс

class Exporter {
public:
    // Конструктор, принимающий ссылку на параметры
    Exporter(const AircraftParameters& params);

    // Основной метод для сохранения в файл
    bool saveToWord(const std::string& filename);

private:
    const AircraftParameters& m_params; // Константная ссылка на параметры
};

#endif
```

Реализация в `Exporter.cpp`:

```cpp
#include "Exporter.h"
#include <iostream>

Exporter::Exporter(const AircraftParameters& params) : m_params(params) {}

bool Exporter::saveToWord(const std::string& filename) {
    // Открываем файл для записи
    std::ofstream outFile(filename + ".rtf");
    if (!outFile.is_open()) {
        std::cerr << "Ошибка: Не удалось создать файл " << filename << ".rtf" << std::endl;
        return false;
    }

    // Записываем заголовок RTF-файла
    outFile << "{\\rtf1\\ansi\\deff0\n";
    outFile << "{\\fonttbl{\\f0 Arial;}}\n"; // Определяем шрифт
    outFile << "\\viewkind4\\uc1\\pard\\fs24\\f0\\par\n"; // Начальные настройки документа

    // Заголовок документа
    outFile << "\\qc\\b Отчёт по параметрам самолёта Ту-154\\b0\\par\\par\n";

    // Начинаем таблицу (самая простая реализация)
    outFile << "\\trowd\\trgaph0\n"; // Начало строки таблицы
    outFile << "\\cellx5000\\cellx10000\n"; // Определяем границы двух ячеек (в twips)
    outFile << "\\pard\\intbl\\b Параметр\\b0\\cell\\b Значение\\b0\\cell\\pard\\intbl\\row\\par\n"; // Заголовки столбцов

    // Заполняем таблицу данными из m_params
    // Пример для нескольких параметров. Вам нужно будет добавить ВСЕ необходимые.
    outFile << "\\trowd\\trgaph0\n";
    outFile << "\\cellx5000\\cellx10000\n";
    outFile << "Скорость полета (V)\\cell " << m_params.flightSpeed << " км/ч\\cell\\row\\par\n";

    outFile << "\\trowd\\trgaph0\n";
    outFile << "\\cellx5000\\cellx10000\n";
    outFile << "Угол атаки (α)\\cell " << m_params.angleOfAttack << " °\\cell\\row\\par\n";

    outFile << "\\trowd\\trgaph0\n";
    outFile << "\\cellx5000\\cellx10000\n";
    outFile << "Тяга двигателей (F)\\cell " << m_params.engineThrust << " кН\\cell\\row\\par\n";

    // Добавьте здесь все остальные параметры точно таким же образом...

    outFile << "\\par\n\\par\n";
    outFile << "Дата расчета: " << __DATE__ << "\\par\n"; // Добавляем дату
    outFile << "}"; // Закрываем RTF-документ

    outFile.close();
    std::cout << "Отчёт успешно сохранён в файл: " << filename << ".rtf" << std::endl;
    return true;
}
```

**Интеграция с main():**
В главной функции, после проведения расчетов, нужно будет вызвать класс.

```cpp
// ... ваш существующий код расчета ...
AircraftParameters params; // Ваш объект с параметрами
// ... проводим все вычисления и заполняем params ...

// Создаем экспортер и сохраняем
Exporter exporter(params);
exporter.saveToWord("Otchet_Po_Poletu"); // Имя файла без расширения
```

---
