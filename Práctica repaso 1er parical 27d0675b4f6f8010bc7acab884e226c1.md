# Práctica repaso 1er parical

Estado: Sin empezar
ítem principal: Programación Concurrente (Programaci%C3%B3n%20Concurrente%202520675b4f6f801eaf70d390ea0e041e.md)

![image.png](Pr%C3%A1ctica%20repaso%201er%20parical/image.png)

```c
sem mutex,esperar[P];
cola C;
bool libre = true;
int aux;
process Persona [id: 0..N-1]{
	if (!libre) { // No debería haber interferencia al leer no?
		P(mutex)
			push(C(id));
		V(mutex)
		P(esperar[id]);
	}
	
	UsarTerminal();
	
	P(mutex)
	if (empty(C)) libre = true;
	else {
		pop(C(aux));
		V(esperar[aux]);
	}
	V(mutex)
}

--------------------------------------------------------------
sem mutex,esperar[P];
cola C;
int idAux;
cola Term;// índices de terminales libres
sem termDisp[T] = ([T] 1);
// terminales libres para usar

process Persona [id: 0..N-1]{
		int posTerm;
		
		P(mutex);
		if(empty(Term)) {
				push(C(id));
			V(mutex);
			P(esperar[id]);
		}
		
		//obtengo la pos libre
		P(mutex)
			posTerm = pop(Term(t));
		V(mutex)
		//uso la terminal
		P(termDisp[posTerm]); 
			UsarTerminal(posTerm);
		V(termDisp[posTerm]);
		//libero la terminal
		P(mutex)
			push(Term(posTerm));
			//despierto al siguiente, en caso de que haya
			if (!empty(C)){
				pop(C(idAux));
				V(esperar[idAux);
			}
		V(mutex)

}
```

![image.png](Pr%C3%A1ctica%20repaso%201er%20parical/image%201.png)

```c
cola CT; //Transacciones
sem mutexCola = 1,mutexVector[10] = ([10] 1);
int vectorContador[10] = ([10] 0);
sem barrera = 0;

Process Worker [0..6]{
	text transaccion;
	P(mutexCola)
	while (!empty(CT)){ //mientras haya transacciones
		transaccion = pop(CT()); //extraigo una t
		V(mutexCola);
		int res += Validar(transaccion);
		P(mutexVector[res]) // Tapo en la pos del vectorContador que uso
			vectorContador[res]++; //Aumento
		V(mutexVector[res]) // Libero
		P(mutexCola) 
	}
	V(mutexCola)
	
	P(mutex)
		terminaron++;
		if ( terminaron == 7 ){
			for i: 0..9{
				write (i,' ', mutexVector[i]);
			}
			for i: 0..6{
				V(barrera)
			}
			V(mutex)
		}
		else {
			V(mutex)
				P(barrera) //hasta q no llega el ultimo se cuelgan 6 acá
		}
}
```

![image.png](Pr%C3%A1ctica%20repaso%201er%20parical/image%202.png)

```c
sem mutex=1,mutexMaquina=1,lleno=100, vacio=0,esperar[U]=([U] 0); 
cola C
bool libre = true;
int aux, cantLatas=100;

Process Usuario [id: 0..U-1]{
	P(mutex)
		if (not libre) {
			push(C(id));
			V(mutex);
			P(esperar[id]);
		} // Si me despiertan libre sigue siendo = false
		else {
			libre = false
			V(mutex)
		}
		
	P(lleno) // Antes de acceder a máquina, para no generar deadlock
	P(mutexMaquina) //Por si hay repositor
		cantLatas--;
		if (cantLatas == 0) V(vacio);
	V(mutexMaquina)
	
		
	P(mutex)
	if (not empty(C)){
		pop(C(aux))
		V(esperar[aux])
	}
	else {
		libre = true;
	}
		V(mutex)
}

Process Repositor {
	P(vacio) //Primero espero por vacio, para no generar deadlock.
	P(mutexMaquina) //Bloquea la máquina, no la cola
		for i: 1 to 100 {
			V(lleno);
			cantLatas++;
		}
	V(mutexMaquina) //Libera la máquina
}

```

![image.png](Pr%C3%A1ctica%20repaso%201er%20parical/image%203.png)

```c
Process Persona [id: 0..N-1] {
    bool prioridad = ...;   // true si es anciano/embarazada
    Autoridad.Pasar(id, prioridad);
    Votar();
    Autoridad.Salir();
}

Monitor Autoridad {
    bool libre = true;
    cond espera[N];
    colaOrdenada fila;   // ordena por (prioridad, llegada)

    Procedure Pasar(idP: in int, prio: in bool) {
        if (not libre) {
            insertar(fila, (idP, prio));     // prio adelante, FIFO dentro
            wait(espera[idP]);
        } else {
            libre = false;
        }
    }

    Procedure Salir() {
        if (not empty(fila)) {
            int idAux;
            sacar(fila, idAux);            // elige por prioridad
            signal(espera[idAux]);         // baton al próximo
        } else {
            libre = true;
        }
    }
}

```

<aside>
〰️

2) Resolver el siguiente problema. En una empresa trabajan 20 vendedores ambulantes que forman 5 equipos de 4 personas cada uno (cada vendedor conoce previamente a qué equipo pertenece). Cada equipo se encarga de vender un producto diferente. Las personas de un equipo se deben juntar antes de comenzar a trabajar. Luego cada integrante del equipo trabaja independientemente del resto vendiendo ejemplares del producto correspondiente. Al terminar cada integrante del grupo debe conocer la cantidad de ejemplares vendidos por el grupo. Nota: maximizar la concurrencia.

</aside>

```c
Monitor Manager{

int listos[5] = 0, ejemplaresVendidos[5] = 0, vendieron[5] = random(50);
cond esperarCompañeros[5] = ([5] 0); // para iniciar y para terminar

	Procedure Listo(equipo: in int){
		listos[equipo]++;
		if (listos[equipo] == 4){
			for i: 0 to 4{
				signal(esperarCompañeros[equipo])
			}
		}
		else wait(esperarCompañeros[equipo])
	}
	
	Procedure Finalizar(equipo,cantVendidos: in int; total: out int){
		ejemplaresVendidos[equipo] += cantVendidos;
		vendieron[5]++;
		if (vendieron[5] == 4){
			total = ejemplaresVendidos[equipo];
			for i:0..4{
				signal(esperarCompañeros[equipo]);
			}
		}
		else {
			wait(esperarCompañeros[equipo]);
		}
	}

}

Process Vendedor[id: 0..19]{
	int equipo = ...; //0..4
	int cantEjemplaresAVender = E; //Número distinto por vendedor
	int cantVendidos = 0;

	Manager.Listo(equipo);
	
	while (cantEjemplaresAVender > 0){
		Vender();
		cantEjemplaresAVender--;
		cantVendidos++;
	}
	int total;
	Manager.Finalizar(equipo,cantVendidos,total)
}

```

![image.png](Pr%C3%A1ctica%20repaso%201er%20parical/image%204.png)

```c
Process Escalador [id:0..29]{
	Paso.Pasar(id)
		Pasando()
	Paso.Liberar()
}

Monitor Puente
	cola C;
	bool libre = true;
	cond esperar[30]

	Procedure Pasar(id: in int){
		if (not libre) {
			push(C(id))
			wait(esperar[id])
		} 
		else libre = false
	}

	Procedure Liberar(){
		if (not empty(C)){
			int idAux;
			pop(C(idAux));
			signal(esperar[idAux]);	
		}	
		else libre = true
	}
End Monitor;
```

![image.png](Pr%C3%A1ctica%20repaso%201er%20parical/image%205.png)

```c
Cola C;
sem mutexCola;
int ticket = 50, pasajeros = 0;

Process Cliente [id: 0..C-1]{
	
}

Process Vendedor {
	while (true){
		P(esperando)
		P(mutexCola)
			pop(C(idC));
		V(mutexCola)
		if(ticket > 0){
			
		}
			
	}
}
```

![image.png](Pr%C3%A1ctica%20repaso%201er%20parical/image%206.png)

![image.png](Pr%C3%A1ctica%20repaso%201er%20parical/image%207.png)

![image.png](Pr%C3%A1ctica%20repaso%201er%20parical/image%208.png)

![image.png](Pr%C3%A1ctica%20repaso%201er%20parical/image%209.png)

![image.png](Pr%C3%A1ctica%20repaso%201er%20parical/image%2010.png)

![image.png](Pr%C3%A1ctica%20repaso%201er%20parical/image%2011.png)

![image.png](Pr%C3%A1ctica%20repaso%201er%20parical/image%2012.png)

Es necesaria la ultima interacción, para asegurarse de que el paciente sale, y no le roben el listo

![image.png](Pr%C3%A1ctica%20repaso%201er%20parical/image%2013.png)

![image.png](Pr%C3%A1ctica%20repaso%201er%20parical/image%2014.png)

La enfermera espera que el paciente le avise que terminó

![image.png](Pr%C3%A1ctica%20repaso%201er%20parical/image%2015.png)

![image.png](Pr%C3%A1ctica%20repaso%201er%20parical/image%2016.png)

![image.png](Pr%C3%A1ctica%20repaso%201er%20parical/image%2017.png)

![image.png](Pr%C3%A1ctica%20repaso%201er%20parical/image%2018.png)