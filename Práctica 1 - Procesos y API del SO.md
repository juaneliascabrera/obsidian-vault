
## Parte 1 - Estado y operaciones sobre procesos

1) **¿Cuáles son los pasos que deben llevarse a cabo para realizar un cambio de contexto?**
	* Interrupción o syscall: paso a modo kernel.
	* Guardado de IP
	* Guardado de registros en el PCB del proceso.
	* El scheduler ejecuta el siguiente proceso elegido.
	* Se carga MMU y PCB del proceso elegido.
	* Se cargan registros del nuevo proceso
	* Se vuelve a modo usuario
	2) 
```c
void Ke_context_switch(PCB* pcb_0, PCB* pcb_1) {
    // 1. Acumulamos el tiempo que este proceso usó la CPU en su quantum
    pcb_0->CPU_TIME = pcb_0->CPU_TIME + ke_current_user_time();

    // 2. Guardamos el contexto de CPU del proceso actual en su PCB
    pcb_0->PC = PC; 
    pcb_0->R0 = R0;
    pcb_0->R1 = R1;
    // ... (podés usar la notación abreviada R0...R15 si es pseudocódigo)
    pcb_0->R15 = R15; // Guardamos el Stack Pointer (SP) de pcb_0

    // 3. Actualizamos los estados en los PCB correspondientes
    pcb_0->STAT = KE_READY;
    pcb_1->STAT = KE_RUNNING;

    // 4. Le avisamos al kernel cuál es el nuevo proceso en ejecución
    set_current_process(pcb_1->P_ID);

    // 5. Reseteamos el reloj del sistema para el nuevo quantum
    ke_reset_current_user_time();

    // 6. Restauramos los registros del proceso que va a ejecutarse
    R0 = pcb_1->R0;
    R1 = pcb_1->R1;
    // ...
    R15 = pcb_1->R15; // ¡Restauramos la pila de pcb_1! Su tope ahora tiene su PC de retorno.

    // 7. Volvemos al espacio de usuario reanudando el proceso nuevo
    ret(); // Desapila la dirección de retorno de pcb_1 y la carga en el PC
}
```
3) Describir la diferencia entre un system call y una llamada a función de biblioteca. 
	Básicamente, una system call o syscall es un tipo de función particular que provee el SO como API para comunicarse con él y poder acceder a los recursos del hardware mediante una abstracción, volviendo agnóstico al desarrollador de tener que conocer el hardware del sistema sobre el que está programando. Una función de biblioteca es de más alto nivel y puede estar compuestas por otras syscalls. Además, las funciones de biblioteca corren en nivel de usuario (ring 3) y las syscalls en nivel de kernel (nivel 0).
4)  ![[Pasted image 20260816170049.png]]
	* De ready puede pasar a running si el scheduler lo elige
	* De running puede pasar a ready si el SO decide que terminó su tiempo
	* De running puede pasar a blocked si pide un servicio externo (por ej: E/S)
	* De running puede pasar a terminated si se usa exit() 
	* De blocked puede pasar a terminated si mandamos sigkill.
	* De blocked puede pasar a ready si obtiene su dato que lo bloqueó.
	* De new pasa a ready cuando se termina de crear el PCB
	* New es el estado inicial de cada proceso apenas fue creado. Luego, pasa a ready.