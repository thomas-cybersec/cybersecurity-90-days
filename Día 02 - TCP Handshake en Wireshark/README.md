# Día 2 – TCP en Wireshark: handshake, RST y retransmisiones

## 1) Qué capturé
Realicé una captura corta en Wireshark mientras navegaba tráfico normal (Google, YouTube y una web adicional).
El objetivo fue practicar análisis TCP en modo SOC para identificar el inicio de conexiones, cierres abruptos y señales de problemas en red.

## 2) Handshake identificado
Para identificar el inicio de una conexión TCP usé estos filtros:

- `tcp.flags.syn == 1` → se observan paquetes con flag SYN (intentos de iniciar una conexión TCP). Interpretación: **“se está abriendo una conexión”**.
- `tcp.flags.syn == 1 && tcp.flags.ack == 0` → muestra SYN puros (primer paquete del handshake). Interpretación: **“este host está intentando conectarse a otro host”**.

Ejemplo real del paquete elegido (inicio de conexión):
- Frame: **1007**
- IP origen → destino: `10.0.2.15` → `142.251.128.32`
- Puerto origen → destino: `39114` → `80`
- Conclusión: el puerto destino es **80**, por lo tanto la conexión corresponde a **HTTP** (no TLS / no cifrado).

## 3) Conexión completa (Follow TCP Stream)
Sobre el mismo paquete ejecuté **Follow → TCP Stream** para ver la conversación completa.

Observaciones:
- Se ven **datos**, no solo handshake.
- Se identifica tráfico **HTTP**.
- La conexión **dura poco**, ya que es un request pequeño que recibe respuesta y termina (no es una sesión larga tipo streaming).

Evidencia observada:
- `POST ...`
- headers (User-Agent, Host, Content-Type)
- respuesta `HTTP/1.1 200 OK`
- `Content-Length: 83`

Interpretación SOC: esto representa una conexión normal (handshake + intercambio de datos + respuesta exitosa).

## 4) RST observado
Para detectar cortes abruptos usé:
- `tcp.flags.reset == 1` → se ven paquetes TCP con flag RST, indicando que una conexión fue cerrada o rechazada de forma abrupta. Interpretación: **“conexión cortada de golpe”**.

Ejemplo observado:
- `10.0.2.15` → `34.160.x.x`

Posibles causas normales:
- cierre abrupto de la página
- puerto cerrado
- firewall cortó la conexión

Nota: un RST **no indica ataque automáticamente**, pero es una señal importante cuando ocurre repetidamente o en patrones raros.

## 5) Retransmisiones
Para identificar retransmisiones usé:
- `tcp.analysis.retransmission` → muestra paquetes reenviados porque no hubo ACK/respuesta a tiempo (pérdida o congestión). Interpretación: **“problemas de red o paquetes perdidos”**.

Resultado:
- ✅ No se detectaron retransmisiones en esta captura, lo cual sugiere tráfico normal y red estable.

## Filtro extra (HTTPS)
- `tcp.port == 443` → permite aislar tráfico HTTPS/TLS (TCP/443), útil para analizar tráfico web seguro.
