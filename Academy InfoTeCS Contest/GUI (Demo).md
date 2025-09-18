## План разработки GUI (для показа ДЕМО) 

1.  **Создание проекта:** Создадим новый проект в Visual Studio типа **MFC Application**.
2.  **Дизайн интерфейса:** С помощью встроенного редактора ресурсов создадим диалоговое окно с элементами управления:
    *   `Edit Control` для ввода пути к файлу, для которого проверяется подпись.
    *   `Edit Control` для ввода пути к файлу с открытым ключом.
    *   `Button` для выбора файлов через стандартный диалог "Open File".
    *   `Button` "Проверить" для запуска твоего алгоритма верификации.
    *   `Static Text` или `Edit Control` с свойством `Read-only` для вывода результата (например, "Подпись верна" или "Ошибка: подпись недействительна").
3.  **Код:** Напишем код, который свяжет нажатие кнопки с чтением файлов, вызовом твоих функций и отображением результата.

### Реализация на MFC (без Win API)

Главная функция верификации называется `VerifySignature` и имеет следующий прототип:

```cpp
// Где-то в  коде (see main code.cpp, в SignatureVerifier.h/cpp)
bool VerifySignature(const CString& filePath, const CString& publicKeyPath, CString& errorMessage);
```
*   `filePath` - путь к подписанному файлу.
*   `publicKeyPath` - путь к файлу с открытым ключом.
*   `errorMessage` - сюда будет записано описание ошибки, если верификация не прошла.
*   Возвращает `true` если подпись верна, `false` если нет.

И код в классе главного диалогового окна (например, `CVerificationDlg`):

**1. Привязка элементов управления к переменным:**

В заголовке класса диалога (`VerificationDlg.h`) объявляем переменные для обмена данными с элементами управления.

```cpp
class CVerificationDlg : public CDialogEx
{
    // ... остальной код, сгенерированный мастером
public:
    CString m_filePath;      // Для пути к файлу
    CString m_publicKeyPath; // Для пути к открытому ключу
    CString m_resultMessage; // Для строки с результатом

protected:
    CEdit m_editResult;      // Для непосредственного доступа к элементу вывода
};
```

**2. Обработчики событий для кнопок:**

В файле реализации диалога (`VerificationDlg.cpp`).

*   Обработчик кнопки "Обзор..." для выбора файла:

```cpp
void CVerificationDlg::OnBnClickedButtonBrowseFile()
{
    // CFileDialog - стандартный диалог выбора файла
    CFileDialog fileDlg(TRUE); // TRUE = открыть файл, FALSE = сохранить файл
    if (fileDlg.DoModal() == IDOK)
    {
        m_filePath = fileDlg.GetPathName();
        UpdateData(FALSE); // Обновляем элементы управления из переменных
    }
}
```

*   Обработчик кнопки "Обзор..." для выбора открытого ключа (аналогичный).

*   Обработчик кнопки "Проверить":

```cpp
void CVerificationDlg::OnBnClickedButtonVerify()
{
    // 1. Обновляем переменные из элементов управления
    UpdateData(TRUE);

    // 2. Простая валидация
    if (m_filePath.IsEmpty() || m_publicKeyPath.IsEmpty())
    {
        MessageBox(_T("Пожалуйста, укажите пути к файлу и открытому ключу."), _T("Ошибка"), MB_ICONERROR);
        return;
    }

    // 3. Вызов [вызов метода\функции] функции верификации!
    CString errorDetail;
    bool isSignatureValid = VerifySignature(m_filePath, m_publicKeyPath, errorDetail);

    // 4. Отображение результата
    if (isSignatureValid)
    {
        m_resultMessage = _T("✅ Подпись ВЕРНА.");
        m_editResult.SetWindowText(m_resultMessage);
    }
    else
    {
        m_resultMessage.Format(_T("❌ Подпись НЕВЕРНА. Ошибка: %s"), errorDetail);
        m_editResult.SetWindowText(m_resultMessage);
    }

    // Перекрасить поле результата для наглядности
    m_editResult.SetBackgroundColor(isSignatureValid ? RGB(200, 255, 200) : RGB(255, 200, 200));
}
```

**3. Инициализация диалога:**

```cpp
BOOL CVerificationDlg::OnInitDialog()
{
    CDialogEx::OnInitDialog();

    // ... код инициализации от мастера ...

    // Сделаем поле результата read-only и настроим шрифт
    m_editResult.SetReadOnly(TRUE);
    // Можно создать жирный шрифт и применить его к m_editResult

    return TRUE; // Статус ОК
}
```
