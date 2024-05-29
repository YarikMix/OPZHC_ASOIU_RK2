# РК-2, вариант 39

## Михалёв Ярослав, ИУ5-61Б

## Задание
Составить на ПОСП (ОПС, GPSS) алгоритм имитационной модели процесса функционирования следующей системы: В составе автоматизированной системы управления технологическим процессом используется управляющий контроллер (УК). По запросам от объекта управления (ОУ) УК выполняет чтение управляющей информации с запоминающего устройства (ЗУ) и передаёт её на ОУ. По каждому запросу может требоваться от 1 до 4 операций чтения информации с ЗУ. Каждая операция включает поиск информации и её считывание. Время поиска Тп - случайная величина от 1 до 3 мс. Объём информации, считываемой в каждой операции, представляет собой случайную величину равномерно распределенную в интервале от 1024 до 4096 бит. Скорость считывания - 10 Мбит/с. Допустимое время реакции УК на запрос - не более 7 мс. Если это время будет превышено, то произойдет сбой. Предусмотреть сбор данных о количестве превышений времени реакции.

```pseudo
; GPSS модель функционирования управляющего контроллера

GENERATE 5,3  ; Генерация запросов каждые 5±3 мс
ASSIGN 1,UNIF(1,4)  ; Количество операций чтения (от 1 до 4)
MARK 1  ; Метка начала обработки запроса

; Цикл чтения управляющей информации
READ_LOOP:
ASSIGN 2,1  ; Счётчик чтений (инициализация)
TEST L 2,ASGN1,END_LOOP  ; Проверка завершения цикла
    ; Операция поиска информации
    ADVANCE UNIF(1,3)  ; Время поиска от 1 до 3 мс

    ; Операция считывания информации
    ASSIGN 3,UNIF(1024,4096)  ; Объём данных от 1024 до 4096 бит
    ADVANCE ASGN3/10E3  ; Время считывания (Скорость 10 Мбит/с)

    ; Увеличение счётчика чтений
    ADD 2,1
    TRANSFER ,READ_LOOP  ; Переход к следующей операции чтения

END_LOOP:
; Оценка времени реакции
SAVEVALUE REACTION_TIME,$TNOW-MARK1  ; Время реакции на запрос

; Проверка превышения допустимого времени реакции
TEST GE REACTION_TIME,7,FAIL  ; Допустимое время реакции 7 мс
TRANSFER ,END

FAIL:
SAVEVALUE FAIL_COUNT,FAIL_COUNT+1  ; Увеличение счётчика сбоев

END:
TERMINATE 1  ; Завершение обработки запроса

; Сбор статистики
START 10000  ; Запуск имитации на 10000 запросов
```

# Пояснения:

process ЗапросОУ: Процесс обработки запроса от объекта управления (ОУ).

### var операции = random(1, 4);: 
Определяет случайное количество операций чтения (от 1 до 4).

### var времяРеакции = 0;: 
Инициализация переменной для подсчета времени реакции.

### for (var i = 1; i <= операции; i++): 
Цикл для выполнения каждой операции чтения.

### var времяПоиска = random(1, 3); delay(времяПоиска);: 
Симуляция времени поиска информации (от 1 до 3 мс).

### var объемИнформации = random(1024, 4096); var времяСчитывания = объемИнформации / (10 * 1024 * 1024 / 1000); delay(времяСчитывания);: 
Симуляция времени считывания информации. Перевод объема данных в миллисекунды (скорость чтения 10 Мбит/с).

### времяРеакции += времяПоиска + времяСчитывания;: 
Увеличение общего времени реакции.

### if (времяРеакции > 7) { increase(Сбой); }: 
Проверка на превышение допустимого времени реакции (7 мс). Если превышено, увеличивается счетчик сбоев.

### track Сбой = 0;: 
Определяет счетчик для отслеживания количества сбоев.

### event ГенерацияЗапросов:
Процесс генерации запросов от ОУ.

### while (true): 
Бесконечный цикл для непрерывной генерации запросов.

### execute ЗапросОУ;: 
Выполнение процесса обработки запроса.

### delay(random(2, 8));: 
Задержка перед следующим запросом (интервал от 2 до 8 мс для моделирования генерации каждые 5±3 мс).

### init: Начальная секция для запуска модели.

### execute ГенерацияЗапросов;: 
Запуск процесса генерации запросов при инициализации модели.

Этот алгоритм имитирует работу системы, собирает данные о времени реакции и количестве превышений допустимого времени.

### Написал псведокод:
```pseudo
Начало

// Инициализация счетчика сбоев
сбои = 0

// Функция для обработки запроса от объекта управления (ОУ)
Функция обработкаЗапроса() {
    // Определяем количество операций чтения для текущего запроса (от 1 до 4)
    операции = случайноеЧисло(1, 4)
    времяРеакции = 0

    // Цикл выполнения операций чтения
    Для i от 1 до операции {
        // Симуляция времени поиска информации (от 1 до 3 мс)
        времяПоиска = случайноеЧисло(1, 3)
        задержка(времяПоиска)
        времяРеакции += времяПоиска

        // Симуляция времени считывания информации
        объемИнформации = случайноеЧисло(1024, 4096) // в битах
        времяСчитывания = объемИнформации / (10 * 1024 * 1024 / 1000) // перевод в миллисекунды
        задержка(времяСчитывания)
        времяРеакции += времяСчитывания
    }

    // Проверка времени реакции
    если времяРеакции > 7 {
        сбои += 1 // Увеличение счетчика сбоев
    }
}

// Функция для генерации запросов от ОУ
Функция генерацияЗапросов() {
    пока (истина) {
        вызов обработкаЗапроса()
        // Генерация запросов каждые 5±3 мс
        задержка(случайноеЧисло(2, 8))
    }
}

// Главная программа
Главная() {
    вызов генерацияЗапросов()
}

Конец
```

### Описание псевдокода:
### сбои = 0: 
Инициализация счетчика для отслеживания количества сбоев.

### Функция обработкаЗапроса(): 
Функция, моделирующая обработку запроса от ОУ.

### операции = случайноеЧисло(1, 4): 
Определяет случайное количество операций чтения (от 1 до 4).

### времяРеакции = 0: 
Инициализация переменной для подсчета времени реакции.

### Для i от 1 до операции: 
Цикл для выполнения каждой операции чтения.

### времяПоиска = случайноеЧисло(1, 3); задержка(времяПоиска);: 
Симуляция времени поиска информации (от 1 до 3 мс).

### объемИнформации = случайноеЧисло(1024, 4096); времяСчитывания = объемИнформации / (10 * 1024 * 1024 / 1000); задержка(времяСчитывания);: Симуляция времени считывания информации. Перевод объема данных в миллисекунды (скорость чтения 10 Мбит/с).

### времяРеакции += времяПоиска + времяСчитывания;: 
Увеличение общего времени реакции.

### если времяРеакции > 7 { сбои += 1; }: 
Проверка на превышение допустимого времени реакции (7 мс). Если превышено, увеличивается счетчик сбоев.

### Функция генерацияЗапросов():
Функция, моделирующая генерацию запросов от ОУ.

### пока (истина) { вызов обработкаЗапроса(); задержка(случайноеЧисло(2, 8)); }: 
Бесконечный цикл для непрерывной генерации запросов с задержкой (интервал от 2 до 8 мс).

### Главная():
Главная функция программы.

### вызов генерацияЗапросов(): 
Запуск процесса генерации запросов при старте программы.

