# 1° Parcial

Estado: Sin empezar
ítem principal: Programación Concurrente (Programaci%C3%B3n%20Concurrente%202520675b4f6f801eaf70d390ea0e041e.md)

![image.png](1%C2%B0%20Parcial/image.png)

```c
cola C;
bool libre = true;
sem espera[19] = ([30], 0);
sem mutex = 1;
int aux;

Process Empleado [id: 0..19]{
	
	--llegaEmpleado
	P(mutex)
	if (libre){
		libre = false;
		V(mutex)
	}
	else {
		push(C(id));
		V(mutex);
		P(espera[id]);
	}
	
	USAR();
	
	P(mutex)
	if (empty(C)) libre = true;
	else {
		pop(C(aux));
		V(espera[aux]);
	}
	V(mutex)
	
}
```

![image.png](1%C2%B0%20Parcial/image%201.png)

```c
sem mutex = 1, terminaron=0;
A[1000000]; //Se dispone: Dividir la carga entre 5 y dsp sumarlo al finalizar
int termine= 0, sum=0;

Process Worker[id:0..4]{
	int i; int cont=0;
	
	for (i:=id*200000; i<((i+1)*200000)-1; i++){
		cont+=A[i];
	}
	
	P(mutex)
		sum += cont;
		termine++;
		if (termine == 5){
			V(terminaron)
			V(mutex)
		}
		else{
				V(mutex)
				P(terminaron)
		} 
	
	P(mutex)
		imprimir(sum)
	V(mutex)
}
```

![image.png](1%C2%B0%20Parcial/image%202.png)

```c
Process Persona [id:0..49]{
	secretario.realizarConsulta();
	secretario.realizarTramite();
}

//Separar en dos monis para dividir la carga
Monitor AdminConsultas
	
	Cola consultas;
	cond espera, hayC;
	int aux;
	text res[49];
	
	Procedure realizarConsulta(idP: in int;cons: in text; R: out text){
		push(consultas(idP,cons));
		singal(hayC);
		wait(espera);
		R = res[idP];
	}
	
	Procedure SigC(idCliente: out int; consulta: out text){
		if(empty(consultas)) wait(hayC);
		pop(consultas(idCliente,consulta));
	}

	Procedure Resultado(idP: in int; R: out text){
		res[idP] = R;
		signal (espera);
	}	
	
End Monitor;

Monitor AdminTramites
	Cola tramites;
	cond espera,hayT;
	int aux;
	text res[49];
	
	Procedure SigT(idCliente: out int; tramite: out text){
		if(empty(tramites)) wait(hayT);
		pop(tramites(idCliente,tramite));
	}
	
	Procedure realizarTramite(idP: in int; tra: in text; R: out text){
		push(tramites(idP,tra));
		signal(hayT);
		wait(espera);
		R = res[idP];
	}
	
	Procedure Resultado(idP: in int; R: out text){
		res[idP] = R;
		signal (espera);
	}	
	
End Monitor;

Process Empelado {
	text cons, res;
	int aux;
	while(true){
		Admin.SigC(aux,cons);
		res = AnalizarConsulta(cons);
		Admin.Resultado(aux,res);
	}
}

Process Supervisor {
	while(true){
		Admin.SigT(aux,tramite);
		res = AnalizarTramite(tramite);
		Admin.Resultado(aux,res);
	}
}
```

![parcial 1ra fecha 2023.jpeg](1%C2%B0%20Parcial/parcial_1ra_fecha_2023.jpeg)

```c
Cola C;
sem mutex=1,esperar[15]=([15] 0),hayPedido=0;
int aux,temp,nro,res[15];

Process Controlador [id:0..14]{
	P(mutex)
		medir(temp);
		push(C(temp,id))
		V(hayPedido);
	V(mutex)
	P(esperar[id]);	
	actuar(res[id]); //param actuar es nro de 1 a 10
	delay(60)	
}

Process Central [2]{
	while(true){
		P(hayPedido);
		P(mutex)
			pop(C(temp,aux));
		V(mutex)
		res[aux] = determinar();	
		V(esperar[aux]);
	}
}
```

![image.png](1%C2%B0%20Parcial/image%203.png)

```c
Process Persona[id:0..P-1]{

}

Monitor Admin
	int cantE=E; text entradas[P];
	Cola C;
	cond esperar[P],sig;
	
	Procedure comprarEntrada(idAux: in int, res: out text){
		// Cómo modelo "indicar que no hay más entradas"
		push(C(idAux));
		signal(sig);
		wait(esperar[idAux]);
		res = entradas[idAux];
	}
	
	Procedure Sig(aux: out int){
		if (empty(C)) wait(sig)
		else pop(C(aux));
	}
	
	Procedure entregarEntrada(e: in text, idAux: in int){
		entradas[idAux] = e;
		signal(esperar[idAux]);
	}
	
End Monitor;

Process Boleteria{
	while(true){
		int aux; text e;
		Admin.Sig(aux);
		e = vender(aux)
		Admin.entregarEntrada(e,aux)
	}
}
```

![image.png](1%C2%B0%20Parcial/image%204.png)