# VPN IPSec IKEv1 Site-to-Site con Túnel GRE

---

## Información del proyecto

**Autor:** Michael David Robles Fermín  
**Matrícula:** 2025-0845  
**Asignatura:** Seguridad de Redes  
**Repositorio:** https://github.com/iClexi/VPN-IKEv1-Tunnel-GRE  
**Video demostrativo:**  
https://youtu.be/opI3pn_s4aI  
**Documentación técnica profesional:**  
[Ver documentación técnica profesional](docs/MichaelRobles_2025-0845_Documentacion-Tecnica-Profesional-VPN-IKEv1-Tunnel-GRE_P3.pdf)  
**Ubicación directa:** `docs/MichaelRobles_2025-0845_Documentacion-Tecnica-Profesional-VPN-IKEv1-Tunnel-GRE_P3.pdf`

---

## Vista general de la topología

```text
PC-A --- SW1 --- R1 --- ISP --- R2 --- SW2 --- PC-B
```

![Topología general](images/MichaelRobles_2025-0845_01-Topologia-General-VPN-IKEv1-Tunnel-GRE_P3.png)

R1 y R2 son los peers VPN. El ISP solo permite conectividad WAN entre ambos extremos.

---

## Descripción general

Esta práctica implementa una VPN Site-to-Site punto a punto usando un túnel GRE protegido con IPSec IKEv1. GRE crea el túnel lógico entre R1 y R2, mientras que IPSec cifra y protege el tráfico GRE.

```text
GRE crea el túnel.
IPSec protege el túnel.
IKEv1 negocia la seguridad de IPSec.
```

---

## Tutorial rápido de configuración

Para configurar el túnel GRE en R1 se usó `interface Tunnel0`, se asignó la IP `172.16.84.1/30`, se configuró `tunnel source GigabitEthernet0/0` y se apuntó el túnel hacia `20.25.8.50` con `tunnel destination`.

```cisco
interface Tunnel0
 description TUNEL-GRE-HACIA-R2
 ip address 172.16.84.1 255.255.255.252
 tunnel source GigabitEthernet0/0
 tunnel destination 20.25.8.50
 tunnel mode gre ip
 no shutdown
```

Para enviar el tráfico hacia la LAN remota se usó una ruta estática hacia el otro extremo del túnel:

```cisco
ip route 192.168.84.0 255.255.255.0 172.16.84.2
```

Para proteger el tráfico GRE con IPSec se configuró IKEv1, una clave precompartida, un transform-set en modo transport, una ACL que permite GRE entre las WAN y un crypto map aplicado en la interfaz WAN.

```cisco
crypto isakmp policy 10
 encr aes 256
 hash sha
 authentication pre-share
 group 14
 lifetime 86400

crypto isakmp key ITLA20250845 address 20.25.8.50
crypto ipsec transform-set TS-IKEV1-GRE esp-aes 256 esp-sha-hmac
 mode transport
access-list 120 permit gre host 20.25.8.46 host 20.25.8.50
crypto map MAP-IKEV1-GRE 10 ipsec-isakmp
 set peer 20.25.8.50
 set transform-set TS-IKEV1-GRE
 match address 120
interface GigabitEthernet0/0
 crypto map MAP-IKEV1-GRE
```

---

## Comparación con los modos anteriores

```text
Policy-Based:
ACL entre LANs + crypto map. No usa Tunnel0.

Route-Based:
Tunnel0 tipo VTI + IPSec profile + rutas.

Tunnel GRE:
Tunnel0 GRE + rutas + crypto map protegiendo GRE entre las WAN.
```

---

## Evidencias

### Topología general

![Topología general](images/MichaelRobles_2025-0845_01-Topologia-General-VPN-IKEv1-Tunnel-GRE_P3.png)

### Interfaces de R2

![R2 show ip interface brief](images/MichaelRobles_2025-0845_02-R2-Show-IP-Interface-Brief_P3.png)

### Tunnel0 en R2

![R2 show interface tunnel0](images/MichaelRobles_2025-0845_03-R2-Show-Interface-Tunnel0_P3.png)

### Ping del túnel y ruta hacia LAN B

![R1 ping tunnel y ruta](images/MichaelRobles_2025-0845_04-R1-Ping-Tunnel-y-Route-LAN-B_P3.png)

### IKEv1 levantado

![R1 show crypto isakmp sa](images/MichaelRobles_2025-0845_05-R1-Show-Crypto-ISAKMP-SA_P3.png)

### IPSec en R2

![R2 show crypto ipsec sa](images/MichaelRobles_2025-0845_06-R2-Show-Crypto-IPSec-SA_P3.png)

### IPSec en R1

![R1 show crypto ipsec sa](images/MichaelRobles_2025-0845_07-R1-Show-Crypto-IPSec-SA_P3.png)

### Ping PC-A hacia PC-B

![PC-A ping PC-B](images/MichaelRobles_2025-0845_08-PC-A-Ping-PC-B_P3.png)

### Ping PC-B hacia PC-A

![PC-B ping PC-A](images/MichaelRobles_2025-0845_09-PC-B-Ping-PC-A_P3.png)

---

## Resultado esperado

La VPN debe aparecer activa con:

```cisco
show crypto isakmp sa
```

Resultado esperado:

```text
QM_IDLE ACTIVE
```

El túnel debe aparecer en estado `up/up` y los contadores de `show crypto ipsec sa` deben aumentar después de los pings entre PC-A y PC-B.
