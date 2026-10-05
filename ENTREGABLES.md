# Laboratorio 3 - Algoritmos de Enrutamiento

Universidad del Valle de Guatemala - CC3067 Redes - Sección 10 - Grupo 1.

Integrantes:

- Pablo Daniel Barillas Moreno - 22193
- Andrés Rafael Chivalán - 21534

## Entregables

- `latex/PDF1_VLSM_y_Conceptos.tex`: desarrollo detallado para transcribir a mano.
- `latex/PDF2_Evidencias_Packet_Tracer.tex`: resultados reales de la topología.
- `output/pdf/`: documentos PDF compilados.
- `packet-tracer/Lab3_Algoritmos_Enrutamiento_Grupo1.pkt`: proyecto funcional.

## Estado final verificado

- 5 routers, 4 switches, 7 equipos finales y 6 enlaces seriales.
- 9 pruebas representativas con 4/4 respuestas y 0% de pérdida.
- Balanceo OSPF ECMP 50/50 y balanceo EIGRP aproximado 75/25 con `variance 3`.
- Resúmenes `192.168.0.0/23` hacia EIGRP y `172.16.0.0/24` como OSPF E1.
- Tercera VLAN de R-EIGRP-1 resuelta como VLAN 30 de administración en `172.16.0.224/28`.

Los documentos se compilan con XeLaTeX. La guía de compilación, los cálculos, la justificación de decisiones y la auditoría completa se encuentran en `README.md`.
