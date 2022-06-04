# deploy_jar

## Демон и скрипт для запуска с защитой от зацикливания

+ Имеется app.jar файл
+ Для его запуска используется команда:
*java -jar app.jar some_out_file "Service is working!"*
+ Напишите простой демон для systemd (Linux), который будет поддерживать работу приложения и перезапускать его в случае выхода из строя процесса.
Необходимо сделать защиту от зацикливания перезапусков, когда процесс постоянно выходит из строя. 

### Создаем скрипт

1) Скопировать файл *app.jar* в папку */usr/local/bin*
> sudo cp /папка/отправитель/app.jar /usr/local/bin
3) Создать скрипт в любом текстовом редакторе в папке */usr/local/bin/* и открыть его
> sudo touch /usr/local/bin/myapp.sh && sudo nano /usr/local/bin/myapp.sh
5) Добавить в открытый файл следующий код
```
#!/bin/sh
SERVICE_NAME=myapp
PATH_TO_JAR=/usr/local/bin/app.jar
PID_PATH_NAME=/tmp/myapp-pid
case $1 in
start)
       echo "Starting $SERVICE_NAME ..."
  if [ ! -f $PID_PATH_NAME ]; then
       nohup java -jar $PATH_TO_JAR /tmp 2>> /dev/null >>/dev/null & echo $! > $PID_PATH_NAME
       echo "$SERVICE_NAME started ..."
  else
       echo "$SERVICE_NAME is already running ..."
  fi
;;
stop)
  if [ -f $PID_PATH_NAME ]; then
         PID=$(cat $PID_PATH_NAME);
         echo "$SERVICE_NAME stoping ..."
         kill $PID;
         echo "$SERVICE_NAME stopped ..."
         rm $PID_PATH_NAME
  else
         echo "$SERVICE_NAME is not running ..."
  fi
;;
restart)
  if [ -f $PID_PATH_NAME ]; then
      PID=$(cat $PID_PATH_NAME);
      echo "$SERVICE_NAME stopping ...";
      kill $PID;
      echo "$SERVICE_NAME stopped ...";
      rm $PID_PATH_NAME
      echo "$SERVICE_NAME starting ..."
      nohup java -jar $PATH_TO_JAR /tmp 2>> /dev/null >> /dev/null & echo $! > $PID_PATH_NAME  
      echo "$SERVICE_NAME started ..."
  else
      echo "$SERVICE_NAME is not running ..."
     fi     ;;
 esac
```

4) Добавить скрипту права на запуск
> sudo chmod +x /usr/local/bin/myapp.sh

### Протестировать скрипт на запуск-стоп-перезапуск можно следующими командами:
```
/usr/local/bin/./myapp.sh start
/usr/local/bin/./myapp.sh stop
/usr/local/bin/./myapp.sh restart
```

### 5) Создаем демон

6) Создать файл myapp.service в любом текстовом редакторе в папке */etc/systemd/system/* и открыть его.
> sudo touch /etc/systemd/system/ && sudo nano /etc/systemd/system/myapp.service
8) Добавить в открытый файл следующий код
~~~
[Unit]
 Description = Deploy Java Service
 StartLimitInterval=200
 StartLimitBurst=5
[Service]
 Type=forking
 Restart=always
 RestartSec=30
 SuccessExitStatus=143 
 ExecStart=/usr/local/bin/myapp.sh start
 ExecStop=/usr/local/bin/myapp.sh stop
 ExecReload=/usr/local/bin/myapp.sh reload
[Install]
 WantedBy=multi-user.target
~~~

8) Сохранить и выйти из текстового редактора
9) Перезагрузить настройки демонов
> sudo systemctl daemon-reload
- команда выполняется каждый раз после внесения изменений в файл конфигурации демонов.

### Управлять работой демона можно следующими командами
```
sudo systemctl daemon-reload
sudo systemctl enable myapp
sudo systemctl start  myapp
sudo systemctl stop   myapp
```
