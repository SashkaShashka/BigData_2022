# Lab 1 - Introduction to Apache Spark
### Установка и запуск 
1. В рабочую папку положили docker-compose.yml
2. Через cmd запустили команду: docker-compose up --build -d
3. Ждем скачивания и запуска образа
4. Для более удобной работы, подключимся к серверу из cmd
4.1. ssh root@localhost -p 2222
4.2. Password: mapr
Ждать.... После установки и запуска можно входить c помощью
5. Откроем выбочую директорию: cd /home/mapr/lab_1
6. Скачаем все необходимое: </br>
6.1. echo 'export PATH=$PATH:/opt/mapr/spark/spark-3.2.0/bin' > /root/.bash_profile </br>
6.2. source /root/.bash_profile </br>
6.3. apt-get update && apt-get install -y python3-distutils python3-apt </br>
6.4. wget https://bootstrap.pypa.io/pip/3.6/get-pip.py </br>
6.5. python3 get-pip.py </br>
6.6. pip install jupyter  </br>
6.7. pip install pyspark </br>
7. Запустим ноутбук командой: jupyter-notebook --ip=0.0.0.0 --port=50001 --allow-root --no-browser </br>
7.1. Открываем ссылку в браузере:</br>
![image](https://user-images.githubusercontent.com/62326372/203779844-c7fdcf70-979d-4c33-b282-e51183c69b9f.png)
### Маленькое описание
В рабочем ноутбуке загрузили необходимые файлы для лабораторной работы, построили классы, методы для парсинга и немного потестировали, что работает корректно.
Далее выполнялись задания. 
##### Все комментарии по методом, и чтобы было сделано в каждой ячейке соотвествующего задания.
### Ответы
#### Найти велосипед с максимальным временем пробега
![image](https://user-images.githubusercontent.com/62326372/203780421-60005db6-a6cd-4c43-b83c-758bcbca9a26.png)
#### Найти наибольшее геодезическое расстояние между станциями
![image](https://user-images.githubusercontent.com/62326372/203780505-46fa00ea-51be-4919-8010-9d56af4437a2.png)
#### Найти путь велосипеда с максимальным временем пробега через станции
![image](https://user-images.githubusercontent.com/62326372/203780585-4b409b55-b369-4b09-b29f-ddd7d4b4820a.png)
#### Найти количество велосипедов в системе
![image](https://user-images.githubusercontent.com/62326372/203780948-f91ee711-467a-47fa-8222-e64f507d10bd.png)
#### Найти пользователей потративших на поездки более 3 часов
![image](https://user-images.githubusercontent.com/62326372/203781004-26fab6d3-d4ed-4037-817d-1da12a44d1e5.png)
