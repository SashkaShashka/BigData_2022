# Lab 4 - Zookeeper 
### По разрешению буду использовать библиотеку kazoo, которая очень похожа на ZooKeeper, но на python.
## Решение
### Dining philosophers
#### Задание
Решите проблему обедающих философов (каждый философ - отдельный процесс в системе)
#### Код и описание
1. Создадим класс ```Philosopher```, который в себе содержит свой id, id левой и правой ложки, id левого и правого соседа, имя задачи и "путь" до вилок.
```python
class Philosopher(Process):
    def __init__(self, task_name: str, id: int, fork_path: str, eat_seconds: int = 15, max_id: int = 5):
        super().__init__()
        self.root = task_name
        self.fork = fork_path
        self.id = id
        self.left_fork_id = id
        self.right_fork_id = id + 1 if id + 1 < max_id else 0
        self.eat_seconds = eat_seconds
        self.partner_id_left = id - 1 if id - 1 >= 0 else max_id-1
        self.partner_id_right = id + 1 if id + 1 < max_id else 0
```
2. Встаем в очередь, чтобы заблокировать стол (аналог общего процесса) и свои вилки (аналог менее общих процессов)
```python
table_lock = zk.Lock(f'{self.root}/table', self.id)
        left_fork = zk.Lock(f'{self.root}/{self.fork}/{self.left_fork_id}', self.id)
        right_fork = zk.Lock(f'{self.root}/{self.fork}/{self.right_fork_id}', self.id)
```
3. Будем эмулировать работу программы в течение ```eat_seconds``` для каждого философа
```python
start = time()
	while time() - start < self.eat_seconds:
```
4. "Блокируем" вилки только, если они свободны и философ поел не больше, чем его соседи
```python
 with table_lock:
	if len(left_fork.contenders()) == 0 and len(right_fork.contenders()) == 0 \
		and counters[self.partner_id_right] >= counters[self.id] \
		and counters[self.partner_id_left] >= counters[self.id]:
    	left_fork.acquire()
    	right_fork.acquire()
```
5. Если вилки "заблокированы", то кушаем, сообщаем об этом и увеличиваем счетчик количества приема пищи и "освобождаем" вилки, иначе думаем
```python
if left_fork.is_acquired and right_fork.is_acquired:
	print(f'Philosopher {self.id}: Im eating')
        counters[self.id] += 1
        sleep(MEAL_TIME_SEC)
        left_fork.release()
        right_fork.release()
	else:
print(f'Philosopher {self.id}: Im thinking')
	sleep(WAITING_TIME_SEC)
```
6. Создаем все процессы необходимые и задаем константы
```python
master_zk = KazooClient()
master_zk.start()
if master_zk.exists('/task1'):
	master_zk.delete('/task1', recursive=True)

master_zk.create('/task1')
master_zk.create('/task1/table')
master_zk.create('/task1/forks')
master_zk.create('/task1/forks/1')
master_zk.create('/task1/forks/2')
master_zk.create('/task1/forks/3')
master_zk.create('/task1/forks/4')
master_zk.create('/task1/forks/5')

root = '/task1'
fork_path = 'forks'
seconds_eat = 30
```
7. Создаем общий ```list```, который хранит в себе количество приемов пищи каждого из философов, т.к. обычные ```list``` и ```int``` не синхронизируются при работе нескольких потоков (ссылки на counter соседнего/соседних филосов в классе ```Philosopher``` хранить нельзя). Также создаем философов, добавляем их в ```list``` 
```python
counters = Manager().list()
p_list = list()
for i in range(0, 5):
	p = Philosopher(root, i, fork_path, seconds_eat)
	counters.append(0)
	p_list.append(p)
```
8. "Запускаем" каждого философа
```python
for p in p_list: 
    p.start()
```
#### Результат
Как можно видеть, каждый философ поел одинаковое количество раз (+-1), двух следующих итерациях поедят другие четыре философа и количество снова сравняется </br>
![image](https://user-images.githubusercontent.com/62326372/204723956-560d0e43-b723-4b4d-9a47-14bf1a1624ed.png)</br>
![image](https://user-images.githubusercontent.com/62326372/204723704-169b7b57-9cdf-4ec1-baae-995499c4b961.png)</br>

