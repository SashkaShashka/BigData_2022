# Lab 2 - Reports with Apache Spark
### Установка и запуск 
1. В рабочую папку положили docker-compose.yml
2. Через cmd запустили команду: docker-compose up --build -d
3. Ждем скачивания и запуска образа
4. Для более удобной работы, подключимся к серверу из cmd  </br>
4.1. ssh root@localhost -p 2222  </br>
4.2. Password: mapr  </br>
5. Откроем выбочую директорию: cd /home/mapr/lab_2
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
### Решение
#### Топ 10 языков по годам
##### 2010 </br>
![image](https://user-images.githubusercontent.com/62326372/203838187-dc7afd5e-9fd5-43cc-8eb5-7f44c4be5196.png)</br>
##### 2011</br>
![image](https://user-images.githubusercontent.com/62326372/203838364-7151539d-e34b-4de3-bb2c-07e7c905fca6.png)</br>
##### 2012</br>
![image](https://user-images.githubusercontent.com/62326372/203838384-a7afb9c9-5ed5-4ead-a06d-e1f9f747a34f.png)</br>
##### 2013</br>
![image](https://user-images.githubusercontent.com/62326372/203838405-3c3d9404-7fd8-4bec-b877-f1f361be841e.png)</br>
##### 2014</br>
![image](https://user-images.githubusercontent.com/62326372/203838423-a69c8285-ee17-47c4-882f-ce9713f0d853.png)</br>
##### 2015</br>
![image](https://user-images.githubusercontent.com/62326372/203838446-fc7ced48-3012-4997-9021-a7f89d8238b2.png)</br>
##### 2016</br>
![image](https://user-images.githubusercontent.com/62326372/203838482-3926c945-b6b4-4f9a-a395-ded7feb87f5b.png)</br>
##### 2017</br>
![image](https://user-images.githubusercontent.com/62326372/203838500-42a0454d-c222-48ec-8602-b9c215928767.png)</br>
##### 2018</br>
![image](https://user-images.githubusercontent.com/62326372/203838522-cdd6939e-ea42-40a2-92db-429f3c079fed.png)</br>
##### 2019</br>
![image](https://user-images.githubusercontent.com/62326372/203838544-65e12829-51a3-405b-99df-d75ce0e33844.png)</br>
