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
### Two phase сommit
#### Задание
Реализуйте двуфазный коммит протокол для high-available регистра (каждый регистр - отдельный процесс в системе)
#### Код и описание
1. Создадим класс ```Client```, который в себе содержит свой id, путь до своего узла и KazooClient.
```python
def __init__(self, root: str, id: int, zk):
	super().__init__()
	self.url = f'{root}/{id}'
	self.root = root
	self.id = id
	self.zk = zk
```
2. В методе ```watch_myself``` происходит обработка действий, которые приходят от ```Coordinator``` - сделать commit, rollback или завершить поток, по причине отсоединия одного из клентов. В коде эта возможность предусмотрена, но не прописано случайное отключение одного из клиентов.
```python
def watch_myself(data, stat):
    if data != ACTION_DEAD:
	if(stat.version == 1):
	    sleep(1)
	print(f'Client {self.id} triggered {stat.version}')
	if stat.version != 0:
	    print(f'Client {self.id} do {data.decode()}')
	print(f'Client {self.id} run')
    else:
	print(f'Client {self.id} exit')
```
3. В методе ```run``` происходит случайный выбор действия ACTION/ROLLBACK, а также происходит запуск функции watch_myself, которая следит за изменениями своего узла. По истечению времени ```WAIT_HARD_WORK_SEC``` клиенты завершают свою работу.
```python
def run(self):      
	self.zk.start()

	value = ACTION_COMMIT if random.random() > 0.5 else ACTION_ROLLBACK
	print(f'Client {self.id} request {value.decode()}')
	self.zk.create(self.url, value, ephemeral=True)
	print(f'Client {self.id} create')
	datawatcher = DataWatch(self.zk, self.url, watch_myself)

	sleep(WAIT_HARD_WORK_SEC)
	self.zk.stop()
	print(f'Client {self.id} stop')
	self.zk.close()
```
4. ```Coordinator``` следит за работой всех потоков и содержит в себе единственное поле timer, который срабатывает по расписанию.
5. В методе ```select_action``` выбирается действие методом голосования и результат выбора отправялется каждому клиенту.
```python
def select_action():
	tr_clients = coordinator.get_children('/task_2/transaction')
	commit_counter = 0
	abort_counter = 0
	for client in tr_clients:
		commit_counter += int(coordinator.get(f'/task_2/transaction/{client}')[0] == ACTION_COMMIT)
		abort_counter +=  int(coordinator.get(f'/task_2/transaction/{client}')[0] == ACTION_ROLLBACK)

	final_action = ACTION_COMMIT if commit_counter == number_of_clients else ACTION_ROLLBACK
	for client in tr_clients:
		coordinator.set(f'/task_2/transaction/{client}', final_action)
````
6. Метод ```check_clients``` получает информацию о клиентах и оповещает других, если один из уже подключенных клиентов отсоединился и говорит им завершить свою работу.
```python
def check_clients():
	tr_clients = coordinator.get_children('/task_2/transaction')
	for i in range(len(Coordinator.session_logs)):
		if Coordinator.session_logs[i] is True and str(i) not in tr_clients:
			print("Switching off")
			Coordinator.timer.cancel()
			for client in tr_clients:
				coordinator.set(f'/task_2/transaction/{client}', ACTION_DEAD)
			sleep(0.5)
			for client in tr_clients:
				zks[int(client)].stop()
				zks[int(client)].close()
				processes[int(client)].kill()
			sys.exit()
```
7. Метод ```watch_clients``` устанавливает значения в ```session_logs``` при первом подключение клиента, а далее происходит обработка количества клиентов.
```python
@coordinator.ChildrenWatch('/task_2/transaction')
	def watch_clients(clients):
		for client in clients:
			Coordinator.session_logs[int(client)] = True

		if len(clients) == 0:
			if Coordinator.timer is not None:
				Coordinator.timer.cancel()
		else:
			if Coordinator.timer is not None:
				Coordinator.timer.cancel()
			Coordinator.timer = threading.Timer(duration, check_clients)
			Coordinator.timer.daemon = True
			Coordinator.timer.start()

		if len(clients) < number_of_clients:
			print(f'Waiting for the others. clients={clients}')
		elif len(clients) == number_of_clients:
			select_action()
```
8. В методе ```main```  создается общий процесс и клиенты
```python
Coordinator.session_logs = [False] * number_of_clients
coordinator = KazooClient()
coordinator.start()

if coordinator.exists('/task_2'):
	coordinator.delete('/task_2', recursive=True)

coordinator.create('/task_2')
coordinator.create('/task_2/transaction')

Coordinator.timer = None
root = '/task_2/transaction'
        
        for i in range(number_of_clients):
            zks.append(KazooClient())
            process = Client(root, i, zks[-1])
            processes.append(process)
            process.start()
            sleep(5)
```
#### Результат
Как видим все работает корректно </br>
![image](https://user-images.githubusercontent.com/62326372/204799029-f5b9c0c4-80d7-4cfe-a060-02b48f099c15.png)
