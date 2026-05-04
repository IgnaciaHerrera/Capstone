# Propuesta de Proyecto Capstone — Asesor de Estrategia F1

## Problema

Los equipos de Fórmula 1 no solo se preguntan si un piloto terminará entre los 10 primeros. Durante un fin de semana de carrera, la pregunta más útil es:

> **Si elegimos una estrategia de parada en boxes diferente, ¿cómo cambia el resultado esperado de la carrera?**

Tu equipo construirá un **Asesor de Estrategia de Carrera**: un sistema de ML reproducible que compare escenarios de estrategia y traduzca la salida del modelo en una recomendación para la mesa de estrategia de una carrera.

---

## Alcance Requerido

Todos los equipos deben completar el asesor a nivel de carrera (Capa 1). Los equipos fuertes pueden intentar la Capa 2.

### Capa 1 — Asesor de Estrategia a Nivel de Carrera

#### Dataset
- **f1_strategy_race_level.csv** (2,447 filas · temporadas 2019–2024 · 1 fila por piloto-carrera)

#### Unidad de Predicción
Un piloto en una carrera.

#### Progresión de Objetivos (Obligatoria en Todos los Hitos)

| Hito | Requisito de Objetivo |
|------|----------------------|
| Hito 1 | Todos los equipos usan `is_top10`. El objetivo uniforme en toda la cohorte permite una comparación directa con la línea base del docente. |
| Hito 2 | Expansión obligatoria: añade al menos un objetivo adicional de `{is_top5, is_top3, finish_position, points}` y compara el valor de decisión entre los dos objetivos. |
| Informe Final | Libre: los equipos pueden mantener su par de objetivos del Hito 2 o expandir si fortalece la recomendación de estrategia. |

#### Expectativa Mínima de Modelado

Construye un modelo calibrado y consciente del sesgo que estime P(finish_position ≤ k) y compare al menos dos escenarios de estrategia.

**Ejemplo de escenario:**

Para un piloto que comienza en el centro del campo en Monza, compara una estrategia de una parada M-H contra una estrategia de dos paradas M-H-S. ¿Cuál proporciona mejor probabilidad esperada de estar entre los 10 primeros, y bajo qué supuestos?

### Capa 2 — Simulador de Degradación a Nivel de Vuelta (Opcional)

#### Dataset
- **f1_strategy_lap_level.csv** (119,908 filas · 1 fila por piloto-carrera-vuelta)

Este es trabajo opcional avanzado. Los equipos usan datos a nivel de vuelta para modelar `lap_time_delta` o degradación como función de la edad del neumático, compuesto, fase de tramo, estado de la pista y contexto meteorológico.

---

## Qué Cuenta como una Buena Recomendación

Una buena recomendación no es solo "el modelo A tiene mayor precisión." Debe decir:

1. Qué estrategia recomiendas.
2. Qué métrica de resultado la respalda.
3. Cuál es la magnitud del beneficio esperado.
4. Qué supuestos podrían invalidar la recomendación.
5. Qué tan confiable/calibrado es el modelo.

---

## Análisis Requeridos

Cada equipo debe incluir:

- [ ] Una línea base simple.
- [ ] Un modelo entrenado.
- [ ] Validación temporal por temporada.
- [ ] Evaluación de calibración o calidad de probabilidad.
- [ ] Análisis de errores segmentado por tipo de estrategia y tipo de circuito.
- [ ] Al menos una comparación de "qué pasaría si".
- [ ] Una discusión explícita sobre sesgo/confusión.
- [ ] PROMPTS.md documentando el uso de IA.

### Métricas Recomendadas

- **Brier score**
- **Log loss**
- **Curva de calibración**
- **ROC-AUC** o precisión promedio (métricas secundarias)
- **MAE** si modelas `finish_position`

---

## Línea Base del Docente (Piso que tu Equipo Debe Superar)

Se proporciona una línea base del docente. Tu modelo del Hito 1 debe igualarla o superarla en el mismo objetivo (`is_top10`) y la misma división temporal, O tu documento de encuadre debe justificar por qué tu enfoque alternativo está mejor alineado con la decisión de estrategia que tu equipo está apoyando.

| Métrica | Valor |
|---------|-------|
| Objetivo | `is_top10` |
| Entrenamiento | temporadas 2019–2021 |
| Calibración | temporada 2022 |
| Prueba | temporadas 2023–2024 |
| Línea base de regla de grilla (Brier score en prueba) | 0.208 |
| Modelo de docente calibrado (Brier score en prueba) | 0.132 |
| Modelo de docente calibrado (ROC-AUC en prueba) | 0.892 |

> La línea base de la regla de grilla es el piso fácil. El modelo del docente calibrado es el piso significativo — superarlo requiere trabajo real. Ambos se reportan para que tu equipo pueda ver qué se ve bien antes de empezar.

---

## Reglas de Sesgo

Las características de estrategia como `n_stops`, `compound_sequence` y `stint_lengths` son observaciones post-carrera en los datos sin procesar. En cualquier otro contexto en este curso, usarlas como predictores sería sesgo.

**Para este capstone están permitidas** porque el producto es una herramienta de comparación de escenarios: estableces intencionalmente esas variables para preguntar "¿qué pasaría si?" El modelo las recibe como entradas de escenario controladas por el usuario, no como información que se conocía mágicamente antes de la carrera. **Tu documento de encuadre debe declarar esta distinción explícitamente** — es un elemento calificado.

### Pautas de Uso de Datos

No uses silenciosamente columnas de incidentes de carrera como `safety car` o `weather outcome` como si se conocieran antes de la carrera. Si las usas, enmarcarlas como:

- Segmentos de auditoría
- Pruebas de estrés de escenarios
- Limitaciones

---

## Limitaciones Conocidas de los Datos

El dataset es una construcción de recuperación ensamblada a partir de artefactos de cursos existentes. Se aplican cinco limitaciones:

1. **La cobertura comienza en 2019**, no en 2018, porque el artefacto de nivel de vuelta recuperado es 2019–2024.

2. **Posición en calificación**: `qualifying_position` es un sustituto de `grid_position`; `qualifying_time_s` está vacío. La columna se mantiene por coherencia pero no construyas una narrativa alrededor del tiempo de calificación. Tratar la calificación como una señal real es un error calificado.

3. **Indicador de safety car**: `safety_car_periods` es un indicador binario por piloto-carrera, no un conteo completo de intervalo de control de carrera.

4. **Las características de estrategia se observan post-carrera** (ver Reglas de Sesgo arriba). Son entradas de escenario en este capstone, no señales pre-carrera.

5. **La elección de estrategia no es independiente**: La elección de estrategia no es independiente del ritmo del coche, piloto, clima e incidentes de carrera. Los equipos deben discutir esta confusión cuando hagan recomendaciones.

> Reconocer y divulgar estas limitaciones contribuye a tu puntuación de "Identificación de limitaciones" (Dimensión 3 de la rúbrica).

---

## Entregables

### Hito 1 — Encuadre del Problema + Línea Base

**Vencimiento:** Miércoles 6 de mayo, 16:20 CLT  
(entrega en clase con observación del asistente — ver la asignación de Canvas del Hito 1 para la política de entrega completa y matriz de penalización individual)

**Enviar:**

- [ ] Un documento de encuadre de 2–3 páginas.
- [ ] Un cuaderno base ejecutable con objetivo = `is_top10`.
- [ ] Notas iniciales de calidad de datos que divulguen las limitaciones del dataset.
- [ ] Plan de validación temporal (train ≤ 2022 / test ≥ 2023 es el estándar del curso; la línea base del docente usa 2019–2021 train / 2022 calibración / 2023–2024 test como variante pedagógica).
- [ ] Plan de comparación "qué pasaría si" inicial.
- [ ] PROMPTS.md.

### Hito 2 — Modelo del Punto Medio + Análisis de Errores

**Vencimiento:** Miércoles 13 de mayo, 16:20 CLT

**Enviar:**

- [ ] Cuaderno de modelado actualizado que cubra ambos objetivos (tu `is_top10` del Hito 1 más tu objetivo de expansión obligatoria).
- [ ] Comparación de línea base vs modelo en ambos objetivos.
- [ ] Análisis de calibración / calidad de probabilidad en ambos objetivos.
- [ ] Análisis de errores segmentado por tipo de estrategia, tipo de circuito, y al menos un contexto adicional.
- [ ] Al menos una comparación de estrategia "qué pasaría si" que use el objetivo expandido para surface una recomendación que `is_top10` solo no podría producir.
- [ ] Plan de mitigación para el informe final.
- [ ] PROMPTS.md actualizado.

### Informe Final + Repositorio

**Vencimiento:** Domingo 17 de mayo, 23:59 CLT

**Enviar un repositorio reproducible e informe de 8–12 páginas.**

### Día de Demostración

**Fecha:** Lunes 18 de mayo, en clase

**Formato de Presentación:**
- 7 minutos de recomendación + 5 minutos de preguntas y respuestas.
- Encuadra la presentación como una recomendación de estrategia para una audiencia de ingeniería de F1.

---

*Última actualización: 4 de mayo de 2026*
