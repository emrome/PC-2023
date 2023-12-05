```
chan pedidos(Pedido, int)
chan entregas[C](Entrega)
chan siguiente;
chan sig_pedido[V](int, Pedido)

Process Cliente[id=1..C]{
    pedido: Pedido;
    entrega: Entrega;

    pedido = armarPedido()
    send pedidos(pedido, ic);
    receive entregas[id](entrega);
}

Process Vendedores[id=1..V]{
    idC: int;
    pedido: Pedido;
    entrega: Entrega;

    while true {
        send siguiente(id);
        receive sig_pedido[id](idC, pedido);
        if(idC = -1){
            //CONTROLAN STOCK
            delay(rand(2..4))
        }
        else {
            entrega = armarEntrega(pedido);
            send entregas[idC](entrega);
        }
    }
}

Process Admin{
    pedido: Pedido;
    idC,idV: int;

    while true{
        receive siguiente(idV)
        if(empty(pedidos)){
            idC=-1;
            pedido=null;
        }
        else{
            receive pedidos(pedido, idC)
        }
        send sig_pedido[idV](idC, pedido)
    }

}
```