# Libreria para PM - MPI

Estado: Sin empezar
ítem principal: Teorías PC (Teor%C3%ADas%20PC%202520675b4f6f80a7b401ea2c307583f0.md)

## Operaciones Send y Receive

![image.png](Libreria%20para%20PM%20-%20MPI%202c20-8ca8/image.png)

### Send bloqueante

Bloquea al enviador hasta que el valor que se quiere enviar, se guarda en algún lugar seguro, cosa de que si alguien quiera modificar la variable, ese valor enviado ya fue guardado

Ociosidad del proceso que quiera hacer el envío. Por la espera que genera el envío del mensaje.

Para reducir la ociosidad, podemos hacer:

- **Send/Recieve sin buffering:** 
No utiliza ningún espacio intermedio para guardar el mensaje, por lo que el envío solo puede completarse si el receptor ya ejecutó un *receive* correspondiente y está listo para copiar el dato directamente; de lo contrario, ambos procesos quedan bloqueados esperando al otro. Esto genera sincronización estricta entre emisor y receptor, dado que el mensaje “pasa de mano en mano” sin almacenarse en ningún lado. El beneficio es que no se necesita memoria adicional para buffers, pero lo negativo es que aumenta la probabilidad de ociosidad si alguno de los dos no llega a tiempo.
- **Send/Receive bloqueantes con buffering:**
    
    Utiliza en el medio un buffer en donde se copia el dato del mensaje envíado, así el proceso trasmisor puede seguir trabajando. Lo malo es que debemos usar un buffer justamente que se encuentre en algún lugar de la memoria (suelen estar en la máquina/memoria del proceso receptor). Se asemeja a PMA.
    

### Send/Recieve NO bloqueante

Para evitar overhead (ociosidad o manejo de buffer) se devuelve el control de la operación inmediatamente.

Requiere un posterior chequeo para asegurarse la finalización de la comunicación. 

Deja en manos del programador asegurar la semántica del SEND.

Hay dos posibilidades:

- **Send/Receive no bloqueantes sin buffering:** Hasta que el dato no haya sido recibido por el otro proceso (no hizo el receive) la comunicación real no se hace. Desde que se hace el send, y el otro proceso hace el receive, **el dato está inseguro.**
- **Send/Receive no bloqueantes con buffering:** El tiempo que el dato no estará seguro, será el tiempo que tarde el dato en llegar desde el send a copiarse en el buffer.

## MPI

![image.png](Libreria%20para%20PM%20-%20MPI%202c20-8ca8/image%201.png)

![image.png](Libreria%20para%20PM%20-%20MPI%202c20-8ca8/image%202.png)

Nosotros vamos a utilizar sólo el comunicador global.

### Adquisición de Información

![image.png](Libreria%20para%20PM%20-%20MPI%202c20-8ca8/image%203.png)

## Tipos de Datos para las comunicaciones

![image.png](Libreria%20para%20PM%20-%20MPI%202c20-8ca8/image%204.png)

MPI_PACKED son una especie de registros.

## Comunicación punto a punto

![image.png](Libreria%20para%20PM%20-%20MPI%202c20-8ca8/image%205.png)

### Comunicación bloqueante punto a punto

![image.png](Libreria%20para%20PM%20-%20MPI%202c20-8ca8/image%206.png)

**Parámetros MPI_Send:**

1.  Direc de memoria a donde está ubicado el dato que tenemos que enviar
2. Cantidad de elementos
3. Tipo de datos de los elementos
4. Rank proceso destino 
5. Etiqueta/tag para identificar mensajes (es un int)
6. Comunicador dentro del cual estamos haciendo el send.

**Parámetros MPI_Recv:**

1. Direc de memoria donde voy a dejar los datos recibidos
2. Cantidad máx de elementos de ese tipo que me pueden llegar
3. Tipo de datos de los elementos
4. Rank proceso origen
5. Etiqueta/tag para identificar mensajes (es un int)
6. Comunicador dentro del cual estamos haciendo el send.}
7. Estado, que permite saber

Rutina que espera hasta que haya un mensaje que coincida, se establece la comunicación, se reciben los datos y se sigue trabajando.

Puede ser que no sepa de qué proceso voy a recibir el mensaje. En este caso, en el rank proceso origen podemos poner un comodín (MPI_ANY_SOURCE) que indica que podemos recibir de cualquier proceso 

De la misma manera, podemos utilizar un comodín en la etiqueta, para recibir distintos pedidos (MPI_ANY_TAG)

- El estado de un Recv, es un registro donde se encuentra el rank del proceso que hizo el envío, y lo mismo para saber qué etiqueta tiene. El tercer campo es para saber si hubo error en la comunicación.
    - También existe el comando MPI_Get_count que devuelve la cantidad de elementos que fueron enviados de una comunicación. A la cual le mando como parámetro el estado de ese Recv, el tipo de dato que se envíaba, y devuelve la cantidad que fueron recibidos.

![image.png](Libreria%20para%20PM%20-%20MPI%202c20-8ca8/image%207.png)

![image.png](Libreria%20para%20PM%20-%20MPI%202c20-8ca8/image%208.png)

### **Comunicación no bloqueante punto a punto**

![image.png](Libreria%20para%20PM%20-%20MPI%202c20-8ca8/image%209.png)

El campo solicitud, podemos saber si la comunicación se terminó.

Se puede hacer un send bloqueante, y un receive no bloqueante, y viceversa.

![image.png](Libreria%20para%20PM%20-%20MPI%202c20-8ca8/image%2010.png)

Tiene sentido si en //Trabaja puedo hacer algo, pero acá no tiene sentido porque no hay nada para hacer en el medio, así que es mejor usar comunicación bloqueante.

Ejemplo de MPI_Test

![image.png](Libreria%20para%20PM%20-%20MPI%202c20-8ca8/image%2011.png)

Campo request que identifica si la comunicación terminó.

## Consulta de mensajes pendientes

Información de un mensaje antes de hacer el Recv(Origen, Cantidad de elementos, Tag).

![image.png](Libreria%20para%20PM%20-%20MPI%202c20-8ca8/image%2012.png)

Probe sirve concretamente por ejemplo, para saber la cantidad de elementos de memoria que se envían, para alocar la memoria necesaria y no desperdiciar de esta alocando más de lo necesario. 

En el caso del IProbe, chequea si hay algun mensaje pendiente que cumple esas características. Tiene los mismos parámetros que antes, y tiene un flag de salida true o false si existe el mensaje que cumple, y el estado indica lo de siempre.

### Comunicaciones Colectivas

![image.png](Libreria%20para%20PM%20-%20MPI%202c20-8ca8/image%2013.png)

Broadcast donde el punto de origen es el p0. Una comunicación tardará una unidad de tiempo. 

1. P0→P1
2. P0→P2, P1→P3
3. **Etc:**

![image.png](Libreria%20para%20PM%20-%20MPI%202c20-8ca8/image%2014.png)

![image.png](Libreria%20para%20PM%20-%20MPI%202c20-8ca8/image%2015.png)

![image.png](Libreria%20para%20PM%20-%20MPI%202c20-8ca8/image%2016.png)

![image.png](Libreria%20para%20PM%20-%20MPI%202c20-8ca8/image%2017.png)

![image.png](Libreria%20para%20PM%20-%20MPI%202c20-8ca8/image%2018.png)

# Minimizando los overheads de comunicación

![image.png](Libreria%20para%20PM%20-%20MPI%202c20-8ca8/image%2019.png)