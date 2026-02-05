# Día 4 – HTTP vs HTTPS en Wireshark: qué se ve y qué no

## 1) Captura realizada
Realicé una captura dirigida en Wireshark durante aproximadamente 2–3 minutos navegando tanto sitios HTTP como HTTPS para comparar qué información se puede observar en cada caso. Para generar tráfico HTTP en claro utilicé páginas como `http://neverssl.com` y `http://httpbin.org`, y luego navegué normalmente en sitios HTTPS como Google y YouTube. El objetivo fue entender qué datos quedan expuestos en tráfico sin cifrar y qué información permanece visible incluso cuando el tráfico está cifrado.

---

## 2) Tráfico HTTP en claro (qué información se expone)
Aplicando el filtro `http` y utilizando **Follow TCP Stream**, se puede observar todo el contenido en texto plano. Esto incluye URLs completas, headers HTTP, método de petición (GET/POST), User-Agent del navegador, host solicitado e incluso credenciales si el sitio no utiliza cifrado.

Como prueba práctica accedí a una página de login (`http://pruebadeuso.com/index.php?r=site/login`) y pude observar en Wireshark información sensible como usuario, ID y contraseña en texto claro. Esto demuestra que HTTP no protege la confidencialidad de los datos y que un atacante con acceso al tráfico podría interceptar fácilmente información sensible.

Además, dentro del tráfico HTTP se pueden identificar claramente:
- navegador utilizado (Firefox según User-Agent)
- host solicitado (`pruebadeuso.com`)
- método HTTP utilizado (`POST` para envío de login)

---

## 3) Tráfico HTTPS (qué se oculta y qué metadatos quedan)
Para analizar tráfico cifrado utilicé el filtro `tls`. En conexiones HTTPS el contenido no es legible porque está cifrado mediante TLS, por lo que no se pueden ver URLs completas, headers ni credenciales.

Sin embargo, permanecen visibles ciertos metadatos importantes para análisis de seguridad:
- direcciones IP origen y destino
- puertos utilizados (normalmente 443)
- tamaño de paquetes y timing de comunicación
- en algunos casos el SNI (Server Name Indication), que puede revelar el dominio solicitado dependiendo de la configuración del servidor.

Esto explica por qué en análisis SOC muchas veces se trabaja con metadatos cuando el contenido está cifrado.

---

## 4) Importancia para SOC y seguridad
Comprender la diferencia entre HTTP y HTTPS es fundamental para análisis de tráfico y detección de amenazas. HTTP expone completamente la comunicación, permitiendo ver credenciales y datos sensibles, lo que representa un riesgo de seguridad importante. HTTPS, en cambio, protege el contenido mediante cifrado, pero aún permite analizar patrones de comunicación, metadatos y comportamientos sospechosos.

Este conocimiento es clave para tareas de monitoreo SOC, análisis forense de red y detección temprana de actividad maliciosa.

---

## Conclusión (Día 4)
- HTTP permite ver toda la información en texto claro, incluyendo posibles credenciales.
- HTTPS cifra el contenido pero deja visibles metadatos útiles para análisis de seguridad.
- El análisis de tráfico cifrado se basa principalmente en patrones, IPs, puertos y comportamiento.
- Entender estas diferencias es esencial para monitoreo de redes, privacidad y detección de amenazas.
