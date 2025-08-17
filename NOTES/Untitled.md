

# PETICIÓN

ahora mismo tengo conectividad total de extremo a extremo, todavia no he creado una ACL en R-PS para permitir trafico de entrada solamente con destino 10.32.32.32/28. Antes quiero probar el DS sin restricciones. Necesito que me ayudes a:

-configurar DNS Forwarding en firewall FW-PS, que reenvié las peticiones DNS que llegan de la red 172.100.100.0/24 al server DNS con IP 10.198.3.69. En las imagenes que te paso tengo la configuracion DNS database que tengo en el Forti FW-PS. si tengo que hacer cambios indicamelo

-configurar un servidor DNS en windows en un PC con IP 10.198.3.69. Este sera el servidor DNS. Debe resolver las siguientes consultas:

pri.ajax.system --> 10.198.3.36

sec.ajax.system --> 10.198.3.52

-Cuando un PC de la red 172.100.100.0/24 haga una consulta de cualquiera de estos dominios reciba la IP correspondiente: ejemplo:

nslookup pri.ajax.system

output: 10.198.3.36

En esta red, los PC apuntan el DNS a la IP 10.32.32.33 que es una interfaz de FW-PS


---

Con esa topología ya viva, hagamos que **FW-PS (FortiGate)** actúe como **resolver** para los PCs de `172.100.100.0/24` y **reenvíe solo el dominio** `ajax.system` a tu **servidor DNS de Windows 10.198.3.69**. Luego configuramos el DNS de Windows y probamos.

---

## 1) FortiGate FW-PS – DNS forwarding “condicional” (domain override)

> Objetivo: los clientes preguntan a **10.32.32.33** (FW-PS).  
> Para **`*.ajax.system`** el FW-PS **reenvía** a **10.198.3.69**.  
> El resto de dominios, si quieres, a otros _forwarders_ (Google/Cloudflare) o lo que tengas.

### CLI (rápido y seguro)

```bash
# 1) Habilitar el DNS proxy (el FGT responderá a los clientes)
config system dns
    set dnsproxy enable
    set primary 1.1.1.1            # opcional para “otros” dominios
    set secondary 8.8.8.8
end

# 2) Activar el servicio DNS en la interfaz de los clientes (10.32.32.33)
config system dns-server
    edit "port4"                    # <-- usa el nombre real de la interfaz 10.32.32.33
        set mode recursive          # el FGT resuelve/forwardea
    next
end

# 3) Override para el dominio ajax.system (forward condicional)
config system dns-database
    edit "AJAX_FORWARD"
        set domain "ajax.system"
        set type forwarder          # si tu versión no muestra 'forwarder', usa 'set authoritative disable' + 'set view public/proxy'
        set dns-server 10.198.3.69  # servidor de Windows
        set ttl 300
    next
end

# (Solo si usas “Local-In Policy” estricta) permite DNS hacia el propio FGT:
config firewall local-in-policy
    edit 0
        set intf "port4"
        set srcaddr "all"
        set dstaddr "all"
        set service "DNS"
        set action accept
    next
end

```

### Comentarios sobre tus capturas

- En tu **DNS Zone** actual veo `Domain Name: forti.ps` y **DNS Forwarder: 10.198.3.69**.  
    Para tu caso, **cámbialo** a **`ajax.system`** (esa es la zona que quieres reenviar). No hace falta crear “DNS Entries” dentro: al ser _forwarder/Proxy_ el FGT **solo reenvía**.
    
- En **DNS Settings** puedes dejar 1.1.1.1 / 8.8.8.8 como _forwarders generales_ (marcan “Unreachable” si no tienes política de salida a Internet desde el propio FGT; no pasa nada para esta prueba si solo te importa `ajax.system`).
    

> Si tu GUI no tiene la opción **Type = Proxy/Forwarder**, crea la zona y pon **Authoritative = OFF** + “DNS Forwarder = 10.198.3.69”. Es lo mismo.

---

## 2) Servidor DNS en Windows (10.198.3.69)

Tienes dos caminos:

### Opción A — **Windows Server (rol DNS)**

1. **Instala el rol DNS**
    
    - Server Manager → _Add Roles and Features_ → **DNS Server**
        
    - (PowerShell) `Install-WindowsFeature DNS -IncludeManagementTools`
        
2. **Crea zona directa primaria**:
    
    - _DNS Manager_ → _Forward Lookup Zones_ → _New Zone…_
        
    - **Zone name:** `ajax.system` → _Primary zone_.
        
3. **Añade los A-records:**
    
    - `pri A 10.198.3.36`
        
    - `sec A 10.198.3.52`
        
4. **Firewall de Windows:** el rol abre UDP/TCP 53 automáticamente. Verifica que responde:
    
    - `nslookup pri.ajax.system 10.198.3.69` (desde cualquier host con reachability).
        

### Opción B — **Windows 10/11** (sin rol DNS)

Instala un DNS ligero como **Technitium DNS Server** (gratuito):

1. Descarga e instala (servicio Windows).
    
2. _Zones → Add New Zone → Primary_ → **Domain:** `ajax.system`.
    
3. Añade registros:
    
    - **A** `pri` → `10.198.3.36`
        
    - **A** `sec` → `10.198.3.52`
        
4. Asegura que escucha en `0.0.0.0:53` y que el **Firewall de Windows** permite UDP/TCP 53.
    
5. Test: `nslookup pri.ajax.system 10.198.3.69`.
    

---

## 3) Pruebas desde un PC de `172.100.100.0/24`

> Ese PC **apunta a 10.32.32.33** como DNS.

```c
nslookup pri.ajax.system 10.32.32.33
nslookup sec.ajax.system 10.32.32.33
```

**Esperado:**

```c
Name:    pri.ajax.system
Address: 10.198.3.36

Name:    sec.ajax.system
Address: 10.198.3.52

```
Prueba también sin indicar servidor (usará el que tenga configurado el PC):

```c
nslookup pri.ajax.system
```

---

## 4) Si no responde como esperas (troubleshooting rápido)

- **Caché del FortiGate**: limpia la caché del DNS-proxy antes de repetir:
    
    `diagnose test application dnsproxy 6    # flush cache`
```c
diagnose test application dnsproxy 6
```
    
- **Local-in policy**: si tienes políticas locales estrictas, añade la que te puse para permitir DNS hacia **port4**.
    
- **Rutas**: el FGT debe poder alcanzar **10.198.3.69** (ya dijiste que hay conectividad total).
    
- **Firewall Windows**: abre **UDP/TCP 53** si usas un DNS “no-rol”.
    
- **Ver en FGT qué hace el proxy**:
    
```c
diagnose sniffer packet any 'port 53 and host 10.198.3.69' 4 0 a
```
    

---

## Resumen

1. En **FW-PS**, habilita DNS-proxy y crea un **domain override** para `ajax.system` → **10.198.3.69**.
    
2. En **Windows (10.198.3.69)**, crea la zona `ajax.system` y los dos **A** (`pri`, `sec`).
    
3. Los clientes (`172.100.100.0/24`) preguntan a **10.32.32.33** y reciben **10.198.3.36/52** según el nombre.
    

Cuando lo tengas funcionando “sin restricciones”, me dices y añadimos la **ACL en R-PS** y, si quieres, pasamos a la **VIP + Split-DNS** para entregar IPs del rango permitido.

---
---
---

# PASO 1: FortiGate FW-PS – DNS forwarding “condicional” (domain override)


desde **cero** en el FortiGate (FW-PS) y **todo por CLI**.  
Objetivo: que el FW-PS (IP **10.32.32.33**) atienda las consultas DNS de los PCs de **172.100.100.0/24** y **reenvíe** cualquier nombre de **`*.ajax.system`** al servidor DNS de Windows **10.198.3.69**. (Aún no creamos VIP ni ACL).

> Nota: en tus capturas la zona se llama `forti.ps`. Para este paso debes usar **`ajax.system`** y marcarla como **forwarder/proxy**, **no** autoritativa. No añadas registros A aquí.

---

## 0) Identifica la interfaz por la que llegan las consultas

(la que tiene **10.32.32.33**; en tu esquema suele ser `port4`)

```
get system interface | grep -f 10.32.32.33

# O mira todas:
show system interface
```

Supongamos que es **`port4`**. Si el nombre es otro, sustitúyelo en los comandos.

---

## 1) Activa el DNS proxy del FortiGate (resolver)

El FGT escuchará en `port4` y reenviará lo que no sea override a tus forwarders generales.

```C
config system dns
    set dnsproxy enable
    set primary 8.8.8.8          # opcional: forwarders “generales”
    set secondary 4.4.4.4
    set dns-cache-limit 10000
    set dns-cache-ttl 300
end

config system dns-server
    edit "port4"                 # interfaz donde están los PCs
        set mode recursive       # el FGT responde/forwardea
    next
end

```

> Si no tienes salida a Internet desde el propio FGT, que “1.1.1.1/8.8.8.8” pongan _Unreachable_ no afecta a este caso; lo importante es el **override** a 10.198.3.69.

---

## 2) Crea el **forwarding condicional** para `ajax.system`

Hay dos sintaxis según versión; cualquiera de las dos te funciona. Usa **una**:

### A) Con _set forwarder_

```C
config system dns-database
    edit "AJAX_FORWARD"
        set domain "ajax.system"
        set authoritative disable        # NO autoritativa
        set view proxy           # <--- IMPORTANTE: usa 'proxy', no 'shadow'
        set forwarder 10.198.3.69
        set ttl 300
    next
end

```


### B) Con tabla de _forwarders_ (vale en 6.4/7.x)

```C
config system dns-database
    edit "AJAX_FORWARD"
        set domain "ajax.system"
        set authoritative disable        # NO autoritativa
        set view proxy           # <--- IMPORTANTE: usa 'proxy', no 'shadow'
        config forwarder
            edit 1
                set ip 10.198.3.69       # tu DNS de Windows
            next
        end
        set ttl 300
    next
end

```

### C) Con `type forwarder` (si tu CLI lo admite)

```C
config system dns-database
    edit "AJAX_FORWARD"
        set domain "ajax.system"
        set type forwarder               # zona de reenvío
        set view proxy           # <--- IMPORTANTE: usa 'proxy', no 'shadow'
        config forwarder
            edit 1
                set ip 10.198.3.69
            next
        end
        set ttl 300
    next
end

```

> No añadas `dns-entry` aquí; esta zona **no** aloja registros, solo **reenvía** todo `*.ajax.system` al 10.198.3.69.

---

## 3) (Solo si usas Local-In Policy estricta) permite DNS al propio FGT

Si tienes políticas de _local-in_ cerradas, abre DNS en `port4`:

```C
config firewall local-in-policy
    edit 0
        set intf "port4"
        set srcaddr "all"
        set dstaddr "all"
        set service "DNS"
        set action accept
    next
end

```

---

## 4) Limpia caché y verifica desde el FortiGate

```c
# Vacía la caché del DNS-proxy
diagnose test application dnsproxy 6

# Ver que la zona está en 'view proxy'
show system dns-database | grep -f AJAX_FORWARD -n
# o simplemente:
show system dns-database


# Observa si el FGT reenvía a 10.198.3.69 cuando un cliente pregunta.
# Deja este sniffer mirando; en el siguiente paso haz la consulta desde un PC.
diagnose sniffer packet any 'port 53 and host 10.198.3.69' 4 0 a
```


---

## 5) Prueba desde un PC de 172.100.100.0/24 (DNS apuntando a 10.32.32.33)

_(Aún no configuramos el DNS de Windows, así que si el servicio no está listo dará NXDOMAIN; lo importante ahora es que la consulta **salga** del FGT hacia 10.198.3.69)._

En un PC de `172.100.100.0/24` (DNS = **10.32.32.33**):
```C
nslookup pri.ajax.system 10.32.32.33
```
o simplemente:
```c
nslookup pri.ajax.system
nslookup sec.ajax.system
```


- En la consola del FGT deberías **ver paquetes DNS** hacia **10.198.3.69** (sniffer del paso 4).
    
- Cuando configures el DNS de Windows con la zona `ajax.system` y los A-records, la respuesta será:
    
    - `pri.ajax.system` → **10.198.3.36**
        
    - `sec.ajax.system` → **10.198.3.52**
        

---

### Resumen del Paso 1

- **DNS proxy** activado y escuchando en la interfaz de clientes.
    
- **Zona de reenvío condicional** `ajax.system` → **10.198.3.69** (NO autoritativa, sin registros).
    
- (Opcional) **local-in policy** para permitir DNS al propio FGT.
    
- Prueba con `nslookup` y observa en el FGT que envía las consultas a **10.198.3.69**.
    

Cuando lo tengas así, pasamos al **Paso 2**: levantar el **DNS de Windows (10.198.3.69)** con la zona **`ajax.system`** y los registros `pri` y `sec`.

---
---

# PASO 2: DNS SERVER EN WINDOWS



## 5) Firewall de Windows (verificación)

El instalador normalmente crea las reglas, pero confirma:
```c
# Habilitar reglas DNS si estuvieran deshabilitadas
New-NetFirewallRule -DisplayName "Technitium DNS TCP 53" -Direction Inbound -Protocol TCP -LocalPort 53 -Action Allow
New-NetFirewallRule -DisplayName "Technitium DNS UDP 53" -Direction Inbound -Protocol UDP -LocalPort 53 -Action Allow
# (opcional) abrir el panel web desde tu LAN
New-NetFirewallRule -DisplayName "Technitium Admin 5380" -Direction Inbound -Protocol TCP -LocalPort 5380 -Action Allow

```