Primero veamos una breve introducción sobre lo que _aportan_ los sistemas operativos:
* Proveen abstracciones a los programas de usuario para poder comunicarse con el hardware.
* Permiten manejar los permisos de forma coherente y segura.
* El contacto primario entre el sistema operativo y los usuarios termina siendo alguna de las siguientes acciones: **escribir, leer, crear y borrar archivos**. 

## Syscalls
Las syscalls terminan siendo esta abstracción que el sistema operativo le provee al usuario. Algunos ejemplos son: **fork(), write(), clone(), etc.** El libro nos presenta una elección: (1) podemos hacer generalidades vagas -el sistema operativo provee syscalls para leer archivos- o podemos ser un poco más específicos: -UNIX tiene una syscall de lectura que requiere 3 parámetros: el archivo, donde colocaremos los datos a leer, y cuántos bytes leeremos-. Se elige la segunda.

Es muy importante recordar que las syscalls varían mucho de sistema a sistema. Cantidad de parámetros, concepto, etc. En esencia, comparten el mismo comportamiento, por lo que es posible, y de hecho, es estándar, tener librerías comunes '_procedure library_' donde tengamos una capa más sobre estas abstracciones. Comunmente está en C.

```c
int main() {
	char buffer[128]; //Array de 128 chars (buffer)
	int fd = open("/home/qobro/uba/so/syscalls/test.txt", O_RDONLY); //Abrimos el archivo en modo READ ONLY

	ssize_t bytes_leidos = read(fd, buffer, 53); //Guardamos la cantidad de bytes leídos (la syscall lo devuelve, suele coincidir con el último parámetro)
	buffer[bytes_leidos] = '\0'; // Agregamos terminador nulo si queremos imprimirlo como string

	printf("Contenido leído (%zd bytes): %s\n", bytes_leidos, buffer); //Stdout
	close(fd); //Cerramos
}
```
Obviando que deberíamos, al ser muy buena práctica, hacer validaciones (por ejemplo if's de si alguna syscall lanza error o algo por el estilo), este código si todo está ok, debería poder mostrar el contenido de `test.txt`. 
> *"If the system call cannot be carried out owing to an invalid parameter or a disk*
*error, count is set to <1, and the error number is put in a global variable, errno.*
*Programs should always check the results of a system call to see if an error*
*occurred"*

Veamos un diagrama simplificado de una llamada a la syscall `read`
![[Pasted image 20260816142147.png]]
1. Primero se pasa fd por el registro RDI (esto en un x86-64 System-V-ABI)
2. Luego pasamos buffer por el RSI
3. Pasamos finalmente nbytes por RDX
4. El SO coloca el código correspondiente de la syscall en el registro RAX
5. Hacemos efectivo el llamado a la función read (jump)
6. Hacemos el trap y pasamos a modo kernel. 
7. Se hace el dispatch hacia el handler
8. Se ejecuta el handler
9. '***Potencialmente'*** volvemos al usuario.
10. Siguiente instrucción

Remarquemos el paso 9: 'potencialmente' supongamos que se estaba pidiendo leer desde el teclado, si no hay ningún input, el sistema se quedará esperando E/S (seguramente el scheduler busque otros procesos que se están ejecutando y cuando nuestro proceso vuelva a pasar de blocked a ready, el scheduler podrá considerarlo de nuevo para ejecución)

### Syscalls para manejo de procesos
Una de las syscalls más importantes (si no la más) es `fork()`, fork permite lanzar un proceso hijo desde un proceso padre. El proceso es exactamente igual por ese instante y devuelve el _pid_ del proceso hijo en el contexto del proceso padre y 0 en el contexto del proceso hijo forkeado.
