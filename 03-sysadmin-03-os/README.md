# Домашнее задание к занятию "3.3. Операционные системы, лекция 1"

---
---

1. *Какой системный вызов делает команда cd ? В прошлом ДЗ мы выяснили,
что cd не является самостоятельной программой, это shell builtin,
поэтому запустить strace непосредственно на cd не получится.
Тем не менее вы можете запустить strace на /bin/bash -c 'cd /tmp' 
В этом случае вы увидите полный список системных вызовов, которые делает сам
bash при старте. Вам нужно найти тот единственный, который относится именно к
cd. Обратите внимание, что strace выдаёт результат своей работы
в поток stderr, а не в stdout.*

```bash
vagrant@vagrant:~$ strace /bin/bash -c 'cd /tmp'

...

chdir("/tmp")

...
```
Тот *"единственный"* - `chdir`

---
2. *Попробуйте использовать команду `file` на объекты разных типов на файловой системе.
Используя `strace` выясните, где находится база данных file на основании которой она делает свои догадки.*

```bash
vagrant@vagrant:~$ strace file /bin/bash
...
stat("/home/vagrant/.magic.mgc", 0x7ffe53b3a5e0) = -1 ENOENT (No such file or directory)
stat("/home/vagrant/.magic", 0x7ffe53b3a5e0) = -1 ENOENT (No such file or directory)
openat(AT_FDCWD, "/etc/magic.mgc", O_RDONLY) = -1 ENOENT (No such file or directory)
stat("/etc/magic", {st_mode=S_IFREG|0644, st_size=111, ...}) = 0
openat(AT_FDCWD, "/etc/magic", O_RDONLY) = 3
fstat(3, {st_mode=S_IFREG|0644, st_size=111, ...}) = 0
read(3, "# Magic local data for file(1) c"..., 4096) = 111
read(3, "", 4096)                       = 0
close(3)                                = 0
openat(AT_FDCWD, "/usr/share/misc/magic.mgc", O_RDONLY) = 3

...
```
В итоге `file` попробовал открыть файлы `~/.magic.mgc`, `~/.magic`, `/etc/magic.mgc` которые не нашел, открыл и прочитал файлы `/etc/magic`, `/usr/share/misc/magic.mgc`

---
3. *Предположим, приложение пишет лог в текстовый файл. Этот файл оказался удален (deleted в lsof), однако возможности сигналом сказать приложению переоткрыть файлы или просто перезапустить приложение – нет. Так как приложение продолжает писать в удаленный файл, место на диске постепенно заканчивается. Основываясь на знаниях о перенаправлении потоков предложите способ обнуления открытого удаленного файла (чтобы освободить место на файловой системе).*

Запускаем ping, удаляем лог, после находим удаленный файл и обнуляем его
```bash
 vagrant@vagrant:~$ ping 8.8.8.8 >ping_log
 vagrant@vagrant:~$ cat ping_log
 #64 bytes from 8.8.8.8: icmp_seq=61 ttl=113 time=33.0 ms
 vagrant@vagrant:~$ rm ping_log
 vagrant@vagrant:~$ ps u
USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
vagrant     1343  0.0  0.4   7340  4212 pts/0    Ss   14:25   0:00 -bash
vagrant     1356  0.0  0.0   7092   936 pts/0    S+   14:26   0:00 ping 8.8.8.8
vagrant     1399  0.1  0.4   7484  4348 pts/1    Ss   14:27   0:00 -bash
vagrant     1414  0.0  0.3   8892  3372 pts/1    R+   14:28   0:00 ps u

vagrant@vagrant:~$ sudo lsof -p 1356 |grep deleted
ping    1356 vagrant    1w   REG  253,0    26325 1048597 /home/vagrant/ping_log (deleted)

vagrant@vagrant:~$ cat /proc/1356/fd/1
#64 bytes from 8.8.8.8: icmp_seq=1601 ttl=110 time=32.9 ms
vagrant@vagrant:~$ > /proc/1356/fd/1

```
файл обнулен:

![ping](img\ping.png)
---
4. *Занимают ли зомби-процессы какие-то ресурсы в ОС (CPU, RAM, IO)?*

Занимают только запись в таблице процессов.

---
5. *В iovisor BCC есть утилита opensnoop:*
```bash
root@vagrant:~# dpkg -L bpfcc-tools | grep sbin/opensnoop
/usr/sbin/opensnoop-bpfcc
```
*На какие файлы вы увидели вызовы группы open за первую секунду работы утилиты? Воспользуйтесь пакетом `bpfcc-tools` для Ubuntu 20.04. Дополнительные [сведения по установке](https://github.com/iovisor/bcc/blob/master/INSTALL.md).*

![opensnoop](img\opensnoop.png)

---
6. *Какой системный вызов использует `uname -a`? Приведите цитату из man по этому системному вызову, где описывается альтернативное местоположение в `/proc`, где можно узнать версию ядра и релиз ОС.*
```bash
uname({sysname="Linux", nodename="vagrant", ...}) = 0
```
`uname -a` использует системный вызов `uname()`

Описание, где можно в `/proc` узнать версию ядра и релиз ОС (строка 50 man):

> Part of the utsname information is also accessible via /proc/sys/kernel/{ostype, hostname, osrelease, version, domainname}.

---
7. *Чем отличается последовательность команд через `;` и через `&&` в `bash`? Есть ли смысл использовать в `bash &&`, если применить `set -e`?*

В случае с && второе выражение выполниться если код возврата первого = 0. В случае с ; это просто разделитель.
Есть ли смысл использовать в bash `&&`, если применить `set -e`?
Из мана по set:
> -e  Exit immediately if a command exits with a non-zero status.

Соответственно действия равнозначны.

---
8. *Из каких опций состоит режим `bash set -euxo pipefail` и почему его хорошо было бы использовать в сценариях?*

`-e` - прерывает выполнение при ошибке любой команды кроме последней в последовательности;

`-u` - неустановленные переменные трактуются как ошибки;

`-x` - в стандартный поток ошибок пишется трассировка по каждой команде до её выполнения (с какими аргументами вызвана)
и в процессе;

`-o pipefail` - если в ряде команд в pipe будет ненулевое значение, то как результат конвейера вернется этот последний
ненулевой код.

Полезность использования в том, что сценарий в таком случае может не иметь обработчика ошибок (не нужно о нём думать)
и не будет продолжать работу при первом появлении ошибки (так как дальнейшая его работа скорее всего будет некорректной).
А если вывод сценария перенаправить в файл, то будет необходимая информация для анализа и устранения ошибки
(не нужно писать дополнительный вывод в протокол).
---
9. *Используя `-o stat` для `ps`, определите, какой наиболее часто встречающийся статус у процессов в системе. В `man ps` ознакомьтесь (`/PROCESS STATE CODES`) что значат дополнительные к основной заглавной буквы статуса процессов. Его можно не учитывать при расчете (считать S, Ss или Ssl равнозначными).*

Наиболее часто встречающийся статус `S` - interruptible sleep (waiting for an event to complete).
> `D`    uninterruptible sleep (usually IO)
> 
> `I`    Idle kernel thread
> 
> `R`    running or runnable (on run queue)
> 
> `S`    interruptible sleep (waiting for an event to complete)
> 
> `T`    stopped by job control signal
> 
> `t`    stopped by debugger during the tracing
> 
> `W`    paging (not valid since the 2.6.xx kernel)
> 
> `X`    dead (should never be seen)
> 
> `Z`    defunct ("zombie") process, terminated but not reaped by its parent
