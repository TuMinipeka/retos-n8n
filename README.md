# Automatización de Procesos con n8n

**Reto Práctico en Parejas — Workflows, APIs y Notificaciones Automáticas**

---

<div align="center">

| | |
|---|---|
| **Integrantes** | Diego Mantilla · Daniel Mayorga |
| **Curso** | Automatización de Procesos |
| **Plataforma** | n8n self-hosted |
| **Infraestructura** | Docker |
| **Entrega** | 2025 |

</div>

---

## Descripción General

Este repositorio documenta el desarrollo de dos retos de automatización construidos con **n8n**, una plataforma de automatización de flujos open-source. Ambos retos fueron desplegados sobre **Docker** en entorno local y se integran con **Discord** para el envío de notificaciones automáticas.

El objetivo central es demostrar la capacidad de conectar APIs externas, procesar datos en tiempo real, aplicar lógica condicional y comunicar resultados de forma automática, sin intervención humana.

---

## Tecnologías Utilizadas

<div align="center">

| Tecnología | Rol en el proyecto |
|---|---|
| ![n8n](https://img.shields.io/badge/n8n-EA4B71?style=flat-square&logo=n8n&logoColor=white) | Motor de automatización y orquestación de workflows |
| ![Docker](https://img.shields.io/badge/Docker-2496ED?style=flat-square&logo=docker&logoColor=white) | Contenedorización y despliegue del entorno n8n |
| ![Discord](https://img.shields.io/badge/Discord-5865F2?style=flat-square&logo=discord&logoColor=white) | Canal de notificaciones automáticas vía Webhook |
| ![JavaScript](https://img.shields.io/badge/JavaScript-F7DF1E?style=flat-square&logo=javascript&logoColor=black) | Lógica de procesamiento en nodos Code |
| ![JSON](https://img.shields.io/badge/JSON-000000?style=flat-square&logo=json&logoColor=white) | Formato de comunicación entre la API y los nodos |

</div>

---

## Estructura del Repositorio

```
n8n-retos/
├── README.md                        ← Este archivo
│
├── Conversor Inteligente de Monedas/
│   └── README.md                    ← Documentación completa del Reto 1
│
└── Sistema de Alertas Climáticas/
    └── README.md                    ← Documentación completa del Reto 2
```

---

## Reto 1 — [Conversor Inteligente de Monedas](./Conversor%20Inteligente%20de%20Monedas/README.md)

> Workflow que consulta tasas de cambio en tiempo real, convierte múltiples monedas a COP y envía un reporte diario formateado a Discord de forma completamente automática.

**Estado:** `Completado ✓`

| | |
|---|---|
| **Trigger** | Automático diario a las 8:00 AM |
| **API** | ExchangeRate-API (pública, sin key) |
| **Monedas** | USD, EUR, GBP, MXN → COP |
| **Notificación** | Discord vía Webhook |
| **Extra** | Tendencia, porcentaje de cambio y formato visual |

```
Schedule Trigger  ──►  HTTP Request  ──►  Code  ──►  Discord
```

**[→ Ver documentación completa](./Conversor%20Inteligente%20de%20Monedas/README.md)**

---

## Infraestructura — Docker

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

```bash
docker compose up -d     # Levantar el entorno
docker ps                # Ver estado del contenedor
docker compose down      # Detener todo
docker logs -f n8n       # Ver logs en tiempo real
```

---

## Integrantes

| Nombre | Rol |
|---|---|
| **Diego Mantilla** | Desarrollo del workflow, documentación y pruebas |
| **Daniel Mayorga** | Desarrollo del workflow, integración con Discord y documentación |

---

## Referencias

- [Documentación oficial n8n](https://docs.n8n.io/)
- [n8n en Docker](https://docs.n8n.io/hosting/installation/docker/)
- [ExchangeRate-API](https://www.exchangerate-api.com/docs/free)
- [Discord Webhooks](https://support.discord.com/hc/en-us/articles/228383668)