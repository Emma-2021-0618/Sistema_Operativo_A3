# UCATECI
- Sistemas operativo 2
- Lizandro Jose Ramirez
- Encuentro 7 Teoria

# Autores
- Enmanuel Sanchez Rodriguez 2021-0618
- Albert Francisco Hernandez Sanchez 2019-0126

# Problema de los filosofos.

Hay cinco filósofos sentados alrededor de una mesa que pasan su vida cenando y pensando. Cada uno dispone de un plato de arroz y un palillo a la izquierda de su plato, pero para comer son necesarios dos palillos y cada filósofo sólo puede coger el que está a su izquierda o el que hay a su derecha. Con un solo palillo en la mano no tienen más remedio que esperar hasta que atrapen otro y puedan seguir comiendo.

Si dos filósofos adyacentes intentan tomar el mismo palillo a la vez se produce una condición de carrera: ambos compiten por lo mismo pero uno se queda sin comer.

Si todos los filósofos cogen el palillo de su derecha al mismo tiempo, todos se quedarán esperando eternamente porque alguien debe liberar el palillo que les falta, cosa que nadie hará porque todos se encuentran en la misma situación (esperando que alguno deje su palillo). Llegado esto, los filósofos se morirán de hambre. A este bloqueo mutuo se le denomina interbloqueo o deadlock.

## Librerias.

~~~
import argparse
import threading
import time
import random
~~~

Librerias las cuales usa el programa para funciona ademas de usar otras para leer parametros que pasaremos por consola con diferentes paranmetros usando la libreria argparse, la libreria de hilos para usarla de manera parelala, libreria de time para usar temporizadores y la libreia random para usar numeros pseudo-aleatorio.

## Variables iniciarles.

~~~
estadoFilosofo = None
candados = []
CANTIDAD_FILOSOFOS = 5
MAXIMO_INTENTOS_FALLIDOS = 10 
RAFAGA_COMER = 1
TOTAL_TIEMPO_COMER = 1  

ESTADO_FILOSOFEANDO = "F"
ESTADO_HAMBRIENTO = "H"
ESTADO_COMIENDO = "C"
ESTADO_SATISFECHO = "S"
ESTADO_MUERTO = "M"
~~~

Variables las cuales declaramos para el uso correcto del programa ademas de otras variables de configuracion del programa, como es declarar el estado del filosofo como ninguno, declarar la curacha y que este vacia, la cantidad de filososfos, el numero de intentos de cada filosofo antes de morir, el tiemp que en come cada filosofo y el tiempo que dura cada uno para tomar los palillos.

Tambien declarar los estados de los filosofos que esta bien declarados y asi se puede identificar ademas de que se puede modificar o alterar su comportamiento.

## Palillos.
~~~
def agarrarPalillos(id_filosofo):
    """Trata de tomar los palillos izquierdo y derecho (en ese orden) y 
       devuelve True si ha podido adquirir ambos y False de lo contrario"""

    palillo_izq = candados[id_filosofo]
    palillo_der = candados[(id_filosofo - 1) % CANTIDAD_FILOSOFOS]

    palillo_izq.acquire()

    if palillo_der.acquire(blocking=False):
        return True
    else:
        palillo_izq.release()
        return False


def liberarPalillos(id_filosofo):
    """Liberación de los palillos adyacentes al filosofo"""
    candados[id_filosofo].release()
    candados[(id_filosofo - 1) % CANTIDAD_FILOSOFOS].release()
~~~
En la primera funcion el filosofo busca el palillo de izquierda a derecha a ver si puede comer o esperar a que llegue a una posicion mas o menor a el, si no pasa turno.

La segunda funcion hace de liberacion de los palillos para que otro filosofo pueda comer cuando el ya esta sastifecho.


## Inicio.
~~~
def iniciarSimulacion(id_filosofo):
    """Función que va ejecutar cada filosofo(hilo)"""

    intentos_fallidos = 0
    tiempo_comiendo = 0

    while tiempo_comiendo < TOTAL_TIEMPO_COMER:
        if agarrarPalillos(id_filosofo):
            # Limpiar los intentos, ya que ya ha terminado de comer
            intentos_fallidos = 0

            # Acción de comer
            tiempo_comer = min(RAFAGA_COMER, TOTAL_TIEMPO_COMER - tiempo_comiendo)
            tiempo_comiendo += tiempo_comer
            print(f"[+] Filosofo {id_filosofo} comiendo [{tiempo_comer} seg.]")
            time.sleep(tiempo_comiendo)
            liberarPalillos(id_filosofo)

            # Filosofar
            estadoFilosofo[id_filosofo] = ESTADO_FILOSOFEANDO
            tiempo_filosofar = random.uniform(0, 5)
            print(f"[*] Filosofo {id_filosofo} filosofando[{tiempo_filosofar:.2f} seg.]")
            time.sleep(tiempo_filosofar)
        else:
            estadoFilosofo[id_filosofo] = ESTADO_HAMBRIENTO
            intentos_fallidos += 1

            if intentos_fallidos >= MAXIMO_INTENTOS_FALLIDOS:
                estadoFilosofo[id_filosofo] = ESTADO_MUERTO
                print(f"[-] Filosofo {id_filosofo} muerto por inanición")
                return
            
            tiempo_reintentar = random.uniform(0, 3)
            print(f"[ ] Filosofo {id_filosofo} esperando tenedores"
                  f" Intento {intentos_fallidos} [{tiempo_reintentar:.2f} seg.]")
            time.sleep(tiempo_reintentar)
~~~

Esta funcion es una columna vetrebal del programa ya que cada filosofo pasa por ella para cumplir o cambiar su estado y saber si seguir o no seguir en el proceso de alimentacion de cada filosofo individualmente ademas de resetear ciertos valores o aumentar dependiendo de que este a su dispocion ademas de que si comen entrar en el estado de filosofar que llibera del estado hambriento un tiempo o en el caso de que el fallo es mayor el filosofo muera.


## Argumentos.
~~~
def obtenerArgumentos():
    """Función que lee los argumentos pasados por la línea de comandos"""
    parser = argparse.ArgumentParser()
    parser.add_argument("-n", "--num_filosofos", 
                        type=int, default=5, help="Número de filósofos (hilos)")
    parser.add_argument("-r", "--rafaga_comer", 
                        type=int, default=4, help="Ráfaga de comer de los filósofos")
    parser.add_argument("-t", "--tiempo_total", 
                        type=int, default=10, help="Tiempo total que requiere comer un filosofo para estar satisfecho")
    parser.add_argument("-i", "--num_intentos", 
                        type=int, default=10, help="Cantidad de intentos antes de que el filósofo muera de inanición")
    return parser.parse_args()
~~~

Funcion que se encarga de leer y procesar las variable superiores y que tiene por defecto variabels exactas en el caso de no seer modificada.


## Inicio de los filosofos.

~~~
if __name__ == '__main__':
    args = obtenerArgumentos()

    # Establecer los argumentos leídos por línea de comandos
    CANTIDAD_FILOSOFOS = args.num_filosofos
    RAFAGA_COMER = args.rafaga_comer
    TOTAL_TIEMPO_COMER = args.tiempo_total
    MAXIMO_INTENTOS_FALLIDOS = args.num_intentos

    estadoFilosofo = CANTIDAD_FILOSOFOS * [ESTADO_FILOSOFEANDO]

    # Inicialización de candados
    for _ in range(CANTIDAD_FILOSOFOS):
        candados.append(threading.RLock())

    hilos = []
    for i in range(args.num_filosofos):
        nuevo_hilo = threading.Thread(target=iniciarSimulacion, args=(i,))
        hilos.append(nuevo_hilo)
    
    # Iniciar ejecución de los hilos
    for hilo in hilos:
        hilo.start()

    # Esperar a que terminen todos los hilos
    for hilo in hilos:
        hilo.join()

~~~

Se establecer los parametros modificados en el programa para leer y iniciar el estado de los filosfos  tambien la liberacion del candados, abrimos los hilos y los iniciamos uno a uno, luego ejecutamos los hilos y ya solo queda esperar que termine de iniciar para unirlos.

## Video (Explicación)

[Haga Clic aquí]() 

<br>

## Breve investigación.

<br>

## Concurrencia.
La programación concurrente envuelve lenguajes de programación y algoritmos usados para implementar los sistemas concurrentes. La programación concurrente es considerada más general que la programación paralela porque puede envolver comunicación de patrones dinámicamente. Los sistemas paralelos siempre tienen un sistema de comunicación bien construido. La meta final de la programación concurrente incluye exactitud, rendimiento y robustez. Sistemas concurrentes como los Sistemas Operativos y sistemas de manejo de bases de datos están generalmente diseñados para operar indefinidamente, incluyendo recuperación automática de las fallas, y no terminar su ejecución inesperadamente. Algunos sistemas concurrentes implementan una concurrencia transparente, en el cual las entidades de concurrencia computacional por un único recurso compartido, pero las complejidades de las mismas están escondidas del programador.Como ellas usan recursos compartidos, los sistemas concurrentes en general requieren la inclusión de un tipo de arbiter en algún lugar en la implementación (normalmente en el hardware), para controlar los accesos a esos recursos. El uso arbitrario introduce la posibilidad de indeterminacy in concurrent computation que tiene mayores implicaciones en la práctica, incluyendo la correctitud y el rendimiento.Algunos modelos de programación concurrente incluyen coprocesses y deterministic concurrency. En estos modelos, hilos de control explícitamente mantienen esos pequeños espacios de tiempos, ya sea al sistema o a otro proceso.

<br>

## Paralelismo vs Concurrencia.

Los términos programación paralela y concurrente computacional pueden resultar similares si no se analizan de forma detenida. Sin embargo, estas metodologías de programación pueden ser diferenciadas, pues mientras que la concurrencia se enfoca más en el diseño del software, el paralelismo se relaciona con la ejecución de este.

Asimismo, es posible plantear una diferenciación en lo que respecta al proceder de concurrencia y paralelismo, ya que, por un lado, el paralelismo ejecuta múltiples tareas de forma simultánea, y por otra parte, la concurrencia computacional ejecuta y gestiona diversas tareas al mismo tiempo. Esto quiere decir que la concurrencia contribuye al procesamiento de varias tareas al tiempo y el paralelismo se encarga de dar resolución a una sola tarea de forma más eficiente.

Los requerimientos del paralelismo y concurrencia permiten también marcar una distancia, debido a que, en el caso del paralelismo, es necesaria la implementación de diversas unidades de procesamiento, mientras que la concurrencia computacional puede llevarse a cabo haciendo uso de solo una unidad de procesamiento. De la misma forma, para la programación paralela o paralelismo se requieren múltiples núcleos que se encarguen de cada uno de sus procesos, mientras que para la ejecución de la computación por concurrencia se necesita solo de un núcleo, por lo que su coste por hardware resulta inferior.

Otra de las diferencias entre paralelismo y concurrencia computacional está relacionada con las capacidades que alcanza cada una de estas metodologías. De esta forma, la programación concurrente o concurrencia se utiliza gracias a sus propiedades para elevar la cantidad de trabajo terminado a la vez, mientras reduce el tiempo de respuesta. Por su lado, el paralelismo se responsabiliza del aumento del rendimiento del sistema y de hacer más rápida la ejecución.

<br>

## Hilos.

Un hilo es un proceso del sistema operativo con características distintas de las de un proceso normal:
- Los hilos existen como subconjuntos de los procesos.
- Los hilos comparten memoria y recursos.
- Los hilos ocupan una dirección diferente en la memoria

En Python 2.X se pueden crear hilos utilizando el módulo threads y en Python 3.X se pueden crear utilizando el módulo _threads. El módulo threading será utilizado para interactuar con el módulo _threads.
¿Cuando implementar hilos? Cuando se quiera ejecutar una función al mismo tiempo que se ejecuta un programa. Cuando se crea software para servidores se quiere que el servidor no reciba solo una sino múltiples conexiones. En pocas palabras, los hilos permiten completar varias tareas al mismo tiempo.

<br>

## Deadblock.

Es el bloqueo permanente de un conjunto de procesos o hilos de ejecución en un sistema concurrente que compiten por recursos del sistema o bien se comunican entre ellos. A diferencia de otros problemas de concurrencia de procesos, no existe una solución general para los interbloqueos.

Todos los interbloqueos surgen de necesidades que no pueden ser satisfechas, por parte de dos o más procesos. En la vida real, un ejemplo puede ser el de dos niños que intentan jugar al arco y flecha, uno toma el arco, el otro la flecha. Ninguno puede jugar hasta que alguno libere lo que tomó.

<br>

## Exclusion mutua.

Durante la ejecución simultánea de procesos, los procesos deben ingresar a la sección crítica (o la sección del programa compartida entre procesos) a veces para su ejecución. Puede suceder que, debido a la ejecución de múltiples procesos a la vez, los valores almacenados en la sección crítica se vuelvan inconsistentes. En otras palabras, los valores dependen de la secuencia de ejecución de las instrucciones, también conocida como condición de carrera. La tarea principal de la sincronización de procesos es deshacerse de las condiciones de carrera mientras se ejecuta la sección crítica. Esto se logra principalmente a través de la exclusión mutua. La exclusión mutua es una propiedad de la sincronización de procesos que establece que "no pueden existir dos procesos en la sección crítica en un momento dado". El término fue acuñado por primera vez por Dijkstra. Cualquier técnica de sincronización de procesos que se utilice debe satisfacer la propiedad de exclusión mutua, sin la cual no sería posible deshacerse de una condición de carrera.

<br>

## No preventivo.

Los subprocesos no preferenciales (también conocidos como cooperativos) suelen ceder manualmente el control para permitir que otros subprocesos se ejecuten antes de que finalicen (aunque depende de ese hilo llamar al yield() (o lo que sea) para que eso suceda.
El hilo de prioridad es más simple. tiene menos sobrecarga.
Normalmente, use preventivo. Si encuentra que su diseño tiene una gran cantidad de sobrecarga de cambio de hilo, los hilos cooperativos serían una optimización posible. En muchos (¿la mayoría?) situaciones, esta será una inversión bastante grande con una rentabilidad mínima.
Sí, de manera predeterminada obtendría subprocesamiento, aunque si busca el paquete CThreads, admite el enhebrado cooperativo. Pocas personas suficientes (ahora) quieren hilos cooperativos que no estoy seguro de que ha sido actualizado en la última década.

<br>

## Mantener y espera.

Los procesos que tienen, en un momento dado, recursos asignados con anterioridad, pueden solicitar nuevos recursos y esperar a que se le asignen sin liberar antes alguno de los recursos que ya tenía asignados.

<br>

## Espera circular.
debe existir una cadena circular de dos o más procesos, cada uno de los cuales espera un recurso poseído por el siguiente miembro de la cadena. Esta condición es una consecuencia potencial de las tres primeras, es decir, dado que se producen las tres primeras condiciones, puede ocurrir una secuencia de eventos que desemboque en un círculo vicioso de espera irresoluble.

<br>

## Solucion a tomar para el interbloqueo con respectos a los filosofos.

Tras la detección de un interbloqueo, se pueden aplicar algunas de las siguientes estrategias para resolverlo:

Eliminación: El sistema operativo selecciona a uno de los procesos que forma parte del interbloqueo y elimina el ciclo acabando con la ejecución de dicho proceso, si no es suficiente se eliminarán procesos hasta que se rompa el ciclo. La selección del proceso se realiza en base a un cierto criterio, por ejemplo, aquel proceso que lleve menos tiempo en ejecución o aquel que sea más voraz consumiendo recursos.Sin embargo, de una manera u otra el trabajo realizado por el proceso se pierde, algo que en algunos casos resulta inadmisible, como en sistemas en tiempo real. Aunque parezca una medida drástica, es la empleada en sistemas operativos convencionales. Aplicar el criterio de selección y eliminar procesos cuando el número de procesos es relativamente bajo puede solucionar el interbloqueo, pero si se da un bloqueo de por ejemplo, centenares de procesos, es una situación prácticamente inmanejable.

Apropiación temporal: Se retira la asignación de un recurso a un proceso (durante el tiempo necesario) para deshacer el interbloqueo (hemos de asegurarnos de que el proceso no se desbloquea al romperse el interbloqueo). Por ejemplo, supongamos que el recurso es una impresora: podríamos retirarle la asignación a un proceso P1 cuando este terminase de imprimir una página, asignarle la impresora a otro proceso P2 y volver a asignársela a P1 cuando P2 haya terminado su ejecución. El problema es que este método solo es posible dependiendo de la naturaleza del proceso. Con frecuencia es imposible recuperarse de esta manera ya que los recursos no pueden ser apropiados.

Puntos de conformidad,sincronismo o checkpoints: Consiste en tomar una imagen del estado del proceso, ya sea periódicamente o a instancia del propio proceso, de manera que si se produce un interbloqueo se vuelve a un estado de la ejecución anterior. Son muy poco usados ya que tienen un elevado coste en memoria y existe la posibilidad de que un proceso permanezca indefinidamente sin progresar, y no todos los recursos permiten almacenar y recuperar su estado. Además, puede darse el caso de que el estado del proceso sea externo al sistema (Como en el caso de una conexión a Base de Datos).
