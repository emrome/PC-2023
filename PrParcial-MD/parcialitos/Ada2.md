```
Procedure VentaEntradas is
    Task Type Cliente;

    Task Portal is
        Entry pedidoRegular(comp: OUT Comprobante);
        Entry pedidoPrioritario(comp: OUT Comprobante);
    End Portal;

    arrClientes: array (1..N) of Cliente;

    Task Body Cliente is
        c: Comprobante
        listo: bool;
    Begin
        listo := false
        if ("es prioritario") {
            Portal.pedidoPrioritario(c);
        }
        else {
            while(listo) loop
                Select 
                    Portal.pedidoRegular(c)
                    listo:= true;
                Or Delay 300
                    null;
                End select:
            End loop;
        }
        if (c != null) then
            Imprimir(c);
        end if;
    End Cliente;

    Task Body Portal is
        entradas: int;
    Begin
        entradas:=E;
        for i in (1..N) loop
            Select
                Accept pedidoPrioritario(comp:OUT Comprobante) do
                    if(entradas>0){
                        entradas--;
                        comp:= generarComp()
                    }
                    else{
                        comp:=null;
                    }
                End pedidoPrioritario;
            or
                when pedidoPrioritario`count = 0 => Accept pedidoRegular(comp:OUT Comprobante) do
                    if(entradas>0){
                        entradas--;
                        comp:= generarComp()
                    }
                    else{
                        comp:=null;
                    }
                End pedidoRegular:
            End select;
        end loop;
    End Portal;
Begin
    null;
End VentaEntradas;
```