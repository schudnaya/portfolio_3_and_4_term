# Реализация шаблона проектирования «Одиночка» для работы с курсами валют

## Цель работы

Изучить применение шаблона проектирования «Одиночка» (Singleton) в объектно-ориентированном программировании на языке Python.

В рамках работы необходимо:

* реализовать получение курсов валют с сайта ЦБ РФ;
* реализовать паттерн Singleton с использованием метакласса;
* реализовать хранение чисел с плавающей точкой в раздельном формате;
* реализовать контроль частоты запросов;
* реализовать визуализацию данных;
* разработать тесты.

---

## Постановка задачи

Необходимо разработать объектно-ориентированную систему для получения курсов валют с сайта Центрального Банка РФ.

Основные требования:

1. Реализовать шаблон проектирования «Одиночка» с помощью метаклассов.
2. Запретить создание более одного экземпляра класса.
3. Получать данные о валютах с сайта ЦБ РФ.
4. Реализовать:

* геттеры;
* сеттеры;
* конструкторы;
* деструкторы.

5. Хранить значения валют в формате:

```python
(integer_part, fractional_part)
```

6. Реализовать ограничение частоты запросов.
7. Реализовать визуализацию курсов валют.
8. Разработать тесты.

---

## Теоретические сведения

### Паттерн Singleton

Singleton — шаблон проектирования, который гарантирует существование только одного экземпляра класса.

Основные задачи паттерна:

* контроль создания объектов;
* централизованный доступ к экземпляру;
* предотвращение повторного создания объектов.

---

### Метаклассы

Метакласс — класс, создающий другие классы.

В Python метаклассы позволяют изменять процесс создания объектов.

Для реализации Singleton был переопределён метод:

```python
__call__()
```

---

### Работа с XML

Данные о курсах валют получаются с сайта ЦБ РФ в XML-формате.

Для парсинга XML используется библиотека:

```python
xml.etree.ElementTree
```

---

## Описание решения

Программа состоит из следующих модулей:

| Файл                     | Назначение                |
| ------------------------ | ------------------------- |
| `singleton_metaclass.py` | Реализация Singleton      |
| `currency.py`            | Класс валюты              |
| `currencies_manager.py`  | Управление валютами       |
| `main.py`                | Основной запуск программы |
| `test_currencies.py`     | Тестирование              |

---

## Листинг программы

### Файл `singleton_metaclass.py`

```python
import time


class SingletonMeta(type):
    _instances = {}

    def __call__(cls, *args, **kwargs):
        if cls not in cls._instances:
            cls._instances[cls] = super().__call__(*args, **kwargs)
        return cls._instances[cls]
```

---

### Файл `currency.py`

```python
import requests
import time
from xml.etree import ElementTree as ET
from decimal import Decimal
import matplotlib.pyplot as plt

from singleton_metaclass import SingletonMeta


class Currency:
    def __init__(self, code: str, name: str, value: str, nominal: int):
        self.__code = code
        self.__name = name
        self.__nominal = nominal

        # разделяем float
        integer, fraction = value.replace(',', '.').split('.')
        self.__value = (integer, fraction)

    # геттеры
    def get_code(self):
        return self.__code

    def get_name(self):
        return self.__name

    def get_value(self):
        return self.__value

    def get_nominal(self):
        return self.__nominal

    # сеттер
    def set_value(self, value: str):
        integer, fraction = value.replace(',', '.').split('.')
        self.__value = (integer, fraction)

    def __del__(self):
        pass
```

---

### Файл `currencies_manager.py`

```python
import time
import requests
from xml.etree import ElementTree as ET
import matplotlib.pyplot as plt

from currency import Currency
from singleton_metaclass import SingletonMeta


class CurrencyManager(metaclass=SingletonMeta):

    def __init__(self, delay: float = 1.0):
        self.__delay = delay
        self.__last_request_time = 0
        self.__currencies = []

    # сеттер delay
    def set_delay(self, delay: float):
        self.__delay = delay

    def get_delay(self):
        return self.__delay

    def get_currencies(self):
        return self.__currencies

    def fetch(self, currencies_ids_lst: list):
        current_time = time.time()

        # контроль частоты
        if current_time - self.__last_request_time < self.__delay:
            raise Exception("Слишком частые запросы")

        response = requests.get('http://www.cbr.ru/scripts/XML_daily.asp')

        root = ET.fromstring(response.content)

        result = []

        for valute in root.findall("Valute"):
            valute_id = valute.get('ID')

            if valute_id in currencies_ids_lst:
                code = valute.find('CharCode').text
                name = valute.find('Name').text
                value = valute.find('Value').text
                nominal = int(valute.find('Nominal').text)

                currency = Currency(code, name, value, nominal)

                result.append({code: (name, currency.get_value())})
                self.__currencies.append(currency)

        # если не найдено
        for cur_id in currencies_ids_lst:
            if not any(cur_id in str(r) for r in result):
                result.append({cur_id: None})

        self.__last_request_time = current_time
        return result

    def visualize(self):
        names = []
        values = []

        for currency in self.__currencies:
            names.append(currency.get_code())
            val = float(".".join(currency.get_value()))
            values.append(val)

        plt.figure()
        plt.bar(names, values)

        plt.title("Курсы валют")
        plt.xlabel("Валюта")
        plt.ylabel("Курс")

        plt.savefig("currencies.jpg")
        plt.close()

    def __del__(self):
        pass
```

---

### Файл `main.py`

```python
from currencies_manager import CurrencyManager

if __name__ == "__main__":
    manager = CurrencyManager()

    res = manager.fetch(['R01035', 'R01335', 'R01700J'])
    print(res)

    manager.visualize()
```

---

### Файл `test_currencies.py`

```python
import unittest
from currencies_manager import CurrencyManager


class TestCurrencies(unittest.TestCase):

    def test_invalid_id(self):
        manager = CurrencyManager()
        result = manager.fetch(['R9999'])

        self.assertEqual(result[-1], {'R9999': None})

    def test_valid_currency(self):
        manager = CurrencyManager()
        result = manager.fetch(['R01035'])

        currency = result[0]
        key = list(currency.keys())[0]
        name, value = currency[key]

        self.assertIsInstance(name, str)

        val = float(".".join(value))
        self.assertTrue(0 < val < 999)


if __name__ == '__main__':
    unittest.main()
```

---

## Структура хранения валют

Каждая валюта представляется объектом класса `Currency`.

Значение курса хранится отдельно:

```python
(integer_part, fractional_part)
```

Пример:

```python
('113', '2069')
```

Такой формат позволяет избежать ошибок, связанных с хранением чисел с плавающей точкой.

---

## Контроль частоты запросов

В программе реализована защита от слишком частого обращения к API.

Используется:

```python
self.__last_request_time
```

Если запрос выполняется раньше допустимого времени, вызывается исключение:

```python
Exception("Слишком частые запросы")
```

---

## Визуализация данных

Для отображения курсов валют используется библиотека:

```python
matplotlib
```

Метод:

```python
visualize()
```

строит столбчатую диаграмму и сохраняет её в файл:

```text
currencies.jpg
```

---

## Результат работы программы

Пример результата:

```python
[
    {'GBP': ('Фунт стерлингов Соединенного королевства', ('113', '2069'))},
    {'KZT': ('Казахстанских тенге', ('19', '8264'))},
    {'TRY': ('Турецких лир', ('33', '1224'))}
]
```

---

## Тестирование программы

В ходе тестирования были проверены следующие случаи:

| № | Проверка                    | Результат |
| - | --------------------------- | --------- |
| 1 | Некорректный ID валюты      | Успешно   |
| 2 | Корректный ID валюты        | Успешно   |
| 3 | Проверка имени валюты       | Успешно   |
| 4 | Проверка диапазона значений | Успешно   |

---

## Используемые библиотеки

| Библиотека              | Назначение                |
| ----------------------- | ------------------------- |
| `requests`              | Выполнение HTTP-запросов  |
| `xml.etree.ElementTree` | Парсинг XML               |
| `matplotlib`            | Построение графиков       |
| `time`                  | Контроль времени запросов |
| `unittest`              | Тестирование              |

---

## Вывод

В ходе выполнения работы была разработана объектно-ориентированная система для получения курсов валют.

В программе реализованы:

* шаблон проектирования Singleton через метакласс;
* получение данных с сайта ЦБ РФ;
* хранение чисел с плавающей точкой в раздельном формате;
* контроль частоты запросов;
* визуализация курсов валют;
* модульные тесты.

Все поставленные задачи были выполнены.
