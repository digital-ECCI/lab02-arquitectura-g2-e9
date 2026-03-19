[sumres_completo.pdf](https://github.com/user-attachments/files/25830693/sumres_completo.pdf)[![Review Assignment Due Date](https://classroom.github.com/assets/deadline-readme-button-22041afd0340ce965d47ae6ef1cefeee28c7c493a6346c4f15d667ab976d596c.svg)](https://classroom.github.com/a/Px-uYaj2)
[![Open in Visual Studio Code](https://classroom.github.com/assets/open-in-vscode-2e0aaae1b6195c2367325f4f02e2d04e9abb55f0b24a779b69b11b9e10269abc.svg)](https://classroom.github.com/online_ide?assignment_repo_id=22918397&assignment_repo_type=AssignmentRepo)
# Lab02 - Sumador/Restador de 4 bits

# Integrantes
<p align="center">
 SAMUEL ALEJANDRO TINOCO PINEDA 
<p align="center">
CRISTIAN EDUARDO HUERTAS MORENO
<p align="center">
SARA GERALDINE MOSCOSO MOGOLLON


# Informe

Indice:

1. [Documentación](#documentación-de-los-circuitos-implementados-implementado)
2. [Simulaciones](#simulaciones)
3. [Evidencias de implementación](#evidencias-de-implementación)
4. [Preguntas](#preguntas)
5. [Conclusiones](#conclusiones)
6. [Referencias](#referencias)

### Documentación del diseño implementado

# 1. Sumador/Restador 
## 1.1 Descripción de módulos y código

El sistema implementa módulos que forman un sumador/restador de 4 bits capaz de resolver números negativos y devolver el valor. A continuación, se detalla el código y el funcionamiento linea por linea de cada uno de los módulos que componen el sumador/restador. 

---

### Módulo 1: Sumador de 1 bit (sum.v)

Este es el bloque principial del diseño. Calcula la suma de dos bits individuales considerando un acarreo de entrada. 

```verilog
module sum (
input a, input b, input ci, output so,output co
);

assign so = ci^(a^b);
assign co = (ci&(a^b))|(a&b);

endmodule
```
### Explicación del código: 

- `module sum`: Se define el nombre del módulo, las entradas de 1 bit (`a`, `b`,`ci` que es el acarreo de entrada) y dos salidas de 1 bit (`so` para la suma, `co` para el acarreo de salida).

- `assign so = ci^(a^b)`: La asignación define la suma con un operador `^` XOR. 

- `assign co = (ci&(a^b))|(a&b)`: La asignación define el acarreo de salida. Se activa si al menos dos de las tres entradas son un "1" lógico. 


### Módulo 2: Sumador/Restador (restador.v)

Este módulo implementa la lógica necesaria para restar números utilizando el complemento a 2, controlada por una señal selectora. 

```verilog
module restador(
input  [3:0] a_tb,
input  sel,
input [3:0] b_tb,
output [3:0] so_tb,
output co_tb
);

sum_compt uut(
.a_tb(a_tb),    
.ci_tb(sel),
.b_tb({4{sel}}^b_tb),
.so_tb(so_tb),
.co_tb(co_tb)
);

endmodule
```
###Explicación del coódigo: 

Este diseño es un "2 por 1": usa un sumador para hacer restas aplicando un truco matemático. Así funciona cada parte:

`input sel`: Es el interruptor maestro. Si vale 0, el circuito suma. Si vale 1, el circuito resta.

`sum_compt uut():` Es el motor del circuito. Reutilizamos el sumador de 4 bits que ya teníamos construido.

`.ci_tb(sel):` Conectamos el selector al acarreo de entrada.

Si vamos a restar (sel = 1), este "1" entra al sumador. Es el primer paso para convertir un número en negativo.

`.b_tb({4{sel}}^b_tb):` Esta es la "magia" del circuito. El símbolo ^ es una compuerta XOR:

`Si sel = 0 (Suma):` El número b_tb pasa tal cual, sin cambios.

`Si sel = 1 (Resta):` El número b_tb se invierte por completo (los 0 se vuelven 1 y viceversa).

En resumen: Cuando quieres restar, el circuito invierte los bits de b_tb y le suma un 1 (gracias al ci_tb). Matemáticamente, esto convierte la suma en una resta usando el método de complemento a 2.

## 1.2 Diagramas

### Restador
El diagrama utiliza la señal `sel` para elegir entre suma (0) o resta (1). En la suma, las compuertas XOR dejan pasar el dato `b_tb` intacto al sumador central. En la resta, las XOR invierten `b_tb` (complemento a 1) y `sel` ingresa como acarreo inicial (`ci_tb=1`), completando el complemento a 2 necesario para entregar la resta correcta en `so_tb`.

## Simulaciones 

### 1. Simulación del sumador/restador completo

![sumres_completo_page-0001](https://github.com/user-attachments/assets/4c2c5f8b-01a7-4f2f-8f79-e28536ac6036)

#### 1.1 Descripción
El diagrama RTL generado ilustra una arquitectura de dos etapas. El primer módulo (uut) ejecuta la operación principal entre las entradas, generando un resultado intermedio y una señal de acarreo (co_tb). Para gestionar adecuadamente los resultados negativos, evalúa dicha señal junto con el selector de operación (sel). Si se detecta una resta con resultado negativo, esta lógica activa el segundo módulo (uut1), el cual corrige el valor intermedio aplicando el complemento a dos para entregar únicamente la magnitud absoluta en la salida final (salida).
#### 1.2 Diagrama restador

![sumres_02_page-0001](https://github.com/user-attachments/assets/6931e027-f92b-435a-844b-80e8db144c24)
"El diagrama RTL muestra la etapa de entrada del sumador/restador. Utiliza compuertas XOR y la señal de control (sel) como acarreo inicial (ci_tb) para invertir el sustraendo y sumarle 1. Esto implementa por hardware el complemento a dos  permitiendo realizar restas reutilizando el mismo módulo sumador."

#### 1.3 simulacion_tb.v

<img width="1643" height="345" alt="Captura de pantalla 2026-03-19 011941" src="https://github.com/user-attachments/assets/7aff991f-0d59-47ae-b4d9-a9555d2d6000" />


La imagen muestra la verificación funcional de un módulo restador/sumador. El enfoque principal de este segmento de la simulación es validar la operación de resta mediante el barrido de valores en las entradas.
Comportamiento en la Región de "Resta Negativa"
Interpretación: La transición de la salida hacia valores como F, E y D confirma que el diseño maneja correctamente el desbordamiento (underflow) y representa los números negativos de forma estándar para sistemas digitales.
## Evidencias de implementación

https://github.com/user-attachments/assets/88ddf45a-61c7-49ad-ba85-487ef2f14ef1


## Conclusiones
En conclusión, se validó con éxito el diseño del restador de 4 bits, confirmando que la lógica aritmética responde de forma precisa en todo el rango de valores. Las pruebas en GTKWave demostraron un manejo correcto del complemento a dos, comportamiento que fue ratificado mediante la implementación física en la FPGA, donde el sistema operó correctamente en hardware real, cumpliendo con todos los requisitos de diseño

## Referencias
J. I. Odriozola, “Sumador Restador,” CircuitVerse, 2024.
[Online]. Available: https://circuitverse.org/users/237192/projects/sumador-restador-e2329d28-c542-4b2a-9111-91a759e6554c

Saber 360°, “Somadores.” [Online]. Available:
https://www.saber360.com.br/es/somadores

