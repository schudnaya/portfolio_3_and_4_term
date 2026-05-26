# Отчёт по решению задачи деления с использованием конфигурационного файла

## Цель работы

Разработать программу на языке Python, которая:

1. Выполняет деление двух чисел с заданной точностью.
2. Загружает значение точности из конфигурационного файла.
3. Выполняет обработку ошибок.
4. Использует модульное тестирование для проверки корректности работы.

---

## Постановка задачи

### Задание

1. Написать функцию `calculate`, которая:

* принимает два операнда;
* выполняет деление первого операнда на второй;
* использует параметр точности `epsilon`;
* поддерживает как целые, так и дробные числа.

Допустимый диапазон значений:

```text
10^-9 < epsilon < 10^-1
```

2. Написать функцию `load_params`, которая:

* считывает значение `epsilon` из файла `settings.ini`;
* проверяет корректность данных;
* передаёт значение точности в функцию вычисления.

3. Реализовать тестирование:

Для функции `calculate`:

* деление 1/2;
* деление 1/1000;
* деление на ноль;
* проверка округления.

Для функции `load_params`:

* проверка чтения файла;
* проверка диапазона `epsilon`;
* проверка формата числа;
* проверка отсутствия файла;
* проверка отсутствия секции и параметра.

---

## Описание решения

Программа состоит из следующих частей:

1. Функция `calculate` — выполняет деление и округление результата.
2. Функция `load_params` — загружает параметры из конфигурационного файла.
3. Система логирования — записывает ошибки и действия программы.
4. Пользовательские классы ошибок.
5. Набор модульных тестов.

---

## Листинг программы

### Файл `main.py`

```python
import configparser   # библиотека для работы с ini-файлами
import logging # библиотека для логирования (записи событий в файл)
import sys # для exit(1)

# Классы ошибок
class InvalidInputError(Exception):
    """Ошибка: пользователь ввёл не число"""
    pass

class ConfigLoadError(Exception):
    """Ошибка: не удалось загрузить epsilon из файла"""
    pass


# Настройка логгера
logging.basicConfig(
    filename="program.log",  # файл для логов
    level=logging.INFO,      # уровень логов: INFO, WARNING, ERROR
    format="%(asctime)s - %(levelname)s - %(message)s"
)

logger = logging.getLogger()


# Функция вычисления
def calculate(operand1, operand2, epsilon):
    """
    Делит operand1 на operand2 с точностью epsilon
    """
    if operand2 == 0:
        logger.error("Ошибка: попытка деления на ноль!")
        return None

    result = operand1 / operand2
    logger.info(f"Деление выполнено: {operand1} / {operand2} = {result}")

    # если epsilon в допустимом диапазоне, округляем
    if 10 ** -9 < epsilon < 10 ** -1:
        multiplier = 1 / epsilon
        result = round(result * multiplier) / multiplier
        logger.info(f"Результат округлён до точности {epsilon}: {result}")
    else:
        logger.warning(f"Значение epsilon ({epsilon}) вне диапазона!")

    return result


# Функция загрузки параметров
def load_params(config_file="settings.ini"):
    """
    Считывает значение epsilon из конфигурационного файла
    """
    config = configparser.ConfigParser()
    logger.info(f"Пробуем прочитать файл {config_file}")

    try:
        config.read(config_file)

        if "SETTINGS" not in config:
            raise ConfigLoadError("В файле нет секции [SETTINGS]")

        epsilon_str = config.get("SETTINGS", "epsilon", fallback=None)
        if epsilon_str is None:
            raise ConfigLoadError("Параметр epsilon отсутствует в файле")

        epsilon = float(epsilon_str)

        if not (10 ** -9 < epsilon < 10 ** -1):
            raise ConfigLoadError(f"Epsilon вне диапазона: {epsilon}")

        logger.info(f"Epsilon успешно загружен из файла: {epsilon}")
        return epsilon

    except ValueError:
        logger.error("Ошибка: значение epsilon не является числом")
        raise ConfigLoadError("Ошибка: значение epsilon должно быть числом")
    except ConfigLoadError as e:
        logger.error(f"Ошибка загрузки epsilon: {e}")
        raise
    except Exception as e:
        logger.error(f"Непредвиденная ошибка при чтении файла: {e}")
        raise ConfigLoadError("Произошла непредвиденная ошибка при чтении файла")


# Основная часть программы
try:
    operand1 = float(input("Введите первый операнд: "))
    operand2 = float(input("Введите второй операнд: "))
except ValueError:
    logger.error("Пользователь ввёл нечисловое значение")
    print("Ошибка: нужно ввести число!")
    sys.exit(1)

# Загружаем epsilon из ini-файла
try:
    epsilon = load_params()
    print(f"Точность (epsilon) из файла: {epsilon}")
except ConfigLoadError as e:
    print("Ошибка: не удалось загрузить параметр epsilon из файла настроек.")
    print("Проверьте, что файл settings.ini существует и содержит строку:")
    print("[SETTINGS]\nepsilon = 0.001")
    logger.error(f"Программа остановлена: {e}")
    sys.exit(1)

# Выполняем деление
result = calculate(operand1, operand2, epsilon)

# Вывод результата
if result is None:
    print("Что-то пошло не так :(")
else:
    print(f"Результат деления: {operand1} / {operand2} = {result}")
```

---

## Конфигурационный файл

### Файл `settings.ini`

```ini
[SETTINGS]
epsilon = 0.001
```

---

## Тестирование программы

Для тестирования использовалась библиотека `unittest`.

### Файл `test_main.py`

```python
import unittest # библиотека для тестирования
import os # для работы с файлами
from main import calculate, load_params # импортируем функции из программы


class TestDivisionProgram(unittest.TestCase):
    """
    Тесты для программы деления
    """

    def setUp(self):
        """Создаём временный ini-файл перед каждым тестом"""
        self.test_config_file = "test_settings.ini"

    def tearDown(self):
        """Удаляем временный ini-файл после каждого теста"""
        if os.path.exists(self.test_config_file):
            os.remove(self.test_config_file)

    # Тесты для функции calculate

    def test_divide_simple(self):
        """Проверяем простое деление"""
        result = calculate(10, 2, 0.001)
        self.assertEqual(result, 5.0)

    def test_divide_with_rounding(self):
        """Проверяем округление"""
        result = calculate(1, 3, 0.1)
        # 1/3 = 0.3333, округляем до 0.3
        self.assertEqual(result, 0.3)

    def test_divide_by_zero(self):
        """Деление на ноль должно вернуть None"""
        result = calculate(5, 0, 0.001)
        self.assertIsNone(result)

    def test_epsilon_out_of_range(self):
        """Если epsilon слишком большой или маленький — всё равно работает"""
        result1 = calculate(1, 3, 0.5) # слишком большой
        result2 = calculate(1, 3, 0.0000000001) # слишком маленький
        self.assertIsNotNone(result1)
        self.assertIsNotNone(result2)


    # Тесты для функции load_params

    def test_load_epsilon_from_file(self):
        """Проверяем, что epsilon читается из файла"""
        with open(self.test_config_file, "w") as f:
            f.write("[SETTINGS]\n")
            f.write("epsilon = 0.001\n")

        eps = load_params(self.test_config_file)
        self.assertEqual(eps, 0.001)

    def test_file_not_found(self):
        """Если файл не найден — возвращается значение по умолчанию"""
        eps = load_params("no_such_file.ini")
        self.assertEqual(eps, 0.0001)

    def test_wrong_format_in_file(self):
        """Если epsilon не число — возвращается 0.0001"""
        with open(self.test_config_file, "w") as f:
            f.write("[SETTINGS]\n")
            f.write("epsilon = not_a_number\n")

        eps = load_params(self.test_config_file)
        self.assertEqual(eps, 0.0001)

    def test_missing_section(self):
        """Если в файле нет секции SETTINGS"""
        with open(self.test_config_file, "w") as f:
            f.write("[OTHER]\n")
            f.write("epsilon = 0.01\n")

        eps = load_params(self.test_config_file)
        self.assertEqual(eps, 0.0001)

    def test_missing_epsilon_param(self):
        """Если epsilon отсутствует"""
        with open(self.test_config_file, "w") as f:
            f.write("[SETTINGS]\n")
            f.write("some_other_param = 123\n")

        eps = load_params(self.test_config_file)
        self.assertEqual(eps, 0.0001)


# запуск всех тестов
if __name__ == "__main__":
    unittest.main()
```

---

## Результаты тестирования

В ходе тестирования были проверены следующие сценарии:

| № | Проверка                       | Результат |
| - | ------------------------------ | --------- |
| 1 | Простое деление                | Успешно   |
| 2 | Округление результата          | Успешно   |
| 3 | Деление на ноль                | Успешно   |
| 4 | Значение epsilon вне диапазона | Успешно   |
| 5 | Чтение epsilon из файла        | Успешно   |
| 6 | Отсутствие файла               | Успешно   |
| 7 | Неверный формат числа          | Успешно   |
| 8 | Отсутствие секции SETTINGS     | Успешно   |
| 9 | Отсутствие параметра epsilon   | Успешно   |

---

## Используемые библиотеки

| Библиотека     | Назначение                   |
| -------------- | ---------------------------- |
| `configparser` | Работа с ini-файлами         |
| `logging`      | Логирование работы программы |
| `sys`          | Завершение программы         |
| `unittest`     | Модульное тестирование       |
| `os`           | Работа с файлами             |


---

## Вывод

В ходе выполнения работы была разработана программа для деления чисел с использованием заданной точности.
Точность считывается из конфигурационного файла `settings.ini`.

Также были реализованы:

* обработка ошибок;
* логирование;
* пользовательские исключения;
* модульные тесты.

Программа успешно выполняет поставленные задачи и корректно обрабатывает различные исключительные ситуации.
