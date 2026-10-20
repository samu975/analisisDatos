# Evidencia de Aprendizaje 1 — Analizando datos de comercio electrónico con Web Scraping

---

**Estudiantes:**
- Rosero Carrera, Samuel
- Rico, Brian
**Institución:** *(completar)*
**Curso:** *(completar)*
**Docente:** *(completar)*
**Fecha de entrega:** *(completar)*

---

## 1. Introducción

El comercio electrónico genera grandes volúmenes de información pública sobre productos, precios y disponibilidad. Acceder a estos datos de forma estructurada permite realizar análisis de mercado, comparaciones de precios y estudios de competitividad. Sin embargo, esta información suele estar dispersa en páginas web diseñadas para consumo humano, no para procesamiento automatizado.

El **web scraping** es un conjunto de técnicas que permiten extraer datos de páginas web de forma programática. En Python existen múltiples herramientas para este fin, cada una con enfoques, ventajas y limitaciones distintas. Este trabajo aplica tres de las más utilizadas —BeautifulSoup, Selenium y Scrapy— sobre un mismo producto de comercio electrónico, con el objetivo de comparar su efectividad y comprender sus diferencias.

## 2. Descripción de Buscalibre y del producto seleccionado

### Buscalibre

Buscalibre (https://www.buscalibre.com.co) es una plataforma de comercio electrónico especializada en la venta de libros en América Latina. Opera en varios países (Chile, Colombia, México, Perú, Argentina) y ofrece un catálogo amplio de libros nacionales e internacionales, con información detallada de cada producto: título, autor, editorial, ISBN, precio, disponibilidad y opiniones de usuarios.

### Producto seleccionado

**Título:** *Cien años de soledad* (Edición conmemorativa de la RAE y la ASALE)
**Autor:** Gabriel García Márquez
**ISBN-13:** 9789585118010
**URL:** https://www.buscalibre.com.co/libro-cien-anos-de-soledad-edicion-conmemorativa-de-la-rae-y-la-asale/9789585118010/p/52342100

Se seleccionó este producto por ser una obra literaria de referencia del autor colombiano más reconocido internacionalmente, lo que garantiza que la ficha del producto contenga información completa y sea de interés académico.

> **Nota:** Los precios, disponibilidad y opiniones pueden variar según la fecha de consulta. Los datos presentados corresponden al momento de ejecución del notebook.

## 3. Planteamiento del problema y tema de interés

### Problema

La información de productos en sitios de comercio electrónico está diseñada para ser consultada manualmente por usuarios. Cuando se requiere extraer datos de forma estructurada —por ejemplo, para comparar precios, analizar disponibilidad o alimentar bases de datos—, es necesario emplear técnicas de web scraping.

Sin embargo, no todas las herramientas de scraping son iguales. Algunas trabajan sobre el HTML estático que devuelve el servidor, mientras que otras ejecutan JavaScript para acceder a contenido dinámico. Cada enfoque tiene implicaciones en velocidad, completitud de los datos, complejidad de implementación y compatibilidad con distintos entornos.

### Tema de interés

El tema central es la **extracción y comparación de información pública de un producto** utilizando tres métodos de web scraping en Python. Se busca determinar cuál método logra extraer más campos, cuáles coinciden en sus resultados y qué limitaciones presenta cada uno.

### Tres métodos de scraping

1. **BeautifulSoup:** Biblioteca para parsear HTML/XML. Trabaja sobre el HTML estático obtenido con `requests`. No ejecuta JavaScript. Es simple, ligera y adecuada para sitios con contenido estático.

2. **Selenium:** Herramienta de automatización de navegadores. Controla un navegador real (Chrome, Firefox) de forma programática, ejecutando JavaScript y esperando cargas dinámicas. Es más pesada pero accede a todo el contenido visible para un usuario.

3. **Scrapy:** Framework completo de web scraping y crawling. Gestiona solicitudes HTTP, concurrencia, respeto a `robots.txt` y pipelines de datos. Trabaja sobre HTML estático pero está diseñado para proyectos a escala.

## 4. Objetivo general y objetivos específicos

### Objetivo general

Aplicar la metodología de web scraping para extraer y comparar información pública del producto *Cien años de soledad* en Buscalibre Colombia, utilizando tres herramientas de Python (BeautifulSoup, Selenium y Scrapy), y evaluar sus diferencias en alcance, facilidad de uso y completitud de los datos obtenidos.

### Objetivos específicos

1. Extraer campos públicos del producto (título, autor, ISBN, precio, disponibilidad, entre otros) empleando cada uno de los tres métodos.
2. Comparar los resultados obtenidos por cada método e identificar diferencias, ventajas y limitaciones.
3. Definir y calcular métricas de éxito que permitan evaluar la efectividad de cada herramienta.
4. Verificar el cumplimiento de las condiciones de acceso del sitio mediante la revisión de `robots.txt`.

## 5. Metodología

### 5.1 Campos objetivo

Se definieron 10 campos a extraer de la ficha del producto:

| # | Campo | Descripción |
|---|-------|-------------|
| 1 | Título | Nombre completo del libro |
| 2 | Autor | Autor(es) del libro |
| 3 | ISBN | Identificador ISBN-13 |
| 4 | Precio actual | Precio de venta en COP |
| 5 | Precio original | Precio sin descuento |
| 6 | Descuento | Porcentaje de descuento |
| 7 | Disponibilidad | Estado de stock |
| 8 | Editorial | Casa editorial |
| 9 | N.º de páginas | Cantidad de páginas |
| 10 | Idioma | Idioma del libro |

### 5.2 Verificación de robots.txt

Antes de realizar cualquier solicitud, se revisó el archivo `robots.txt` de Buscalibre Colombia (https://www.buscalibre.com.co/robots.txt). Hallazgos relevantes:

- Las fichas de producto (`/libro-...`) **no están bloqueadas**.
- Las rutas bloqueadas son: carrito (`/carro/`), pagos (`/pagar/`), área de usuario (`/u/`) y búsquedas con parámetros.
- No se define un `Crawl-delay`.
- Bots como `ClaudeBot`, `ChatGPT-User` y `Google-Extended` están explícitamente permitidos.

Se realizó **una única solicitud por método** a una sola página de producto, lo que constituye un uso mínimo y respetuoso del sitio.

### 5.3 Método 1 — BeautifulSoup

Se realizó una solicitud HTTP GET con la biblioteca `requests` y se parseó el HTML con BeautifulSoup y el parser `lxml`. La extracción se realizó mediante:

- Meta-etiquetas Open Graph (`og:title`, `og:description`).
- Datos estructurados JSON-LD (`<script type="application/ld+json">`).
- Selectores CSS (`.nombre`, `.precioAhora`, `.precioAntes`, `.descuento`, `.stock`).
- Tablas de ficha técnica.
- ISBN extraído de la URL como respaldo.

### 5.4 Método 2 — Selenium

Se utilizó Selenium con Chrome en modo *headless* (sin interfaz gráfica). El código detecta automáticamente si se ejecuta en Google Colab (donde instala Chromium) o en un entorno local. Selenium espera a que el contenido dinámico cargue (con `WebDriverWait`) y luego extrae los campos usando los mismos selectores CSS, además de acceder a la ficha técnica cargada por JavaScript y a los datos JSON-LD del `page_source`.

### 5.5 Método 3 — Scrapy

Se implementó un spider de Scrapy que solicita la ficha del producto y extrae los campos usando selectores CSS de Scrapy. Para evitar conflictos con el event loop de Jupyter, el spider se escribe en un archivo temporal y se ejecuta como subproceso. Los resultados se guardan en un archivo JSON que el notebook lee y procesa. Scrapy respeta `robots.txt` por defecto (`ROBOTSTXT_OBEY = True`).

## 6. Métricas de éxito

Se definieron tres métricas para evaluar objetivamente cada método:

### 6.1 Completitud

**Definición:** Proporción de campos extraídos (valor no nulo) sobre el total de campos objetivo.

**Fórmula:** `Completitud = Campos obtenidos / 10`

Se calcula por método. Un valor de 100 % indica que todos los campos fueron extraídos.

### 6.2 Concordancia

**Definición:** Proporción de campos donde al menos dos métodos obtienen el mismo valor no nulo.

**Fórmula:** `Concordancia = Campos concordantes / 10`

Mide la consistencia entre los métodos. Un valor alto indica que los datos extraídos son confiables y reproducibles.

### 6.3 Ejecución sin errores

**Definición:** Si el método se ejecutó completamente sin excepciones no controladas.

**Valores posibles:** Sí / No.

### 6.4 Resultados de las métricas

| Métrica | BeautifulSoup | Selenium | Scrapy |
|---------|:---:|:---:|:---:|
| Completitud | 9/10 = 90 % | 8/10 = 80 % | 9/10 = 90 % |
| Ejecución sin errores | Sí | Sí | Sí |

**Concordancia general entre métodos:** 9/10 = 90 %.

Nueve de los diez campos fueron extraídos con el mismo valor por al menos dos métodos. El único campo que ningún método obtuvo fue *editorial*, ya que la ficha del producto en Buscalibre no incluye ese dato. Selenium no capturó *disponibilidad* debido a que el elemento no era visible en el viewport del navegador headless al momento de la consulta.

## 7. Resultados y comparación

### 7.1 Tabla comparativa

Datos obtenidos en la ejecución del 19 de octubre de 2026.

| Campo | BeautifulSoup | Selenium | Scrapy |
|-------|--------------|----------|--------|
| Título | Cien años de soledad (Edición conmemorativa de la RAE y la ASALE) | Cien años de soledad (Edición conmemorativa de la RAE y la ASALE) | Cien años de soledad (Edición conmemorativa de la RAE y la ASALE) |
| Autor | Gabriel García Márquez | Gabriel García Márquez | Gabriel García Márquez |
| ISBN | 9789585118010 | 9789585118010 | 9789585118010 |
| Precio actual | $ 64.350 | $ 64.350 | $ 64.350 |
| Precio original | $ 99.000 | $ 99.000 | $ 99.000 |
| Descuento | -35 % | -35 % | -35 % |
| Disponibilidad | Quedan 18 unidades | *(no extraído)* | Quedan 18 unidades |
| Editorial | *(no disponible en la ficha)* | *(no disponible en la ficha)* | *(no disponible en la ficha)* |
| N.º de páginas | 614 | 614 | 614 |
| Idioma | Español | Español | Español |

### 7.2 Análisis comparativo

| Aspecto | BeautifulSoup | Selenium | Scrapy |
|---------|--------------|----------|--------|
| Tipo de acceso | HTML estático | Navegador completo (JS) | HTML estático |
| Velocidad | Muy rápida | Lenta | Rápida |
| Dependencias | Ligeras | Pesadas (Chrome) | Moderadas (Twisted) |
| Contenido dinámico | No | Sí | No |
| Escalabilidad | Manual | Manual | Alta |
| Uso en Colab | Sin problemas | Requiere instalar Chromium | Posibles conflictos con event loop |
| Respeto a robots.txt | Manual | Manual | Automático |

### 7.3 Diferencias y causas de discrepancia

- **BeautifulSoup y Scrapy** obtuvieron resultados idénticos (9/10 campos), lo cual es coherente porque ambos trabajan sobre el mismo HTML estático devuelto por el servidor.
- **Selenium** obtuvo 8/10 campos. No logró capturar la *disponibilidad* (elemento `p.stock`), probablemente porque el navegador headless no desplegó ese elemento en el viewport al momento de la consulta. Sin embargo, fue el único método que accedió al DOM tras la ejecución de JavaScript, lo que le permitiría extraer contenido dinámico en otros escenarios.
- El campo **editorial** no fue extraído por ningún método porque la ficha del producto en Buscalibre no incluye una fila de "Editorial" o "Sello" en su estructura HTML.
- Los campos comunes (título, autor, ISBN, precios, descuento, páginas, idioma) coincidieron exactamente entre los tres métodos, lo que confirma la fiabilidad de la extracción.

### 7.4 Limitaciones

- Los resultados dependen del estado del sitio al momento de la ejecución.
- Si el sitio implementa protecciones anti-bot, uno o más métodos podrían no funcionar.
- Selenium requiere un navegador instalado, lo que limita su portabilidad.
- Scrapy puede presentar conflictos con el event loop asíncrono de los notebooks.
- Solo se analizó un producto; los resultados no son generalizables a todo el sitio.

## 8. Conclusiones

1. **BeautifulSoup y Scrapy lograron la mayor completitud (90 %).** Ambos extrajeron 9 de 10 campos a partir del HTML estático, demostrando que gran parte de la información de producto en Buscalibre está disponible sin necesidad de ejecutar JavaScript.

2. **Selenium obtuvo 80 % de completitud pero aporta ventajas en otros escenarios.** Aunque en este caso no capturó la disponibilidad, Selenium es el único método capaz de acceder a contenido cargado dinámicamente, lo que lo hace indispensable en sitios que dependen fuertemente de JavaScript.

3. **La concordancia del 90 % confirma la fiabilidad de los datos.** Nueve de diez campos fueron extraídos con valores idénticos por al menos dos métodos, lo que valida que la información obtenida es consistente y correcta.

4. **El respeto a las condiciones de acceso es fundamental.** Se verificó `robots.txt` antes de realizar solicitudes, se usó una única solicitud por método y no se intentó evadir ningún mecanismo de protección. Las fichas de producto están explícitamente permitidas.

5. **Los tres métodos son complementarios.** En un proyecto real, BeautifulSoup es ideal para prototipos rápidos, Selenium para sitios dinámicos y Scrapy para recolección a escala con gestión automática de `robots.txt` y concurrencia.

## 9. Bibliografía

- Buscalibre Colombia. (s.f.). *Cien años de soledad (Edición conmemorativa de la RAE y la ASALE)*. Recuperado de https://www.buscalibre.com.co/libro-cien-anos-de-soledad-edicion-conmemorativa-de-la-rae-y-la-asale/9789585118010/p/52342100

- Buscalibre Colombia. (s.f.). *robots.txt*. Recuperado de https://www.buscalibre.com.co/robots.txt

- Richardson, L. (2023). *Beautiful Soup Documentation*. Recuperado de https://www.crummy.com/software/BeautifulSoup/bs4/doc/

- Selenium Project. (2024). *Selenium with Python*. Recuperado de https://selenium-python.readthedocs.io/

- Scrapy Project. (2024). *Scrapy Documentation*. Recuperado de https://docs.scrapy.org/en/latest/

- Python Software Foundation. (2024). *Requests: HTTP for Humans*. Recuperado de https://docs.python-requests.org/en/latest/

- Mitchell, R. (2018). *Web Scraping with Python: Collecting More Data from the Modern Web* (2.ª ed.). O'Reilly Media.
