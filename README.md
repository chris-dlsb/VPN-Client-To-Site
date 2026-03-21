# VPN-Client-To-Site

# VPN Remote Access (Client-to-Site) con FortiGate

**Estudiante:** Cristopher de los Santos  
**Matrícula:** 2024-1414

## 1. Objetivo de la Red
El objetivo de esta asignación es implementar una conexión segura mediante un túnel **IPsec Remote Access (Client-to-Site)**. Se busca permitir que un usuario remoto (Windows 10 con FortiClient) acceda de forma segura a los recursos internos de la infraestructura corporativa a través de un proveedor de servicios de internet (ISP) simulado, garantizando la confidencialidad de los datos y la segmentación del tráfico mediante políticas de firewall.

## 2. Topología de Red
La infraestructura diseñada en GNS3 para esta práctica consta de:
* **FortiGate-NGFW:** Firewall perimetral encargado de la terminación del túnel, autenticación de usuarios y filtrado de paquetes.
* **Router IOU (ISP):** Actúa como el core de transporte (Internet) entre el cliente y el firewall.
* **Cliente Remoto (Windows 10):** Host externo que utiliza el software **FortiClient** para establecer la conexión VPN.
* **Nodos Finales (VPCS):** Hosts internos ubicados en diferentes segmentos de red (LAN A y LAN B) para validar la conectividad.

### Tabla de Direccionamiento (Basada en Matrícula 2024-1414)

| Dispositivo | Interfaz | Dirección IP | Descripción |
| :--- | :--- | :--- | :--- |
| **FortiGate** | Port1 (WAN) | 200.24.14.2/30 | Enlace de salida al ISP |
| **FortiGate** | Port2 (LAN A) | 10.24.14.1/24 | Gateway Red de Matrícula A |
| **FortiGate** | Port3 (LAN B) | 10.14.14.1/24 | Gateway Red de Matrícula B |
| **ISP (IOU)** | e0/0 | 200.24.14.1/30 | Gateway WAN del FortiGate |
| **Windows 10** | VPN Virtual | 172.16.14.10 | IP asignada mediante DHCP-over-IPsec |
| **VPC 1** | Ethernet0 | 10.24.14.10/24 | Host Interno en LAN A |
| **VPC 2** | Ethernet0 | 10.14.14.11/24 | Host Interno en LAN B |

### foto de la topologia
<img width="1178" height="656" alt="image" src="https://github.com/user-attachments/assets/ea67f088-0afc-4e7d-a02a-849ff7ee29da" />


---

## 3. Configuraciones Realizadas

### A. Configuración de Interfaces y Seguridad Local
Se asignaron IPs estáticas a las interfaces físicas. Se creó un grupo de usuarios (**VPN_Remote_Group**) y un usuario local (`cristopher`) para la autenticación **XAuth**. Se configuró un servidor DHCP interno en la fase de la VPN para la entrega automática de parámetros de red a los clientes remotos.
<img width="522" height="203" alt="image" src="https://github.com/user-attachments/assets/0c7472e8-8636-43eb-9159-098750a7bf50" />
<img width="223" height="153" alt="image" src="https://github.com/user-attachments/assets/d54897d9-cf1e-4bf2-98ce-bf12b9c65c4b" />

### B. Fase 1 y Fase 2 de IPsec (Ajustes de Interoperabilidad)
Para asegurar la estabilidad del túnel en el entorno virtualizado y cumplir con los requisitos técnicos, se aplicaron los siguientes parámetros:
* **Algoritmos de Cifrado:** DES y SHA-1 (seleccionados para compatibilidad de licencia y rendimiento).
<img width="522" height="187" alt="image" src="https://github.com/user-attachments/assets/8bda0cae-3530-4dad-aa9b-9860587fdabb" />
<img width="517" height="339" alt="image" src="https://github.com/user-attachments/assets/a15c8de8-91cd-49ff-a578-2bec5294ca6e" />

* **Modo de Intercambio:** Aggressive Mode con el uso de **Local ID** (`VPN_ITLA`).
* **Diffie-Hellman:** Grupo 5 (1536-bit), optimizado para la capacidad de cómputo del firewall virtual.*

  <img width="522" height="228" alt="image" src="https://github.com/user-attachments/assets/efe69fd4-9db0-44e0-b8a6-89cf4ccad272" />


### C. Políticas de Firewall y Ruteo Inverso
Se implementaron políticas de seguridad para permitir el tráfico desde la interfaz virtual de la VPN hacia las interfaces físicas `port2` y `port3`. Se habilitó **Split-Tunneling** en la configuración del portal para que el cliente Windows solo dirija el tráfico de las redes `10.24.14.0/24` y `10.14.14.0/24` a través del túnel, manteniendo su salida a Internet local sin afectación.
<img width="694" height="274" alt="image" src="https://github.com/user-attachments/assets/617668c3-91a1-4aad-9121-377e73010281" />

### configuracion de forticlient en windows 10
aqui hacemos la configuracion al igual que en nuestro fortigate para evitar que no se pueda conectar

### autenticacion y fase 1
<img width="694" height="274" alt="image" src="https://github.com/user-attachments/assets/6b535336-26ba-4507-8718-545005778176" />

### fase 2
<img width="694" height="274" alt="image" src="https://github.com/user-attachments/assets/26a9d1f0-a12e-4284-89ee-9fa19486e79c" />

para que funcione debe de haber conexion al isp y completamente

### conexion al cliente windows
<img width="694" height="274" alt="image" src="https://github.com/user-attachments/assets/79e48c7c-6200-4624-8752-e468e0c5bc20" />
