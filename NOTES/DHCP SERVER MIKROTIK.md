

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

- POR CLI
    - 1️⃣ Asignar una IP a la interfaz ether4
    - 2️⃣ Crear el pool de direcciones
    - 3️⃣ Crear el DHCP Server
    - 4️⃣ Configurar la red DHCP
    - 5️⃣ Habilitar el servidor
- POR WINBOX (GUI)
```

---
---

# POR CLI

## **1️⃣ Asignar una IP a la interfaz ether4**

Antes de poder servir direcciones DHCP, ether4 debe tener su **IP de gateway** configurada:

```
/ip address 
add address=10.80.70.1/24 interface=ether4
```

_(Ajusta la red y máscara a lo que necesites)_

---

## **2️⃣ Crear el pool de direcciones**

Este es el rango que el DHCP entregará a los clientes:

```
/ip pool 
add name=pool-ether4 ranges=10.80.70.10-10.80.70.100
```

_(Ajusta el rango a tu preferencia)_

---

## **3️⃣ Crear el DHCP Server**

En CLI sería:
```

/ip dhcp-server 
add name=DHCPserver_ether4 interface=ether4 address-pool=pool-ether4 lease-time=1d authoritative=yes
```

En WinBox (lo que muestras en la captura):

- **Interface** → `ether4`
    
- **Address Pool** → selecciona `pool-ether4` (o el que creaste).
    
- **Lease Time** → `1d` o el tiempo que quieras.
    
- **Authoritative** → `yes`.
    
- **Bootp Support** → `static` (o `none` si no usas BOOTP).
    

---

## **4️⃣ Configurar la red DHCP**

Debes indicarle al servidor DHCP cuál es la red, el gateway y el DNS:

```
/ip dhcp-server network 
add address=10.80.70.0/24 gateway=10.80.70.1 dns-server=8.8.8.8,1.1.1.1
```

En WinBox esto está en: **IP → DHCP Server → pestaña "Networks"**.

---

## **5️⃣ Habilitar el servidor**

En CLI:

```
/ip dhcp-server enable DHCPserver_ether4
```

En WinBox, asegúrate de que la casilla **enabled** esté marcada (en tu captura lo está).

---

✅ Con esto, cualquier equipo conectado a **ether4** recibirá automáticamente IP, gateway y DNS según la configuración que pusiste.



---
---

# POR WINBOX (GUI)

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

Nota: en DNS Server, hemos añadido la IP del DNS a la que tienen que atacar los dispositivos de esta red.
