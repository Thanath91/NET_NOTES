
```insta-toc
---
title:
  name: Table of Contents
  level: 1
  center: false
exclude: ""
style:
  listType: dash
omit: []
levels:
  min: 1
  max: 6
---

# Table of Contents

- R-AJAX (Mikrotik HEX RB750 GR3)
    - 1. CONEXIÓN POR PRIMERA VEZ
        - NOTA: RESETEAR CONFIGURACIÓN
    - 2. QUITAR INTERFACES DEL BRIDGE QUE VIENE POR DEFECTO
    - 3. AÑADIR IP A INTERFACES
        - NOTA: CREAR UN DHCP SERVER EN UNA INTERFAZ
    - 4. HABILITAR ACCESO SSH POR LAS INTERFACES
```
  
![[Configuración inicial Mikrotik 750-1755285723520.jpeg]]

---
## R-AJAX (Mikrotik HEX RB750 GR3)

### 1. CONEXIÓN POR PRIMERA VEZ

Para acceder por primera vez, nos conectamos al puerto Ether2 desde nuestro PC-ADMIN.

Abrimos Winbox y buscamos vecinos Mikrotik por la dirección MAC:
![[LAB_AJAX-1755258889691.jpeg]]

Nos reconocerá la  MAC de nuestro dispositivo. Pinchamos en el device con su MAC, rellenamos las credenciales (por defecto el user es `admin` sin contraseña) y nos conectamos:
![[LAB_AJAX-1755259017675.jpeg]]

Si no hay DHCP habilitado y una IP local en el mikrotik a la que acceder, nos abrirá la sesión por la MAC, usando las credenciales existentes:
![[Configuración inicial Mikrotik 750-1755291352057.jpeg]]



En otro caso, si existe una configuración previa (la que viene por defecto), la IP local del router es 192.168.88.1. Así que si nos ponemos en nuestro PC una IP en esa red, nos abrirá la sesión por IP:
![[LAB_AJAX-1755259055379.jpeg]]



---
#### NOTA: RESETEAR CONFIGURACIÓN

Podemos hacer un `Reset Configuration` porque aquí han tocado cosas. Vamos a System\Reset Configuration
![[LAB_AJAX-1755292031009.jpeg]]

Pinchamos en la opción, si queremos, de mantener los usuarios. Reiniciará el dispositivo.


---
---
### 2. QUITAR INTERFACES DEL BRIDGE QUE VIENE POR DEFECTO

Vamos a Quic Set para ver la configuración rápida que viene por defecto en los puertos:
![[LAB_AJAX-1755259145361.jpeg]]
Podemos quitar el NAT.

Donde pone Internet, es el puerto Ether1, el resto de puerto darán DHCP de la Local Network 192.168.88.0/24


Tendremos que configurar la interfaz de administración que conecte con el switch. Elegiremos la interfaz Ether5 y la IP 10.80.70.2/30, que conectara con la interfaz fa0/7 del switch MLS.

Pero primero, tenemos que deshacer el `bridge` por defecto que vienen en el router, en que están añadidos todos los puertos Ether2-5, lo que hace que se comporten como puertos de Switch en una LAN e impide que se comporten como puertos Gigabit individuales (redes distintas).

Para ello, lo primero que hacemos al conectarnos por Winbox es ir al apartado `Bridge`:
![[LAB_AJAX-1755266719804.jpeg]]

seleccionar cada puerto individualmente y darle a `Remove`:
![[LAB_AJAX-1755260013157.jpeg]]

hasta que quede así (dejaremos el 2 tal y como está por seguridad):
![[LAB_AJAX-1755266824196.jpeg]]



---
### 3. AÑADIR IP A INTERFACES

Estamos conectados al puerto fa0/7 a través del Ether5, en la red 10.80.70.0/24, por tanto:

```
/ip address
add address=10.80.70.2/24 interface=ether5
```

![[LAB_AJAX-1755259685543.jpeg]]


Pero también se puede hacer por Winbox, en el apartado `IP Addresses`:
![[LAB_AJAX-1755266887692.jpeg]]

Añadimos una dirección:
![[LAB_AJAX-1755266910701.jpeg]]




Interfaz Ether3 que conecta con MPLS_AJAX:
![[LAB_AJAX-1755265140912.jpeg]]

Interfaz Ether4 que conecta con la red del cliente AJAX:
![[LAB_AJAX-1755265520079.jpeg]]

![[LAB_AJAX-1755265878185.jpeg]]



Ruta para llegar a las red del PC-ADMIN:
![[LAB_AJAX-1755265797821.jpeg]]



---

#### NOTA: CREAR UN DHCP SERVER EN UNA INTERFAZ
podemos crear un DHCP Server en la interfaz que necesitemos. Para ello, primero creamos un pool de direcciones de la red de Ether4:
![[LAB_AJAX-1755293209301.jpeg]]

Ya que estamos crearemos un pool para Ether5, la interfaz de gestión:
![[LAB_AJAX-1755293288279.jpeg]]




Ahora vamos a IP\DHCP Server y creamos uno en la Ether4:
![[LAB_AJAX-1755292958456.jpeg]]

![[LAB_AJAX-1755293432026.jpeg]]

Una vez creado, para habilitarlo, seleccionamos el nuevo server y le damos al tick azul (Enable):
![[LAB_AJAX-1755293507915.jpeg]]
De momento, estará en rojo porque no tenemos nada conectado.


Por último, en la pestaña `Networks`, añadimos la red que se difundirá por Ether5:
![[Configuración inicial Mikrotik 750-1755293800007.jpeg]]


---
---
### 4. HABILITAR ACCESO SSH POR LAS INTERFACES


Para permitir, de momento, el acceso SSH, telnet y Winbox por cualquier interfaz, tendremos que ir a IP Services, entrar en los servicios deseados y en el campo `Available From` poner la IP `0.0.0.0/0`:

![[LAB_AJAX-1755266006447.jpeg]]
Después en Apply y OK.


De manera que quedará así:
![[LAB_AJAX-1755266097221.jpeg]]


También, por conveniencia, añadiremos la pass `admin`:
![[LAB_AJAX-1755266261302.jpeg]]


Además nos aseguraremos que el usuario `admin` sea accesible desde cualquier red. Vamos a System\Users:
![[LAB_AJAX-1755266314247.jpeg]]
Le damos a Apply y Ok.







Tendremos que añadir la interfaz Ether5 a la lista LAN para permitir el tráfico de entrada por esta interfaz misma, ya que si vemos las reglas del firewall que viene por defecto, descartará (DROP) cualquier tráfico que no venga desde las interfaces que estén en la lista LAN. Para ver las reglas, usamos el comando:
```
/ip firewall print
```
o bien en IP Firewall de Winbox:
![[LAB_AJAX-1755284066828.jpeg]]

La regla 4 es la que hace que el tráfico se dropee. Vamos al partado `Interfaces` de la izquierda, vamos a la pestaña  `Innterface List` y añadimos las interfaces por las que queramos que haya tráfico, que serán Ether3-5:
![[LAB_AJAX-1755284138595.jpeg]]

![[LAB_AJAX-1755284208551.jpeg]]

![[LAB_AJAX-1755284243690.jpeg]]





Si ahora intentamos comunicarnos por SSH, usando Teraterm, por ejemplo:
![[LAB_AJAX-1755284295102.jpeg]]

![[LAB_AJAX-1755283860531.jpeg]]

![[LAB_AJAX-1755284673768.jpeg]]

![[LAB_AJAX-1755284699371.jpeg]]




Por SecureCRT:

![[LAB_AJAX-1755284360895.jpeg]]

![[LAB_AJAX-1755284382490.jpeg]]

![[LAB_AJAX-1755284399551.jpeg]]



Además, también podremos conectarnos por Winbox si nos resulta más cómodo gestionar por GUI:
![[LAB_AJAX-1755284456472.jpeg]]




En prinicipio, ya estaría el router preparado para su administración remota. Es decir, ya no tenemos que estar directamente conectados al router, sino que podemos conectarnos al switch de administración y llegar al mikrotik sin problemas.

Más adelante se configurarán más cosas para el laboratorio AJAX, como una ruta estática por defecto.