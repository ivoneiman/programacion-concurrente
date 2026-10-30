# Ejemplos y Técnicas

Estado: Sin empezar
ítem principal: Monitores (Monitores%2026f0675b4f6f8063bc0de055da2a8664.md)

![image.png](Ejemplos%20y%20T%C3%A9cnicas/image.png)

Se mantiene un cierto orden, si hay varios procesos dormidos, se despierta al primero debido al signal. El orden no es total, porque puede ser que algun proceso nuevo le gane el acceso a quien fue despertado

### Passing the Conditions

Evita problemas de inanición, de acuerdo al orden en que fueron adquiriendo cuando fueron dormidos.

![image.png](Ejemplos%20y%20T%C3%A9cnicas/image%201.png)

No lo entendí

Ejemplo práctico:

![image.png](Ejemplos%20y%20T%C3%A9cnicas/image%202.png)

Así es en la práctica para evitar los conflictos de acceso cuando se despierta a un proceso y justo llega otro. Para controlar el acceso en orden al Cajero. *(No hay que re chequear la condición con un while)*

---

### Prioridades

**Imaginemos que ahora si llega una persona anciana tiene prioridad sobre el uso del cajero:**

Necesitamos una cola ordenada que además de tener la info del id, va a tener sobre la edad (pasada como parámetro en el procedimiento). Se define una cota para determinar si es persona mayor o no (si va delante de la cola o al final).

Para esto voy a necesitar variables de condición privadas para todos los procesos

![image.png](Ejemplos%20y%20T%C3%A9cnicas/image%203.png)

---

### Ejercicio 4

En un banco hay 3 empoleados. Y hay N clientes que deben ser atendidos por uno de ellos (cualquiera) de acuerdo al orden de llegada. Cuando uno de los empleados lo atiende el cliente le entrega los papeles y espera el resultado

(Un monitor por empleado)

![image.png](Ejemplos%20y%20T%C3%A9cnicas/image%204.png)

Ahora veamos la interacción entre el cliente y el empleado

```java
Process Cliente [id: 0..n-1]{
int idE;
text papel,res;

Banco.llegada(idE)
Escritorio[idE].atención(papel,res);
}

Process Empleado [id: 0..2]
{ text datos;
while true(){
	Banco.próximo(id);
	Escritorio[id].esperarDatos(datos);
	res - *resolver la solicitud en base a datos*
	Escritorio[id].enviarResultado(res);
	}
}
```

**En cada escritorio debemos tener una variable para que el cliente deje los datos, y otra donde van a estar los resultados para que el empleado se comunique.** 

- Hay que demorarse como cliente hasta que el empleado haya devuelto el resultado *vcCliente*. Y el empleado debe esperar los datos en otra variable condición *vcEmpleado*

**DEADLOCK:** Si el Cliente llega al escritorio antes que el Empleado, entonces el Empleado se duerme y nadie lo despierta porque el Cliente ya hizo el signal. Entonces debo usar una variable booleana *listo* que me indique si el cliente ya llegó, y sólo dormirse si *listo* es falso

![image.png](Ejemplos%20y%20T%C3%A9cnicas/image%205.png)

---

## Barrera

El delay debe hacerse una sola vez

![image.png](Ejemplos%20y%20T%C3%A9cnicas/image%206.png)

El delay podría ser en el monitor, cuando llega el último jugador. O bien usar otro proceso que simula el partido y se duerme hasta que todos llegan; luego hace el delay y despierta a todos para que se vayan.

Entonces:

![image.png](Ejemplos%20y%20T%C3%A9cnicas/image%207.png)

Si el primer if fuera <, el último proceso no se duerme, por lo tanto no jugaría el partido

Mi pregunta es: Si primero despierta al partido, no puede pasar que el partido avise que terminó antes de que este proceso se encole en el wait(espera)?

---

# Cliente / Servidor

![image.png](Ejemplos%20y%20T%C3%A9cnicas/image%208.png)

---

Pequeña modificación para el caso en que haya 2 o + servidores

El problema es que en el procedimiento Sig se genera competencia por el monitor luego de ser despertado en el HayPedido y provocando una interferencia.

Para solucionarlo es muy sencillo, hay que re chequear la condición

![image.png](Ejemplos%20y%20T%C3%A9cnicas/image%209.png)

No es busy waiting, solo se demora una pasada más, en caso de que otro proceso le robe el dato.

El detalle es que en ese caso, puede ser que las secuencias termines en desorden con respecto a su llegada, por lo tanto podríamos despertar al Cliente equivocado. Debemos tener variables cond privadas *(espera).*

**NO CAMBIA LA SOLUCIÓN EN CASO DE SER MÁS SERVIDORES**

![image.png](Ejemplos%20y%20T%C3%A9cnicas/image%2010.png)