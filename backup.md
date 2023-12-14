# contenido base del codigo para el respaldo de los contenedores de docker.

~~~bash
    #!/bin/bash

    # Creación de las carpetas para almacenar los backups y los logs
    mkdir -p /mnt/sstorage/logs
    mkdir -p /mnt/sstorage/backups

    # Establecer una ruta para almacenar los backups
    backups_path="/mnt/sstorage/backups/"

    # Establecer una ruta para almacenar los logs
    logs_path="/mnt/sstorage/logs/"

    # Ejecución de comando para listar los Docker containers
    docker ps -a --format "{{.Names}}" > $logs_path"containers.txt"

    # Leer el archivo de texto con los nombres de los containers
    while read -r line
    do
        # Iterar sobre cada línea del archivo de texto agregando el comando docker export para exportar el container agregándole la fecha y hora de creación
        docker export $line > $backups_path$line-$(date +%Y-%m-%d-%H-%M).tar
        
        # Almacenaje de los logs por contenedor
        docker logs --tail=1000  $line > $logs_path$line-$(date +%Y-%m-%d-%H-%M).log
    done < $logs_path"containers.txt"

    # Función para obtener el número de la semana del año de un archivo
    get_week_number() {
        echo $(date -d "$(stat -c %Y "$1")" +%U)
    }

    # Comprobar la existencia de los backups en la ruta establecida
    if [ -d "$backups_path" ]; then
        cd $backups_path
        for container in $(docker ps -a --format "{{.Names}}"); do
            # Obtener los archivos de backup para el contenedor actual, ordenados por fecha de modificación
            backup_files=($(ls -t $backups_path$container-*.tar))
            total_backups=${#backup_files[@]}
            
            if (( total_backups > 7 )); then
                # Renombrar el séptimo archivo de backup más reciente y agregar la semana del año
                week_number=$(get_week_number "${backup_files[6]}")
                mv "${backup_files[6]}" "${backup_files[6]%.tar}-week$week_number.tar"
                # Eliminar los archivos de backup más antiguos que no están entre los 7 más recientes
                for (( i=7; i<total_backups; i++ )); do
                    rm "${backup_files[i]}"
                done
            fi
            
            # Verificar la existencia de archivos con la terminación "week" y el mismo número de semana
            week_files=($(ls $backups_path$container-*week$week_number.tar 2>/dev/null))
            total_week_files=${#week_files[@]}
            if (( total_week_files == 3 )); then
                # Tomar el último archivo y renombrarlo con la terminación del mes
                mv "${week_files[2]}" "${week_files[2]/week$week_number/month$(date +%m).tar}"
                # Eliminar los otros dos archivos de la semana
                rm "${week_files[0]}" "${week_files[1]}"
            fi
        done
    fi
~~~