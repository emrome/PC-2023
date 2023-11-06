## PMA: Pasaje de Mensajes Asincrónico
### 1. 
#### Suponga que N clientes llegan a la cola de un banco y que serán atendidos por sus empleados. Analice el problema y defina qué procesos, recursos y comunicaciones serán necesarios/convenientes para resolver el problema. L
#### a. Existe un único empleado, el cual atiende por orden de llegada
```
chan Fila(int);
chan Termine[N]()

Process Empleado {
    int idP;
    while(true){
        receive Fila(idP)
        // atiende cliente idP
        send Termine[idP]()
    }
}

Process Cliente[id=0..N]{
    send File(id)
    receive Termine[id]()
}
```

#### b. Ídem a pero considerando que hay 2 empleados para atender, ¿qué debe modificarse en la solución anterior?
```
chan Fila(int);
chan Termine[N]()

Process Empleado[id=0..1] {
    int idP;
    while(true){
        receive Fila(idP)
        // atiende cliente idP
        send Termine[idP]()
    }
}

Process Cliente[id=0..N]{
    send File(id)
    receive Termine[id]()
}
```

#### c. Ídem b pero considerando que, si no hay clientes para atender, los empleados realizan tareas administrativas durante 15 minutos. ¿Se puede resolver sin usar procesos adicionales? ¿Qué consecuencias implicaría?

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
        send Pedido(id)
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
        receive Pedido(idE)
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


### 2.
#### Se desea modelar el funcionamiento de un banco en el cual existen 5 cajas para realizar pagos. Existen P clientes que desean hacer un pago. Para esto, cada una selecciona la caja donde hay menos personas esperando; una vez seleccionada, espera a ser atendido. En cada caja, los clientes son atendidos por orden de llegada por los cajeros. Luego del pago, se les entrega un comprobante. Nota: maximizando la concurrencia.
```
chan Comprobantes[P](Comprobante)
chan SigCliente[5](int, int)
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

    send SigCliente[caja](id, pago)
    receive Comprobantes[id](comp)
}

Process Cajero[id=0..4]{
    int idC;
    Comprobante c;

    while(true){
        receive SigCliente[id](idC)
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

### 3. 
#### Se debe modelar el funcionamiento de una casa de comida rápida, en la cual trabajan 2 cocineros y 3 vendedores, y que debe atender a C clientes. El modelado debe considerar que:
#### - Cada cliente realiza un pedido y luego espera a que se lo entreguen.
#### - Los pedidos que hacen los clientes son tomados por cualquiera de los vendedores y se lo pasan a los cocineros para que realicen el plato. Cuando no hay pedidos para atender, los vendedores aprovechan para reponer un pack de bebidas de la heladera (tardan entre 1 y 3 minutos para hacer esto).
#### - Repetidamente cada cocinero toma un pedido pendiente dejado por los vendedores, lo cocina y se lo entrega directamente al cliente correspondiente. 
#### Nota: maximizar la concurrencia
```
chan PedidosCliente(int, text)
chan Entrega[C](Entrega)
chan PedidoCocinero(int, text)

chan Siguiente[3](int)
chan PedidoVendedor(int)

Process Cliente[id=0..C-1]{
    text mi_pedido=obtenerPedido()
    Entrega entrega;

    send Pedidos(id, mi_pedido)
    receive Entrega[id](entrega)
}


Process Coordinador{
    int idC, idV;
    text pedido;

    while(true){
        receive PedidoVendedor(idV)
        if(empty(PedidosCliente)){
            idC = -1;
            pedido = ""
        }
        else{
            receive PedidosCliente(idC, pedido)
        }
        send Siguiente[idV](idC, pedido)
    }
}


Process Vendedor[id=0..2]{
    int idC;
    text pedido;

    while(true){
        send PedidoVendedor(id)
        receive Siguiente[id](idC, pedido)
        if (idC <> -1) {
            send PedidoCocinero(idC, pedido)
        }
        else{
            //reponer heladera
            delay(rand(60,180))
        }
    }
}


Process Cocinero[id=0..1]{
    Entrega entrega;
    int idC;
    text pedido;

    while(true){
        receive PedidoCocinero(idC, pedido)
        entrega = //cocinar pedido cliente idC
        send Entrega[idC](entrega)
    }
}
```

### 4.
#### Simular la atención en un locutorio con 10 cabinas telefónicas, el cual tiene un empleado que se encarga de atender a N clientes. Al llegar, cada cliente espera hasta que el empleado le indique a qué cabina ir, la usa y luego se dirige al empleado para pagarle. El empleado atiende a los clientes en el orden en que hacen los pedidos, pero siempre dando prioridad a los que terminaron de usar la cabina. A cada cliente se le entrega un ticket factura. 
#### Nota: maximizar la concurrencia; suponga que hay una función Cobrar() llamada por el empleado que simula que el empleado le cobra al cliente.

```
chan Llegada(int);
chan CabinaAsignada[N](int);
chan Termine(int, int);
chan Factura[N](int, Factura);

Process Empleado{
    cola cabinas; //cargada
    int idC, cabina;
    Factura factura;

    while(true){
        if (not empty(Llegada) and not cola.empty() and 
            empty(Termine)) -->
                        receive Llegada(idC)
                        cabina = cola.pop()
                        send CabinaAsignada[idC](cabina)
            (not empty(Termine)) -->
                        receive Termine(idC, cabina)
                        factura = Cobrar(idC)
                        send Factura[idC](factura)
                        cola.push(cabina)
    }
}

Process Cliente[id=0..N-1]{
    int mi_cabina;
    Factura mi_factura;

    send Llegada(id)
    receive CabinaAsignada[id](mi_cabina)
    // Usar cabina
    sleep(rand())
    send Termine(id, mi_cabina)
    receive Factura[id](mi_factura)
}

```

### 5.
#### Resolver la administración de las impresoras de una oficina. Hay 3 impresoras, N usuarios y 1 director. Los usuarios y el director están continuamente trabajando y cada tanto envían documentos a imprimir. Cada impresora, cuando está libre, toma un documento y lo imprime, de acuerdo con el orden de llegada, pero siempre dando prioridad a los pedidos del director. 
#### Nota: los usuarios y el director no deben esperar a que se imprima el documento

```
chan PedidosDirector(text)
chan PedidosUsuario(text)
chan Señal()
chan Libre()

chan Documentos(text)

Process Admin{
    text doc;

    while(true){
        receive Libre()
        receive Señal()
        if        
            (not empty(PedidosDirector)) -->
                            receive PedidosDirector(doc)
            (not empty(PedidosUsuario) and empty(PedidosDirector)) -->
                            receive PedidosUsuario(doc)
        end if

        send Documentos(doc)
    }
}

Process Director{
    text doc;
    while(true){
        //TRABAJAR
        sleep(rand())
        doc = agarroDoc()
        send PedidosDirector(doc)
        send Señal()
    }
}


Process Usuario[id=0..N]{
    text doc;
    while(true){
        //TRABAJAR
        sleep(rand())
        doc = agarroDoc()
        send PedidosUsuario(doc)
        send Señal()
    }
}

Process Impresora[id=0..2]{
    text doc;
    while(true){
        receive Documentos(doc)
 