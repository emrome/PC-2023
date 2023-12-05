```
Procedure Parcial is
    Task Type Estudiante;

    Task Programador is
        Entry errorUrgente(error: IN text);
        Entry errorImportante(error: IN text);
        Entry errorSecundario(error: IN text);
    End Programador;

    arrEstudiantes: array (1..5) of Estudiante;

    Task Body Estudiante is
        error: text;
    Begin
        for i in 1..10 loop
            error:= // Busca error
            if("es urgente") then
                Programador.errorUrgente(error);
            else if ("es importante") then 
                Programador.errorImportante(error);
            else
                Programador.errorSecundario(error);
            end if;
        end loop;
    End Estudiante;

    Task Body Programador is
        cantErrores: int :=0;
    Begin
        while (cantErrores < 50) loop
            Select 
                Accept errorUrgente(error: IN text) do
                    //Resolver error
                    delay(rand())
                End errorUrgente;
                cantErrores++;
            or
                When errorUrgente`count = 0 => Accept errorImportante(error: IN text) do
                                                    //Resolver error
                                                    delay(rand())
                                                End errorImportante;
                                                cantErrores++;
            or
                When (errorUrgente`count = 0) and (errorImportante`count = 0) => Accept errorSecundario(error: IN text) do
                                                                                    //Resolver error
                                                                                    delay(rand())
                                                                                End errorSecundario;
                                                                                cantErrores++;
            else
                delay(300) //Trabaja en nuevo sistema;
            End select;
        end loop;
    End Programador
Begin
    null;
End Parcial;
```