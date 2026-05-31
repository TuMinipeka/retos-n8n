# 🌦️ Sistema de Alertas Climáticas Automatizadas con n8n

> Automatización inteligente que consulta el clima en tiempo real y envía alertas personalizadas a Discord **únicamente** cuando existe una alta probabilidad de lluvia, eliminando el spam y garantizando notificaciones relevantes.

---

## 📋 Tabla de contenidos

1. [Explicación del problema](#-explicación-del-problema)
2. [Investigación realizada](#-investigación-realizada)
3. [Prerrequisitos](#-prerrequisitos)
4. [Desarrollo paso a paso](#️-desarrollo-paso-a-paso)
5. [Capturas del workflow](#️-capturas-del-workflow)
6. [APIs utilizadas](#-apis-utilizadas)
7. [Resultados obtenidos](#-resultados-obtenidos)
8. [Capturas de Discord](#-capturas-de-discord)
9. [Conclusiones finales](#-conclusiones-finales)

---

## 📝 Explicación del problema

### Contexto

El monitoreo del clima de forma manual es ineficiente y propenso a errores humanos. Las aplicaciones meteorológicas genéricas envían notificaciones sin distinción, lo que genera fatiga de alertas en los usuarios, quienes terminan ignorándolas por completo.

### Problema central

Se necesita un sistema que:
- Consulte el clima automáticamente en horarios programados.
- Evalúe **matemáticamente** si realmente existe riesgo de lluvia.
- Notifique **solo cuando sea necesario**, respetando la atención del usuario.
- Sea capaz de monitorear **múltiples ciudades simultáneamente** sin duplicar la lógica del flujo.

### Objetivo

Construir una automatización robusta con n8n que actúe como un asistente meteorológico inteligente, aplicando una fórmula de ponderación propia para calcular la probabilidad real de lluvia y enviando alertas enriquecidas a Discord únicamente cuando el umbral de riesgo lo justifica.

### Reglas de negocio

| Regla | Descripción |
|---|---|
| **Umbral de alerta** | Solo se notifica si `probabilidad_lluvia >= 70%` |
| **Cero spam** | Si ninguna ciudad supera el umbral, no se envía ningún mensaje |
| **Multi-ciudad** | El flujo evalúa todas las ciudades en una sola ejecución |
| **Horario automático** | El sistema se ejecuta solo, sin intervención manual |

---

## 🔍 Investigación realizada

### APIs meteorológicas evaluadas

Se evaluaron varias APIs antes de seleccionar la solución final:

| API | Ventaja | Desventaja | Seleccionada |
|---|---|---|---|
| **OpenWeatherMap** | Gratuita, baja latencia, respuesta JSON detallada | Límite de 60 req/min en tier free | ✅ Sí |
| WeatherAPI | Interfaz amigable | Menos datos en tier gratuito | ❌ No |
| Open-Meteo | Sin API key | Menor cobertura en ciudades colombianas | ❌ No |

**Elección final: OpenWeatherMap** — provee variables clave como `main.humidity`, `clouds.all`, `main.temp` y `weather[0].description` con la granularidad necesaria para calcular la probabilidad de lluvia de forma matemática.

### Fórmula de probabilidad de lluvia

Tras investigar los factores meteorológicos más correlacionados con la precipitación, se diseñó la siguiente fórmula ponderada:
`
```
probabilidad_lluvia = (Humedad * 0.6) + (Nubosidad * 0.4)
```
![captura evidencia formula del nodo set edit field](./evidencias/probabilidadlluvia.png)

**Justificación de los pesos:**
- **Humedad (60%):** Es el indicador más directo de precipitación inminente. Una humedad superior al 80% es altamente predictiva de lluvia.
- **Nubosidad (40%):** Complementa la humedad. Un cielo completamente nublado sin alta humedad puede no generar lluvia.

> **Ejemplo:** Humedad = 85%, Nubosidad = 90% → `(85 * 0.6) + (90 * 0.4) = 51 + 36 = 87%` ✅ Se envía alerta.

### Procesamiento paralelo en n8n

Se investigó la arquitectura de procesamiento de ítems de n8n. Cuando un nodo genera un array de objetos JSON, los nodos siguientes procesan **cada ítem de forma independiente y secuencial**. Esto permite evaluar N ciudades sin construir bucles visuales complejos, simplemente inyectando un array desde el nodo Code.

### Nodos clave estudiados

- **Schedule Trigger:** Disparador basado en cron para ejecuciones programadas.
- **Code Node:** Ejecución de JavaScript puro para generar o transformar datos.
- **HTTP Request / OpenWeatherMap Node:** Conexión a APIs externas con credenciales.
- **Edit Fields (Set Node):** Creación y transformación de variables calculadas.
- **IF Node:** Lógica condicional booleana para bifurcar el flujo.
- **Discord Node:** Envío de mensajes formateados mediante webhook o bot.

---

## 🛠️ Prerrequisitos

Antes de comenzar, asegúrate de tener lo siguiente:

### Herramientas necesarias

- [n8n](https://n8n.io/) instalado localmente o una cuenta en [n8n.io Cloud](https://app.n8n.io/)
- Cuenta gratuita en [OpenWeatherMap](https://openweathermap.org/api)
- Servidor de Discord con permisos para crear bots o webhooks

### Cómo obtener la API Key de OpenWeatherMap

1. Regístrate en [https://openweathermap.org/](https://openweathermap.org/)
2. Ve a tu perfil → **My API Keys**
3. Copia la API Key predeterminada (o crea una nueva)
4. Espera hasta 2 horas para que la key se active por primera vez

### Cómo configurar el bot de Discord

**Opción A — Webhook (más simple):**
1. Ve a tu servidor de Discord → canal destino → ⚙️ **Editar canal**
2. Sección **Integraciones** → **Webhooks** → **Nuevo Webhook**
3. Asigna un nombre y foto al bot, luego copia la **URL del Webhook**

**Opción B — Bot con token (usada en este proyecto):**
1. Ve al [Discord Developer Portal](https://discord.com/developers/applications)
2. Crea una nueva aplicación → sección **Bot** → **Add Bot**
3. Copia el **Token** del bot
4. En **OAuth2 → URL Generator**, selecciona los scopes `bot` y el permiso `Send Messages`
5. Usa la URL generada para invitar el bot a tu servidor

---

## ⚙️ Desarrollo paso a paso

### Paso 1 — Crear un nuevo workflow en n8n

1. Abre n8n y haz clic en **"New Workflow"**
2. Asigna el nombre: `Sistema de Alertas Climáticas`
3. Guarda el workflow con `Ctrl + S` antes de comenzar

---

### Paso 2 — Nodo Schedule Trigger (Disparador automático)

Este nodo inicia el flujo automáticamente según un horario definido.

1. Haz clic en el **"+"** para agregar el primer nodo
2. Busca y selecciona **"Schedule Trigger"**
3. Configura los parámetros:
   - **Trigger Interval:** `Days`
   - **Days Between Triggers:** `1`
   - **Trigger at Hour:** `7` (7:00 AM)
   - **Trigger at Minute:** `0`
4. Esto ejecutará el flujo todos los días a las 7:00 AM

> **Alternativa con cron:** Si prefieres expresión cron, usa `0 7 * * *` (cada día a las 7 AM).

---

### Paso 3 — Nodo Code (Generar lista de ciudades)

Este nodo inyecta el listado de ciudades que serán consultadas.

1. Agrega un nodo **"Code"** conectado al Schedule Trigger
2. Configura:
   - **Language:** `JavaScript`
   - **Mode:** `Run Once for All Items`
3. Pega el siguiente código:

```javascript
const ciudades = [
  { ciudad: "Giron,CO" },
  { ciudad: "Bucaramanga,CO" },
  { ciudad: "Bogota,CO" }
];

return ciudades.map(item => ({ json: item }));
```

4. Haz clic en **"Execute Node"** para verificar que genera 3 ítems correctamente

> **Para agregar más ciudades:** Añade objetos al array con el formato `{ ciudad: "NombreCiudad,CodigoPais" }`. Los códigos de país siguen el estándar ISO 3166-1 alpha-2 (CO = Colombia, US = Estados Unidos, MX = México).

---

### Paso 4 — Nodo OpenWeatherMap (Consulta de clima)

Este nodo consulta la API por cada ciudad generada en el paso anterior.

1. Agrega el nodo **"OpenWeatherMap"** conectado al nodo Code
2. Configura las credenciales:
   - Haz clic en **"Create new credential"**
   - Pega tu **API Key** de OpenWeatherMap
   - Guarda con el nombre `OpenWeatherMap API`
3. Configura los parámetros del nodo:
   - **Operation:** `Current Weather`
   - **City:** `{{ $json.ciudad }}` (expresión dinámica)
   - **Language:** `Spanish` (opcional, para la descripción del clima)
4. Ejecuta el nodo y verifica que devuelve datos para cada ciudad

**Variables clave que usaremos del JSON de respuesta:**

```
main.humidity    → Humedad relativa (0-100%)
clouds.all       → Porcentaje de nubosidad (0-100%)
main.temp        → Temperatura actual en °C
name             → Nombre oficial de la ciudad
weather[0].description → Descripción textual del clima
```

---

### Paso 5 — Nodo Edit Fields (Calcular probabilidad de lluvia)

Este nodo transforma los datos crudos de la API y calcula nuestra métrica de riesgo.

1. Agrega el nodo **"Edit Fields (Set)"** conectado al nodo OpenWeatherMap
2. En la sección de campos, agrega los siguientes campos :

{{ ($json.main.humidity * 0.6) + ($json.clouds.all * 0.4) }}

3. Ejecuta el nodo y verifica que `probabilidad_lluvia` aparece calculada correctamente

---

### Paso 6 — Nodo IF (Filtro lógico de alertas)

Este nodo bifurca el flujo: solo las ciudades con alta probabilidad de lluvia avanzan hacia la notificación.

1. Agrega el nodo **"IF"** conectado al nodo Edit Fields
2. Configura la condición:
   - **Value 1:** `{{ $json.probabilidad_lluvia }}` (tipo: Number)
   - **Operation:** `Greater than or equal to`
   - **Value 2:** `70`
3. El nodo creará dos ramas:
   - ✅ **True:** ciudades con probabilidad ≥ 70% → continúan al nodo Discord
   - ❌ **False:** ciudades con probabilidad < 70% → rama vacía (fin del flujo)

> **La rama False se deja intencionalmente vacía.** Esto implementa la política de "cero spam": si ninguna ciudad supera el umbral, el workflow termina sin enviar ninguna notificación.

---

### Paso 7 — Nodo Discord (Envío de la alerta)

Este nodo envía el mensaje formateado al canal de Discord configurado.

1. Agrega el nodo **"Discord"** conectado a la rama **True** del nodo IF
2. Configura las credenciales:
   - Selecciona **"Bot Token"** o **"Webhook"** según tu configuración
   - Ingresa el token del bot o la URL del webhook
3. Configura el mensaje con el siguiente contenido (sintaxis Markdown de Discord):

```
🌧️ **ALERTA DE LLUVIA — {{ $json.ciudad }}**

📊 **Probabilidad de lluvia:** {{ Math.round($json.probabilidad_lluvia) }}%
🌡️ **Temperatura actual:** {{ $json.temperatura }}°C
💧 **Humedad:** {{ $json.humedad }}%
☁️ **Nubosidad:** {{ $json.nubosidad }}%
🌤️ **Condición:** {{ $json.descripcion }}

⚠️ Se recomienda llevar paraguas hoy.
```

4. Asegúrate de seleccionar el **canal correcto** donde el bot tiene permisos de escritura
5. Ejecuta el nodo para enviar un mensaje de prueba

---

### Paso 8 — Activar el workflow

1. Revisa que todos los nodos estén conectados correctamente en la vista del flujo
2. Haz clic en el toggle **"Active"** (esquina superior derecha) para activar el workflow
3. El sistema comenzará a ejecutarse automáticamente según el horario configurado

> Para una prueba inmediata, haz clic en **"Execute Workflow"** sin necesidad de esperar al horario programado.

---

## 🖼️ Capturas del workflow

### Vista general del flujo completo

Arquitectura completa del workflow mostrando la cadena de nodos: Schedule Trigger → Code → OpenWeatherMap → Edit Fields → IF → Discord.

![Vista general del workflow](./evidencias/flujogeneral.png)

---

### Configuración del nodo IF (Filtro lógico)

Detalle de la condición configurada en el nodo IF: `probabilidad_lluvia >= 70`.

![Configuración del nodo IF](./evidencias/parametrosif.png)

---

### Salida del nodo Code (Array de ciudades)

JSON generado por el nodo Code con los 3 ítems que representan las ciudades a monitorear.

![Salida del nodo Code](./evidencias/itemsnode.png)

---

## 🔌 APIs utilizadas

### 1. OpenWeatherMap API — Current Weather Data

| Detalle | Valor |
|---|---|
| **Endpoint** | `https://api.openweathermap.org/data/2.5/weather` |
| **Método** | `GET` |
| **Autenticación** | API Key (query param `appid`) |
| **Tier utilizado** | Free (hasta 1,000 llamadas/día) |
| **Formato de respuesta** | JSON |

**Parámetros de la petición:**

```
q       = {ciudad},{pais}   → Ej: "Bucaramanga,CO"
appid   = {tu_api_key}
units   = metric            → Temperatura en Celsius
lang    = es                → Descripciones en español
```

**Fragmento de respuesta JSON:**

```json
{
  "name": "Bucaramanga",
  "main": {
    "temp": 28.4,
    "humidity": 82
  },
  "clouds": {
    "all": 75
  },
  "weather": [
    {
      "description": "nubes dispersas"
    }
  ]
}
```

![Configuración del nodo OpenWeatherMap](./evidencias/weatherconfi.png)

---

### 2. Discord API — Bot de notificaciones

| Detalle | Valor |
|---|---|
| **Método de integración** | Bot Token (nodo nativo de n8n) |
| **Permisos requeridos** | `Send Messages`, `Embed Links` |
| **Formato de mensaje** | Markdown de Discord |
| **Trigger** | Condicional (solo si `probabilidad_lluvia >= 70`) |

El bot fue creado desde el [Discord Developer Portal](https://discord.com/developers/applications) e invitado al servidor con los permisos mínimos necesarios para operar.

![Bot de Discord configurado](./evidencias/botdiscordcreado.png)

---

## 📊 Resultados obtenidos

### Eficiencia del sistema de filtrado

El nodo IF funciona como un guardián que garantiza la relevancia de cada notificación:

| Escenario | Humedad | Nubosidad | Probabilidad | Resultado |
|---|---|---|---|---|
| Día lluvioso | 85% | 90% | **87%** | ✅ Alerta enviada |
| Día nublado | 65% | 80% | **71%** | ✅ Alerta enviada |
| Día parcialmente nublado | 55% | 60% | **57%** | ❌ Sin notificación |
| Día despejado | 30% | 10% | **22%** | ❌ Sin notificación |

### Procesamiento multi-ciudad independiente

El sistema evaluó las 3 ciudades configuradas en una sola ejecución. Gracias al motor de ítems de n8n, cada ciudad se procesa de forma aislada:

- Si **Bogotá** supera el umbral y **Girón** no → solo se notifica Bogotá
- Si **ninguna** supera el umbral → no se envía ningún mensaje
- Si **todas** superan el umbral → se envían 3 mensajes independientes

### Mensajería enriquecida

Los mensajes enviados a Discord incluyen todas las variables dinámicas extraídas del JSON de OpenWeatherMap, entregando contexto completo en cada alerta sin repetición de información.

### Cumplimiento de reglas de negocio

| Regla | Estado |
|---|---|
| Umbral de alerta al 70% | ✅ Implementado y verificado |
| Cero spam en días despejados | ✅ Rama False vacía confirmada |
| Procesamiento multi-ciudad | ✅ 3 ciudades en una ejecución |
| Ejecución automática programada | ✅ Cron configurado a las 7 AM |

---

## 📱 Capturas de Discord

### Evidencia de funcionamiento — Alerta recibida en Discord

Mensaje de alerta recibido en el canal de Discord cuando la probabilidad de lluvia superó el umbral del 70%. Se puede observar el formato enriquecido con las variables dinámicas de temperatura, humedad, nubosidad y probabilidad calculada.

![Alerta recibida en Discord](./evidencias/botdiscordfuncionando.png)

---

## 💡 Conclusiones finales

### Lo que aprendimos

1. **El poder del procesamiento por ítems de n8n:** No fue necesario construir bucles visuales complejos para manejar múltiples ciudades. Al generar un array desde el nodo Code, n8n itera automáticamente cada ítem a través de los nodos siguientes, reduciendo significativamente la complejidad visual del flujo.

2. **La importancia de transformar los datos antes de aplicar lógica:** El nodo Edit Fields actúa como una capa de normalización crítica. Sin él, sería imposible aplicar la fórmula de probabilidad o estructurar el mensaje de Discord de forma limpia. Siempre se deben transformar los datos crudos antes de tomar decisiones condicionales.

3. **La política de "cero spam" es una decisión de diseño, no una limitación:** Dejar la rama *False* del nodo IF intencionalmente vacía es una decisión deliberada de arquitectura. Un sistema que notifica cuando no hay nada importante que comunicar pierde la confianza del usuario.

4. **Las fórmulas propias superan los campos predefinidos:** OpenWeatherMap provee un campo `pop` (probability of precipitation) en su endpoint de pronóstico, pero no en el de clima actual. Diseñar una fórmula propia a partir de variables correlacionadas (humedad + nubosidad) demostró ser una solución efectiva y personalizable.

5. **n8n como plataforma de producción real:** Este proyecto no es solo un ejercicio académico. La combinación de Schedule Trigger + procesamiento condicional + notificación a mensajería instantánea es un patrón de arquitectura usado en sistemas de alertas empresariales reales.

### Posibles mejoras futuras

- Integrar el endpoint de **pronóstico de 5 días** (`/forecast`) para alertar con anticipación.
- Agregar un nodo **Spreadsheet** o **Airtable** para registrar un histórico de las alertas enviadas.
- Configurar un **canal de Telegram** como canal alternativo de notificación.
- Añadir lógica de **severidad** (moderada, alta, crítica) con distintos emojis y colores según el porcentaje calculado.
- Implementar **geolocalización dinámica** para consultar ciudades basadas en la ubicación de los usuarios registrados.

---

> **Desarrollado por:** Daniel Santiago Mayorga 
> **Stack:** n8n · OpenWeatherMap API · Discord Bot API  

