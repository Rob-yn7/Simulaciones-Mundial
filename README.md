# Simulaciones Mundial 2026 ⚽🏆

¿Quién va a ganar el Mundial 2026? Este repositorio responde con datos: scrapea todo el histórico de estadísticas de las selecciones, entrena dos modelos XGBoost y simula **1000 mundiales** con Monte Carlo para estimar el rendimiento esperado de cada selección — hasta predecir los partidos del torneo.

---

## 🔄 El pipeline, paso a paso

### 1️⃣ Scraping — [`01_Scraping/`](01_Scraping/)
Un scraper (Selenium + BeautifulSoup) se descarga **todo el histórico de estadísticas de FlashScore**: resultados, xG, posesión, remates a puerta, córneres, faltas, paradas, pases... de los últimos partidos de cada selección, más el ranking FIFA. Si no quieres scrapear desde cero, los datos ya descargados están en [`Data/`](Data/).

### 2️⃣ Limpieza de datos — [`02_Limpieza_Datos/`](02_Limpieza_Datos/)
El notebook `Data Cleaning.ipynb` hace el JOIN de las distintas fuentes descargadas y una buena limpieza para **pasar del HTML scrapeado en bruto a un dataset modelable**: una fila por partido con las estadísticas de los dos equipos.

### 3️⃣ Ingeniería de variables
Sobre los datos limpios se construyen **medias móviles (últimos 5 partidos e histórico), ratios y diferencias** entre equipos para capturar el *estado de forma* de cada selección justo antes del partido: diferencia de puntos Elo/FIFA, probabilidad implícita del Elo, tiers de nivel, pesos por confederación y diferenciales de cada estadística. Eso es lo que el modelo "ve" para predecir el resultado.

### 4️⃣ Dos modelos XGBoost — [`03_Modelado_Simulacion/`](03_Modelado_Simulacion/)
Con los datos listos se entrenan dos modelos:

1. **Modelo de goles**: dos regresores XGBoost con objetivo Tweedie (a medio camino entre Poisson y Gamma, ideal para fútbol) que predicen los goles esperados de cada equipo.
2. **Modelo de resultado**: un clasificador XGBoost 1X2 que usa como meta-variables las predicciones de goles del primero, con **probabilidades calibradas** (calibración isotónica) y validación temporal para no mezclar pasado y futuro.

Este entrenamiento y la simulación posterior se repiten notebook a notebook a medida que avanza el torneo real, reentrenando/recalculando cada vez con los cruces ya conocidos: `Poisson_16avos_VSC.ipynb`, `Poisson_octavos_VSC.ipynb`, `Poisson_cuartos_VSC.ipynb` y `Poisson_semifinales_VSC.ipynb`.

### 5️⃣ El comportamiento estocástico: Monte Carlo con 1000 mundiales
¿Por qué no basta con simular un mundial? Porque la realidad es **estocástica**. Si solo simulásemos uno, una selección con el 51% de probabilidades de pasar, pasaría exactamente igual que una con el 99%. Para capturar el efecto de la varianza, simulamos **1000 mundiales diferentes**: el equipo del 51% se clasifica en ~5.100 de ellos y el del 99% en ~9.900. Así vemos el efecto de las probabilidades en todos los cruces que tendríamos hasta la final — fase de grupos partido a partido, mejores terceros, dieciseisavos, octavos, cuartos, semifinales y final.


---

## 📊 Tabla de predicción: de dieciseisavos a semifinales

Se muestra una tirada estocástica completa del cuadro eliminatorio (partido a partido, con goles esperados, probabilidades del modelo, resultado en 90 minutos y si se resolvió en penaltis), tal como la registra `auditar_eliminatorias_detallado()` en cada notebook. Cada tabla arranca desde la ronda que le da nombre al notebook, usando ya los resultados reales conocidos hasta ese punto, y llega hasta la final.

### Desde dieciseisavos (R32)

| Fase              | Local           | Visitante          |   xG_L |   xG_V |   Prob_L (%) |   Prob_X (%) |   Prob_V (%) | Resultado 90min   | Avanza Rda   | Detalle            |
|:------------------|:----------------|:-------------------|-------:|-------:|-------------:|-------------:|-------------:|:------------------|:-------------|:-------------------|
| 1 - Dieciseisavos | Sudáfrica       | Canadá             |   1.15 |   1.4  |         23.1 |         42.7 |         34.2 | Empate            | Canadá       | Ganado en Penaltis |
| 1 - Dieciseisavos | Brasil          | Japón              |   1.34 |   1.13 |         42.1 |         38.3 |         19.6 | Japón             | Japón        | Tiempo Regular     |
| 1 - Dieciseisavos | Alemania        | Paraguay           |   1.55 |   1.05 |         66.1 |         25.6 |          8.4 | Alemania          | Alemania     | Tiempo Regular     |
| 1 - Dieciseisavos | Países Bajos    | Marruecos          |   1.33 |   1.31 |         38.8 |         24.7 |         36.5 | Empate            | Marruecos    | Ganado en Penaltis |
| 1 - Dieciseisavos | Costa de Marfil | Noruega            |   1.27 |   1.28 |         33   |         26   |         41   | Noruega           | Noruega      | Tiempo Regular     |
| 1 - Dieciseisavos | Francia         | Suecia             |   1.8  |   1.03 |         64.5 |         20.9 |         14.6 | Francia           | Francia      | Tiempo Regular     |
| 1 - Dieciseisavos | México          | Ecuador            |   1.38 |   1.23 |         39.5 |         33.3 |         27.2 | México            | México       | Tiempo Regular     |
| 1 - Dieciseisavos | Inglaterra      | RD Congo           |   1.61 |   0.97 |         64.6 |         27.1 |          8.2 | Inglaterra        | Inglaterra   | Tiempo Regular     |
| 1 - Dieciseisavos | Bélgica         | Senegal            |   1.34 |   1.26 |         28.6 |         25.6 |         45.7 | Senegal           | Senegal      | Tiempo Regular     |
| 1 - Dieciseisavos | EE. UU.         | Bosnia-Herzegovina |   1.6  |   1.14 |         39.5 |         41.1 |         19.4 | EE. UU.           | EE. UU.      | Tiempo Regular     |
| 1 - Dieciseisavos | España          | Austria            |   1.64 |   1.09 |         32.1 |         58.4 |          9.5 | España            | España       | Tiempo Regular     |
| 1 - Dieciseisavos | Portugal        | Croacia            |   1.42 |   1.28 |         38.4 |         44.8 |         16.7 | Portugal          | Portugal     | Tiempo Regular     |
| 1 - Dieciseisavos | Suiza           | Argelia            |   1.34 |   1.23 |         26.7 |         45   |         28.3 | Suiza             | Suiza        | Tiempo Regular     |
| 1 - Dieciseisavos | Australia       | Egipto             |   1.21 |   1.35 |         28.2 |         34.4 |         37.4 | Empate            | Egipto       | Ganado en Penaltis |
| 1 - Dieciseisavos | Argentina       | Cabo Verde         |   1.88 |   0.93 |         59.8 |         28.6 |         11.6 | Argentina         | Argentina    | Tiempo Regular     |
| 1 - Dieciseisavos | Colombia        | Ghana              |   1.72 |   0.95 |         52.9 |         25.4 |         21.7 | Empate            | Colombia     | Ganado en Penaltis |
| 2 - Octavos       | Canadá          | Japón              |   1.15 |   1.32 |         16.9 |         57.4 |         25.7 | Canadá            | Canadá       | Tiempo Regular     |
| 2 - Octavos       | Alemania        | Marruecos          |   1.35 |   1.31 |         30.6 |         41.9 |         27.5 | Alemania          | Alemania     | Tiempo Regular     |
| 2 - Octavos       | Noruega         | Francia            |   1.06 |   1.53 |         20.2 |         24.4 |         55.3 | Francia           | Francia      | Tiempo Regular     |
| 2 - Octavos       | México          | Inglaterra         |   1.18 |   1.51 |         17.8 |         29.2 |         53   | México            | México       | Tiempo Regular     |
| 2 - Octavos       | Senegal         | EE. UU.            |   1.34 |   1.26 |         49.2 |         28.5 |         22.2 | Empate            | Senegal      | Ganado en Penaltis |
| 2 - Octavos       | España          | Portugal           |   1.42 |   1.2  |         26.1 |         58.9 |         15   | Portugal          | Portugal     | Tiempo Regular     |
| 2 - Octavos       | Suiza           | Egipto             |   1.37 |   1.22 |         24.4 |         54.9 |         20.6 | Egipto            | Egipto       | Tiempo Regular     |
| 2 - Octavos       | Argentina       | Colombia           |   1.53 |   1.13 |         47.4 |         39.3 |         13.3 | Empate            | Argentina    | Ganado en Penaltis |
| 3 - Cuartos       | Canadá          | Alemania           |   1.11 |   1.54 |         13.8 |         37.7 |         48.5 | Empate            | Alemania     | Ganado en Penaltis |
| 3 - Cuartos       | Francia         | México             |   1.51 |   1.22 |         53.3 |         24.1 |         22.6 | Francia           | Francia      | Tiempo Regular     |
| 3 - Cuartos       | Senegal         | Portugal           |   1.22 |   1.34 |         25.9 |         47   |         27.1 | Senegal           | Senegal      | Tiempo Regular     |
| 3 - Cuartos       | Egipto          | Argentina          |   1.01 |   1.61 |         15.4 |         50.3 |         34.3 | Argentina         | Argentina    | Tiempo Regular     |
| 4 - Semifinales   | Alemania        | Francia            |   1.27 |   1.47 |         25.8 |         38.1 |         36.2 | Alemania          | Alemania     | Tiempo Regular     |
| 4 - Semifinales   | Senegal         | Argentina          |   1.16 |   1.45 |         25.2 |         46   |         28.8 | Argentina         | Argentina    | Tiempo Regular     |
| 5 - Final         | Alemania        | Argentina          |   1.25 |   1.45 |         22.1 |         56.5 |         21.4 | Empate            | Argentina    | Ganado en Penaltis |

### Desde octavos de final

| Fase            | Local          | Visitante      |   xG_L |   xG_V |   Prob_L (%) |   Prob_X (%) |   Prob_V (%) | Resultado 90min   | Avanza Rda     | Detalle            |
|:----------------|:---------------|:---------------|-------:|-------:|-------------:|-------------:|-------------:|:------------------|:---------------|:-------------------|
| 1 - Octavos     | Canadá         | Marruecos      |   1.14 |   1.53 |         16.4 |         35   |         48.6 | Marruecos         | Marruecos      | Tiempo Regular     |
| 1 - Octavos     | Paraguay       | Francia        |   0.98 |   1.81 |         10.8 |         18.3 |         70.9 | Empate            | Francia        | Ganado en Penaltis |
| 1 - Octavos     | Brasil         | Noruega        |   1.44 |   1.13 |         54.4 |         25.1 |         20.5 | Empate            | Noruega        | Ganado en Penaltis |
| 1 - Octavos     | México         | Inglaterra     |   1.18 |   1.51 |         17.8 |         29.2 |         53   | Inglaterra        | Inglaterra     | Tiempo Regular     |
| 1 - Octavos     | Portugal       | España         |   1.2  |   1.42 |         15   |         58.9 |         26.1 | Empate            | España         | Ganado en Penaltis |
| 1 - Octavos     | Estados Unidos | Bélgica        |   1.68 |   1.56 |         20.4 |         48   |         31.5 | Empate            | Estados Unidos | Ganado en Penaltis |
| 1 - Octavos     | Argentina      | Egipto         |   1.61 |   1.01 |         34.3 |         50.3 |         15.4 | Empate            | Egipto         | Ganado en Penaltis |
| 1 - Octavos     | Suiza          | Colombia       |   1.34 |   1.33 |         26.6 |         35.6 |         37.8 | Suiza             | Suiza          | Tiempo Regular     |
| 2 - Cuartos     | Marruecos      | Francia        |   1.21 |   1.46 |         28.6 |         29.4 |         41.9 | Francia           | Francia        | Tiempo Regular     |
| 2 - Cuartos     | Noruega        | Inglaterra     |   1.13 |   1.51 |         17.2 |         35.4 |         47.3 | Inglaterra        | Inglaterra     | Tiempo Regular     |
| 2 - Cuartos     | España         | Estados Unidos |   1.55 |   1.55 |         30.8 |         54.3 |         14.9 | España            | España         | Tiempo Regular     |
| 2 - Cuartos     | Egipto         | Suiza          |   1.22 |   1.37 |         20.6 |         54.9 |         24.4 | Suiza             | Suiza          | Tiempo Regular     |
| 3 - Semifinales | Francia        | Inglaterra     |   1.39 |   1.35 |         24.4 |         44.6 |         31   | Empate            | Inglaterra     | Ganado en Penaltis |
| 3 - Semifinales | España         | Suiza          |   1.52 |   1.12 |         52.3 |         33.3 |         14.4 | España            | España         | Tiempo Regular     |
| 4 - Final       | Inglaterra     | España         |   1.25 |   1.39 |         18.9 |         61.9 |         19.2 | Inglaterra        | Inglaterra     | Tiempo Regular     |

### Desde cuartos de final

| Fase            | Local     | Visitante   |   xG_L |   xG_V |   Prob_L (%) |   Prob_X (%) |   Prob_V (%) | Resultado 90min   | Avanza Rda   | Detalle            |
|:----------------|:----------|:------------|-------:|-------:|-------------:|-------------:|-------------:|:------------------|:-------------|:-------------------|
| 1 - Cuartos     | Francia   | Marruecos   |   1.46 |   1.21 |         41.9 |         29.4 |         28.6 | Empate            | Marruecos    | Ganado en Penaltis |
| 1 - Cuartos     | España    | Bélgica     |   1.53 |   1.14 |         37.4 |         50   |         12.6 | Bélgica           | Bélgica      | Tiempo Regular     |
| 1 - Cuartos     | Noruega   | Inglaterra  |   1.13 |   1.51 |         17.2 |         35.4 |         47.3 | Empate            | Noruega      | Ganado en Penaltis |
| 1 - Cuartos     | Argentina | Suiza       |   1.54 |   1.18 |         46.3 |         34.9 |         18.7 | Suiza             | Suiza        | Tiempo Regular     |
| 2 - Semifinales | Marruecos | Bélgica     |   1.37 |   1.35 |         45.4 |         30.2 |         24.4 | Marruecos         | Marruecos    | Tiempo Regular     |
| 2 - Semifinales | Noruega   | Suiza       |   1.3  |   1.32 |         28.5 |         41.8 |         29.8 | Noruega           | Noruega      | Tiempo Regular     |
| 3 - Final       | Marruecos | Noruega     |   1.42 |   1.2  |         47.7 |         28   |         24.3 | Marruecos         | Marruecos    | Tiempo Regular     |

### Desde semifinales

| Fase            | Local      | Visitante   |   xG_L |   xG_V |   Prob_L (%) |   Prob_X (%) |   Prob_V (%) | Resultado 90min   | Avanza Rda   | Detalle            |
|:----------------|:-----------|:------------|-------:|-------:|-------------:|-------------:|-------------:|:------------------|:-------------|:-------------------|
| 1 - Semifinales | Francia    | España      |   1.31 |   1.39 |         27.7 |         45   |         27.3 | Francia           | Francia      | Tiempo Regular     |
| 1 - Semifinales | Inglaterra | Argentina   |   1.33 |   1.37 |         21.5 |         61.8 |         16.7 | Inglaterra        | Inglaterra   | Tiempo Regular     |
| 2 - Final       | Francia    | Inglaterra  |   1.39 |   1.35 |         24.4 |         44.6 |         31   | Empate            | Inglaterra   | Ganado en Penaltis |

---

## 🏆 Tabla de probabilidades de campeón

A diferencia de la tabla anterior (una única tirada estocástica), aquí se agregan las **1000 simulaciones de Monte Carlo**: para cada selección clasificada, el porcentaje de veces que alcanzó cada ronda. Las columnas son acumulativas — la probabilidad de llegar a Cuartos ya incluye haber superado R32 y Octavos. Igual que arriba, hay una tabla por notebook, según la ronda real en la que quedó el torneo al momento de correr la simulación.

### Desde dieciseisavos (R32)

| Selección | R32 | Octavos | Cuartos | Semis | Final | Campeón |
|---|---|---|---|---|---|---|
| Inglaterra | 100.0 | 88.4 | 71.3 | 45.0 | 28.8 | 18.0 |
| Argentina | 100.0 | 85.1 | 67.9 | 48.7 | 25.8 | 13.7 |
| España | 100.0 | 78.1 | 52.9 | 38.1 | 24.1 | 13.7 |
| Francia | 100.0 | 80.4 | 57.4 | 30.6 | 18.1 | 10.9 |
| Brasil | 100.0 | 65.3 | 52.1 | 30.1 | 15.0 | 8.8 |
| Senegal | 100.0 | 58.1 | 43.2 | 19.9 | 10.4 | 5.7 |
| Alemania | 100.0 | 88.5 | 40.9 | 23.4 | 10.7 | 5.2 |
| Portugal | 100.0 | 69.4 | 29.5 | 17.8 | 11.8 | 5.1 |
| Países Bajos | 100.0 | 51.5 | 28.6 | 15.6 | 7.8 | 3.8 |
| Marruecos | 100.0 | 48.5 | 28.5 | 15.3 | 7.3 | 3.5 |
| Argelia | 100.0 | 53.1 | 32.7 | 14.5 | 6.4 | 3.0 |
| Croacia | 100.0 | 30.6 | 11.0 | 6.3 | 3.4 | 1.2 |
| México | 100.0 | 59.6 | 18.3 | 8.5 | 2.7 | 1.1 |
| Bélgica | 100.0 | 41.9 | 29.9 | 9.7 | 5.1 | 0.8 |
| Japón | 100.0 | 34.7 | 23.1 | 9.7 | 2.3 | 0.8 |
| Canadá | 100.0 | 57.7 | 17.1 | 4.1 | 2.1 | 0.7 |
| Noruega | 100.0 | 55.4 | 17.4 | 5.2 | 2.0 | 0.7 |
| EE. UU. | 100.0 | 66.0 | 21.8 | 5.2 | 1.6 | 0.7 |
| Egipto | 100.0 | 58.9 | 25.9 | 9.1 | 3.8 | 0.6 |
| Suiza | 100.0 | 46.9 | 26.6 | 10.7 | 2.6 | 0.4 |
| Colombia | 100.0 | 69.9 | 20.5 | 10.1 | 2.5 | 0.4 |
| Costa de Marfil | 100.0 | 44.6 | 17.3 | 5.9 | 1.7 | 0.4 |
| Sudáfrica | 100.0 | 42.3 | 7.7 | 1.5 | 0.6 | 0.4 |
| Australia | 100.0 | 41.1 | 14.8 | 5.2 | 1.4 | 0.1 |
| Austria | 100.0 | 21.9 | 6.6 | 2.5 | 0.7 | 0.1 |
| Ecuador | 100.0 | 40.4 | 6.9 | 2.6 | 0.5 | 0.1 |
| Cabo Verde | 100.0 | 14.9 | 5.9 | 1.3 | 0.2 | 0.1 |
| RD Congo | 100.0 | 11.6 | 3.5 | 0.9 | 0.2 | 0.0 |
| Suecia | 100.0 | 19.6 | 7.9 | 1.3 | 0.1 | 0.0 |
| Bosnia-Herzegovina | 100.0 | 34.0 | 5.1 | 0.5 | 0.1 | 0.0 |
| Ghana | 100.0 | 30.1 | 5.7 | 0.4 | 0.1 | 0.0 |
| Paraguay | 100.0 | 11.5 | 2.0 | 0.3 | 0.1 | 0.0 |

### Desde octavos de final

| Selección | Octavos | Cuartos | Semis | Final | Campeón |
|---|---|---|---|---|---|
| Inglaterra | 100.0 | 76.5 | 46.6 | 29.3 | 19.9 |
| España | 100.0 | 61.4 | 45.8 | 31.8 | 16.3 |
| Francia | 100.0 | 85.7 | 53.2 | 27.4 | 16.0 |
| Brasil | 100.0 | 70.6 | 35.7 | 19.8 | 12.5 |
| Argentina | 100.0 | 68.8 | 50.8 | 22.7 | 9.0 |
| Marruecos | 100.0 | 76.5 | 36.0 | 15.9 | 8.6 |
| Portugal | 100.0 | 38.6 | 24.3 | 16.1 | 6.8 |
| Bélgica | 100.0 | 61.1 | 15.9 | 7.9 | 3.2 |
| Estados Unidos | 100.0 | 38.9 | 14.0 | 5.8 | 1.9 |
| Colombia | 100.0 | 61.0 | 20.7 | 5.9 | 1.3 |
| Noruega | 100.0 | 29.4 | 9.2 | 2.5 | 1.3 |
| Suiza | 100.0 | 39.0 | 14.2 | 5.1 | 0.9 |
| Canadá | 100.0 | 23.5 | 8.5 | 2.5 | 0.8 |
| Egipto | 100.0 | 31.2 | 14.3 | 4.7 | 0.7 |
| México | 100.0 | 23.5 | 8.5 | 2.3 | 0.7 |
| Paraguay | 100.0 | 14.3 | 2.3 | 0.3 | 0.1 |

### Desde cuartos de final

| Selección | Cuartos | Semis | Final | Campeón |
|---|---|---|---|---|
| Inglaterra | 100.0 | 73.2 | 45.8 | 25.3 |
| España | 100.0 | 75.7 | 43.8 | 23.0 |
| Argentina | 100.0 | 70.4 | 34.5 | 17.7 |
| Francia | 100.0 | 58.4 | 30.9 | 17.2 |
| Marruecos | 100.0 | 41.6 | 17.0 | 7.8 |
| Noruega | 100.0 | 26.8 | 11.1 | 3.7 |
| Bélgica | 100.0 | 24.3 | 8.3 | 3.3 |
| Suiza | 100.0 | 29.6 | 8.6 | 2.0 |

### Desde semifinales

| Selección | Semis | Final | Campeón |
|---|---|---|---|
| España | 100.0 | 50.1 | 27.3 |
| Inglaterra | 100.0 | 55.5 | 27.1 |
| Francia | 100.0 | 49.9 | 26.1 |
| Argentina | 100.0 | 44.5 | 19.5 |

> Todos los valores están en porcentaje (%), generados con `estadisticas_mundial.sort_values('Campeon', ascending=False)` al final de cada notebook.

---

## 📂 Estructura del repositorio

```
├── 01_Scraping/              # Extracción de datos de FlashScore (Selenium)
│   └── Scrapper.py
├── 02_Limpieza_Datos/        # JOIN y limpieza → dataset modelable
│   └── Data Cleaning.ipynb
├── 03_Modelado_Simulacion/   # Ingeniería de variables, XGBoost y Monte Carlo
│   ├── Poisson_16avos_VSC.ipynb
│   ├── Poisson_octavos_VSC.ipynb
│   ├── Poisson_cuartos_VSC.ipynb
│   └── Poisson_semifinales_VSC.ipynb
├── Data/                     # Datos scrapeados y procesados (CSV)
```