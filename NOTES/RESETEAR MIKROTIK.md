
Apagamos el mikortik

Pulsamos y mantenemos pulsado el botón reset y devolvemos la corriente.

Mantenemos durante 5-10 segundos hasta que parpadee una luz verde y soltamos.


Después del pitido, en Winbox, ya podemos a darle a `Refresh` y nos saldrá la MAC del dispoitivo:
![[RESETEAR MIKROTIK-1755334271057.jpeg]]
Como se puede ver, esta sesión no es con IP, ya que se ve que hay una 0.0.0.0. La sesión se establecerá con la MAC.

Una vez conectados, nos saltará un mensaje con la `Default Configuration`
![[RESETEAR MIKROTIK-1755334386594.jpeg]]



![[RESETEAR MIKROTIK-1755334505070.jpeg]]

Podemos añadir una contraseña: admin/admin


La configuración rápida que esta puesta por defecto:
![[RESETEAR MIKROTIK-1755334609533.jpeg]]


Si vamos a nuestro PC y consultamos el `ipconfig` veremos que el DHCP le ha dado una IP al PC:
![[RESETEAR MIKROTIK-1755334693876.jpeg]]

Coincide con el tráfico de la IP que estamos viendo en Winbox:
![[RESETEAR MIKROTIK-1755334726273.jpeg]]



---

Reseteamos y nos da IP:
![[RESETEAR MIKROTIK-1755336599162.jpeg]]
