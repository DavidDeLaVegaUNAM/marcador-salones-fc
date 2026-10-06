# Recorridos FC · Facultad de Ciencias, UNAM

App de campo para medir, caminando, el grafo de movilidad de la Facultad: los **nodos** (salones, laboratorios, puntos de pasillo, entradas, escaleras, rampas y elevadores) y las **aristas** entre ellos, con su longitud en metros. El acelerómetro del teléfono cuenta los pasos, la brújula da el rumbo y el recorrido se dibuja sobre el mapa oficial de la Facultad.

Forma parte de la tesis *Teoría de Redes para el Diseño y Análisis de los Horarios de la Facultad de Ciencias* (David de la Vega Bautista).

Sitio: https://daviddelavegaunam.github.io/marcador-salones-fc/

## Antes de salir

1. Abre la página en el teléfono (Chrome en Android, Safari en iPhone) **con señal**. Una vez cargada funciona sin conexión.
2. En **Ajustes y calibración** revisa el largo de paso. Para recalibrarlo toca **Calibrar**, camina una distancia conocida (por ejemplo 10 m) y toca **Terminar calibración**.
3. La **declinación** (4.5° en CDMX) convierte el rumbo magnético de la brújula en rumbo verdadero. No la cambies salvo que el rastro salga girado de forma sistemática.

## En campo

**Empezar.** Toca **Iniciar** y luego toca en el mapa el punto donde estás parado. La app pregunta qué hay ahí: una entrada, un punto de pasillo o un nodo que ya existe. Ese primer toque es el primer ancla.

**Caminar.** Lleva el teléfono al frente, como si leyeras. Arriba siempre ves el piso, el tipo de tramo con sus metros, el rumbo y el estado de los sensores.

**Tramos.** Siempre hay un tramo abierto; por omisión es *Pasillo*. Cambia el tipo con la barra (Pasillo · Andador · Escalera · Rampa · Elevador) al empezar cada parte del camino. Cada cambio cierra el tramo anterior en un nodo, y el tramo mide lo caminado entre los dos toques.

- **Escalera:** al terminar toca *Pasillo* (o el tipo que siga). La app pide el piso de llegada, los escalones y los descansos, y cambia el piso actual sola.
- **Rampa:** igual que la escalera, pero sin escalones. El piso de llegada puede ser el mismo. Los andadores exteriores inclinados sin escalones también se registran como rampa.
- **Elevador:** no cuenta pasos. Al salir indica el piso de llegada.
- **Andador:** para caminar por fuera, entre edificios. Es el único tramo que usa **GPS** (ver abajo).

**GPS en el Andador.** Al abrir un tramo *Andador* la app pide la ubicación y la enciende; al cerrarlo, la apaga. En pasillos, escaleras y elevadores no se usa, porque bajo techo da errores de 20 a 50 m.
- En el mapa aparece un círculo con la precisión del GPS: verde si es de ±8 m o menos, naranja si es peor. Arriba, un chip dice `GPS ±n m`.
- **El GPS no mueve tu rastro solo.** Cuando el círculo está verde aparece el botón **Usar GPS como ancla**: funciona igual que *Estoy aquí*, pero con la posición del GPS.
- `longitud_m` sigue siendo la del podómetro. Al cerrar el andador, la arista guarda aparte `gps_m`, el largo del rastro GPS, para comparar.
- Para calcular `gps_m` solo se usan puntos con precisión de ±10 m o menos. Entre puntos se exige una separación de al menos el doble de su precisión, para que el temblor del GPS no infle la suma. En una curva cerrada esto puede quedarse un poco corto.

**Marcar un lugar.** Elige el edificio (con el número del mapa) y el lugar, y toca **Marcar lugar** en la puerta. El pasillo se parte en ese punto y el lugar cuelga de él. El catálogo incluye los laboratorios de los edificios A y B de Biología. Si al lugar solo se entra por otro (por ejemplo, la Sala del Consejo por la Dirección), activa *Se entra por otro lugar* y elige el nodo.

**Punto aquí.** Sirve para marcar una esquina, un cruce, una entrada o un descanso. Si ya pasaste por ese punto, elígelo de la lista *¿Es uno de estos?* (nodos del mismo piso a menos de 15 m). Así el grafo cierra ciclos y tu posición se corrige a la del nodo.

**Estoy aquí.** Cuando pases por un punto reconocible (una puerta, una esquina, la fuente), toca **Estoy aquí** y luego tu posición real en el mapa. El tramo desde el ancla anterior se gira y escala para caer ahí. **Los metros medidos no cambian:** solo cambia el dibujo. Hazlo al cambiar de piso y después de dar vueltas.

**Red del campus y avance.** Sobre el mapa se dibujan los andadores de la red de circulación de la Facultad (las líneas rojas de la tesis): 97 tramos, unos 2.1 km. Cada tramo cambia de color según lo que ya caminaste, sumando todos los días:
- **rojo punteado:** falta;
- **naranja:** a medias;
- **verde:** recorrido (al menos 80 % de su largo con tu rastro a 6 m o menos).

Debajo de los pisos, un contador dice cuántos andadores llevas. Con zoom aparecen los nombres de los cruces (`C12`, `C39`…) para ubicar qué tramo falta. El botón **Red** la muestra u oculta. El color solo sirve para orientarse; la medida que entra a la tesis la decide `esqueleto_medido.py`. Para actualizar la red después de cambiar el esqueleto: `python esqueleto_medido.py --red-app`.

**Descartar tramo.** Si diste vueltas buscando un lugar, toca **Descartar tramo**. Se tiran los pasos desde el último nodo y la posición regresa a él. Camina de vuelta a ese nodo y sigue desde ahí.

**Deshacer.** Revierte la última acción (nodo, tramo, ancla, descarte o piso). Los pasos que diste después se conservan.

## Al terminar

En **Datos**:

- `nodos_<fecha>.csv`, `aristas_<fecha>.csv` y `rastro_<fecha>.csv` (UTF-8 con BOM). Los dos primeros entran directo a `grafo_edificio.py`.
- **Respaldo JSON**: el grafo completo. Descárgalo cada día; con **Importar respaldo** se recupera en otro teléfono o después de borrar el navegador.

Los recorridos de días distintos se suman al mismo grafo. Todo se guarda en el teléfono y no se envía a ningún servidor.

## Formato de salida

**nodos.csv** `id,label,tipo,piso,lugar_id,x,y,notas`
- `x, y` están en píxeles del mapa oficial (`mapa_ciencias.png`, 1417 × 752), ya corregidos por los anclajes.
- `piso` es un entero, o `a-b` en los nodos de escalera.
- `lugar_id` es el id del catálogo de la Facultad; va vacío en los laboratorios agregados y en los nodos que no son lugares.

**aristas.csv** `origen,destino,tipo_arista,longitud_m,desnivel_m,n_escalones,notas,recta_m,gps_m,gps_precision_m`
- `longitud_m` es la medida del podómetro (pasos × zancada) y va vacía cuando no se midió. Nunca se estima.
- `recta_m` es la distancia en línea recta en el mapa entre los extremos, para compararla con lo caminado.
- En una escalera o rampa, la longitud medida se reparte en partes iguales entre sus aristas, y la nota lo dice.
- `gps_m` y `gps_precision_m` (la mediana de la precisión) solo se llenan en los andadores con al menos dos puntos GPS de ±10 m o menos. Si no los hubo, van vacíos.

**rastro.csv** `recorrido,t_iso,evento,piso,tramo_id,tipo_tramo,pasos,metros,rumbo_mag,rumbo_verdadero,x_crudo,y_crudo,x,y,nodo_id,lat,lon,precision_m`
- Los eventos son `paso`, `nodo`, `ancla`, `inicio_tramo`, `fin_tramo`, `descarte` y `gps`.
- `lat, lon, precision_m` solo se llenan en los eventos `gps`.
- `x_crudo, y_crudo` es la posición que la app calculó en ese momento. `x, y` es la posición después de los re-anclajes.

## Tipos de arista

Salen del tipo de tramo:
- Pasillo → `pasillo`, o `acceso_edificio` si toca una entrada, o `elevador_acceso` si toca un elevador.
- Andador → `pasillo_exterior`.
- Escalera → `escalera`.
- Rampa → `rampa`.
- Elevador → `elevador_vertical`, con longitud 0.

Al marcar un lugar se agrega `acceso_salon`, y con *Se entra por otro lugar* se agrega `acceso_interno`.

## Mapa y coordenadas

El mapa está georreferenciado contra OpenStreetMap: 1 px = 0.425 m, con un giro de −2.3°. De píxel (px, py), con y hacia abajo, a metros locales este/norte:

```
E =  0.424628·px − 0.017169·py − 370.252
N = −0.017169·px − 0.424628·py + 113.448
```

El origen (0, 0) está en lat 19.3244628, lon −99.1787148. Todos los pisos usan el mapa de planta baja y el rastro cambia de color según el piso.

Para el GPS, (lat, lon) pasa a metros con `E = (lon − lon₀)·R·cos(lat₀)·π/180` y `N = (lat − lat₀)·R·π/180`, con R = 6 371 008.8 m. Es la misma proyección con la que se georreferenció el mapa contra OpenStreetMap.

## Del teléfono a la tesis

En la tesis, el guion `simulacion/esqueleto_medido.py nodos_<fecha>.csv aristas_<fecha>.csv` pasa las medidas de los andadores a la red de circulación del campus (`circulacion_aristas.csv`):
- Cada nodo de la app se asocia con el nodo del esqueleto más cercano, a 15 px o menos.
- Una cadena de tramos entre dos nodos vecinos del esqueleto reemplaza la longitud estimada de esa arista.
- Si en la cadena hay una escalera o una rampa, la arista se parte en un nodo `V_ESC` o `V_RAM`.

Para que esto funcione, en las esquinas por las que ya pasaste usa **¿Es uno de estos?** en lugar de crear un nodo nuevo.

## Laboratorios de Biología

El catálogo oficial de la Facultad (`fciencias.unam.mx/plantel`) solo trae salones para los edificios A y B de Biología. Los 33 laboratorios agregados (ids `ABL01`–`ABL13` y `BBL01`–`BBL20`) salen de las fichas de laboratorios del sitio de la Facultad. Cada nodo guarda en sus notas la URL de su ficha. Dos laboratorios ocupan dos pisos y aparecen una vez por piso.
