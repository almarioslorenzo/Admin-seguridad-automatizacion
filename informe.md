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