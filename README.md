# Resistencia Comercial — Buscador de combinaciones con diagrama y código de colores

Aplicación web de un solo archivo que, dado un valor de resistencia deseado y una
tolerancia comercial (10 % o 1 %), encuentra la combinación de **1, 2 o 3 resistencias
comerciales** en **serie, paralelo o mixta** cuyo valor equivalente sea el más próximo
al solicitado. Para cada resultado muestra el **esquema de conexión**, el **código de
colores** de cada resistencia individual y el **porcentaje de precisión** del ajuste.

---

## 1. Contenido del paquete

El proyecto se compone de **dos archivos** que deben mantenerse juntos en la misma
carpeta:

| Archivo | Tipo | Descripción |
|---|---|---|
| `index.html` | Página web (HTML + CSS + JavaScript embebidos) | El programa completo. No requiere servidor, instalación ni dependencias externas. Se abre directamente en cualquier navegador moderno. |
| `icono.png` | Imagen PNG | Icono del proyecto. Se utiliza como favicon de la página y como imagen identificativa de la aplicación. |

> **Nota:** el archivo `index.html` referencia a `icono.png` mediante
> `<link rel="icon" href="icono.png">`. Si se renombra o mueve la imagen, debe
> actualizarse esa ruta en la cabecera del HTML para que el icono siga apareciendo.

---

## 2. Cómo ejecutar el programa

1. Descomprimir o copiar ambos archivos (`index.html` e `icono.png`) en la misma carpeta.
2. Hacer doble clic sobre `index.html`, o arrastrarlo a una ventana del navegador.
3. No se necesita conexión a Internet, ni Node.js, ni Python, ni ningún servidor local.

**Navegadores compatibles:** Chrome, Edge, Firefox y Safari en versiones recientes
(se usan `requestAnimationFrame`, `performance.now()` y `accent-color`, disponibles en
todos los navegadores actuales).

---

## 3. Uso de la interfaz

La pantalla principal contiene un panel de entrada con cuatro controles:

| Control | Función |
|---|---|
| **Resistencia deseada (Ω)** | Valor objetivo que se quiere obtener. Acepta decimales (ej. `1570`, `4.7`, `22000`). Debe ser mayor que cero. |
| **Tolerancia comercial** | Serie de valores normalizados a utilizar: **10 % (E12)** o **1 % (E96)**. Determina el conjunto de resistencias disponibles. |
| **Nº máximo de resistencias** | Botones de opción **1**, **2** o **3**. Limita la búsqueda a combinaciones de como máximo ese número de resistencias. |
| **Botón ⚡ Calcular** | Lanza la búsqueda. Se deshabilita mientras el cálculo está en curso. |

Al pulsar **Calcular**:

1. Aparece una **barra de progreso** con el porcentaje de avance y una estimación del
   tiempo restante (ETA). La barra se actualiza por tramos para que la página siga
   respondiendo incluso en equipos lentos.
2. Al terminar, se muestran los resultados (ver sección 4).

---

## 4. Interpretación de los resultados

### 4.1 Cabecera del mejor resultado

- **Valor equivalente** de la mejor combinación encontrada, en formato abreviado
  (Ω, kΩ, MΩ) y con su valor exacto en ohmios entre paréntesis.
- **Combinación** en notación legible, por ejemplo `1.5 kΩ + 47 Ω` o `(2.2 kΩ ∥ 10 kΩ)`.
- **Precisión**: porcentaje de exactitud del valor obtenido respecto al solicitado.
  Se calcula como `100 − |desviación %|`. Se muestra con un indicador de color:
  - verde si la precisión es ≥ 99 %,
  - ámbar si está entre 95 % y 99 %,
  - rojo si es < 95 %.
- **Desviación firmada**: indica si el valor comercial queda por encima (`+`) o por
  debajo (`−`) del valor pedido. Se conserva como dato secundario porque la dirección
  del error es útil al ajustar con un potenciómetro o al elegir la resistencia de recorte.
- **Nº de resistencias** y **tolerancia** utilizados.

### 4.2 Tabla de opciones

Se listan las **5 mejores combinaciones** ordenadas de menor a mayor desviación. Cada
fila muestra: posición, valor equivalente, combinación, número de resistencias, tipo
(serie / paralelo / híbrido) y precisión.

- **Las filas son interactivas:** al hacer clic en cualquiera de ellas, el diagrama de
  la sección inferior se actualiza para mostrar esa combinación concreta.
- La fila del **mejor resultado** aparece resaltada en azul claro.
- La fila **seleccionada** aparece con un borde azul más marcado.

### 4.3 Diagrama del resultado

Se compone de dos partes:

1. **Esquema de conexión**: dibujo SVG de la red (una, dos o tres resistencias en
   serie, paralelo o configuración mixta), con la descripción textual debajo.
2. **Resistencias individuales**: por cada resistencia de la combinación se muestra:
   - su valor,
   - un dibujo de la resistencia con sus **cuatro bandas de color** (tres de valor y
     una de tolerancia),
   - la **leyenda de colores** con el nombre de cada banda. La banda de tolerancia se
     marca en negrita con el símbolo `⟂ Tol` para distinguirla de las bandas de valor.

El diagrama sigue a la fila seleccionada en la tabla; si no se ha seleccionado ninguna,
muestra la del mejor resultado.

---

## 5. Fundamento técnico

### 5.1 Valores comerciales

- **E12 (10 %)**: 12 valores por década — `1.0, 1.2, 1.5, 1.8, 2.2, 2.7, 3.3, 3.9, 4.7, 5.6, 6.8, 8.2`.
- **E96 (1 %)**: 96 valores por década, desde `1.00` hasta `9.76`.

El programa genera el conjunto completo de valores multiplicando estas bases por
potencias de 10 dentro del rango útil.

### 5.2 Poda del espacio de búsqueda

Para evitar cálculos innecesarios, el conjunto de valores se **poda** al intervalo
`[objetivo / 4, objetivo × 4]`. Justificación: con un máximo de tres resistencias,

- la conexión en serie más grande posible es `3 × R_max`, y
- la conexión en paralelo más pequeña posible es `R_min / 3`.

Por tanto, ningún valor fuera de ese intervalo puede formar parte de una combinación
útil. Esta poda reduce el número de valores de ~600 a ~80–120 y disminuye el coste del
bucle triple en dos órdenes de magnitud, **sin pérdida de precisión** para objetivos
prácticos.

### 5.3 Combinaciones evaluadas

Para cada terna `a ≤ b ≤ c` del conjunto podado se calculan las topologías esenciales:

- **1 resistencia:** `a`
- **2 resistencias:** `a + b` (serie) y `a ∥ b` (paralelo)
- **3 resistencias:**
  - `a + b + c` (serie pura)
  - `a ∥ b ∥ c` (paralelo puro)
  - `(a ∥ b) + c` (híbrida)
  - `a + (b ∥ c)` (híbrida)
  - `(a + b) ∥ c` (híbrida)
  - `a ∥ (b + c)` (híbrida)

La restricción `a ≤ b ≤ c` elimina permutaciones redundantes. Los resultados se
deduplican por valor equivalente redondeado a cuatro decimales.

### 5.4 Cálculo por tramos y barra de progreso

El bucle de búsqueda se ejecuta en **tramos de 15 000 operaciones** dentro de
`requestAnimationFrame`. Entre tramo y tramo se cede el hilo al navegador, lo que
permite:

- que la interfaz siga respondiendo (no aparece el aviso de "página no responde"),
- que la barra de progreso se pinte y actualice suavemente,
- calcular una ETA real a partir del tiempo transcurrido y la fracción completada.

La barra se inserta en el DOM y se fuerza un repintado (`offsetHeight`) **antes** de
iniciar el cálculo, de modo que sea visible desde el primer instante.

### 5.5 Código de colores

Se utiliza el estándar de **4 bandas**:

| Banda | Significado |
|---|---|
| 1.ª | Primera cifra significativa |
| 2.ª | Segunda cifra significativa |
| 3.ª | Multiplicador (potencia de 10) |
| 4.ª | Tolerancia |

Colores de tolerancia utilizados: **plata** (±10 %), **oro** (±5 %) y **marrón** (±1 %).
El programa trabaja con las tolerancias de 10 % y 1 %; el oro queda previsto por si se
añade la serie E24 en el futuro.

---

## 6. Estructura del archivo HTML

El archivo `index.html` es autocontenido y se organiza en tres bloques:

1. **`<head>`**: metadatos, enlace al icono (`icono.png`) y hoja de estilos embebida.
2. **`<body>`**: estructura de la interfaz (panel de entrada, área de resultados y pie).
3. **`<script>`**: toda la lógica en JavaScript, organizada en funciones:
   - `buildPool(tol, target)` — genera y poda el conjunto de valores comerciales.
   - `searchChunked(...)` — búsqueda por tramos con reporte de progreso.
   - `TopK` — estructura auxiliar que conserva las 20 mejores combinaciones.
   - `bandsFor(r, tol)` — traduce una resistencia a sus cuatro colores.
   - `resistorSVG(...)` y `networkSVG(...)` — dibujo vectorial del componente y de la red.
   - `renderMain()` — construye la cabecera, la tabla y el diagrama.
   - `showProgress(...)` / `updateProgress(...)` — control de la barra de progreso.

No se utilizan librerías externas ni peticiones de red.

---

## 7. Limitaciones conocidas

- El intervalo de valores comerciales generado va de `0.01 Ω` a `10 MΩ`
  (décadas −2 a 6). Fuera de ese rango no se ofrecen resultados.
- Para valores muy pequeños (< 10 Ω) o muy grandes (> 1 MΩ) el multiplicador del código
  de colores puede quedar fuera de la escala estándar de 4 bandas; en esos casos se
  aproxima a negro como multiplicador.
- La tolerancia se aplica al conjunto de valores disponibles, no al resultado final: la
  precisión mostrada es la del **valor nominal**, no la del peor caso considerando la
  tolerancia de cada componente.
- La búsqueda se limita a un máximo de tres resistencias, tal como se selecciona en la
  interfaz.

---

## 8. Créditos y licencia

Programa desarrollado como herramienta de cálculo de resistencias equivalentes con
valores comerciales. Uso libre para fines educativos, de prototipado y de reparación
electrónica.

Los archivos que componen el proyecto son:

- `index.html` — programa.
- `icono.png` — icono.

Cualquier modificación del HTML debe conservar la referencia a `icono.png` para que el
icono se muestre correctamente.
