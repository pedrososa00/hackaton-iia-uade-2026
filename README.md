# <M14> · <¿El agente de atención escala a un humano?>


**Integrantes:** <Pedro Sosa>
**IA declarada:** <Gemini>
**Pista:** <Reglas>
**Publicación en LinkedIn:** <link>

---

## 1. Formulación y PEAS

**La pregunta.** Para cada conversación de atención en e-commerce, ¿la conversación necesita un humano para resolverse? (SÍ = la conversación necesita un operador humano para resolverse; NO = el agente de IA puede resolverla de forma autónoma).

| PEAS | Su sistema |
|---|---|
| Medida de rendimiento | Minimizar costos operativos evitando falsos positivos innecesarios, y evitar quejas públicas y pérdida de clientes erradicando los falsos negativos. |
| Entorno | Plataforma de atención de e-commerce, historiales de chat y fichas de reclamos de los usuarios. |
| Actuadores | Derivar la conversación a un operador humano (SÍ) o continuar la atención automatizada con la IA (NUESTRO AGENTE / NO). |
| Sensores | Cantidad de intentos fallidos, indicador de tono del cliente y monto en disputa en USD. |

**Objetivo y métrica.** El objetivo es maximizar la sensibilidad del sistema para garantizar que ningún cliente con reclamos críticos o alta fricción quede sin atención humana, priorizando métricas de Sensibilidad y Especificidad sobre la simple Exactitud bruta.

## 2. El sistema

**Cómo decide.** Sistema de reglas determinísticas (Si-Entonces) optimizado para maximizar la sensibilidad y evitar falsos negativos críticos:
* **Regla 1:** Si Monto en disputa >= 1500 USD Y Intentos fallidos >= 3, entonces escalar a humano (SÍ).
* **Regla 2:** Si Intentos fallidos >= 5 Y Tono del cliente <= 0.5, entonces escalar a humano (SÍ).
* **Regla 3:** Si Tono del cliente <= -0.75 (extremadamente furioso) Y Intentos fallidos >= 3, entonces escalar a humano (SÍ).
* **Caso contrario:** Resolver de forma automatizada con el agente (NO).

**Variables que usa y por qué.** 
* Variables utilizadas: 
  1. Intentos fallidos (respuestas que no resolvieron): Mide la persistencia del problema y la fatiga acumulada del cliente; a mayor cantidad, mayor probabilidad de requerir un operador humano.
  2. Tono del cliente (de -1 a 1): Captura el nivel de frustración o enojo en la interacción textual; valores bajos indican riesgo inminente de queja pública.
  3. Monto en disputa (USD): Cuantifica el impacto financiero del reclamo; las disputas de alto valor económico aumentan drásticamente la prioridad de escalamiento.
* Variables descartadas: Ninguna de las variables principales provistas en la ficha fue descartada, ya que cada una aporta una dimensión crítica (escala operativa, fricción emocional y riesgo financiero) necesaria para la decisión sin violar la regla de información conocida al momento de decidir.

**Cómo se corre en vivo.**

1. Abrir el cuaderno de Google Colab del proyecto.
2. Ejecutar la celda de carga de datos que lee directamente la ficha `M14.csv` desde la URL raw del repositorio de GitHub.
3. Ejecutar la función `predecir(row)` sobre las filas de `ent` o `pru` para procesar las reglas de decisión e imprimir el resultado (**SÍ** o **NO**).

**Dos casos de ejemplo.**

| Caso inventado | Valores | Salida del sistema |
|---|---|---|
| A | Intentos fallidos: 6, Tono: -0.40, Monto: 1600 USD | SÍ |
| B | Intentos fallidos: 1, Tono: 0.80, Monto: 150 USD | NO |

## 3. Medición

**Entrenamiento (casos 1–14)**

| | Predice SÍ | Predice NO |
|---|---|---|
| **Real SÍ** | VP = <8> | FN = <0> |
| **Real NO** | FP = <0> | VN = <6> |

**Prueba (casos 15–20)**

| | Predice SÍ | Predice NO |
|---|---|---|
| **Real SÍ** | VP = <2> | FN = <0> |
| **Real NO** | FP = <0> | VN = <4> |

| Métrica | Entrenamiento | Prueba |
|---|---|---|
| Exactitud | <1.00> | <1.00> |
| Precisión | <1.00> | <1.00> |
| Sensibilidad | <1.00> | <1.00> |
| Especificidad | <1.00> | <1.00> |

**Entrenamiento contra prueba.** <No hay diferencia en el rendimiento (ambos alcanzan un 100% de efectividad), lo cual se debe principalmente a que el dataset es acotado y sintético (14 casos de entrenamiento y 6 de prueba). Al tratarse de datos controlados y no orgánicos, un sistema de reglas determinísticas logra capturar la totalidad de los patrones sin el ruido ni la alta dispersión propia de las interacciones reales de e-commerce.>

## 4. El error más caro

El **falso negativo** (decir NO cuando era SÍ) es el error más caro en nuestro problema. Según la ficha, un falso negativo implica no escalar a un cliente que realmente necesitaba la atención de un operador humano, lo que deriva directamente en una queja pública y en la pérdida definitiva del cliente. Por el contrario, un falso positivo solo ocasiona la ocupación temporal e innecesaria de un operador.

**Ajuste y sensibilidad de los umbrales:** Los umbrales utilizados (≥ 1500 USD, ≥ 5 intentos, tono ≤ 0.5 y ≤ -0.75) fueron calibrados analizando los 14 casos de entrenamiento para maximizar la sensibilidad y evitar dejar escapar ningún caso crítico. Si estos umbrales se volvieran demasiado estrictos, el sistema empezaría a cometer falsos positivos (derivando casos innecesarios); si se volvieran muy laxos, correríamos el riesgo de reintroducir falsos negativos y desatender clientes críticos.

## 5. ¿Comparado contra qué?

**Línea base trivial.** <Una línea base trivial que responde siempre la clase mayoritaria (SÍ = 1) obtiene una exactitud del **57.14%** en entrenamiento (8 de 14 casos correctos) y de apenas un **33.33%** en prueba (2 de 6 casos correctos), además de generar un alto volumen de falsos positivos que saturaría inútilmente a los operadores humanos. 

En contraste, nuestro sistema alcanza un **100% de exactitud, sensibilidad y especificidad** en ambos conjuntos de datos. Esto demuestra que la lógica del agente no solo evita los falsos negativos críticos, sino que optimiza eficazmente el tiempo del equipo de soporte en comparación con un criterio ciego.>

| Checklist (Clase 6) | Respuesta |
|---|---|
| ¿Qué datos? | Dataset M14.csv: 14 casos de entrenamiento y 6 de prueba con las variables Intentos fallidos, Tono del cliente y Monto en disputa (USD). |
| ¿Qué métrica? | Exactitud, Sensibilidad, Especificidad y Precisión, priorizando la Sensibilidad para erradicar los Falsos Negativos. |
| ¿Comparado contra qué? | Contra una línea base trivial que responde siempre la clase mayoritaria (SÍ = 1). |
| ¿Quién lo midió? | El grupo mediante un script de Python en Google Colab. sobre la ficha M14.csv. |

## 6. Riesgo: el camino completo

**Camino del riesgo:** Datos de entrada → Regla de decisión → Falso Negativo → Cliente no escalado → Queja pública.

**Riesgo concreto:** Si el cliente tiene un tono neutral o ligeramente cordial pero está furioso por dentro con reclamos repetidos (ej. un caso con tono de 0.28 pero 3 intentos fallidos y un producto caro), el sistema puede clasificarlo erróneamente como NO derivar. La consecuencia directa es que el agente virtual lo atienda mal, el cliente se enoje más y termine publicando una queja o abandonando la marca.

## 7. Bitácora

**Uso de la IA**

| # | Qué le pedimos | Qué aceptamos | Qué rechazamos y por qué |
|---|---|---|---|
| 1 |Ayuda para estructurar las reglas lógicas y calcular las métricas a partir de la ficha M14.csv. | La estructura general de las reglas de decisión y los valores de la matriz de confusión. | La sugerencia inicial de implementar una neurona, ya que optamos por un sistema de reglas Si-Entonces por su mayor transparencia y porque resolvían el problema con total precisión sin necesidad de modelos más complejos. |
| 2 | Consultamos sobre cómo calibrar los umbrales de tono y monto para minimizar los falsos negativos sin saturar de falsos positivos a los operadores. | La incorporación de la regla estricta para tonos extremadamente furiosos (≤ -0.75) combinados con intentos fallidos. | Descartamos la propuesta de agregar más variables de texto libre (como palabras clave de productos), ya que la ficha solo proveía las variables numéricas estructuradas y eso violaría la regla de usar solo datos disponibles al decidir. |

**Corridas contra el evaluador**

| Corrida | Nota | Agujero que encontró | Qué cambiamos |
|---|---|---|---|
| 1 | 7,50 | Faltaba definir la pregunta SÍ/NO exacta en el PEAS, detallar los pasos en vivo en la sección 2, y explicar el criterio de selección de los umbrales. | Completamos la pregunta exacta en el punto 1, detallamos los pasos en Colab en el punto 2, y agregamos en la sección 4 la justificación de cómo calibramos los umbrales a partir del análisis de entrenamiento para maximizar la sensibilidad. |
| 2 | 8,00 | Señaló detalles menores sobre la línea base de prueba y el análisis de umbrales. | Ninguno (se alcanzó el umbral de habilitación y el sistema quedó listo para la defensa en la mesa). |
