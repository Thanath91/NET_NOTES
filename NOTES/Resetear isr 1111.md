
Adquirimos un Cisco ISR C1111 de 8P Giga de segunda mano, sin embargo no se preocuparon de entregárnoslo limpio:
![[isr 1111-1755387956276.jpeg]]

Después de 2 horas de intentos infructuosos, me encuentro con un foro de cisco en el que alguien tuvo el mismo problema:
https://community.cisco.com/t5/network-management/correct-procedure-for-default-factory-reset-of-cisco-c1111-4p/td-p/4394163/page/2

Este usuario es el que me da la idea de usar el `Ctrl+C` cuando el router se está iniciando para acceder al modo ROMMON:
![[Resetear isr 1111-1755393704663.jpeg]]


---

Para resetear el C1111 he usado el programa HyperTerminal
https://www.hilgraeve.com/hyperterminal-trial/
Con estos ajustes:
![[isr 1111-1755392729773.jpeg]]

![[isr 1111-1755392767656.jpeg]]

![[isr 1111-1755392786815.jpeg]]

![[isr 1111-1755392810987.jpeg]]

---

## ENTRAR AL MODO ROMMON

Apagamos el router, volvemos a encender y presionamos algunas veces `Ctrl+C`, y apareceremos en ROMMON con `rommon 1 >`

Una vez lleguemos aquí, para que al iniciar se ignore la configuración existente emitimos el comando:
```
confreg 0x2142
```
y tras este el siguiente para reiniciar el router:
```
reset
```
![[isr 1111-1755392904809.jpeg]]

Una vez reiniciado, veremos que ya no aparece ningún hostname y que podemos entrar al modo de configuración global:
![[isr 1111-1755393012291.jpeg]]


Para borrar la `startup-config` y los archivos VLAN, desde el modo EXEC Privilege (#) emitimos estos comandos, pulsando simplemente Enter si nos pide confirmación al borrar:
```
erase startup-config
```
```
delete vlan.dat
```

![[isr 1111-1755392558821.jpeg]]

Además, volveremos a dejar igual la variable que modificamos en el ROMMON para saltarnos la configuración, guardamos y reiniciamos:
```
conf t
config-register 0x2102
end
write memory
reload

```
![[isr 1111-1755392689258.jpeg]]

Tras reiniciar:
![[isr 1111-1755393012291.jpeg]]

Ya tenemos el router limpio, lo podemos comprobar haciendo un show de la configuración de inicio y ver que está completamente vacía:
```
Router#sh startup-config
Using 1284 out of 33554432 bytes
!
! Last configuration change at 00:55:57 UTC Sun Aug 17 2025
!
version 16.9
service timestamps debug datetime msec
service timestamps log datetime msec
platform qfp utilization monitor load 80
no platform punt-keepalive disable-kernel-core
!
hostname Router
!
boot-start-marker
boot-end-marker
!
!
!
no aaa new-model
!
!
!
!
login on-success log
!
!
!
!
!
!
!
subscriber templating
multilink bundle-name authenticated
!
!
!
!
!
license udi pid C1111-8P sn FCZ2532R24Z
license accept end user agreement
no license smart enable
!
diagnostic bootup level minimal
!
spanning-tree extend system-id
!
!
!
redundancy
 mode none
!
!
vlan internal allocation policy ascending
!
!
!
!
!
!
interface GigabitEthernet0/0/0
 no ip address
 negotiation auto
!
interface GigabitEthernet0/0/1
 no ip address
 negotiation auto
!
interface GigabitEthernet0/1/0
!
interface GigabitEthernet0/1/1
!
interface GigabitEthernet0/1/2
!
interface GigabitEthernet0/1/3
!
interface GigabitEthernet0/1/4
!
interface GigabitEthernet0/1/5
!
interface GigabitEthernet0/1/6
!
interface GigabitEthernet0/1/7
!
interface Vlan1
 no ip address
!
ip forward-protocol nd
no ip http server
ip http secure-server
!
!
!
!
!
!
control-plane
!
!
line con 0
 transport input none
 stopbits 1
line vty 0 4
 login
!
!
!
!
!
!
end
```


Tampoco tenemos IPs asignadas:
![[Resetear isr 1111-1755393475534.jpeg]]


Y la configuración VLAN está por defecto:
![[Resetear isr 1111-1755393514001.jpeg]]

---

### Comandos útiles en ROMMON (por si los necesitas)

- Ver el registro actual:
    
```
    rommon 1 > set
```
    
- Arrancar una imagen concreta manualmente (si hiciera falta):
    
```
    rommon 1 > dir bootflash: 
    rommon 2 > boot bootflash:c1100-universalk9_ias.<version>.SPA.bin
```
    

---
