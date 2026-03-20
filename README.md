# 🌐 VirtualBox Networking: Guía Completa de Configuración

Este repositorio contiene guías y recursos para configurar correctamente las redes en Oracle VM VirtualBox. El objetivo principal es lograr el escenario ideal para desarrollo y pruebas: que la máquina virtual (VM) tenga acceso a Internet y, simultáneamente, sea accesible desde la máquina física (Host). En el inicio de mi aprendizaje fue uno los puntos donde invertí más tiempo, incluso ahora cuando monto algun laboratorio de prueba tengo que repasar algunos conceptos. Espero que esta guía sirva de referencia a alguien más.
---

## 🛠️ Modos de Red en VirtualBox

A continuación se detallan los modos de red disponibles y sus casos de uso específicos:

### 1. NAT (Network Address Translation)
* **Cómo funciona:** La MV comparte la IP del host. VirtualBox actúa como un router interno. El host no puede acceder a la MV a menos que se configure "Port Forwarding".
* **Acceso a Internet:** ✅ Sí.
* **Comunicación entre MV:** ❌ No (aisladas entre sí).
* **Comunicación con el Host:** ❌ No (solo mediante reenvío de puertos).
* **Uso típico:** Navegación web básica y descargas seguras donde la MV no necesita ser "vista" desde fuera.

### 2. Adaptador Puente (Bridged Adapter)
* **Cómo funciona:** La MV actúa como un equipo independiente en la red física. Obtiene su propia IP del router real (casa/oficina).
* **Acceso a Internet:** ✅ Sí.
* **Comunicación entre MV:** ✅ Sí (con todos los equipos de la red física).
* **Comunicación con el Host:** ✅ Sí.
* **Uso típico:** Servidores web/archivos y pruebas de red reales accesibles por otros dispositivos locales.

### 3. Solo-Anfitrión (Host-Only)
* **Cómo funciona:** Crea una red privada totalmente aislada del exterior. Solo interactúan el Host y las MVs conectadas a este adaptador.
* **Acceso a Internet:** ❌ No.
* **Comunicación entre MV:** ✅ Sí.
* **Comunicación con el Host:** ✅ Sí.
* **Uso típico:** Entornos de pruebas aislados, bases de datos privadas y clusters de desarrollo.

### 4. Red NAT (NAT Network)
* **Cómo funciona:** Similar al NAT simple, pero permite que varias MVs se comuniquen entre sí dentro de la misma subred privada mientras salen a internet.
* **Acceso a Internet:** ✅ Sí.
* **Comunicación entre MV:** ✅ Sí (si comparten la misma "NAT Network").
* **Comunicación con el Host:** ❌ No (requiere forwarding).
* **Uso típico:** Grupos de servidores que necesitan internet y hablarse entre ellos (ej: Web + DB).

### 5. Red Interna (Internal Network)
* **Cómo funciona:** Aislamiento total. Ni el Host ni Internet tienen acceso. Solo las MVs con el mismo nombre de red interna se ven.
* **Acceso a Internet:** ❌ No.
* **Comunicación entre MV:** ✅ Sí.
* **Comunicación con el Host:** ❌ No.
* **Uso típico:** Malware analysis, laboratorios de hacking ético y redes altamente sensibles.

### 6. Adaptador Genérico (Generic Driver)
* **Cómo funciona:** Permite usar drivers especiales (UDP Tunnel, VDE) para conectar MVs entre diferentes hosts físicos.
* **Uso típico:** Configuraciones experimentales o redes distribuidas avanzadas.
---

## 📊 Tabla Comparativa (Resumen Visual)

| Modo | Internet | Comunicación MVs | Comunicación Host | Aislamiento |
| :--- | :---: | :---: | :---: | :--- |
| **NAT** | ✅ Sí | ❌ No* | ❌ No* | Medio |
| **Puente** | ✅ Sí | ✅ Sí | ✅ Sí | Nulo (Expuesta) |
| **Solo-Anfitrión** | ❌ No | ✅ Sí | ✅ Sí | Alto |
| **Red NAT** | ✅ Sí | ✅ Sí | ❌ No* | Medio |
| **Interna** | ❌ No | ✅ Sí | ❌ No | Total |

*\* Excepto mediante reglas de "Port Forwarding".*

![Infografia Virtualbox](./img/vm_virtualbox.png) 
---

## Conectividad interna entre máquinas y externa (internet)

Al trabajar con entornos virtualizados, una de las mayores frustraciones es la conectividad. A menudo nos encontramos con que la VM tiene Internet pero no podemos conectarnos a ella por SSH o HTTP desde nuestro PC, o viceversa.

Para solucionar esto, esta guía promueve el uso de un diseño de red híbrido. En lugar de intentar forzar un solo adaptador de red para que lo haga todo, configuramos dos tarjetas de red virtuales para separar las responsabilidades:

    Adaptador 1 (NAT): Proporciona un túnel seguro a la VM para salir a Internet, realizar actualizaciones de sistema y descargar paquetes.
    Adaptador 2 (Solo-Anfitrión / Host-Only): Crea una red privada y estable exclusiva entre el Host (tu PC físico) y la VM. Es el "cable" por el que te conectarás para trabajar (SSH, Bases de Datos, servidores Web).

---

## ❓ FAQ: Solución de Problemas Comunes

### 1. 🛑 Error "NS_ERROR_NOT_IMPLEMENTED" en Windows
Si al intentar configurar el adaptador Solo-Anfitrión recibes este error o ves "IPv4: No conectado" en Windows:
* **Solución:** Ve a `Conexiones de Red` en Windows, clic derecho en el adaptador de VirtualBox > `Propiedades` > `IPv4`. Asigna manualmente la IP `192.168.56.1` con máscara `255.255.255.0`. Esto despierta la interfaz en el host.

### 2. 📭 La VM tiene IPv6 pero no IPv4
* **Causa:** El servidor DHCP de VirtualBox no ha entregado IP.
* **Solución:** Revisa en `Archivo > Herramientas > Network Manager` que el DHCP esté habilitado. Si persiste, asigna una IP fija en la VM (ej: `192.168.56.101`).

### 3. 📂 No existe la carpeta `/etc/netplan` en Ubuntu
* **Causa:** Estás usando Ubuntu Desktop, que gestiona la red mediante **NetworkManager**.
* **Solución:** Usa la interfaz gráfica o el comando `nmcli`. Si prefieres Netplan, debes crear el archivo `.yaml` manualmente siguiendo la indentación estricta de 2 espacios.

---
_Documentación compilada por: **Pere Cler**_  
[🚀 Visitar cler.cloud](https://cler.cloud)
