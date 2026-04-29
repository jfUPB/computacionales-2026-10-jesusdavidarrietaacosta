# Unidad 2

## Bitácora de proceso de aprendizaje


## Bitácora de aplicación 

```asm
    @10
    D=A
    @17
    M=D     

    @20
    D=A
    @18
    M=D      

   
    @17
    D=A
    @R0
    M=D

    
    @18
    D=A
    @R1
    M=D

  
    @R0
    A=M
    D=M
    @R13
    M=D

  
    @R1
    A=M
    D=M
    @R0
    A=M
    M=D


    @R13
    D=M
    @R1
    A=M
    M=D

(END)
    @END
    0;JMP
```

Este programa implementa un intercambio de valores usando punteros en ensamblador Hack. Primero se inicializan dos posiciones de memoria: RAM[17] con el valor 10 y RAM[18] con el valor 20. Luego se cargan esas direcciones en los registros R0 y R1, que actúan como punteros: R0 contiene la dirección 17 y R1 la dirección 18. A partir de ese momento, el programa ya no trabaja directamente con @17 ni @18 para el intercambio, sino que accede a los datos de forma indirecta usando los punteros, mediante instrucciones como @R0 A=M y @R1 A=M, que permiten leer o escribir en la dirección almacenada en esos registros.En la parte del swap, el valor apuntado por R0 se lee y se guarda en R13 como variable temporal. Luego se lee el valor apuntado por R1 y se copia en la dirección apuntada por R0. Finalmente, el valor almacenado en R13 se copia en la dirección apuntada por R1. De esta manera se logra intercambiar los contenidos de RAM[17] y RAM[18] sin perder información. El programa termina con un bucle infinito para evitar que siga ejecutando instrucciones, y el resultado final es que los valores en esas dos posiciones de memoria quedan intercambiados.

<img width="1914" height="958" alt="Captura de pantalla 2026-02-11 171812" src="https://github.com/user-attachments/assets/a609ef57-1d9d-4787-874b-2b138cd4c5bf" />

Explicacióm (arriba)


<img width="1913" height="955" alt="Captura de pantalla 2026-02-11 171859" src="https://github.com/user-attachments/assets/ac9fc31c-357e-49fe-8424-7d4695dabd87" />


Explicacióm (arriba)



## Bitácora de reflexión


