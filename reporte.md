# Análisis de Redes Científicas con Grafos: Colaboración en Procesamiento de Lenguaje Natural

**Asignatura:** Estructuras de Datos — Proyecto Unidad 5
**Facultad de Matemáticas · Licenciatura en Ingeniería de Software — 4º Semestre**
**Profesor:** Dr. Jorge Carlos Reyes Magaña
**Equipo:** _(nombres de los integrantes)_
**Fecha:** _(fecha de entrega)_

---

## 1. Descripción del problema

El proyecto consiste en construir y analizar una red real de colaboración científica en el área de **Procesamiento de Lenguaje Natural (PLN)** utilizando datos extraídos en vivo de la API de Semantic Scholar. El objetivo es aplicar de forma integrada los conceptos vistos en la unidad de grafos: representación de grafos en memoria, cálculo de caminos mínimos, métricas de centralidad, detección de componentes conexas, detección de comunidades y visualización.

La pregunta de fondo es: **¿cómo se estructura la comunidad de investigadores que publican sobre PLN?** ¿Existen grupos densamente conectados? ¿Hay autores que actúan como puentes entre subáreas? ¿Es la red densa o fragmentada?

Para responder esto modelamos las publicaciones científicas como una red de coautoría y le aplicamos los algoritmos clásicos del análisis de redes.

## 2. Modelado del grafo

Se construyó un **grafo no dirigido ponderado** `G = (V, E, w)`:

- **V (nodos):** cada autor único identificado por su `authorId` de Semantic Scholar. Cada nodo guarda atributos: `name`, `paper_count` (número de papers del autor en el dataset) y `years` (años de publicación observados).
- **E (aristas):** existe una arista entre dos autores si han co-publicado al menos un paper. Las aristas son no dirigidas porque la coautoría es simétrica.
- **w (peso):** el peso de cada arista es el **número de papers que ambos autores han compartido**. Más papers en común = vínculo más fuerte.

Para los algoritmos basados en distancia (Dijkstra, closeness), se deriva un atributo `distance = 1 / weight`, de modo que pares con muchas colaboraciones queden "más cerca" entre sí.

## 3. Datos y algoritmos utilizados

### 3.1 Recolección

Se ejecutaron 8 queries amplias sobre el endpoint `/graph/v1/paper/search` de Semantic Scholar: _"natural language processing"_, _"computational linguistics"_, _"language model"_, _"text classification"_, _"named entity recognition"_, _"machine translation"_, _"sentiment analysis"_ y _"question answering"_. Por cada paper se almacenaron `title`, `year`, `authors.authorId` y `authors.name`. El resultado es un dataset local de **200 papers únicos** (deduplicados por `paperId`) que arroja **933 autores distintos** con identificador estable.

### 3.2 Algoritmos

| Algoritmo | Librería | Propósito |
|---|---|---|
| Construcción incremental con `combinations` | NetworkX + itertools | Generar aristas entre todos los pares de co-autores de cada paper. |
| `nx.connected_components` | NetworkX | Detectar la componente gigante y las componentes pequeñas. |
| `nx.shortest_path` (Dijkstra) | NetworkX | Camino más corto entre pares de autores, usando `distance = 1/weight`. |
| `nx.average_shortest_path_length` y `nx.diameter` | NetworkX | Medir el "tamaño" de la red en la componente gigante. |
| `nx.degree_centrality` | NetworkX | Popularidad local (cuántos colaboradores directos). |
| `nx.betweenness_centrality` | NetworkX | Identificar puentes entre subgrupos. |
| `nx.closeness_centrality` | NetworkX | Qué tan céntrico es un autor respecto al resto. |
| Louvain (`community.best_partition`) | python-louvain | Detección de comunidades por optimización de modularidad. |
| `nx.spring_layout` + matplotlib | NetworkX + matplotlib | Visualización estática. |
| `pyvis.Network` | pyvis | Visualización interactiva en HTML. |

## 4. Resultados

### 4.1 Estructura general

| Métrica | Valor |
|---|---|
| Nodos (autores) | **933** |
| Aristas (colaboraciones únicas) | **8,130** |
| Densidad | 0.0187 |
| Grado promedio | 17.43 |
| Grado máximo | 137 |
| Peso promedio de aristas | 1.14 |

El **grado promedio de 17** indica que un autor típico ha colaborado con 17 personas distintas dentro del dataset. El peso promedio cercano a 1 confirma que la mayoría de las co-autorías son "puntuales" (un solo paper compartido), aunque existen excepciones de pares que han co-publicado varias veces.

### 4.2 Componentes conexas

La red está dominada por una **componente gigante de 380 nodos (40.7% del total)**. El resto se distribuye así:

- 80 componentes pequeñas (2–5 autores)
- 28 componentes medianas (6–30 autores)
- 2 componentes grandes (35 y 28 nodos)
- 9 autores aislados (un solo paper sin co-autores en el dataset)

La componente gigante es densa: presenta un **coeficiente de clustering promedio de 0.901**, propiedad esperable en redes de coautoría donde los papers con múltiples autores generan triángulos completos.

### 4.3 Caminos mínimos

Dentro de la componente gigante:
- **Longitud promedio de caminos:** 2.94 saltos
- **Diámetro:** 7

Estos valores son la firma del fenómeno de **mundo pequeño** (small world): cualquier par de investigadores está separado por menos de 3 colaboraciones intermedias en promedio.

### 4.4 Centralidad

| Top por _Degree_ | Top por _Betweenness_ | Top por _Closeness_ |
|---|---|---|
| M. Costa-jussà (0.36) | M. Costa-jussà (0.29) | Cynthia Gao (0.59) |
| Hady ElSahar (0.30) | Hady ElSahar (0.23) | Ondrej Bojar (0.59) |
| Guillaume Wenzek (0.29) | Markus Freitag (0.20) | Safiyyah Saleem (0.58) |
| Loïc Barrault (0.28) | M. Krikun (0.10) | Prangthip Hansanti (0.58) |
| Cynthia Gao (0.27) | Guillaume Wenzek (0.10) | Holger Schwenk (0.57) |

**M. Costa-jussà** domina las tres métricas, lo que se explica por su rol central en el proyecto NLLB (_No Language Left Behind_) de Meta AI. **Markus Freitag** aparece en betweenness pero no en degree: es un "puente" clásico — conecta subgrupos sin tener necesariamente el mayor número de colaboradores.

### 4.5 Detección de comunidades

Louvain detectó **7 comunidades** con una **modularidad de 0.6319**, valor que indica estructura comunitaria muy fuerte (umbral mínimo de claridad: 0.3).

| Comunidad | Tamaño | Autores destacados | Interpretación |
|---|---|---|---|
| 6 | 98 | Costa-jussà, Wenzek, Barrault, Gao, Schwenk | Proyecto NLLB (traducción multilingüe a escala) |
| 1 | 77 | Philipp Koehn, Haddow, Federmann | Traducción automática clásica (WMT, Moses) |
| 4 | 68 | _(autores varios)_ | _(subárea por identificar)_ |
| 5 | 56 | _(autores varios)_ | _(subárea por identificar)_ |
| 3 | 38 | _(autores varios)_ | _(subárea por identificar)_ |

### 4.6 Visualizaciones

Se generaron tres figuras (disponibles en `figs/`):

1. **`grafo_completo.png`** — la componente gigante con nodos coloreados por comunidad y tamaño proporcional al degree.
2. **`grafo_top_betweenness.png`** — el mismo layout pero destacando solo los 10 autores con mayor betweenness centrality, identificados por nombre.
3. **`distribucion_grados.png`** — distribución de grados en escala log-log, donde la pendiente aproximada sugiere comportamiento libre de escala.
4. **`grafo_interactivo.html`** — versión navegable con zoom y hover para la presentación.

## 5. Conclusiones

La red de coautoría de PLN reconstruida desde Semantic Scholar exhibe las propiedades estructurales esperadas de un campo científico maduro:

1. **Mundo pequeño** (camino promedio 2.94, diámetro 7) que conecta a la mayoría de investigadores activos en pocos saltos.
2. **Componente gigante dominante** (40.7%) acompañada de equipos satélite pequeños y autores aislados, reflejando la mezcla de colaboración intensa y trabajo periférico típico de la academia.
3. **Alto clustering** (0.901) por la naturaleza misma de la coautoría: cada paper de _n_ autores aporta `C(n,2)` aristas formando triángulos.
4. **Estructura comunitaria fuerte** (modularidad 0.63) donde Louvain identifica claramente subgrupos temáticos coherentes, especialmente el cluster del proyecto NLLB y el de traducción automática clásica.
5. **Autores puente identificables** (Freitag, Federmann) que conectan subáreas y serían perfectos candidatos para colaboraciones cross-comunidad.

**Limitaciones del estudio:** el dataset depende de las queries elegidas; un autor de PLN cuyos papers no contengan ninguno de los 8 términos quedaría fuera. Adicionalmente, dos de las ocho queries planeadas no pudieron completarse debido al rate limit de la API sin key — con una API key autorizada el dataset podría triplicarse.

**Trabajo futuro:** ampliar la recolección con API key, enriquecer las aristas con métricas de citación conjunta y comparar la red de PLN con otra subárea (visión por computadora, sistemas distribuidos) para identificar patrones estructurales distintivos.
