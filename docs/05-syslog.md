Syslog
Lograr que el NAS reciba todos los syslogs de los dispositivos finales que se necesita.
Lo primero que hice fue activar el llamemosle "colector" de syslog en el dispositivo NAS.
Primero creamos el archivo junto con la carpeta donde vamos a alojar la conf.
``sudo mkdir -p /var/log/remoto
sudo nano /etc/rsyslog.d/10-remoto.conf``
Luego dentro usamos esta conf:

``module(load="imudp")
input(type="imudp" port="514")
module(load="imtcp")
input(type="imtcp" port="514") 
template(name="PorEquipo" type="string" string="/var/log/remoto/%FROMHOST-IP%.log")
if ($fromhost-ip != "127.0.0.1") then { action(type="omfile" dynaFile="PorEquipo") stop}``

para activar el trafico de udp y tcp por el puerto 514, luego con el comando `sudo rsyslogd -N1` validamos y reseteamos el deamon.
despues con un sudo ufw allow vamos a hacer una rule para el firewall para que cualquiera de la red del local pueda transmitir el log hacia el NAS

despues vamos a crear un archivo en /etc/logrotate.d/remoto para hacer las reglas del logrotate.

``/var/log/remoto/*.log {
    weekly
    rotate 8
    compress
    missingok
    notifempty
}``

que lo que hace es que sea semanal, solo 8 logs a la vez, cada semana va a comprimir, con el missingok no nos tira error por no haber archivo y no rota archivos vacios.

con el comando `logger -n` probamos nuestro "colector"
