# Metodología TDSP
## Predicción del Riesgo de Diabetes con la Metodología TDSP

**Materia:** Metodología de la Investigación
**Autores:** Fabiola López Velázquez · Jorge Luis Ortiz Libreros · Joshua Iván Olivares Blanco

## Descripción

**TDSP** (*Team Data Science Process*) es una metodología de Microsoft para proyectos de ciencia de datos en equipo. Cubre el ciclo completo en cinco etapas: comprensión del negocio, adquisición y comprensión de los datos, modelado, implementación y aceptación del cliente. A diferencia de CRISP-DM, define roles, entregables y una estructura de proyecto estandarizada, lo que facilita la colaboración y el seguimiento.

Análisis de una base de datos de diabetes con **15,000 registros**, aplicando la metodología **TDSP (Team Data Science Process)** de Microsoft. TDSP proporciona una estructura estandarizada que abarca desde la adquisición de los datos hasta la implementación del modelo, y es un enfoque pensado para el trabajo en equipo.

El resultado es un modelo de **clasificación supervisada multiclase** que estratifica el riesgo de diabetes en tres niveles: `Low`, `Moderate` y `High`.

## Roles del equipo

Para estructurar operativamente el proyecto, el equipo (3 integrantes) se repartió las fases del ciclo de vida del modelo:

| Integrante | Rol | Responsabilidades |
|---|---|---|
| **Joshua Iván Olivares Blanco** | Líder del Proyecto (*Project Lead*) | Gestión metodológica y documental. Estructuró el proyecto bajo las normas TDSP, redactó las justificaciones de las fases de Comprensión del Negocio y Aceptación del Cliente, y consolidó los hallazgos en el reporte final. |
| **Fabiola López Velázquez** | Científica de Datos (*Data Scientist*) | Modelado analítico y matemático. Ajustó el árbol de decisión multiclase, determinó el parámetro de complejidad óptimo mediante validación cruzada y extrajo las métricas de evaluación (Accuracy, índice Kappa y sensibilidad por clase). |
| **Jorge Luis Ortiz Libreros** | Ingeniero de Datos (*Data Engineer / Architect*) | Infraestructura y preparación de la información. Trató la base de datos (categoría `Unknown` para valores faltantes, eliminación de variables sin varianza predictiva), hizo la partición estratificada de la muestra y serializó el modelo para la implementación. |

Además, se usó **R** para generar las gráficas y se programó una función que las guarda automáticamente en una carpeta externa (`docs`, en el escritorio), para facilitar su manipulación y asegurar su resguardo.

---

## 1. Comprensión del Negocio (*Business Understanding*)

- **Planteamiento del problema:** evaluar el riesgo de padecer diabetes es un proceso complejo que requiere analizar múltiples variables simultáneamente. Existe la necesidad metodológica de una herramienta automatizada y basada en evidencia que apoye el diagnóstico clínico.
- **Objetivo de la investigación:** desarrollar e implementar un modelo predictivo de clasificación supervisada multiclase para estratificar el riesgo de diabetes en tres niveles (`Low`, `Moderate` y `High`), garantizando la interpretabilidad de las reglas de decisión clínica.
- **Contexto clínico:** el modelo busca apoyar la toma de decisiones utilizando características demográficas y mediciones clínicas.

## 2. Adquisición y Comprensión de Datos (*Data Acquisition and Understanding*)

### Exploración de la base de datos

- La base contiene **15,000 registros**.
- Se detectaron valores faltantes (NA), que fueron categorizados como `"Unknown"` para preservar el volumen de datos.
- Se eliminó la variable `patient_id` por no poseer valor predictivo.

El estudio se basó en una muestra de 15,000 observaciones depuradas. La distribución de la variable objetivo muestra una prevalencia del **60 % para riesgo Low**, **25 % para Moderate** y **15 % para High**.

![Distribución del riesgo de diabetes](images/distribucion_riesgo.png)

### Tratamiento de datos

Para mantener la integridad y el volumen de la muestra, **no se eliminaron** las observaciones con valores faltantes en variables de estilo de vida (`smoking_status`, `alcohol_consumption`, `income_bracket`). En su lugar, se agruparon en una nueva categoría llamada `"Unknown"`, permitiendo que el algoritmo interprete la ausencia de información como un patrón.

La variable `patient_id` fue descartada por carecer de varianza predictiva.

### Análisis visual y justificación de los datos

La exploración multivariada demostró que, aunque la mediana del Índice de Masa Corporal (BMI) y la edad aumentan conforme se agrava el riesgo, existe un **traslape estadístico significativo** entre las categorías. Este hallazgo visual justifica la insuficiencia de métodos de corte simples y exige la aplicación de algoritmos de *Machine Learning*.

![BMI y edad según nivel de riesgo](images/bmi_edad_por_riesgo.png)

## 3. Modelado (*Modeling*)

### Desarrollo del árbol de decisión

Se ajustó un modelo de árbol de clasificación.

![Árbol de decisión inicial](images/arbol_inicial.png)

### Validación cruzada y poda

Se determinó el parámetro de complejidad (`cp`) óptimo minimizando el error relativo de validación cruzada. El árbol podado define las reglas de decisión óptimas para la clasificación.

![Error relativo de CV vs parámetro de complejidad](images/error_cv_vs_cp.png)

### Evaluación del modelo

Resultados sobre el conjunto de prueba (**4,500 registros**):

| Métrica | Valor |
|---|---|
| **Accuracy** | 76.2 % (IC 95 %: 74.93 % – 77.44 %) |
| **Índice Kappa** | 0.5569 |
| **Valor p (prueba de McNemar)** | < 2.2e-16 |

El valor p indica que el modelo es estadísticamente superior al *No Information Rate* (60 %).

### Rendimiento por clase

| Clase | Resultado destacado |
|---|---|
| **Low** | Mayor sensibilidad (89.81 %) y alta precisión general (*Balanced Accuracy*: 82.88 %). |
| **High** | Excelente especificidad (98.32 %), lo que indica gran capacidad para evitar falsos positivos en el nivel más grave. Sensibilidad: 52.88 %. |
| **Moderate** | Es el mayor reto de clasificación por su solapamiento natural con los casos leves y graves (Sensibilidad: 57.51 %, Valor Predictivo Positivo: 52.99 %). |

### Árbol de decisión podado

![Árbol de decisión podado](images/arbol_podado.png)

### Importancia de las variables

El análisis demostró que `fasting_blood_sugar` (glucosa en ayunas) y `hba1c_level` (hemoglobina glucosilada) son las variables predictoras más determinantes para los factores de riesgo.

![Importancia de variables en el árbol final](images/importancia_variables.png)

## 4. Implementación (*Deployment*)

El modelo final podado fue **serializado** como un artefacto listo para integrarse en una aplicación clínica de escritorio para predicciones en tiempo real.

Se utilizó R para procesar y analizar la información del riesgo de diabetes. Como TDSP es un enfoque más estructurado para equipos, se asignaron roles específicos para cubrir todas las fases de la metodología.

También se usó código en R para generar y guardar al mismo tiempo los gráficos, fuera del entorno de R, en una carpeta del escritorio. Esto facilita su manipulación y permite que todos los integrantes accedan fácilmente a la información.

## 5. Aceptación del Cliente (*Customer Acceptance*)

TDSP permitió estandarizar el proceso desde el dato crudo hasta un modelo desplegable.

- **Validación:** el modelo demuestra un alto rendimiento para descartar correctamente casos graves (`High`) e identificar casos leves (`Low`).
- **Plan de acción iterativo:** dado el rendimiento en la clase `Moderate`, la siguiente iteración del ciclo de vida TDSP contemplará probar **técnicas de ensamble** para afinar las fronteras de decisión en los casos intermedios.

**Conclusión:** el modelo es clínicamente viable como una herramienta de **descarte rápido para riesgos bajos** y como **confirmación para riesgos altos**.

## 6. Referencias

- Aggarwal, M. (s. f.). *Diabetes risk prediction*. Kaggle. https://www.kaggle.com/datasets/mansiaggarwal88/diabetes-risk-prediction
- Microsoft. (2017). *Team Data Science Process (TDSP)*. Microsoft Learn. https://learn.microsoft.com/es-es/azure/architecture/data-science-process/overview
- R Core Team. (2023). *R: A language and environment for statistical computing*. R Foundation for Statistical Computing. https://www.R-project.org/
- Therneau, T., Atkinson, B., & Ripley, B. (2023). *rpart: Recursive Partitioning and Regression Trees* (R package version 4.1.21). https://CRAN.R-project.org/package=rpart
