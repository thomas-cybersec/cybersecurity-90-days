# Day 01 - Wireshark: Tráfico normal (YouTube + sitio web River Plate)

## Objetivo
Capturar tráfico real de navegación normal y reconocer protocolos comunes en un entorno real.

---

## Actividad realizada
Abrí y navegué en:
- **YouTube**
- **Sitio web de River Plate**

Luego realicé una captura en Wireshark para analizar el tráfico generado.

---

## Protocolos observados
Durante la captura identifiqué:

- **DNS**
- **TLS**
- **TCP**
- **HTTP**
- **QUIC**

---

## Análisis por protocolo

### DNS
- Mi dispositivo consulta al **Gateway / DNS** qué IP corresponde al dominio que quiero visitar.
- Se observa **Query + Response**, por eso se ven ambas IPs.
- Transporte: **UDP / Puerto 53**

---

### TLS
- Se establece una conexión segura entre mi dispositivo y el servidor.
- Se observa el **TLS Handshake** (negociación del cifrado) y luego intercambio de información cifrada.
- Transporte: **TCP / Puerto 443 (HTTPS)**

---

### TCP
- Se observa el **3-way handshake** inicial.
- Luego puede aparecer el **handshake TLS** y el intercambio normal de datos.
- Predomina tráfico con **TCP/443** durante la navegación segura.

---

### HTTP
- Se observan paquetes relacionados con **OCSP**, que son consultas para validar certificados TLS.
- Aunque usa HTTP (puerto 80), **no significa tráfico inseguro**, sino parte del funcionamiento de HTTPS.
- Transporte: **TCP / Puerto 80**

---

### QUIC
- Se observa que **no hay TCP handshake** ni TLS handshake por separado.
- QUIC realiza handshake e intercambio cifrado directamente.
- Normal que haya más tráfico del servidor hacia mi PC al consumir contenido (ej: video).
- Transporte: **UDP / Puerto 443**

---

## 3 IPs destino observadas (servidores)
Ejemplo de tráfico TLS:

- **Origen (cliente):** `10.0.2.15` (mi dispositivo)
- **Destino (servidor):** `142.251.128.40` (servidor web)


Otras IPs destino observadas durante navegación:
- IP 2: `172.67.71.67`  IP pública asociada a **Cloudflare** (CDN / protección). Esto indica que el sitio web utiliza Cloudflare como intermediario para acelerar el contenido y proteger el servidor real (WAF / anti-DDoS / proxy).
- IP 3: `3.160.119.67` IP pública utilizada como **edge** para entrega de contenido web. Probablemente pertenece a infraestructura tipo **CDN/Edge Network**, que acelera la carga entregando contenido desde nodos cercanos al usuario.

> Nota: completar con 2 IPs más que aparezcan con frecuencia en tu captura.

---

## Paquete elegido (TLS) - Datos clave
Ejemplo paquete TLS:

- **IP origen:** `10.0.2.15`
- **IP destino:** `142.251.128.40`

- **Puerto origen:** `40342` (puerto efímero del cliente)
- **Puerto destino:** `443` (HTTPS)

---

## Qué significa puerto 443 vs 80
- **443 = HTTPS (cifrado con TLS)**
- **80 = HTTP (sin cifrar)**

---

## Aprendizaje clave: MAC vs IP
- **MAC Address → Capa 2 (Enlace):** identifica dispositivo en la red local.
- **IP Address → Capa 3 (Red):** identifica un host en la red / internet.
- **Puertos TCP/UDP → Capa 4 (Transporte):** identifican servicios/aplicaciones.
- **TLS → Capa 7 (Aplicación):** protocolo de cifrado para conexiones seguras.

---


