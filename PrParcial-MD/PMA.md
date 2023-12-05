## 1. Suponga que N clientes llegan a la cola de un banco y que serán atendidos por sus empleados. Analice el problema y defina qué procesos, recursos y comunicaciones serán necesarios/convenientes para resolver el problema. Luego, resuelva considerando las siguientes situaciones:
- a. Existe un único empleado, el cual atiende por orden de llegada.

```
chan pedido(int);
chan listo[N];

Process Cliente[id=1..N]{
    send pedido(id);
    receive listo[id];
}

Process Empleado{
    idC:int;

    for i:=1 to N {
        receive pedido(idC);
        delay(rand())//Atender cliente idC
        send listo[idC];
    }
}
```

- b. Ídem a) pero considerando que hay 2 empleados para atender, ¿qué debe 
modificarse en la solución anterior?

```
chan pedido(int);
chan listo[N];

Process Cliente[id=1..N]{
    send pedido(id);
    receive listo[id];
}

Process Empleado[id=1..2]{
    idC:int;
    while(true){
        receive pedido(idC);
        delay(rand()) //Atender Cliente idC
        send listo[idC];
    }   
}
```
- c. Ídem b) pero considerando que, si no hay clientes para atender, los empleados 
realizan tareas administrativas durante 15 minutos. ¿Se puede resolver sin usar 
procesos adicionales? ¿Qué consecuencias implicaría?

Si un proceso adicional se podria usar empty pero podria generar demora innecesaria ya que dos procesos toman del mismo canal.

```
chan Fila(int);
chan Termine[N]()
chan Pedido(int)
chan Siguiente[2](int)

Process Cliente[id=0..N]{
    send Fila(id)
    receive Termine[id]()
}


Process Empleado[id=0..1] {
    int idC;
    while(true){
        send espero_pedido(id)
        receive Siguiente[id](idC)

        if(idC<> -1){
            //atender cliente idC
            send Termine[idC]()
        }
        else {
            delay(900) // Realiza tareas administrativas
        }
    }
}

Process Coordinador{
    int idC;
    int idE;

    while(true){
        receive espero_pedido(idE)
        if(empty(Fila)){
            idC = -1
        }    
        else{
            receive Fila(idC)
        }
        send Siguiente[idE](idC)
    }
}
```

## 2. Se desea modelar el funcionamiento de un banco en el cual existen 5 cajas para realizar pagos. Existen P clientes que desean hacer un pago. Para esto, cada una selecciona la caja donde hay menos personas esperando; una vez seleccionada, espera a ser atendido. En cada caja, los clientes son atendidos por orden de llegada por los cajeros. Luego del pago, se les entrega un comprobante. Nota: maximizando la concurrencia

```
chan PedidoAtencion(int);
chan AsignarCaja[N](int);
chan Fila[5](int, Pago);
chan Comprobantes[P](Comprobante);

Process Cliente[id=1..P]{
    Comprobante: Comprobante;
    Pago: Pago; //Inicializado
    MiCaja:int;

    send PedidoAtencion(id);
    receive AsignarCaja[id](MiCaja);
    send Fila[MiCaja](id, Pago);
    receive Comprobantes[id](Comprobante);
}

Process AdministradorCajas {
    int contador[5]=([5]=0);
    idC:int;
    caja:int;

    while (true){
        receive PedidoAtencion(idC);
        caja = calcularMinimo(contador);
        send AsignarCaja[idC](caja);
        contador[caja]++;
    }
}

Process Cajero[id=1..5]{
    c:Comprobante;
    p:Pago;
    idC: int;

    while(true){
        receive Fila[id](idC, p)
        c = GenerarComprobante(p);
        send Comprobantes[idC](c);
    }
}
``` 
MAAAAAAL

### Corregido 
2) 
```
chan Comprobantes[P](Comprobante)
chan Fila[5](int, int)
chan PedidoAtencion(int)
chan CajaAsignada[P](int)
chan Atendido(int)
chan Señal()

Process Cliente[id=0..P-1]{
    int caja;
    int mi_pago;
    Comprobante comp;

    send PedidoAtencion(id)
    send Señal()
    receive CajaAsignada[P](caja)

    send Fila[caja](id, pago)
    receive Comprobantes[id](comp)
}

Process Cajero[id=0..4]{
    int idC;
    Comprobante c;

    while(true){
        receive Fila[id](idC)
        c = //generar comprobante cliente idC
        send Comprobantes[idC](c)
        send Atendido(id)
        send Señal()
    }
}

Process Coordinador{
    int contador[5]=([5]=0)
    int idC;
    int idCaja;
    int idMin, min=1000;

    while(true){
        receive Señal()
        if (not empty(PedidoAtencion)) --> 
                        receive PedidoAtencion(idC)
                        idMin = //calculoMinimo
                        contador[idMin]++
                        send CajaAsignada[idC](idMin)

           (not empty(Atendido)) --> 
                        receive Atendido(idCaja)
                        contador[idCaja]--
        end if
    }
}
```

## 3. 3. Se debe modelar el funcionamiento de una casa de comida rápida, en la cual trabajan 2 cocineros y 3 vendedores, y que debe atender a C clientes. El modelado debe considerar que:
- Cada cliente realiza un pedido y luego espera a que se lo entreguen.
- Los pedidos que hacen los clientes son tomados por cualquiera de los vendedores y se lo pasan a los cocineros para que realicen el plato. Cuando no hay pedidos para atender,  los vendedores aprovechan para reponer un pack de bebidas de la heladera (tardan entre 1 y 3 minutos para hacer esto).
- Repetidamente cada cocinero toma un pedido pendiente dejado por los vendedores, lo cocina y se lo entrega directamente al cliente correspondiente.

Nota: maximizar la concurrencia.

```
chan pedido_vendedor(int, Pedido)
chan pedido_cocinero(int, Pedido)
chan entrega_pedido[C][Entrega]

chan quiero_pedido(int)
chan siguiente_pedido(int, Pedido)

Process Cliente[id=1..C]{
    pedido: Pedido;
    entrega: Entrega;

    send pedido_vendedor(id, pedido);
    receive entrega_pedido[id](entrega);
}

Process Vendedor[id=1..3]{
    idC: int;
    pedido: Pedido;

    while(true){
        send quiero_pedido(id);
        receive siguiente_pedido(idC, pedido);
        if(idC == -1){
            delay(rand(1..3)) //Reponer stock
        }
        else {
            pedido_cocinero(idC, pedido)
        }
    }
}

Process Admin{
    idC: int;
    idV: int;
    pedido: Pedido;

    while(true){
        receive quiero_pedido(idV);
        if(empty(pedido_vendedor)){
            idC = -1;
            pedido = null;
        }
        else {
            receive pedido_vendedor(idC, pedido);
        }
        send siguiente_pedido(idC, pedido);
    }
}

Process Cocinero[id=1..2]{
    idC: int;
    pedido: Pedido;
    entrega: Entrega;

    while(true){
        receive pedido_cocinero(idC, pedido);
        entrega = prepararPlato(pedido);
        send entregas[idC](entrega);
    }
}
```

## 4. Simular la atención en un locutorio con 10 cabinas telefónicas, el cual tiene un empleado que se encarga de atender a N clientes. Al llegar, cada cliente espera hasta que el empleado le indique a qué cabina ir, la usa y luego se dirige al empleado para pagarle. El empleado atiende a los clientes en el orden en que hacen los pedidos, pero siempre dando prioridad a los que terminaron de usar la cabina. A cada cliente se le entrega un ticket factura. Nota: maximizar la concurrencia; suponga que hay una función Cobrar() llamada por el empleado que simula que el empleado le cobra al cliente.

```
chan pedidoCabina(int);
chan asignarCabina[N](int);
chan termineCabina(int, int);
chan facturas[N](Factura);
chan Señal;

Process Cliente[id=1..N]{
    cabina: int;
    factura: Factura;

    send pedidoCabina(id);
    send señal()
    receive asignarCabina[id](cabina)
    //USAR CABINA
    send termineCabina(cabina, id)
    send señal()
    receive facturas[id](factura);
}

Process Empleado{
    cola cabinasLibres: int //Inicializado
    idC:int;
    cabina:int;

    while(true){
        receive señal()
        if
          - ((empty(termineCabina))  and (not cabinas.empty()) and (not empty(pedidoCabina))) --> 
                                                                  receive pedidoCabina(idC)
                                                                  cabina = cabinas.pop();
                                                                  send asignarCabina[idC](cabina);
          - not empty(termineCabina) --> receive termineCabina(cabina, idC)
                                         cabinas.push(cabina);
                                         factura = Cobrar()
                                         facturas[idC](factura);
        end if;
    }
}
```

## 5. Resolver la administración de las impresoras de una oficina. Hay 3 impresoras, N usuarios y 1 director. Los usuarios y el director están continuamente trabajando y cada tanto envían documentos a imprimir. Cada impresora, cuando está libre, toma un documento y lo imprime, de acuerdo con el orden de llegada, pero siempre dando prioridad a los pedidos del director. Nota: los usuarios y el director no deben esperar a que se imprima el documento

```

```
