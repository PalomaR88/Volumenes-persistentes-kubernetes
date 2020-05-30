# Creación de volúmenes perisistentes en Kubernetes
Se van a crear dos voúmenes persistentes con sus respectivas llamadas. 

En primer lugar se necesita cierta configuración en el nodo master y los minion para crear volúmenes persistentes.


## En el nodo master
Se necesitan instalar los siguientes paqueres: **nfs-common** y  **nfs-kernel-server**.
~~~
sudo apt install nfs-common nfs-kernel-server
~~~

**Como root** se crea el directorio donde se alojará el volumen, con la opción **-p** para crear los directorios padre en el caso que falten:
~~~
mkdir -p /home/shared/<nombre_directorio>
~~~

Se añaden las siguientes líneas al direcotrio **/etc/exports**. Este fichero indica todos los directorios que un servidor exporta a los clientes.
~~~ 
echo "/home/shared/<nombre_directorio> *(rw,sync,no_root_squash,no_all_squash)" >> /etc/exports
~~~

Contra posibles conflictos se elimina el siguiente demonio de nfs:
~~~
rm /lib/systemd/system/nfs-common.service
~~~

Se carga la configuración del administrador de systemd:
~~~
systemctl daemon-reload
~~~

Se cambia el propietario de **/home/shared**:
~~~
chown -R systemd-coredump:root /home/shared
~~~

Y se reinicia el servicio:
~~~
systemctl restart nfs-kernel-server
~~~

## En los minions:
En los nodos que realizan la labor de minions en el cluster se debe instalar el paquete **nfs-common**:
~~~
sudo apt install -y nfs-common
~~~

**Como root** se realizan los siguientes pasos como en el master:
~~~
rm /lib/systemd/system/nfs-common.service
systemctl daemon-reload
mkdir -p /var/data/<nombre_directorio>
~~~

Por último, en el direcotrio creado anteriormente, se monta indicando el tipo, **-t**, nfs4 con la dirección IP del nodo master y la ruta del directorio:
~~~
mount -t nfs4 <IP_master>:/home/shared/<nombre_directorio> /var/data/<nombre_directorio>
~~~

## Creación del volúmen en Kubernetes:
En primer lugar se tiene que crear el objeto. Se va a crear a través de un fichero .yaml con la siguiente estructura:
~~~
apiVersion: v1
kind: PersistentVolume
metadata:
  name: <nombre_volumen>
spec:
  capacity:
    storage: <capacidad>Gi
  accessModes:
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Recycle
  nfs:
    path: <ruta_volúmen>
    server: <dirección_IP_master>
~~~

Un ejemplo:
~~~
apiVersion: v1
kind: PersistentVolume
metadata:
  name: volumen1
spec:
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Recycle
  nfs:
    path: /home/shared/volumen1
    server: 10.0.0.3
~~~

Para utilizarse hay que crear una solicitud de almacenamiento del volumen, con un fichero con la siguiente estructura:
~~~
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: <nombre_PVClaim>
  namespace: <nombre_namespace>
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: <capacidad>Gi
~~~


