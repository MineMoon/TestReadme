```markdown
# Лабораторная работа №2: Система "Отделы и сотрудники"

## Описание проекта
Данный репозиторий представляет собой реализацию системы управления отделами и сотрудниками на Java. Проект демонстрирует работу с объектно-ориентированным программированием, включая инкапсуляцию, композицию, циклические ссылки и управление состоянием объектов.

## Содержание
1. [Описание проекта](#описание-проекта)
2. [Структура проекта](#структура-проекта)
3. [Класс Department](#класс-department)
4. [Класс Employee](#класс-employee)
5. [Взаимодействие классов](#взаимодействие-классов)
6. [Особенности реализации](#особенности-реализации)
7. [Как использовать](#как-использовать)

## Структура проекта
Проект состоит из следующих классов:

- `Department` - класс, представляющий отдел компании
- `Employee` - класс, представляющий сотрудника компании
- `Main` - главный класс с точкой входа программы (для тестирования)

## Класс Department

### Поля класса:
- `name` - название отдела
- `boss` - ссылка на начальника отдела (Employee)
- `employees` - список сотрудников в отделе (ArrayList<Employee>)

### Конструкторы:
```java
public Department() // создает отдел с именем "unknown" без сотрудников
public Department(String name) // создает отдел с указанным именем
public Department(String name, Employee boss) // создает отдел с именем и начальником
public Department(String name, Employee boss, ArrayList<Employee> employees) // полный конструктор
public Department(Department department) // конструктор копирования
```

### Основные методы:

#### Свойства (properties)
- `getName()` - возвращает копию названия отдела
- `setName(String name)` - устанавливает название отдела
- `getBoss()` - возвращает копию начальника отдела
- `getRefBoss()` - возвращает прямую ссылку на начальника (для внутреннего использования)
- `setBoss(Employee boss)` - назначает начальника отдела
- `getEmployees()` - возвращает копию списка сотрудников
- `setEmployees(ArrayList<Employee> employees)` - устанавливает список сотрудников

#### Бизнес-логика
- `addEmployee(Employee employee)` - добавляет сотрудника в отдел
- `removeEmployee(Employee employee)` - удаляет сотрудника из отдела
- `includesEmployee(Employee employee)` - проверяет наличие сотрудника в отделе

#### Особенности реализации:
```java
public void addEmployee(Employee employee) {
    if(employee != null && !this.employees.contains(employee)){
        // Безопасное удаление из старого отдела
        if(employee.getRefDepartment() != null && employee.getRefDepartment() != this){
            employee.getRefDepartment().removeEmployee(employee);
        }
        this.employees.add(employee);
        employee.setDepartment(this);
    }
}
```
**Логика работы:** При добавлении сотрудника в отдел, он автоматически удаляется из предыдущего отдела (если был в одном) и устанавливается двусторонняя связь.

## Класс Employee

### Поля класса:
- `name` - имя сотрудника
- `department` - ссылка на отдел, в котором работает сотрудник

### Конструкторы:
```java
public Employee() // создает сотрудника с именем "unknown" без отдела
public Employee(String name) // создает сотрудника с указанным именем
public Employee(String name, Department department) // создает сотрудника и добавляет в отдел
public Employee(Employee copyEmployee) // конструктор копирования
```

### Основные методы:

#### Свойства (properties)
- `getName()` - возвращает копию имени сотрудника
- `setName(String name)` - устанавливает имя сотрудника
- `getDepartment()` - возвращает копию отдела сотрудника
- `getRefDepartment()` - возвращает прямую ссылку на отдел (для внутреннего использования)
- `setDepartment(Department department)` - устанавливает отдел сотрудника (package-private)

#### Особенности реализации:
```java
public Employee(String name, Department department){
    this.setName(name);
    if(department != null){
        department.addEmployee(this); // отдел сам установит связь
    }
}
```
**Логика работы:** Конструктор делегирует установку связи отделу через метод `addEmployee()`, что обеспечивает согласованность данных.

## Взаимодействие классов

### Двусторонняя связь
Система поддерживает двустороннюю связь между отделами и сотрудниками:
- Отдел знает своих сотрудников через список `employees`
- Сотрудник знает свой отдел через ссылку `department`

### Управление циклическими ссылками
Для избежания проблем с циклическими ссылками реализованы:
- Методы `getRefBoss()` и `getRefDepartment()` для внутреннего использования
- Конструктор копирования создает "безопасные" копии
- Метод `toString()` избегает рекурсивных вызовов

### Согласованность данных
При изменении связей автоматически поддерживается согласованность:
```java
// При установке начальника
public void setBoss(Employee boss) {
    this.boss = boss;
    if (boss != null && !this.includesEmployee(boss)) {
        this.addEmployee(boss); // начальник автоматически добавляется в отдел
    }
}

// При удалении сотрудника
public void removeEmployee(Employee employee) {
    if(employee!=null && this.employees.contains(employee)){
        this.employees.remove(employee);
        if (employee == this.boss){
            this.boss = null; // если увольняют начальника
        }
        employee.setDepartment(null); // разрываем связь у сотрудника
    }
}
```

## Особенности реализации

### Обработка граничных случаев
- Проверки на `null` во всех критических методах
- Обработка пустых строк в именах
- Защита от добавления дубликатов сотрудников

### Безопасное копирование
```java
public Department getDepartment() {
    if (this.department == null) {
        return null;
    } else {
        return new Department(department); // возвращает копию, а не оригинал
    }
}
```

### Инкапсуляция
- Поля объявлены как `private`
- Для внутреннего использования в пакете доступны методы с `package-private` модификатором
- Геттеры возвращают копии объектов для защиты от внешних изменений

## Как использовать

### Создание отделов и сотрудников:
```java
Department itDepartment = new Department("IT");
Employee emp1 = new Employee("Алексей Петров", itDepartment);
Employee emp2 = new Employee("Мария Сидорова", itDepartment);
```

### Назначение начальника:
```java
itDepartment.setBoss(emp1);
```

### Перемещение сотрудника между отделами:
```java
Department hrDepartment = new Department("HR");
hrDepartment.addEmployee(emp2); // автоматически удалит из IT отдела
```

### Копирование отдела:
```java
Department itCopy = new Department(itDepartment);
```

### Вывод информации:
```java
System.out.println(itDepartment.toString());
System.out.println(emp1.toString());
```

Система автоматически управляет всеми связями и обеспечивает целостность данных при любых операциях с отделами и сотрудниками.
```
