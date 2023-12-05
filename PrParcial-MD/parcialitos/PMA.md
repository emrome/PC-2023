# Resolver con PMA (Pasaje de Mensajes ASINCRÓNICOS) el siguiente problema. 
## Simular la atención en un locutorio con 10 cabinas telefónicas, que tiene un empleado que se encarga de atender a los clientes. Hay N clientes que al llegar esperan hasta que el empleado les indica a que cabina ir, la usan y luego se dirigen al empleado para pagarle. El empleado atiende a los clientes en el orden en que hacen los pedidos, pero siempre dando prioridad a los que terminaron de usar la cabina. Nota: maximizar la concurrencia; suponga que hay una función Cobrar() llamada por el empleado que simula que el empleado le cobra al cliente.

```
chan pedidoCabina(int)
chan asginarCabina[N](int);
chan pedidoPagar(int, int);
chan listo[N];

Process Cliente[id=1..N]{
    cabina: int;

    send pedidoCabina(id);
    receive asignarCabina[id](cabina)
    //USAR CABINA
    send pedidoPagar(cabina, id);
    recieve listo[id]()
}

Process Empleado{
    cabina: int;
    idC: int;
    cola cabinas(int) //inicalizada con 10 cabinas;

    for i:= 1 to N {
        if 
            - not empty(pedidoCabina) and !cabinas.empty() and empty(pedidoPagar)--> receive pedidoCabina(idC)
                                                                                    send asignarCabina[idC](cabinas.pop())
            - not empty(pedidoPagar) --> receive pedidoPagar(cabina, idC)
                                         cabinas.push(cabina)
                                         Cobrar()
                                         send listo[idC]()
        fi
    }

}
```