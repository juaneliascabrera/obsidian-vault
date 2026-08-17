Vamos a hacer algunas definiciones importantes en la materia para no confundirlas:
* **Programa**: Secuencia de pasos escrita en algún lenguaje de programación.
* **Proceso:** Instancia ejecutándose de un programa. Se le asigna un *pid* o *process id*.

### Proceso
Un proceso, en esencia, está compuesto por un área de texto (código máquina del programa), un área de datos (donde almacenamos el heap) y un stack para el proceso en sí. 

#### ¿Qué puede hacer un proceso? 
Relativamente pocas cosas: 
* Terminar
* Lanzar un proceso hijo (`system(), fork(), exec()`)
* Ejecutar algo en la CPU
* **Hacer una syscall**
* Realizar entrada/salida a los dispositivos.

#### Terminación de un proceso
Al terminar, el proceso le indica al SO que ya puede liberar todos sus recursos (`exit()`), a su vez, indica su status de terminación (0  = ok, y se usan distintos números para distintos errores)

Ese código de status le es reportado al **padre** del proceso. 

#### Árbol de procesos
Todos los procesos están organizados jerárquicamente como un árbol. Al comenzar, el SO lanza un proceso llamado **init o systemd**. Luego, todos los demás procesos viven dentro de esta jerarquía (en última instancia, todos parten de systemd/init)

#### fork
`fork()` es una llamada al sistema que crea un proceso exactamente igual al actual. El resultado es el *pid* del proceso hijo y el padre puede suspenderse hasta que el hijo termine (`wait()`)