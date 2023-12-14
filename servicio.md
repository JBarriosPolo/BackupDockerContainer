# 🧑‍🔧Creación de un servicio.

a continuación se muestra el código de un servicio que se encarga de ejecutar el script de backup antes mencionado.

~~~bash
    [Unit]
    Description=Docker Backup Service

    [Service]
    Type=simple
    ExecStart=/home/deploy/test.sh
    Restart=on-failure
    RestartSec=5s

    [Install]
    WantedBy=multi-user.target
~~~

** ⚠️<span style="color:#EBC004"> Advertencia </span>⚠️ ** 
> El servicio debe ser creado conjunto al siguiente archivo de timer para que se ejecute cada 24 horas.

## 🧑‍🔧 archivo de timer
~~~bash
    [Unit]
    Description=Run docker_backup.service every day at 1:00 AM

    [Timer]
    OnCalendar=*-*-* 01:00:00

    [Install]
    WantedBy=timers.target
~~~
