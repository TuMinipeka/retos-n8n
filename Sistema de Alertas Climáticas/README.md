# retos-n8n
# 🌦️ Alertas Climáticas Automatizadas con n8n

## 📝 Explicación del problema
* **Contexto:** El monitoreo manual del clima es ineficiente y las alertas genéricas suelen ser ignoradas por los usuarios.
* **Objetivo:** Construir una automatización robusta, cercana a un entorno de producción real, para la toma de decisiones climáticas.
* **Regla de negocio principal:** El sistema debe consultar el clima en horarios específicos y enviar alertas a Discord **solo** si hay una alta probabilidad de lluvia. Si el día está despejado, no debe ejecutarse ninguna acción (política de cero spam).
* **Escalabilidad:** El flujo debe ser capaz de evaluar múltiples ciudades de forma simultánea e independiente en una sola ejecución.

## 🔍 Investigación realizada
* **APIs Meteorológicas:** Selección de OpenWeatherMap debido a su baja latencia y la estructura detallada de su respuesta JSON (incluyendo variables clave como temperatura, nubosidad y humedad).
* **Nodos Condicionales (IF Nodes):** Estudio de la lógica booleana en n8n para desviar el flujo de datos y detener procesos de forma controlada.
* **Cálculo de Probabilidad:** Implementación de una fórmula ponderada matemática matemática para calcular el riesgo de precipitación: `(Humedad * 0.6) + (Nubosidad * 0.4)`.
* **Procesamiento de múltiples ítems:** Análisis de la iteración automática de n8n sobre matrices de datos (arrays) para ejecutar procesos paralelos sin necesidad de construir bucles visuales complejos.

## ⚙️ Desarrollo paso a paso
1. **Schedule Trigger:** Configuración de un disparador basado en tiempo (cron job) para ejecutar el monitoreo automáticamente cada mañana.
2. **Nodo Code (Lista de ciudades):** Inyección de código JavaScript en modo *Run Once for All Items* para generar un array JSON con las ciudades objetivo (ej. `Girón, CO`, `Bucaramanga, CO`, `Bogotá, CO`).
3. **Nodo OpenWeatherMap:** Configuración de variables dinámicas (`{{ $json.ciudad }}`) para consultar la API por cada ítem generado en el nodo anterior.
4. **Nodo Edit Fields:** Creación de la variable calculada `probabilidad_lluvia`, aplicando la fórmula matemática de ponderación sobre los datos crudos de la API.
5. **Nodo IF (Filtro Lógico):** Condición estricta configurada en `probabilidad_lluvia >= 70`. La rama *False* se dejó intencionalmente vacía para detener la ejecución y cumplir la regla de "cero spam".
6. **Nodo Discord:** Conectado a la rama *True* para dar formato a la alerta usando sintaxis Markdown, inyectando los valores de probabilidad, temperatura y nombre de la ciudad.

## 🖼️ Capturas del workflow
* **Vista General del Flujo:** ![Captura del workflow general](./evidencias/flujogeneral.png)
* **Configuración de parametros configuracion If:** ![Captura de confi if node](./evidencias/parametrosif.png)
* **Generación de Ítems:** ![Captura del json del node code](./evidencias/itemsnode.png)

## 🔌 APIs utilizadas
* **OpenWeatherMap API (Current Weather Data):** Para la ingesta de datos meteorológicos en tiempo real (nodos `main.humidity`, `clouds.all`, `main.temp`, `name`).
![Captura de confi OpenWeatherApi](./evidencias/weatherconfi.png)

* **Discord API (vía nodo nativo de n8n):** Para el enrutamiento y entrega del payload con la notificación final.
![Captura de bot discord usado](./evidencias/botdiscordcreado.png)



## 📊 Resultados obtenidos
* **Eficiencia de Notificaciones:** Cumplimiento total de la regla de negocio; el sistema omite el envío de mensajes si la probabilidad es inferior al umbral del 70%.

* **Procesamiento Paralelo Simulado:** Aislamiento exitoso de datos por ciudad. Si la condicion se cumple en Bogotá pero no en Girón, el nodo IF filtra los ítems y notifica exclusivamente el caso positivo.
* **Mensajería Enriquecida:** Extracción correcta de variables dinámicas del JSON para entregar alertas personalizadas y visualmente atractivas.

## 📱 Capturas de Discord
* **Evidencia de Funcionamiento:** ![Captura de bot discord usado](./evidencias/botdiscordfuncionando.png)


## 💡 Conclusiones finales
* La combinación de nodos IF junto con el motor de procesamiento por ítems de n8n permite construir arquitecturas eficientes y escalables sin recurrir a flujos redundantes.
* La normalización y transformación de datos crudos (vía Edit Fields) es un paso fundamental antes de aplicar compuertas lógicas condicionales.
* El proyecto no solo cumple con los requisitos base de automatización condicional, sino que implementa buenas prácticas de desarrollo visual al manejar arrays de datos dinámicos.