# Resumen promo

Estado: Sin empezar
ítem principal: Programación Concurrente (Programaci%C3%B3n%20Concurrente%202520675b4f6f801eaf70d390ea0e041e.md)

### Defina programa concurrente, paralelo y distribuido

Un programa concurrente es aquel que tiene distintas tareas/procesos, los cuales son programas secuencuales, que se ejecutan de forma simultánea

En un programa paralelo, los procesos se ejecutan en distintos procesadores, al mismo tiempo, por lo tanto, en paralelo.

Un programa distribuido, los procesos se comunican con pasajes de mensajes, RPC o rendezvous. 

---

La concurrencia estudia procesos que se ejecutan en paralelo, el problema aparece cuando esos procesos compartes recursos como memoria, variables, impresoras, etc. De ahí nacen dos riesgos principales:

**Interferencia:** dos procesos acceden o modifican la misma variable y el resultado depende del orden (no determinismo)

**Sincronización:** coordinar las secuencias de ejecución para evitar interferencias o garantizar orden.

Entonces, se tiene que preservar la consistencia de los datos y maximizar la concurrencia.

---

### Condiciones deseables de un sistema concurrente

**Exclusión mutua:** solo un proceso a la vez accede a la SC.

**Progreso:** si nadie está en la SC, alguno que quiera entrar debe poder hacerlo

**Espera limitada:** nadie espera indefinidamente para entrar (no inanición)

**Ausencia de deadlock:** no queden todos bloqueados esperando recursos entre sí

---

### 5 errores típicos

**Atomicidad de grano grueso:** toda la SC dentro de un único lock genera menos concurrencia, por lo tanto,

**Atomicidad de grano fino:** proteger solo lo mínimo necesario es más eficiente

**Starvation:** si la política de turno no es justa

**Deadlock:** ciclos de espera mutua, cada uno tiene un recurso y espera otro.

---

### Semáforos (PDF Clase 3)

Un semáforo es una variable entera que solo puede ser modificada mediante dos operaciones atómicas:

P(s): espera a que s > 0 bloqueando al proceso, y una vez se cumple s>0, despierta y decrementa el valor de este semáforo.

V(s) señala a un proceso bloqueado incrementando el valor de s en uno.

---

### Passing the baton

Este patrón evita avalanchas de señales (muchos procesos despertando a la vez) y garantiza que la EM pasa de un proceso a otro directamente. En esta técnica, nunca hay más de un proceso despierto a la vez, y el “libre”, funciona como testigo del recurso 

---

### Barreras

Sirven para sincronizar N procesos en un punto común (nadie avanza hasta que todos llegaron)

---

### Qué es un monitor

Es una abstracción de sincronización de alto nivel

A diferencia de los semáforos, el compilador garantiza la EM automática.

### Variables cond

Las variables condición sirve para esperar un evento dentro del monitor, son como los semáforos privados, pero no conservan señales:

- wait(c) el proceso se bloquea y libera el montior
- signal(c) despierta exactamente a uno que espera en c (ordenadamente). Si no hay nadie esperando el signal se pierde.

**El monitor puede implementar distintas políticas de atención**

| Política | Cómo se modela |
| --- | --- |
| **FIFO** (orden de llegada) | `cola C` con `push/pop` simples |
| **Prioridad** | `colaOrdenada` (por edad, tipo, etc.) |
| **Múltiples colas** | una para cada tipo (p. ej. prioritarios / comunes) |

**Usar `if` en lugar de `while`** en `wait()` → causa condiciones de carrera.

---