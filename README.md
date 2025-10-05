# Лабораторная работа №2: Объектно-ориентированное программирование в Java

## Содержание
1. [Структура проекта](#структура-проекта)
2. [Как работает основной код в Main](#как-работает-основной-код-в-main)
3. [Класс Time](#класс-time)
4. [Класс House](#класс-house)
5. [Классы Department и Employee](#классы-department-и-employee)
6. [Класс Gun](#класс-gun)
7. [Вспомогательные классы](#вспомогательные-классы)

## Структура проекта
Проект состоит из следующих классов:

- `Main` - главный класс с точкой входа программы
- `Time` - класс для представления времени суток
- `House` - класс для представления дома с этажами
- `Gun` - класс для представления пистолета с патронами
- `UserInput` - класс для обработки пользовательского ввода
- `Check` - класс с вспомогательными методами проверки
- Пакет `Departament` содержащий:
  - `Department` - класс для представления отдела
  - `Employee` - класс для представления сотрудника

## Как работает основной код в Main
```java
do{
    if(choice == 2){
        choiceTask = input.inputChoiceInt(1,4, "тему:\n1. Время\n2. Дом\n3. Отделы и сотрудники\n4. Пистолет");
    }

    switch (choiceTask) {
        case 1: // Работа с Time
        case 2: // Работа с House  
        case 3: // Работа с Department и Employee
        case 4: // Работа с Gun
    }
    
    System.out.println("Повторить?\n1. Да\n2. К выбору задания\n3. Выход");
    choice = input.inputDiaposonInt(1, 3, "вариант");
    if(choice == 3){
        exit = true;
    }
} while (!exit);
```

### Логика работы основного цикла:
- Переменная `choiceTask` отвечает за выбор темы для демонстрации
- Переменная `choice` управляет поведением после выполнения задания:
  - 1 - повторить текущее задание
  - 2 - вернуться к выбору темы
  - 3 - выход из программы
- Цикл `do-while` обеспечивает непрерывную работу программы до явного выхода пользователем

## Класс Time

### Назначение:
Класс представляет время суток в 24-часовом формате, храня количество секунд, прошедших с начала суток.

### Конструкторы:
- `Time()` - создает время 00:00:00
- `Time(int totalSeconds)` - создает время из заданного количества секунд
- `Time(Time time)` - конструктор копирования

### Свойства:
- `getTotalSeconds()` - возвращает общее количество секунд
- `setTotalSeconds(int totalSeconds)` - устанавливает время, проверяя корректность данных

### Логика работы toString():
```java
@Override
public String toString() {
    return String.format("%d:%02d:%02d", totalSeconds % 86400 / 3600, 
                         totalSeconds % 3600 / 60, totalSeconds % 60);
}
```
Используется статистический метод класса String - format

**Форматирование времени:**
- `totalSeconds % 86400 / 3600` - вычисляет часы (остаток от деления на секунды в сутках)
- `totalSeconds % 3600 / 60` - вычисляет минуты
- `totalSeconds % 60` - вычисляет секунды
- Формат `%02d` 0 - флаг обеспечивающий вывод с ведущими нулями, 2 - кол-во ячеек, d - число.

## Класс House

### Назначение:
Класс представляет дом с определенным количеством этажей. Реализован как объект с 1 полем, неизменяемым после создания.

### Конструкторы:
- `House(int floor)` - создает дом с указанным количеством этажей
- `House(House house)` - конструктор копирования

### Особенности реализации:
- Поле `floor` объявлено как `final` - нельзя изменить после создания
- Валидация в конструкторе: этажи должны быть в диапазоне 0-164
- Автоматическое определение правильного окончания для слова "этаж"

### Логика работы:
```java
@Override
public String toString() {
    if (floor % 100 >= 11 && floor % 100 <= 14) {
        return String.format("Дом с %d %s", floor, "этажами");
    }
    if(floor % 10 == 1){
        return String.format("Дом с %d %s", floor, "этажом");
    }
    return String.format("Дом с %d %s", floor, "этажами");
}
```
**Определение падежа:**
- Для чисел, оканчивающихся на 1 (кроме 11) - "этажом"
- Для чисел 11-14 - "этажами"  
- Для остальных случаев - "этажами"

## Классы Department и Employee

### Архитектурное решение - использование пакета:
Классы `Department` и `Employee` были вынесены в отдельный пакет `Departament` по следующим причинам:

1. **Логическая группировка** - эти классы тесно связаны и представляют единую логику
2. **Инкапсуляция** - пакет позволяет контролировать видимость методов
3. **Предотвращение циклических зависимостей** - четкое разделение ответственности

### Класс Employee

#### Назначение:
Представляет сотрудника с именем и связью с отделом.

#### Конструкторы:
- `Employee()` - сотрудник без имени и отдела
- `Employee(String name)` - сотрудник с именем без отдела
- `Employee(String name, Department department)` - сотрудник с именем и отделом
- `Employee(Employee copyEmployee)` - конструктор глубокого копирования

Во всех конструкторах классов делается проверка на исключения. Либо напрямую в конструкторе, либо через методы set[имя какого-либо поля], которые также обрабатывают исключения.

#### Свойства с особыми областями видимости:

```java
// Public методы - доступны извне пакета
public String getName() { return new String(name); }
public Department getDepartment() { 
    if (this.department == null) return null;
    return new Department(department); // Возвращает копию
}

// Package-private методы - доступны только внутри пакета
Department getRefDepartment() { return this.department; }
void setDepartment(Department department) { 
    this.department = department; 
}
```

**Обоснование различных областей видимости:**
- `getRefDepartment()` и `setDepartment()` с package-private видимостью предотвращают несанкционированное изменение связей извне пакета
- `getDepartment()` возвращает копию отдела для обеспечения инкапсуляции
- Перегрузка конструкторов повзоляют различные сценарии создания сотрудника

#### Логика toString():
```java
@Override
public String toString() {
    if (this.department == null) { // находится ли сотрудник в каком либо отделе
        ...
        return String.format("%s не находится ни в одном из отделов", this.getName());
    }
    if (this.department.getRefBoss() == this) { // если является боссом
        ...
        return String.format("%s начальник отдела %s", this.getName(), depName);
    }
    ///
    // формирование String depName и String bossName
    ///
    return String.format("%s работает в отделе %s, начальник которого %s",
                this.getName(), depName, bossName);
}
```

### Класс Department

#### Назначение:
Представляет отдел с названием, начальником и списком сотрудников.

#### Конструкторы:
- `Department()` - отдел без названия
- `Department(String name)` - отдел с названием
- `Department(String name, Employee boss)` - отдел с названием и начальником
- `Department(String name, Employee boss, ArrayList<Employee> employees)` - полная инициализация
- `Department(Department department)` - конструктор глубокого копирования

#### Ключевые методы:

**Управление сотрудниками:**
```java
public void addEmployee(Employee employee) {
    if(employee != null && !this.employees.contains(employee)){
        // Удаляем сотрудника из предыдущего отдела
        if(employee.getRefDepartment() != null && employee.getRefDepartment() != this){
            employee.getRefDepartment().removeEmployee(employee);
        }
        this.employees.add(employee);
        employee.setDepartment(this); // Используем package-private метод из класса Employee
    }
}

public void removeEmployee(Employee employee) {
    if(employee != null && this.employees.contains(employee)){
        this.employees.remove(employee);
        if (employee == this.boss){
            this.boss = null; // Сброс начальника при удалении
        }
        employee.setDepartment(null);
    }
}
```
Добавление и удаление сотрудника из отдела происходит именно с помощью этих методов. Нельзя использовать setDepartament у объекта класса Employee.

#### Методы с особыми областями видимости:

```java
// Public методы
public Employee getBoss() { 
    if(boss == null) return null;
    return new Employee(boss); // Возвращает копию
}

// Package-private метод
Employee getRefBoss() { return this.boss; }
```

**Обоснование:**
- `getRefBoss()` позволяет работать с оригинальным объектом внутри пакета для эффективности
- `getBoss()` возвращает копию для защиты от внешних изменений

#### Логика toString():
```java
@Override
public String toString() {
    String result = "Отдел\n==" + name + "==\n";
    
    if (boss != null) {
        result += "Начальник: " + boss.getName() + "\n";
    } else {
        result += "Начальник пока не назначен\n";
    }
    
    result += "Сотрудники:\n";
    // Вывод списка сотрудников (исключая начальника)
    // ...
    
    return result;
}
```
Вся информация по отделу формируется в String объект result.

## Класс Gun

### Назначение:
Класс представляет пистолет с ограниченным количеством патронов.

### Конструкторы:
- `Gun()` - пистолет с 5 патронами по умолчанию
- `Gun(int ammo)` - пистолет с указанным количеством патронов
- `Gun(Gun copyGun)` - конструктор копирования

### Методы:
```java
public void shot(){ // стрельба
    if(this.ammo > 0){
        System.out.println("БАХ!");
        this.ammo--;
    } else {
        System.out.println("Клац...");
    }
}

public void reload(){ // перезарядка (по заданию не предусматривалось)
    System.out.println("ПЕРЕЗАРЯДКА!!!");
    this.ammo = 12; // Максимальная емкость
}
```

## Вспомогательные классы

### Класс Check
Содержит методы валидации:
- `isInteger(String str)` - проверка целого числа
- `Positive(int num)` - проверка неотрицательности

### Класс UserInput
Обеспечивает безопасный ввод данных с консоли:
- `inputInt()`, `inputPositiveInt()` - ввод чисел
- `inputDiaposonInt()` - ввод числа в диапазоне
- `inputChoiceInt()` - ввод выбора из меню
- `inputString()` - ввод строки
