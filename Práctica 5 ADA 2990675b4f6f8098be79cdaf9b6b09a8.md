# Práctica 5 ADA

Estado: Sin empezar
ítem principal: Programación Concurrente (Programaci%C3%B3n%20Concurrente%202520675b4f6f801eaf70d390ea0e041e.md)

![image.png](Pr%C3%A1ctica%205%20ADA/image.png)

![image.png](Pr%C3%A1ctica%205%20ADA/image%201.png)

![image.png](Pr%C3%A1ctica%205%20ADA/image%202.png)

```arduino
Procedure Ejercicio1 is
Task puente is
	Entry PasarAuto;
	Entry PasarCamioneta;
	Entry PasarCamion;
	Entry Salir(peso: IN int);
	
Task Body puente is
	total:int; 
begin 
	total = 0;
	loop
		SELECT
			WHEN (total <5) => ACCEPT PasarAuto do
													total+=1;
													END PasarAuto
			OR
				WHEN (total<4) => ACCEPT PasarCamioneta do
													total+=2;
													END PasarCamioneta
			OR 
			 WHEN (total<3)=> ACCEPT PasarCamion do
													total+=3;
												END PasarCamion
			OR
				ACCEPT Salir(peso: IN int) do
						total-=peso;
				END Salir;
	end loop;
end puente;

Task type Auto;
ArrA: array (1..A) of Auto;
Task Body Auto is
	miPeso : int;
begin
	miPeso = asignarPeso(); //funcion ficticia que asigna el peso del auto
	puente.PasarAuto;
	// El auto cruza el puente
	puente.Salir(miPeso);
end

Task type Camioneta;
ArrB: array(1..B) of Camioneta
Task Body camioneta is
	miPeso : int;
begin
	miPeso = asignarPeso(); //funcion ficticia que asigna el peso del auto
	puente.PasarCamioneta;
	// El auto cruza el puente
	puente.Salir(miPeso);
end

Task type Camion;
ArrC:array(1..C) of Camion;
Task Body camion is
	miPeso : int;
begin
	miPeso = asignarPeso(); //funcion ficticia que asigna el peso del auto
	puente.PasarCamion;
	// El auto cruza el puente
	puente.Salir(miPeso);
end

Begin
	NULL;
End Ejercicio1;

-------------------------------------------------------------------------
Para que los camiones tengan más prioridad que el resto de los vehiculos

Task puente is
	Entry PasarAuto();
	Entry PasarCamioneta();
	Entry PasarCamion();
	Entry Salir(peso: IN int);
	
Task Body puente is
	total:int; 
begin 
	total = 0;
	loop
		SELECT
			WHEN ((PasarCamion'count = 0) AND (total <5)) => 
													ACCEPT PasarAuto do
													total+=1;
													END PasarAuto
			OR
				WHEN ((PasarCamion'count = 0) AND (total<4)) =>
												  ACCEPT PasarCamioneta do
													total+=2;
													END PasarCamioneta
			OR 
			 WHEN (total<3)=> ACCEPT PasarCamion do
													total+=3;
												END PasarCamion
			OR
				ACCEPT Salir(peso: IN int) do
										total-=peso;
				END Salir;
	end loop;
end puente;
```

![image.png](Pr%C3%A1ctica%205%20ADA/image%203.png)

```arduino
Procedure Ejercicio2 is

Task type Cliente is
	entry recibirComprobante(comprobante: OUT texto);
End Cliente;
arrClientes: array[1..N] of Cliente;
Task Body Cliente is
	comprobante:texto;
	monto:double;
begin
	monto = calcularMonto(); 
	Empleado.realizarPago(monto,id);
	Empleado.recibirComprobante(comprobante);
end Cliente;

Task Empleado is
	entry realizarPago(monto: in double, idC: IN int);
End Empleado;

Task Body Empleado is
	pago:double;
	comprobante:texto;
begin
	for i in 1..N loop
		accept realizarPago(pago, idC);
		comprobante: generarComprobante();
		Cliente(idC).recibirComprobante(comprobante)
	end loop;
end Empleado;

Begin
	null;
End Ejercicio2;

-------------------------------------------------------------------------

Los clientes se retiran si esperan + de 10 minutos

Task Body Cliente is
	comprobante:texto;
	monto:double;
begin
	monto = calcularMonto(); 
	SELECT
		Empleado.realizarPago(monto,id);
		Empleado.recibirComprobante(comprobante);
	OR DELAY 600.0
		NULL;
	END SELECT;
end Cliente;

-------------------------------------------------------------------------
Los clientes esperan a lo sumo 10 minutos para ser atendidos. 
Si pasado ese lapso no fueron atendidos, entonces 
solicitan atención una vez más y se retiran 
si no son atendidos inmediatamente

Task Body Cliente is
	comprobante:texto;
	monto:double;
begin
	monto = calcularMonto(); 
	SELECT
		Empleado.realizarPago(monto,id);
		Empleado.recibirComprobante(comprobante);
	OR DELAY 600.0
		SELECT
			Empleado.realizarPago(monto,id);
			Empleado.recibirComprobante(comprobante);
		ELSE
			NULL;
		END SELECT;
	END SELECT;
end Cliente;

```

![image.png](Pr%C3%A1ctica%205%20ADA/image%204.png)

```arduino
Procedure Ejercicio3 is

Task Periferico1;
Task Body Periferico1 is
Begin
	loop
		SELECT 
			Central.señal1;
		OR DELAY (120)
			NULL;
		END SELECT;
	end loop;
End;

Task periferico2;
Task Body Periferico2 is
Begin
	señal = generarSeñal()
	loop
		SELECT 
			Central.señal2(señal);
			señal = generarSeñal()
		ELSE 
			Delay(60);
		END SELECT;
	end loop;
End;

Task Timer IS
	entry iniciarTimer;
End Timer;
Task Body Timer is
Begin 
	loop
		ACCEPT iniciarTimer;
		DELAY(180)
		Central.Aviso;
	end loop;
End;

Task Central is
	entry señal1;
	entry señal2;
	entry aviso;
End Central;
Task Body Central is
	fin:bool
Begin
	ACCEPT señal1;	
	loop
		SELECT
			ACCEPT señal2; //
			fin=false;
			Timer.iniciarTimer;
			while(not fin) loop
				SELECT
						WHEN (aviso'count = 0) =>
								ACCEPT señal2
						OR
							ACCEPT Aviso DO
								fin=true;
							END Aviso;
				END SELECT;
			end loop;
		ELSE
			ACCEPT señal1;
		END SELECT;
	end loop

End Central;
Begin
	null;
End Ejercicio3;
```

![image.png](Pr%C3%A1ctica%205%20ADA/image%205.png)

```arduino
Procedure Clinica is
Task type persona;
arrayP: array(1..P) of Persona;
Task Body Persona is
Begin
	SELECT
		Medico.Atender;
	OR DELAY(300)
		for 1..2 loop
			DELAY(600)
			Medico.Atender;
		end loop;
	END SELECT;
End Persona;

Task type  Enfermera;
arrayE: array(1..P) of Enfermera;
Task body Enfermera is
pedido:texto;
Begin
	loop
		pedido = generarPedido();
		SELECT
			Medico.Pedido(pedido)
		ELSE
			Escritorio.DejarNota(pedido)
		END SELECT;
	end loop;
End Enfermera;

Task Escritorio is
	entry DejarNota(nota: IN texto);
Task Body Escritorio is
nota:texto;
Begin
	nota:'No hay Notas';
	loop
		ACCEPT DejarNota(nota);
		Medico.atenderNota(nota);
	end loop;

End Escritorio;

Task Medico is
	entry Pedido(pedido: IN OUT texto);
	entry Atender;
	entry AtenderNota(nota: OUT texto);
End Medico;
Task Body Medico is
	nota,pedido:texto;
Begin
	loop
		SELECT
			ACCEPT Atender;
		ELSE
			ACCEPT Pedido(pedido)
		ELSE
			ACCEPT atenderNota(nota)
		END SELECT;
	end loop;
End Medico;

Begin
	null;
End Clinica;
```

![image.png](Pr%C3%A1ctica%205%20ADA/image%206.png)

Pensé en hacer un administrador en el medio, porque si hago que el servidor solo se administre, y espere la corrección de cada usuario, pierdo mucha concurrencia ya que el ACCEPT DO es bloqueante. En cambio, con mi solución, aprovecho el tiempo en el que el Usuario realiza correcciones, para corregir otros documentos.

```arduino
Procedure Universidad is

Task type Usuario is
	entry identificacion(Pos: IN int);
end Usuario;
ArrU: array (1..U) of Usuario;
Task Body Usuario is
	res:bool;
	doc:text;	
begin
	doc=trabajarDoc();
	res=true;
	while(res) loop // mientras haya errores
		SELECT
			Administrador.pedido(doc,res); //este param recibido modifica mi var?
			if (res) doc=corregirDoc(doc);
		OR DELAY(120.0)
			Delay(60.0);
		END SELECT;
	end loop

end Usuario;

Task Administrador is
	entry pedido(doc: IN texto, res:OUT bool);
end Administrador;

Task Body Administrador is
begin
	loop // No se cuántas veces tendrán que corregir los docs
		ACCEPT pedido(doc: IN text, res: OUT bool) do
				res=CorregirDoc(doc);
		END siguiente;
	end loop;
end Administrador;
```

![image.png](Pr%C3%A1ctica%205%20ADA/image%207.png)

```arduino
// Mi segunda variante es Persona ↔ Equipo ↔ Admin
Task Type Persona is
	entry comenzar;
	entry eqGanador(eqGan: IN int);
end Persona;
Task Body Persona is
	int equipo,total;
begin
	Equipos(equipo).barrera;
	accept comenzar;
		
	//juntan monedas
	total=0;
	for(i=0;i<14;i++){
		total += Moneda();
	}
	Equipos(equipo).sumar(total)
	accept eqGanador(eqGan: IN int);
end Persona;

Task Type Equipo is
	entry IdentE(id: IN int);
	entry barrera;
	entry sumar(junte: IN int);
	entry ganador(eq: IN int);
end Equipo;

Equipos : array (1 .. 5) of Equipo;

Task Body Equipo is
int nro,cont,tot,eqG;
begin
	accept IdentE(id: IN int) do
		nro = id;
	end IdentE;
	// gestiona la barrera
	cont=0;
	while(cont<4) accept barrera; cont++;
	for (i=0;i<4;i++){
		Persona(i).comenzar;
	}
	//Junta los valores
	cont=0; tot=0;
	while(cont<4){
		accept junte(total) do
			tot=total;
		end junte;
		cont++;
	}
	//Informa al Admin
	Admin.finEquipo(tot,nro);
	accept ganador(eqG: IN int);
	for (i=0;i<4;i++){
		Persona(i).ganador(eqG);
	}
end Equipo;

Task Admin is
	entry finEquipo(total: IN int, idE: IN int);
end Admin;
Task Body Admin is
	max,maxE,totParcial,idE:int;
begin

	max=0;
	for(i=0;i<5;i++){
		accept finEquipo(totParcial,idE) do
			if (totParcial>max) then
				max=totParcial; maxE=idE;
			end if;
		end finEquipo;
	}
	for(i=0;i<5;i++){
		Equipo(i).ganador(maxE);
	}
end Admin;

Begin //los Ident siempre en el pp, fuera de las task.
	for(i=0;i<5;i++){
		Equipo(i).Ident(i);
	}
End ejercicio6;

```

![image.png](Pr%C3%A1ctica%205%20ADA/image%208.png)

```arduino
Task Type Worker is
	entry ident(id: IN int);
end Worker;

ArrWorker: array (0..9) of Worker;

Task Body worker is
	int vector[100000]; //Se dispone inicializado previamente
	int totalParcial;
begin
	totalParcial=0;
	for i in 0..99999 loop
		totalParcial+=vector[i];
	end loop;
	Coordinador.fin(totalParcial);
end Worker;

Task Coordinador is
	entry fin(totalParcial: IN int);
end Admin;

Task Body Coordinador in is
totAcumulado,cantNros:int;
promedio:double;
begin
	cantNros=1.000.000;
	totAcumulado=0;
	for i in 0..9 loop
		accept fin(totalParcial: IN int) do
			totAcumulado+=totalParcial
		end fin;
	end loop;
	
	promedio = totAcumulado / cantNros;
	
end Admin;
```

![image.png](Pr%C3%A1ctica%205%20ADA/image%209.png)

```arduino
Procedure ejercicio8 is
Task Especialista is
	entry pedido(test: OUT double);
	entry simil(cod: IN int; valor: IN double);
end Especialista;

Task Body Especialista is
codImg,codSimil,valorMaxSimil: int;
imagen:double;
begin
	loop
		imagen:= TomarHuella();//toma el valor de una imagen de una huella test (función inventada)
		min=99999; //valor más semejante
		cant=0; //cant de muestras devueltas por los 8 servidores
		while(cant<8) loop
			SELECT
				ACCEPT pedido(test: OUT double) do
					test = imagen;
				end pedido;
			OR
			  WHEN (pedido'count = 0) => ACCEPT simil(cod,valor) do
									codSimil=cod;
									if(valor/imagen<min){
										min=valor/imagen; valorMaxSimil=codSimil;
									}
									cant++;
								end simil;
			END SELECT;//al salir del do, todas las sentencias son no elegibles, por lo tanto
			//ya conoce el código de la huella con mayor similitud entre las devueltas
		end loop;//end while
		for i in 1..8 loop
			Servidor(i).siguiente;
		end loop;
	end loop; //loop infinito
end Especialista;

Task Type Servidor;
Task Body Servidor is
	codHuellaTest,codHuellaSimil,valorSimil:double;
begin
	loop
		Especialista.pedido(test)
		Buscar(codHuellaTest,codHuellaSimil,valorSimil); //mensaje a su base de datos
		Especialista.simil(codHuellaSimil,valorSimil);
		ACCEPT siguiente; //Para no procesar dos veces la misma imagen
	end loop;
end Servidor;

Begin
	null;
End ejercicio8;

```

![image.png](Pr%C3%A1ctica%205%20ADA/image%2010.png)

```c
Procedure ejercicio9 is

Task Type Persona is
	entry camion;
end Persona;
ArrPer: array (1..P) of Persona;
Task Body Persona is
	paso:bool;
	id:int;
begin
	//identificar persona
	ACCEPT Ident(idP: IN int);
		id:=idP;
	end Ident;
	
	paso=false
	Admin.reclamo(id)
	while(not paso) loop
		SELECT
			ACCEPT camion;
			paso = true;
		OR DELAY 900.0
			Admin.reclamo(id);
		END SELECT;
	end loop;
end Persona;

Task Admin is
	entry reclamo(llego: OUT bool)
end Admin;

Task body Admin is
	personasAtendidas,idP:int
	reclamos : array (1..P) of int;
begin
	max = -1;
	cantAtender = 0;  //Cantidad de Reclamos a atender en total
	reclamos(P) = ((P) 0);
	personasAtendidas=0;
	while(personasAtendidas<P) loop
		SELECT
			ACCEPT reclamo(idP: IN int) do
					if (reclamos(idP) > -1) {
						cantAtender++;
						reclamos(idP)++;
					}
			end reclamo;
		OR
			WHEN (cantAtender>0) => ACCEPT libre(idP: OUT int) do	
																	idP = posMax(reclamos);	
																  reclamos(idP) = -1;
																	personasAtendidas++;
																	cantAtender -= reclamos(idP); //por eso serestan 
															 end libre;
	end loop
end Admin;

Task Type Camion;
ArrC: array (1..3) of Camion
Task Body Camion is
	fin:bool;
	cant,id:int;

begin
	fin = false;
	loop
		Admin.Libre(idP);
		RecoresResiduos();
		Persona(idP).camion;
	end loop;
end;

Begin
	for i in 1..3 loop
		ArrP(i).Ident(i);
	end loop
End ejercicio9;
```