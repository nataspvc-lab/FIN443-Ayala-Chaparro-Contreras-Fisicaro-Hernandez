# Bitácora de Trabajo Asistido por IA — Proyecto FIN443

**Curso:** Programación en Finanzas (FIN443) · Evaluación 2 · Magíster en Finanzas UAI  
**Grupo:** Ayala, Chaparro, Contreras, Fisicaro y Hernández  
**Fecha de consolidación:** 28 de noviembre de 2026  
**Título del trabajo:** *Contagio o interdependencia: correlaciones bursátiles en América Latina, 2005–2025*

---

## 1. Propósito y alcance de la asistencia con IA

Este documento deja constancia cronológica, técnica y verificable de todas las etapas en las que se utilizó asistencia de inteligencia artificial durante el desarrollo, auditoría, extensión metodológica y validación final del proyecto. Todo el trabajo estuvo acotado por tres principios rectores:

1. **Rigor académico:** cada ajuste o extensión econométrica debía derivarse directamente de los 16 artículos PDF disponibles en `Papers/`.
2. **Replicabilidad y eficiencia:** el pipeline completo debía ejecutarse en R base desde una sola instrucción, sin dependencias frágiles frente al formato de datos de entrada y en un tiempo ampliamente inferior a 2 minutos.
3. **Trazabilidad total:** cada cifra reportada en tablas, texto o consola debía existir de forma unívoca en `resultados/numeros_informe.csv` vinculada a su archivo CSV de salida y a la línea exacta de `codigo/analisis_fin443.R`.

---

## 2. Etapa 1 — Revisión de literatura confirmada y replicabilidad inicial (`FIN443_Contagio/`)

En la fase exploratoria se analizaron los 16 artículos en `Papers/` (previamente referenciados también bajo `Confirmed/`) y la estructura de `base_unidas.xlsx`, documentando los hallazgos en `FIN443_Contagio/01_Modelo_y_resultados.md`, `FIN443_Contagio/02_Pasos_RStudio.md` y `FIN443_Contagio/03_Bitacora_y_Literatura_Confirmada.md`:

- **Explotación del canal cambiario:** se identificó que la base incluye tipos de cambio bilaterales (`*_tc`) además de índices locales (`*_precio_local`) y en dólares (`*_precio_usd`), verificando la identidad exacta $P^{\text{USD}}_{i,t} = P^{\text{Local}}_{i,t} / TC_{i,t}$ y $r^{\text{USD}}_{i,t} = r^{\text{Local}}_{i,t} + s_{i,t}$ (con error máximo observado de $3{,}22 \times 10^{-13}$).
- **Normalización de fechas e imputación LOCF:** se diseñó un protocolo robusto de lectura de fechas heterogéneas (seriales Excel, `DD/MM/YYYY`, `MM/DD/YYYY`, ISO, meses en español/inglés), ordenamiento cronológico, deduplicación determinística e imputación *Last Observation Carried Forward* (`na_locf_vec()`, siguiendo la convención de Chiang, Jeon y Li, 2007).

---

## 3. Etapa 2 — Auditoría y robustecimiento del código base del grupo (`codigo/`)

Posteriormente se auditó la carpeta modular entregada por el equipo (`codigo/main.R`, `codigo/parametros.R`, `codigo/defensa.R`, `codigo/verificacion.R` y `codigo/analisis_fin443.R`), corrigiendo errores técnicos y añadiendo capas de robustez **sin alterar la arquitectura ni introducir modelos distintos** en esa primera auditoría:

- **`codigo/parametros.R`:** auditado íntegramente; no requirió cambios.
- **Ingesta y fechas (`[B1]`, `[B2]`, `[B13]` en `codigo/analisis_fin443.R`):**
  - `find_fin443_input()` amplió la búsqueda de rutas alternativas (`datos/base_unidas.csv`, `datos/bases_unidas.csv`, `.xlsx` en raíz y `FIN443_Contagio/datos/`), y se agregó `strict_input` en `prepare_sample()`/`run_fin443()` (activado desde `codigo/main.R` y `codigo/defensa.R` cuando se pasa ruta explícita por CLI).
  - `read_fin443()` elimina marcas UTF-8 BOM (`\ufeff` o `ï»¿`) en nombres de columnas.
  - `parse_fin443_dates()` corrigió la expresión regular `"[T ]"` a `"[Tt ]"` para soportar marcas ISO-8601 tras la conversión previa a minúsculas en `normalize_month_tokens()`.
  - `audit_fin443()` se parametrizó mediante `markets = panel$markets` para auditar subconjuntos definidos en `PARAMETROS$mercados`.
- **Forbes–Rigobon y ventana semanal (`[B7]`):**
  - En `contrast_event()`, se corrigió el posible pivoteo de columnas en `.lm.fit()` (`if (isTRUE(fit$pivoted)) co <- co[order(fit$pivot), , drop = FALSE]`).
  - Se añadió la bandera diagnóstica `alerta_var_cae = (obs["delta_a", ] < 0) | (obs["delta_b", ] < 0)` ante caídas de varianza en crisis (p. ej., Perú en Eurozona o Argentina en Taper).
  - En `contrast_scenario(..., weekly = FALSE)`, se corrigió la contaminación de la semana parcial del evento en el escenario `USD_ajustado_semanal` (`keep <- !weekly | (dates >= anch) | (dates + 6L < anch)`).
- **Colas, spillovers y carteras (`[B8]`, `[B9]`, `[B10]`):**
  - `exceedance_correlations()` protege el radicando con `sqrt(pmax(1 - rho^2, 0))`.
  - `coexceedances()` itera por índice de fila (`seq_len(nrow(events))`) preservando la clase `Date` y protege denominadores con `max(sum(mask), 1L)`.
  - `var_gfevd()` anula coeficientes no finitos, acota diagonales con `pmax(..., 1e-12)` y vectoriza el cálculo con `rowSums(PS * Psi[[h]])`; `spillover_analysis()` acota `window <- min(window, nrow(x))`.
  - `min_variance()` incorpora un respaldo iterativo Frank–Wolfe cuando `k > 10L`; `factor_covariance_bhz()` valida rango completo y finitud devolviendo `NULL` ante singularidad; `jobson_korkie_memmel_p()` protege el denominador con `max(s1 * s2, 1e-12)`; y `portfolio_metrics()` incorpora `turnover_local_hedged` eliminando el `NA` previo en `rotacion_anual_pct` de `carteras_cobertura_cambiaria.csv` (`63,14%` en `MinVar_movil_cubierta_teorica`).
- **Trazabilidad en código y consola (`[B11]`, `[B12]`, `[B13]`):**
  - `finish_registry()` prioriza coincidencias exactas con `# ---- [tag]` para registrar el número de línea preciso en `codigo/analisis_fin443.R`.
  - Se creó `print_console_summary()` para imprimir todas las tablas resumidas junto con la tabla `[B12]` de trazabilidad en la consola de R.
- **Pruebas unitarias y reproducibilidad (`codigo/verificacion.R`, `codigo/defensa.R`):**
  - Se añadió `find_root()` en `codigo/verificacion.R` para ejecutar desde cualquier directorio y se sumaron las pruebas **V9** (exclusión de semana parcial en ventana previa) y **V10** (BOM + ISO-T + filas desordenadas + imputación LOCF).
  - En `codigo/defensa.R` se incorporó `strict_input`, `weekly = FALSE`, semilla y la columna `alerta_var_cae`.

---

## 4. Etapa 3 — Extensiones metodológicas fundamentadas en `Papers/` (`codigo/`)

Con autorización explícita para profundizar el trabajo manteniendo un tiempo prudente de ejecución y rigor bibliográfico, se integraron cuatro extensiones en `codigo/analisis_fin443.R`:

1. **Inferencia robusta HAC (`[B3b]`):** `ols_hac()` con kernel de Bartlett y rezago automático de Newey–West, más respaldo pseudoinverso `ginv_fallback()`.
2. **Regresiones dinámicas DCC sobre crisis y estrés global (`[B5b]` — Chiang, Jeon y Li, 2007):**
   - `chiang_dcc_regressions()` estima la ecuación de media de $\hat{\rho}_{ij,t}$ y de varianza residual $\ln(\hat{u}_t^2)$ sobre `VIX_alto_q90` y dummies de crisis (exportando `resultados/chiang_dcc_regresiones.csv`).
   - Resultado clave: en `Promedio_Regional`, `VIX_alto_q90` eleva significativamente el DCC tanto en `USD_bruto` ($+0{,}004508$, $p_{\text{HAC}} = 3{,}42\times 10^{-6}$) como en `USD_ajustado` ($+0{,}001289$, $p_{\text{HAC}} = 0{,}0121$), mientras que ninguna de las nueve crisis individuales resulta significativa al controlar por VIX (0 de 9 en ambas capas).
3. **Test de Corsetti, Pericoli y Sbracia (`[B7b]` — CPS, 2005):**
   - `cps_benchmark_corr()`, `cps_effect()` y `contrast_cps_scenario()` permiten $\lambda_j > 0$ estimando la razón varianza idiosincrática / sistemática mediante los factores globales exógenos (`SP500` y `dlogVIX`) proyectados desde la ventana pre-crisis (exportando `resultados/corsetti_pericoli_sbracia.csv` y `resultados/resumen_corsetti_pericoli_sbracia.csv`).
   - Resultado clave: frente a `0 de 90` pruebas significativas con Holm en Forbes–Rigobon, CPS detecta **22 de 90** con $p < 0{,}05$ sin ajuste y **4** tras Holm global (**3** en Lehman: Argentina–Brasil, Brasil–México y Chile–Perú, más **1** en Taper: Chile–Perú), con los 10 pares significativos en Lehman bajo Holm intra-crisis (exceso medio $+0{,}3256$, $p_{\min} = 5{,}51\times 10^{-7}$). Se documenta la cautela en Chile–Perú durante Lehman porque $\lambda$ en Perú alcanza el tope código de 250.
4. **Modelo multifactorial y dispersión transversal (`[B7c]` — Bekaert et al., 2014 y 2016):**
   - `bekaert_factor_contagion()` estima el modelo de 3 factores ortogonalizados (`SP500`, `dlogVIX` ortogonalizado y regional *leave-one-out*) con interacciones de crisis, test conjunto de Wald ($\text{df}=3$), descomposición de varianza (`VR`) y regresión HAC de la dispersión transversal $CS_t$ (exportando `resultados/bekaert_contagio_factorial.csv` y `resultados/bekaert_dispersion_transversal.csv`).
   - Resultado clave: el test de Wald rechaza estabilidad en `Crisis_Agrupadas` solo en **Argentina** ($p = 0{,}0111$) y **Brasil** ($p = 0{,}000193$; Perú marginal $p = 0{,}0701$), explicando la interdependencia entre `21,0%` y `76,8%` de la varianza frente a `-3,0%` a `18,5%` por cambio de betas; además, $CS_t$ aumenta significativamente en crisis ($+0{,}3123$, $p_{\text{HAC}} = 0{,}00186$).
5. **Retorno Equivalente Cierto en carteras (`[B10]` — DeMiguel et al., 2009; Ang y Bekaert, 2002):**
   - `portfolio_metrics()` añade `CEQ_anual_gamma1_pct` y `CEQ_anual_gamma5_pct` en `carteras_metricas.csv` y `carteras_cobertura_cambiaria.csv`. Para $\gamma = 5$, todas las carteras USD sin cobertura obtienen CEQ negativo (`-2,59%` la histórica móvil vs. `-5,87%` en $1/N$), mientras que `MinVar_movil_cubierta_teorica` alcanza $+1{,}194\%$ anual con volatilidad de `14,81%`.
6. **Ampliación de `codigo/verificacion.R`:** se añadieron **V11** (CPS con $\lambda=0$ equivale a FR), **V12** (`ols_hac` reproduce exactamente MCO con `vcov` positiva) y **V13** (simulación Bekaert recupera salto $\gamma_{\text{US}} = 0{,}6$, $p_{\text{Wald}} = 3{,}56\times 10^{-61}$).

---

## 5. Etapa 4 — Auditoría integral del informe, trazabilidad al 100% y recompilación (`informe.tex` / `informe.pdf`)

Al auditar el borrador `informe.pdf` entregado por el grupo frente a los resultados de R y los 16 PDFs en `Papers/`, se realizaron las siguientes acciones:

1. **Trazabilidad completa en `codigo/analisis_fin443.R`:** se amplió el registro de `run_fin443()` hasta alcanzar **261 cifras en `resultados/numeros_informe.csv`** (con `0` valores `NA` en `linea` y `valor`), cubriendo también los indicadores de limpieza y error máximo de precios (`[T0]`), p-valores de VIX en Chiang (`[T1b]`), descomposición de varianza por país (`[T2]`), p-valores individuales destacados en FR y CPS (`[T3]`, `[T3c]`), varianza de interdependencia en Bekaert (`[T3d]`), probabilidades condicionales de coexcedencias (`[T4]`), desglose direccional de spillovers (`[T5]`) y CEQ de cobertura cambiaria (`[T6]`).
2. **Correcciones de contenido, datos y redacción en `informe.tex` → `informe.pdf`:**
   - **Huella MD5 y rutas (Sección 2):** se corrigió el hash desactualizado (`d56848c6...1f942`) por el real verificado del archivo de entrada (`d1509f43d08346281d482607f0c8e419`, abreviado `d1509f43...0c8e419`) y se documentó el respaldo automático de `datos/base_unidas.csv` a `FIN443_Contagio/datos/base_unidas.xlsx` junto con la convención LOCF.
   - **Ubicación metodológica (Sección 3):** se movió la oración sobre robustez de Forbes–Rigobon (*USD brutos, 60 sesiones, frecuencia semanal y 999 réplicas de bootstrap*) desde la viñeta de *Modelo multifactorial* hacia su bloque correcto (*Contagio: test de Forbes y Rigobon*).
   - **Numeración secuencial y diseño de tablas (Sección 4):** se corrigió el salto de numeración original (`Tabla 1 → Tabla 5 → Tabla 2 → Tabla 3 → Tabla 4`), estableciendo una secuencia estricta **Tabla 1** a **Tabla 6**, corrigiendo encabezados/notas (especificando las capas USD bruto, USD ajustado y CPS en la Tabla 1) y eliminando cortes de palabras (`commodities`, `Idiosincrático`, `Devaluación`).
   - **Exactitud numérica en Validación (Sección 6):** se corrigió el error máximo reportado de `V1` (`2·10^-15` → **`7,99·10^-15`**), se precisó el error de identidad cambiaria en **`3,22·10^-13`** y se reflejaron los resultados vigentes (**0,13 minutos**, **261 cifras**, **16 de 16 pruebas OK**, y coincidencia exacta de `codigo/defensa.R Lehman`).
   - **Verificación de referencias y citas (Sección 8 y Referencias):** se cotejaron los 16 artículos PDF en `Papers/` confirmando exactitud bibliográfica (autores, revistas, volúmenes, números, páginas y DOIs, incluido Longin y Solnik, 2001). Se añadieron en el texto las citas faltantes declaradas en Tabla 6 (Chiang et al., 2007 en Sección 2; Bekaert et al., 2016 en 4.3 y 4.4; Boyer et al., 2006 en 4.4; Bae et al., 2003 en 4.5; Bekaert et al., 2009 y Ang & Bekaert, 2002 en 4.7) y se ajustó la maquetación con XeLaTeX (`es-tabla`, `xurl`, `booktabs`) logrando 9 páginas limpias sin advertencias ni desbordes de margen.

---

## 6. Verificación reproducible final y mapa de artefactos

| Verificación ejecutada | Comando | Resultado observado |
| :--- | :--- | :--- |
| Corrida completa del pipeline | `Rscript codigo/main.R` | Código salida `0` · **`0,13 minutos`** (~7,8 s) · **261 cifras** en `resultados/numeros_informe.csv` (`0` `NA`). |
| Batería de invariantes y simulación | `Rscript codigo/verificacion.R` | Código salida `0` · **`16 de 16 pruebas OK`** (`V1`–`V13`). |
| Reproducibilidad puntual | `Rscript codigo/defensa.R Lehman` | Código salida `0` · `0,1 s` · coincidencia exacta de los 10 pares con `resultados/forbes_rigobon.csv`. |
| Compilación editorial del informe | `xelatex -interaction=nonstopmode informe.tex` (×2) | Código salida `0` · `informe.pdf` (9 páginas), sin cortes de palabras ni desbordes de margen. |

### Archivos principales del proyecto
- **Código fuente:** `codigo/main.R`, `codigo/parametros.R`, `codigo/analisis_fin443.R`, `codigo/defensa.R`, `codigo/verificacion.R`.
- **Informe y bitácoras:** `informe.tex`, `informe.pdf`, `BITACORA_IA.md` (este documento), más las bitácoras previas en `FIN443_Contagio/`.
- **Salidas y trazabilidad:** `resultados/` (incluyendo `numeros_informe.csv`, `bitacora_ejecucion.txt`, tablas `.csv` y figuras `.png`) y `resultados_defensa/`.
