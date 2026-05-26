# Отчёт по разработке программы построения бинарного дерева

## Цель работы

Разработать программу на языке Python для рекурсивного построения бинарного дерева с заданными параметрами.

Также необходимо:

* реализовать отображение дерева в виде словаря;
* реализовать вывод дерева;
* разработать модульные тесты с использованием библиотеки `unittest`.

---

## Постановка задачи

Необходимо реализовать функцию:

```python
def gen_bin_tree(height=<number>, root=<number>):
    pass
```

Функция должна строить бинарное дерево с учётом следующих условий:

* в корне дерева находится число `root`;
* высота дерева задаётся параметром `height`;
* левый потомок вычисляется по формуле:

```text
left_leaf = root^3
```

* правый потомок вычисляется по формуле:

```text
right_leaf = (root * 2) - 1
```

Если параметры не переданы, используются значения по умолчанию:

```text
root = 12
height = 4
```

Построение дерева должно быть реализовано рекурсивно.

---

## Описание алгоритма

Алгоритм работы функции `gen_bin_tree`:

1. Проверяется значение высоты дерева.
2. Если высота меньше либо равна нулю — функция возвращает `None`.
3. Вычисляются значения левого и правого потомков.
4. Рекурсивно создаются левое и правое поддеревья.
5. Формируется узел дерева в виде словаря.
6. Возвращается готовая структура дерева.

Каждый узел дерева содержит:

* значение корня (`root`);
* левое поддерево (`left`);
* правое поддерево (`right`).

---

## Листинг программы

### Файл `main.py`

```python
def gen_bin_tree(height=4, root=12):

    # Базовый случай: если высота 0 или меньше, возвращаем None
    if height <= 0:
        return None

    # Вычисляем левого и правого потомка по заданным формулам
    left_value = root ** 3  # root в степени 3
    right_value = (root * 2) - 1  # root умножить на 2, минус 1

    # Рекурсивно строим левое и правое поддеревья
    # Высота уменьшается на 1 для следующих уровней
    left_tree = gen_bin_tree(height - 1, left_value)
    right_tree = gen_bin_tree(height - 1, right_value)

    # Создаем узел дерева в виде словаря
    node = {
        'root': root,
        'left': left_tree,
        'right': right_tree
    }

    return node


def print_tree(tree, level=0): # Выводит дерево
    if tree is None:
        return

    # Отступ для визуализации уровней
    indent = "  " * level

    print(f"{indent}root: {tree['root']}")

    if tree['left'] is not None:
        print(f"{indent}left:")
        print_tree(tree['left'], level + 1)

    if tree['right'] is not None:
        print(f"{indent}right:")
        print_tree(tree['right'], level + 1)


# Основная программа
if __name__ == "__main__":
    print("Бинарное дерево")

    # Создаем дерево с параметрами по умолчанию
    tree1 = gen_bin_tree()
    print("Дерево с параметрами по умолчанию (height=4, root=12):")
    print(tree1)
    print("\nВывод:")
    print_tree(tree1)
```

---

## Представление дерева

Пример структуры дерева в виде словаря:

```python
{
    'root': 12,
    'left': {
        'root': 1728,
        'left': ...,
        'right': ...
    },
    'right': {
        'root': 23,
        'left': ...,
        'right': ...
    }
}
```

---

## Тестирование программы

Для проверки корректности работы программы были реализованы модульные тесты.

### Файл `test.py`

```python
import unittest
from main import gen_bin_tree


class TestBinTree(unittest.TestCase):

    def test_height_zero(self):
        """Тест для высоты 0 - должно вернуть None"""
        result = gen_bin_tree(height=0)
        self.assertIsNone(result)

    def test_height_negative(self):
        """Тест для отрицательной высоты - должно вернуть None"""
        result = gen_bin_tree(height=-1)
        self.assertIsNone(result)

    def test_height_one(self):
        """Тест для высоты 1 - только корень"""
        result = gen_bin_tree(height=1, root=5)
        expected = {
            'root': 5,
            'left': None,
            'right': None
        }
        self.assertEqual(result, expected)

    def test_height_two(self):
        """Тест для высоты 2"""
        result = gen_bin_tree(height=2, root=3)

        # Проверяем корень
        self.assertEqual(result['root'], 3)

        # Проверяем, что есть левое и правое поддеревья
        self.assertIsNotNone(result['left'])
        self.assertIsNotNone(result['right'])

        # Проверяем значения потомков
        self.assertEqual(result['left']['root'], 3 ** 3)  # 27
        self.assertEqual(result['right']['root'], (3 * 2) - 1)  # 5

        # Проверяем, что потомки являются листьями (нет своих детей)
        self.assertIsNone(result['left']['left'])
        self.assertIsNone(result['left']['right'])
        self.assertIsNone(result['right']['left'])
        self.assertIsNone(result['right']['right'])

    def test_default_parameters(self):
        """Тест с параметрами по умолчанию"""
        result = gen_bin_tree()

        # Проверяем корень по умолчанию
        self.assertEqual(result['root'], 12)

        # Проверяем, что дерево имеет правильную структуру (высота 4)
        self.assertIsNotNone(result['left'])
        self.assertIsNotNone(result['right'])

        # Проверяем значения непосредственных потомков
        self.assertEqual(result['left']['root'], 12 ** 3)  # 1728
        self.assertEqual(result['right']['root'], (12 * 2) - 1)  # 23

    def test_large_height(self):
        """Тест с большой высотой (проверка рекурсии)"""
        result = gen_bin_tree(height=5, root=1)

        # Проверяем, что дерево построено без ошибок
        self.assertIsNotNone(result)
        self.assertIsNotNone(result['left'])
        self.assertIsNotNone(result['right'])

        # Проверяем глубину дерева
        # Левый потомок левого потомка...
        left_left = result['left']['left']
        self.assertIsNotNone(left_left)


if __name__ == '__main__':
    unittest.main()
```

---

## Результаты тестирования

В ходе тестирования были проверены следующие случаи:

| № | Проверка                              | Результат |
| - | ------------------------------------- | --------- |
| 1 | Высота дерева равна 0                 | Успешно   |
| 2 | Отрицательная высота дерева           | Успешно   |
| 3 | Дерево высоты 1                       | Успешно   |
| 4 | Дерево высоты 2                       | Успешно   |
| 5 | Использование параметров по умолчанию | Успешно   |
| 6 | Построение дерева большой высоты      | Успешно   |

---

## Используемые структуры данных

Для хранения бинарного дерева использовался словарь Python (`dict`).

Каждый узел дерева содержит:

```python
{
    'root': значение,
    'left': левое поддерево,
    'right': правое поддерево
}
```

Такой способ хранения позволяет:

* удобно представлять структуру дерева;
* легко получать доступ к потомкам;
* рекурсивно обходить дерево.

---

## Используемые библиотеки

| Библиотека | Назначение             |
| ---------- | ---------------------- |
| `unittest` | Модульное тестирование |

---

## Вывод

В ходе выполнения работы была разработана программа для рекурсивного построения бинарного дерева.

Программа:

* поддерживает параметры высоты и значения корня;
* использует рекурсивный алгоритм;
* хранит дерево в виде словаря;
* позволяет выводить структуру дерева;
* успешно проходит модульные тесты.

Все поставленные задачи были выполнены.
