## 1. Suponga que existe un antivirus distribuido que se compone de R procesos robots Examinadores y 1 proceso Analizador. Los procesos Examinadores están buscando continuamente posibles sitios web infectados; cada vez que encuentran uno avisan la dirección y luego continúan buscando. El proceso Analizador se encarga de hacer todas las pruebas necesarias con cada uno de los sitios encontrados por los robots para determinar si están o no infectados. Implemente una solución con PMS.

```
Process Analizador {
    sitioWeb: string;

    while(true){
        Admin!dame_sitio();
        Admin?toma_sitio(sitioWeb);
        delay(rand()) //Analiza sitio web
    }
}

Process Examinador[id=1..R]{
    sitioWeb: string;
    while(True){
        sitioWeb = //Buscar sitio infectado;
        Admin!nuevo_sitio(sitioWeb);
    }
}

Process Admin{
    cola sitios : string;
    sitio: string;
    do 
        - ;Examinador[*]?nuevo_sitio(sitio) --> sitios.push(sitio);
        - !sitios.empty(); Analizador?dame_sitio() --> sitio = sitios.pop();
                                                        Analizador!toma_sitio(sitio)
    od;
}
```

## 2 En un laboratorio de genética veterinaria hay 3 empleados. El primero de ellos continuamente prepara las muestras de ADN; cada vez que termina, se la envía al segundo empleado y vuelve a su trabajo. El segundo empleado toma cada muestra de ADN preparada, arma el set de análisis que se deben realizar con ella y espera el resultado para archivarlo. Por último, el tercer empleado se encarga de realizar el análisis y devolverle el resultado al segundo empleado

```
Process Preparador{
    muestra: Muestra;

    while(true){
        muestra = prepararMuestra()
        Buffer!nueva_muestra(muestra)
    }
}

Process Buffer{
    cola muestras: Muestra;
    muestra: Muestra;
    do
        - !muestras.empty; Armador?quiero_muestra() --> muestra = muestras.pop()
                                                        Armador!toma_muestra(muestra)
        - Preparador?nueva_muestra(muestra) --> muestras.push(muestra);
    od
}

Process Armador{
    muestra: Muestra;
    set: Set;
    resultado: Resultado;
    while(true){
        Buffer!quiero_muestra()
        Buffer?toma_muestra(muestra)
        set = armarSet(muestra);
        Analizador!nuevo_set(set)
        Analizador?resultado(resultado);
        //Archivar resultado
    }
}

Process Analizador{
    set: Set;
    resultado: Resultado
    while(true){
        Armador?nuevo_set(set);
        resultado = //Anlizar Set
        Armador!resultado(resultado)
    }
}
```

## 3 En un examen final hay N alumnos y P profesores. Cada alumno resuelve su examen, lo entrega y espera a que alguno de los profesores lo corrija y le indique la nota. Los profesores corrigen los exámenes respetando el orden en que los alumnos van entregando. 
a) Considerando que P=1.

```
Process Alumno[id=1..N]{
    examen: Examen;
    nota: int;

    examen = resolverExamen();
    Admin!entrega_examen(id, examen);
    Profesor[*]?entrega_nota(nota)
}

Process Profesor{
    examen: Examen;
    nota: int;
    idA: int;

    for i:= 1 to N {
        Admin!quiero_examen()
        Admin?nuevo_examen(idA, examen)
        nota = corregirExamen(examen);
        Alumno[idA]!entrega_nota(nota);
    }
}

Process Admin{
    cola examenes: (int, Examen);
    examen: Examen;
    idA: int;

    do
        - ;Alumno[*]!entrega_examen(idA, examen) --> examenes.push(idA, examen);

        - !examenes.empty(); Profesor?quiero_examen() --> idA, examen = examenes.pop()
                                                          Profesor!nuevo_examen(idA, examen);
    od
}
```

b) Considerando que P>1.

```
Process Alumno[id=1..N]{
    examen: Examen;
    nota: int;

    examen = resolverExamen();
    Admin!entrega_examen(id, examen);
    Profesor[*]?entrega_nota(nota)
}

Process Profesor[id=1..P]{
    examen: Examen;
    nota: int;
    idA: int;

    for i:= 1 to N {
        Admin!quiero_examen(id)
        Admin?nuevo_examen(idA, examen)
        nota = corregirExamen(examen);
        Alumno[idA]!entrega_nota(nota);
    }
}

Process Admin{
    cola examenes: (int, Examen);
    idP: int;
    examen: Examen;
    idA: int;

    do
        - ;Alumno[*]!entrega_examen(idA, examen) --> examenes.push(idA, examen);

        - !examenes.empty(); Profesor[*]?quiero_examen(idP) --> idA, examen = examenes.pop()
                                                                Profesor[idP]!nuevo_examen(idA, examen);
    od
}
```

c) Ídem b) pero considerando que los alumnos no comienzan a realizar su examen hasta que todos hayan llegado al aula.
Nota: maximizar la concurrencia y no generar demora innecesaria.

```
Process Alumno[id=1..N]{
    examen: Examen;
    nota: int;

    Coordinador!llegada()
    Coordinador!empezar()
    examen = resolverExamen();
    Admin!entrega_examen(id, examen);
    Profesor[*]?entrega_nota(nota)
}

Process Coordinador{

    for i:= 1 to N {
        Alumno[*]?llegada()
    }
    for i:= 1 to N {
        Alumno[*]?empezar()
    }
}
Process Profesor[id=1..P]{
    examen: Examen;
    nota: int;
    idA: int;

    for i:= 1 to N {
        Admin!quiero_examen(id)
        Admin?nuevo_examen(idA, examen)
        nota = corregirExamen(examen);
        Alumno[idA]!entrega_nota(nota);
    }
}

Process Admin{
    cola examenes: (int, Examen);
    idP: int;
    examen: Examen;
    idA: int;

    do
        - ;Alumno[*]!entrega_examen(idA, examen) --> examenes.push(idA, examen);

        - !examenes.empty(); Profesor[*]?quiero_examen(idP) --> idA, examen = examenes.pop()
                                                                Profesor[idP]!nuevo_examen(idA, examen);
    od
}

```

## 4 En una exposición aeronáutica hay un simulador de vuelo (que debe ser usado con exclusión mutua) y un empleado encargado de administrar su uso. Hay P personas que esperan a que el empleado lo deje acceder al simulador, lo usa por un rato y se retira. El empleado deja usar el simulador a las personas respetando el orden de llegada. Nota: cada persona usa sólo una vez el simulador. 

```
Process Persona[id=1..P] {

    Admin!quiero_pasar(id)
    Empleado?puedo_pasar()
    // Usa simulador
    Empleado!termine()
}

Process Empleado{
    idP: int;
    Admin!quiero_siguiente();
    Admin?toma_siguiente(idP)
    Persona[idP]!puedo_pasar();
    Persona[idP]?termine();
}

Process Admin{
    cola personas: int;
    idP: int;

    do 
        - Persona[*]?quiero_pasar(idP) --> personas.push(idP);
        - !personas.empty(); Empleado?quiero_siguiente() --> idP = personas.pop()
                                                             Empleado!toma_siguiente(idP);
    od
}
```

## 5 En un estadio de fútbol hay una máquina expendedora de gaseosas que debe ser usada por E Espectadores de acuerdo al orden de llegada. Cuando el espectador accede a la máquina en su turno usa la máquina y luego se retira para dejar al siguiente. Nota: cada Espectador una sólo una vez la máquina

```

```