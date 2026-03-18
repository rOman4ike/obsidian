# Refs
 
1. https://ru.hexlet.io/blog/posts/faylovyy-deskriptor-chto-eto-prostymi-slovami
2. https://habr.com/ru/companies/otus/articles/894854/
3. https://linuxtldr.com/proc-file-system/

# Questions

1. Команды для поиска file descriptors
2. Что такое `/proc/<PID>/fd`

# Raw info

## Start

> A file descriptor is a positive integer that acts as a unique identifier (or handle) for “files” and other I/O resources, such as pipes, sockets, blocks, devices, or terminal I/O.

All the file descriptor records are kept in a file descriptor table in the kernel. When a file is opened, a new file descriptor (or integer value) is given to that file in the file descriptor table.

For example, if you open a “example_file1.txt” file (which is nothing but a process), it will be allocated with the available file descriptor (for example, 101), and a new entry will be created in the file descriptor table.

And when you open another file like “example_file2.txt“, it will be allocated to another available file descriptor like 102, and another entry will be created in the file descriptor table.

|File Descriptor|Process|
|---|---|
|101|example_file1.txt|
|102|example_file2.txt|


В Linux всё — файл.

- **0** — это стандартный ввод `stdin`. Сюда поступают команды, нажатия клавиш и всякое такое.
    
- **1** — это стандартный вывод `stdout`. Все, что вы видите в терминале (например, результат выполнения команды), идет сюда.
    
- **2** — это стандартный поток ошибок `stderr`. Любая ошибка, предупреждение или критическая ситуация — всё это тут.

Эти операторы позволяют направить вывод туда, куда вам нужно, будь то файл, команда или даже другой поток.

> Оператор`>` - рабочий конвейер, который берет stdout и перенаправляет его в указанный файл.


> Оператор `2>` - берет `stderr` и пишет его в отдельный файл.


> Оператор `&>` - и stdout, и stderr вместе.

> Оператор `2>&1` - 


> `tee` - позволяет одновременно записывать данные в файл и выводить их на экран.

> `read` - ??

> `exec` - ??

Файловые дескрипторы в Linux и других ОС решают несколько задач.
1. Обеспечивают взаимодействие между программами и ОС: Когда программа открывает файл, она запрашивает у операционной системы доступ к нему. В ответ ОС выделяет файловый дескриптор, который программа использует для выполнения операций с этим файлом.
2. Управляют открытыми файлами и ресурсами: В операционной системе количество одновременно открытых файлов ограничено. Файловые дескрипторы позволяют системе отслеживать, какие из них открыты и какие операции с ними выполняются. Приведем классический пример: какой-то из файлов занимает слишком много места и мешает работе других программ. Через таблицу файловых дескрипторов можно увидеть, какой именно это файл, и изменить его состояние, чтобы продолжить работу.
3. Унифицируют работу с разными типами данных
4. Перенаправление стандартных потоков:
5. Контролируют утечки файловых дескрипторов: В системе можно установить ограничение на количество файловых дескрипторов. Если программа открывает файлы, но не закрывает их, возникает утечка дескрипторов, что может привести к нехватке ресурсов

**Важно:** файловый дескриптор может быть только положительным числом. Если число отрицательное, как в примере, появится сообщение об ошибке.
## What is the File Descriptor Table in Linux?

When a process or I/O device makes a successful request, the kernel returns a file descriptor to that process and keeps the list of current and all running process file descriptors in the file descriptor table, which is somewhere in the kernel.

Now, your process might depend on other system resources like input and output; as this event is also a process, it also has a file descriptor, which will be attached to your process in the file descriptor table.

Each file descriptor in the file descriptor table points to an entry in the kernel’s global file table. The file table entry maintains the record of file (or other I/O resource) modes like (`r`)ead, (`w`)rite, and (`e`)xecute

Also, the file table entry points to a third table known as the inode table that points to actual file information like size, modification date, pointer, etc.

![Kernel table](https://linuxtldr.com/wp-content/uploads/2022/12/Kernel-table-1024x710.webp)

## Predefined File Descriptors

By default, three types of standard POSIX file descriptors exist in the file descriptor table, and you might already be familiar with them as [data streams in Linux](https://linuxtldr.com/understanding-streams-in-linux/):

| File Descriptor | Name            | Abbreviation |
| --------------- | --------------- | ------------ |
| `0`             | Standard Input  | stdin        |
| `1`             | Standard Output | stdout       |
| `2`             | Standard Error  | stderr       |
Apart from them, every other process has its own set of file descriptors, but few of them (except for some daemons) also utilize the above-mentioned file descriptors to handle input, output, and errors for the process.

## List all of a Running Process’s File Descriptors

As you just learned, each running process in Linux has its own set of file descriptors, but it also uses others to identify the specific file when communicating with kernel space via system calls or library calls.

Find the Process ID (or PID)

First, find out your process identifier (or PID) using the ps command before viewing the file descriptors under it

```
$ ps aux | grep gedit
```

### Using the ls command

List all of the file descriptors and the files they refer to under a certain PID by listing the content of the “`/proc/PID/fd/`” path, where PID is the process ID, using the [ls command](https://linuxtldr.com/ls-command/).

- [Everything About /proc File System in Linux](https://linuxtldr.com/proc-file-system/)

```
$ ls -la /proc/11472/fd/
```

Output:

![Listing the process file descriptors using the ls command](https://linuxtldr.com/wp-content/uploads/2022/12/Listing-the-process-d-1024x562.webp)

### Using the lsof command

The lsof command is used to list the information about running processes in the system and can also be used to list the file descriptor under a specific PID.

For that, use the “`-d`” flag to specify a range of file descriptors, with the “`-p`” option specifying the PID. To combine this selection, use the “`-a`” flag.

```
$ lsof -a -d 0-2147483647 -p 11472
```

Output:

![Listing the process file descriptors using the lsof command](https://linuxtldr.com/wp-content/uploads/2022/12/Listing-the-process-file-descriptors-using-the-lsof-command-1024x437.webp)

## What is the Purpose of File Descriptors in the First Place?

The file descriptor, along with the file table, keep track of each running process’s permissions in your system and maintain data integrity.

A running process can inherit the functionality of another process by inheriting its file descriptor, as you just learned in this article.