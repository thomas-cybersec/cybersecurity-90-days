# Día 5 – QUIC/HTTP3 en Wireshark: tráfico moderno y desafíos SOC

## Qué capturé
Realicé una captura de tráfico en Wireshark mientras navegaba sitios modernos durante varios minutos, principalmente YouTube, Google y otras páginas actuales. El objetivo fue observar tráfico QUIC (HTTP/3) en un entorno real y entender cómo se diferencia del tráfico tradicional basado en TCP/TLS desde una perspectiva SOC.

Durante la captura se observó tráfico UDP en el puerto 443, característico del protocolo QUIC, utilizado ampliamente por servicios modernos como YouTube debido a su menor latencia y mejor rendimiento en streaming.

## Diferencia TCP vs QUIC
El tráfico TCP tradicional utiliza handshake TCP (SYN → SYN/ACK → ACK) seguido de handshake TLS para establecer una conexión segura. Esto genera mayor latencia inicial y más pasos antes de comenzar la transmisión de datos.

QUIC, en cambio, utiliza UDP puerto 443 e integra el cifrado TLS desde el inicio. Esto elimina la necesidad de múltiples handshakes separados, reduce la latencia y permite un flujo de datos más continuo, algo especialmente útil en streaming y navegación moderna.

En Wireshark esto se observa como:
- TCP: handshake visible, tráfico segmentado y reconexiones más frecuentes.
- QUIC: flujo más continuo sobre UDP, handshake propio cifrado y menor latencia aparente.

## Por qué QUIC complica la inspección
QUIC integra cifrado desde el inicio de la comunicación, lo que dificulta la inspección profunda del contenido por parte de herramientas de seguridad tradicionales. Además, al usar UDP evita algunas técnicas clásicas de inspección basadas en TCP.

Esto implica que muchas soluciones SOC deben apoyarse más en:
- metadatos (IPs, timing, tamaño de paquetes)
- patrones de comportamiento
- correlación con logs y telemetría adicional

Más que en la inspección directa del contenido.

## Cuándo podría ser sospechoso
Aunque QUIC es normal en tráfico moderno, puede resultar sospechoso cuando:

- existe tráfico QUIC constante hacia IPs desconocidas
- hay comunicaciones en horarios atípicos (ej: madrugada)
- se observa volumen inusual de tráfico saliente
- dominios o IPs no reconocidos sin contexto legítimo

Esto podría indicar:
- C2 malware cifrado
- exfiltración de datos
- beaconing disfrazado como tráfico web normal

## Conclusión SOC
QUIC representa la evolución del tráfico web hacia comunicaciones más rápidas y cifradas. Si bien mejora la performance y seguridad para el usuario, también introduce desafíos para monitoreo SOC, ya que reduce la visibilidad del contenido.

El análisis debe enfocarse cada vez más en metadatos, comportamiento de red y correlación de eventos para detectar actividad potencialmente maliciosa.
