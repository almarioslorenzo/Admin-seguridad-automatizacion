## UNIVERSIDAD AUTÓNOMA DE ENTRE RÍOS - FACULTAD DE CIENCIA Y TECNOLOGÍA - INGENIERÍA EN TELECOMUNICACIONES

# TRABAJO PRÁCTICO 4
#  Administración, seguridad y automatización de redes
| Concepto | Detalle |
| :--- | :--- |
| **Materia** | Servicios de Telecomunicaciones y Redes |
| **Año** | 2026 |
| **Modalidad** | Grupal (2-3 integrantes) |
| **Duración** | 3 semanas |
| **Entrega** | Martes 9/6 (defensa: miércoles 10/6) |
| **Integrantes** | Alma Ríos, Gastón Barreto |

## FASE 1: Problema disparador
### **Preguntas al cliente**
El responsable técnico de la empresa les plantea estas preguntas concretas:
1. Tenemos todo en la misma red: servidores web, bases de datos, gestión. ¿Cómo separo esto sin cambiar hardware?
2. Nos hackearon 3 veces en 3 meses. ¿Cómo protejo los 80 sitios web sin poner algo individual en cada uno?
3. Un cliente accedió a la base de datos de otro cliente porque estaban en la misma red. ¿Cómo evito que esto vuelva a pasar?
4. Si se cae el router de borde, perdemos todo. ¿Cómo evito eso?

## FASE 2: Mapa de aprendizaje
| Ya sabemos | Necesitamos aprender | Cómo lo vamos a aprender |
| :--- | :--- | :--- |
| **Segmentación práctica con VLANs:** Configuración de direccionamiento, subredes, y puesta en marcha de VLANs en routers y switches. | **Seguridad práctica en servidores web:** Cómo pasar de la teoría a la práctica instalando y configurando herramientas que actúen como un escudo para frenar ataques web. | Buscando tutoriales prácticos para desplegar herramientas como Nginx o ModSecurity, usando laboratorios controlados o contenedores para ver cómo bloquean el tráfico malicioso. |
| **Seguridad informática:** Conceptos teóricos sobre ataques comunes en capa de aplicación (como inyecciones SQL, hackeos de páginas o ataques de fuerza bruta). | **Aislamiento real entre servidores y bases de datos:** Cómo configurar los cortafuegos internos de los sistemas operativos y los permisos de las bases de datos para aplicar el concepto de Zero Trust ("no confiar en nadie"). | Estudiando guías básicas de configuración de firewalls de sistema y aprendiendo a restringir los accesos por IP en bases de datos como MySQL. |
| **Conectividad e introducción a Zero Trust:** Entendimiento conceptual de la filosofía de seguridad interna y la necesidad de aislar a los clientes entre sí. | **Alta disponibilidad y redundancia:** Cómo configurar dos routers para que trabajen en equipo mediante protocolos, logrando que si uno falla, el otro tome el control automáticamente. | Revisando guías sencillas sobre el protocolo VRRP y simulando la desconexión de un router en un entorno virtual para comprobar que el servicio no se corte. |
| **Topologías de red lineales:** Cómo funciona una red con una única salida a internet y el riesgo que implica no tener un respaldo físico. | **Automatización avanzada con Oxidized:** Cómo profundizar en el uso de Oxidized para centralizar y automatizar por completo la recolección de configuraciones de los equipos de red. | Investigando la integración de Oxidized con contenedores o servidores locales, y configurando alertas o disparadores para que guarden los respaldos de forma automática al detectar cambios. |
| **Control de versiones con Git:** Manejo de comandos de Git (commits, push, ramas) e integración con repositorios remotos para guardar código y configuraciones de forma ordenada. | Se aplica como herramienta de soporte para guardar los archivos de configuración y scripts que se aprendan en las otras fases. | Utilizando los conocimientos de Git ya adquiridos para subir los archivos de configuración de los firewalls, WAF y routers a un repositorio seguro. |

## Fase 3: Investigación
### VLAN
Una VLAN (Virtual LAN) divide un switch físico en múltiples redes lógicas independientes. Los dispositivos en la misma VLAN se comunican entre sí como si estuvieran en la misma red, aunque estén en puertos físicos distintos. Dispositivos en VLANs distintas no se ven entre sí sin que un router o firewall lo permita explícitamente, lo que es precisamente el objetivo de seguridad.

Cuando un frame Ethernet cruza un enlace que lleva múltiples VLANs (enlace trunk), se le inserta una etiqueta de 4 bytes entre el campo de origen MAC y el campo EtherType. Esa etiqueta contiene el VLAN ID (VID), un número entre 1 y 4094. El switch destino lee el VID, determina a qué VLAN pertenece el tráfico y lo entrega al puerto correspondiente.

### Puertos tagged vs untagged
* Puerto tagged (trunk): El frame viaja con la etiqueta 802.1Q intacta. Se usa en enlaces entre switches o entre switch y router, donde un solo cable debe transportar tráfico de múltiples VLANs. El equipo del otro lado entiende y procesa esas etiquetas.
* Puerto untagged (access): El switch quita la etiqueta antes de entregar el frame al dispositivo final (un servidor, una PC). El host no necesita saber nada de VLANs. Cada puerto untagged pertenece a exactamente una VLAN; esa es su PVID (Port VLAN ID).

| VLAN | Nombre | IP MikroTik A | IP MikroTik B | IP Virtual (VRRP) | Rango hosts | Uso |
| :--- | :----- | :------------ | :------------ | :----------------- | :---------- | :-- |
| 10 | Web | 10.10.10.2/24 | 10.10.10.3/24 | 10.10.10.1 | .4 - .254 | Servidores web (WAF via Docker Compose) |
| 20 | DB | 10.10.20.2/24 | 10.10.20.3/24 | 10.10.20.1 | .4 - .254 | Bases de datos |
| 30 | Gestión | 10.10.30.2/24 | 10.10.30.3/24 | 10.10.30.1 | .4 - .254 | Administración, SSH |

### Firewall
El firewall de MikroTik es una herramienta fundamental para controlar y proteger el tráfico de red. Funciona mediante reglas que se organizan en diferentes chains (cadenas), las cuales determinan qué tipo de tráfico será analizado y qué acciones se aplicarán sobre él.

Las tres chains principales del firewall son:
| Chain | Qué procesa | Ejemplo |
|--------|------------|----------|
| **Input** | Tráfico dirigido al propio router MikroTik. | Un administrador se conecta por WinBox, SSH o hace ping al router. |
| **Forward** | Tráfico que atraviesa el router entre diferentes redes o entre la LAN e Internet. | Un usuario navega por Internet desde una PC conectada a la red local. |
| **Output** | Tráfico generado por el propio router y enviado hacia otros dispositivos o servicios. | El router realiza una consulta DNS, envía logs o ejecuta un ping a 8.8.8.8. |

Cada una tiene una función específica según el destino o el origen de los paquetes de datos.

### Reglas de MikroTik
Las reglas de MikroTik son instrucciones configuradas en el router que determinan cómo debe tratarse el tráfico de red. Estas reglas forman parte principalmente del Firewall, aunque también existen reglas para NAT, enrutamiento y calidad de servicio.

Cada vez que un paquete de datos llega al router o pasa a través de él, MikroTik compara ese paquete con las reglas configuradas. Cuando encuentra una regla que coincide con las características del paquete (origen, destino, protocolo, puerto, interfaz, etc.), ejecuta la acción definida.

Una regla suele estar formada por:
* Chain: dónde se evaluará el tráfico (input, forward u output).
* Condiciones: características que debe cumplir el tráfico (IP origen, IP destino, protocolo, puerto, interfaz, etc.).
* Acción: qué hacer con el tráfico si cumple las condiciones.
  
| Acción | Descripción |
|---------|-------------|
| **accept** | Permite el tráfico que coincide con la regla. |
| **drop** | Descarta el tráfico sin enviar ninguna respuesta al origen. |
| **reject** | Rechaza el tráfico y envía una notificación al origen indicando que fue bloqueado. |
| **log** | Registra información del tráfico en los logs del router para monitoreo o diagnóstico. |
| **masquerade** | Realiza NAT reemplazando la dirección IP privada por la IP de salida del router. |

Las reglas en MikroTik se crean dentro del firewall mediante comandos de RouterOS o utilizando la interfaz gráfica WinBox.

Creación desde WinBox:
1. Abrir WinBox y conectarse al router.
2. Ir a IP → Firewall.
3. Seleccionar la pestaña Filter Rules.
4. Hacer clic en + para agregar una nueva regla.
5. Configurar la chain, las condiciones y la acción.
6. Guardar los cambios con Apply y OK.

Las reglas se procesan de arriba hacia abajo. Cuando un paquete coincide con una regla que toma una decisión definitiva (por ejemplo, accept o drop), las reglas posteriores ya no se evalúan para ese paquete. Por ello, el orden de las reglas es fundamental para el correcto funcionamiento del firewall.

| Tipo | Código | ¿Qué hace? |
|--------|---------|------------|
| **Sintaxis básica** | `add chain=<chain> [condiciones] action=<acción>` | Define una nueva regla indicando dónde se evaluará el tráfico, qué condiciones debe cumplir y qué acción se ejecutará. |
| **Permitir acceso por WinBox** | `add chain=input protocol=tcp dst-port=8291 action=accept` | Permite conexiones al router mediante WinBox (puerto 8291). |
| **Bloquear acceso entre VLANs** | `add chain=forward src-address=192.168.30.0/24 dst-address=192.168.10.0/24 action=drop` | Impide que los equipos de la VLAN Web accedan a la VLAN Gestión. |
| **Permitir tráfico entre VLANs** | `add chain=forward src-address=192.168.20.0/24 dst-address=192.168.10.0/24 action=accept` | Autoriza la comunicación desde la VLAN Usuarios hacia la VLAN Gestión. |
| **Permitir navegación a Internet** | `add chain=forward src-address=192.168.20.0/24 action=accept` | Permite que los equipos de la red 192.168.20.0/24 envíen tráfico a través del router. |
| **Registrar tráfico en logs** | `add chain=forward action=log` | Registra en los logs del router el tráfico que coincida con la regla. |
| **NAT para salida a Internet** | `add chain=srcnat out-interface=ether1 action=masquerade` | Traduce las direcciones IP privadas de la red interna a la IP pública del router para acceder a Internet. |

### Connection tracking Mikrotik
El Connection Tracking (seguimiento de conexiones) es una función de MikroTik que permite al router identificar, registrar y controlar el estado de las conexiones de red que atraviesan el dispositivo.

Gracias a esta función, el router puede saber si un paquete pertenece a:
* Una conexión nueva.
* Una conexión ya establecida.
* Una conexión relacionada con otra.
* Una conexión inválida.

Esto es fundamental para el funcionamiento del firewall, NAT y otras funciones avanzadas de RouterOS.

Cuando un dispositivo inicia una comunicación (por ejemplo, abrir una página web), MikroTik crea un registro de esa conexión en una tabla interna.

Luego, todos los paquetes relacionados con esa comunicación son identificados automáticamente y clasificados según su estado.
| Estado | Descripción |
|---------|-------------|
| **new** | El paquete inicia una nueva conexión. |
| **established** | El paquete pertenece a una conexión que ya fue establecida previamente. |
| **related** | El paquete está asociado o relacionado con una conexión existente. |
| **invalid** | El paquete no puede asociarse a una conexión válida o presenta errores. |

En MikroTik, las reglas del firewall se evalúan de arriba hacia abajo. La práctica recomendada es colocar al principio reglas que acepten conexiones established y related, ya que la mayor parte del tráfico de una red corresponde a conexiones que ya están en curso.

**Ejemplo:**

*/ip firewall filter*

*add chain=forward connection-state=established,related action=accept*

*add chain=forward connection-state=invalid action=drop*

*add chain=forward connection-state=new src-address=192.168.20.0/24 action=accept*

Así los paquetes de conexiones ya establecidas son aceptados inmediatamente. Los paquetes inválidos se descartan rápidamente. Solo las conexiones nuevas pasan por las reglas más específicas de filtrado.

De esta manera, el router evita analizar miles de paquetes contra todas las reglas del firewall.

| Beneficio | Explicación |
|------------|------------|
| **Menor uso de CPU** | Los paquetes de conexiones establecidas se procesan rápidamente sin revisar todas las reglas del firewall. |
| **Menor latencia** | El tráfico atraviesa el router más rápido al requerir menos verificaciones. |
| **Mayor escalabilidad** | El router puede manejar una mayor cantidad de usuarios y conexiones simultáneas. |
| **Firewall más eficiente** | Las reglas complejas se aplican únicamente a las conexiones nuevas que requieren evaluación. |
| **Mejor aprovechamiento de Connection Tracking** | El router utiliza la información del estado de las conexiones para tomar decisiones de forma rápida y eficiente. |

Las reglas que aceptan tráfico established y related se colocan al principio porque representan la mayor parte del tráfico de una red. Al procesarlas primero, MikroTik reduce la cantidad de comparaciones que realiza por paquete, disminuye la carga del procesador y mejora significativamente el rendimiento del router.

### WAF
Un WAF (Web Application Firewall) o Firewall de Aplicaciones Web es un sistema de seguridad diseñado para proteger aplicaciones y sitios web frente a ataques dirigidos a la capa de aplicación (capa 7 del modelo OSI).

A diferencia de un firewall tradicional, que filtra tráfico según direcciones IP, puertos o protocolos, un WAF analiza el contenido de las solicitudes HTTP y HTTPS para detectar y bloquear actividades maliciosas.

El WAF se ubica entre los usuarios e Internet y la aplicación web que se desea proteger. Todo el tráfico web pasa primero por el WAF, que inspecciona las solicitudes y decide si permitirlas o bloquearlas.

**Amenazas que puede bloquear:**
| Ataque | Descripción |
|---------|-------------|
| **SQL Injection (SQLi)** | Intentos de ejecutar consultas maliciosas en una base de datos mediante la inserción de código SQL. |
| **Cross-Site Scripting (XSS)** | Inyección de scripts maliciosos en páginas web que serán ejecutados en el navegador de otros usuarios. |
| **Cross-Site Request Forgery (CSRF)** | Realización de acciones no autorizadas aprovechando la sesión autenticada de un usuario. |
| **Directory Traversal** | Intentos de acceder a archivos o directorios restringidos del servidor mediante la manipulación de rutas. |
| **Bot Malicioso** | Tráfico automatizado utilizado para ataques, recopilación de información o abuso de servicios. |
| **DDoS a nivel aplicación** | Saturación de los recursos de una aplicación web mediante un gran volumen de solicitudes HTTP/HTTPS. |

| Firewall tradicional | WAF (Web Application Firewall) |
|---------------------|--------------------------------|
| Protege redes y dispositivos. | Protege aplicaciones y sitios web. |
| Filtra tráfico según direcciones IP, puertos y protocolos. | Analiza solicitudes HTTP/HTTPS y su contenido. |
| Opera principalmente en las capas 3 (Red) y 4 (Transporte) del modelo OSI. | Opera principalmente en la capa 7 (Aplicación) del modelo OSI. |
| Bloquea accesos no autorizados a la red. | Bloquea ataques dirigidos a aplicaciones web. |
| Controla el tráfico entre redes o dispositivos. | Inspecciona formularios, URLs, cookies y parámetros de las aplicaciones web. |
| Protege contra amenazas de red como escaneos de puertos o accesos indebidos. | Protege contra ataques como SQL Injection, XSS y CSRF. |

**Alternativas open-source:**
| WAF | Despliegue | OWASP | Rate Limit | Comunidad | Documentación | Dificultad |
|------|------------|-----------|------------|------------|---------------|------------|
| BunkerWeb | Docker, Linux, Kubernetes, Reverse Proxy | Sí (integrado) | Sí | Media-Alta | Muy buena | Baja |
| ModSecurity + Coraza | Apache, Nginx, HAProxy, Caddy, aplicaciones Go | Sí (compatibilidad completa) | Depende del proxy utilizado | Muy alta | Excelente | Alta |
| Caddy Security | Plugin para Caddy Server | Parcial (requiere configuración adicional) | Sí | Media | Buena | Media |
| OpenAppSec | Docker, Kubernetes, Nginx, Kong, APISIX | Sí (protección OWASP Top 10) | Sí | Media | Muy buena | Media |
| NAXSI | Nginx | Parcial | No nativo | Media | Buena | Media-Alta |

Para nuestro caso elegimos BunkerWeb porque ofrece un equilibrio muy bueno entre seguridad y facilidad de uso. Se despliega fácilmente mediante Docker, integra protección basada en OWASP, incluye limitación de tasa (rate limiting), posee una interfaz web de administración y requiere menos configuración inicial que ModSecurity o Coraza. Además, su documentación es clara. Para una pequeña o mediana organización, permite implementar un WAF funcional en poco tiempo sin sacrificar capacidades de protección.

### VRRP
VRRP (Virtual Router Redundancy Protocol) es un protocolo de redundancia que permite que varios routers trabajen juntos para proporcionar una puerta de enlace predeterminada (gateway) altamente disponible. Su objetivo es evitar que la red quede sin acceso si el router principal falla.

VRRP crea un router virtual con una dirección IP compartida entre varios routers físicos.
* Un router actúa como Master (activo).
* Un router actúa como Backup (respaldo).
* Los dispositivos de la red utilizan la IP virtual como gateway.
* Si el Master falla, uno de los Backup toma automáticamente su lugar.