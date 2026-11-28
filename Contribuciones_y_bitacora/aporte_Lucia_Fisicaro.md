# Aporte de Lucía Fisicaro · FIN443 E2

Usuario GitHub: Micaa1725 · RUT: [completar]

---

## 1. Para `contribuciones.md` (fila de Lucía)

| Integrante | RUT | Usuario GitHub | Secciones del informe | Código | Datos | Referencias (n° en referencias_E2.csv) |
|---|---|---|---|---|---|---|
| Lucía Fisicaro | [completar] | Micaa1725 | 2 (Datos y fuentes), 6 (Validación: prueba de replicabilidad en Windows) y revisión final de cifras y citas de todo el informe | Revisión crítica del código, pruebas de ejecución, especificación de la versión optimizada (test analítico FR, DCC vectorizado, `parametros.R`, `defensa.R`, `verificacion.R`, `numeros_informe.csv`) | Búsqueda, descarga y validación de tipos de cambio (6 monedas), S&P Merval y COLCAP; revisión de calidad de las bases iniciales; subida de `datos/originales/` | 5 (Engle, 2002), 9 (Longin y Solnik, 2001), 15 (Bekaert, Harvey, Kiguel y Wang, 2016) |

### Detalle de lo que hice

**Datos (23 de noviembre)**
- Revisé las bases iniciales del grupo (IPSA, Ibovespa, IPC, S&P 500, VIX y S&P/BVL Perú): rango de fechas, frecuencia, formato, huecos, datos repetidos y valores extremos. Detecté que la serie de Perú viene en variación diaria y no en precios.
- Descargué los tipos de cambio diarios de 2005 a 2025 de las 6 monedas (CLP, BRL, MXN, ARS, COP y PEN) desde Investing.com. Como el sitio limita cada descarga a 5.000 filas, los bajé en dos tramos (2005–2014 y 2015–2025) y validé que llegaran completos.
- Cuando Bloomberg no tuvo licencia para el Merval y el COLCAP (los dos pasaron a ser índices MSCI), conseguí ambas series en Investing.com y verifiqué su cobertura: el Merval es completo desde 2005 y el COLCAP parte el 11-01-2008. Ese dato fundamentó la decisión de excluir a Colombia.

**GitHub (27 de noviembre)**
- Subí `datos/originales/` con todos los archivos fuente: tipos de cambio, Merval, COLCAP e índices. Corregí el nombre de la carpeta a minúsculas para que coincidiera con la ruta del código.
- Subí `referencias_E2.csv` al repositorio.

**Revisión y pruebas del código (27 de noviembre)**
- Corrí en mi computador (Windows, R 4.3.1) la versión anterior del código. Tardó **30,4 minutos**, sobre el límite de 15 del instructivo, y eso hizo evidente que había que optimizarlo.
- Revisé con IA la versión simplificada del grupo y anoté los problemas encontrados:
  - el contagio en las PASO aparecía solo por el piso del valor p del bootstrap (1/2.000 × 90 pruebas de Holm);
  - `defensa.R` daba resultados distintos a `main.R`;
  - 13 de las 16 referencias no se usaban en el informe;
  - las cifras del informe no estaban trazadas al código;
  - había parámetros repetidos dentro del código y un visor HTML innecesario.
- Pedí y validé una versión corregida con test analítico de Forbes-Rigobon (más bootstrap como robustez), DCC vectorizado, parámetros centralizados, trazabilidad (`numeros_informe.csv`), modo defensa coherente con el main, script de verificación y tres análisis cortos para usar las referencias de Longin y Solnik, Bae et al. y Diebold y Yilmaz. La versión final del grupo se construyó sobre ella.
- Corrí la versión nueva en mi computador: **0,90 minutos en Windows**, con una ruta con tildes, sin errores.

**Revisión final (28 de noviembre)**
- Verifiqué con IA la entrega final:
  - la replicabilidad, corriendo el código desde una carpeta vacía (diferencias de orden 1e-14);
  - los 494 números del informe contra `resultados/`;
  - las citas, contra los PDF de los papers.
- Detecté y corregí en el informe:
  - una atribución incorrecta: el LOCF se atribuía a Chiang et al. (2007), pero el paper no lo menciona;
  - el tiempo de ejecución (0,13 en el informe vs 0,38 minutos en la bitácora);
  - el MD5 de la base (aparecía el del Excel en vez del CSV que realmente se lee);
  - la descripción de la regresión de Chiang et al.
- Agregué las fuentes de los datos al LEEME.

---

## 2. Para `bitacora_ia.md` (entradas de Lucía)

**Herramienta:** Claude (Anthropic). **Datos sensibles:** no compartí RUT ni datos personales. A la IA solo le pasé bases de mercado públicas, código y documentos del curso.

### 23-nov · Revisión de bases y descarga de datos
- **Prompts principales:**
  - "Ayúdame a entender qué quiere el profe, qué vamos a hacer y cuál es mi tarea."
  - "Revisa que las bases estén bien. ¿Debo buscar? ¿De dónde y cuáles?"
  - "¿Podría buscar el tipo de cambio yo? ¿Dónde lo busco?"
- **Qué propuso la IA:** un checklist de revisión (fechas, frecuencia, formato, huecos y moneda) y un análisis automático de las 6 bases con código; fuentes para los tipos de cambio (Yahoo, Google Sheets, bancos centrales, Investing.com) y descargar en dos tramos por el límite de filas.
- **Qué decidí:** usar Investing.com para las 6 monedas y para el Merval y el COLCAP.
- **Qué verifiqué:** con la IA revisé cada archivo descargado (rango, número de días, huecos y saltos). Detecté descargas incompletas (se bajaba solo el último mes porque el rango no se aplicaba) y las repetí hasta que las 6 monedas quedaron completas de 2005 a 2025.
- **Errores de la IA:** sugirió descargar desde Yahoo Finance, pero el botón de descarga ahora es solo para Premium, así que se cambió de fuente. La IA identificó (con búsqueda web) que el Merval y el COLCAP pasaron a ser índices MSCI, lo que explicaba la falta de licencia en Bloomberg.

### 27-nov · Revisión del código y prueba de replicabilidad
- **Prompts principales:**
  - "Revisa si está bien, dime qué está bien, qué no y qué modificar."
  - "Dame las recomendaciones de por qué va lento y cómo optimizarlo."
  - "Analiza el código, todos los errores, lo que falta, lo que se podría optimizar" (adjuntando las instrucciones generales de IA del grupo).
  - "Modifica y haz todo lo que habías dicho: que cumpla todo lo que pide el instructivo y que no tarde mucho."
- **Qué propuso la IA:** la lista de errores (piso del p bootstrap con Holm, defensa distinta del main, referencias sin usar, falta de trazabilidad, números mágicos, visor HTML) y 3 opciones para el p de Forbes-Rigobon: (1) test analítico como principal y bootstrap como robustez, (2) Holm por crisis, (3) más réplicas.
- **Qué se decidió:** la opción 1, sacar el visor, centralizar parámetros y agregar tres análisis cortos para usar las referencias. [El grupo debe confirmar que esta decisión fue acordada.]
- **Qué verifiqué:**
  - corrí la versión anterior en mi PC: 30,4 minutos;
  - la nueva versión da las mismas cifras que la anterior en DCC, carteras, correlaciones y canal cambiario;
  - `defensa.R` reproduce exactamente el main;
  - dos corridas dan archivos idénticos;
  - una base con errores puestos a propósito (fechas dd/mm, punto y coma, comas decimales, duplicado, faltante, desorden) da los mismos resultados;
  - pasan las 11 pruebas de `verificacion.R`;
  - la nueva versión corrió en mi PC en 0,90 minutos.
- **Errores de la IA:** una prueba de `verificacion.R` falló al principio porque no consideraba la regularización 1e-8 de la diagonal en `min_variance`; se corrigió la prueba. La figura de colas salió con la leyenda tapando los puntos; se corrigió.

### 28-nov · Verificación final antes de entregar
- **Prompt principal:** "Revisa si no falla nada, si es replicable, si cada cifra del informe está respaldada por el código y si las citas están bien y corroboran lo que hicimos" (adjuntando los 16 PDF).
- **Qué hizo la IA:** corrió el código desde cero y comparó los resultados; cruzó los 494 números del informe con `resultados/`; revisó las citas contra el texto de los PDF.
- **Qué se encontró y corrigió:**
  - la atribución del LOCF a Chiang et al. (2007), que el paper no respalda;
  - el tiempo de 0,13 minutos, que no calzaba con la bitácora (0,38 minutos);
  - el MD5 del Excel, citado como si fuera el del CSV;
  - la frase que atribuía a Chiang et al. el control por VIX.
  
  Además faltaban `referencias_E2.csv` actualizado, `contribuciones.md` y `.gitignore`, y había que sacar `Papers/` del repositorio público.
- **Qué verifiqué yo:** [completar: por ejemplo, revisé en el PDF de Chiang et al. la sección de datos y confirmé que no menciona LOCF].
