A continuación se detallan los bloques de comandos aplicados secuencialmente en las consolas de cada RouterOS.

### 1️⃣ MikroTik-1 (VRRP Master)

```nodescript
# ==========================================
# PARTE 1: ARQUITECTURA DE VLANs (Bridge VLAN Filtering)
# ==========================================

# Crear el puente lógico principal con filtrado de VLAN activado
/interface bridge add name=bridge1 vlan-filtering=yes

# Asociar el puerto físico de bajada (ether2) al bridge
/interface bridge port add bridge=bridge1 interface=ether2

# Definir el etiquetado (Tagged/Untagged) de las VLANs en el Trunk
/interface bridge vlan
add bridge=bridge1 tagged=bridge1,ether2 vlan-ids=10
add bridge=bridge1 tagged=bridge1,ether2 vlan-ids=20
add bridge=bridge1 tagged=bridge1,ether2 vlan-ids=30

# Crear las interfaces virtuales VLAN asignadas sobre el Bridge
/interface vlan add interface=bridge1 name=vlan10 vlan-id=10
/interface vlan add interface=bridge1 name=vlan20 vlan-id=20
/interface vlan add interface=bridge1 name=vlan30 vlan-id=30

# Asignar el direccionamiento IP físico a cada interfaz de VLAN
/ip address add address=10.10.10.2/24 interface=vlan10
/ip address add address=10.10.20.2/24 interface=vlan20
/ip address add address=10.10.30.2/24 interface=vlan30

# ==========================================
# PARTE 2: ALTA DISPONIBILIDAD (VRRP Unicast)
# ==========================================

# Crear las instancias VRRPv3 con Prioridad 200 (Master)
/interface vrrp
add interface=vlan10 name=vrrp_web version=3 priority=200
add interface=vlan20 name=vrrp_db version=3 priority=200
add interface=vlan30 name=vrrp_gest version=3 priority=200

# Forzar el modo Unicast apuntando directamente a la IP física del Backup (.3)
/interface/vrrp
set [find name=vrrp_web] remote-address=10.10.10.3
set [find name=vrrp_db] remote-address=10.10.20.3
set [find name=vrrp_gest] remote-address=10.10.30.3

# Asignar las IPs Virtuales (Gateways de la red) a las interfaces VRRP
/ip address add address=10.10.10.1/24 interface=vrrp_web
/ip address add address=10.10.20.1/24 interface=vrrp_db
/ip address add address=10.10.30.1/24 interface=vrrp_gest

# ==========================================
# PARTE 3: AUTOMATIZACIÓN (Servidores DHCP - Pool Inferior)
# ==========================================

# Definir los pools de IP para la primera mitad del rango disponible
/ip pool add name=pool_vlan10 ranges=10.10.10.10-10.10.10.100
/ip pool add name=pool_vlan20 ranges=10.10.20.10-10.10.20.100
/ip pool add name=pool_vlan30 ranges=10.10.30.10-10.10.30.100

# Configurar los parámetros de red a entregar (Gateway apuntando a la IP Virtual .1)
/ip dhcp-server network add address=10.10.10.0/24 gateway=10.10.10.1 dns-server=8.8.8.8,1.1.1.1
/ip dhcp-server network add address=10.10.20.0/24 gateway=10.10.20.1 dns-server=8.8.8.8,1.1.1.1
/ip dhcp-server network add address=10.10.30.0/24 gateway=10.10.30.1 dns-server=8.8.8.8,1.1.1.1

# Levantar y activar los servidores DHCP en las interfaces físicas VLAN
/ip dhcp-server add name=dhcp_vlan10 interface=vlan10 address-pool=pool_vlan10 disabled=no
/ip dhcp-server add name=dhcp_vlan20 interface=vlan20 address-pool=pool_vlan20 disabled=no
/ip dhcp-server add name=dhcp_vlan30 interface=vlan30 address-pool=pool_vlan30 disabled=no

# ==========================================
# PARTE 4: BORDE (Salida a Internet WAN)
# ==========================================
/ip dhcp-client add interface=ether1 disabled=no
/ip firewall nat add chain=srcnat out-interface=ether1 action=masquerade
```
### 1️⃣ MikroTik-3 (VRRP Backup)
```
****# ==========================================
# PARTE 1: ARQUITECTURA DE VLANs (Bridge VLAN Filtering)
# ==========================================
/interface bridge add name=bridge1 vlan-filtering=yes
/interface bridge port add bridge=bridge1 interface=ether2

/interface bridge vlan
add bridge=bridge1 tagged=bridge1,ether2 vlan-ids=10
add bridge=bridge1 tagged=bridge1,ether2 vlan-ids=20
add bridge=bridge1 tagged=bridge1,ether2 vlan-ids=30

/interface vlan add interface=bridge1 name=vlan10 vlan-id=10
/interface vlan add interface=bridge1 name=vlan20 vlan-id=20
/interface vlan add interface=bridge1 name=vlan30 vlan-id=30

# Asignar el direccionamiento IP físico correspondiente al nodo de respaldo (.3)
/ip address add address=10.10.10.3/24 interface=vlan10
/ip address add address=10.10.20.3/24 interface=vlan20
/ip address add address=10.10.30.3/24 interface=vlan30

# ==========================================
# PARTE 2: ALTA DISPONIBILIDAD (VRRP Unicast)
# ==========================================

# Crear las instancias VRRPv3 con Prioridad 100 (Backup)
/interface vrrp
add interface=vlan10 name=vrrp_web version=3 priority=100
add interface=vlan20 name=vrrp_db version=3 priority=100
add interface=vlan30 name=vrrp_gest version=3 priority=100

# Forzar el modo Unicast apuntando directamente a la IP física del Master (.2)
/interface/vrrp
set [find name=vrrp_web] remote-address=10.10.10.2
set [find name=vrrp_db] remote-address=10.10.20.2
set [find name=vrrp_gest] remote-address=10.10.30.2

# Asignar las mismas IPs Virtuales compartidas
/ip address add address=10.10.10.1/24 interface=vrrp_web
/ip address add address=10.10.20.1/24 interface=vrrp_db
/ip address add address=10.10.30.1/24 interface=vrrp_gest

# ==========================================
# PARTE 3: AUTOMATIZACIÓN (Servidores DHCP - Pool Superior)
# ==========================================

# Definir los pools de IP para la segunda mitad del rango para evitar colisiones
/ip pool add name=pool_vlan10 ranges=10.10.10.101-10.10.10.200
/ip pool add name=pool_vlan20 ranges=10.10.20.101-10.10.20.200
/ip pool add name=pool_vlan30 ranges=10.10.30.101-10.10.30.200

/ip dhcp-server network add address=10.10.10.0/24 gateway=10.10.10.1 dns-server=8.8.8.8,1.1.1.1
/ip dhcp-server network add address=10.10.20.0/24 gateway=10.10.20.1 dns-server=8.8.8.8,1.1.1.1
/ip dhcp-server network add address=10.10.30.0/24 gateway=10.10.30.1 dns-server=8.8.8.8,1.1.1.1

/ip dhcp-server add name=dhcp_vlan10 interface=vlan10 address-pool=pool_vlan10 disabled=no
/ip dhcp-server add name=dhcp_vlan20 interface=vlan20 address-pool=pool_vlan20 disabled=no
/ip dhcp-server add name=dhcp_vlan30 interface=vlan30 address-pool=pool_vlan30 disabled=no

# ==========================================
# PARTE 4: BORDE (Salida a Internet WAN)
# ==========================================
/ip dhcp-client add interface=ether1 disabled=no
/ip firewall nat add chain=srcnat out-interface=ether1 action=masquerade
```
# ==========================================
# Políticas de Firewall (Aplicar por igual en AMBOS Routers)
Fragmento de código
# ==========================================
```
# ==========================================
# PARTE 5: POLÍTICAS DE FIREWALL STRICT INTER-VLAN
# ==========================================

# Limpiar filtros previos en la cadena forward
/ip firewall filter remove [find chain=forward]

# Establecer Connection Tracking Eficiente (Aceptar established/related primero)
/ip firewall filter add chain=forward action=fasttrack-connection connection-state=established,related comment="FastTrack"
/ip firewall filter add chain=forward action=accept connection-state=established,related comment="Aceptar establecidas y relacionadas"

# VLAN Gestión (30) - Acceso irrestricto
/ip firewall filter add chain=forward action=accept src-address=10.10.30.0/24 comment="Gestion accede a todo"

# VLAN Web (10) hacia VLAN DB (20) - Solo peticiones al puerto SQL (3306)
/ip firewall filter add chain=forward action=accept src-address=10.10.10.0/24 dst-address=10.10.20.0/24 protocol=tcp dst-port=3306 comment="Web a DB por puerto 3306"

# VLAN Web (10) hacia Internet WAN
/ip firewall filter add chain=forward action=accept src-address=10.10.10.0/24 out-interface=ether1 comment="Web sale a Internet"

# Política por defecto: DROP total para el resto de los flujos cruzados
/ip firewall filter add chain=forward action=drop comment="Bloquear trafico no autorizado inter-VLAN"
```
