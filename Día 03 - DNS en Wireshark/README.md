# Día 3 – DNS en Wireshark: tráfico normal, NXDOMAIN y dominios sospechosos

## 1) Qué capturé
Realicé una captura de tráfico DNS en Wireshark mientras navegaba por sitios web comunes durante una sesión normal de uso.  
El objetivo fue analizar cómo se ve el tráfico DNS legítimo, identificar dominios frecuentes y entender eventos NXDOMAIN desde una perspectiva SOC.

---

## 2) Cómo se ve una query DNS y una respuesta
Para analizar consultas DNS utilicé los siguientes filtros:

- `dns` → ver todo el tráfico DNS
- `dns.flags.response == 0` → ver **queries DNS**
- `dns.flags.response == 1` → ver **responses DNS**

Ejemplo de query DNS observada:

- **IP origen → IP destino:**  
  `10.0.2.15` (mi dispositivo) → `192.168.0.1` (gateway/DNS)
- **Acción:** consulta DNS para resolver un dominio
- **Dominio consultado (dns.qry.name):** `madridistas.com`
- **Transaction ID:** `0xfaf0`

Inicialmente se observó una consulta **AAAA (IPv6)** que no devolvió dirección IP.  
El servidor respondió con un **SOA**, indicando que no existe registro AAAA para ese dominio.


Luego, al consultar el registro **A (IPv4)** con el filtro: 
dns.qry.name == "madridistas.com" && dns.qry.type == 1


Se obtuvo la **respuesta DNS correcta**, resolviendo el dominio a la IP:

- **23.2.189.250**

Esto representa el flujo DNS normal:  
query → response → resolución del dominio a una IP.

---

## 3) Dominios más consultados (Top 5)
Para identificar los dominios más consultados utilicé tráfico DNS completo y un análisis desde Kali Linux con `tshark`.

Comando utilizado en kali:
tshark -r dns_dia3.pcapng -Y "dns.flags.response==0" -T fields -e dns.qry.name | sort | uniq -c | sort -nr | head -20

Los **5 dominios más consultados** fueron:

- `www.googletagmanager.com`
- `www.google.com`
- `fonts.gstatic.com`
- `i.ytimg.com`
- `googleads.g.doubleclick.net`

Interpretación SOC:
Estos dominios son **normales y esperables**, asociados a servicios de Google, YouTube, fuentes web y publicidad.  
Su alta frecuencia es típica de navegación legítima.

---

## 4) NXDOMAIN: qué significa y por qué aparece

Para detectar errores DNS utilicé el filtro:
dns.flags.rcode != 0


Resultado:
- **No se observaron respuestas NXDOMAIN** durante el período de captura.

### ¿Qué es NXDOMAIN?
NXDOMAIN es una respuesta DNS que indica que el **dominio consultado no existe** en el sistema DNS.

### Causas normales de NXDOMAIN:
- Error tipográfico del usuario (dominio mal escrito)
- Consultas de trackers o scripts obsoletos
- Dominios bloqueados por adblock, firewall o antivirus

### Causas sospechosas (SOC):
- Malware probando dominios generados automáticamente (**DGA**)
- Intentos de comunicación con servidores **C2** que ya no están activos

---

## 5) Señales de dominio sospechoso (SOC thinking)
Durante el análisis no se detectaron dominios con patrones sospechosos.

Indicadores que se buscaron:
- Nombres de dominio largos y aleatorios
- Alta entropía (caracteres sin sentido)
- Consultas repetidas a dominios inexistentes
- Patrones compatibles con DGA o tunneling DNS

Conclusión:
El tráfico DNS observado corresponde a **navegación normal**, sin indicios de actividad maliciosa.

---

## Conclusión (Día 3)
- Analicé tráfico DNS normal y entendí el flujo query/response.
- Identifiqué dominios legítimos y frecuentes.
- Comprendí el significado y la importancia de NXDOMAIN.
- Practiqué detección básica de dominios sospechosos con mentalidad SOC.


