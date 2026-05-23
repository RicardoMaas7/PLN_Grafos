# Guion de la presentación — 15 minutos

**Distribución sugerida:** 12 slides de contenido + 1 portada + 1 cierre = 14 slides
**Ritmo:** ~60 segundos por slide. Si son 4 integrantes, cada uno toma 3-4 slides.

---

## Slide 1 — Portada (30 seg)

- Título: _Análisis de Redes Científicas con Grafos: la red de colaboración en PLN_
- Equipo, materia, profesor, fecha
- _Hablar:_ presentar al equipo, anunciar el tema y la pregunta central.

---

## Slide 2 — Pregunta y motivación (1 min)

> ¿Cómo se estructura la comunidad de investigadores que publican sobre Procesamiento de Lenguaje Natural?

- ¿Hay líderes claros?
- ¿Existen subáreas que apenas se hablan entre sí?
- ¿Quiénes son los "puentes"?

_Hablar:_ situar al público y prometer que vamos a responder esto con datos reales.

---

## Slide 3 — Datos: Semantic Scholar (1 min)

- Fuente: API oficial de Semantic Scholar (semanticscholar.org)
- 8 queries de PLN: NLP, computational linguistics, language model, MT, NER, sentiment, QA, text classification
- Endpoint: `/graph/v1/paper/search` — campos: title, year, authors
- Total recolectado: **200 papers únicos · 933 autores únicos**

_Hablar:_ enfatizar que son datos reales en vivo, no un dataset prefabricado.

---

## Slide 4 — Modelado del grafo (1 min)

```
Nodos    = autores (authorId)
Aristas  = co-autorías
Peso     = nº de papers compartidos
Distance = 1 / peso  (para Dijkstra)
```

Diagrama pequeño: 3 autores compartiendo 2 papers, mostrando el grafo resultante con peso 2 entre dos de ellos.

_Hablar:_ explicar la decisión de invertir el peso para distancias.

---

## Slide 5 — Estadísticas generales (1 min)

- 933 nodos
- 8,130 aristas
- Densidad 0.0187 (red dispersa, como todas las sociales)
- Grado promedio 17.43
- Grado máximo 137 (el paper de NLLB con muchos co-autores)

_Hablar:_ "Cada autor tiene 17 colaboradores en promedio, pero hay un autor con 137 — eso es señal de jerarquía".

---

## Slide 6 — Componentes conexas (1 min)

Gráfico de barras de tamaño de componentes:
- 1 gigante de 380
- 2 grandes (35, 28)
- 28 medianas
- 80 pequeñas
- 9 aislados

_Hablar:_ "El 40.7% de los autores forman un solo bloque conectado. El resto son grupos pequeños y satélites".

---

## Slide 7 — Caminos mínimos (1 min)

- Longitud promedio: **2.94 saltos**
- Diámetro: **7**
- Ejemplo: B. Haddow → Philipp Koehn = 1 salto

**Concepto clave: mundo pequeño** ("six degrees of separation"). En PLN son ~3 grados.

_Hablar:_ usar la analogía social, todos la entienden.

---

## Slide 8 — Centralidad (2 min, la slide más importante)

Tabla con los top 5 de cada centralidad. Resaltar:
- **M. Costa-jussà domina las 3** → líder del proyecto NLLB
- **Markus Freitag** alto en betweenness pero no en degree → puente

Mostrar `figs/grafo_top_betweenness.png` aquí.

_Hablar:_ explicar la diferencia entre las tres centralidades con un ejemplo cotidiano (el popular del salón vs. el que conecta dos grupos vs. el que está en el centro de todo).

---

## Slide 9 — Detección de comunidades (1.5 min)

- Algoritmo: **Louvain** (maximiza modularidad)
- Modularidad obtenida: **0.6319** (muy alta — umbral 0.3 = clara)
- 7 comunidades detectadas

Mostrar `figs/grafo_completo.png` con la leyenda de comunidades.

_Hablar:_ "El algoritmo agrupa automáticamente, sin conocer el tema de los papers, y aún así detecta la división NLLB vs. WMT clásico".

---

## Slide 10 — Comunidades identificadas (1.5 min)

| Comunidad | Tamaño | Líderes | Interpretación |
|---|---|---|---|
| 6 | 98 | Costa-jussà, Schwenk | **NLLB / traducción multilingüe** |
| 1 | 77 | Philipp Koehn | **MT clásica / WMT** |
| 4 | 68 | _(...)_ | _(otra subárea)_ |
| 5 | 56 | _(...)_ | _(otra subárea)_ |

_Hablar:_ "Los nombres de cada grupo no fueron asignados por nosotros; los inferimos al ver quiénes los lideran. Es lo bonito del unsupervised learning aplicado a grafos".

---

## Slide 11 — Visualización interactiva (1.5 min)

Mostrar `figs/grafo_interactivo.html` en vivo en el navegador. Hacer zoom a la comunidad NLLB. Pasar el mouse sobre Costa-jussà para mostrar el tooltip.

_Hablar:_ esta es la slide de impacto visual. Dejar que la audiencia "vea" la red.

---

## Slide 12 — Distribución de grados (1 min)

Mostrar `figs/distribucion_grados.png`. Explicar:
- Si la nube de puntos cae aproximadamente sobre una recta en log-log → red **libre de escala**
- Pocos autores con muchísimos colaboradores; muchísimos autores con pocos

_Hablar:_ "Esto es lo mismo que se observa en Twitter, en citaciones académicas, en internet. Es una propiedad fundamental de redes naturales".

---

## Slide 13 — Limitaciones (45 seg)

- Dataset acotado a las 8 queries elegidas
- 2 de 8 queries no pudieron completarse por rate limit (sin API key)
- El peso no captura el orden de autoría ni el impacto del paper

_Hablar:_ ser honestos sobre los límites refuerza la credibilidad técnica.

---

## Slide 14 — Conclusiones (1 min)

5 hallazgos en bullets cortos:
1. Mundo pequeño confirmado (2.94 saltos promedio)
2. Componente gigante densa con clustering 0.90
3. 7 comunidades con modularidad 0.63
4. Líder global: Costa-jussà (NLLB)
5. Puente clave: Freitag (conecta NLLB con MT clásica)

_Hablar:_ cerrar con la respuesta a la pregunta inicial.

---

## Slide 15 — Cierre / Q&A (45 seg)

- Repositorio GitHub: `github.com/RicardoMaas7/PLN_Grafos`
- "Gracias / preguntas"

---

## Tips para la presentación

1. **Ensayen una vez completo antes del día.** Cronometren.
2. **Pongan el grafo interactivo abierto en una pestaña antes de empezar** — no hagan esperar a la audiencia mientras carga.
3. **Hablen del proceso, no del código.** El profe ya leyó el código; en la expo va a evaluar entendimiento.
4. **Si les preguntan algo que no saben:** "buena pregunta, queda como trabajo futuro". Es académicamente válido.
