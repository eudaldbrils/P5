PAV - P5: síntesis musical polifónica
=====================================

Obtenga su copia del repositorio de la práctica accediendo a [Práctica 5](https://github.com/albino-pav/P5) 
y pulsando sobre el botón `Fork` situado en la esquina superior derecha. A continuación, siga las
instrucciones de la [Práctica 2](https://github.com/albino-pav/P2) para crear una rama con el apellido de
los integrantes del grupo de prácticas, dar de alta al resto de integrantes como colaboradores del proyecto
y crear la copias locales del repositorio.

Como entrega deberá realizar un *pull request* con el contenido de su copia del repositorio. Recuerde que
los ficheros entregados deberán estar en condiciones de ser ejecutados con sólo ejecutar:

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~.sh
  make release
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

A modo de memoria de la práctica, complete, en este mismo documento y usando el formato *markdown*, los
ejercicios indicados.

Ejercicios.
-----------

### Envolvente ADSR.

Tomando como modelo un instrumento sencillo (puede usar el InstrumentDumb), genere cuatro instrumentos que
permitan visualizar el funcionamiento de la curva ADSR.

* Un instrumento con una envolvente ADSR genérica, para el que se aprecie con claridad cada uno de sus
  parámetros: ataque (A), caída (D), mantenimiento (S) y liberación (R).
  
  En este caso se observa una envolvente ADSR genérica. Al principio aparece el ataque que es el trozo creciente de la gráfica, seguidamente aparece la caída que sucede después del ataque. Mientras el músico no libera la nota que está tocando esta se mantiene a una cierta intensidad, este tramo se llama de mantenimiento. Cúando el músico libera la nota aparece el tramo de liberación donde la onda se extinge. 
  
![imagen](https://user-images.githubusercontent.com/91128741/174171564-c4eab114-60c0-4b75-b88d-247d58a46ec9.png)


* Un instrumento *percusivo*, como una guitarra o un piano, en el que el sonido tenga un ataque rápido, no
  haya mantenimiemto y el sonido se apague lentamente.
  
  - Para un instrumento de este tipo, tenemos dos situaciones posibles:
  - 
    * El intérprete mantiene la nota *pulsada* hasta su completa extinción.
	
	En este caso se observa como el músico mantiene la nota pulsada hasta la desaparición del sonido, la nota después del ataque desciende sin mantenimiento. Se llega a la extinción natural del sonido.
![imagen](https://user-images.githubusercontent.com/91128741/174170420-e343556d-32d1-42db-9e7d-562496228805.png)




    * El intérprete da por finalizada la nota antes de su completa extinción, iniciándose una disminución
	  abrupta del sonido hasta su finalización.
	  
	  En este caso, en cambio, se ve como no aparece el estado de mantenimiento ni el de liberación. Eso es debido a que el músico ha decidido parar de golpe el sonido y por lo tanto no se ha llegado a la extinción natural del sonido.
	  
	 ![imagen](https://user-images.githubusercontent.com/91128741/174172151-8377f368-c1ad-461f-bb78-090cb4b81d0d.png)


  - Debera representar en esta memoria **ambos** posibles finales de la nota.
* Un instrumento *plano*, como los de cuerdas frotadas (violines y semejantes) o algunos de viento. En
  ellos, el ataque es relativamente rápido hasta alcanzar el nivel de mantenimiento (sin sobrecarga), y la
  liberación también es bastante rápida.
  
  Por otro lado, los instrumentos de viento no tienen el tramo de caída, simplemente se desvanece el sonido cuando se termina la nota.
  
![imagen](https://user-images.githubusercontent.com/91128741/174173462-8793d23c-7ee7-4f65-afe5-73bd5e533eb8.png)

Para los cuatro casos, deberá incluir una gráfica en la que se visualice claramente la curva ADSR. Deberá
añadir la información necesaria para su correcta interpretación, aunque esa información puede reducirse a
colocar etiquetas y títulos adecuados en la propia gráfica (se valorará positivamente esta alternativa).

### Instrumentos Dumb y Seno.

Implemente el instrumento `Seno` tomando como modelo el `InstrumentDumb`. La señal **deberá** formarse
mediante búsqueda de los valores en una tabla.

- Incluya, a continuación, el código del fichero `seno.cpp` con los métodos de la clase Seno.
```c++
#include <iostream>
#include <math.h>
#include "seno.h"
#include "keyvalue.h"

#include <stdlib.h>

using namespace upc;
using namespace std;

Seno::Seno(const std::string &param) : adsr(SamplingRate, param) {
  bActive = false;
  x.resize(BSIZE);

  KeyValue kv(param);
  int N;
  if (!kv.to_int("N",N))
    N = 40; //default value
  FILE *f;
  f = fopen("taula.log","a");
  tbl.resize(N);
  float phase = 0, step = 2 * M_PI /(float) N;
  index = 0;
  for (int i=0; i < N ; ++i) {
    tbl[i] = sin(phase);
    fprintf(f,"%f\n",tbl[i]);
    phase += step;
  }
  fclose(f);
}
void Seno::command(long cmd, long note, long vel) {
  if (cmd == 9) {		//'Key' pressed: attack begins
    bActive = true;
    adsr.start();
    phase = 0;
    float F0=440.0*pow(2,(((float)note-69.0)/12.0))/SamplingRate; 
    A=vel/128.0;
    step=2*M_PI*F0;
    cout << step;
  }
  else if(cmd==0 || cmd==8){
    adsr.stop();
  }
}
const vector<float> & Seno::synthesize() {
  if (not adsr.active()) {
    x.assign(x.size(), 0);
    bActive = false;
    return x;
  }   
  else if (not bActive)
    return x;

  FILE *fp;
  fp = fopen("xvec.log","a");
  for (unsigned int i=0; i<x.size(); ++i) {
    x[i] = 0.3*A*sin(phase);
    fprintf(fp,"%f\n",x[i]);
    phase = phase + step;
    while(phase>2*M_PI)
      phase = phase - 2*M_PI;
  }
  adsr(x); //apply envelope to x and update internal status of ADSR
  fclose(fp);
  return x;
}

```
- Explique qué método se ha seguido para asignar un valor a la señal a partir de los contenidos en la tabla,
  e incluya una gráfica en la que se vean claramente (use pelotitas en lugar de líneas) los valores de la
  tabla y los de la señal generada.
  
  La tabla simplemente guarda en un vector de N posiciones sin(2*pi/N) y cada posición que se avanza corresponde a variar el argumento. float phase = 0, step = 2 * M_PI /(float) N;
phase += step;

Para crear el instrumento Seno se necesita que la fase del seno empiece por 0. Para empezar se debe introducir la f0 a la que se trabaja, utilizamos la siguiente fórmula: Note=69+12·log2(f0/440) y aislamos f0. Se pone valor a la variable step, este valor nos ayudará a iterar el argumento del seno cuando se utiliza la sintetización.
  

- Si ha implementado la síntesis por tabla almacenada en fichero externo, incluya a continuación el código
  del método `command()`.

### Efectos sonoros.

- Incluya dos gráficas en las que se vean, claramente, el efecto del trémolo y el vibrato sobre una señal
  sinusoidal. Deberá explicar detalladamente cómo se manifiestan los parámetros del efecto (frecuencia e
  índice de modulación) en la señal generada (se valorará que la explicación esté contenida en las propias
  gráficas, sin necesidad de *literatura*).
  Tremolo
  ![imagen](https://user-images.githubusercontent.com/91128741/174166406-6c8e0572-e9e0-4b9f-a435-bc3b64cd0b6a.png)
  
  Vibrato
  
  ![image](https://user-images.githubusercontent.com/91085077/174452629-0ef1e3af-6a1f-4006-bb57-16ceb93696a4.png)

	
- Si ha generado algún efecto por su cuenta, explique en qué consiste, cómo lo ha implementado y qué
  resultado ha producido. Incluya, en el directorio `work/ejemplos`, los ficheros necesarios para apreciar
  el efecto, e indique, a continuación, la orden necesaria para generar los ficheros de audio usando el
  programa `synth`.
  

### Síntesis FM.

Construya un instrumento de síntesis FM, según las explicaciones contenidas en el enunciado y el artículo
de [John M. Chowning](https://web.eecs.umich.edu/~fessler/course/100/misc/chowning-73-tso.pdf). El
instrumento usará como parámetros **básicos** los números `N1` y `N2`, y el índice de modulación `I`, que
deberá venir expresado en semitonos.

- Use el instrumento para generar un vibrato de *parámetros razonables* e incluya una gráfica en la que se
  vea, claramente, la correspondencia entre los valores `N1`, `N2` e `I` con la señal obtenida.
  El resulatado se ha obtenido aplicando el siguiente comando:
  ![image](https://user-images.githubusercontent.com/91085077/174477193-75d31691-9225-43a8-aa2e-0b0934361659.png)
  
  La gráfica es la siguiente:
  
  
- Use el instrumento para generar un sonido tipo clarinete y otro tipo campana. Tome los parámetros del
  sonido (N1, N2 e I) y de la envolvente ADSR del citado artículo. Con estos sonidos, genere sendas escalas
  diatónicas (fichero `doremi.sco`) y ponga el resultado en los ficheros `work/doremi/clarinete.wav` y
  `work/doremi/campana.work`.
  
  Los parámetros utilizados para la campana son los siguientes:
  
  ![image](https://user-images.githubusercontent.com/91085077/174478480-fe9bd1b7-1982-41ff-8373-860f456ac257.png)
  
  Los parámetros utilizados para el clarinete son los siguientes:
  
  ![image](https://user-images.githubusercontent.com/91085077/174478467-cfa8e294-e402-400c-9728-3be04af0614a.png)
  
  Los ficheros resultantes se pueden encontrar encontrar en la carpeta mencionada por el enunciado
  * También puede colgar en el directorio work/doremi otras escalas usando sonidos *interesantes*. Por
    ejemplo, violines, pianos, percusiones, espadas láser de la
	[Guerra de las Galaxias](https://www.starwars.com/), etc.

### Orquestación usando el programa synth.

Use el programa `synth` para generar canciones a partir de su partitura MIDI. Como mínimo, deberá incluir la
*orquestación* de la canción *You've got a friend in me* (fichero `ToyStory_A_Friend_in_me.sco`) del genial
[Randy Newman](https://open.spotify.com/artist/3HQyFCFFfJO3KKBlUfZsyW/about).

- En este triste arreglo, la pista 1 corresponde al instrumento solista (puede ser un piano, flautas,
  violines, etc.), y la 2 al bajo (bajo eléctrico, contrabajo, tuba, etc.).
  Los instrumentos utilizados y los parámetros para crearlos son los siguientes:
  
![image](https://user-images.githubusercontent.com/91085077/174483966-96c6e51f-1b2b-451a-bbca-e48be75452b0.png)


- Coloque el resultado, junto con los ficheros necesarios para generarlo, en el directorio `work/music`.
- Indique, a continuación, la orden necesaria para generar la señal (suponiendo que todos los archivos
  necesarios están en directorio indicado).
  
  ![image](https://user-images.githubusercontent.com/91085077/174479481-7fb19919-6707-4352-9589-ee0ee13ad542.png)


También puede orquestar otros temas más complejos, como la banda sonora de *Hawaii5-0* o el villacinco de
John Lennon *Happy Xmas (War Is Over)* (fichero `The_Christmas_Song_Lennon.sco`), o cualquier otra canción
de su agrado o composición. Se valorará la riqueza instrumental, su modelado y el resultado final.
- Coloque los ficheros generados, junto a sus ficheros `score`, `instruments` y `efffects`, en el directorio
  `work/music`.
- Indique, a continuación, la orden necesaria para generar cada una de las señales usando los distintos
  ficheros.
