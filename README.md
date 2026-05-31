<div align="center">

```
╔══════════════════════════════════════════════════════════════════════════════╗
║                                                                              ║
║    A U T O M A T I Z A C I O N   D E   P R O C E S O S   C O N   n 8 n   ║
║    ────────────────────────────────────────────────────────────────────     ║
║             Reto Practico en Parejas  ·  CampusLands 2025                   ║
║                                                                              ║
╚══════════════════════════════════════════════════════════════════════════════╝
```

<br>

[![n8n](https://img.shields.io/badge/n8n-Workflow_Engine-EA4B71?style=for-the-badge&logo=n8n&logoColor=white)](https://n8n.io/)
[![Docker](https://img.shields.io/badge/Docker-Self_Hosted-2496ED?style=for-the-badge&logo=docker&logoColor=white)](https://docs.docker.com/)
[![Discord](https://img.shields.io/badge/Discord-Notificaciones-5865F2?style=for-the-badge&logo=discord&logoColor=white)](https://discord.com/)
[![JavaScript](https://img.shields.io/badge/JavaScript-ES6-F7DF1E?style=for-the-badge&logo=javascript&logoColor=black)](https://developer.mozilla.org/es/docs/Web/JavaScript)
[![Retos](https://img.shields.io/badge/Retos_Completados-2_/_2-4CAF50?style=for-the-badge)](.)

<br>

*Dos retos de automatizacion construidos con n8n, desplegados en Docker*
*e integrados con Discord para el envio de notificaciones en tiempo real.*

<br>

| | |
|:---:|:---:|
| **Integrantes** | Diego Mantilla · Daniel Mayorga |
| **Curso** | Automatizacion de Procesos |
| **Plataforma** | n8n self-hosted |
| **Infraestructura** | Docker |
| **Entrega** | 2025 |

</div>

<br>

---

## Descripcion General

Construimos este repositorio para documentar el desarrollo de dos retos de automatizacion con **n8n**, una plataforma open-source de orquestacion de flujos. Ambos retos corren sobre **Docker** en entorno local y se comunican con **Discord** para entregar notificaciones automaticas sin ninguna intervencion manual.

El hilo conductor de los dos proyectos es el mismo: conectar APIs externas, procesar datos en tiempo real, aplicar logica condicional y entregar resultados de forma completamente autonoma.

<br>

---

## Stack tecnologico

<div align="center">

| Tecnologia | Rol |
|:----------:|:----|
| [![n8n](https://img.shields.io/badge/n8n-EA4B71?style=flat-square&logo=n8n&logoColor=white)](https://n8n.io/) | Motor de automatizacion y orquestacion visual de workflows |
| [![Docker](https://img.shields.io/badge/Docker-2496ED?style=flat-square&logo=docker&logoColor=white)](https://docs.docker.com/) | Contenedorizacion y despliegue del entorno n8n |
| [![Discord](https://img.shields.io/badge/Discord-5865F2?style=flat-square&logo=discord&logoColor=white)](https://discord.com/) | Canal de notificaciones automaticas vía Webhook y Bot Token |
| [![JavaScript](https://img.shields.io/badge/JavaScript-F7DF1E?style=flat-square&logo=javascript&logoColor=black)](https://developer.mozilla.org/) | Logica de transformacion y calculo en nodos Code |
| [![JSON](https://img.shields.io/badge/JSON-000000?style=flat-square&logo=json&logoColor=white)](https://www.json.org/) | Formato de comunicacion entre APIs y nodos del flujo |

</div>

<br>

---

## Estructura del repositorio

```
retos-n8n/
│
├── README.md                             ← Este archivo
│
├── Conversor Inteligente de Monedas/
│   ├── README.md                         ← Documentacion completa del Reto 1
│   └── evidencias/                       ← Capturas del workflow y Discord
│
└── Sistema de Alertas Climaticas/
    ├── README.md                         ← Documentacion completa del Reto 2
    └── evidencias/                       ← Capturas del workflow y Discord
```

<br>

---

## Los Retos

<table>
<tr>

<td width="50%" valign="top">

### Reto 1 — Conversor de Monedas

[![Estado](https://img.shields.io/badge/Estado-Completado-4CAF50?style=flat-square)](./Conversor%20Inteligente%20de%20Monedas/README.md)

Workflow que consulta tasas de cambio en tiempo real, convierte multiples monedas a COP y envia un reporte diario formateado a Discord de forma completamente automatica.

```
Schedule Trigger
      │
      ▼
 HTTP Request
 ExchangeRate-API
      │
      ▼
   Code Node
 Calculos y formato
      │
      ▼
   Discord
 Reporte diario
```

| Campo | Detalle |
|:------|:--------|
| Trigger | Diario · 8:00 AM |
| API | ExchangeRate-API |
| Monedas | USD EUR GBP MXN → COP |
| Canal | Discord Webhook |

**[→ Ver documentacion completa](./Conversor%20Inteligente%20de%20Monedas/README.md)**

</td>

<td width="50%" valign="top">

### Reto 2 — Alertas Climaticas

[![Estado](https://img.shields.io/badge/Estado-Completado-4CAF50?style=flat-square)](./Sistema%20de%20Alertas%20Clim%C3%A1ticas/README.md)

Workflow que monitorea el clima de multiples ciudades, aplica una formula propia de probabilidad de lluvia y envia alertas a Discord **unicamente** cuando el riesgo supera el 70%.

```
Schedule Trigger
      │
      ▼
  Code Node
  3 ciudades
      │
      ▼
OpenWeatherMap
  x ciudad
      │
      ▼
 Edit Fields
 prob_lluvia
      │
      ▼
  IF >= 70?
  ┌───┴───┐
 SI      NO
  │       │
Discord  [fin]
```

| Campo | Detalle |
|:------|:--------|
| Trigger | Diario · 7:00 AM |
| API | OpenWeatherMap |
| Ciudades | Giron · BGA · BOG |
| Canal | Discord Bot Token |

**[→ Ver documentacion completa](./Sistema%20de%20Alertas%20Clim%C3%A1ticas/README.md)**

</td>

</tr>
</table>

<br>

---

## Infraestructura — Docker

<details>
<summary><strong>Ver configuracion del entorno</strong></summary>

<br>

Ambos workflows corren sobre una instancia de n8n levantada con Docker Compose en entorno local. La configuracion define la zona horaria en America/Bogota para que los Schedule Triggers se ejecuten en hora colombiana.

```yaml
version: "3.8"

services:
  n8n:
    image: docker.n8n.io/n8nio/n8n
    container_name: n8n
    restart: unless-stopped
    ports:
      - "5678:5678"
    environment:
      - N8N_HOST=localhost
      - N8N_PORT=5678
      - N8N_PROTOCOL=http
      - GENERIC_TIMEZONE=America/Bogota
      - TZ=America/Bogota
    volumes:
      - n8n_data:/home/node/.n8n

volumes:
  n8n_data:
```

**Comandos utiles:**

```bash
docker compose up -d      # Levantar el entorno en segundo plano
docker ps                 # Verificar que el contenedor esta corriendo
docker compose down       # Detener y eliminar los contenedores
docker logs -f n8n        # Ver logs en tiempo real
```

> n8n queda disponible en `http://localhost:5678` una vez levantado el contenedor.

</details>

<br>

---

## Equipo

<table>
<tr>

<td width="50%" align="center">

**Diego Mantilla**

[![GitHub](https://img.shields.io/badge/GitHub-181717?style=flat-square&logo=github&logoColor=white)](https://github.com/TuMinipeka)

Desarrollo del workflow · Documentacion · Pruebas

*Reto 1 — Conversor de Monedas*

</td>

<td width="50%" align="center">

**Daniel Mayorga**

[![GitHub](https://img.shields.io/badge/GitHub-181717?style=flat-square&logo=github&logoColor=white)](https://github.com/TuMinipeka)

Desarrollo del workflow · Integracion con Discord · Documentacion

*Reto 2 — Alertas Climaticas*

</td>

</tr>
</table>

<br>

---

## Referencias

| Recurso | Enlace |
|:--------|:-------|
| Documentacion oficial n8n | [docs.n8n.io](https://docs.n8n.io/) |
| n8n en Docker | [Guia de instalacion](https://docs.n8n.io/hosting/installation/docker/) |
| ExchangeRate-API | [Documentacion gratuita](https://www.exchangerate-api.com/docs/free) |
| OpenWeatherMap API | [Current Weather Data](https://openweathermap.org/api) |
| Discord Webhooks | [Guia oficial](https://support.discord.com/hc/en-us/articles/228383668) |

<br>

---

```
════════════════════════════════════════════════════════════════════════════════
  Diego Mantilla · Daniel Mayorga  |  CampusLands  |  Automatizacion con n8n
════════════════════════════════════════════════════════════════════════════════
```
