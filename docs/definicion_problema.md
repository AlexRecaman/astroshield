# AstroShield: definición del problema (Entregable 1)

Fundamentos de Inteligencia Artificial · LITIID73

Equipo: @AlexRecaman, @gretelZavaleta, @Henrrucho, @TITAFA

Documento previo a la implementación: recoge las decisiones que convierten el escenario del enunciado en un problema que un algoritmo evolutivo puede resolver. Las secciones 1.1 a 1.8 son las mismas del notebook `astroshield.ipynb` (§1).

## 1.1 Supuestos del modelo

- La Tierra es una esfera de radio 6371 km. Las posiciones se expresan en latitud y longitud (grados).
- Cada satélite se modela por su **punto subsatélite**: el punto de la superficie que tiene justo debajo. No se simula movimiento orbital; la configuración es una "foto" de la constelación. El enunciado permite esta simplificación.
- Todos los satélites están a la misma altura y ven un círculo de la superficie (su **huella**) cuyo radio depende de esa altura y de la elevación mínima a la que el sensor puede observar.
- Las zonas de desechos y las posiciones ocupadas son círculos sobre la esfera. Restringen **dónde puede estar un satélite**, no lo que puede observar: un satélite fuera de una zona puede observar una región que esté debajo de ella.

## 1.2 Representación de una configuración

Una configuración es una matriz de **8 × 2**: una fila por satélite con su `(latitud, longitud)` en grados. Es el genoma del algoritmo evolutivo. El orden de los satélites no importa: dos matrices con las mismas filas en distinto orden son la misma constelación.

## 1.3 Cuándo una región está cubierta

- **Radio de la huella.** Para altura `h = 550 km` y elevación mínima `ε = 30°`, el ángulo central de la huella es
  `ρ = 90° − ε − asin(R·cos ε / (R + h))` ≈ **7.1°** (≈ 790 km sobre la superficie). Un punto está dentro de la huella si su distancia angular al punto subsatélite es ≤ ρ.
- **Área de la región.** Cada región es un disco de 300 km de radio (≈ 2.7°) alrededor de su centro. Se representa con **37 puntos**: el centro y tres anillos de 12 puntos a 1/3, 2/3 y 3/3 del radio.
- **Fracción cubierta `f`.** Proporción de esos 37 puntos que quedan dentro de al menos una huella.
- **Fracción redundante `d`.** Proporción de los puntos que quedan dentro de dos o más huellas.
- Una región está **cubierta** si `f ≥ 0.9`.

## 1.4 Calidad de una configuración

Las prioridades se traducen en pesos: **Alta = 3, Media = 2, Normal = 1** (suma 13 para las seis regiones). La calidad es

`F = Σ wᵣ·fᵣ / Σ wᵣ  +  β · Σ wᵣ·dᵣ / Σ wᵣ`,  con `β = 0.25`.

- El primer término (máximo 1) premia cubrir las regiones, más las de prioridad alta.
- El segundo premia la redundancia: con 8 satélites y 6 regiones sobran 2, y conviene usarlos para observar dos veces las regiones más importantes. `β` es pequeño para que la redundancia nunca valga más que cubrir una región nueva.
- Como ninguna pareja de regiones está a menos de 2·(ρ + 2.7°), ningún satélite puede cubrir dos regiones a la vez. El máximo alcanzable es entonces `1 + 0.25·(3 + 3)/13 ≈ 1.115`: todo cubierto y las dos regiones de prioridad Alta con doble cobertura. Ese valor sirve como referencia para medir qué tan cerca queda el algoritmo.

## 1.5 Configuraciones inválidas

Una configuración es inválida si algún satélite:

- está a una distancia angular menor que el radio de una de las tres **zonas de desechos** (Atlántico Norte 12°, Pacífico Sur 10°, polar norte 8°);
- está a menos de 5° de los **objetos A o B**;
- está a menos de **1°** de otro satélite de la constelación. Esta restricción la agrega el equipo para impedir dos satélites en la misma posición.

La **violación** de una configuración es la suma de cuántos grados invade cada satélite cada zona prohibida. Vale 0 si la configuración es válida.

## 1.6 Estrategia evolutiva

Algoritmo genético con codificación real:

- **Población inicial:** posiciones al azar distribuidas uniformemente sobre la esfera; con restricciones, solo posiciones válidas (muestreo con rechazo).
- **Selección:** torneo de 3 individuos.
- **Cruce uniforme por satélite:** el hijo toma cada satélite completo de uno de los dos padres. Se intercambian satélites, no coordenadas sueltas, para no romper posiciones que ya funcionan.
- **Mutación en dos escalas:** desplazamiento gaussiano de un satélite, la mitad de las veces grande (σ = 6°, explora) y la otra mitad fino (σ = 1.5°, ajusta). Con baja probabilidad, reubicación total del satélite en un punto al azar para escapar de óptimos locales. En pruebas preliminares con σ fijo de 6° el algoritmo se quedaba cerca del óptimo sin poder afinar la doble cobertura, porque para que dos satélites cubran por completo la misma región ambos deben quedar a menos de ≈ 4.4° de su centro.
- **Elitismo:** los 2 mejores pasan sin cambios a la siguiente generación.
- **Restricciones:** reparación (si una mutación deja un satélite en zona prohibida se busca la posición válida más cercana) y **regla de Deb** al comparar: una configuración válida siempre gana a una inválida, y entre inválidas gana la de menor violación. El resultado que se reporta es siempre la mejor configuración válida; si no hay ninguna, se informa.
- **Registro:** en cada generación se guarda la mejor, media y peor calidad, cuántos individuos son válidos y cuántas regiones cubre el mejor.

## 1.7 Parámetros y valores iniciales

| Parámetro | Valor inicial | Por qué |
|---|---|---|
| Satélites | 8 | Dato del problema |
| Altura / elevación mínima | 550 km / 30° | Órbita baja típica de observación; da una huella de ≈ 7° que alcanza para una región pero no para dos |
| Umbral de región cubierta | f ≥ 0.9 | Tolera que quede fuera el borde de la región |
| β (redundancia) | 0.25 | Menor que el peso de una región Normal (1/13 ≈ 0.077 frente a 0.25·3/13 ≈ 0.058 por región Alta duplicada) |
| Población / generaciones | 80 / 300 | Ajustado con 10 semillas: con 60 individuos y 40 generaciones de paciencia algunas corridas se quedaban con 5 regiones |
| Torneo | 3 | Presión de selección moderada |
| Prob. de cruce / mutación / reubicación | 0.9 / 0.2 por satélite / 0.05 | Valores habituales; se revisan con las pruebas |
| σ de mutación | 6° o 1.5° (50 % cada una) | 6° es del orden del radio de la huella; 1.5° permite afinar la doble cobertura |
| Elitismo | 2 | Conserva la mejor solución sin reducir la diversidad |
| Paciencia (parada) | 80 generaciones sin mejora | Corta ejecuciones estancadas sin detener las que aún mejoran despacio |

## 1.8 Cuándo una solución es suficientemente buena

- Las 6 regiones cubiertas (`f ≥ 0.9`).
- Las 2 regiones de prioridad Alta con redundancia.
- Violación 0 (cumple todas las restricciones).
- Calidad cercana al máximo de referencia (≈ 1.115).
- El algoritmo supera a una búsqueda aleatoria con el mismo número de evaluaciones.

Que el algoritmo deje de mejorar no prueba que encontró la mejor solución; por eso se compara con la referencia teórica, con varias semillas y con la búsqueda aleatoria.

## 1.9 Datos del escenario

| Región | Centro (lat, lon) | Prioridad | Peso |
|---|---|---|---|
| Península de Yucatán | 20.5, -89.0 | Alta | 3 |
| Amazonía brasileña | -3.5, -62.0 | Alta | 3 |
| California | 36.8, -119.4 | Media | 2 |
| Sahel | 15.0, 0.0 | Media | 2 |
| Japón | 36.0, 138.0 | Media | 2 |
| Interior de Australia | -25.0, 133.0 | Normal | 1 |

Radio de observación inicial: 300 km. Zonas de desechos: Atlántico Norte (45, -35; radio 12°), Pacífico Sur (-30, -120; radio 10°), región polar norte (78, 30; radio 8°). Posiciones ocupadas: Objeto A (10, 45; radio 5°) y Objeto B (-15, 110; radio 5°).
