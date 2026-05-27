# Паттерн «Декоратор» для работы с курсами валют

## Цель работы

Изучить применение шаблона проектирования «Декоратор» на языке Python.

В рамках работы необходимо:

* реализовать базовый компонент для получения курсов валют через API Центробанка РФ;
* реализовать декораторы для преобразования данных в YAML и CSV;
* использовать абстрактные классы и интерфейсы;
* реализовать сохранение данных в файлы;
* оформить программу в соответствии с требованиями PEP-8, PEP-257 и PEP-484.

---

## Постановка задачи

На основе шаблона «Декоратор» необходимо реализовать:

1. Базовый компонент, который:

* получает данные о курсах валют через API Центробанка;
* возвращает данные в формате JSON.

2. Конкретные декораторы:

* декоратор YAML;
* декоратор CSV.

Каждый декоратор должен:

* преобразовывать данные в соответствующий формат;
* сохранять результат в файл.

Дополнительные требования:

* использование `ABC` и `@abstractmethod`;
* наличие аннотаций типов;
* наличие документации;
* наличие тестов.

---

## Теоретические сведения

### Паттерн «Декоратор»

Паттерн «Декоратор» позволяет динамически расширять функциональность объектов без изменения их исходного кода.

Основные элементы паттерна:

| Элемент           | Назначение                             |
| ----------------- | -------------------------------------- |
| Component         | Базовый интерфейс компонентов          |
| ConcreteComponent | Основная реализация                    |
| Decorator         | Базовый класс декораторов              |
| ConcreteDecorator | Конкретное расширение функциональности |

---

### Абстрактные классы

Для реализации интерфейсов используется модуль `abc`.

Абстрактные методы объявляются с помощью декоратора:

```python
@abstractmethod
```

Это гарантирует обязательную реализацию методов в дочерних классах.

---

## Описание решения

Программа состоит из следующих компонентов:

1. Абстрактный интерфейс `Component`.
2. Класс `CurrencyComponent` для получения JSON-данных.
3. Базовый декоратор `Decorator`.
4. Декоратор `YamlDecorator`.
5. Декоратор `CsvDecorator`.

В качестве источника данных используется API Центробанка РФ:

```text
https://www.cbr-xml-daily.ru/daily_json.js
```

---

## Листинг программы

### Файл `main.py`

```python
import csv
import json
from abc import ABC, abstractmethod
from typing import Dict, Any

import requests
import yaml


class Component(ABC):
    """
    Базовый интерфейс компонента.
    """

    @abstractmethod
    def operation(self) -> Any:
        """
        Метод получения данных.
        """
        pass

    @abstractmethod
    def save(self, filename: str) -> None:
        """
        Метод сохранения данных в файл.
        """
        pass


class CurrencyComponent(Component):
    """
    Базовый компонент.
    Получает курсы валют в JSON.
    """

    URL = "https://www.cbr-xml-daily.ru/daily_json.js"

    def operation(self) -> Dict[str, Any]:
        """
        Получает данные от API ЦБ РФ.
        """

        response = requests.get(self.URL, timeout=10)

        return response.json()

    def save(self, filename: str) -> None:
        """
        Сохраняет JSON в файл.
        """

        data = self.operation()

        with open(filename, "w", encoding="utf-8") as file:
            json.dump(data, file, ensure_ascii=False, indent=4)


class Decorator(Component):
    """
    Базовый декоратор.
    """

    def __init__(self, component: Component) -> None:
        self.component = component

    def operation(self) -> Any:
        return self.component.operation()

    def save(self, filename: str) -> None:
        self.component.save(filename)


class YamlDecorator(Decorator):
    """
    Декоратор для YAML.
    """

    def operation(self) -> str:
        """
        Возвращает данные в YAML.
        """

        data = self.component.operation()

        yaml_data = yaml.dump(
            data,
            allow_unicode=True
        )

        return yaml_data

    def save(self, filename: str) -> None:
        """
        Сохраняет YAML в файл.
        """

        yaml_data = self.operation()

        with open(filename, "w", encoding="utf-8") as file:
            file.write(yaml_data)


class CsvDecorator(Decorator):
    """
    Декоратор для CSV.
    """

    def operation(self) -> str:
        """
        Возвращает CSV строку.
        """

        data = self.component.operation()

        result = "CharCode,Name,Value\n"

        for value in data["Valute"].values():
            result += (
                f"{value['CharCode']},"
                f"{value['Name']},"
                f"{value['Value']}\n"
            )

        return result

    def save(self, filename: str) -> None:
        """
        Сохраняет CSV в файл.
        """

        data = self.component.operation()

        with open(
            filename,
            "w",
            newline="",
            encoding="utf-8"
        ) as file:

            writer = csv.writer(file)

            writer.writerow(
                ["CharCode", "Name", "Value"]
            )

            for value in data["Valute"].values():
                writer.writerow(
                    [
                        value["CharCode"],
                        value["Name"],
                        value["Value"]
                    ]
                )


if __name__ == "__main__":

    component = CurrencyComponent()

    print("JSON:")
    print(component.operation())

    yaml_component = YamlDecorator(component)

    print("YAML:")
    print(yaml_component.operation())

    csv_component = CsvDecorator(component)

    print("CSV:")
    print(csv_component.operation())

    component.save("rates.json")
    yaml_component.save("rates.yaml")
    csv_component.save("rates.csv")
```

---

## Структура программы

### Интерфейс `Component`

Определяет общий интерфейс компонентов:

```python
operation()
save()
```

---

### Класс `CurrencyComponent`

Основной компонент программы:

* выполняет HTTP-запрос к API;
* получает JSON-данные;
* сохраняет данные в JSON-файл.

---

### Класс `YamlDecorator`

Декоратор:

* преобразует JSON в YAML;
* сохраняет данные в YAML-файл.

Для работы используется библиотека:

```python
yaml
```

---

### Класс `CsvDecorator`

Декоратор:

* преобразует данные в CSV;
* сохраняет данные в CSV-файл.

Для работы используется встроенная библиотека:

```python
csv
```

---

## Используемые библиотеки

| Библиотека | Назначение                     |
| ---------- | ------------------------------ |
| `requests` | Выполнение HTTP-запросов       |
| `json`     | Работа с JSON                  |
| `yaml`     | Работа с YAML                  |
| `csv`      | Работа с CSV                   |
| `abc`      | Реализация абстрактных классов |
| `typing`   | Аннотация типов                |

---

## Результат работы программы

После запуска программы:

1. Получаются актуальные курсы валют.
2. Данные отображаются:

* в формате JSON;
* в формате YAML;
* в формате CSV.

3. Создаются файлы:

```text
rates.json
rates.yaml
rates.csv
```

---

## Тестирование программы

Для проверки работы программы были реализованы тесты для:

* базового компонента;
* YAML-декоратора;
* CSV-декоратора.

Проверялись:

* корректность преобразования данных;
* корректность сохранения файлов;
* работа методов `operation()` и `save()`.

---

## Вывод

В ходе выполнения работы был реализован шаблон проектирования «Декоратор».

Программа:

* получает курсы валют через API Центробанка РФ;
* поддерживает преобразование данных в YAML и CSV;
* использует абстрактные классы и интерфейсы;
* сохраняет данные в файлы различных форматов.

Также были использованы:

* аннотации типов;
* документация PEP-257;
* требования PEP-8.

Все поставленные задачи были выполнены.
