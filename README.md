# AstroShield: evolucionar una constelación inteligente

Proyecto de Fundamentos de Inteligencia Artificial (LITIID73). Un algoritmo evolutivo diseña una constelación de 8 satélites que observa seis regiones de la Tierra, respetando zonas de desechos espaciales y posiciones ocupadas, y el resultado se explora en un globo 3D.

[![Abrir en Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/AlexRecaman/astroshield/blob/main/astroshield.ipynb)

## Cómo ejecutarlo

1. Abrir el notebook con el botón de arriba (o en Colab: *Archivo → Abrir cuaderno → GitHub* y buscar `AlexRecaman/astroshield`).
2. *Entorno de ejecución → Ejecutar todo*. Usa `numpy`, `pandas`, `scipy`, `matplotlib` y `plotly`, que ya vienen en Colab. Tarda unos minutos.
3. Local: `pip install numpy pandas scipy matplotlib plotly jupyter` y `jupyter notebook astroshield.ipynb`.

Todas las ejecuciones usan semillas fijas, así que son repetibles. Las cifras del análisis (§14) se calculan desde los resultados al ejecutar; no están escritas a mano.

## Contenido del repositorio

| Archivo | Qué es |
|---|---|
| `astroshield.ipynb` | Notebook completo: definición, modelo, algoritmo, restricciones, pruebas, globo 3D y análisis |
| `docs/definicion_problema.md` (`.docx`, `.pdf`) | Entregable 1: decisiones que convierten el escenario en un problema resoluble |
| `globo_astroshield.html` | Globo 3D exportado de una ejecución (se abre en cualquier navegador) |

## Estructura del notebook y entregables

| Sección | Qué contiene | Entregable |
|---|---|---|
| §0 a §1 | Cómo ejecutar y definición del problema | 1 |
| §2 a §3 | Escenario (datos editables), geometría esférica y modelo de cobertura, con pruebas | 2 |
| §4 a §6 | Función de calidad, algoritmo evolutivo y etapa 1 (sin restricciones) | 2 y 3 |
| §7 a §9 | Restricciones espaciales (zonas, posiciones ocupadas), etapa 2 y cambio de condiciones | 2 y 3 |
| §10 | Pruebas: zonas, posiciones ocupadas, espacio reducido, condición inviable | 3 |
| §11 | Globo 3D: configuración inicial frente a final y casos difíciles | 5 |
| §12 a §13 | Variación entre 10 ejecuciones y prueba propia (algoritmo evolutivo frente a búsqueda aleatoria) | 3 |
| §14 | Análisis de resultados con datos de las ejecuciones | 4 |
| §15 | Uso de IA y verificaciones | 6 |

## Modelo en una línea

Una configuración es una matriz 8×2 de puntos subsatélite (lat, lon). Una región está cubierta si al menos el 90 % de sus puntos de muestra queda dentro de la huella (ρ ≈ 7.1°) de algún satélite. La calidad suma la cobertura ponderada por prioridad (Alta 3, Media 2, Normal 1) más un bono pequeño por doble cobertura. Una configuración con un satélite en una zona prohibida es inválida y siempre pierde contra una válida. Los detalles están en `docs/definicion_problema.md`.

## Equipo y commits

El notebook se construyó por partes, cada una subida en un commit de su autor:

| Parte | Responsable | Secciones |
|---|---|---|
| 1 | @AlexRecaman | §0 a §3: definición, escenario, geometría y cobertura |
| 2 | @gretelZavaleta | §4 a §6: calidad, algoritmo evolutivo, etapa 1 |
| 3 | @Henrrucho | §7 a §10: restricciones, cambio de condiciones y pruebas |
| 4 | @TITAFA | §11 a §15: globo 3D, variación, prueba propia, análisis |

## Uso de IA

El código, la estructura del notebook y los textos se generaron con Claude (Claude Code) a partir del enunciado y de las decisiones de diseño que el equipo revisó. Las verificaciones realizadas (pruebas con `assert`, referencia teórica, comparación con búsqueda aleatoria, diez semillas, comprobación de la figura contra el resultado) están descritas en la sección §15 del notebook.

## Limitaciones principales

El modelo es una "foto" sin órbitas ni tiempo, las huellas son círculos iguales y los umbrales (cobertura 0.9, pesos 3-2-1, β = 0.25) son decisiones del modelo. Que el algoritmo deje de mejorar no demuestra que encontró la mejor configuración posible. El análisis de §14 cuantifica estas limitaciones con resultados reales.
