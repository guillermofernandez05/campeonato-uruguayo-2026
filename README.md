# Análisis estadístico del Campeonato Uruguayo 2026

### Localía, juveniles y goleadores

**¿Es verdad que los equipos rinden más de local? ¿Hay clubes que dependen de su estadio más que otros? ¿Y poner juveniles cuesta puntos?** Analicé la temporada 2026 del Campeonato Uruguayo, hasta la fecha 33, para responderlo con datos, y algunas respuestas no son las que uno esperaría.

![Resumen del tablero](capturas/01-resumen.png)

Temporada 2026 del Campeonato Uruguayo, tabla anual (Apertura + Intermedio + Clausura), con datos de FBref hasta la fecha 33: 16 equipos, 533 jugadores y 264 partidos. Tablero de 4 páginas en Power BI.

**Qué muestra este proyecto:** modelado de datos y DAX en Power BI, inferencia estadística y comunicar resultados con sus límites, sin exagerarlos.

---

## Pregunta 1 · ¿Cuánto vale realmente jugar de local?

En el fútbol se da por hecho que la localía pesa: se nota en cómo se arma un fixture, en cómo se cotiza un partido y en cómo se juzga una campaña. Pero pocas veces alguien la mide.

La medí en lo que importa para la tabla: **puntos por partido** (PPP). En los 264 partidos de la temporada, el local sacó **1,53** y el visitante **1,22**. El local ganó el **42,4%** de los partidos, el visitante el **32,2%**, y el **25,4%** terminó empatado.

**Lo que se concluye:** jugar de local vale unos **0,31 puntos por partido**. A lo largo de los 16 o 17 partidos de local de una temporada, son unos **5 puntos en la tabla**, que en el Uruguayo pueden ser la diferencia entre entrar o no a una copa.

Pero hay un matiz: con 264 partidos, esa ventaja queda **justo en el límite de lo que se puede distinguir del azar** (p ≈ 0,05). Es consistente con lo que se observa en el fútbol en general, aunque esta temporada sola apenas alcanza para confirmarla.

## Pregunta 2 · ¿Qué equipo depende más de jugar de local?

Comparé cuántos puntos por partido sumó cada equipo de local y de visitante. A primera vista aparecen casos muy llamativos:

- **Depor. Maldonado** suma **2,24** puntos por partido de local y **1,31** de visitante: casi un punto de diferencia.
- **Racing** y **Defensor** también rinden mucho más de local.
- **Peñarol**, líder de la tabla, hace lo contrario: suma más de visitante (**2,06**) que de local (**1,82**).

Pero hay una trampa. Cada equipo jugó solo unos **16 partidos de local y 16 de visitante**, y con tan pocos partidos los promedios se mueven mucho por casualidad: un par de goles en el último minuto alcanzan para cambiarlos.

Para saber si una diferencia es real o casualidad, le calculé a cada equipo un **margen de error**. Con 16 partidos, ese margen es de **casi un punto por partido**: más grande que casi todas las diferencias que aparecían.

Teniéndolo en cuenta:

- En **15 de los 16 equipos**, la diferencia entre local y visitante podría ser casualidad. Peñarol incluido.
- **Solo Depor. Maldonado** queda afuera de su margen de error, y por muy poco. Pero al comparar 16 equipos, es esperable que alguno lo logre por suerte.

**Lo que se concluye:** con una sola temporada no alcanza para decir que un equipo dependa más de jugar de local que otro. El ranking que parecía contar una historia es, en su mayoría, casualidad.

![Página de localía: diferencia local − visitante con margen de error por equipo](capturas/02-localia.png)

***Cómo leer el gráfico:** la barra muestra cuántos puntos más (o menos) suma cada equipo de local que de visitante. La línea horizontal es el margen de error. Si la línea toca el 0 (la línea punteada), la diferencia podría ser casualidad. Solo Depor. Maldonado, en dorado, no lo toca.*

## Pregunta 3 · ¿Quién apuesta por los juveniles, y le cuesta puntos?

El fútbol uruguayo vive de formar y vender jugadores, así que la pregunta tiene peso: ¿qué clubes les dan lugar a sus juveniles, y lo pagan en la tabla?

Lo medí en **minutos jugados** por sub-21 (nacidos en 2005 o después), no en cantidad de jugadores, porque un club puede usar diez juveniles cinco minutos cada uno y no estar apostando por ninguno.

- **Defensor** es el club que más apuesta: el **40,7%** de sus minutos fueron para sub-21, casi el doble que el segundo (Torque, 23,5%).
- **Depor. Maldonado**, segundo en la tabla, es el que menos: apenas **1,7%**.
- ¿Les cuesta puntos? La correlación entre el % de minutos sub-21 y los puntos es de **−0,21**, débil y no significativa (p = 0,44). Además, la explica casi toda Defensor: **sin ese club queda en −0,04**, prácticamente cero. Torque (4.º) y Racing (5.º) están entre los que más minutos les dan a juveniles.
- Y los sub-21 rinden: jugaron el **13,7%** de los minutos de la liga e hicieron el **14,1%** de los goles.

**Lo que se concluye:** con estos datos, apostar por juveniles no aparece asociado a sumar menos puntos, y los juveniles hacen goles en proporción a lo que juegan.

![Página de juveniles](capturas/03-juveniles.png)

## Más del tablero · Jugadores

La cuarta página mira a los goleadores desde otro ángulo: no solo cuántos goles, sino **cuánto les costó hacerlos** y **cuánto depende cada equipo de uno solo**.

- **Abel Hernández** (Peñarol) es el más eficiente: **0,86 goles cada 90 minutos**, entre los jugadores con al menos 5 goles.
- **Albion** es el equipo que más depende de un jugador: **Álvaro López** hizo el **39%** de sus goles.

![Página de jugadores](capturas/04-jugadores.png)

El tablero completo, en una sola vista, está en [`campeonato_uruguayo.pdf`](campeonato_uruguayo.pdf).

## Limitaciones

- **Una sola temporada.** Es la limitación que domina todo lo demás, y es el hallazgo central del proyecto.
- **Temporada en curso.** Los datos van hasta la fecha 33. Los 16 equipos llevan la misma cantidad de partidos, así que la comparación es pareja, pero los números no son definitivos.
- **Goles sin autor.** La liga tiene 622 goles; 595 tienen autor identificado en FBref. Los otros 27 son en contra o no están asignados. Las métricas por jugador usan los 595; las de equipo, los 622.
- **Los márgenes de error asumen que los partidos son independientes** y no ajustan por la calidad del rival ni por cómo se repartieron los partidos de local y visitante en el fixture.
- **No hay xG ni datos de eventos.** Todo el análisis está construido sobre resultados, goles y minutos.

## Datos

Fuente: [FBref](https://fbref.com) (Sports Reference).

## Autor

**Guillermo Fernández** — estudiante de la Licenciatura en Estadística (Perfil Tecnológico), Udelar.

[LinkedIn](https://www.linkedin.com/in/guillermo-fernandez-r)
