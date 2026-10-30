# Exclusión Mutua/Condición

Estado: Sin empezar
ítem principal: Programación Concurrente (Programaci%C3%B3n%20Concurrente%202520675b4f6f801eaf70d390ea0e041e.md)

5.d

```c
bool libre = true; Cola C; int sig = -1;   // variables globales

Process Persona[id:0..N-1]
{
  while (true)
  {
    <Agregar(C, id)>;                   // cada persona se anota en la cola
    <await(sig == id)>;                  // espera hasta que el coordinador le asigne el turno
    Imprimir(documento);
    <libre = true;
     if (empty(C)) sig = -1;
     else sig = sacar(C);>;              // aviso que terminé y paso al siguiente
  }
}

Process Coordinador
{
  while (true)
  {
    <await(!empty(C) && libre)>;         // espero que haya alguien en la cola y la impresora libre
    sig = primero(C);                    // elijo a quién le toca (p. ej., el primero de la cola)
    libre = false;                       // marco que la impresora está ocupada
  }
}
```

Análisis de las 4 propiedades:
Exclusión mutua ✅

```c
int turno = 1;

Process SC1::
{
while (true) {
while (turno == 2) skip;   // espera ocupada
SC;
turno = 2;
SNC;
}
}

Process SC2::
{
while (true) {
while (turno == 1) skip;   // espera ocupada
SC;
turno = 1;
SNC;
}
}
```

### Análisis de las 4 propiedades:

1. **Exclusión mutua ✅**
    - Sí se cumple: nunca pueden estar los dos en la sección crítica al mismo tiempo, porque la variable `turno` garantiza que solo uno pasa.
2. **Progreso 🚨 (NO se cumple bien)**
    - El problema: si `SC1` termina su sección crítica y asigna `turno = 2`, pero `SC2` está en su sección no crítica (no quiere entrar todavía), entonces `SC1` queda bloqueado en `while(turno==2)` aunque la sección crítica esté libre.
    - Eso viola la condición de progreso (porque aunque nadie esté en SC, un proceso que quiere entrar no puede hacerlo).
3. **Espera acotada ❌**
    - No se garantiza: puede darse que siempre quede bloqueado el mismo proceso si el otro está mucho tiempo en su SNC y luego reentra justo cuando el otro quiere usar SC.
    - En particular, el proceso que no tiene el turno debe **esperar siempre** aunque el otro no quiera entrar → eso es una forma de **inanición**.
4. **Ausencia de interbloqueo ✅**
    - Sí se cumple: no puede pasar que los dos queden bloqueados permanentemente, porque la variable `turno` siempre le da la chance a alguno.

---

Por lo tanto, esta solución **no es correcta en general** (la exclusión está bien, pero se rompe el progreso y la espera acotada).

—-

```c
// Variables globales compartidas
bool flag[0..N-1] = {false};     // cada proceso avisa si quiere entrar
bool permiso[0..N-1] = {false};  // coordinador habilita el acceso

Process SC[i:0..N-1]
{
  while (true)
  {
    flag[i] = true;// Aviso que quiero entrar
    while (permiso[i] == false) skip;// Espero el permiso del coordinador
    SC;
    flag[i] = false;   // Aviso que terminé
    permiso[i] = false;   // libero mi permiso
    SNC;
  }
}

Process Coordinador
{
  while (true)
  {
    // Recorre los procesos buscando alguno que haya pedido entrar
    for j in 0..N-1 {
      if (flag[j] == true && permiso[j] == false) {
        permiso[j] = true;    // le doy permiso a ese proceso
        // El coordinador puede ser justo, por ejemplo atendiendo en orden
      }
    }
  }
}

```