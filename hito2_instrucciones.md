# Hito 2 — Midpoint Model + Error Analysis

---

## 📋 VERSIÓN EN ESPAÑOL

### Resumen General

**Módulo:** Módulo 5 — Unidad IV — Capstone: F1 Race Strategy Advisor  
**Tipo:** Entrega en equipo (equipos de 2–3 integrantes)  
**Ponderación:** 5% de NP (20% de la calificación del Capstone)  
**Plazo:** Miércoles 13 de mayo de 2026 — 23:59 CLT

#### Resumen Rápido

Tu equipo debe enviar el paquete de Hito 2 al repositorio GitHub del equipo, con la URL de GitHub enviada a través de esta tarea de Canvas. El cambio estructural más importante respecto a Hito 1: **tu equipo debe modelar dos targets, no uno**. El `is_top10` original se mantiene desde Hito 1, y debes agregar un target de expansión elegido de una lista fija. El par de targets debe presentar una recomendación de estrategia que `is_top10` solo no podría producir.

**Hito 2 se califica predominantemente en:**
- Análisis de errores (35%)
- Comparación de modelos (30%)

Tu análisis de errores debe estar segmentado por tipo de estrategia, tipo de circuito y al menos un contexto adicional. Tu comparación de modelos debe mostrar evidencia en ambos targets.

---

### Expansión de Target Obligatoria

En Hito 1 modelaste `is_top10` para que la cohorte pudiera compararse directamente contra la línea base del docente. Para Hito 2, **tu equipo debe expandir a al menos un target adicional** elegido de esta lista:

- `is_top5`
- `is_top3`
- `finish_position` (regresión)
- `points` (regresión)

Tu entrega de Hito 2 **debe incluir ambos targets** — el `is_top10` original de Hito 1 más el target de expansión — con un análisis lado a lado que muestre lo que cada target revela sobre decisiones de estrategia.

**⚠️ La expansión no es una pregunta de investigación; es un requisito estructural.** Los equipos que envíen Hito 2 solo en `is_top10` perderán puntos en las Dimensiones 1 y 3 de la rúbrica (la comparación de modelos y el análisis de errores no pueden evaluarse sin dos targets).

#### ¿Por qué esta expansión es obligatoria?

Un modelo binario `is_top10` puede ocultar compensaciones de estrategia. Una estrategia puede preservar P(top10) mientras reduce P(top3) — la compensación es invisible si solo modelas el límite top-10. El propósito de Hito 2 es exponer esa compensación y razonar sobre ella. Tu comparación what-if debe demostrar al menos un caso donde los dos targets disienten en la estrategia recomendada.

#### Cómo elegir tu target de expansión

Elige el target que mejor se ajuste al contexto de decisión de estrategia que tu equipo articuló en Hito 1. Ejemplos:

| Si tu contexto de decisión en Hito 1 es sobre... | Un natural target de expansión es... |
|---|---|
| Podio o no para constructores top | `is_top3` |
| Posiciones que otorgan puntos | `is_top10` (ya cubierto) más `points` |
| Predicción de pérdida de posición | `finish_position` regresión |
| Diferenciación de pilotos de midfield | `is_top5` |

No necesitas justificar la elección como "el mejor target" — solo como "el target que añade la mayor cantidad de información de valor de decisión más allá de `is_top10` para nuestro contexto de decisión específico."

---

### Qué estás Enviando

Un paquete completo de Hito 2 comprometido con el repositorio GitHub de tu equipo. El repositorio debe contener:

#### 1. **hito2_modeling.ipynb**
Notebook reproducible de entrenamiento y evaluación cubriendo ambos targets. El notebook debe:
- Cargar el mismo dataset que Hito 1
- Usar el mismo split temporal bloqueado (train 2019–2021 / calibración 2022 / test 2023–2024)
- Entrenar y evaluar modelos para ambos targets
- Producir salida de probabilidad calibrada para targets binarios y análisis de calidad de probabilidad para targets de regresión

#### 2. **baseline_comparison.md**
Tabla de comparación de línea base en ambos targets. Para `is_top10`, compara contra la línea base del docente (Brier 0.132). Para tu target de expansión, compara contra una línea base apropiada que justifiques.

#### 3. **error_analysis.md**
Segmentado por:
- Tipo de estrategia (one_stop / two_stop / three_plus_stop / no_stop)
- Tipo de circuito (street / permanent / hybrid)
- Al menos un contexto adicional (clima, categoría de constructor, fase de carrera, temporada — tu elección con justificación)
- **Ambos targets deben analizarse**

#### 4. **whatif_comparison.md**
Al menos una comparación de estrategia que use el target de expansión para exponer una recomendación que `is_top10` solo no podría producir. Ejemplo concreto (no es obligatorio usar este): una estrategia de 1-stop puede preservar P(top10) pero reducir P(top3) — esa compensación es invisible si solo modelas `is_top10`.

#### 5. **leakage_audit.md**
Checklist de resultados de auditoría de fuga y confusión. Incluye una nota explícita sobre si tu tratamiento de características de estrategia como entradas de escenario se sostiene bajo ambos targets. La limitación de confusión (la elección de estrategia se correlaciona con ritmo del auto, piloto, clima) debe abordarse.

#### 6. **mitigations.md**
Una lista de riesgos y mitigaciones para el informe final. Como mínimo: ¿en qué fallaría tu modelo y qué cambiarías antes del despliegue?

#### 7. **README.md actualizado y PROMPTS.md**
- README.md reflejando ambos targets
- PROMPTS.md extendido con documentación de cómo se usó y validó el razonamiento asistido por IA específicamente para la expansión del modelo

---

### Política de Envío

Aplica la política de curso estándar para Hito 2:

- **Plazo:** Miércoles 13 de mayo, 23:59 CLT
- **Penalidad por envío tardío:** −10% de la calificación del equipo por cada período de 24 horas de atraso
- **No hay envío antes del domingo 17 de mayo 23:59** → Hito 2 = 1.0 para el equipo
- **Hito 2 no tiene requisito de observación en clase.** Los equipos trabajan de forma asincrónica entre el lunes 11 de mayo (retorno del instructor + retroalimentación de Hito 1) y el plazo del miércoles 13 de mayo. El tiempo de clase durante la semana 11 es estudio estructurado con circulación del instructor.

**Ausencia justificada de las sesiones de la Semana 11 no afecta la calificación de Hito 2** — Hito 2 es asincrónico. Sin embargo, faltar a clase reduce el tiempo que tienes acceso a retroalimentación del instructor.

---

### Línea de Tiempo: Lunes 11 a Miércoles 13 de Mayo

Esta es la ventana de trabajo para Hito 2. Planifica para ella:

- **Lunes 11 de mayo en clase:** Retroalimentación de Hito 1 entregada. Cada equipo recibe un resumen escrito de qué funcionó y qué abordar. El tiempo de clase es estudio estructurado para Hito 2.
- **Miércoles 13 de mayo en clase:** Última sesión apoyada por instructor antes del plazo de Hito 2. Estudio con circulación del instructor. Los equipos que no hayan comenzado el análisis de errores a las 13:50 no terminarán a tiempo.

**Consejo importante:** Usa el martes 12 de mayo fuera de clase para configurar la pipeline de entrenamiento del segundo target. No intentes hacerlo desde cero en clase el miércoles.

---

### Rúbrica de Evaluación

| Dimensión | Ponderación | Qué Buscamos |
|---|---|---|
| Análisis de errores | 35% | Análisis estructurado segmentado por tipo de estrategia, tipo de circuito y ≥1 contexto adicional. Evidencia en ambos targets. Hipótesis de modo de fallo concretas, no declaraciones genéricas. Visualizaciones claras e interpretables. |
| Comparación de modelos | 30% | Líneas base + modelos comparados en ambos targets usando métricas consistentes. Calibración / calidad de probabilidad reportada. El target de expansión produce una recomendación what-if que `is_top10` solo no podría. |
| Mitigaciones + limitaciones | 20% | Las limitaciones son concretas, con consecuencias. Las mitigaciones son realistas y conectadas a modos de fallo específicos del análisis de errores. La limitación de confusión de estrategia se aborda. |
| Reproducibilidad | 15% | Clone limpio funciona de punta a punta. README actualizado. PROMPTS.md actual. Todos los artefactos en el repo. Ambos targets reproducibles desde pasos documentados. |

**Nota sobre pesos respecto a Hito 1 (25/25/30/20):** Hito 2 cambia énfasis hacia análisis crítico (35% análisis de errores vs 25% framing). El estándar para un "buen" Hito 2 es si tu equipo puede explicar dónde falla el modelo y por qué, no solo si funciona bien.

---

### Qué Se Ve Bien vs Trampas Comunes

#### ✅ Un Hito 2 Exitoso

- Tiene todos los artefactos requeridos presentes y ejecutables, incluyendo evidencia en ambos targets
- Muestra que los dos targets disienten en **al menos una recomendación de estrategia específica**, y explica el desacuerdo sustancialmente (no solo "los números son diferentes")
- Tiene análisis de errores que **nombra contextos específicos donde el modelo falla** (ej: "estrategias de dos paradas en circuitos callejeros en carreras mojadas") en lugar de declaraciones genéricas
- **Reporta calibración honestamente**, incluyendo casos donde el modelo está bien calibrado para un target pero no para otro
- Discute la **limitación de confusión de estrategia** (la estrategia se correlaciona con ritmo del auto, piloto, clima) y muestra al menos un intento de mitigación
- Tiene un **PROMPTS.md que documenta cómo se usó específicamente el razonamiento asistido por IA** para la expansión del modelo — no solo "pedimos ayuda a IA"

#### ❌ Trampas Comunes a Evitar

- **Enviar solo `is_top10` esperando que cuente.** No cuenta. La expansión es estructural, no opcional.
- **Agregar un segundo target pero no comparar entre targets.** El punto es la comparación, no solo el segundo modelo.
- **Análisis de errores genérico** ("el modelo tiene error más alto en algunos casos"). La especificidad es lo que obtiene puntos: ¿qué tipo de estrategia, qué tipo de circuito, qué condición climática?
- **Métricas de calibración para targets binarios pero sin discusión de calidad de probabilidad para targets de regresión.** Si tu expansión es `finish_position` o `points`, necesitas una medida de calidad apropiada para regresión (distribución de MAE, pérdida de cuantil, calibración de intervalo de predicción).
- **Ignorar la limitación de confusión.** La elección de estrategia no es aleatoria. Tu análisis de errores debe al mínimo mencionar esto y explicar cómo afecta tus recomendaciones.

---

### Conexión con el Informe Final y Demo Day

El Informe Final (vence el domingo 17 de mayo) extiende Hito 2 agregando la narrativa integradora y el análisis de reproducibilidad comparativa. El análisis de errores que escribes para Hito 2 se convierte en un capítulo del Informe Final.

Demo Day (lunes 18 de mayo) enmarca tu trabajo como una recomendación de estrategia para una audiencia de ingeniería de Aston Martin. Los pitch más fuertes de Demo Day muestran una decisión específica donde el modelo añade valor claro más allá de una heurística — esa decisión típicamente viene de la comparación what-if de Hito 2.

**Si tu comparación what-if de Hito 2 es genérica, tu pitch de Demo Day será genérico.** Las compensaciones concretas (con pilotos, circuitos, estrategias específicos) hacen pitches convincentes. Las compensaciones vagas no.

---

### Política de Uso de IA

Aplica Open-AI. **PROMPTS.md debe actualizarse para cubrir el trabajo de Hito 2.** Documenta cómo se usó IA para:

- Implementar la pipeline del target de expansión
- Generar código de segmentación de análisis de errores
- Validar calibración / calidad de probabilidad
- Razonar sobre confusión

**Al menos una sugerencia de IA rechazada o corregida debe documentarse.** El estándar de 6 campos aplica (Contexto · Prompts · Output · Validación · Adaptaciones · Decisión Final).

---

---

## 📋 ENGLISH VERSION

### General Overview

**Module:** Module 5 — Unit IV — Capstone: F1 Race Strategy Advisor  
**Type:** Team deliverable (teams of 2–3)  
**Weight:** 5% of NP (20% of the Capstone grade)  
**Deadline:** Wednesday, May 13, 2026 — 23:59 CLT

#### Quick Summary

You and your teammates submit a Hito 2 package to your team's GitHub repository, with the GitHub URL submitted via this Canvas assignment. The single most important structural change vs Hito 1: **your team must now model two targets, not one**. The original `is_top10` carries over from Hito 1, and you must add an expansion target chosen from a fixed list. The pair of targets must surface a strategy recommendation that `is_top10` alone could not produce.

**Hito 2 is graded predominantly on:**
- Error analysis (35%)
- Model comparison (30%)

Your error analysis must be sliced by strategy type, circuit type, and at least one additional context. Your model comparison must show evidence on both targets.

---

### Mandatory Target Expansion

In Hito 1 your team modeled `is_top10` so the cohort could be compared directly against the docent baseline. For Hito 2, **your team must expand to at least one additional target** chosen from this list:

- `is_top5`
- `is_top3`
- `finish_position` (regression)
- `points` (regression)

Your Hito 2 deliverable **must include both targets** — the original `is_top10` carried over from Hito 1 and the expansion target — with a side-by-side analysis that surfaces what each target reveals about strategy decisions.

**⚠️ The expansion is not a research question; it is a structural requirement.** Teams that submit Hito 2 only on `is_top10` will lose points on Dimensions 1 and 3 of the rubric (model comparison and error analysis cannot be evaluated without two targets).

#### Why This Expansion Is Mandatory

A binary `is_top10` model can hide strategy trade-offs. A strategy may preserve P(top10) while reducing P(top3) — the trade-off is invisible if you only model the top-10 boundary. The point of Hito 2 is to surface that trade-off and reason about it. Your what-if comparison must demonstrate at least one such case where the two targets disagree on the recommended strategy.

#### How to Choose Your Expansion Target

Choose the target that best fits the strategy decision your team articulated in Hito 1. Examples:

| If your Hito 1 decision context is about... | A natural expansion target is... |
|---|---|
| Podium-or-not for top constructors | `is_top3` |
| Points-paying positions | `is_top10` (already covered) plus `points` |
| Position-loss prediction | `finish_position` regression |
| Differentiating midfield drivers | `is_top5` |

You don't need to justify the choice as "the best target" — only as "the target that adds the most decision-value information beyond `is_top10` for our specific decision context."

---

### What You're Submitting

A complete Hito 2 package committed to your team's GitHub repository. The repository must contain:

#### 1. **hito2_modeling.ipynb**
A reproducible training and evaluation notebook covering both targets. The notebook should:
- Load the same dataset as Hito 1
- Use the same locked temporal split (train 2019–2021 / calibration 2022 / test 2023–2024)
- Train and evaluate models for both targets
- Produce calibrated probability output for binary targets, and a probability-quality analysis for regression targets

#### 2. **baseline_comparison.md**
A baseline comparison table on both targets. For `is_top10`, compare against the docent baseline (Brier 0.132). For your expansion target, compare against an appropriate baseline you justify.

#### 3. **error_analysis.md**
Sliced by:
- Strategy type (one_stop / two_stop / three_plus_stop / no_stop)
- Circuit type (street / permanent / hybrid)
- At least one additional context (weather, constructor tier, race phase, season, your choice with justification)
- **Both targets must be analyzed**

#### 4. **whatif_comparison.md**
At least one strategy comparison that uses the expansion target to surface a recommendation that `is_top10` alone could not produce. Concrete example (you are not required to use this exact one): a 1-stop strategy may preserve P(top10) but reduce P(top3) — that trade-off is invisible if you only model `is_top10`.

#### 5. **leakage_audit.md**
Leakage and confounding guard checklist results. Include an explicit note on whether your treatment of strategy features as scenario inputs holds up under both targets. The confounding limitation (strategy choice correlates with car pace, driver, weather) must be addressed.

#### 6. **mitigations.md**
A list of risks and mitigations for the final report. At minimum: what would your model fail on, and what would you change before deployment?

#### 7. **Updated README.md and PROMPTS.md**
- README.md reflecting both targets
- PROMPTS.md extended with documentation of how AI-assisted reasoning was used and validated for the model expansion specifically

---

### Submission Policy

Standard course policy applies for Hito 2:

- **Deadline:** Wednesday May 13, 23:59 CLT
- **Late submission penalty:** −10% of the team grade for each 24-hour period late
- **No submission by Sunday May 17 23:59** → Hito 2 = 1.0 for the team
- **Hito 2 has no in-class observation requirement.** Teams work asynchronously between Mon May 11 (instructor return + Hito 1 feedback) and Wed May 13 deadline. Class time during Week 11 is structured studio with instructor circulation.

**Justified absence from the Week 11 sessions does not affect Hito 2 grade** — Hito 2 is asynchronous. However, missing class reduces the time you have access to instructor feedback.

---

### Timeline: Monday May 11 to Wednesday May 13

This is the working window for Hito 2. Plan for it:

- **Monday May 11 in class:** Hito 1 feedback delivered. Each team receives a written summary of what worked and what to address. Class time is structured studio for Hito 2.
- **Wednesday May 13 in class:** Last instructor-supported session before the Hito 2 deadline. Studio with instructor circulation. Teams that haven't started error analysis by 13:50 will not finish on time.

**Important tip:** Use Tuesday May 12 outside class to set up the second target's training pipeline. Don't try to do it from scratch in class on Wednesday.

---

### Rubric

| Dimension | Weight | What We Look For |
|---|---|---|
| Error analysis | 35% | Structured analysis sliced by strategy type, circuit type, and ≥1 additional context. Evidence on both targets. Concrete failure-mode hypotheses, not generic statements. Visualizations clear and interpretable. |
| Model comparison | 30% | Baselines + models compared on both targets using consistent metrics. Calibration / probability-quality reported. The expansion target produces a what-if recommendation `is_top10` alone could not. |
| Mitigations + limitations | 20% | Limitations are concrete, with consequences. Mitigations are realistic and connected to specific failure modes from the error analysis. The strategy-confounding limitation is addressed. |
| Reproducibility | 15% | Clean clone runs end-to-end. README updated. PROMPTS.md current. All artifacts in repo. Both targets reproducible from documented steps. |

**Note on weights compared to Hito 1 (25/25/30/20):** Hito 2 shifts emphasis toward critical analysis (35% error analysis vs 25% framing). The bar for "good" Hito 2 is whether your team can explain where the model fails and why, not just whether it performs well.

---

### What Good Looks Like vs Common Pitfalls

#### ✅ A Passing Hito 2

- Has all required artifacts present and runnable, including evidence on both targets
- Shows that the two targets disagree on **at least one specific strategy recommendation**, and explains the disagreement substantively (not just "the numbers are different")
- Has error analysis that **names specific contexts where the model fails** (e.g., "two-stop strategies at street circuits in wet races") rather than generic statements
- **Reports calibration that is honestly assessed**, including cases where the model is well-calibrated for one target but not the other
- Discusses the **strategy-confounding limitation** (strategy correlates with car pace, driver, weather) and shows at least one mitigation attempt
- Has a **PROMPTS.md that documents how AI-assisted reasoning was specifically used** for the model expansion — not just generic "we asked AI for help"

#### ❌ Common Pitfalls to Avoid

- **Submitting only `is_top10` and hoping it counts.** It doesn't. The expansion is structural, not optional.
- **Adding a second target but not comparing across targets.** The point is the comparison, not just the second model.
- **Generic error analysis** ("the model has higher error in some cases"). Specificity is what gets points: which strategy type, which circuit type, which weather condition?
- **Calibration metrics for binary targets but no probability-quality discussion for regression targets.** If your expansion is `finish_position` or `points`, you need a regression-appropriate quality measure (MAE distribution, quantile loss, prediction interval calibration).
- **Ignoring the confounding limitation.** Strategy choice is not random. Your error analysis should at minimum mention this and explain how it affects your recommendations.

---

### How Hito 2 Connects to the Final Report and Demo Day

The Final Report (due Sun May 17) extends Hito 2 by adding the integrative narrative and the comparative reproducibility analysis. The error analysis you write for Hito 2 becomes a chapter in the Final Report.

Demo Day (Mon May 18) frames your work as a strategy recommendation to an Aston Martin engineering audience. The strongest Demo Day pitches show one specific decision where the model adds clear value beyond a heuristic — that decision typically comes from the Hito 2 what-if comparison.

**If your Hito 2 what-if comparison is generic, your Demo Day pitch will be generic.** Concrete trade-offs (with specific drivers, circuits, strategies) make compelling pitches. Vague trade-offs do not.

---

### AI Use Policy

Open-AI applies. **PROMPTS.md must be updated to cover the Hito 2 work.** Document how AI was used to:

- Implement the expansion target's pipeline
- Generate error-analysis slicing code
- Validate calibration / probability quality
- Reason about confounding

**At least one rejected or corrected AI suggestion must be documented.** The 6-field standard applies (Context · Prompts · Output · Validation · Adaptations · Final Decision).

---

