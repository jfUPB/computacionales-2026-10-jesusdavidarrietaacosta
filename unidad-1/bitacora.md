<img width="1910" height="848" alt="image" src="https://github.com/user-attachments/assets/e743b2f6-abaf-4770-9371-e1be90788e7e" /># Unidad 1

## Bitácora de proceso de aprendizaje


## Bitácora de aplicación 

### actividad 2
1. M=M+1
2. indicar qué instrucción de la ROM se va a ejecutar.
3. ambas se usan para guardar datos pero la "I" guarda los datos y @READKEYBOARD Se usa para saltar a esa parte del código
4. se necista utilzar @KBD para el teclado y @SCREEN para la pantalla
5.  
@i
D=M
@KBD
D=D-A
@READKEYBOARD
D;JGE

### Actividad 4
@12
M=0        

@i
M=1        

(LOOP)
@i
D=M
@6
D=D-A     
@END
D;JGE     

@i
D=M
@12
M=D+M    

@i
M=M+1     

@LOOP
0;JMP     

(END)
@END
0;JMP

### Actividad 5
@12
M=0

@55
D=A
@i
M=D      

(LOOP)
@i
D=M
@66
D=D-A
@END
D;JGE

@i
D=M
@12
M=D+M

@i
M=M+1

@LOOP
0;JMP

(END)
@END
0;JMP


## Bitácora de reflexión

