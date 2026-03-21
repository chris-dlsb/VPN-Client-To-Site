### Configuración del FortiGate (NGFW)
## Interfaces de Red
config system interface
    edit "port1"
        set alias "WAN"
        set ip 200.24.14.2 255.255.255.252
        set allowaccess ping
    next
    edit "port2"
        set alias "LAN_A"
        set ip 10.24.14.1 255.255.255.0
        set allowaccess ping
    next
    edit "port3"
        set alias "LAN_B"
        set ip 10.14.14.1 255.255.255.0
        set allowaccess ping
    next
end

### Usuario y Grupo para VPN

config user local
    edit "cristopher"
        set type password
        set passwd itla2026
    next
end

config user group
    edit "VPN_Remote_Group"
        set member "cristopher"
    next
end

### Fase 1 y Fase 2 de la VPN (IPsec)

# Fase 1: Parámetros del túnel
config vpn ipsec phase1-interface
    edit "vpn_client_site"
        set type dynamic
        set interface "port1"
        set mode aggressive
        set psksecret itla2026
        set localid "VPN_ITLA"
        set proposal des-sha256
        set dhgrp 5
        set authusrgrp "VPN_Remote_Group"
        set split-tunneling enable
        set ipv4-split-tunnel-include "Mis_Redes_Internas"
    next
end

# Fase 2: Selectores de tráfico
config vpn ipsec phase2-interface
    edit "vpn_client_site_p2"
        set phase1name "vpn_client_site"
        set proposal des-sha256
        set pfs disable  # Vital para entorno GNS3
        set keylifeseconds 3600
        set src-subnet 0.0.0.0 0.0.0.0
        set dst-subnet 0.0.0.0 0.0.0.0
    next
end

### Objetos de Red y Rutas
config firewall address
    edit "Red_LAN_A"
        set subnet 10.24.14.0 255.255.255.0
    next
    edit "Red_LAN_B"
        set subnet 10.14.14.0 255.255.255.0
    next
end

config firewall addrgrp
    edit "Mis_Redes_Internas"
        set member "Red_LAN_A" "Red_LAN_B"
    next
end

config router static
    edit 1
        set gateway 200.24.14.1
        set device "port1"
    next
end

### Políticas de Firewall

config firewall policy
    edit 1
        set name "VPN_to_Internal"
        set srcintf "vpn_client_site"
        set dstintf "port2" "port3"
        set srcaddr "all"
        set dstaddr "all"
        set action accept
        set schedule "always"
        set service "ALL"
    next
    edit 2
        set name "Internal_to_VPN"
        set srcintf "port2" "port3"
        set dstintf "vpn_client_site"
        set srcaddr "all"
        set dstaddr "all"
        set action accept
        set schedule "always"
        set service "ALL"
    next
end

### Configuración del Router ISP (Cisco IOU)

conf t
interface Ethernet0/0
 ip address 200.24.14.1 255.255.255.252
 no shutdown
exit

# Ruta para que el ISP sepa cómo volver al FortiGate (si es necesario)
ip route 0.0.0.0 0.0.0.0 200.24.14.2
write

### Configuración de los Nodos Finales (VPCS)

ip 10.24.14.10 255.255.255.0 10.24.14.1
save

### VPC 2 (Conectado al Port 3):

ip 10.14.14.11 255.255.255.0 10.14.14.1
save

## Configuración del Cliente (FortiClient Windows)
Para que coincida con lo anterior, los parámetros en el software son:

Remote Gateway: 200.24.14.2

Authentication Method: Pre-Shared Key (itla2026)

IKE Mode: Aggressive

Local ID: VPN_ITLA

Phase 1 Encryption: DES / SHA256 / DH Group 5

Phase 2 Encryption: DES / SHA256 / PFS OFF


