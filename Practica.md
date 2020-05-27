# Creación de volúmenes perisistentes en Kubernetes
Se van a crear dos voúmenes persistentes con sus respectivas llamadas. 

En primer lugar se necesita cierta configuración en el nodo master y los minion para crear volúmenes persistentes.


## En el nodo master
Se necesitan instalar los siguientes paqueres: nfs-common nfs-kernel-server
~~~
sudo apt install nfs-common nfs-kernel-server
~~~
*
*
*
*
*
***********sigo por aquí, ya el master hecho, pero lo tengo que explicar. y hacer los nodos y explicarlos
~~~
debian@kubemaster:~$ sudo apt install nfs-common nfs-kernel-server
root@kubemaster:/home/debian# mkdir -p /home/shared/volumen7 /home/shared/volumen8
root@kubemaster:/home/debian# echo "/home/shared/volumen7 *(rw,sync,no_root_squash,no_all_squash)" >> /etc/exports
root@kubemaster:/home/debian# echo "/home/shared/volumen8 *(rw,sync,no_root_squash,no_all_squash)" >> /etc/exports
root@kubemaster:/home/debian# rm /lib/systemd/system/nfs-common.service
root@kubemaster:/home/debian# systemctl daemon-reload
root@kubemaster:/home/debian# chown -R systemd-coredump:root /home/shared
root@kubemaster:/home/debian# systemctl restart nfs-kernel-server
~~~

En los nodos:
~~~
debian@kubeminion1:~$ sudo apt install -y nfs-common
root@kubeminion1:/home/debian# rm /lib/systemd/system/nfs-common.service
root@kubeminion1:/home/debian# systemctl daemon-reload
root@kubeminion1:/home/debian# mkdir -p /var/data/volumen5 /var/data/volumen6
root@kubeminion1:/home/debian# mount -t nfs4 10.0.0.3:/home/shared/volumen5 /var/data/volumen5
root@kubeminion1:/home/debian# mount -t nfs4 10.0.0.3:/home/shared/volumen6 /var/data/volumen6
~~~