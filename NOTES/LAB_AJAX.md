

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

- CREDENCIALES
- FASE 1: CONFIGURACIONES BÁSICAS DE CONECTIVIDAD EN RED MGMT
    - PC-ADMIN
    - MLS: SWITCH ADMINISTRACIÓN
        - HABILITAR SSH
        - INTERFACES
        - RUTAS
    - FW-PS (F81-A)
    - FW-AJAX (F81-B)
    - R-AWS (Mikrotik HEX RB750 GR3)
    - R-AJAX (Mikrotik HEX RB750 GR3)
        - HABILITAR ACCESO SSH
    - R-PS (Cisco ISR C1111 8P)
- FASE 2: CONFIGURACIONES AVANZADAS
    - FW-PS (F81-A 10.80.10.2)
        - RUTAS ESTÁTICAS
        - Políticas de flujo entre interfaces
        - DNS
    - FW-AJAX (F81-B 10.80.30.2)
        - RUTAS ESTÁTICAS
        - Políticas de flujo entre interfaces
    - R-AWS (Mikrotik Norte)
    - R-AJAX (Mikrotik Sur)
    - R-PS (Cisco ISR C1111)
- VERIFICACIÓN CONECTIVIDAD TOTAL
- FASE 3: DNS
    - FW-PS
```


---
---

# CREDENCIALES

Forti: admin/admin


Switch MLS / Router R-PS:
local database user:
```
netadmin/netadmin
```

console pass:
```
ciscoconpa55
```
vty/ssh local pass:
```
ciscovtypass55
```
enable pass:
```
ciscoenpa55
```



---
---
# FASE 1: CONFIGURACIONES BÁSICAS DE CONECTIVIDAD EN RED MGMT

El objetivo será llegar a todos los dispositivos de red desde PC-ADMIN, y una vez así, configurar todo lo necesario para simular el laboratorio AJAX.

![[LAB_AJAX-1755285746945.jpeg]]

## PC-ADMIN

Podemos seleccionar la casilla de obtener DHCP automáticamente.

Al final, debe estar conectado al fa0/8 del switch de gestión MLS, que brindará DHCP en el rango 10.80.0.0/24

![[LAB_AJAX-1755257237280.jpeg]]

![[LAB_AJAX-1755257253131.jpeg]]



Vamos a añadir una ruta estática para que no haya conflictos con el WIFI del PC. Vamos a añadir una ruta con destino la red 10.80.0.0/16 a través del gateway 10.80.0.1, de esta manera, para llegar a cualquier subred de gestión del laboratorio saldrá por este gateway.

Ejecutamos CMD como administrador y escribimos el siguiente comando:
```
route add 10.80.0.0 mask 255.255.0.0 10.80.0.1
```
La ruta NO ES PERMANENTE, se borrará tras el reinicio del PC. De momento la añadiremos con cada reinicio.
![[LAB_AJAX-1755342518527.jpeg]]


Para ver las rutas existentes en Windows:
```
route print
```
![[LAB_AJAX-1755342547385.jpeg]]



---
---
## MLS: SWITCH ADMINISTRACIÓN

Se ha accedido por consola la primera vez, para configurar el direccionamiento de las interfaces. Se mantendrá esta conexión por consola como conexión auxiliar, aunque una vez desplegadas las redes de mantenimiento, no debería hacer falta.


### HABILITAR SSH


```
conf t
hostname MLS
ip domain-name mgmt.net
crypto key generate rsa modulus 2048
username netadmin privilege 15 secret netadmin

ip ssh version 2

line vty 0 15
transport input ssh
exec-timeout 60 0
logging synchronous
login local
exit
do wr

```

![[LAB_AJAX-1755268048103.jpeg]]



El comando `logging synchronous` evita que los mensajes del sistema (syslog) interrumpan lo que estás escribiendo en la línea de comandos. Por ejemplo, si llega un mensaje de enlace subido/bajado mientras escribes, el prompt se reescribe ordenadamente en la siguiente línea, en lugar de cortarte el texto a medias.


Ya que estamos podemos crear una contraseña de enable:
```
enable secret ciscoenpa55
```

y modificar el acceso por consola:
```
line con 0
password ciscoconpa55
exec-timeout 60 0
logging synchronous
login local
exit
do wr
```

---

### INTERFACES

Comando global para permitir enrutamiento:
```
ip routing
```

Para cada interfaz configuraremos:
```
int fa0/1
des MGMT_FW-PS_F81A
no switchport
ip add 10.80.10.1 255.255.255.0
no shut
exit

```

```
int fa0/2
des MGMT_SW-AWS
no switchport
ip add 10.80.20.1 255.255.255.0
no shut
exit

```

```
int fa0/3
des MGMT_FW-AJAX_F81B
no switchport
ip add 10.80.30.1 255.255.255.0
no shut
exit

```

```
int fa0/4
des MGMT_SW-CLIENT
no switchport
ip add 10.80.40.1 255.255.255.0
no shut
exit

```

```
int fa0/5
des MGMT_R-AWS
no switchport
ip add 10.80.50.1 255.255.255.0
no shut
exit

```

```
int fa0/6
des MGMT_R-PS
no switchport
ip add 10.80.60.1 255.255.255.0
no shut
exit

```

```
int fa0/7
des MGMT_R-AJAX
no switchport
ip add 10.80.70.1 255.255.255.0
no shut
exit

```

```
int fa0/8
des MGMT_MAIN
no switchport
ip add 10.80.0.1 255.255.255.0
no shut
exit
```

![[LAB_AJAX-1755254159473.jpeg]]

En el puerto fa0/8 tendremos conectado el PC-ADMIN

---
### RUTAS

```
ip routing
```

Como tal, solo necesitaremos comunicación con los dispositivos conectados directamente al switch, en principio no se necesitan rutas. Además, no necesitamos que estos dispositivos se comuniquen entre sí.

---

DHCP para fa0/8

```
ip dhcp excluded-address 10.80.0.1 10.80.0.50
ip dhcp pool MGMT_MAIN
network 10.80.0.0 255.255.255.0
default-router 10.80.0.1
exit

service dhcp
do wr

```

Verificar clientes conectados:
```
show ip dhcp binding
```

Ver configuración del servidor:
```
show running-config | include dhcp
```


---
---
## FW-PS (F81-A)

Para entrar por primera vez antes de tener la red de administración montada, podemos conectarnos por el port1 que brinda DHCP de la red 192.168.1.0/24 y el Forti tiene IP 192.168.1.99. O bien por consola.


Interfaz de MGMT:
![[LAB_AJAX-1755252201770.jpeg|-7]]

![[LAB_AJAX-1755252152891.jpeg]]

Creamos algunos objetos de red:
![[LAB_AJAX-1755254726742.jpeg]]

![[LAB_AJAX-1755254878054.jpeg]]



Configuraremos el port4 como `mpls_transit`:
![[LAB_AJAX-1755256518130.jpeg]]


Y el port5 como `aws_transit`:
![[LAB_AJAX-1755256594451.jpeg]]

---

## FW-AJAX (F81-B)

Para entrar por primera vez antes de tener la red de administración montada, podemos conectarnos por el port1 que brinda DHCP de la red 192.168.1.0/24 y el Forti tiene IP 192.168.1.99. O bien por consola.


Objetos de red:
![[LAB_AJAX-1755257477674.jpeg]]

![[LAB_AJAX-1755254878054.jpeg]]




Interfaz MGMT:
![[LAB_AJAX-1755257647196.jpeg]]


Ruta estática a red de PC-ADMIN:
![[LAB_AJAX-1755257783449.jpeg]]


Interfaz port4 como `ps_transit`:
![[LAB_AJAX-1755257898645.jpeg]]


Interfaz port5 como `client_transit`:
![[LAB_AJAX-1755257943954.jpeg]]



---

## R-AWS (Mikrotik HEX RB750 GR3)

Para acceder por primera vez, nos conectamos al puerto Ether2.

Tendremos que configurar la interfaz de administración que conecte con el switch. Elegiremos la interfaz Ether5 y la IP 10.80.50.2/30, que conectara con la interfaz fa0/5 del switch MLS.

Este router tien una version de RouterOS superior a la del otro HEX RB750 Gr3:
![[LAB_AJAX-1755291600520.jpeg]]

Cambia ligeramente algunas cosas, por ejemplo, ahora hay un apartado dedicado a SSH.

De hecho en este router no venía nada, no habia IP local, solo usuario admin/admin. Hemos creado una IP local por si perdemos acceso:
![[LAB_AJAX-1755291819671.jpeg]]


Tampoco hay reglas en el Firewall. Vamos a hacer un `Reset Configuration` porque aquí han tocado cosas. Vamos a System\Reset Configuration
![[LAB_AJAX-1755292031009.jpeg]]

Pinchamos en la opción de mantener los usuarios. Solo hay uno: admin/admin. Reiniciará el dispositivo. Una vez reseteado, ahora sí vemos que existe un bridge por defecto:
![[LAB_AJAX-1755292338782.jpeg]]


Y sí hay una IP local:
![[LAB_AJAX-1755292136373.jpeg]]
Quitamos el NAT.


También hay reglas de Firewall:
![[LAB_AJAX-1755292409074.jpeg]]



Hacemos como en el otro router, solo dejamos el puerto Ether2 en el bridge:


Quitamos las interfaces Ether3-5 del bridge para que puedan comportarse como puertos individuales:
![[LAB_AJAX-1755292620441.jpeg]]

![[LAB_AJAX-1755292643564.jpeg]]




Añadimos los puertos Ether3-5 a la lista LAN:
![[LAB_AJAX-1755292499506.jpeg]]


Configuramos las direcciones IP de las interfaces que se usarán en el laboratorio:
![[LAB_AJAX-1755292715491.jpeg]]

![[LAB_AJAX-1755292916498.jpeg]]


También podemos crear un DHCP Server en la interfaz que necesitemos. Para ello, primero creamos un pool de direcciones de la red de Ether4:
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

Nota: en DNS Server, hemos añadido la IP del DNS a la que tienen que atacar los dispositivos de esta red.



Añadimos una ruta estática para alcanzar PC-ADMIN a través de la Ether5:



Veamos si el SSH esta activo:
![[LAB_AJAX-1755340690371.jpeg]]


Regla de Firewall
Por defecto, hay una regla que va abloquear el tráfico entrante si este no proviene de una de las interfaces dentro de la lista de interfaces `LAN`.
![[LAB_AJAX-1755340798038.jpeg]]


Lo que podemos hacer es cambiar la `LAN` por un `all` y permitir el tráfico entrante desde cualquier red. Más adelante puede ser modificado.
![[LAB_AJAX-1755340919543.jpeg]]

![[LAB_AJAX-1755340940321.jpeg]]






---

## R-AJAX (Mikrotik HEX RB750 GR3)

Para acceder por primera vez, nos conectamos al puerto Ether2 desde nuestro PC-ADMIN.

Abrimos Winbox y buscamos vecinos Mikrotik por la dirección MAC:
![[LAB_AJAX-1755258889691.jpeg]]

Nos reconocerá la  MAC de nuestro dispositivo. Pinchamos en el device con su MAC, rellenamos las credenciales (por defecto el user es `admin` sin contraseña) y nos conectamos:
![[LAB_AJAX-1755259017675.jpeg]]

![[LAB_AJAX-1755259055379.jpeg]]


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


#### HABILITAR ACCESO SSH


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

---
---

## R-PS (Cisco ISR C1111 8P)

Ponemos una  contraseña para ENABLE y ciframos la vista de las contraseñas en el sh run:
```
enable secret ciscoenpa55
service password-encryption
```

regulamos el acceso por consola:
```
line con 0
password ciscoconpa55
exec-timeout 60 0
logging synchronous
login local
exit
do wr
```


Habilitamos acceso SSHv2:
```
conf t
hostname R-PS
ip domain name mgmt.net
username netadmin privilege 15 secret netadmin
crypto key generate rsa modulus 2048
ip ssh version 2

line vty 0 15
transport input ssh
exec-timeout 60 0
logging synchronous
login local
exit
do wr
```

Una ruta para llegar a PC-ADMIN:
```
ip route 10.80.0.0 255.255.255.0 10.80.60.1
```

Una ip para la administración remota:
```
vlan 99
name MGMT

int vlan99
des Ip_for_MGMT
ip add 10.80.60.2 255.255.255.0
no shut
end
wr

```


---
---
---

# FASE 2: CONFIGURACIONES AVANZADAS





---

## FW-PS (F81-A 10.80.10.2)

### RUTAS ESTÁTICAS

Añadimos una ruta estática para alcanzar la red de AWS CLOUD (10.198.3.0) y una ruta estática por defecto para reenviar el resto de tráfico por el port4:
![[LAB_AJAX-1755384739476.jpeg]]

---

### Políticas de flujo entre interfaces

Necesitaremos crear una política que permita el flujo entre las interfaces port4 y port5, que de momento permita todo. Hay que crear una regla de ida y otra de vuelta:

![[LAB_AJAX-1755401114724.jpeg]]

![[LAB_AJAX-1755401163625.jpeg]]

![[LAB_AJAX-1755401223329.jpeg]]


---

### DNS

![[LAB_AJAX-1755385104638.jpeg]]

---
---
## FW-AJAX (F81-B 10.80.30.2)

### RUTAS ESTÁTICAS

Debido a que este Firewall se encuentra en medio de todo, tendremos que crear varias rutas estáticas para alcanzar todas las redes.

3 rutas para alcanzar las redes de FW-PS:
![[LAB_AJAX-1755402767172.jpeg]]

1 Ruta para alcanzar la red del cliente AJAX:
![[LAB_AJAX-1755402834919.jpeg]]



---

### Políticas de flujo entre interfaces

Necesitaremos crear una política que permita el flujo entre las interfaces port4 y port5, que de momento permita todo. Hay que crear una regla de ida y otra de vuelta:

![[LAB_AJAX-1755402911216.jpeg]]

![[LAB_AJAX-1755402955190.jpeg]]


![[LAB_AJAX-1755402983250.jpeg]]








---

## R-AWS (Mikrotik Norte)

![[LAB_AJAX-1755401745230.jpeg]]

Vamos a añadir una ruta estática por defecto:
```
/ip/route/add dst-address=0.0.0.0/0 gateway=10.50.50.1 distance=1 check-gateway=ping

```

![[LAB_AJAX-1755401520781.jpeg]]

Para verificar:
```
/tool/ping 10.50.50.2
/tool/ping 10.32.32.46
```







---

## R-AJAX (Mikrotik Sur)

Versión inferior de RouterOS.

Vamos a añadir una ruta estática por defecto:

```
/ip route add dst-address=0.0.0.0/0 gateway=20.20.20.1 distance=1 check-gateway=ping
```
![[LAB_AJAX-1755402357249.jpeg]]

![[LAB_AJAX-1755402262382.jpeg]]


![[LAB_AJAX-1755402400618.jpeg]]


---
## R-PS (Cisco ISR C1111)

Vamos a poner las IP a los puertos que conectan con los Fortigate:

```
conf t

vlan 10
name MPLS_TRANSIT

int vlan10
des inside_PS
ip add 10.32.32.46 255.255.255.240
no shut
exit

int gi0/1/0
des Connect to FW-PS
swi mode access
swi acc vlan 10
spanning-tree portfast
no shut
end
wr

```

```
conf t

vlan 20
name PS_TRANSIT

int vlan20
des outside_PS
ip add 20.20.10.2 255.255.255.252
no shut
exit

int gi0/1/2
des Connect to FW-AJAX
swi mode access
swi acc vlan 20
spanning-tree portfast
no shut
end
wr
```

![[LAB_AJAX-1755400436429.jpeg]]

![[LAB_AJAX-1755400414574.jpeg]]


Rutas para alcanzar las redes detrás del FW-PS:
```
ip route 10.50.50.0 255.255.255.252 10.32.32.33
ip route 10.198.3.0 255.255.255.0 10.32.32.33 
```
![[LAB_AJAX-1755400658333.jpeg]]


Rutas para alcanzar las redes detrás del FW-AJAX:
```
ip route 20.20.20.0 255.255.255.252 20.20.10.1
ip route 172.100.100.0 255.255.255.0 20.20.10.1
```
![[LAB_AJAX-1755400829729.jpeg]]




---
---
---

# VERIFICACIÓN CONECTIVIDAD TOTAL

Router Norte: 7.11.2 (2023)
Router Sur: ver 6.43.4 (2018)


Desde router Norte R-AWS (10.198.3.1) a router Sur R-AJAX (172.100.100.1):
![[LAB_AJAX-1755403143903.jpeg]]


Desde router Sur R-AJAX (172.100.100.1) a router Norte R-AWS (10.198.3.1):
![[LAB_AJAX-1755403317305.jpeg]]

desde el Winbox:
![[LAB_AJAX-1755403423756.jpeg]]

Y si hacemos un traceroute, veremos los saltos que va haciendo por el camino:
![[LAB_AJAX-1755403478850.jpeg]]


---



---
---
---

# FASE 3: DNS


---

### FW-PS

Con esta configuración se activa el DNS FORWARDING:

![[LAB_AJAX-1755468172904.jpeg]]

![[LAB_AJAX-1755468185748.jpeg]]

![[LAB_AJAX-1755468200715.jpeg]]


---




```c
config firewall vip
    edit "VIP_PRI_AJAX"
        set extip 10.32.32.34
        set mappedip 10.198.3.36
        set extintf "port4"
    next
    edit "VIP_SEC_AJAX"
        set extip 10.32.32.34
        set mappedip 10.198.3.52
        set extintf "port4"
    next
end

```

![[LAB_AJAX-1755469025790.jpeg]]











---




---



---





---




---
