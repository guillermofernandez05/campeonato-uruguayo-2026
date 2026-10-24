# Medidas DAX

Las 24 medidas del modelo, copiadas tal cual están en el `.pbix` y agrupadas por página. Las de intervalo de confianza son el núcleo del análisis de localía.

Dos convenciones que se repiten en casi todas:

- **Exclusiones.** Los partidos sin resultado (`Goles_Local` vacío) y la final del Intermedio (`CONTAINSSTRING(Partidos[Torneo], "Final")`), que se jugó en cancha neutral y no suma para la tabla anual.
- **Puntos de un partido.** `SWITCH(TRUE(), ganó, 3, empató, 1, 0)`, calculado fila a fila sobre `Partidos`.

---

## Localía

### El truco del visitante: una relación inactiva

`Partidos` se relaciona con `Equipos` por la columna `Local` (relación activa). Para calcular lo que hizo cada equipo de visitante hay una segunda relación, `Partidos[Visitante] → Equipos[Equipo]`, que está **inactiva** y solo se enciende dentro de las medidas de visitante con `USERELATIONSHIP`. Así, con un mismo eje de equipos, una medida mira sus partidos de local y la otra sus partidos de visitante.

### `PPP Local` / `PPP Visitante`

Puntos por partido de cada equipo según dónde jugó. Sin ningún equipo seleccionado, dan el promedio de toda la liga (1,53 y 1,22).

```dax
PPP Local =
AVERAGEX (
    FILTER (
        Partidos,
        NOT ISBLANK ( Partidos[Goles_Local] )
            && NOT CONTAINSSTRING ( Partidos[Torneo], "Final" )
    ),
    SWITCH (
        TRUE (),
        Partidos[Goles_Local] > Partidos[Goles_Visitante], 3,
        Partidos[Goles_Local] = Partidos[Goles_Visitante], 1,
        0
    )
)

PPP Visitante =
CALCULATE (
    AVERAGEX (
        FILTER (
            Partidos,
            NOT ISBLANK ( Partidos[Goles_Visitante] )
                && NOT CONTAINSSTRING ( Partidos[Torneo], "Final" )
        ),
        SWITCH (
            TRUE (),
            Partidos[Goles_Visitante] > Partidos[Goles_Local], 3,
            Partidos[Goles_Visitante] = Partidos[Goles_Local], 1,
            0
        )
    ),
    USERELATIONSHIP ( Partidos[Visitante], Equipos[Equipo] )
)
```

### `Dependencia Local`

La diferencia local − visitante. Es el largo de las barras del gráfico principal y, a nivel liga, la tarjeta "Puntos de ventaja por jugar de local" (+0,31).

```dax
Dependencia Local = [PPP Local] - [PPP Visitante]
```

### Intervalo de confianza del 95%

El error estándar de la diferencia sale de la varianza de los puntos por partido (0, 1 o 3) de cada lado, dividida por la cantidad de partidos de ese lado (error estándar de Welch para dos muestras independientes). `IC Inferior` e `IC Superior` alimentan las barras de error nativas del gráfico, en modo de cotas absolutas.

```dax
N Local =
COUNTROWS ( FILTER ( Partidos, NOT ISBLANK ( Partidos[Goles_Local] ) && NOT CONTAINSSTRING ( Partidos[Torneo], "Final" ) ) )

N Visitante =
CALCULATE (
    COUNTROWS ( FILTER ( Partidos, NOT ISBLANK ( Partidos[Goles_Visitante] ) && NOT CONTAINSSTRING ( Partidos[Torneo], "Final" ) ) ),
    USERELATIONSHIP ( Partidos[Visitante], Equipos[Equipo] )
)

Desvio PPP Local =
STDEVX.S (
    FILTER ( Partidos, NOT ISBLANK ( Partidos[Goles_Local] ) && NOT CONTAINSSTRING ( Partidos[Torneo], "Final" ) ),
    SWITCH ( TRUE (), Partidos[Goles_Local] > Partidos[Goles_Visitante], 3, Partidos[Goles_Local] = Partidos[Goles_Visitante], 1, 0 )
)

Desvio PPP Visitante =
CALCULATE (
    STDEVX.S (
        FILTER ( Partidos, NOT ISBLANK ( Partidos[Goles_Visitante] ) && NOT CONTAINSSTRING ( Partidos[Torneo], "Final" ) ),
        SWITCH ( TRUE (), Partidos[Goles_Visitante] > Partidos[Goles_Local], 3, Partidos[Goles_Visitante] = Partidos[Goles_Local], 1, 0 )
    ),
    USERELATIONSHIP ( Partidos[Visitante], Equipos[Equipo] )
)

Error Estandar Dependencia =
SQRT ( DIVIDE ( [Desvio PPP Local] ^ 2, [N Local] ) + DIVIDE ( [Desvio PPP Visitante] ^ 2, [N Visitante] ) )

IC Inferior = [Dependencia Local] - 1.96 * [Error Estandar Dependencia]

IC Superior = [Dependencia Local] + 1.96 * [Error Estandar Dependencia]
```

**Color de las barras.** No es una medida sino formato condicional sobre `IC Inferior`: gris cuando es ≤ 0 (el margen incluye el cero) y dorado cuando es > 0. Esta temporada solo Depor. Maldonado queda en dorado.

### Medidas de liga

Las tarjetas de la página de Localía. Se calculan sobre los 264 partidos con resultado, sin la final.

```dax
Partidos Jugados =
COUNTROWS (
    FILTER (
        Partidos,
        NOT ISBLANK ( Partidos[Goles_Local] )
            && NOT CONTAINSSTRING ( Partidos[Torneo], "Final" )
    )
)

% Victorias Local =
DIVIDE (
    COUNTROWS ( FILTER ( Partidos, Partidos[Goles_Local] > Partidos[Goles_Visitante] && NOT CONTAINSSTRING ( Partidos[Torneo], "Final" ) ) ),
    [Partidos Jugados]
)

% Empates =
DIVIDE (
    COUNTROWS ( FILTER ( Partidos, NOT ISBLANK ( Partidos[Goles_Local] ) && Partidos[Goles_Local] = Partidos[Goles_Visitante] && NOT CONTAINSSTRING ( Partidos[Torneo], "Final" ) ) ),
    [Partidos Jugados]
)

% Victorias Visitante =
DIVIDE (
    COUNTROWS ( FILTER ( Partidos, Partidos[Goles_Local] < Partidos[Goles_Visitante] && NOT CONTAINSSTRING ( Partidos[Torneo], "Final" ) ) ),
    [Partidos Jugados]
)
```

---

## Juveniles

Sub-21 = nacidos en 2005 o después. Se mide en **minutos** y no en cantidad de jugadores: un club puede usar diez juveniles cinco minutos cada uno y no estar apostando por ninguno.

```dax
Minutos Totales = SUM ( Estadisticas[Minutos] )

Minutos Sub21 = CALCULATE ( [Minutos Totales], Estadisticas[Año] >= 2005 )

% Minutos Sub21 = DIVIDE ( [Minutos Sub21], [Minutos Totales] )

Jugadores Sub21 =
CALCULATE ( DISTINCTCOUNT ( Estadisticas[Jugador] ), Estadisticas[Año] >= 2005, Estadisticas[Minutos] > 0 )

Goles Sub21 = CALCULATE ( SUM ( Estadisticas[Goles] ), Estadisticas[Año] >= 2005 )

% Goles Sub21 = DIVIDE ( [Goles Sub21], SUM ( Estadisticas[Goles] ) )
```

`% Goles Sub21` va al lado de `% Minutos Sub21` en la página: 14,1% de los goles contra 13,7% de los minutos. Esa comparación es la que le da contexto al número.

`Edad Promedio (ponderada)` es la edad del plantel ponderada por los minutos que jugó cada uno. Aparece en el tooltip del gráfico de minutos sub-21.

```dax
Edad Promedio (ponderada) =
DIVIDE (
    SUMX ( Estadisticas, ( 2026 - Estadisticas[Año] ) * Estadisticas[Minutos] ),
    [Minutos Totales]
)
```

---

## Jugadores

### `Goles cada 90`

Goles por cada 90 minutos jugados. Queda en blanco para jugadores con menos de 5 goles, para que alguien con 1 gol en 40 minutos no encabece el ranking.

```dax
Goles cada 90 =
VAR g = SUM ( Estadisticas[Goles] )
RETURN IF ( g >= 5, DIVIDE ( g * 90, [Minutos Totales] ) )
```

### `% Goles del Goleador`

Qué parte de los goles del equipo hizo su máximo goleador. Mide cuánto depende el ataque de un solo jugador.

```dax
% Goles del Goleador =
DIVIDE (
    MAXX ( VALUES ( Estadisticas[Jugador] ), CALCULATE ( SUM ( Estadisticas[Goles] ) ) ),
    SUM ( Estadisticas[Goles] )
)
```

### `Orden Goleadores`

Medida auxiliar para el filtro "Top 20" de la tabla de goleadores. Ordena por goles y desempata a favor de quien necesitó menos minutos, así el filtro devuelve exactamente 20 filas aunque haya empates en goles.

```dax
Orden Goleadores = SUM ( Estadisticas[Goles] ) * 10000 - SUM ( Estadisticas[Minutos] ) / 10
```

---

## Notas

- **Goles.** Las métricas por jugador usan los 595 goles con autor identificado en FBref; las de equipo (tabla de posiciones y tarjeta "Goles" de Resumen), los 622 de la liga. Los 27 de diferencia son goles en contra o sin autor asignado.
- **Los intervalos asumen independencia entre partidos** y no ajustan por la calidad del rival.
