




Para ver que interfaces están en modo switchport:
```
show interfaces switchport
```



---

# CONFIGURAR IP ADDRESS EN INTERFAZ L2

necesito configurar las interfaces gi0/1/0 y 0/1/2 con las IP respectivas: 10.32.32.46/28 conecta con un Fortigate F81E en el puerto port4 (mpls_transit) con IP 10.32.32.33/28 20.20.10.2/30 conecta con un F81E en el puerto port4 (ps_transit) con IP 20.20.10.1/30

###### ChatGPT Plus

Pensó durante 20s

En el **C1111-8P** los puertos **Gi0/1/x** son **L2 (switch)**, así que no aceptan `ip address` directamente. La forma correcta es dar IP a una **SVI (interface VlanX)** y poner cada puerto **en acceso** a su VLAN (tráfico untagged). Abajo te dejo el paso a paso para tus dos enlaces.

---

## Objetivo

- **Gi0/1/0** ↔ FortiGate port4 (**mpls_transit**)  
    Red `10.32.32.32/28` → IP del Cisco: **10.32.32.46/28** (FGT = 10.32.32.33)
    
- **Gi0/1/2** ↔ FortiGate port4 (**ps_transit**)  
    Red `20.20.10.0/30` → IP del Cisco: **20.20.10.2/30** (FGT = 20.20.10.1)
    

---

## Configuración recomendada (con SVI)

```C
enable
configure terminal

! ===== Enlace MPLS_TRANSIT por Gi0/1/0 =====
vlan 100
 name MPLS_TRANSIT
exit

interface Vlan100
 description MPLS_TRANSIT to FGT
 ip address 10.32.32.46 255.255.255.240
 no shutdown
exit

interface GigabitEthernet0/1/0
 description to FGT port4 (mpls_transit)
 switchport mode access
 switchport access vlan 100
 spanning-tree portfast
 no shutdown
exit

! ===== Enlace PS_TRANSIT por Gi0/1/2 =====
vlan 200
 name PS_TRANSIT
exit

interface Vlan200
 description PS_TRANSIT to FGT
 ip address 20.20.10.2 255.255.255.252
 no shutdown
exit

interface GigabitEthernet0/1/2
 description to FGT port4 (ps_transit)
 switchport mode access
 switchport access vlan 200
 spanning-tree portfast
 no shutdown
exit

end
write memory

```

### Verificación

```C
show ip interface brief
show vlan brief
show interfaces switchport | include GigabitEthernet0/1/0|0/1/2

! Pings con origen en cada SVI
ping 10.32.32.33 source 10.32.32.46
ping 20.20.10.1  source 20.20.10.2

```

> Así queda cada puerto como **acceso L2** a su VLAN y la **IP de capa 3** vive en la **SVI** correspondiente. Funciona perfecto para enlaces punto a punto con FortiGate a través de un puerto L2.

---

## (Opcional) Método alternativo si tu IOS permite **routed ports**

En algunos builds de IOS XE puedes convertir ciertos puertos a L3 con `no switchport`. Si el comando es **aceptado** en tus `Gi0/1/x`, podrías hacerlo así (más “clásico”):

```C
conf t
interface GigabitEthernet0/1/0
 description to FGT port4 (mpls_transit)
 no switchport
 ip address 10.32.32.46 255.255.255.240
 no shutdown
exit

interface GigabitEthernet0/1/2
 description to FGT port4 (ps_transit)
 no switchport
 ip address 20.20.10.2 255.255.255.252
 no shutdown
exit
end
wr

```

> Si el equipo **rechaza** `no switchport`, vuelve al método **SVI** (el estándar en C1111-8P).

---

## Notas rápidas

- En el FortiGate, deja esas interfaces **sin tagging** (IP directa) para que casen con los puertos **access** del Cisco.
    
- No necesitas rutas para esas dos redes (son **conectadas**). Añade estáticas solo si quieres alcanzar **otras** redes detrás del FortiGate.
    
- Si luego vas a usar NAT o rutas dinámicas (OSPF/BGP), te ayudo a integrarlo.