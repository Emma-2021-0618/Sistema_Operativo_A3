# UCATECI
- Sistemas operativo 2
- Lizandro Jose Ramirez
- Encuentro 7 Teoria

# Autores
- Enmanuel Sanchez Rodriguez 2021-0618
- Albert Francisco Hernandez Sanchez 2019-0126

# Problema de los filosofos

Hay cinco filósofos sentados alrededor de una mesa que pasan su vida cenando y pensando. Cada uno dispone de un plato de arroz y un palillo a la izquierda de su plato, pero para comer son necesarios dos palillos y cada filósofo sólo puede coger el que está a su izquierda o el que hay a su derecha. Con un solo palillo en la mano no tienen más remedio que esperar hasta que atrapen otro y puedan seguir comiendo.

Si dos filósofos adyacentes intentan tomar el mismo palillo a la vez se produce una condición de carrera: ambos compiten por lo mismo pero uno se queda sin comer.

Si todos los filósofos cogen el palillo de su derecha al mismo tiempo, todos se quedarán esperando eternamente porque alguien debe liberar el palillo que les falta, cosa que nadie hará porque todos se encuentran en la misma situación (esperando que alguno deje su palillo). Llegado esto, los filósofos se morirán de hambre. A este bloqueo mutuo se le denomina interbloqueo o deadlock.

## librerias

~~~
~~~

~~~
import argparse
import threading
import time
import random
~~~

librerias las cuales usa el programa para funciona ademas de usar otras para leer parametros que pasaremos por consola con diferentes paranmetros usando la libreria argparse, la libreria de hilos para usarla de manera parelala, libreria de time para usar temporizadores y la libreia random para usar numeros pseudo-aleatorio

## Variables iniciarles

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

variables las cuales declaramos para el uso correcto del programa ademas de otras variables de configuracion del programa, como es declarar el estado del filosofo como ninguno, declarar la curacha y que este vacia, la cantidad de filososfos, el numero de intentos de cada filosofo antes de morir, el tiemp que en come cada filosofo y el tiempo que dura cada uno para tomar los palillos.

tambien declarar los estados de los filosofos que esta bien declarados y asi se puede identificar ademas de que se puede modificar o alterar su comportamiento.

## Palillos
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
en la primera funcion el filosofo busca el palillo de izquierda a derecha a ver si puede comer o esperar a que llegue a una posicion mas o menor a el, si no pasa turno.

la segunda funcion hace de liberacion de los palillos para que otro filosofo pueda comer cuando el ya esta sastifecho.


## inicio
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

esta funcion es una columna vetrebal del programa ya que cada filosofo pasa por ella para cumplir o cambiar su estado y saber si seguir o no seguir en el proceso de alimentacion de cada filosofo individualmente ademas de resetear ciertos valores o aumentar dependiendo de que este a su dispocion ademas de que si comen entrar en el estado de filosofar que llibera del estado hambriento un tiempo o en el caso de que el fallo es mayor el filosofo muera.


## Argumentos
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

funcion que se encarga de leer y procesar las variable superiores y que tiene por defecto variabels exactas en el caso de no seer modificada.


## inicio de los filosofos

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

se establecer los parametros modificados en el programa para leer y iniciar el estado de los filosfos  tambien la liberacion del candados, abrimos los hilos y los iniciamos uno a uno, luego ejecutamos los hilos y ya solo queda esperar que termine de iniciar para unirlos
