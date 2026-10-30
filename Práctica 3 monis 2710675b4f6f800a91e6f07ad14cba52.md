# Práctica 3 monis

Estado: Sin empezar
ítem principal: Programación Concurrente (Programaci%C3%B3n%20Concurrente%202520675b4f6f801eaf70d390ea0e041e.md)

[Practica-3---Monitores.pdf](Pr%C3%A1ctica%203%20monis/Practica-3---Monitores.pdf)

![image.png](Pr%C3%A1ctica%203%20monis/image.png)

![image.png](Pr%C3%A1ctica%203%20monis/image%201.png)

1. Yo creo que si funciona correctamente, ya que gracias al while usado dentro del procedimiento entrarPuente, se controla que dos procesos no accedan simultáneamente/concurrentemente al monitor. Si en vez de un while fuera un if, en ese caso, cuando se despierta al proceso dormido con signal, si otro proceso gana el acceso al monitor cuando todavía cant = 0, ambos estarían entrando a la SC al mismo tiempo. El while permite re chequear la condición para que esto no suceda.
2. Claramente, si hablamos de no usar monitor, simplemente usando un semáforo mutex que simule el paso del puente, simplificaríamos mucho más este programa.
3. La solución original no respeta el orden de llegada por lo mencionado en a., dado el caso en el que un auto al ser despertado, pierda por competencia el acceso al monitor por un auto recién llegado. 

![image.png](Pr%C3%A1ctica%203%20monis/image%202.png)

Monitor motor para administrar el acceso de hasta 5 consultas.

procesos: solicitarAcceso,finalizarLectura

variables: int cant, cond cola

```jsx
Monitor Motor
	cond cola;
	int cant=0;
	
	Procedure solicitarAcceso();
		while (cant==5) wait(cola);
		cant++; 
	end;
	
	Procedure finalizarLectura();
		cant--
		signal(cola)
	end;
End Monitor;

Process Proceso[N]{
	Motor.solicitarAcceso();
	//Leer BD
	Motor.finalizarLectura();
}
```

![image.png](Pr%C3%A1ctica%203%20monis/image%203.png)

```jsx
Process Persona[id: 0..N-1]{
	Fotocopiadora.solicitarAcceso();
		Fotocopiar();
	Fotocopiadora.finalizar();
}//Solución que no respeta el orden.
Monitor Fotocopiadora
	cond cola;
	bool libre=true;
	
	Procedure solicitarAcceso();
		while (not libre) wait(cola);
		libre = false; 
	end;
	
	Procedure finalizar();
		libre = true;
		signal(cola)
	end;
End Monitor;

// b) Modifique la solución de (a) para el caso en que se deba respetar el orden de llegada.
Process Persona[id: 0..N-1]{
	Fotocopiadora.solicitarAcceso();
		Fotocopiar();
	Fotocopiadora.finalizar();
}
Monitor Fotocopiadora
	cond cola;
	int esperando=0;
	bool libre=true;
	
	Procedure solicitarAcceso();
		if (not libre){
			esperando ++;
			wait(cola)
		}
		else libre = false;
	end;
	
	Procedure finalizar();
		if (esperando > 0) { // si hay alguien esperando, le paso el lugar
			esperando--
			signal(cola);
		}
		else libre = true; // si no lo dejo libre para quien venga luego, acceda.
	end;
End Monitor;

// c. prioridad por edad
Process Persona[id: 0..N-1]{
	Fotocopiadora.solicitarAcceso(id,edad);
		Fotocopiar();
	Fotocopiadora.finalizar();
}
Monitor Fotocopiadora
	cond cola[N];
	colaOrdenada fila;
	int idAux, esperando=0;
	bool libre;
	
	Procedure solicitarAcceso(idPersona,edad: in int);
		if (not libre){
			insertar(fila,idPersona,edad);
			esperando ++;
			wait(cola[idPersona])
		}
		else libre = false;
	end;
	
	Procedure finalizar();
		if (esperando > 0) { // si hay alguien esperando, le paso el lugar
			esperando--
			sacar(fila,idAux)
			signal(cola[idAux]);
		}
		else libre = true; // si no lo dejo libre para quien venga luego, acceda.
	end;
End Monitor;

//editar a para respetar el orden por id
Process Persona[id: 0..N-1]{
	Fotocopiadora.solicitarAcceso(id);
		Fotocopiar();
	Fotocopiadora.finalizar();
}
Monitor Fotocopiadora
	cond cola[N];
	int orden=0;
	bool libre=true;
	
	Procedure solicitarAcceso(idPersona: in int);
		while (not libre) wait(cola[idPersona])
		if (orden == idPersona){ // Si es mi turno
			libre = false; // Desocupada -> Ocupo
		}
	end;
	// ejemplo
	Procedure finalizar();
		orden++;
		signal(cola[orden]);
	end;
End Monitor;

//e) Modifique la solución de (b) para el caso en que además haya un Empleado
// que le indica a cada persona cuando debe usar la fotocopiadora.
Process Persona[id: 0..N-1]{
	Fotocopiadora.solicitarAcceso();
		Fotocopiar();
	Fotocopiadora.finalizar();
}

Process Empleado{

}

Monitor Fotocopiadora
	cond cola;
	int esperando=0;
	bool libre;
	
	Procedure solicitarAcceso();
		if (not libre){
			esperando ++;
			wait(cola)
		}
		else libre = false;
	end;
	
	Procedure finalizar();
		if (esperando > 0) { // si hay alguien esperando, le paso el lugar
			esperando--
			signal(cola);
		}
		else libre = true; // si no lo dejo libre para quien venga luego, acceda.
	end;
End Monitor;
```

---

![image.png](Pr%C3%A1ctica%203%20monis/image%204.png)

```jsx
Process Vehiculo [id: 0..N-1]{
int peso = rand;
puente.solicitarAcceso(id,peso);
--cruzarPuente;
puente.finCruce(peso);
}

Monitor Puente{
	int pesoAct = 0;
	Cola C; //Esto debería ser cola especial 
	cond[N] esperar;
	bool libre=true;
	int idAux,pesoAux=0; 
	
	procedure solicitarAcceso(in int id,peso){
		if (!libre || pesoAct+peso > 50000){ // si no puede pasar
			push (C (id,peso)); //se encola
			wait(esperar[id]); // y demora
		}
		libre=false;
		pesoAct=pesoAct+peso; //una vez despierto, aumento pesoAct
		--concedeAcceso 
	}
	
	procedure finCruce(peso){
		
		pesoAct -= peso;
		if (!C.isEmpty()){//si la cola no está vacía
				pop(C(idAux,pesoAux));
				if (pesoAct+pesoaux > 50000){
					pushFront(C,(idAux,pesoAux)); //Lo encolo en la pos que estaba
				} 
				else{//si no, despierto, y concedo acceso
					signal(esperar[idAux]);
				}
		}
		else 
		{//si está vacía la cola, entonces libero el recurso
			libre=true;		
		}
	}

}
```

Tengo que encontrar la forma de desencolar en orden sólo sí, el peso del siguiente, sumado al pesoAct, no supera los 50000.