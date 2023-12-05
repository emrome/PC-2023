```
Process Enfermera{
    idC, dni: int;
    comp: Comprobandte
    for i:= 1 to 25 {
        Admin!siguiente();
        Admin?vacunar(idC)
        Persona[idC]?entregoDNI(dni)
        //Vacuna Persona
        comp:=generarComprobante(idC)
        Persona[idC]!comprobante(comp)
    }
}

Process Admin{
    cola personas: int;
    idC: int;
    for i:= 1 to 50{
        if 
            - !personas.empty();Enfermera?siguiente() --> Enfermera!vacunar(personas.pop())
            - Persona[*]?atender(idC) --> personas.push(idC)
        fi
    }
}

Process Persona[id=1..25]{
    comp: Comprobante;
    dni: int;

    Admin!atender(id);
    Enfermera!entregoDNI(dni)
    Enfermera?comprobante(comp);

    Voluntario!llegada();
    Voluntario!me_quiero_ir();
}

Process Voluntario{

    for i:=1 to 25{
        Voluntario[*]?llegada()
    }

    delay(600) //10 minutos de la charla

    for i:=1 to 25{
        Voluntario[*]?me_quiero_ir()
    }
}
```