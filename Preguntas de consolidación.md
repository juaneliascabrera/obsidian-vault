1. ¿Qué significan todos los argumentos de la función `int main()` de C?

**Respuesta:**
`main()` puede declararse de varias formas; la completa es `int main(int argc, char *argv[])` (y en muchos Unix también `char *envp[]`):

- `int` (valor de retorno): es el **código de salida** del proceso. `0` indica éxito y un valor distinto de `0` indica error. Ese valor lo recibe el proceso padre (o la shell) y puede inspeccionarlo con `wait()`/`waitpid()` o con `$?` en bash.
- `int argc` (argument count): cantidad de argumentos que recibe el programa desde la línea de comandos, **incluyendo el nombre del programa**. Por eso siempre es al menos `1`. En `./prog a b c`, `argc == 4`.
- `char *argv[]` (argument vector): array de cadenas. `argv[0]` es el nombre del programa, `argv[1]` es el primer argumento, ..., `argv[argc-1]` es el último, y `argv[argc]` es `NULL` (marca el fin del array).
- `char *envp[]` (opcional, no es C estándar): array con las variables de entorno, cada una con la forma `NOMBRE=valor`, terminado en `NULL`.

2. Entiendo que `fork()` devuelve 0 para el hijo y el PID del hijo para el padre, ¿pero qué significa que te devuelva -1 y todas las otras cosas que te puede devolver?

**Respuesta:**
`fork()` devuelve exactamente una de estas tres posibilidades, y con eso distinguís en qué proceso estás:

- `0` → estás en el **proceso hijo**. Ojo: `0` NO es el PID del hijo; es solo un marcador para decir "soy el hijo".
- `> 0` → estás en el **proceso padre**, y ese valor es el **PID real del hijo** recién creado. Es la única forma que tiene el padre de conocer ese PID.
- `-1` → **falló la creación**: no se creó ningún hijo. El motivo concreto queda en la variable global `errno`:
  - `EAGAIN`: se superó el límite de procesos del usuario (`RLIMIT_NPROC`) o del sistema, o no hay recursos para crear la tabla/stack del proceso.
  - `ENOMEM`: no hay memoria suficiente para el nuevo proceso.

No hay otros valores posibles; "negativos distintos de -1" no ocurren. Por eso el patrón típico es `if (pid < 0) { /* error */ }`, `else if (pid == 0) { /* hijo */ }`, `else { /* padre */ }`.

3. ¿Cuáles son los headers correctos para las syscalls más utilizadas? fork() -que aparentemente no es una syscall sino un wrapper sobre clone >:c-, wait(), waitpid(), kill(), etc.
4. ¿Cuáles son las señales más comunes?