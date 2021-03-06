# Управление заданиями пользователя

Для управления ресурсами кластера, а именно ресурсами узлов расчетного поля, используется система SLURM Workload Manager (в прошлом просто SLURM от Simple Linux Utility for Resource Management). Диспетчер данной системы следит за тем, кто, в каком объеме и на какое время занимает ресурсы узлов кластера. В настоящее время единицей выделения ресурсов является одно физическое ядро процессора (включая работы в режиме гиперпоточности).

Взаимодействие пользователя со SLURM чаще всего сопровождается указанием уникального номера задания, которое автоматически назначается пользователю после успешного выделения ресурсов. По данном номеру задания пользователь может получить всю информацию о его работе.

При работе рекомендуется использовать шпаргалку по основным командам [Slurm](https://slurm.schedmd.com/pdfs/summary.pdf).
Также за подробными инструкциями рекомендуется обращаться к [официальной документации](https://slurm.schedmd.com/documentation.html)

### Основные команды пользователя SLURM для управления заданиями

* `sinfo` - состояние узлов кластера
* `squeue` - состояние очереди заданий кластера
* `srun` - непосредственный запуск заданий
* `salloc` - запрос ресурсов и их использование в интерактивном режиме
* `sbatch` - фоновый запуск заданий (через очередь)
* `scancel` - отмена задания (прерывание исполнения, если в данный момент считается)
* `sacct` - статистика обработанных заданий
* `sstat` - статистика исполняемых заданий
* `scontrol` - управление диспетчерами SLURM
* `sreport` - отчетная информация

### Запрос ресурсов кластера (sbatch/salloc/srun)

Запуск программ возможен тремя способами:

* `sbatch` - фоновый пакетный режим
* `salloc` - режим резервирования ресурсов кластера для работы в интерактивном режиме
* `srun` - режим поточного исполнения программы

Данная группа команд используется для указания системе запустить некоторым образом программу пользователя на заданном количестве ресурсов на заданное время. Чаще всего требуются следующие 3 параметра:

- `n <число>` : кол-во запрашиваемых ядер, соответствующих количеству параллельных процессов задания пользователя; в соответствии с этим числом из набора доступных выделяется набор узлов кластера, которые далее будут использоваться для запуска задания пользователя;
- `N <число>` : минимальное кол-во узлов кластера, необходимое для запуска задания
- `t DD-HH:MM:SS` : время в формате ДНЕЙ-ЧАСОВ:МИНУТ:СЕКУНД, на которое требуется выделить узлы/ядра кластера;
- `p <intel/amd/gpu>` : очередь, в которой запускать задачу

#### Примеры
```bash
salloc -N 2 -n 80 -t 5-00:00:00 myjob.sh
```
Запросить минимум два полностью свободных от других задач узла кластера, на которых необходимо запустить задание myjob.sh, которому требуется 80 ядер.

```bash
salloc -n 80 -t 5-00:00:00 myjob.sh
```
Запросить некоторое количество узлов, на которых набирается 80 свободных ядер (не
факт, что их будет 2 в связи с тем, что могут исполняться и другие задания), на которых
необходимо запустить задание myjob.sh, которому требуется 80 ядер.

```bash
salloc -n 16 -t 5-00:00:00 myjob.sh
```
Аналогично верхнему примеру. Здесь тоже не гарантируется, что ядра могут быть выделены на одном узле. Хотя по умолчанию система будет в первую очередь выделять те ядра, которые с точки зрения физической топологии располагаются ближе друг к другу (ядра одного процессора).

```
salloc -N 1 -n 40 -t 20-12:00:00 myjob.sh
```
В данном случае задание будет заблокировано, так как пользователь запросил время счета, превышающее текущий установленный предел. 
Пределы вычислительных очередей приведены в выводе команды `sinfo`.

```
salloc -N 10 -n 4000 -t 12:00:00 myjob.sh
```

Это тоже пример задания, которое будет по умолчанию заблокировано. Пользователь запросил количество ядер и узлов, превышающее текущий установленный предел.

Отдельно отметим , что все описанные пределы активны по умолчанию. В некоторых случаях Администраторы для отдельных заданий могут отменить действие пределов и, таким образом, разрешить выполнение заданий на большом количестве ядер на большое время. Такие запуски требуют предварительного согласования с Администраторами. Пользователю необходимо направить запрос по электронной почте с кратким указанием причины такой необходимости. Все запросы рассматриваются в индивидуальном порядке и зависят от текущей общей загрузки кластера.

Все три команды `sbatch/alloc/srun` принимают указанные выше параметры. Все три команды отправляют запрос на ресурсы, и, если таковые ресурсы имеются в наличии, то переходят к запуска задания. Отличие состоит в том, что делает команда дальше с параметром, указывающим на исполняемое задание, а также в том, что произойдет, если свободных ресурсов в данный момент не окажется.

### Интерактивный запуск

```
salloc [запрос ресурсов] [<команда> [<параметры команды> …]]
```
После выделения ресурсов salloc запустит (на узле доступа, с которого работает пользователь!!!) заданную команду. По окончанию исполнения команды произойдет автоматическое освобождение ресурсов. Данную команду удобно использовать вместе с интерпретатором командной строки:
```bash
[user@master ~]$ salloc -N 3 -p intel
salloc: Granted job allocation NNN
[user@master ~]$ # Тут можно запускать задачи например через mpirun без указания требуемых ресурсов
[user@master ~]$ # для завершения сессии нажмите сочетание клавиш ctrl+D
[user@master ~]$ exit
salloc: Relinquishing job allocation NNN
salloc: Job allocation NNN has been revoked.
[user@master ~]$
```
В этом случае пользователю запускается новый экземпляр оболочки, который имеет привязку к выделенным ресурсам, и пользователю сообщается НОМЕР ЕГО ЗАДАНИЯ. Пользователь может самостоятельно перейти на выделенный узел командой `ssh n#` (где # - номер узла) и осуществить необходимый запуск заданий. Поиск выделенного узла можно осуществить командой squeue (см. далее). По выходу из данного экземпляра оболочки произойдет автоматическое освобождение ресурсов.

### Фоновый запуск

```
sbatch [запрос ресурсов] [<bash-скрипт> [<параметры скрипта> ...]]
```

Пример запуска скрипта:
```bash
[user@master ~]$ sbatch -n 20 ./testjob.sh
Submitted batch job NNNN
[user@master ~]$
```
sbatch сразу запросит выделение указанных ресурсов, выведет уникальный номер задания пользователю в соответствии с запросом и на этом завершит свое выполнение. Запуск скрипта уже будет контролироваться диспетчерами SLURM самостоятельно. Пользователю лишь необходимо дождаться окончания выполнения задания.

Вместо вывода на экран все данные сохраняются в текстовый файл и размещаются в ту же папку, из которой запущено задание.

Запуск параллельных длительных вычислительных заданий рекомендуется производить через sbatch. Содержимое скрипта может быть достаточно произвольным до собственно момента запуска MPI-программы. Пример файла `testjob.sh`:
```bash
#!/bin/bash
## Секция с параметрами задачи. Список параметров не исключительный, все параметры не обязательные.
#SBATCH --job-name=serial_job_test    # Имя задачи
#SBATCH --mail-type=END,FAIL          # Когда уведомить о начале/завершении задачи (NONE, BEGIN, END, FAIL, ALL)
#SBATCH --mail-user=email@ufl.edu     # Куда отправить письмо	
#SBATCH --ntasks=10                    # Запустить на одном CPU
#SBATCH --partition=intel 			  # Запустить в очереди Intel
#SBATCH --mem=1gb                     # Требуемая память на задачу
#SBATCH --time=00:05:00               # Лимит времени выполнения hrs:min:sec
#SBATCH --output=serial_test_%j.log   # Имя файла для сохранения Standard output и error log
pwd; hostname; date

module load python

echo "Run MPI Job"

srun ./program1
srun -n 5 ./program2
srun -n 5 ./program3

date
```

Синтаксически данный файл должен представлять из себя обычный скрипт, написанный на языке командной оболочки. Допускается использовать все, что поддерживается самим интерпретатором командой строки, который указывается в первой строчке скрипта. Как указывалось выше, поддержка других интерпретаторов допускается.
Пользователь должен проверить, что данный файл является исполняемым:
```
chmod +x testjob.sh
```

Необходимо обратить внимание на параллельный запуск. Он должен выполняться с помощью команды SLURM `srun` (по аналогии с `mpirun`, предлагаемой всеми реализациями MPI). В скрипт у `srun` можно переопределять параметры запуска. Например, можно изменить значение параметра `-n` - кол-во параллельных процессов, которые будут созданы при запуске параллельнйо программы. Если `srun` ничего не указывается, то по умолчанию переносятся все параметры из `sbatch` и `salloc`.

В одном скрипте допускается поочередный запуск множества MPI-программ. Каждый srun в порядке следования в скрипте будет выполняться по завершению предыдущего.
Непосредственный запуск скрипта осуществляется следующим образом. Когда стало известно какие узлы выделены под задачу пользователя, но одном из этих узлов (главный узел) диспетчером узла запускается данный скрипт. Скрипт выполняется интерпретатором последовательно в соответствии с содержимым скрипта от имени пользователя, запустившего задание. Когда встречается `srun`, он автоматически начинает взаимодействие с SLURM, и производит запуск MPI-программы. Осуществляет разбиение и порождение параллельных процессов, обеспечивает настройку сетевой среды. По завершению параллельного запуска, все процессы уничтожаются, и продолжается последовательное выполнение скрипта.

### Экспресс-выполнение программы

```
srun [запрос ресурсов] [<MPI-команда> [<параметры команды> ...]]
```

После выделения ресурсов `srun` запустит (на выделенном узле(-ах) заданную MPI программу). По окончанию исполнения произойдет автоматическое освобождение ресурсов.

Данную команду удобно использовать для очень коротких запусков при наличии свободных ресурсов. Если в качестве команды srun будет предоставлено не MPI-программа, то запущенно такое количество экземпляров, которое соответствует параметру `-n`.

Также, srun можно использовать по аналогии с salloc, указав интерпритатор в качестве запускаемой программы:
```
srun -n 8 -N 2 --pty /bin/bash
```

### Текущая занятость кластера (sinfo)

Для того чтобы узнать состояние узлов кластера используется команды `sinfo`.

```
[user@master ~]$ sinfo
PARTITION AVAIL  TIMELIMIT  NODES  STATE NODELIST
intel        up 15-00:00:0      1  alloc n5
intel        up 15-00:00:0      7   idle n[6-12]
amd*         up 15-00:00:0      1  down* n1
amd*         up 15-00:00:0      3   idle n[2-4]
gpu          up 15-00:00:0      1   idle master
[yshevchenko@master ~]$
```

Основной вывод состоит из шести столбцов.

* `PARTITION` - раздел кластера. Фактически раздел можно представлять как очередь, в которую задания попадают по заявкам от пользователей. Доступно три раздела:
	1. `gpu` - главный узел, на котором установлены видеокарты. На нем рекомендуется запускать CUDA-задачи.
	2. `amd`* - узлы на базе процесоров AMD EPYC, звездочка означает что эта очередь будет выбрана по умолчанию.
	3. `intel` - узлы на базе процессоров Intel Xeon.
* `AVAIL` - состояние раздела
* `TIMELIMIT` - ограничение на время счета в очереди
* `NODES` - количество узлов в очереди с заданным состоянием и именами
* `STATE` - состояние узлов
	* `down` - выключены в административном порядке
	* `alloc` - используются для счета, заняты целиком
	* `mix` - используются для счета (обычно обозначает частичное использование
ресурсов)
	* `maint` - узлы на техническом обслуживании, временно выведены из эксплуатации
	* `idle` - свободны
* `NODELIST` - сетевые имена узлов. Квадратные скобки обозначают диапазоны узлов.
Например, `n[6-8,10-11]` обозначает узлы с сетевыми именами `n6`,`n7`,`n8`,`n10`,`n11`. По этим именам пользователь может с
помощью SSH переходить на узлы и отслеживать непосредственную работу его расчетов. Выход с узла осуществляется стандартной командой exit (ctrl+D).

### Просмотр очереди задач (squeue)

Для того чтобы узнать состояние очередей на кластере используется команда `squeue`

```
[user@master ~]$ squeue
 JOBID  PARTITION NAME     USER  ST TIME       NODES NODELIST(REASON)
 105328 intel     nl_.bat  user1 PD 0:00       4 	 (QOSMaxNodePerUserLimit)
 107333 amd       amp_400. user2 R  1:02       1 	 n2
 107334 gpu       amp_400. user2 R  1:02       1 	 master
 105301 intel     kap2.bat user1 R  8:56:00    4 	 n[6-9]
 105321 intel     nl2_.bat user1 R  2-10:52:46 1 	 n[10]
```

Основной вывод состоит из семи столбцов:

* `JOBID` - уникальный номер задания
* `PARTITION` - очередь на кластере
* `NAME` - название задания (можно задавать параметром --name через sbatch/salloc/srun;
по умолчанию имя скрипта или команды, указанной при запуске задания)
* `USER` - имя пользователя, инициатора данного задания
* `ST` - состояние задания
	* `PD` - запуска отложен, ожидает ресурсов (PENDING)
	* `R` - исполняется (RUNNING)
	* другие состояния скорее носят вспомогательный характер, их можно посмотреть в справочном руководстве (man squeue)
* `NODES` - количество используемых узлов
* `NODELIST(REASON)` - либо имена используемых узлов, либо вероятная причина того, почему данное задание ожидает в очереди
	* `Resources` - задание ожидает, потому что нет свободных ресурсов
	* `Priority` - задание ожидает, потому что впереди в очереди есть более приоритетные задания
	* другие причины чаще всего означают виды ограничений, по которым данное задание сейчас не может быть запланировано на запуск

По команде `squeue --start` можно посмотреть планируемое время запуска задания, если таковое уже было определено.

### Отмена задания (scancel)

Отмена задания осуществляется командой `scancel` и указанием номер отменяемого задания.

```
[user@master~]$ scancel 12345
```

###  Статистика обработанных заданий (sacct)

Для получения статистики выполненных заданий можно использовать команды sacct.
Основные параметры ее использования:

* `-S <дата-время>` : выводит информацию о заданиях с этой даты
* `-E <дата-время>` : выводит информацию о заданиях по эту дату
* `-o <format>` : список столбцов вывода информации. Параметр `--helpformat` выводит весь список поддерживаемых имен столбцов. Столбы указывается через запятую без пробелов. После имени столбца может идти знак процента и число (например,
`%20`), что означает использовать 20 символов для вывода информации в этом столбце. Процент используется для случаев, когда информации больше, чем предусмотрено стандартным размером столбца
* `-u` : имя пользователя, для которого выводит статистику

Пример:

```
[user@master ~]$ sacct -u user-S 2017-07-01T00:00:00 -E 2017-07-05T23:59:59 -o
JobId,JobName%10,State%20,Start%24,Elapsed
```