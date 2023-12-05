## Se requiere modelar un puente de un único sentido que soporta hasta 5 unidades de peso. El peso de los vehículos depende del tipo: cada auto pesa 1 unidad, cada camioneta pesa 2 unidades y cada camión 3 unidades. Suponga que hay una cantidad innumerable de vehículos (A autos, B camionetas y C camiones). Analice el problema y defina qué tareas, recursos y sincronizaciones serán necesarios/convenientes para resolver el problema. 
a. Realice la solución suponiendo que todos los vehículos tienen la misma prioridad.
b. Modifique la solución para que tengan mayor prioridad los camiones que el resto de los 
vehículos

```ada
Procedure PasarPuente is
    Task Type Auto;
    Task Type Camion;
    Task Type Camioneta;

    Task Puente is 
        Entry pasarAuto;
        Entry pasarCamion;
        Entry pasarCamioneta;
        Entry salirAuto;
        Entry salirCamion;
        Entry salirCamioneta;
    End Puente;

    arrAutos: array (1..A) of Auto;
    arrCamion: array (1..C) of Camion;
    arrCamioneta: array (1..B) of Camioneta;

    Task Body Auto is
    begin
        Puente.pasarAuto;
        //Pasa Por PUENTE
        Puente.salirAuto;
    End Auto;

    Task Body Camion is
    begin
        Puente.pasarCamion;
        //Pasa Por PUENTE
        Puente.salirCamion;
    End Camion;
    
    Task Body Camioneta is
    begin
        Puente.pasarCamioneta;
        //Pasa Por PUENTE
        Puente.salirCamioneta;
    End Camioneta;
    
    Task Body Puente is
        pasoMaximo: int;
    begin
        pesoMaximo:= 5;
        loop;
            select
                when (pesoMaximo -1) >= 0 => Accept pasarAuto;
                                            pesoMaximo -= 1;
            or
                when (pesoMaximo - 2) >= 0 => Accept pasarCamioneta;
                                            pesoMaximo -= 2;
            or
                when (pesoMaximo - 3) >= 0 => Accept pasarCamion;
                                            pesoMaximo -= 3;
            or
                Accept salirAuto;
                pesoMaximo += 1;
            or  
                Accept salirCamioneta;
                pesoMaximo += 2;
            or
                Accept salirCamion;
                pesoMaximo += 3;
            end select;
        end loop;
    End Puente;
Begin
    null;
End PasarPuente;
```

## 2. Se quiere modelar el funcionamiento de un banco, al cual llegan clientes que deben realizar un pago y retirar un comprobante. Existe un único empleado en el banco, el cual atiende de acuerdo con el orden de llegada. Los clientes llegan y si esperan más de 10 minutos se retiran sin realizar el pago.

```ada
Procedure Banco is
    Task Type Cliente;

    Task Empleado is
        Entry atender(pago: IN Pago, comp: OUT Comprobante)
    End Empleado;

    arrCliente: array (1..N) of Cliente;

    Task Body Cliente is
        pago: Pago;
        c: Comprobante;
    Begin
        Select 
            Empleado.atender(pago, c);
        or Delay 600
            null;
        End select;
    End;

    Task body Empleado is
    begin
        loop;
            Accept atender(pago: IN Pago, comp: OUT Comprobante) do
                comp := realizarPago(pago)
            End atender;
        end loop;
    end Empleado;
Begin

End Banco;
```