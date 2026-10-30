# Libreria para PM - MPI

Estado: Sin empezar
ítem principal: Teorías PC (Teor%C3%ADas%20PC%202520675b4f6f80a7b401ea2c307583f0.md)

![image.png](Libreria%20para%20PM%20-%20MPI/image.png)

**Sin buffering:** esperar a que el dato haya sido por el otro proceso

**Con buffering: r**educe ese tiempo osocio, utilizando un buffer mediático, que graba el dato que se envía, para que el receptor lo busque ahí dentro. Lo malo es tener que usar el buffer, que tiene que estar en algún lugar de la memoria. Probablemente los dos procesos estén en máquinas separadas. *Entonces hay que definir dónde va a estar este buffer. Lo más probable es que esté puesto en el proceso receptor.*