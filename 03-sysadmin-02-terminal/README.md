# Домашнее задание к занятию "3.2. Работа в терминале, лекция 2"

---

1. Какого типа команда `cd`? Попробуйте объяснить, почему она именно такого типа; опишите ход своих мыслей, если считаете что она могла бы быть другого типа.

- Встроенная в систему. Это "*Alma Mater*" команда для взаимодействия с системой.
```bash
vagrant@vagrant:~$ type cd
type cd
cd is a shell builtin
```

2. Какая альтернатива без pipe команде `grep <some_string> <some_file> | wc -l`? `man grep` поможет в ответе на этот вопрос. Ознакомьтесь с [документом](http://www.smallo.ruhr.de/award.html) о других подобных некорректных вариантах использования pipe.

- Альтернативой служит команда `grep -c <some_string> <some_file>`

3. Какой процесс с PID `1` является родителем для всех процессов в вашей виртуальной машине Ubuntu 20.04?

- Процесс с *PID* за номером *1* - **systemd**

4. Как будет выглядеть команда, которая перенаправит вывод stderr `ls` на другую сессию терминала?

- `ls -error 2> /dev/pts/pts/1`

5. Получится ли одновременно передать команде файл на stdin и вывести ее stdout в другой файл? Приведите работающий пример.

- Получится:
```bash
vagrant@vagrant:~$ grep alias < .bashrc > ~/grep_output_file
grep alias < .bashrc > ~/grep_output_file
vagrant@vagrant:~$ cat grep_output_file
cat grep_output_file
# enable color support of ls and also add handy aliases
    alias ls='ls --color=auto'
    #alias dir='dir --color=auto'
    #alias vdir='vdir --color=auto'
    alias grep='grep --color=auto'
    alias fgrep='fgrep --color=auto'
    alias egrep='egrep --color=auto'
# some more ls aliases
alias ll='ls -alF'
alias la='ls -A'
alias l='ls -CF'
# Add an "alert" alias for long running commands.  Use like so:
alias alert='notify-send --urgency=low -i "$([ $? = 0 ] && echo terminal || echo error)" "$(history|tail -n1|sed -e '\''s/^\s*[0-9]\+\s*//;s/[;&|]\s*alert$//'\'')"'
# ~/.bash_aliases, instead of adding them here directly.
if [ -f ~/.bash_aliases ]; then
    . ~/.bash_aliases
vagrant@vagrant:~$
```

6. Получится ли находясь в графическом режиме, вывести данные из PTY в какой-либо из эмуляторов TTY? Сможете ли вы наблюдать выводимые данные?

- Да, возможно.
`echo "from tty with love /dev/pts/1" >/dev/tty3`

7. Выполните команду `bash 5>&1`. К чему она приведет? Что будет, если вы выполните `echo netology > /proc/$$/fd/5`? Почему так происходит?

- Команда `bash 5>&1` запускает оболочку с перенаправлением потока ввода-вывода с файловым дескриптором 5 в стандартный поток вывода. В переменной `$$` хранится PID текущей оболочки, поэтому вторая команда выводит строку "netology" в поток с дескриптором 5 текущей оболочки, который, в свою очередь, перенаправлен в поток стандартного вывода, связанного с текущим терминалом.

8. Получится ли в качестве входного потока для pipe использовать только stderr команды, не потеряв при этом отображение stdout на pty? Напоминаем: по умолчанию через pipe передается только stdout команды слева от `|` на stdin команды справа. Это можно сделать, поменяв стандартные потоки местами через промежуточный новый дескриптор, который вы научились создавать в предыдущем вопросе.

- Что-то типа через *посредников*:
`ls /dir 6>&1 7>&2 2>&6 1>&7 | grep txt`
9. Что выведет команда `cat /proc/$$/environ`? Как еще можно получить аналогичный по содержанию вывод?

- Команда выведет переменные окружения текущей оболочки. Аналогичный по содержанию вывод команды `env`, только строки будут форматированные.

10. Используя `man`, опишите что доступно по адресам `/proc/<PID>/cmdline`, `/proc/<PID>/exe`.

- `/proc/[pid]/cmdline`
Данный файл, доступный только для чтения, содержит полную командную строку процесса, если процесс не является зомби. В
последнем случае этот файл пуст, поэтому чтение из него вернёт 0 символов.

- `/proc/[pid]/exe`
Файл является символьной ссылкой, содержащей актуальный путь выполняемой команды. 

11. Узнайте, какую наиболее старшую версию набора инструкций SSE поддерживает ваш процессор с помощью `/proc/cpuinfo`.

- `cat /proc/cpuinfo | grep sse` показывает старшую sse4_2

12. При открытии нового окна терминала и `vagrant ssh` создается новая сессия и выделяется pty. Это можно подтвердить командой tty, которая упоминалась в лекции 3.2. Однако:
```bash
vagrant@netology1:~$ ssh localhost 'tty'
not a tty
```
Почитайте, почему так происходит, и как изменить поведение.

- При выполнении удаленной команды через SSH не происходит аллокации TTY для удаленной сессии, т.к. эта сессия может быть использована для передачи файлов или другого содержимого. 

```bash
vagrant@vagrant:~$ ssh localhost 'tty'
...
Warning: Permanently added 'localhost' (ECDSA) to the list of known hosts.
vagrant@localhost's password:
not a tty
```
Аргумент -t используется для принудительного создания TTY
```bash
vagrant@vagrant:~$ ssh -t localhost 'tty'
vagrant@localhost's password:
/dev/pts/1
Connection to localhost closed.
vagrant@vagrant:~$
```

13. Бывает, что есть необходимость переместить запущенный процесс из одной сессии в другую. Попробуйте сделать это, воспользовавшись `reptyr`. Например, так можно перенести в `screen` процесс, который вы запустили по ошибке в обычной SSH-сессии.

- Получилось, но предыдущий терминал повис

Было на подвисшем терминале:
```bash
vagrant@vagrant:/$ ps u
USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
vagrant     1495  0.0  3.6  39640 36308 pts/0    Ss   15:34   0:00 -bash
vagrant     1743  0.0  0.4   7236  4196 pts/0    S    17:59   0:00 bash -v
vagrant     1852  0.0  0.4   7236  4160 pts/0    S+   20:37   0:00 bash
vagrant     2192  0.0  0.3   7932  3992 pts/0    T    21:31   0:00 htop -t
vagrant     2219  0.0  0.2   5836  2536 pts/0    T    21:35   0:00 less /var/log/faillog
vagrant     2280  0.0  0.4   7336  4204 pts/1    Ss   21:38   0:00 -bash
vagrant     2292  0.0  0.3   6820  3084 pts/0    S    21:39   0:00 screen
vagrant     2294  0.0  0.4   7236  4048 pts/2    Ss   21:39   0:00 /bin/bash
vagrant     2344  0.0  0.3   8892  3324 pts/2    R+   21:49   0:00 ps u
vagrant@vagrant:/$ reptyr 2292 
```
Стало на новом терминале:
```bash
vagrant@vagrant:~$ ps u
USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
vagrant     1495  0.0  3.6  39640 36308 pts/0    Ss   15:34   0:00 -bash
vagrant     1743  0.0  0.4   7236  4196 pts/0    S    17:59   0:00 bash -v
vagrant     1852  0.0  0.4   7236  4160 pts/0    S+   20:37   0:00 bash
vagrant     2192  0.0  0.3   7932  3992 pts/0    T    21:31   0:00 htop -t
vagrant     2219  0.0  0.2   5836  2536 pts/0    T    21:35   0:00 less /var/log/faillog
vagrant     2280  0.0  0.4   7336  4204 pts/1    Ss   21:38   0:00 -bash
vagrant     2292  0.0  0.3   6820  3100 pts/3    Ss+  21:39   0:00 screen
vagrant     2294  0.0  0.4   7236  4048 pts/2    Ss   21:39   0:00 /bin/bash
vagrant     2345  2.1  0.1   2592  1768 pts/2    S+   21:49   0:01 reptyr 2292
vagrant     2346  0.0  0.0      0     0 pts/0    Z    21:49   0:00 [screen] <defunct>
vagrant     2350  0.0  0.3   8892  3412 pts/1    R+   21:50   0:00 ps u
```
14. `sudo echo string > /root/new_file` не даст выполнить перенаправление под обычным пользователем, так как перенаправлением занимается процесс shell'а, который запущен без `sudo` под вашим пользователем. Для решения данной проблемы можно использовать конструкцию `echo string | sudo tee /root/new_file`. Узнайте что делает команда `tee` и почему в отличие от `sudo echo` команда с `sudo tee` будет работать.

- `tee` - читает данные из `stdin` и одновременно выводит их в `stdout` и указанный файл.

Главный плюс tee в том, что перенаправлением данных в файл она занимается лично, а не с помощью оболочки.
Поэтому совместно с sudo, tee имеет практически неограниченные возможности писать в чужие файлы.
