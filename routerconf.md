##Bloque A: Creación del Bridge y las VLANs (Capa 2)
/interface bridge add name=bridge1 vlan-filtering=yes
/interface bridge port add bridge=bridge1 interface=ether2

/interface bridge vlan
add bridge=bridge1 tagged=bridge1,ether2 vlan-ids=10
add bridge=bridge1 tagged=bridge1,ether2 vlan-ids=20
add bridge=bridge1 tagged=bridge1,ether2 vlan-ids=30

/interface vlan
add interface=bridge1 name=vlan10_web vlan-id=10
add interface=bridge1 name=vlan20_db vlan-id=20
add interface=bridge1 name=vlan30_gest vlan-id=30
##Bloque B: Direccionamiento IP y redundancia VRRP (Prioridad Máxima: 200)
/ip address
add address=10.10.10.2/24 interface=vlan10_web
add address=10.10.20.2/24 interface=vlan20_db
add address=10.10.30.2/24 interface=vlan30_gest

/interface vrrp
add interface=vlan10_web name=vrrp_web priority=200 vrid=10
add interface=vlan20_db name=vrrp_db priority=200 vrid=20
add interface=vlan30_gest name=vrrp_gest priority=200 vrid=30

/ip address
add address=10.10.10.1/24 interface=vrrp_web
add address=10.10.20.1/24 interface=vrrp_db
add address=10.10.30.1/24 interface=vrrp_gest
##Bloque C: Salida a Internet (WAN y NAT)
/ip dhcp-client add disabled=no interface=ether1
/ip firewall nat add action=masquerade chain=srcnat out-interface=ether1
----------------------------------------------
MikroTik-3 ( Respaldo/Backup)
# Cambiamos las IPs físicas a las del Router B (.3)
/ip address set [find address="10.10.10.2/24"] address=10.10.10.3/24
/ip address set [find address="10.10.20.2/24"] address=10.10.20.3/24
/ip address set [find address="10.10.30.2/24"] address=10.10.30.3/24

# Bajamos la prioridad de VRRP a 100 para que sea Backup
/interface vrrp set [find name="vrrp_web"] priority=100
/interface vrrp set [find name="vrrp_db"] priority=100
/interface vrrp set [find name="vrrp_gest"] priority=100
