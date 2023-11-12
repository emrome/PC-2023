Consultar ej5 de PMS
Desde 4-8 de ADA
## Ejercicio 1

Se requiere modelar un puente de un único sentido que soporta hasta 5 unidades de peso.

El peso de los vehículos depende del tipo: cada auto pesa 1 unidad, cada camioneta pesa 2 unidades y cada camión 3 unidades. Suponga que hay una cantidad innumerable de vehículos (A autos, B camionetas y C camiones). Analice el problema y defina qué tareas, recursos y sincronizaciones serán necesarios/convenientes para resolver el problema.

a. Realice la solución suponiendo que todos los vehículos tienen la misma prioridad.


```ada
Procedure PASAR_PUENTE is

TASK puente is
    entry INGRESO_AUTO;
    entry INGRESO_CAMIONETA;
    entry INGRESO_CAMION;
    entry SALIR_AUTO;
    entry SALIR_CAMIONETA;
    entry SALIR_CAMION;
END puente;

TASK TYPE vehiculo; 

arrAuto: array (1..A) of vehiculo;
arrCamioneta: array (1..B) of vehiculo;
arrCamion: array (1..C) of vehiculo;

TASK BODY vehiculo is
BEGIN
    if("Auto")then
        Puente.INGRESO_AUTO;
    else if("Camioneta")then
        Puente.INGRESO_CAMIONETA;
    else 
        Puente.INGRESO_CAMION;

    if("Auto")then
        Puente.SALIR_AUTO;
    else if("Camioneta")then
        Puente.SALIR_CAMIONETA;
    else 
        Puente.SALIR_CAMION;

END vehiculo;

TASK BODY Puente is
    PesoActual:int
BEGIN
    PesoActual = 0;
    SELECT
        ACCEPT SALIR _AUTO;
        PesoActual -= 1;
    OR
        ACCEPT SALIR_CAMIONETA;
        PesoActual -= 2;
    OR
        ACCEPT SALIR_CAMION;
        PesoActual -= 3;
    OR
        WHEN (PesoActual + 1 <= 5) => ACCEPT INGRESO_AUTO;
        PesoActual += 1;
    OR
        WHEN (PesoActual + 2 <= 5) =>ACCEPT INGRESO_CAMIONETA;
        PesoActual += 2;
    OR
        WHEN (PesoActual + 3 <= 5) =>ACCEPT INGRESO_CAMION;
        PesoActual += 3;
END Puente;
                 
BEGIN
    Null;
END PASAR_PUENTE;

```


b. Modifique la solución para que tengan mayor prioridad los camiones que el resto de los vehículos.

```ada
Procedure PASAR_PUENTE is

TASK puente is
    entry INGRESO_AUTO;
    entry INGRESO_CAMIONETA;
    entry INGRESO_CAMION;
    entry SALIR_AUTO;
    entry SALIR_CAMIONETA;
    entry SALIR_CAMION;
END puente;

arrAuto: array (1..A) of vehiculo;
arrCamioneta: array (1..B) of vehiculo;
arrCamion: array (1..C) of vehiculo;

TASK BODY vehiculo is
BEGIN
    if("Auto")then
        Puente.INGRESO_AUTO;
    else if("Camioneta")then
        Puente.INGRESO_CAMIONETA;
    else 
        Puente.INGRESO_CAMION;

    if("Auto")then
        Puente.SALIR_AUTO;
    else if("Camioneta")then
        Puente.SALIR_CAMIONETA;
    else 
        Puente.SALIR_CAMION;

END vehiculo;

//lo mismo con los demas

TASK BODY Puente is
    PesoActual:int
BEGIN
    PesoActual = 0;
    SELECT
        ACCEPT SALIR _AUTO;
        PesoActual -= 1;
    OR
        ACCEPT SALIR_CAMIONETA;
        PesoActual -= 2;
    OR
        ACCEPT SALIR_CAMION;
        PesoActual -= 3;
    OR
        WHEN (PesoActual + 1 <= 5) and INGRESO_CAMION´COUNT = 0 => ACCEPT INGRESO_AUTO;
        PesoActual += 1;
    OR
        WHEN (PesoActual + 2 <= 5) and INGRESO_CAMION´COUNT = 0 =>ACCEPT INGRESO_CAMIONETA;
        PesoActual += 2;
    OR
        WHEN (PesoActual + 3 <= 5) =>ACCEPT INGRESO_CAMION;
        PesoActual += 3;
END Puente;
                 
BEGIN
    Null;
END;

```

## Ejercicio 2 

Se quiere modelar el funcionamiento de un banco, al cual llegan clientes que deben realizar un pago y retirar un comprobante. Existe un único empleado en el banco, el cual atiende de acuerdo con el orden de llegada. Los clientes llegan y si esperan más de 10 minutos se retiran sin realizar el pago.

```ada

Procedure Banco is


TASK EMPLEADO is
    ENTRY Pagar (Pago: IN int; C: OUT text);
END EMPLEADO;

TASK TYPE Cliente;

arrClientes: array (1..N) of Cliente;

TASK BODY CLIENTE is
    pago: int;
    comp: texto;
BEGIN
    SELECT 
        Empleado.Pagar(pago, comp);
    OR DELAY 600
        null;
    END SELECT;
END CLIENTE;

TASK BODY EMPLEADO is
BEGIN
    for i in 1..N loop
        ACCEPT Pagar(Pago: IN int, C: OUT text) do
            C := RealizarPago(Pago);
        END Pagar;
    end loop;
END EMPLEADO;

BEGIN
    null
END Banco;
```


## Ejercicio 3

Se dispone de un sistema compuesto por 1 central y 2 procesos periféricos, que se
comunican continuamente. Se requiere modelar su funcionamiento considerando las
siguientes condiciones:

- La central siempre comienza su ejecución tomando una señal del proceso 1; luego
toma aleatoriamente señales de cualquiera de los dos indefinidamente. Al recibir una
señal de proceso 2, recibe señales del mismo proceso durante 3 minutos.

- Los procesos periféricos envían señales continuamente a la central. La señal del
proceso 1 será considerada vieja (se deshecha) si en 2 minutos no fue recibida. Si la
señal del proceso 2 no puede ser recibida inmediatamente, entonces espera 1 minuto y
vuelve a mandarla (no se deshecha).


```ada
Procedure SistemaDeComputo is

Task Timer is
    Entry Empezar;
End Timer;

Task Central is
    Entry Señal1;
    Entry Señal2;
    Entry Termino;
End Central;

Task Type Periferico1;
Task Type Periferico2;

Proceso1, Proceso2 : Periferico;

Task Body Timer is 
Begin
    Accept Empiezo;
    delay(180);
    Central.Termino;
End

Task Body Central is 
Begin
    Accept Señal1;
    loop
        Select
            Accept Señal1;
        or
            Accept Señal2;
            Timer.Empiezo
            loop
                Select
                    When (Termino'count = 0) => Accept Señal2;
                or
                    Accept Termino;
                    break;
            End Loop
        End Select;
    End loop;
End Central;

Task Body Periferico1 is 
Begin
    loop
        // Genera Señal
        Select 
            Central.Señal1;
        Or Delay 120;
            null;
        End Select;
    end loop;
End Periferico1;


Task Body Periferico2 is 
Begin
    // Genera Señal
    loop
        Select
            Central.Señal2;
            // Genera Señal
        Else
            delay(60)
        End Select;
    end loop;
End Periferico2;

Begin
    null;
End SistemaDeComputo;

```

## Ejercicio 4
En una clínica existe un médico de guardia que recibe continuamente peticiones de atención de las E enfermeras que trabajan en su piso y de las P personas que llegan a la clínica ser atendidos.

Cuando una persona necesita que la atiendan espera a lo sumo 5 minutos a que el médico lo haga, si pasado ese tiempo no lo hace, espera 10 minutos y vuelve a requerir la atención del médico. Si no es atendida tres veces, se enoja y se retira de la clínica. 

Cuando una enfermera requiere la atención del médico, si este no lo atiende inmediatamente le hace una nota y se la deja en el *consultorio* para que esta resuelva su pedido en el momento que pueda (el pedido puede ser que el médico le firme algún papel). Cuando la petición ha sido recibida por el médico o la nota ha sido dejada en el escritorio, continúa trabajando y haciendo más peticiones.

El médico atiende los pedidos dándole prioridad a los enfermos que llegan para ser atendidos. 

Cuando atiende un pedido, recibe la solicitud y la procesa durante un cierto tiempo. Cuando está libre aprovecha a procesar las notas dejadas por las enfermeras

```ada
Procedure Clinica is

Task Medico is
    Entry pedidoPersona;
    Entry pedidoEnfermera;
End Medico; 

Task Consultorio is
    Entry dejarNota(N: IN text);
    Entry pedidoNota(Ok: OUT bool, Nota: OUT text);
End Consultorio;

Task Type Enfermera;
Task Type Persona;

arrEnfermeras: array (1..E) of Enfermera;
arrPersonas: array (1..P) of Persona;

Task Body Consultorio is
    notas: queue of text;
Begin
    loop
        Select
            Accept dejarNota(N: IN text) do
                notas.push(N);
            End dejarNota;
        or 
            Accept pedidoNota(Ok: OUT bool; Nota: OUT text) do
                if (not notas.empty())then
                    Ok := true;
                    Nota := notas.pop()
                else
                    Ok := false;
                    Nota := "no hay nota";
            End pedidoNota;
        End Select;
    end loop;
End Consultorio;

Task Body Medico is
Begin
    loop
        Select
            When (pedidoPersona'count = 0) => Accept pedidoEnfermera do
                                                //Atiende Enfermera
                                              End PedidoEnfermera;
        or
            Accept pedidoPersona do
                //Atiende Persona
            End pedidoPersona;
        else
            Select
                Consultorio.pedidoNota(ok, nota)
                if (ok) then
                    resolverNota(nota);
                
            Else
                null;
            End Select;
        End Select;
    end loop;
End Medico;

Task Body Enfermera is
    nota:text;
Begin
    loop
        Select
            Medico.pedidoEnfermera;
        else
            nota := hacerNota()
            Consultorio.dejarNota(nota)
        End Select;
    end loop;
End Enfermera;

Task Body Persona is
    contador:int;
Begin
    contador := 0;
    loop
        Select 
            Medico.pedidoPersona;
        Or Delay 300 // 5 minutos
            contador +=1;
            if (contador < 3) then
                delay(600);
            end if;
        End Select;
    end loop;
End Persona;

Begin 
    null;
End Clinica;
```

## Ejercicio 5

En un sistema para acreditar carreras universitarias, hay UN Servidor que atiende pedidos de U Usuarios de a uno a la vez y de acuerdo con el orden en que se hacen los pedidos.

Cada usuario trabaja en el documento a presentar, y luego lo envía al servidor; espera la respuesta de este que le indica si está todo bien o hay algún error. Mientras haya algún error, vuelve a trabajar con el documento y a enviarlo al servidor. Cuando el servidor le responde que está todo bien, el usuario se retira. Cuando un usuario envía un pedido espera a lo sumo 2 minutos a que sea recibido por el servidor, pasado ese tiempo espera un minuto y vuelve a intentarlo (usando el mismo documento).

```ada
Procedure acreditarCarreras is

Task Servidor is
    Entry pedido(doc: IN text, ok: OUT bool);
End Servidor;

Task Type Usuario;

arrUsuarios: array (1..U) of Usuario;

Task Body Servidor is
Begin
    loop
        Accept pedido(doc IN: text, ok OUT: bool) do
            ok := //Analiza documento(doc);
            rand();
        End pedido;
    end loop;
End Servidor;

Task Body Usuario is
    documento: text;
    Ok: bool;
    termine: bool;
Begin
    termine := False;
    Ok := False;
    documento := //Generar documento;
    
    while Ok = False loop
        Select
            Servidor.pedido(documento, Ok);
        Or Delay 120
            delay(60);
        End Select;
        if Ok = False
            documento := corregirErrores(documento)
        end if;
    end loop;

End Usuario;

Begin
End acreditarCarreras;
```

## Ejercicio 6

En una playa hay 5 equipos de 4 personas cada uno (en total son 20 personas donde cada
una conoce previamente a que equipo pertenece). Cuando las personas van llegando esperan con los de su equipo hasta que el mismo esté completo (hayan llegado los 4 integrantes), a partir de ese momento el equipo comienza a jugar. El juego consiste en que cada integrante del grupo junta 15 monedas de a una en una playa (las monedas pueden ser de 1, 2 o 5 pesos) y se suman los montos de las 60 monedas conseguidas en el grupo. Al finalizar cada persona debe conocer el grupo que más dinero junto. 

*Nota*: maximizar la concurrencia. Suponga que para simular la búsqueda de una moneda por parte de una persona existe una función Moneda() que retorna el valor de la moneda encontrada.

```ada
Procedure Juego is

    Task Type Equipo is
        Entry Ident(Pos: IN int);
        Entry Llegada;
        Entry QuieroEmpezar;
        Entry Monedas(pesos: IN int)
    End Equipo;

    Task Type Persona;

    Task Juez is
        Entry Resultado(Res: IN int, Id: IN int);
        Entry Ganador(Ganador: OUT int);
    End Juez;

    arrEquipos: array (1..5) of Equipo;
    arrPersonas: array (1..20) of Persona;

    Task Body Juez is
        max: int;
        equipo_max: int;
        resultados: array (1..5) of int;
    Begin
        max:= -1;
        equipo_max:=0;
        For i in 1..5 loop
            Accept Resultado(Res: IN int, Id: IN int) do
                    resultados[Id]:= Res;
            End Resultado;
        end loop;

        For i in 1..5 loop
            if resultados[id] > max
                max:= resultados[i];
                equipo_max:= i;
            end if;
        end loop;

        For i in 1..20 loop
            Accept Ganador(ganador: OUT int) do
                ganador:= equipo_max; 
            End Ganador;
        end loop;

    End Juez;

    Task Body Equipo is
        total: int;
        id: int;
    Begin
        For i in 1..10 loop
            Accept Llegada;
        end loop;

        For i in 1..10 loop
            Accept QuieroEmpezar;
        end loop;

        total:= 0;
        For i in 1..4 loop
            Accept Monedas(pesos: IN int) do
                total += pesos;
            End Monedas;
        end loop;    
        
        Accept Ident(Pos: IN int) do 
            id:= Pos;
        End Ident;

        Juez.Resultado(pesos, id);
    End Equipo;

    Task Body Persona is
        equipo: int; //Conoce el equipo
        monedas: int;
        ganador: int
    Begin
        monedas := 0;
        arrEquipos[equipo].Llegada;
        arrEquipos[equipo].QuieroEmpezar;

        For i in 1..15 loop
            monedas += Moneda();
        end loop;
        arrEquipos[equipo].Monedas(monedas);
        Juez.Ganador(ganador);
    End Persona;

Begin    
    for i in 1..5 loop
        arrEquipos(i).Ident(i);
    end loop
End Juego;
```


## Ejercicio 7

Hay un sistema de reconocimiento de huellas dactilares de la policía que tiene 8 Servidores para realizar el reconocimiento, cada uno de ellos trabajando con una Base de Datos propia; a su vez hay un Especialista que utiliza indefinidamente. 

El sistema funciona de la siguiente manera: el Especialista toma una imagen de una huella (TEST) y se la envía a los servidores para que cada uno de ellos le devuelva el código y el valor de similitud de la huella que más se asemeja a TEST en su BD; al final del procesamiento, el especialista debe conocer el código de la huella con mayor valor de similitud entre las devueltas por los 8 servidores. Cuando ha terminado de procesar una huella comienza nuevamente todo el ciclo.

*Nota*: suponga que existe una función Buscar(test, código, valor) que utiliza cada Servidor donde recibe como parámetro de entrada la huella test, y devuelve como parámetros de salida el código y el valor de similitud de la huella más parecida a test en la BD correspondiente. Maximizar la concurrencia y no generar demora innecesaria.

```ada
Procedure reconocimientoHuellas is

Task Especialista is
    Entry Test(Huella: OUT img??)
    Entry Respuesta(Codigo: IN int, Valor: IN int);
End Especialista;

Task Type Servidor;

arrServidores: array (1..8) of Servidor;

Task Body Servidor is
    test: img;
    valor, codigo: int;
    BD: bd;
Begin
    loop
        Especialista.Test(test);
        Buscar(test, codigo, valor);
        Especialista.Respuesta(codigo, valor);
    end loop;
End Servidor;

Task Body Especialista is
    max_valor: int;
    aux_valor: int;
    aux_cod: int;
    cod: int;

    huella: img;
Begin
    loop
        huella:= //Tomar imagen de huella;
        For i in (1..8) loop
            Accept Test(Huella: OUT img) do
                Huella:= huella;
            End Test;
        End loop;
        max_valor:=0;
        For i in (1..8) loop
            Accept Respuesta(Codigo: IN int, Valor: IN int) do
                aux_cod:= Codigo;
                aux_valor:=Valor;
            End Respuesta;
            if aux_valor > max_valor 
                max_valor:= aux_valor;
                cod:= aux_cod;
            end if;
        End loop;
    end loop;
End Especialista;

Begin
    null;
End reconocimientoHuellas;
```


## Ejercicio 8

Una empresa de limpieza se encarga de recolectar residuos en una ciudad por medio de 3 camiones. Hay P personas que hacen continuos reclamos hasta que uno de los camiones pase por su casa. Cada persona hace un reclamo, espera a lo sumo 15 minutos a que llegue un camión y si no vuelve a hacer el reclamo y a esperar a lo sumo 15 minutos a que llegue un camión y así sucesivamente hasta que el camión llegue y recolecte los residuos; en ese momento deja de hacer reclamos y se va. Cuando un camión está libre la empresa lo envía a la casa de la persona que más reclamos ha hecho sin ser atendido. 

*Nota*: maximizar la concurrencia.

```ada
Procedure recolectarResiduos is

Task Type Camion;

Task Admin is
    Entry Reclamo(Id: IN int);
    Entry SigPersona(Id: OUT int);
    Entry MeVoy(Id: IN int);
End Admin;

Task Type Persona is
    Entry Identificador(Pos: IN int);
    Entry Termine;
End Persona;

Task Empresa is
    Entry Proximo(Id: OUT int);
End Empresa;

arrCamiones: array (1..3) of Camion;
arrPersonas: array (1..P) of Persona;

Task Body Camion is
    idAux: int;
Begin
    loop
        Servidor.Proximo(idAux);
        //Recolectar residuos idAux;
        arrPersonas[idAux].Termine;
    end loop;
End Camion;

Task Body Empresa is
    idAux: int;
Begin
    loop
        Accept Proximo(id: OUT int) do
            Admin.SigPersona(idAux)
            id:= idAux;
        End Proximo;
    End loop;
End Empresa;

Task Body Persona is
    id: int;
    recolectado: bool;
Begin
    recolectado:= False;
    Accept Identificador(Pos: IN int) do
        id:= Pos;
    End Identificador;

    while recolectado = False loop
        Admin.Reclamo(id;)
        Select
            Accept Termine;
            recolectado:= True;
        Or Delay 900
        End Select;
    end loop;
    Admin.MeVoy(id);
End Persona;

Task Body Admin is
    cantReclamos: array (1..P) of int;
    activos: array (1..P) o bool;//Inicializa en True;
    idAux: int;
Begin
    loop
        Select
            Accept Reclamo(Id: IN int) do
                cantReclamos[Id] += 1;
            End Reclamo;
        Or  
            Accept SigPersona(Id: OUT int) do
                Id:= calcularMinimo(cantReclamos, activos);
            End SigPersona;
        Or
            Accept MeVoy(Id: IN int) do
                activos[Id]:= False
            End Mevoy;
        End Select;
    end loop;
End Camion;

Begin
    For i in (1..P) loop
        arrPersonas[i].Identificador(i);
    End loop;
End recolectarResiduos;
```