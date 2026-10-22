# Paso 1 — Tabla de datos expuestos

| Dato encontrado | Fuente | ¿Para qué le sirve al atacante? | Contramedida propuesta |
|---|---|---|---|
| Nombres, cargos y correos directos de los 3 socios (Marta, Andrés, Lucía) | 1 | Permite phishing dirigido (spear phishing) suplantando a un cargo concreto | Formación anti-phishing; no publicar organigrama completo con roles sensibles |
| Software de gestión identificado: ERP GestiCloud y portal de clientes propio | 1 | Permite buscar vulnerabilidades conocidas de esas plataformas o crear portales falsos de login | Revisar exposición pública de qué tecnologías se usan; no detallarlas en la web |
| Servidor Ubuntu 8.04 + Apache + vsftpd (versión muy antigua) | 2 | Software obsoleto con vulnerabilidades públicas y documentadas, explotable directamente | Actualizar/migrar el servidor; aplicar parches; dar de baja software fuera de soporte |
| Windows 7 en 3 puestos | 2 | Sistema operativo sin soporte de seguridad, vulnerable a malware y exploits conocidos | Actualizar a un SO con soporte activo |
| Email personal de la registrante del dominio (marta.ferrer.personal@...) | 3 | Vector de phishing fuera del entorno corporativo, donde hay menos controles | Usar privacidad WHOIS / datos de contacto corporativos, no personales |
| Teléfono personal de Marta Ferrer en el WHOIS | 3 | Permite vishing (llamadas fraudulentas) o smishing (SMS) directo | Activar privacidad WHOIS |
| IPs de todos los servicios: web, portal, NAS, "pruebas", mail | 4 | Mapa completo de la infraestructura para escanear puertos y buscar fallos en cada sistema, incluido un servidor de "pruebas" potencialmente menos protegido | Segmentar red, no exponer NAS ni entornos de pruebas a Internet, usar firewall |
| SPF configurado como v=spf1 +all | 4 | Política SPF totalmente abierta: permite que cualquiera envíe correos falsificando el dominio (email spoofing) | Corregir SPF a política restrictiva e implementar DKIM/DMARC |
| Ausencia de DKIM y DMARC | 4 | Sin estos, el spoofing de correo del dominio no se detecta ni se bloquea, ideal para BEC (fraude del CEO) | Implementar DKIM y DMARC con política de rechazo |
| Foto de la mesa de trabajo con nota adhesiva legible | 5 | Seguridad física/OSINT: una nota pegada en el monitor suele contener contraseñas, PINs o accesos | Política de "mesa limpia"; nunca anotar credenciales en notas visibles |
| Marca y modelo de la impresora visible en la foto | 5 | Permite buscar vulnerabilidades específicas del dispositivo (muchas impresoras en red son vectores de entrada) | No publicar fotos detalladas del equipamiento interno |
| Publicación de teletrabajo en cafetería con wifi público, mencionando proyecto concreto del Ayuntamiento | 5 | Indica momento y lugar donde el objetivo usa red no segura (ataques de interceptación en wifi público) y confirma un proyecto/cliente real para pretextos de ingeniería social | Formación sobre uso de VPN en redes públicas; evitar publicar ubicación/actividad en tiempo real |
| Respuesta automática que indica quién gestiona pagos y cambios bancarios, más fechas exactas de ausencia | 6 | Información perfecta para fraude del CEO/BEC: suplantar a Marta mientras está fuera y pedir a Lucía un cambio de cuenta bancaria o pago urgente | No detallar en el fuera de oficina quién gestiona pagos ni dar fechas exactas; verificar cambios bancarios siempre por canal alternativo (llamada) |
| Horario de cierre total de la oficina (1-15 agosto) publicado en la web | 1 y 6 | Indica fechas exactas en que el local está vacío, útil para intrusión física | No publicar fechas exactas de cierre total en web pública |

# Paso 2 — Categorías de riesgo combinadas (resumen, no plan operativo)
## Suplantación de email
Si el dominio no aplica DMARC o está en p=none, y SPF/DKIM no están alineados o son permisivos, como atacantes podemos intentar enviar correos que parezcan del dominio legítimo, usar un dominio visualmente similar o manipular Reply-To. Nuestro objetivo sería hacernos pasar por la dirección, proveedor o finanzas para pedir un pago, cambiar datos bancarios o conseguir credenciales.
Señales: fallos SPF/DKIM/DMARC en reportes, dominios parecidos registrados, correos con urgencia anómala, cambios de Reply-To, peticiones de pago fuera del procedimiento.
Control: DMARC en cuarentena/rechazo progresivo, alineación SPF/DKIM, monitorización de dominios similares, doble verificación de pagos por canal alternativo.

# Paso 3 — Las 3 contramedidas más importantes

1. **Corregir SPF/DKIM/DMARC.** Es la vulnerabilidad más crítica porque habilita el spoofing de email, que es la puerta de entrada a casi todos los demás ataques de ingeniería social detectados (BEC, phishing dirigido). Corregirla cierra de un solo cambio el vector más peligroso y más barato de explotar.
2. **Actualizar el servidor (Ubuntu 8.04/Apache/vsftpd) y los equipos con Windows 7.** Es la única vía de compromiso puramente técnico y directo de la infraestructura; software sin soporte tiene vulnerabilidades públicas y conocidas, lo que lo hace explotable sin necesidad de engañar a nadie.
3. **No publicar información operativa sensible** (quién gestiona pagos, fechas exactas de cierre total, ubicación/actividad en tiempo real, fotos detalladas del puesto de trabajo). Se elige esta sobre otras contramedidas "menores" porque es la que alimenta directamente el pretexto de ingeniería social: sin este contexto, un email falso o una llamada fraudulenta resulta mucho menos convincente, aunque el atacante ya tenga el resto de los datos.

Se priorizan estas tres porque cada una corta una *categoría* de riesgo distinta (email, técnico, humano) en vez de ser contramedidas redundantes dentro de la misma categoría.
