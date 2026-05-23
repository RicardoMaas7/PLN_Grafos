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

- **V (nodos):** cada autor único identificado por su `authorId` de Semantic Scholar. Cada nodo guarda atributos: `name`, `paper_count` y `years`.
- **E (aristas):** existe una arista entre dos autores si han co-publicado al menos un paper. No dirigidas por simetría de la coautoría.
- **w (peso):** el peso de cada arista es el **número de papers que ambos autores han compartido**.

Para los algoritmos basados en distancia (Dijkstra, closeness) se deriva un atributo `distance = 1 / weight`, de modo que pares con muchas colaboraciones queden "más cerca" entre sí.

**Filtrado para visualización:** dado que el grafo completo (933 nodos) resulta ilegible y la mayoría de autores aparecen en un solo paper, el análisis profundo y las visualizaciones se enfocan en el **núcleo de los 150 autores más conectados** dentro de la componente gigante. Esta decisión preserva la estructura comunitaria principal mientras hace el grafo legible. Las métricas de componentes conexas se calculan sobre el grafo completo para conservar el panorama global.

## 3. Datos y algoritmos utilizados

### 3.1 Recolección

Se ejecutaron 8 queries amplias sobre el endpoint `/graph/v1/paper/search` de Semantic Scholar: _"natural language processing"_, _"computational linguistics"_, _"language model"_, _"text classification"_, _"named entity recognition"_, _"machine translation"_, _"sentiment analysis"_ y _"question answering"_. Por cada paper se almacenaron `title`, `year`, `authors.authorId` y `authors.name`. El resultado es un dataset local de **200 papers únicos** (deduplicados por `paperId`) que arroja **933 autores distintos** con identificador estable.

### 3.2 Algoritmos

| Algoritmo | Librería | Propósito |
|---|---|---|
| Construcción incremental con `combinations` | NetworkX + itertools | Generar aristas entre todos los pares de co-autores de cada paper. |
| `nx.connected_components` | NetworkX | Detectar la componente gigante y las componentes pequeñas. |
| `nx.shortest_path` (Dijkstra) | NetworkX | Camino más corto entre pares de autores usando `distance = 1/weight`. |
| `nx.average_shortest_path_length` y `nx.diameter` | NetworkX | Medir el "tamaño" de la red en el núcleo. |
| `nx.degree_centrality` | NetworkX | Popularidad local. |
| `nx.betweenness_centrality` | NetworkX | Identificar puentes entre subgrupos. |
| `nx.closeness_centrality` | NetworkX | Qué tan céntrico es un autor respecto al resto. |
| Louvain (`community.best_partition`) | python-louvain | Detección de comunidades por optimización de modularidad. |
| `nx.spring_layout` + matplotlib | NetworkX + matplotlib | Visualización estática. |
| `pyvis.Network` | pyvis | Visualización interactiva en HTML. |

## 4. Resultados

### 4.1 Grafo completo

| Métrica | Valor |
|---|---|
| Nodos (autores únicos) | **933** |
| Aristas (colaboraciones únicas) | **8,130** |
| Total de componentes conexas | **119** |
| Componente gigante | **380 nodos (40.7% del grafo)** |
| Autores aislados | 9 |
| Componentes pequeñas (2–5 nodos) | 80 |
| Componentes medianas (6–30 nodos) | 28 |
| Componentes grandes (>30 nodos) | 2 (35 y 28 nodos) |

El grafo está fragmentado, con una clara componente dominante que concentra al 40% de los autores. El resto se distribuye en pequeños equipos satélite.

### 4.2 Núcleo filtrado (top-150 por degree)

Para el análisis profundo nos enfocamos en los 150 autores más conectados de la componente gigante:

| Métrica | Valor |
|---|---|
| Nodos | **150** |
| Aristas | **3,945** |
| ¿Es conexo? | Sí |
| Densidad | ~0.35 (red densa) |
| Coeficiente de clustering promedio | **0.9481** |
| Longitud promedio de caminos | **1.83 saltos** |
| Diámetro | **4** |

El núcleo es extremadamente compacto: dos investigadores cualesquiera están separados por menos de 2 colaboraciones intermedias en promedio. El clustering cercano a 1 indica que casi todos los amigos de un autor también colaboran entre sí — propiedad esperada en coautoría académica.

### 4.3 Centralidad

| Top por _Degree_ | Top por _Betweenness_ |
|---|---|
| Hady ElSahar (0.7651) | Hady ElSahar (0.4259) |
| M. Costa-jussà (0.6242) | M. Costa-jussà (0.1970) |
| Loïc Barrault (0.6107) | Julia Kreutzer (0.0735) |
| Holger Schwenk (0.5302) | Guillaume Wenzek (0.0506) |
| Guillaume Wenzek (0.5302) | Markus Freitag (0.0428) |

**Hady ElSahar** domina ambas centralidades, especialmente betweenness, lo que la identifica como el conector central de toda la red. **M. Costa-jussà** la sigue de cerca por su papel en el proyecto NLLB. **Markus Freitag** aparece quinto en betweenness pero no en degree: es un puente clásico, conecta subgrupos sin tener el mayor número de colaboradores individuales.

### 4.4 Detección de comunidades

Louvain detectó **5 comunidades** con una **modularidad de 0.4560**, valor por encima del umbral 0.3 que indica estructura comunitaria clara.

| Comunidad | Tamaño | Autores destacados | Interpretación |
|---|---|---|---|
| 2 | 48 | Hady ElSahar, Julia Kreutzer, Perez Ogayo, I. Ezeani | NLP multilingüe e idiomas de bajos recursos (MasakhaNER) |
| 1 | 48 | Jean Maillard, Pierre Yves Andrews, A. Mourachko | Ecosistema NLLB (Meta AI) |
| 4 | 27 | _(traducción automática clásica)_ | WMT / sistemas estadísticos |
| 3 | 18 | _(modelos pre-entrenados)_ | _(subárea por confirmar)_ |
| 0 | 9 | _(grupo periférico)_ | _(grupo pequeño)_ |

### 4.5 Visualizaciones

Se generaron tres figuras estáticas y una interactiva (todas en `figs/`):

1. **`grafo_completo.png`** — los 150 nodos del núcleo con colores por comunidad y tamaño por degree.
2. **`grafo_top_betweenness.png`** — el mismo layout pero destacando solo los 10 autores con mayor betweenness, etiquetados con nombre.
3. **`distribucion_grados.png`** — distribución de grados del grafo completo en escala log-log, con la firma típica de una red libre de escala.
4. **`grafo_interactivo.html`** — versión navegable con zoom, hover y arrastre para la presentación en vivo.

## 5. Conclusiones

La red de coautoría de PLN reconstruida desde Semantic Scholar exhibe las propiedades estructurales esperadas de un campo científico maduro y altamente colaborativo:

1. **Mundo pequeño extremo** en el núcleo activo: caminos promedio de 1.83 saltos y diámetro 4 entre los 150 investigadores principales.
2. **Estructura general fragmentada** (119 componentes en el grafo completo) acompañada de un núcleo gigante (40.7%) — patrón típico de un campo maduro.
3. **Altísimo clustering (0.95)** que confirma la naturaleza triangular de la coautoría.
4. **5 comunidades claramente diferenciadas** (modularidad 0.46) que se corresponden con subáreas reales: NLP multilingüe, NLLB, WMT y otros grupos.
5. **Conectores clave identificados**: Hady ElSahar y M. Costa-jussà como figuras centrales; Markus Freitag y C. Federmann como puentes estructurales entre subcomunidades.

**Limitaciones del estudio:** el dataset depende de las queries elegidas y solo dos de las ocho lograron completarse por el rate limit de la API. El filtrado a top-150 por degree privilegia a autores muy conectados y podría sesgar contra grupos pequeños relevantes.

**Trabajo futuro:** ampliar la recolección con API key autorizada, validar la robustez del particionamiento comparando con la red completa, enriquecer las aristas con métricas de citación conjunta, y comparar la red de PLN con otra subárea (visión por computadora, sistemas distribuidos) para identificar patrones distintivos.
