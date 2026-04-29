# Unidad 7
## Bitácora de aplicación 

Evidencia 1
Glfw crea el contexto de opengl para que glad pueda utilizar las funciones de opengl, glad depende del contexto para cargar dinamicamente las funciones de opengl atravez del driver grafico

glwfw: maneja el input, crea la ventana y genera el contexto
glad: carga las funciones de opengl en tiempo de ejecucion

Evidencia 2
<img width="1365" height="767" alt="image" src="https://github.com/user-attachments/assets/1340546b-20fc-4c4a-bd33-cc39312e4abb" />
En la captura se observa que la ejecución se detiene justo antes de la llamada a glDrawArrays, momento en el cual el pipeline gráfico ya ha sido completamente configurado.
Se evidencia que un VAO válido (VAO = 1) está activo, lo que implica que contiene la configuración de los atributos de vértice previamente definida mediante glVertexAttribPointer y glEnableVertexAttribArray.
Dado que el shader utiliza layout(location = 0) in vec3 aPos, y que el renderizado del triángulo ocurre correctamente, se concluye que el atributo 0 está siendo alimentado desde el VBO asociado al VAO.
En el perfil core de OpenGL, este flujo es obligatorio para que glDrawArrays funcione, lo que constituye evidencia indirecta pero válida de que los datos del arreglo de vértices están llegando al vertex shader.

Evidencia 3

Antes de glDrawArrays, se observa que las variables x y y cambian entre iteraciones, mientras que el VAO y el VBO permanecen constantes, ya que no se ejecuta ninguna llamada a glBufferData. Estos valores se envían al shader como uniforms mediante glUniform4f y glUniform2f, modificando el color (ourColor) y la posición (offset) del triángulo. Esto demuestra que el cambio visual no proviene de los datos de vértice, sino de los uniforms, lo cual es posible porque estos son variables globales del shader independientes del VBO y pueden 
actualizarse en cada draw call sin alterar la geometría.

Evidencia 4
<img width="1365" height="767" alt="image" src="https://github.com/user-attachments/assets/6811e6c7-9fc8-45fd-bea7-e663ccfcf361" />

Se cambio glUniform4f(colorLocation, x, y, 0.0f, 1.0f);
Por: glUniform4f(colorLocation, 5.0f, -2.0f, 10.0f, 1.0f);


Evnie un color extremo, por un lado esperaba que el color empezaria a cambiar de maneras extrañas o en la gama del color seleccionado pero al final no paso nada de eso, lo que paso fue que se envío valores fuera del rango estándar [0,1] esto genra que al renderizar, OpenGL ajusta (clampa) estos valores al rango válido del framebuffer, produciendo un color saturado (por ejemplo, magenta) evitando que cambiara de color
esto conclutye que el resultado visual en OpenGL no depende únicamente de los datos almacenados en el VBO, sino también de los uniforms enviados al shader. A través del depurador y la modificación de valores como ourColor y offset, se evidencia que es posible cambiar el color y la posición del objeto sin alterar la geometría. Esto es posible porque los uniforms son variables globales del shader que se actualizan en cada draw call, permitiendo un control dinámico y eficiente del renderizado sin necesidad de modificar los datos de vértices.

Evidencia 5
se escoge el calculo de la posicion del mouse 

float x = (float)xpos / SCR_WIDTH;
float y = (float)ypos / SCR_HEIGHT;

Este cálculo se realiza en C++ y no en GLSL porque depende de datos externos al pipeline gráfico, en este caso la posición del mouse obtenida mediante glfwGetCursorPos. GLSL (el shader) no tiene acceso directo a dispositivos de entrada ni al sistema operativo, por lo que no puede obtener esta información por sí mismo.Además, este valor solo necesita calcularse una vez por frame, por lo que es más eficiente hacerlo en la CPU y enviarlo como uniform al shader, en lugar de recalcularlo para cada vértice o fragmento en la GPU.En conclusión, los cálculos que dependen de entrada del usuario o lógica externa se realizan en C++, mientras que GLSL se utiliza para operaciones masivas y paralelas sobre los datos ya enviados al pipeline gráfico.



## Bitácora de reflexión
