# Campeonato Uruguayo 2026 — análisis en Power BI

**¿Qué equipos dependen más de jugar en su cancha? Armé el ranking, después calculé los márgenes de error, y el ranking se desarmó.**

Análisis de la temporada 2026 del Campeonato Uruguayo (tabla anual: Apertura + Intermedio + Clausura) con datos de FBref. Tablero de 4 páginas en Power BI, con el foco puesto en distinguir lo que los datos sostienen de lo que parece un hallazgo y es ruido.

Datos hasta la fecha 33. 16 equipos · 533 jugadores · 264 partidos.

![Página de localía](capturas/02-localia.png)

---

## El problema

La ventaja de jugar de local es un supuesto que todo el mundo da por cierto en el fútbol, y se usa para decidir: cómo se arma un fixture, cómo se cotiza un partido, cómo se evalúa una campaña.

La pregunta concreta: **¿cuánto vale realmente jugar en casa, y se puede afirmar que un club en particular depende más de su cancha que el resto?**

Como pregunta secundaria: **¿qué clubes les dan minutos reales a los juveniles, y les cuesta puntos hacerlo?**

## El enfoque

Calculé puntos por partido (PPP) de local y de visitante para cada uno de los 16 equipos sobre la tabla anual. Para cada club, la diferencia local − visitante con su intervalo de confianza del 95%.

El corte por club es el que importa, porque es el nivel al que efectivamente se toman las decisiones. Y es justamente el nivel donde la muestra se queda corta: cada equipo jugó unos 16 partidos de cada lado.

**Exclusiones:** partidos sin resultado, y la final del Intermedio (se jugó en cancha neutral y no suma para la tabla anual).

**Definiciones:** PPP = promedio de puntos por partido jugado. Sub-21 = nacidos en 2005 o después, medido en minutos jugados y no en cantidad de jugadores.

## Los hallazgos

### 1. La ventaja de local existe a nivel liga, y es modesta

El local saca **1,53 puntos por partido** y el visitante **1,22**: una ventaja de **+0,31**. El local gana el 42,4% de los partidos, el visitante el 32,2%, y el 25,4% termina empatado. Con 264 partidos, esa diferencia queda **justo en el límite de la significancia (p ≈ 0,05)**.

### 2. A nivel club, el ranking es ruido

Con unos 16 partidos de cada lado por equipo, el margen de error de la diferencia ronda **±0,9 puntos por partido**. **15 de los 16 intervalos cruzan el cero.**

Depor. Maldonado (+0,92) es el único cuyo intervalo no toca el cero, y por poco. Pero al comparar 16 equipos es esperable que alguno supere el umbral por azar, y **un test de heterogeneidad entre clubes da p = 0,92**: no hay evidencia de que unos dependan más de su cancha que otros.

El caso más llamativo: **Peñarol**, primero en la tabla, sumó más de visitante (2,06) que de local (1,82). También está dentro del ruido — su intervalo va de −1,1 a +0,7.

### 3. Darles minutos a los juveniles no parece costar puntos

Defensor les dio el **41%** de sus minutos a sub-21; Depor. Maldonado, **1,7%**. La correlación entre % de minutos sub-21 y puntos es de **−0,21, no significativa**, y la explica casi toda Defensor: sin ese club queda en **−0,04**. Torque (4.º) y Racing (5.º) están entre los que más minutos les dan.

Los sub-21 jugaron el **13,7%** de los minutos e hicieron el **14,1%** de los goles: rinden en proporción a lo que juegan.

## Qué haría con esto

No usar splits local/visitante de una sola temporada para decisiones a nivel club. Para que la pregunta tenga respuesta hace falta agrupar varias temporadas, o estimar con un modelo que encoja los efectos individuales hacia la media de la liga.

Lo que sí es usable hoy es el valor agregado de la liga: **+0,31 puntos por partido**.

## Limitaciones

- **Una sola temporada.** Es la limitación que domina todo lo demás y es el hallazgo central del proyecto.
- **Temporada en curso.** Los datos van hasta la fecha 33. Los 16 equipos llevan la misma cantidad de partidos, así que la comparación local/visitante es pareja, pero los números no son definitivos.
- **Goles sin autor.** La liga tiene 622 goles; 595 tienen autor identificado en FBref. Los otros 27 son en contra o no están asignados. Las métricas por jugador usan los 595; las de equipo, los 622.
- **Los intervalos asumen independencia entre partidos** y no ajustan por calidad de rival ni por el reparto local/visitante del fixture.
- **No hay xG ni datos de eventos.** Todo el análisis está construido sobre resultados, goles y minutos.

---

## Estructura del repositorio

```
.
├── README.md
├── campeonato_uruguayo.pdf          # el tablero exportado, 4 páginas
├── capturas/                        # imágenes de cada página
├── datos/                           # las cuatro tablas que consume el tablero, en CSV
│   ├── equipos.csv
│   ├── clasificacion.csv
│   ├── partidos.csv
│   └── estadisticas.csv
└── powerbi/
    ├── campeonato_uruguayo.pbix     # el archivo de Power BI
    └── medidas-dax.md               # las 24 medidas DAX, documentadas
```

## Cómo se armaron los datos

Las tablas se recopilaron de FBref en una planilla, una hoja por tabla, y se guardaron como CSV en `datos/`. La recopilación no está automatizada en este repositorio: los CSV son el punto de partida.

El resto del proceso está dentro del `.pbix`:

- **Power Query:** tipos de datos y alguna corrección. Los CSV quedan tal como vinieron.
- **DAX:** las exclusiones (partidos sin resultado y la final del Intermedio) se aplican dentro de las medidas, así cualquier visual que las use respeta la misma regla.

## El modelo

Cuatro tablas:

| Tabla | Qué contiene |
|---|---|
| `Equipos` | Dimensión: un registro por club, con escudo |
| `Clasificacion` | Tabla de posiciones: PJ, G, E, P, GF, GC, DG, Pts |
| `Partidos` | Un registro por partido, con torneo, local, visitante y resultado |
| `Estadisticas` | Un registro por jugador: club, posición, año de nacimiento, minutos, goles, asistencias, tarjetas |

`Partidos` se relaciona con `Equipos` por el equipo local. Para analizar lo que hizo cada equipo de visitante hay una segunda relación por la columna `Visitante`, inactiva, que se activa solo dentro de las medidas de visitante con `USERELATIONSHIP`.

Las medidas —incluidas las de intervalo de confianza— están en [`powerbi/medidas-dax.md`](powerbi/medidas-dax.md).

## Cómo abrirlo

Abrir `powerbi/campeonato_uruguayo.pbix` con Power BI Desktop (gratuito). El archivo ya trae los datos cargados. Para volver a leerlos desde los CSV: *Inicio → Transformar datos → Editar parámetros*, poner en `RutaDatos` la ruta de la carpeta `datos` de tu copia del repositorio (terminada en `\`) y *Actualizar*. Sin Power BI, el tablero completo está en [`campeonato_uruguayo.pdf`](campeonato_uruguayo.pdf).

## Datos

Fuente: [FBref](https://fbref.com) (Sports Reference). Los datos se incluyen acá con fines educativos y de portfolio, bajo los [términos de uso de Sports Reference](https://www.sports-reference.com/termsofuse.html). Este proyecto no tiene afiliación con Sports Reference, la AUF ni ningún club.

## Autor

**Guillermo Fernández** — estudiante de la Licenciatura en Estadística (Perfil Tecnológico), Udelar.

[LinkedIn](https://www.linkedin.com/in/guillermo-fernandez-r)
