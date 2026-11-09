# Model Card: Clasificador de Admisión (Auditoría de Sesgo)

> Plantilla basada en _Model Cards for Model Reporting_ (Mitchell et al., 2019).

## Model Details

- **Developed by:** Santiago Sánchez
- **Model Date:** 2026-11-08
- **Model Version:** v1.0.0
- **Model Type:** Clasificador binario (evaluado a partir de sus predicciones)
- **License:** MIT
- **Contact:** SantiagoAvila21 (repositorio de GitHub)

## Intended Use

- **Primary Intended Uses:** Automatización inicial del filtrado de solicitudes para priorizar la revisión de un analista.
- **Primary Intended Users:** Analistas de admisión y equipos de riesgo.
- **Out-of-Scope Uses:** Decisiones jurídicas, diagnósticos médicos definitivos o cualquier exclusión automática sin supervisión humana.

## Training & Evaluation Data

- **Dataset Overview:** Muestra de validación de 24 registros (12 por grupo) con las columnas `id`, `true_label`, `predicted_label` y `protected_group`.
- **Protected Attributes:** `protected_group`.
- **Group Definition:** `0` = grupo mayoritario; `1` = grupo minoritario.

## Quantitative Performance & Fairness Audit

_Resultados obtenidos tras ejecutar `scripts/calculate_audit.py` sobre el dataset sesgado:_

| Métrica                                    | Valor Obtenido | Umbral Objetivo |
| :----------------------------------------- | :------------- | :-------------- |
| **Accuracy Global**                        | 62.5%          | $> 85\%$        |
| **Tasa de Aprobación (Grupo Mayoritario)** | 75.0%          | -               |
| **Tasa de Aprobación (Grupo Minoritario)** | 25.0%          | -               |
| **TPR / Recall (Grupo Mayoritario)**       | 85.7%          | -               |
| **TPR / Recall (Grupo Minoritario)**       | 37.5%          | -               |
| **Disparate Impact Ratio (DI)**            | 0.333          | **$\ge 0.80$**  |
| **Statistical Parity Difference (SPD)**    | -0.500         | $\approx 0$     |
| **Equal Opportunity Difference (EOD)**     | -0.482         | $\approx 0$     |

- **Resultado de la regla de los 4/5:** [ ] Aprobado [x] Violación detectada
- **Conclusión de la Auditoría:** El modelo **no pasa** la prueba de equidad: el DI = 0.333 < 0.80, con SPD = -0.500 y EOD = -0.482. El grupo minoritario es aceptado mucho menos y, entre las personas realmente calificadas, se le reconocen muchas menos. Debe volver a desarrollo para mitigación de sesgos antes de cualquier despliegue.

## Ethical Considerations & Limitations

- **Known Biases:** El modelo reproduce un sesgo histórico contra el grupo minoritario: aplica un estándar de aceptación más estricto a ese grupo (FPR de 0% frente a 60% del mayoritario) y descarta a personas calificadas (5 falsos negativos en el grupo 1).
- **Limitations of this Audit:** Muestra muy pequeña (12 registros por grupo), un único atributo protegido, sin análisis interseccional ni intervalos de confianza; las métricas son sensibles a pocos casos.
- **Mitigations Applied / Recommended:** Se recomiendan ajuste de umbral de decisión por grupo (post-procesamiento), reponderación/remuestreo del entrenamiento o restricciones de equidad en proceso (p. ej. Fairlearn `ExponentiatedGradient`), acompañadas de supervisión humana. La mitigación aplicada en el taller fue simulada (sustitución del CSV por `fair_predictions.csv`).

## Sign-off

- **Auditor:** Santiago Sánchez
- **Audit Date:** 2026-11-08
- **Decision:** [ ] Aprobado para despliegue [x] Devuelto a desarrollo
