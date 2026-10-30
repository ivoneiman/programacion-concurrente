# Paradigmas de Programacion Paralela

Estado: Sin empezar
ítem principal: Teorías PC (Teor%C3%ADas%20PC%202520675b4f6f80a7b401ea2c307583f0.md)

![image.png](Paradigmas%20de%20Programacion%20Paralela/image.png)

## Cliente / Servidor

Esquema dominante en las aplicaciones de procesamiento distribuido. Los servidores son procesos que esperan pedidos de servicios de múltiples clientes. Naturalmente unos y otros pueden ejecutarse en procesadores diferentes. 

### **Comunicación bidireccional.**

**Mecanismos de invocación variados (rendezvous, RPC, monitores).**

## Master / worker

**Proceso Master** que tiene todas las tareas, que entrega a los workers para luego recibir los resultados de éstos.

Posible cuello de botella, en caso de que haya slaves muy rápidos, o recibir muchos mensajes de muchos procesos worker que generen overhead (retardo importante).

## Pipeline

Resuelve el problema en pasos, y el stream de datos pasa entre los procesos, y cada uno realiza una tarea sobre él. Lo más natural es que sea un mapeo natural a un arreglo lineal de procesadores.

Hay que tener en cuenta que puede haber extensiones:

- Procesadores especializados no iguales
- Más de un procesador para una tarea determinada
- El flujo puede no ser una línea simple (ejemplo: ensamble de autos con varias líneas que son combinadas) → procesamiento sistólico.

## Dividir y conquistar

Paralelismo recursivo. Donde el problema general puede descomponerse en procesos recursivos que trabajan sobre partes del conjunto total de datos.

Se genera un árbol, donde cada nodo sería un proceso, que recibe datos, si puede los procesa, sino, crea un nro de hijos y les distribuye los datos.

## SPMD

El programador genera un programa único que ejecuta cada nodo sobre una porción del dominio de datos. 

Dos fases: 1) elección de la distribuición de datos y 2) generación del programa paralelo

Suele implicar **paralelismo iterativo.** donde un programa consta de un cojunto de procesos los cuales tiene 1 o más loops. Cada proceso es un programa iterativo.

Generalmente, el dominio de datos se divide entre los procesos siguiendo diferentes patrones.

multiplicación de matrices

![image.png](Paradigmas%20de%20Programacion%20Paralela/image%201.png)

![image.png](Paradigmas%20de%20Programacion%20Paralela/image%202.png)

![image.png](Paradigmas%20de%20Programacion%20Paralela/image%203.png)

Para mejorar el speedup, habría que asignar menos filas al procesador lento y más a los rápidos (balanceo de carga heterogéneo).

> **el tiempo paralelo total siempre está determinado por el procesador que termina último**, es decir, *el más lento*. Aunque 7 procesadores terminen antes, **el resultado final de la matriz C no está completo hasta que el más lento termina sus filas**.
>