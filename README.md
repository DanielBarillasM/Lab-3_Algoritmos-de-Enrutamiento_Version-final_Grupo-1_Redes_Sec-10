<div align="center">

# Laboratorio 3 · Algoritmos de Enrutamiento

### OSPF · EIGRP · VLSM · VLANs · Redistribución · Balanceo de carga

**Universidad del Valle de Guatemala · CC3067 Redes · Sección 10 · Grupo 1**

| Integrante | Carné |
|:--|:--:|
| Pablo Daniel Barillas Moreno | 22193 |
| Andrés Rafael Chivalán | 21534 |

</div>

---

## Estado del laboratorio

| Indicador | Resultado verificado |
|:--|:--|
| Estado de la topología | ✅ Saludable: 0 enlaces caídos y 0 IP duplicadas |
| Equipos funcionales | ✅ 5 routers, 4 switches y 7 equipos finales |
| Enlaces seriales | ✅ 6 subredes punto a punto `/30` |
| Conectividad estabilizada | ✅ 9 de 9 pruebas representativas, 4/4 respuestas y 0% de pérdida |
| Balanceo OSPF | ✅ ECMP: dos rutas de métrica 21 y reparto 1:1 |
| Balanceo EIGRP | ✅ `variance 3`: relación métrica 3:1, equivalente a 75/25 |
| Intercambio entre protocolos | ✅ Controlado en R-CENTRAL, con métrica semilla hacia EIGRP y resumen OSPF E1 |
| Sumarización | ✅ Una ruta por dominio: `192.168.0.0/23` y `172.16.0.0/24` |
| Estado técnico | ✅ Implementación y documentación verificadas |

## Topología implementada

<p align="center">
  <img src="latex/figuras/topologia_lab3.png" alt="Topología del Laboratorio 3" width="900">
</p>

La red conserva los cinco routers y las seis conexiones seriales obligatorias. R-CENTRAL participa simultáneamente en OSPF área 0 y EIGRP AS 100, y es el único punto de redistribución.

```text
R-OSPF-1 -------- R-OSPF-2
    \                 /
     \               /
          R-CENTRAL
     /               \
    /                 \
R-EIGRP-1 -------- R-EIGRP-2
```

Los sitios R-OSPF-1 y R-EIGRP-1 utilizan router-on-a-stick; R-OSPF-2 y R-EIGRP-2 poseen una LAN sin VLAN adicional.

## Plan de direccionamiento

### LAN y VLAN

| Dominio | Sitio / segmento | Hosts solicitados | Red asignada | Gateway | Capacidad útil |
|:--|:--|--:|:--|:--|--:|
| OSPF | R-OSPF-2 · usuarios | 200 | `192.168.0.0/24` | `192.168.0.1` | 254 |
| OSPF | R-OSPF-1 · VLAN 10 usuarios | 50 | `192.168.1.0/26` | `192.168.1.1` | 62 |
| OSPF | R-OSPF-1 · VLAN 30 administración | 10 | `192.168.1.64/28` | `192.168.1.65` | 14 |
| EIGRP | R-EIGRP-2 · usuarios | 100 | `172.16.0.0/25` | `172.16.0.1` | 126 |
| EIGRP | R-EIGRP-1 · VLAN 10 usuarios | 60 | `172.16.0.128/26` | `172.16.0.129` | 62 |
| EIGRP | R-EIGRP-1 · VLAN 20 voz | 30 | `172.16.0.192/27` | `172.16.0.193` | 30 |
| EIGRP | R-EIGRP-1 · VLAN 30 administración | 10 | `172.16.0.224/28` | `172.16.0.225` | 14 |

### Enlaces punto a punto

| Enlace | Subred | Extremo A | Extremo B |
|:--|:--|:--|:--|
| R-OSPF-1 ↔ R-OSPF-2 | `10.0.0.0/30` | `10.0.0.1` | `10.0.0.2` |
| R-OSPF-1 ↔ R-CENTRAL | `10.0.0.4/30` | `10.0.0.5` | `10.0.0.6` |
| R-OSPF-2 ↔ R-CENTRAL | `10.0.0.8/30` | `10.0.0.9` | `10.0.0.10` |
| R-EIGRP-1 ↔ R-EIGRP-2 | `10.0.0.12/30` | `10.0.0.13` | `10.0.0.14` |
| R-EIGRP-1 ↔ R-CENTRAL | `10.0.0.16/30` | `10.0.0.17` | `10.0.0.18` |
| R-EIGRP-2 ↔ R-CENTRAL | `10.0.0.20/30` | `10.0.0.21` | `10.0.0.22` |

No existen solapamientos. Los bloques base respetan lo solicitado: `192.168.0.0/20` para el lado OSPF, `172.16.0.0/20` para el lado EIGRP y `10.0.0.0/24` para enlaces entre routers.

## Resultados de enrutamiento

### OSPF área 0: balanceo de costo igual

En R-OSPF-1, el enlace directo hacia R-OSPF-2 tiene costo 20. El camino alternativo por R-CENTRAL suma 10 + 10 = 20. Al agregar el costo de la interfaz de destino, ambas entradas aparecen con métrica 21:

| Destino | Siguiente salto | Interfaz | Métrica | Reparto |
|:--|:--|:--|:--:|:--:|
| `192.168.0.0/24` | `10.0.0.2` | `Serial0/0/0` | 21 | 1 |
| `192.168.0.0/24` | `10.0.0.6` | `Serial0/0/1` | 21 | 1 |

Resultado: **ECMP 50/50** entre el enlace directo y el camino por R-CENTRAL.

### EIGRP AS 100: balanceo desigual 75/25

Para `172.16.0.0/25` desde R-EIGRP-1 se verificaron estas métricas:

| Camino | Métrica total | Distancia reportada | Función |
|:--|--:|--:|:--|
| Directo a R-EIGRP-2 | 2,170,112 | 2,816 | Successor |
| Vía R-CENTRAL | 6,510,336 | 1,683,712 | Feasible successor |

La condición de factibilidad se cumple porque `1,683,712 < 2,170,112`. Además:

```text
6,510,336 / 2,170,112 = 3
```

Con `variance 3`, EIGRP instala ambos caminos. Como el reparto es inversamente proporcional a la métrica, la relación es 3:1: aproximadamente **75% por la ruta principal y 25% por la secundaria**.

### Redistribución y rutas de resumen

| Dirección | Implementación | Estado |
|:--|:--|:--:|
| OSPF → EIGRP | `redistribute ospf 1 metric 1544 20000 255 1 1500` | ✅ |
| Resumen OSPF hacia EIGRP | `192.168.0.0/23` en las interfaces EIGRP de R-CENTRAL | ✅ |
| EIGRP → OSPF | Resumen estático `172.16.0.0/24` a Null0 + `redistribute static metric-type 1 subnets` | ✅ |
| Resumen EIGRP en OSPF | `O E1 172.16.0.0/24`, una sola ruta | ✅ |

Se eligió OSPF **E1** para que la métrica externa incluya también el costo interno hasta R-CENTRAL. Como el IOS 1941 emulado no admite `summary-address` para externas OSPF, R-CENTRAL conserva las rutas EIGRP específicas y crea `172.16.0.0/24` hacia Null0 para redistribuir una sola ruta. Las rutas específicas ganan por coincidencia de prefijo más largo, por lo que el resumen no provoca pérdida de tráfico.

## Pruebas de conectividad

La batería final, después de resolución ARP, produjo los siguientes resultados reales desde el MCP de Packet Tracer:

| Origen | Destino | Resultado |
|:--|:--|:--:|
| PC-O1-USUARIOS | `172.16.0.2` | ✅ 4/4 · 0% pérdida |
| PC-O1-USUARIOS | `172.16.0.226` | ✅ 4/4 · 0% pérdida |
| PC-O1-ADMIN | `172.16.0.194` | ✅ 4/4 · 0% pérdida |
| PC-O2-USUARIOS | `172.16.0.130` | ✅ 4/4 · 0% pérdida |
| PC-O2-USUARIOS | `172.16.0.226` | ✅ 4/4 · 0% pérdida |
| PC-E1-USUARIOS | `192.168.0.2` | ✅ 4/4 · 0% pérdida |
| PC-E1-VOZ | `192.168.1.66` | ✅ 4/4 · 0% pérdida |
| PC-E1-ADMIN | `192.168.0.2` | ✅ 4/4 · 0% pérdida |
| PC-E2-USUARIOS | `192.168.1.2` | ✅ 4/4 · 0% pérdida |

Trazas extremo a extremo verificadas:

```text
OSPF → EIGRP
192.168.1.1 → 10.0.0.6 → 10.0.0.17 → 172.16.0.226

EIGRP → OSPF
172.16.0.1 → 10.0.0.22 → 10.0.0.5 → 192.168.1.66
```