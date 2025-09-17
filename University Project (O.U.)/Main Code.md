
💻 **Код программы "Полетный планшет" (Палетка) для Ту-154**

```cpp
// FlightTabletTu154.cpp
// Основной файл программы с графическим интерфейсом, реализующий расчет летных параметров для Ту-154.
// Среда разработки: Qt Creator, компилятор MinGW или MSVC
// Сборка: qmake && make или через IDE Qt Creator

#include <QtWidgets> // Подключение всех модулей Qt Widgets для GUI

// --- Класс для хранения и расчета данных полета ---
class FlightCalculator : public QObject {
    Q_OBJECT // Макрос Qt для поддержки сигналов и слотов

public:
    explicit FlightCalculator(QObject *parent = nullptr) : QObject(parent) {}

    // Структура для хранения входных параметров
    struct InputParameters {
        double takeoffWeight;   // Взлетная масса, кг
        double payload;         // Полезная нагрузка, кг
        double fuelWeight;      // Масса топлива, кг
        double flightDistance;  // Дальность полета, км
        double cruiseSpeed;     // Крейсерская скорость, км/ч
        int passengerCount;     // Количество пассажиров
    };

    // Структура для хранения расчетных выходных параметров
    struct OutputParameters {
        double landingWeight;    // Посадочная масса, кг
        double maxLoad;          // Максимальная нагрузка, кг
        double recommendedSpeed; // Рекомендуемая скорость, км/ч
        double fuelConsumption;  // Расход топлива, кг/ч
        QString warnings;        // Предупреждения и рекомендации
    };

    // Основная функция расчета параметров
    OutputParameters calculate(const InputParameters &input) {
        OutputParameters output;

        // 1. Расчет посадочной массы: взлетная масса - расход топлива на весь полет
        // Упрощенная модель: предполагаем средний расход 
        double totalFuelConsumed = input.flightDistance * 0.065; // Пример: 65 кг/км
        output.landingWeight = input.takeoffWeight - totalFuelConsumed;

        // 2. Расчет максимальной нагрузки (упрощенно)
        output.maxLoad = input.payload + input.fuelWeight;

        // 3. Расчет рекомендуемой скорости на основе взлетной массы и дальности
        // Эмпирическая формула, основанная на аэродинамических характеристиках Ту-154 
        if (input.takeoffWeight > 80000) {
            output.recommendedSpeed = 850.0;
        } else if (input.takeoffWeight > 60000) {
            output.recommendedSpeed = 830.0;
        } else {
            output.recommendedSpeed = 810.0;
        }

        // 4. Расчет удельного расхода топлива
        output.fuelConsumption = totalFuelConsumed / (input.flightDistance / input.cruiseSpeed); // кг/ч

        // 5. Генерация предупреждений
        output.warnings = "";
        if (output.landingWeight > 60000) {
            output.warnings += "Внимание: высокая посадочная масса. Убедитесь в достаточности длины ВПП.\n";
        }
        if (input.fuelWeight < input.flightDistance * 0.07) {
            output.warnings += "Предупреждение: топлива может быть недостаточно для полета на заданную дистанцию.\n";
        }

        return output;
    }
};

// --- Класс главного окна приложения ---
class MainWindow : public QMainWindow {
    Q_OBJECT

public:
    MainWindow(QWidget *parent = nullptr) : QMainWindow(parent) {
        setupUI(); // Инициализация интерфейса
        connectSignals(); // Подключение сигналов и слотов
    }

private:
    // Указатели на элементы управления для работы с ними в коде
    QLineEdit *takeoffWeightEdit;
    QLineEdit *payloadEdit;
    QLineEdit *fuelWeightEdit;
    QLineEdit *distanceEdit;
    QLineEdit *speedEdit;
    QLineEdit *passengersEdit;
    QTextEdit *resultDisplay;
    FlightCalculator calculator;

    // Настройка пользовательского интерфейса
    void setupUI() {
        QWidget *centralWidget = new QWidget(this);
        QGridLayout *layout = new QGridLayout(centralWidget);

        // Создание меток и полей ввода
        layout->addWidget(new QLabel("Взлетная масса (кг):"), 0, 0);
        takeoffWeightEdit = new QLineEdit();
        layout->addWidget(takeoffWeightEdit, 0, 1);

        layout->addWidget(new QLabel("Полезная нагрузка (кг):"), 1, 0);
        payloadEdit = new QLineEdit();
        layout->addWidget(payloadEdit, 1, 1);

        layout->addWidget(new QLabel("Масса топлива (кг):"), 2, 0);
        fuelWeightEdit = new QLineEdit();
        layout->addWidget(fuelWeightEdit, 2, 1);

        layout->addWidget(new QLabel("Дальность полета (км):"), 3, 0);
        distanceEdit = new QLineEdit();
        layout->addWidget(distanceEdit, 3, 1);

        layout->addWidget(new QLabel("Крейсерская скорость (км/ч):"), 4, 0);
        speedEdit = new QLineEdit();
        layout->addWidget(speedEdit, 4, 1);

        layout->addWidget(new QLabel("Количество пассажиров:"), 5, 0);
        passengersEdit = new QLineEdit();
        layout->addWidget(passengersEdit, 5, 1);

        // Кнопка расчета
        QPushButton *calculateButton = new QPushButton("Рассчитать");
        layout->addWidget(calculateButton, 6, 0, 1, 2);

        // Текстовое поле для вывода результатов
        resultDisplay = new QTextEdit();
        resultDisplay->setReadOnly(true); // Только для чтения
        layout->addWidget(resultDisplay, 7, 0, 1, 2);

        setCentralWidget(centralWidget);
        setWindowTitle("Полетный планшет Ту-154");
        resize(600, 500); // Начальный размер окна
    }

    // Подключение сигналов к слотам (обработчикам событий)
    void connectSignals() {
        connect(findChild<QPushButton*>(""), &QPushButton::clicked, this, &MainWindow::onCalculateClicked);
    }

private slots:
    // Обработчик нажатия кнопки "Рассчитать"
    void onCalculateClicked() {
        // Получение данных из полей ввода
        FlightCalculator::InputParameters input;
        input.takeoffWeight = takeoffWeightEdit->text().toDouble();
        input.payload = payloadEdit->text().toDouble();
        input.fuelWeight = fuelWeightEdit->text().toDouble();
        input.flightDistance = distanceEdit->text().toDouble();
        input.cruiseSpeed = speedEdit->text().toDouble();
        input.passengerCount = passengersEdit->text().toInt();

        // Вызов функции расчета
        FlightCalculator::OutputParameters output = calculator.calculate(input);

        // Форматирование и вывод результатов
        QString resultText = QString(
            "РЕЗУЛЬТАТЫ РАСЧЕТА:\n"
            "-----------------\n"
            "• Посадочная масса: %1 кг\n"
            "• Максимальная нагрузка: %2 кг\n"
            "• Рекомендуемая скорость: %3 км/ч\n"
            "• Расход топлива: %4 кг/ч\n\n"
            "ПРЕДУПРЕЖДЕНИЯ:\n%5"
        )
        .arg(output.landingWeight, 0, 'f', 0)
        .arg(output.maxLoad, 0, 'f', 0)
        .arg(output.recommendedSpeed, 0, 'f', 0)
        .arg(output.fuelConsumption, 0, 'f', 1)
        .arg(output.warnings);

        resultDisplay->setPlainText(resultText);
    }
};

// Точка входа в приложение
int main(int argc, char *argv[]) {
    QApplication app(argc, argv); // Создание объекта приложения Qt

    MainWindow window; // Создание главного окна
    window.show();     // Отображение окна

    return app.exec(); // Запуск главного цикла обработки событий
}

#include "main.moc" // Подключение метаобъектного кода Qt
```

---

📋 **Пояснения к ключевым блокам кода:**

1.  **Структуры `InputParameters` и `OutputParameters`**:
    *   Эти структуры организованы для инкапсуляции всех входных и выходных данных. Это улучшает читаемость и позволяет легко расширять программу новыми параметрами .

2.  **Функция `calculate` в классе `FlightCalculator`**:
    *   Это ядро программы, где выполняются все расчеты. Реальные формулы для Ту-154 были бы сложнее и основывались бы на Руководстве по летной эксплуатации (РЛЭ). Приведенные формулы — упрощенные примеры, иллюстрирующие логику .

3.  **Метод `setupUI`**:
    *   Здесь создается графический интерфейс. Qt использует компоновки (`QLayout`) для автоматического расположения виджетов, что удобно для кроссплатформенной разработки .

4.  **Обработчик события `onCalculateClicked`**:
    *   Слот, который вызывается при нажатии кнопки. Он отвечает за чтение введенных данных, вызов калькулятора и отображение результатов. Важно, что интерфейс не "зависает" во время расчетов .

5.  **Использование Qt**:
    *   Фреймворк Qt был выбран потому, что он мощный, кроссплатформенный (Windows, Linux, macOS) и широко используется для разработки подобного ПО. Его сигнально-слотная система — элегантное решение для связи объектов в C++ .

---

🚀 **Чтобы собрать и запустить этот код:**

1.  Установите [Qt Creator](https://www.qt.io/product/development-tools) и библиотеку Qt.
2.  Создайте новый проект "Qt Widgets Application".
3.  Замените содержимое файла `main.cpp` на предоставленный код.
4.  Нажмите **Сборка → Запустить** в Qt Creator.
