# Memoria distribuida

Estado: Sin empezar
ítem principal: Teorías PC (Teor%C3%ADas%20PC%202520675b4f6f80a7b401ea2c307583f0.md)

# Clase 6

## Programación concurrente en memoria distribuida

### Conceptos generales

- **Arquitecturas de memoria distribuida** → procesadores + memoria local + red de comunicaciones + mecanismo de comunicación / sincronización
    - **Intercambio de mensajes.**
- **Programa distribuido:** programa concurrente comunicado por mensajes. Supone la ejecución sobre una arquitectura de memoria distribuida, aunque puedan ejecutarse sobre una de memoria compartida
- **Primitivas de pasaje de mensajes:** interfaz con el sistema de comunicaciones ⇒ semáforos + datos + sincronización
- Los procesos SOLO **comparten canales** (físicos o lógicos). Variantes para los canales:
    - Según cuantos o quiénes puedan enviar/recibir información por los canales se utiliza
        - **Mailbox:** Canales compartidos. Todos pueden enviar y recibir a través de esos canales
        - **input port:** Por estos canales un único proceso puede ser receptor, pero cualquiera puede enviar. Cliente-Servidor (que recibe muchos pedidos)
        - **link - Punto a punto:** Un único proceso puede enviar información por el canal y un único proceso puede recibirla.
    - Según si los datos van en un sentido o ambos: **Uni o bidireccionales**
    - Según si se requiere que ambos procesos lleguen a un punto de comunicación o no: **sincrónicos o asincrónicos**

---

### Características

- Los canales (físicos o lógicos) son lo único que comparten los procesos
    - Variables locales a un proceso (”cuidador”)
    - La EM no requiere mecanismo especial
    - Los procesos interactúan comunicándose
    - Accedidos por primitivas de envío y recepción
- **Mecanismos para el Procesamiento Distribuido:**
    - Pasaje de Mensajes Asincrónicos (PMA)
        - Canales MailBox, comunicación unidireccional y asincrónica
    - Pasaje de Mensajes Sincrónico (PMS)
        - Canales link, comunicación unidireccional y asincrónica
    - Llamado a Procedimientos Remotos (RPC) SÓLO TEÓRICO
        - Canales link, comunicación bidireccional y sincrónica
    - Rendezvous
        - Igual que RPC
- La sincronización de la comunicación interproceso depende del patrón de interacción depende del patrón de interacción
    - **Productores y consumidores (Filtros o pipes):**
        - Procesos (flitros) que reciben información de otros procesos, la procesan, y la pasan a otros procesos. El ejemplo clásico es las pipelines de los procesadores. Otro es aplicar varios filtros a una imagen
    - **Clientes y servidores:**
        - Varios procesos cliente que necesitan que uno o más servidores resuelvan algo. Los procesos servidores lo único que hacen es recibir esas solicitudes y resolverlas. Los clientes deben esperar al resultado
    - Pares (peers) que interactúan:
        - Varios procesos con las mismas características (quizás iguales) trabajan sobre el mismo problema. Todos tienen la misma categoría, y cada tanto van interactuando entre sí para alguna necesidad del problema.
    
    → Cada mecanismo es más adecuado para determinados patrones.
    

## Relación entre mecanismos de sincronización

![image.png](Memoria%20distribuida/image.png)

La forma en la que trabajan los semáforos se bifurca en 2 grandes ramas: Por un lado continúa con lo que es memoria compartida, intentando usar una herramienta con mayor grado de abstracción y que permite resolver implícitamente la EM, los monitores.

Y por otro lado, también hay una relación entre semáforos y el pasaje de mensajes, sobre todo el PM asincrónico, que tiene una forma de sincronización similar a la que tienen los semáforos → la espera de un mensaje en un canal vs. el P y envío de un mensaje vs el V.

Luego, monitores se relaciona con RPC, porque es como si fuera distintos monitores distribuidos en distintas máquinas.

También PM, sobre todo el sincrónico, se relaciona mucho con Rendezvous ya que son muy parecidos pero la principal diferencia es que en Rendezvous la comunicación es bidireccional.

Finalmente RPC y Rendezvous se relacionan entre ellos porque la comunicación es bidireccional.

---

## Pasaje de Mensajes Asincrónicos (PMA)

Canales en OMA: colas de mensajes que han sino enviados y aun no han sido recibidos. Siempre se utilizan para enviar msj del mismo tipo (ya sea entero, doble, etc).

- Se declaran en el programa, con palabres clave
    - chan entrada (char);
    - chan acceso_disco (INT cilindro, INT bloque, INT cant, CHAR* buffer);
    - chan resultado[n] (INT)
- En PMA los canales son de tipo mailbox, comunicación unidireccional y asincrónica. Para este canal tenemos operaciones:
    - Send: Un proceso agrega un mensaje al final de la cola (”ilimitada”) de un canal ejecutando un send, que no bloquea al emisor: send ch(expr1, .., exprn)
    - Receive: un proceso recibe un mensaje desde un canal con recieve, que demora (”bloquea”) al receptor hasta que en el canal haya al menos un mensaje; luego toma el primero y lo almacena en var locales: recieve ch(var1, ..,varn).
        - Receive es una primitiva bloqueante (duerme al proceso), ya que produce un delay. Semántica: el proceso NO hace nada hasta recibir un mensaje en la cola correspondiente al canal. No se queda chequeando continuamente un mensaje en el canal. NO es necesario hacer polling.

### Características de los canales

- Acceso a los contenidos de cada canal: atómico y respeta orden FIFO
- En principio los canales son ilimitados, aunque en las implementaciones reales tendrán un tamaño de buffer asignado
- Se supone que los mensajes NO se pierden ni se modifican y que todo msj enviado en algún momento puede ser “leído”
- empty(ch) → determina si la cola de un canal está vacía. Útil cuando el proceso puede puede hacer trabajo productivo mientras espera un mensaje, <u>**pero debe usarse con cuidado por las posibles interferencias:**</u>
    - Podría ser false, y no haber más mensajes cuando sigue ejecutando (si no es el único en recibir por ese canal)
    - Podría ser true, y sin embargo existir un mensaje al momento de que el proceso reanuda la ejecución
- Los canales pueden ser compartidos por los procesos, por lo tanto se declaran como globales a ellos. Según la forma en que se usan podrían ser:
    - Cualquier proceso puede enviar o recibir por alguno de los canales declarados. Estos se denominan *mailboxes*.
    - En algunos casos un canal tiene un solo receptor y muchos emisores (*input port*)
    - Si el canal tiene un único emisor y un único receptor se lo denomina *link*: provee un “camino” entre el emisor y su receptor.

## Ejemplo

```c
// Canales de comunicación: ambos son SINCRÓNICOS (tipo mensaje).
// - 'entrada' transporta caracteres individuales.
// - 'salida' transporta líneas completas de texto (vectores de char de CantMax).
chan entrada(char), salida(char[CantMax]);

// -------------------------------------------------------------
// Proceso Proceso_1 (Productor de caracteres)
// -------------------------------------------------------------
// Lee caracteres desde el teclado (uno por uno) y los envía a través
// del canal 'entrada' hacia el proceso intermedio 'Carac_a_Linea'.
//
Process Proceso_1 {
    char a;
    WHILE (true) {
        leer_caracter_por_teclado(a);   // simula lectura del teclado
        send entrada(a);                // envía el carácter leído
    }
}

// -------------------------------------------------------------
// Proceso Carac_a_Linea
// -------------------------------------------------------------
// Recibe caracteres por el canal 'entrada' (provenientes del Proceso_1),
// los va almacenando uno a uno en el vector 'linea'.
// Cada vez que detecta el fin de línea (carácter CR) o llega al tamaño máximo,
// forma una línea completa (terminada en EOL) y la envía por el canal 'salida'
// hacia el Proceso_2.
//
// En otras palabras: convierte un flujo de caracteres en un flujo de líneas.
//
Process Carac_a_Linea {
    char linea[CantMax];
    int i = 0;

    WHILE (true) {
        // Espera un carácter y lo almacena en la primera posición de la línea
        receive entrada(linea[i]);

        // Mientras no sea 'enter' (CR) y haya espacio en el vector,
        // sigue recibiendo y agregando caracteres a la línea.
        WHILE (linea[i] != CR and i < CantMax) {
            i = i + 1;
            receive entrada(linea[i]);
        }

        // Cuando detecta fin de línea (CR), agrega el carácter EOL (terminador)
        // para marcar el fin de la cadena y envía la línea completa.
        linea[i] := EOL;
        send salida(linea);

        // Reinicia el índice para construir una nueva línea.
        i := 0;
    }
}

// -------------------------------------------------------------
// Proceso Proceso_2 (Consumidor de líneas)
// -------------------------------------------------------------
// Recibe líneas completas desde el canal 'salida' (enviadas por Carac_a_Linea)
// y las imprime en pantalla.
//
Process Proceso_2 {
    char res[CantMax];
    WHILE (true) {
        receive salida(res);            // recibe una línea completa
        imprimir_en_pantalla(res);      // la muestra por pantalla
    }
}

```

---

## Ejemplos con patrones de interacción

### Productores y consumidores (filtro)

### **Red de ordenación un conjunto de números desordenados:**

- **Filtro:** proceso que recibe mensajes de uno o más canales de entrada, los trabaja y envía mensajes a uno o más canales de salida. La salida de un filtro es función de su estado inicial y de los valores recibidos
- Esta función del filtro puede especificarse por un predicado que relacione con los valores de los mensajes de salida.
- **Problema:** ordenar una lista de N números de modo ascendente. Podemos pensar en un filtro Sort con un canal de entrada (N números desordenados) y un canal de salida (N números ordenados).

![image.png](Memoria%20distribuida/image%201.png)

¿Cómo sabemos que recibimos todos los números que hay que ordenar?

- conoce N.
- envía N como el primer elemento a recibir por el canal entrada
- cierra la lista de N números con un valor especial o “centinela” (Cuando no hay más números se manda ese valor como fin de secuencia. Opción que vamos a manejar nosotros)

Armar una red de pequeños procesos filtros que interactuan entre ellos para armar el vector ordenado. **merge network**

![image.png](Memoria%20distribuida/image%202.png)

Para que esas dos secunecias que llegan ordenadas, tenemos que asociarlas con otro proceso más. Dividiendo la mitad de los datos, hasta que en un momento, vamos a tener procesos merge que por cada uno de los dos canales de entrada entrará un elemento.

![image.png](Memoria%20distribuida/image%203.png)

![image.png](Memoria%20distribuida/image%204.png)

- n-1 procesos; el ancho de la red es log2 n
- Canales de entrada y salida compartidos
- Puede programarse usando:
    - Static naming (arreglo global de canales, y cada instancia de Merge recibe desde 2 elementos del arreglo y envía a otro ⇒ embeber el árbol del arreglo).
    - Dynamic naming (canales globales, parametrizar los procesos, y darle a cada proceso 3 canales al clearlo; todos los Merge son indénticos, pero se necesita un coordinador).
- Los filtros podemos conectarlos de distintas maneras. Sólo se necesita que la salida de uno cumpla las suposiciones de entrada del otro ⇒ pueden reemplazarse si se mantienen los comportamientos de e/s.
    - En este caso necesitamos respetar los datos se envien ordenados y que se envíe un centinela

### Clientes y Servidores

### Monitores Activos

- Un Servidor es un proceso que maneja pedidos (requerimientos) de otros procesos clientes. Un proceso Cliente que envía un mensaje al canal de requerimientos general, luego recibe el resultado desde un canal de respuesta propio.
- En un sistema distribuido, lo natural es que el proceso Servidor resida en un procesador físico y M procesos Cliente residan en otros procesadores (N≤M).
- Monitor → Manejador de recurso. Encapsula variables permanentes que registral el estado, y provee un conjunto de procedures. Los simulamos, usando procesos servidores y PM, como procesos activos en lugar de como conjuntos pasivos de procedures.

![image.png](Memoria%20distribuida/image%205.png)

![image.png](Memoria%20distribuida/image%206.png)

![image.png](Memoria%20distribuida/image%207.png)

En caso de que haya multiples operaciones para realizar, se define un arreglo de tipos, para saber cual se necesita para el tipo de operacion en el caso de que necesite un cliente realizar multiples operaciones

![image.png](Memoria%20distribuida/image%208.png)

![image.png](Memoria%20distribuida/image%209.png)

### Ahora, simulación de condición por PMA

Los clientes hacen lo mismo.

El proceso servidor, tendrá que administrar el uso de recursos. El administrador indica el recurso a usar. Estas unidades a utilizar, como impresoras libres, se mantienen en un conjunto llamado *set* con ciertas operaciones que permiten insertar y remover recursos de ellas. Y vamos a tener algo que nos indique cuantas son las impresoras en este caso que estarían disponibles en este momento. Eso indica si hay que dormir al proceso que solicite en caso de que no haya impresoras.

![image.png](Memoria%20distribuida/image%2010.png)

Este código es un passing the baton. En la práctica no vamos a usar el empty porque esto no es un monitor jaja

Para hacerlo en c

```c
type clase_op = enum(adquirir,liberar);
chan request(int idCliente, claseOp oper, int idUnidad);
chan respuesta[n] (int id_unidad);

Process Administrador_Recurso{
	int disponible = MAXUNIDADES;
	set unidades = valor inicial disponible;
	queue pendientes; //simula var cond
	while (true){
		recieve request (IdCliente, oper, id_unidad);
		if (oper == adquirir){
			if (disponible > 0){
			disponible = disponible - 1;
			remove (unidades, id_unidad);
				send respuesta[IdCliente](id_unidad);
			}
			else push(pendientes, IdCliente);	
		}
		else{ // oper == liberar
			if empty(pendientes){
				disponible = disponible + 1;
				insert(unidades, id_unidad);
			}
			else{
				pop(pendientes, IdCliente);
				send respuesta[IdCliente](id_unidad);
			}
		}
	}
}

Process Cliente [i= 1 to n]{
	int id_unidad;
	send request(i,adquirir,0); //envía cero porque no se necesita el param
	recieve respuesta[i](id_unidad);
		//Usa la unidad
	send request(i,liberar,id_unidad);
}
```

![image.png](Memoria%20distribuida/image%2011.png)

### Sentencias de alternativa múltiple

Resolución del mismo problema con sentencias de alternativa múltiple. Ventajas y desventajas

Tiene sentido usar este tipo de solución cuando el servidor puede recibir distintos tipos de pedidos, y en algunos puede atender cualquier tipo de pedidos y en otro momento, algún tipo en particular. Si no tiene más de 2 o 3 tipos de pedidos distintos, conviene la solución anterior.

El servidor recibe los pedidos si está en condiciones de atenderlas, si no, las guarda en el canal de comunicación. Necesitamos un canal por cada pedido diferente: Uno para la solicitud y otro para indicar la liberación de esta. 

Para esto necesitamos el if de múltiples alternativas.

```c
chan pedido (int idCliente)
chan liberar (int idUnidad)
chan respuesta[n] (int idUnidad) //Servidor indica cual usar

Process Cliente[i = 1 to n]{
	int id_unidad;
	send pedido(i); //diferencia de canales, ya no es el mismo
	recieve respuesta[i](id_unidad);
	//uso la unidad
	send liberar(id_unidad);
}

Process Admin_Recurso{
	int disponible = MAXUNIDADES;
	set unidades = valor inicial disponible;
	int id_unidad, idCliente;
	while(true){
		if ( (not empty(pedido) and (disponible >0) )->
			recieve pedido(idCliente);
			disponible = disponible - 1;
			remove(unidades,id_unidad);
			send respuesta[idCliente](id_unidad);
		// chequeo que haya algo en el canal liberar para hacerlo
		(not empty(liberar)) -> 
			recieve liberar(id_unidad)
			disponible = disponible + 1
			insert(unidades,id_unidad) 
		} //if
	} //while
} //process Administrador_Recurso

```

Es como si en vez de tener dos operaciones, usas el if con sentencias de alternativa múltiple. **Siempre hay que poner como parte de las opciones si el canal no está vacío.** Si no preguntamos, podría ser que uno se quede bloqueado, intentando recibir un pedido de solicitud de un cliente, y resulta que todavía no había nada pendiente. Y mientras tanto un montón de clientes querrían liberar cuando estoy dormido.

Qué pasaría si no hay pedidos ni para pedido ni para pendiente. En este caso ambas alternativas son falsas, entonces sale del if y vuelve a entrar por el loop infinito. En ese caso estaríamos haciendo busy waiting, pero como es pasaje de mensaje asincrónico cuando otro tipo de solución sea demasiado compleja.

![image.png](Memoria%20distribuida/image%2012.png)

Cualquier docente puede atender de entrada a un alumno con el cual se quedará interactuando internamente por su canal propio del alumno↔docente privado. Alumno(cliente) Docente(Servidor)

Para esto tenemos el canal atención global y los rta_atención, consulta y respuesta, privados.

![image.png](Memoria%20distribuida/image%2013.png)

La interacción termina cuando el alumno indica que no hay más consultas, por ejemplo, usando un texto centinela, que libere al docente para volver a esperar alumnos que tengan consultas.

```c
chan atención (int);
chan consulta[D](string);
chan rta_atencion[A](int); //indicar id docente
chan respuesta[A](string); //respuesta a las consultas del A

Process Alumno [a = 1 to A]{
	int idDocente;
	string preg,res;
	
	send atencion(a);
	receive rta_atencion[a](idDocente);
	while(tenga consultas para hacer){
		send consulta[idDocente](preg);
		receive respuesta[a](res);
	}
	send consulta [idDocente]('FIN')
}

Process Docente [d = 1 to D]{
	string preg,res;
	int idAlumno;
	bool seguir = false;
	
	while(true){
		receive atencion(idAlumno);
		send rta_atencion[idAlumno](d);
		seguir = true;
		while (seguir){
			receive consulta[d](preg);
			if (preg == 'FIN') seguir = false;
			else{
				res = resolver la pregunta (preg);
				send respuesta[idAlumno](res);
			}
		}
	}
}
```

- Este ejemplo de interacción se denomina **continuidad conversacional** (desde la solicitud de atención hasta la última consulta).
- **atención** es un canal compartido por el que cualquier *Docente* puede recibir. Si cada canal puede tener un solo receptor, se necesita otro proceso intermedio. Recibe por un único canal todos los pedidos, y todos los pedidos de los docentes, actuando como intermediario.

### Tercer patrón de interacción - Intercambio de Valores

Cada proceso tiene un dato local V y los N procesos deben saber cuál es el menor y cuál es el mayor de los valores

Ejemplo donde los procesadores están conectados por tres modelos de arquitectura: *centralizado, simétrico* y en *anillo circular*

Cada proceso le manda el valor al proceso central, quien calcula los N datos ordenándolos y reenvía la información del mayor y menor a todos los procesos

2*N-1 Mensajes de totalidad

```c
chan valores(int), resultados[n-1](int minimo, int maximo);
Process P[i= 1 to n-1]{
	int v; int minimo,maximo;
	send valores(v);
	// i-1 porque el vector va de 0 a n-2, porque el faltante es el operador
	receive resultados[i-1](minimo,maximo);
} 

Process P[0]{
	int v; int nuevo, minimo = v, maximo =v;
	for [i=1 to n-1]{
		receive valores(nuevo);
		if (nuevo <minimo) minimo = nuevo;
		if nuevo>maximo) maximo = nuevo;
	}
	// No entiendo por que no hacer de 0 a n-2
	for [i=1 to n-1]
		send resultados[i-1](minimo, maximo);
}
```

Si sólo se utiliza para una sola iteración podemos tener un único canal de resultados. Si los procesos trabajan infinitamente no. Porque podría ser que un proceso, el cual envío su valor, esperó a que terminó la iteración y al recibir su resultado, siendo más rápido, genere un nuevo valor y recibe el resultado que espera de la segunda iteración. Pero como hay msj pendientes de la primer iteración que por ahí otros procesos no alcanzaron a verlo, toma uno de ellos que no corresponde a él. En este caso si o si usamos el arreglo

### Solución Simétrica

- En esta arquitectura “full conected” hay un canal entre cada par de procesos y todos ellos ejecutan el mismo algoritmo
- Cada proceso trasmite su dato local V a los N-1 restantes procesos. Luego recibe y procesa los N-1 datos que le faltan, de modo que en paralelo toda la arquitectura está calculando el mínimo y el máximo y toda la arquitectura tiene acceso a los N datos.
- Ejmplo de solución SPMD: cada proceso ejecuta el mismo programa pero trabaja sobre datos distintos. La desventaja es que tenemos mucha cantidad de mensajes, ahora es N*(N-1).
- Si disponemos de una primitiva de broadcast, serán nuevamente N mensajes. Pero no va a ser el caso de la práctica

```c
chan valores[n](int);
Process P(i=0 to n-1]{
	for [k=0 to n-1 st k <> i]
		send valores
	for [k=0 to n-1 st k <> i]{
		receive valores[i](nuevo);
		if (nuevo < minimo) minimo = nuevo;
		if (nuevo > maximo) maximo = nuevo;
	}
}
```

En este caso no se puede usar un canal único porque se mezclarían todos los valores y probablemente uno de los datos que envío es el que se recibe nuevamente

### Anillo

![image.png](Memoria%20distribuida/image%2014.png)

El proceso 0 es quien arranca y hace algo diferente, se encarga de mandar su mínimo y máximo al proceso 1, y espera a que por su canal (0) alguien le envíe el mínimo y máximo final. Y recién ahí es el resultado final, y se lo manda al proceso 1 para que él sepa que el res final es ese

Si es el proceso n-1 ahí termina, y para eso me sirve que el proceso 1 sepa cuándo es el final

Desventajas:

- Ya no es simétrico
- Cantidad de mensajes
- 

# PMS

- Los canales son de tipo link o punto a punto (1 emisor 1 receptor).
- La otra diferencia es que el send es sincrónico, ya no se deja en una cola de mensajes y se sigue trabajando, si no que, es bloqueante. Lo llamaremos por ahora sync_send en teoría
    - El trasmisor queda esperando que el mensaje sea recibido por el receptor (genera una demora)
    - La cola de mensajes asociada con un send sobre un canal se reduce a 1 mensaje ⇒ menos memoria
    - Naturalmente el grado de concurrencia se reduce respecto de la sincronización por PMA porque los emisores se bloquean.
    - Las posibilidades de deadlock son mayores en comunicación sincrónica

![image.png](Memoria%20distribuida/image%2015.png)

### Caso 1: Productor y consumidor tardan lo mismo en producir que consumirlo

Imaginemos que tardan un segundo cada uno, en este caso, se ejecutan paso a paso, y no se pierde tiempo entre un envío y otro. N segundos + 1 segundo lo que tarda el prod y el consumidor no hacía nada, el programa estaría terminado.

En asincrónico en este caso sería lo mismo, porque el **productor no tiene retardo.** Por eso no hay diferencia en ambos

### Caso 2: Productor es más rápido que lo que se tarda en consumir

Productor tarda 1 segundo y consumidor 10 en consumir. Habrá retardo, ya que el consumiudor estará 10 seg trabajando con ese elemento, y durante ese tiempo el productor ya generó y queda bloqueado esperando a que se pueda recibir el msj. *(La aplicación realmente no se demora, porque el trabajo se marca por el consumidor ⇒ 10s*N + 1s producir el 1er elem)*

La diferencia es que en el asincrónico esto no sucede, ya que el send no es bloqueante. *(La aplicación realmente no se demora, porque el trabajo se marca por el consumidor ⇒ 10s*N + 1s producir el 1er elem)*

### Caso 3: Productor es más lento que consumidor

Productor tarda 10 segundos y se consume en 1 segundo. Entonces, el consumidor queda demorado 9 segundos esperando el próximo elemento. En este caso quien se bloquea es el Consumidor, y el Productor marcará el ritmo

Pasa lo mismo en el asincrónico, porque el receive es bloqueante y funcionan igual

---

Si los cálculos del productor se realizan mucho más rápido que los del consumidor en las primeras n/2 operaciones, y luego se realizan mucho más lento en las otras, 

Con PMS los pares send/receive se completarán asumiendo la demora del proceso que más tiempo consuma. Si la relación de tiempo fuera 10 a 1 significaría multiplicar por 10 los tiempos totales

Con PMA, al principio el productor es más rápido y sus mensajes se encolan. Luego el consumidr es más rápido y “descuenta” tiempo consumiendo la cola de mensajes.

> *Acá si se refleja que al usar sincrónico se demora con diferencia y hay Mayor concurrencia en PMA. Para evitar esto, y lograr el mismo efecto con PMS se debe interponer un proceso “buffer” intermedio, para simular el guardado de los mensajes.*
> 

Lo mismo pasa si hubiera más de un prod o cons. Necesitamos simular un canal compartido con un intermediario buffer, donde cualquier prod envía los msj y este los recibe encolándolos, para que los consumidores le pidan elementos generados de esa cola.

![image.png](Memoria%20distribuida/image%2016.png)

Si dos procesos intentan intercambiar valores, y ambos hacen un send primero, se bloquean generando deadlock