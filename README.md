# PLN_Grafos
## 5. Conclusiones

### Sobre la red de PLN

A partir de 200 papers reales descargados de Semantic Scholar reconstruimos una red de colaboración con **933 autores únicos** y **8,130 colaboraciones** distintas. Para el análisis profundo y la visualización nos enfocamos en el **núcleo de los 150 autores más conectados** dentro de la componente gigante, superando ampliamente el mínimo de 100 que pide la rúbrica.

### Hallazgos principales

1. **La red exhibe la propiedad de mundo pequeño en grado extremo.** Dentro del núcleo activo de 150 autores, la **longitud promedio de los caminos es de 1.83 saltos** y el **diámetro es de 4**. Es decir, dos investigadores cualesquiera del núcleo están separados por menos de 2 colaboraciones intermedias en promedio — un patrón clásico de redes sociales muy densas.

2. **Estructura general fuertemente fragmentada.** El grafo completo tiene **119 componentes** conexas: una gigante de **380 nodos (40.7% del total)**, dos grandes (35 y 28), 28 medianas y 80 micro-componentes. Adicionalmente **9 autores aparecen aislados**. Esto refleja la estructura típica de un campo científico maduro: un núcleo activo de colaboradores frecuentes rodeado de equipos satélite pequeños.

3. **El núcleo activo es altamente "triangulado".** El coeficiente de clustering promedio del subgrafo filtrado es **0.948**, indicando que los colaboradores de un autor del núcleo también colaboran entre sí casi siempre. Esto se explica por la naturaleza misma de la coautoría: papers con muchos autores generan triángulos completos.

4. **Estructura comunitaria clara.** El algoritmo de Louvain detectó **5 comunidades** con una **modularidad de 0.4560** (por encima del umbral 0.3 que indica estructura clara). Las dos comunidades más grandes (48 autores cada una) corresponden a:
   - **Comunidad liderada por Hady ElSahar, Julia Kreutzer y autores africanos del proyecto** _MasakhaNER / multilingual NLP_, enfocados en idiomas de bajos recursos.
   - **Comunidad liderada por Jean Maillard, Pierre Yves Andrews y Alexandre Mourachko**, asociados al ecosistema del proyecto _NLLB_ de Meta AI.

5. **Autores puente identificados.** La centralidad de intermediación reveló que **Markus Freitag** y **C. Federmann** actúan como bridges entre subcomunidades sin tener el mayor número de colaboradores individuales — son los conectores clave de la red.

6. **Hady ElSahar y M. Costa-jussà lideran todas las métricas de centralidad**, lo que los identifica como las figuras más influyentes del núcleo. Hady ElSahar sobresale especialmente en betweenness (0.426), señalando que prácticamente todos los caminos entre subgrupos pasan por ella.

### Limitaciones

- El dataset proviene únicamente de búsquedas por palabra clave en Semantic Scholar; un autor de PLN cuyos papers no contengan ninguno de los 8 términos consultados quedaría fuera.
- Solo dos de las ocho queries planeadas lograron pasar el rate limit de la API sin key. Con una API key oficial podríamos triplicar el tamaño del dataset.
- El filtrado a top-150 por degree privilegia a autores muy conectados — podría sesgar contra grupos pequeños pero relevantes.

### Trabajo futuro

- Solicitar la API key de Semantic Scholar para ampliar la recolección.
- Comparar la estructura de comunidades antes y después del filtrado para validar que los grupos detectados son robustos.
- Enriquecer las aristas con métricas adicionales: número de citas conjuntas, distancia temporal entre colaboraciones.
- Comparar la red de PLN con la de otra subárea de computación para identificar patrones estructurales distintivos.