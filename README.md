# Эмулятор процессора 1804
При запуске требуется передать путь к настроечному файлу, который лежит в resources/settings.cfg


## Правила записи программы
Программа записывается в виде символов `0`, `1`, `x`. Все символы `x` будут заменены на `0`. 
Длина команды должна составлять *36* символов. Для написания коммантария нужно начать строку с символа `#`. Разрешены 
только однострочные комментарии. Чтобы видеть комментарий в выводе программы, нужно написать `##` перед комментарием. 
Написанный таким образом комментарий привязывается к команде, которая следует за ним. Если будет указано несколько однострочных комментарив для передачи на вывод, будет выбран самый последний из них.

Формат команды микропроцессора 1804 можно в [wiki этого проекта](https://github.com/Hiraev/proc1804/wiki).

## Настроечный файл
Настроечный файл содержит следющие поля: 
1. `input` - файл с программой;
2. `output` - файл для записи информации о состоянии регистров;
3. `writer` - файл настройки печати;
4. `help` - файл с информацией о командах эмулятора;
5. `welcome` - файл с сообщением приветсвия.

Данный файл можно не трогать, за исключением `input` и `output`, если у вас несколько программ.

## Файл настройки вывода
Файл `writer` содержит поле `line`, которое определяет формат выводимой в файл и в консолько информацией.
Все необходимые поля нужно указывать разделяя их символом `:`. Список доступных полей можно посмотреть в файле [PrintableValue.java](https://github.com/Hiraev/proc1804/blob/master/src/system/PrintableValue.java). Они соответсвуют элементам перечисления. 

Выдержка из PrintableValue.java
```Java
public enum PrintableValue {
    COMMENT, N, Y, F, Q, NEXT, ADDR, FLAGS, OVR, C4, F3, Z,
    REGS, REG0, REG1, REG2, REG3, REG4, REG5, REG6, REG7, REG8, REG9, REG10, REG11, REG12, REG13, REG14, REG15,
    STACKS, STACK0, STACK1, STACK2, STACK3,
    SP
}
```

Все названия говорящие, поэтому можно легко сориентироваться в них и настроить вывод нужным образом, но некоторые могут потребовать допонительного объяснения. `N` - номер такта процессора после которого были получены значения, `NEXT` - адрес следующей команды, `ADDR` - текущий адрес. Поле `COMMENT` говорит о том, что будут выведены комментарии из входного файла с программой. Так же стоить обратить внимание на то, что поля `REGS`, `STACKS`, `FLAGS` не могут быть использованы с их более специфичными формами. Дублировать поля тоже запрещено.

Пример файла настройки вывода:
```cfg
line = N:ADDR:NEXT:Y:F:Q:REG0:COMMENT
```
Данный файл настроит вывод на печать выбраных полей в указанном порядке.

## Команды эмулятора

После того как эмулятор будет запущен вы получите сообщение приветсвия и информацию о командах.
После чего последует приглашение на ввод команды:
```txt
command:
```

На данный момент доступны следующие команды:
1. `c\clk [NUM]` - выполнить 1 так (или NUM тактов, если указано) процессора;
2. `s\state` - вывести текущее состояние;
3. `sh\stateHistory` - вывести все предыдущие состояния процессора;
4. `h\help` - вывести информационное сообщение;
5. `e\exit` - завершить работу и записать всю историю состояний в выходной файл.


## Пример работы 

Пусть у нас есть следующая программа:

_input.txt_

```bash
# Этот комментарий будет проигнорирован
## Загрузка 0010 в REG0
xxxx xxxx 0010 x011 x111 x011 xxxx 0000 0011
## Загрузка 1111 в Q
xxxx xxxx 0010 x000 x111 x011 xxxx xxxx 0000
## 	Загрузка 0000 в REG1
xxxx xxxx 0010 x011 x111 x011 xxxx 0001 0000
## 	Загрузка 0000 в REG2
xxxx xxxx 0010 x011 x111 x011 xxxx 0010 0000
#4 	Начинаем цикл
## 	Сделать REG2 = !(REG2 xor Q)
xxxx xxxx 0010 x011 x000 x110 0010 0010 xxxx
## Сделать REG1 = !(REG1 xor REG0)
xxxx xxxx 0010 x011 x001 x110 0000 0001 xxxx
## Циклический сдвиг вправо {REG0, Q} / 2-> {REG0, Q}
xxxx xxxx 0010 1100 0011 x011 xxxx 0000 xxxx
## REG0 = REG0 & 0111 (маска), дальше
xxxx xxxx 0010 x011 x101 x100 0000 0000 0111
#8 Выполняем ИЛИ REG0 or Q чтобы узнать ноль или нет
## Перейти в 4, если Z = 0
0000 0100 0000 x001 x000 x011 0000 xxxx xxxx
#9
## Y = REG0
xxxx xxxx 0010 x011 x011 x011 xxxx 0000 xxxx
#10
## Y = REG1, перейти в 11
0000 1001 0001 x011 x011 x011 xxxx 0001 xxxx
```

Настроечный файл вывода выглядит следующим образом:
```cfg
#
#   Default settings - print everything
#   Write a format of printing line
#   You can use any format you want
#   line = Y:F:REG0 - for print only this three values
#   You can see all available values in src/system/PrintableValue.java
#
line = N:ADDR:NEXT:Y:F:REG0:REG3:REG2:REG1:Q:COMMENT
```

Запустим эмулятор передав ему путь к настроечному файлу: `resources/settings.cfg`. Если вы не знаете как в _Intelij IDEA_ передать аргументы программе, то мне вас очень жаль. 
После запуска будет получено сообщение с приветсвием. 
Введем сразу `s` для вывода текущего состояния регистров процессора.
```
command: s
<--- state
+----+----+----+----+----+-----+-----+-----+-----+----+--------------------------------------------------+
| clk|addr|next|   Y|   F| REG0| REG3| REG2| REG1|   Q|comment                                           |
+----+----+----+----+----+-----+-----+-----+-----+----+--------------------------------------------------+
|   0|   0|   0|0000|0000| 0000| 0000| 0000| 0000|0000|Загрузка 0010 в REG0                              |
+----+----+----+----+----+-----+-----+-----+-----+----+--------------------------------------------------+
command: 
```
Так как еще ни одного такта ещ не было и мы даже не начали читать команды, то все значения будут нулевыми. 
Комментарий дублируется для нулевого состояния и после загрузки первый команда - это небольшой баг.

Сделаем теперь один такт процессора и выведем новое состояние
```
command: c
<--- clk
command: s
<--- state
+----+----+----+----+----+-----+-----+-----+-----+----+--------------------------------------------------+
| clk|addr|next|   Y|   F| REG0| REG3| REG2| REG1|   Q|comment                                           |
+----+----+----+----+----+-----+-----+-----+-----+----+--------------------------------------------------+
|   1|   0|   1|0011|0011| 0011| 0000| 0000| 0000|0000|Загрузка 0010 в REG0                              |
+----+----+----+----+----+-----+-----+-----+-----+----+--------------------------------------------------+
command: 
```

Сделаем еще 5 тактов и выведем всю историю состояний.

```
command: c 5
<--- clk 5 times
command: sh
<--- stateHistory
+----+----+----+----+----+-----+-----+-----+-----+----+--------------------------------------------------+
| clk|addr|next|   Y|   F| REG0| REG3| REG2| REG1|   Q|comment                                           |
+----+----+----+----+----+-----+-----+-----+-----+----+--------------------------------------------------+
|   0|   0|   0|0000|0000| 0000| 0000| 0000| 0000|0000|Загрузка 0010 в REG0                              |
|   1|   0|   1|0011|0011| 0011| 0000| 0000| 0000|0000|Загрузка 0010 в REG0                              |
|   2|   1|   2|0000|0000| 0011| 0000| 0000| 0000|0000|Загрузка 1111 в Q                                 |
|   3|   2|   3|0000|0000| 0011| 0000| 0000| 0000|0000|Загрузка 0000 в REG1                              |
|   4|   3|   4|0000|0000| 0011| 0000| 0000| 0000|0000|Загрузка 0000 в REG2                              |
|   5|   4|   5|0000|0000| 0011| 0000| 0000| 0000|0000|Сделать REG2 = !(REG2 xor Q)                      |
|   6|   5|   6|0011|0011| 0011| 0000| 0000| 0011|0000|Сделать REG1 = !(REG1 xor REG0)                   |
+----+----+----+----+----+-----+-----+-----+-----+----+--------------------------------------------------+
command: 
```

Для завершения работы введем команду `e` после чего все состояния процессора на всех тактах его работы будут
записаны в файл, указанный в `resources/settings.cfg`.

Таким образом работает эмулятор. Обо всех багах прошу сообщать незамедлительно. Пишите `issues`, а лучше 
найдите причину ошибки, избавьтесь от нее и сделайте `pull request`.
