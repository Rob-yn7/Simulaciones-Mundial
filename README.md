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
Con los datos listos, en `Modelling.ipynb` se entrenan dos modelos:

1. **Modelo de goles**: dos regresores XGBoost con objetivo Tweedie (a medio camino entre Poisson y Gamma, ideal para fútbol) que predicen los goles esperados de cada equipo.
2. **Modelo de resultado**: un clasificador XGBoost 1X2 que usa como meta-variables las predicciones de goles del primero, con **probabilidades calibradas** (calibración isotónica) y validación temporal para no mezclar pasado y futuro.

### 5️⃣ El comportamiento estocástico: Monte Carlo con 1000 mundiales
¿Por qué no basta con simular un mundial? Porque la realidad es **estocástica**. Si solo simulásemos uno, una selección con el 51% de probabilidades de pasar, pasaría exactamente igual que una con el 99%. Para capturar el efecto de la varianza, simulamos **1000 mundiales diferentes**: el equipo del 51% se clasifica en ~5.100 de ellos y el del 99% en ~9.900. Así vemos el efecto de las probabilidades en todos los cruces que tendríamos hasta la final — fase de grupos partido a partido, mejores terceros, dieciseisavos, octavos, cuartos, semifinales y final.


---

## 📂 Estructura del repositorio

```
├── 01_Scraping/              # Extracción de datos de FlashScore (Selenium)
│   └── Scrapper.py
├── 02_Limpieza_Datos/        # JOIN y limpieza → dataset modelable
│   └── Data Cleaning.ipynb
├── 03_Modelado_Simulacion/   # Ingeniería de variables, XGBoost y Monte Carlo
│   └── Poisson_octavos_VSC.ipynb
├── Data/                     # Datos scrapeados y procesados (CSV)
```

