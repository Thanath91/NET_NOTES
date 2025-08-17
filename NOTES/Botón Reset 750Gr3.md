

Si el **botón de reset** del **MikroTik hEX RB750Gr3** no está reseteando nada, hay varias razones posibles, y en este modelo es muy común que la gente no lo suelte en el momento correcto o que el router esté arrancando en un modo diferente.

Te explico cómo funciona realmente y qué puedes probar para recuperarlo.

---

## 🔹 1. Comportamiento del botón RESET en el RB750Gr3

En este modelo el botón tiene **tres modos** dependiendo del tiempo que lo mantengas:

|Tiempo de pulsación|Modo|Qué hace|
|---|---|---|
|~5 segundos (hasta que el LED ACT parpadea rápido)|**Reset normal**|Borra la configuración y carga la default (192.168.88.1/24, sin pass)|
|~15 segundos (LED ACT parpadea lento, luego rápido)|**Reset + no-defaults**|Borra la configuración y no carga configuración por defecto|
|~20-25 segundos|**Netinstall mode**|Arranca esperando reinstalación por red (puerto ether1)|

Si lo mantienes demasiado, no resetea configuración, entra a **Netinstall** y por eso parece que “no hace nada”.

---

## 🔹 2. Reset correcto paso a paso

1. **Apaga** el MikroTik (desconecta la corriente).
    
2. **Mantén pulsado** el botón RESET.
    
3. **Enciende** el MikroTik **sin soltar el botón**.
    
4. **Observa el LED ACT**:
    
    - Cuando parpadee rápido por primera vez (**a los ~5 segundos**), suelta el botón → esto hace _reset normal_.
        
5. Espera 1-2 minutos a que reinicie con la config por defecto.
    

---

## 🔹 3. Si el reset normal no funciona

Puede pasar si:

- Tienes activada la opción **"Protected RouterBOOT"** (evita reset por botón).  
    Para quitarla necesitarías entrar por consola o Netinstall.
    
- El botón físicamente está dañado.
    
- El firmware tiene corrupción.
    

---

## 🔹 4. Recuperación con Netinstall (modo forzado)

Si el botón está protegido o la config bloquea acceso, puedes **reinstalar RouterOS** con **Netinstall**:

1. Descarga Netinstall desde la web oficial de MikroTik.
    
2. Conecta el PC **directamente a ether1** del MikroTik.
    
3. Configura el PC con IP estática `192.168.88.2/24`.
    
4. Entra en modo Netinstall:
    
    - Apaga el MikroTik.
        
    - Pulsa RESET.
        
    - Enciende y mantén RESET ~20-25 segundos hasta que el LED ACT parpadee de forma diferente (modo Netinstall).
        
5. En Netinstall selecciona la imagen de RouterOS correcta y reinstala.
    
6. Al terminar, arrancará con config por defecto.
    

---

📌 **Conclusión**:  
Si el reset físico no hace nada, probablemente estés **pulsando demasiado tiempo** y entrando en Netinstall o tengas **Protected RouterBOOT** activado.  
En ese caso, la vía segura es usar **Netinstall** para reinstalar y dejarlo limpio.

---

Si quieres, puedo pasarte **un tutorial paso a paso con capturas para hacer Netinstall en un RB750Gr3** y dejarlo como recién comprado.  
¿Quieres que te lo prepare?

##### Tú dijiste:

tengo el led pwr

###### ChatGPT Plus

Vale, si en tu **MikroTik hEX RB750Gr3** solo ves **el LED PWR** encendido y ningún otro (ni ACT, ni puertos), eso ya no es un problema de configuración:  
significa que el router **ni siquiera está cargando RouterOS** o que **se ha quedado colgado en el arranque**.

Eso ocurre normalmente en tres casos:

1. **Firmware corrupto** (configuración dañada o actualización interrumpida).
    
2. **Modo Netinstall** esperando reinstalación.
    
3. **Fallo de hardware** (menos común, pero posible).
    

---

## 🔹 Cómo saber si está en modo Netinstall o bloqueado

- Si lo conectas al PC **por un puerto LAN**, ¿ves actividad de red en el LED del puerto?
    
    - **Sí** → probablemente está en modo Netinstall.
        
    - **No** → puede estar colgado antes de inicializar interfaces.
        

---

## 🔹 Recuperación recomendada: **Netinstall**

Es el procedimiento oficial de MikroTik para reinstalar el sistema cuando no arranca.

**Pasos resumidos**:

1. **Descargar herramientas**:
    
    - Netinstall (elige la versión estable).
        
    - Imagen RouterOS **mipsbe** (porque tu RB750Gr3 usa arquitectura MIPSBE).
        
2. **Configurar el PC**:
    
    - Conectar **directamente** el PC a **ether1** del MikroTik.
        
    - Desactivar Wi-Fi y demás interfaces.
        
    - Asignar IP estática: `192.168.88.2/24`.
        
3. **Poner MikroTik en modo Netinstall**:
    
    - Apagar el router.
        
    - Mantener pulsado RESET.
        
    - Encenderlo y seguir pulsando ~20-25 segundos hasta que el LED ACT parpadee de forma diferente (si no parpadea, suelta a los 25 seg).
        
4. **Usar Netinstall**:
    
    - Ejecutar Netinstall como administrador.
        
    - Seleccionar la interfaz de red del PC.
        
    - El MikroTik debería aparecer en la lista (por MAC).
        
    - Cargar la imagen `.npk` de RouterOS.
        
    - Pulsar **Install** y esperar a que termine.
        
    - Reiniciar el MikroTik.
        
5. **Acceso**:
    
    - Volverá a la configuración por defecto: IP `192.168.88.1`, usuario `admin`, sin contraseña.
        

---

📌 **Nota**:  
Si incluso en Netinstall **no aparece el dispositivo** y solo ves el LED PWR fijo, lo más probable es que haya un fallo físico en la placa o la memoria, y ahí ya solo se podría recuperar con soporte técnico o cambio de hardware.