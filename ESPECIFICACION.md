# Recorridos FC: especificación para reescribir el marcador de salones

> Documento de traspaso para que Claude (en la nube) construya la versión nueva de la app.
> Autor del proyecto: David de la Vega Bautista ("Davis"), Facultad de Ciencias, UNAM.
> Fecha: 25-sep-2026. Repositorio: https://github.com/DavidDeLaVegaUNAM/marcador-salones-fc
> Sitio publicado: https://daviddelavegaunam.github.io/marcador-salones-fc/

---

## 1. Para qué sirve

Davis escribe una tesis de matemáticas aplicadas sobre la **dificultad de moverse por la Facultad de Ciencias**. Modela cada edificio y el campus como un **grafo**:

- Los **nodos** son lugares donde uno puede estar: un salón, un punto del pasillo, una entrada, una escalera, una rampa o un elevador.
- Las **aristas** son pasos que se pueden dar entre dos nodos, con su **longitud en metros** y su desnivel.

A partir de esos metros se calcula el costo energético de cada trayecto (ecuación de Pandolf). **Hoy casi todas las longitudes son estimaciones.** La app sirve para medirlas caminando: Davis recorre la Facultad con el teléfono y la app registra el camino, los nodos y el largo de cada tramo.

La versión actual (v2) ya mide distancias con un podómetro. **Lo que falta es:**

1. **Ver el recorrido sobre el mapa oficial de la Facultad**, no sobre un lienzo en blanco.
2. **Corregir la deriva** tocando el mapa en un punto conocido.
3. **Indicar en qué piso se está.**
4. **Registrar tramos con tipo** (pasillo, andador exterior, escalera, rampa, elevador): se toca un botón al empezar el tramo, otro al terminar, y la app guarda cuánto midió.
5. **Registrar entradas y salidas de los edificios.**
6. **Exportar directamente nodos y aristas** con el formato que ya usa la tesis.

---

## 2. Estado actual del repositorio

```
marcador-salones-fc/
  index.html    app completa, un solo archivo (HTML + CSS + JS, sin dependencias)
  README.md
```

`index.html` se generó desde la tesis: una plantilla más el **catálogo de lugares de la Facultad** incrustado como JSON en `<script type="application/json" id="catalogo">`. Son 12 edificios y 498 lugares con esta forma:

```json
[{"id": 1, "nombre": "Edificio Poniente", "prefijo": "PO",
  "lugares": [{"id": 194, "nombre": "Coordinación de Servicios Editoriales", "nivel": "Sótano", "piso": -1}, ...]}, ...]
```

**Conserva ese bloque JSON tal cual** y léelo del `index.html` actual. Es la fuente oficial de nombres e ids (`lugar_id`) y no debe inventarse.

### Qué hace la v2 (funciona y Davis la usó en campo)

- **Podómetro.** Usa `devicemotion` → `accelerationIncludingGravity` para calcular la magnitud de la aceleración. Tiene dos filtros exponenciales, uno lento (0.98) y uno rápido (0.7), y cuenta un paso cuando `rápida − lenta > umbral`. Después de cada paso hay un tiempo muerto de 300 ms y un rearme cuando la diferencia vuelve a ser menor que 0. El umbral es ajustable; por omisión vale 1.2 m/s².
- **Brújula.** Toma `webkitCompassHeading` en iPhone y `deviceorientationabsolute` → `360 − alpha` en Android.
- **Navegación a estima.** En cada paso: `x += zancada·sin(rumbo)`, `y += zancada·cos(rumbo)`.
- **Calibración.** Se camina una distancia conocida y la zancada se recalcula como distancia/pasos.
- **Pantalla encendida.** Usa `navigator.wakeLock`.
- **Almacenamiento.** Guarda en `localStorage` con la llave `recorridos-salones-fc`, así que funciona sin señal.
- **Marcas.** Cada marca registra edificio, salón y nivel del catálogo, x, y, pasos, metros y los pasos y metros desde la marca anterior.
- **Exportación.** «Descargar CSV» fuera de claude.ai y «Sincronizar» dentro de claude.ai; la segunda ya no se usa.

### Lo que aprendimos en campo (25-sep, edificios O y P, 74 marcas)

- **Los largos de pasillo salieron bien:** los pasillos del O midieron 89–91 m y OpenStreetMap da 90 m.
- **El teléfono de Davis detecta aproximadamente un paso por metro.** Cuenta de menos, y la calibración dio 0.91 m/paso. La calibración se queda y debe ser fácil de repetir.
- **La deriva de posición es el problema principal.** Dentro de un piso la forma es buena, pero al cambiar de piso por una escalera, o después de dar vueltas buscando un lugar, la posición se corrió hasta 23 m. Una planta baja salió girada 20°, seguramente por interferencia metálica. **Por eso el re-anclaje tocando el mapa es obligatorio.**
- **El rumbo de la brújula es magnético.** Para alinear con el mapa hubo que girar 4.5°, que es la declinación magnética de CDMX. Debe ser un ajuste editable, con 4.5° por omisión.
- **Davis dio vueltas buscando lugares.** El camino caminado es más largo que la distancia real entre puertas, así que la app debe permitir cortar o descartar ese tramo (ver 4.5).
- **Hay lugares que no son accesibles desde el pasillo.** Por ejemplo, a la Sala del Consejo Técnico solo se entra por la Dirección, y algunas oficinas no están a la vista. Debe poder marcarse un lugar como «acceso por otro lugar».

---

## 3. Decisiones que ya tomó Davis

| Tema | Decisión |
|---|---|
| Corrección de la deriva | **Tocar el mapa.** Al pasar por un punto reconocible (una puerta, una esquina, la fuente) toca dónde está; el rastro se re-ancla ahí y el tramo desde el ancla anterior se corrige. |
| Qué capturar en escaleras | **Piso de llegada**, y **número de escalones y de descansos**. No hay barómetro en el navegador, así que el piso lo indica él. |
| Dónde vive la app | **El mismo repo, reemplazando `index.html`.** El enlace de GitHub Pages se mantiene. |
| Salida | **Tres CSV**: `nodos`, `aristas` (formato de la tesis, sección 6) y `rastro` crudo. |
| GPS | **No se usa.** Dentro de los edificios da ±20–50 m, y Davis ya lo descartó. |

---

## 4. Requisitos de la versión nueva

### 4.1 Mapa oficial de fondo

- **Imagen:** `mapa_ciencias.png`, 1417 × 752 px, RGBA. Se sube al repo junto a `index.html`. Es el mapa oficial de la Facultad; la columna de leyenda empieza en **x ≥ 1010 px** y puede recortarse u ocultarse.
- **Interacción:** zoom con pellizco, arrastre con un dedo y un botón «centrar en mí».
- **Qué se dibuja encima:** el rastro, los nodos y la posición actual.
- **El mapa SÍ está a escala.** Se georreferenció contra OpenStreetMap con 8 edificios: **1 px = 0.4250 m**, giro de −2.3°, error de 3 a 12 m por edificio. Transformación de píxel (px, py) de la imagen, con y hacia abajo, a metros locales E (este) y N (norte):

  ```
  E =  0.424628·px + (−0.017169)·py − 370.252
  N = −0.017169·px − 0.424628·py + 113.448
  ```

  El origen (0, 0) está en lat 19.3244628, lon −99.1787148. Usa la inversa para pasar de metros a píxeles.
- **Rumbo verdadero.** El rumbo de la brújula es magnético. Para convertirlo: `rumbo_verdadero = rumbo_brújula + DECLINACION`, con DECLINACION editable y 4.5° por omisión. Cada paso avanza en (E, N) con el rumbo verdadero y luego se convierte a píxeles del mapa.

### 4.2 Inicio y re-anclaje

- **Al iniciar un recorrido,** Davis toca en el mapa dónde está parado. Ese es el primer ancla.
- **Botón «Estoy aquí».** Después toca el mapa en su posición real. La app:
  1. Toma el tramo del rastro desde el ancla anterior hasta ahora.
  2. Le aplica una **similaridad** (rotación + escala uniforme alrededor del ancla anterior) para que su final caiga en el punto tocado.
  3. Reescribe las posiciones corregidas de ese tramo y de los nodos marcados en él.
  4. **Guarda el punto sin corregir y el corregido.** Los dos quedan en el rastro, con las columnas `x_crudo`/`y_crudo` y `x`/`y`.
- **Los metros de las aristas NO se reescalan con el re-anclaje.** Siguen siendo pasos × zancada, porque es la medida directa. Solo cambia la posición en el mapa. En la arista se guarda también la distancia recta en el mapa entre sus extremos, para comparar las dos.

### 4.3 Piso actual

- **Selector de piso** siempre visible: Sótano (−1), PB (0), 1, 2, 3 y 4. Cada punto del rastro y cada nodo guardan el piso.
- **Colores por piso** en el rastro, con un filtro para ver uno o todos los pisos.
- **Al terminar un tramo de escalera, rampa o elevador,** el piso de llegada que se capture cambia el piso actual automáticamente.

### 4.4 Tipos de nodo

Son los de la tesis, con los mismos colores que las figuras:

| tipo | significado | color |
|---|---|---|
| `V_S` | salón o espacio funcional (del catálogo) | `#00b0f0` |
| `V_PCI` | punto de pasillo interior (esquina, cruce, frente a una puerta) | `#a6a6a6` |
| `V_PCE` | pasillo o andador de conexión externa (entre edificios) | `#7f6bb3` |
| `V_EYS` | entrada o salida de un edificio | `#e03c31` |
| `V_ESC` | escalera (cada tramo entre dos pisos) | `#ffc000` |
| `V_ELC` | elevador (uno por piso) | `#00b050` |
| `V_RAM` | rampa | `#e377c2` |

**Formas de crear un nodo:**

- **Marcar un salón.** Es el flujo actual: se elige edificio y lugar del catálogo y se toca el botón grande. El tipo es `V_S` y el `lugar_id` es el del catálogo.
- **Marcar un punto.** Se elige el tipo (PCI, PCE, EYS) y, opcionalmente, un nombre. En `V_EYS` se pide a qué edificio pertenece la entrada y una nota, por ejemplo «caseta de vigilancia a 5 m».
- **Reusar un nodo existente.** Es **clave para que el grafo tenga ciclos.** Cuando Davis vuelve a pasar por un nodo ya marcado, por ejemplo la misma entrada, la app ofrece los nodos del mismo piso a menos de 15 m en el mapa y él elige «es este». No se crea un nodo nuevo; la arista llega al que ya existía.
- **Un salón sin acceso directo desde el pasillo** puede marcarse con la opción «se entra por…», eligiendo otro nodo. La arista se crea hacia ese nodo con tipo `acceso_interno`.

### 4.5 Tramos (lo nuevo más importante)

La lógica es: **se toca un botón al empezar, se camina y se toca otro al terminar; el tramo mide lo caminado entre los dos toques.**

- **Barra de tipos de tramo:** Pasillo · Andador exterior · Escalera · Rampa · Elevador.
- **Tramo activo por omisión:** siempre hay uno, y por omisión es «Pasillo». Cambiar de tipo **cierra el tramo actual en un nodo**: el automático es un `V_PCI`, o el que elija Davis. Ahí se abre el tramo nuevo.
- **Lo que se guarda al cerrar cualquier tramo:** pasos, metros (pasos × zancada), duración en segundos, piso de salida, piso de llegada y los nodos de inicio y fin.
- **Escalera:** al cerrar se pide **piso de llegada**, **número de escalones** y **número de descansos**. Se crea un nodo `V_ESC` por cada piso que se cruzó, con piso `"a-b"` (por ejemplo `"0-1"`), y aristas `escalera` con `n_escalones`. El desnivel queda vacío a menos que se capture.
- **Rampa:** al cerrar se pide el piso de llegada, que puede ser el mismo piso. Se crea un nodo `V_RAM` y aristas `rampa`.
- **Elevador:** no cuenta pasos; se ignoran mientras dure. Al cerrar se pide el piso de llegada. Se crean los nodos `V_ELC` de salida y de llegada (o se reusan si ya existen), con una arista `elevador_vertical` y longitud 0.
- **Andador exterior:** entre edificios. Las aristas son `pasillo_exterior` y los nodos intermedios, `V_PCE`.
- **Descartar el tramo:** botón para «di vueltas buscando». Tira los pasos desde el último nodo y deja la posición donde se tocó «Estoy aquí», o en el último nodo.

**Ejemplo real** de lo que la app debe poder capturar (edificios O y P):

> Entro por la Entrada de P (V_EYS) → pasillo 4.7 m → cruce con el corredor sur (V_PCI) → corredor 14 m → descanso de escalera (V_PCE) → escalera, 21 escalones, 3 descansos, llego al piso 1 → corredor 17 m → pasillo del O (V_PCI) → …

### 4.6 Aristas generadas

Cada tramo cerrado entre dos nodos produce una arista con este esquema (sección 6):

- `longitud_m`: metros del podómetro, con 1 decimal.
- `tipo_arista`: uno de `pasillo`, `pasillo_exterior`, `acceso_salon`, `acceso_edificio`, `acceso_interno`, `escalera`, `rampa`, `elevador_acceso` o `elevador_vertical`.
- `notas`: debe decir `PODÓMETRO <fecha>; zancada <z> m` para distinguirla de las estimaciones.

**Caso particular: nodo marcado a mitad de un tramo de pasillo.** Por ejemplo, un salón. El pasillo se parte en dos aristas y el salón cuelga del punto del pasillo con una arista `acceso_salon`. Su longitud va vacía, porque la marca se toma en la puerta.

### 4.7 Persistencia y exportación

- **Todo en `localStorage`,** con una llave nueva `recorridos-fc-v3`. Hay que migrar o leer las marcas viejas de `recorridos-salones-fc` como nodos `V_S` sueltos.
- **Varios recorridos en días distintos** deben sumar al **mismo grafo**: los nodos reusables persisten entre sesiones.
- **Botones de exportación:**
  - `nodos_<fecha>.csv`, `aristas_<fecha>.csv` y `rastro_<fecha>.csv`, en UTF-8 con BOM.
  - **Respaldo JSON** completo, para exportar e importar entre teléfonos o después de borrar el navegador.
- **Deshacer** la última acción: nodo, tramo o ancla.

### 4.8 Interfaz

- Teléfono, uso con **una mano, caminando**. Botones grandes; el mapa ocupa arriba de la mitad de la pantalla.
- Español. Modo claro y oscuro (la v2 ya tiene los tokens de color en `:root`).
- **Siempre visibles:** piso actual, tipo de tramo actual, metros del tramo, rumbo y estado de los sensores.
- **Avisos claros** cuando no hay acelerómetro o brújula. En iPhone hay que pedir permiso con `DeviceMotionEvent.requestPermission()` desde un toque.

---

## 5. Restricciones técnicas

- **Un solo `index.html` más `mapa_ciencias.png`.** Sin frameworks, sin compilación y sin servidor: es GitHub Pages estático. Solo se permiten bibliotecas de `cdn.jsdelivr.net` o `cdnjs.cloudflare.com` si de verdad hacen falta; para zoom y arrastre sobre una imagen basta con SVG o canvas y JS propio.
- **Sin comentarios en el código.** Es preferencia firme de Davis; las explicaciones van en el README o en el chat.
- **No inventar datos.** Una longitud que no se midió va **vacía**, nunca estimada.
- Todo debe funcionar **sin señal** una vez cargada la página.
- **Commits** a nombre de Davis con su correo de Ciencias, como los anteriores del repo. **Terminar con un README actualizado** que explique cómo usar la app en campo.

---

## 6. Formato de salida (debe coincidir con la tesis)

Estos CSV entran directo al guion `grafo_edificio.py` de la tesis, que dibuja el grafo estratificado.

**nodos.csv**
```
id,label,tipo,piso,lugar_id,x,y,notas
PO497,Vigilancia 1,V_S,0,497,812.4,388.1,marca de podómetro
PIPO0_09,PIPO0_09,V_PCI,0,,815.0,390.2,pasillo
ESC_S01,ESC_S01,V_ESC,0-1,,820.3,392.0,21 escalones; 3 descansos
EYS_ENTRADA,EYS_ENTRADA,V_EYS,0,,805.0,391.0,entrada sur de P
```
- `id`: para salones, `<prefijo del edificio><lugar_id>` (el prefijo viene en el catálogo). Si el nombre ya es un código de salón (`P101`, `O214`, `Y001`), se usa el nombre. Para los demás, un id legible y único.
- `piso`: un entero, o `"a-b"` en los tramos de escalera.
- `x, y`: **píxeles del mapa oficial** (corregidos por anclaje).
- `lugar_id`: vacío si el nodo no es del catálogo.

**aristas.csv**
```
origen,destino,tipo_arista,longitud_m,desnivel_m,n_escalones,notas
PIPO0_09,PCE_S0,pasillo,14.2,0,0,PODÓMETRO 2026-09-30; zancada 0.91 m
PCE_S0,ESC_S01,escalera,6.1,,11,PODÓMETRO 2026-09-30; 21 escalones; 3 descansos
```

**rastro.csv**, una fila por paso o por evento:
```
recorrido,t_iso,evento,piso,tramo_id,tipo_tramo,pasos,metros,rumbo_mag,rumbo_verdadero,x_crudo,y_crudo,x,y,nodo_id
```
Los valores de `evento` son `paso`, `nodo`, `ancla`, `inicio_tramo`, `fin_tramo` y `descarte`.

---

## 7. Criterios de aceptación

1. `node --check` sobre el JS extraído no marca errores.
2. **Prueba sintética de la transformación.** Píxel → metros → píxel devuelve el mismo punto (±0.01 px). Además, 100 pasos de 1 m con rumbo verdadero 0° avanzan ≈ 235.3 px hacia el norte del mapa (100/0.425), con la inclinación de −2.3°.
3. **Prueba sintética del re-anclaje.** Un tramo recto de 50 m girado 10° y re-anclado a su punto verdadero queda sin giro, y su `longitud_m` sigue en 50.0.
4. Exportar, borrar `localStorage`, importar el JSON y volver a exportar da CSV idénticos.
5. Los CSV exportados pasan la validación de `grafo_edificio.py`: ids únicos, tipos válidos y ninguna arista con extremos inexistentes.
6. Probado a mano en Chrome Android: iniciar, anclar, caminar un pasillo, marcar un salón, subir una escalera (21 escalones, 3 descansos, piso 1), re-anclar y exportar.

---

## 8. Preguntas abiertas (si surgen, preguntar a Davis; no suponer)

- **Tramos de exterior cuesta abajo o cuesta arriba sin escalones.** ¿Se registran como `rampa`?
- **Ampliación de la leyenda del mapa.** ¿Se muestra la lista de edificios de la leyenda como ayuda para elegir el edificio?
- **Mapas por piso.** El mapa oficial es de planta baja. Para los pisos superiores se usa el mismo mapa de fondo (el rastro cambia de color). Si Davis consigue planos por piso, deben poder cambiarse después.

---

## 9. Adenda del 29-sep-2026: modo esqueleto (recorrer los andadores de toda la Facultad)

**Estado al 29-sep:** la v3 de las secciones 1–8 **ya está construida** en `marcador-salones-fc/index.html` (1099 líneas; el README ya está al día). Pasa `node --check`. **No tiene commit ni está publicada.** La prueba en campo sigue pendiente.

**Qué quiere Davis ahora:** recorrer los andadores principales (las líneas rojas de `figures/Origen_Idea/CirculacionFC.png`) y registrar la distancia, la forma y dónde hay escaleras y rampas, con su posición en tiempo real sobre el mapa.

### Decisiones de Davis (29-sep)
| Tema | Decisión |
|---|---|
| Escaleras y rampas | **Se marcan a mano** con la barra de tramos que ya existe: escalones, descansos y piso de llegada. Sin detección automática y sin phyphox. Esto resuelve la primera pregunta de la sección 8: una cuesta sin escalones es `rampa`. |
| Posición en tiempo real | En **exteriores** el GPS sí sirve (±5 m a cielo abierto). La decisión «GPS: no se usa» de la sección 3 era para interiores y **sigue en pie** para los tramos Pasillo, Escalera y Elevador. |

### Cambios a la app (lo único que falta construir)
1. **GPS solo en el tramo «Andador».** Usar `navigator.geolocation.watchPosition` con `enableHighAccuracy`. Se prende al abrir un Andador y se apaga al cerrarlo.
   - El punto GPS se dibuja en el mapa como un círculo de precisión, convertido con la inversa de la fórmula de 4.1 (el origen es el mismo, lat 19.3244628, lon −99.1787148).
   - **No mueve el rastro solo.** Si Davis quiere corregir la posición, usa «Estoy aquí» como hasta ahora; se agrega un botón «usar GPS como ancla» cuando la precisión sea de 8 m o menos.
2. **Columnas nuevas en `aristas.csv`:** `gps_m` (largo del rastro GPS del tramo, solo con puntos de precisión ≤ 10 m; vacío si no hubo) y `gps_precision_m` (mediana). `longitud_m` sigue siendo la del podómetro. **No se estima nada.**
3. **Columnas nuevas en `rastro.csv`:** `lat, lon, precision_m` en los eventos `gps`.
4. Nada más: nodos, tramos, re-anclaje y exportación **no cambian**.

### Guion nuevo en la tesis: `simulacion/esqueleto_medido.py`
Toma `nodos_<fecha>.csv` y `aristas_<fecha>.csv` de la app y reemplaza, en `circulacion_aristas.csv`, las longitudes `ESTIMADO_OSM` por medidas de campo.
- **Paso de coordenadas:**
  1. Píxeles de `mapa_ciencias.png` → metros E, N con la fórmula de 4.1.
  2. E, N → píxeles de `CirculacionFC.png` con la inversa de la similaridad que ajusta `georreferenciar()` en `campus_circulacion.py`. Esa función hoy solo devuelve la escala y los errores: hay que hacer que también devuelva `a, c, tx, ty`. El origen es el mismo `LAT0, LON0`.
  - Ojo con la regla 5 de `.claude/CLAUDE.md`: los píxeles de un mapa no sirven en el otro sin este paso.
- **Asociación:** cada nodo de la app se asocia al nodo `Cxx` más cercano con `mas_cercano()` (tolerancia de 15 px). Si no hay ninguno, se reporta y no se inventa.
- **Aristas:** una cadena de aristas de la app entre dos `Cxx` que ya son vecinos en el esqueleto reemplaza la longitud de esa arista. `estatus = PODÓMETRO <fecha>` y la nota lleva zancada, `gps_m` y `recta_m`.
- **Escaleras y rampas:** si la cadena contiene una escalera o una rampa, la arista del esqueleto pasa a `tipo_arista = escalera` o `rampa`, con `n_escalones`. Si el tramo es largo, se parte con un nodo `V_ESC` o `V_RAM` intermedio, como en `campus_*.csv`.
- **Salida:** reescribe `circulacion_aristas.csv` e imprime la tabla «antes (ESTIMADO_OSM) y después (medido)». Después se corre `facultad_integrada.py`.

### Criterios de aceptación extra
1. Prueba sintética GPS → píxel → GPS: devuelve el mismo punto (±0.1 m).
2. Prueba sintética de `esqueleto_medido.py`: un recorrido fabricado sobre C38→C39→C40 (Entrada → P → Auditorio ABC) reemplaza exactamente esas dos aristas y no toca ninguna otra.
3. `node --check` sin errores. Después, Davis lo prueba caminando un andador de largo conocido.
4. Commit y push a nombre de Davis (su correo de Ciencias), **solo con su visto bueno**, porque el sitio público cambia.
