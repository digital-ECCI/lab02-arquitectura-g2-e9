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

### Módulo 2: Sumador completo de 4 bits (sum_compt)

Este módulo agrupa cuatro sumadores de 1 bit en cascada para procesar numeros de 4 bits. 

```verilog
module sum_compt(
input  [3:0] a_tb,
input  [3:0] b_tb,
input  ci_tb,
output [3:0] so_tb,
output co_tb
);

wire ci0, ci1, ci2;

sum uut(
.a(a_tb[0]),
.b(b_tb[0]),
.ci(ci_tb),
.so(so_tb[0]),
.co(ci0)
);

sum uut1(
.a(a_tb[1]),
.b(b_tb[1]),
.ci(ci0),
.so(so_tb[1]),
.co(ci1)
);

sum uut2(
.a(a_tb[2]),
.b(b_tb[2]),
.so(so_tb[2]),
.ci(ci1),
.co(ci2)
);

sum uut3(
.a(a_tb[3]),
.b(b_tb[3]),
.so(so_tb[3]),
.ci(ci2),
.co(co_tb)
);

endmodule
```
### Explicación del código: 

- `input [3:0] a_tb, output [3:0] so_tb`:Define los vectores de 4 bits para las entradas y la salida de suma.

- `wire ci0, ci1, ci2`: Declara cables lógicos.

- `sum uut(), sum uut1()`: Son las instanciaciones del módulo `sum` de 1 bit.

### Módulo 3: Sumador/Restador (restador.v)

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

- `input sel`: Se añade una nueva entrada de 1 bit que actúa como selector de operación (0 = suma y 1 = resta). 

- `sum_compt uut()`: Se instancia el sumador de 4 bits explicado en el paso anterior. 

- `.ci_tb(sel)`: Conecta el selector directamente al acarreo de entrada del sumador. Si es resta (sel = 1), suma un 1 inicial, lo cual es el primer paso del complemento a 2. 

- `.b_tb({4{sel}}^b_tb)`: Esta linea es muy importante. `{4{sel}}` esta parte replica el valor de `sel` cuatro veces. Luego aplica el operador XOR nivel de bits (`^`) con `b_tb`. 

  - Si `sel = 0` (suma): `0000 ^ b_tb` deja pasar a b_tb intacto. 

  - Si `sel = 1` (suma): `1111 ^ b_tb` (complemento a 1). Sumando al `ci_tb = 1` del paso anterior, se logra el complemento a 2. 

### Módulo 4: Módulo final (restador_sumador_completo.v)

Este módulo implementa la lógica para restar números utilizando el complemento a 2, controlada por una señal selectora. 

```verilog
module restador_sumador_completo(
input  [3:0] a_tb,
input  sel,
input [3:0] b_tb,
output [3:0] salida,
output co
);

 wire [3:0] b_tb1;
 wire co1,sel2;

restador uut(
.a_tb(a_tb), 
.b_tb(b_tb),     
.sel(sel),  
.so_tb(b_tb1), 
.co_tb(co1)
); 

restador uut1(
    .a_tb(4'b0000),       
    .sel(sel & (~co1)),   
    .b_tb(b_tb1),         
    .so_tb(salida),
    .co_tb(co)
);

endmodule
```

### Explicación del código

- `restador uut()`: Ejecuta la primera operación (A+B o A-B). El resultado se guarda en el cable temporal `b_tb1`.

- `.sel(sel & (~co1))`: Lógica de control, será `1` si la operación original es una resta (`sel = 1`). Y si el resultado dio negativo se arroja un acarreo `~co1` que invierte el 0 a 1. 


## 1.2 Diagramas

### Restador
El diagrama utiliza la señal `sel` para elegir entre suma (0) o resta (1). En la suma, las compuertas XOR dejan pasar el dato `b_tb` intacto al sumador central. En la resta, las XOR invierten `b_tb` (complemento a 1) y `sel` ingresa como acarreo inicial (`ci_tb=1`), completando el complemento a 2 necesario para entregar la resta correcta en `so_tb`.

### Sumador/restador completo

El diagrama implementa dos etapas para asegurar siempre un resultado positivo. El primer bloque (`uut`) ejecuta la operación inicial. Si se solicitó una resta y el resultado da negativo (indicado por un acarreo `co_tb = 0`), la compuerta AND central lo detecta y activa el segundo bloque (`uut1`). Este último invierte nuevamente el número en complemento a 2, entregando la magnitud absoluta final en `salida`.

## Simulaciones 

### 1. Simulación del sumador/restador completo

![sumres_completo_page-0001](https://github.com/user-attachments/assets/4c2c5f8b-01a7-4f2f-8f79-e28536ac6036)

#### 1.1 Descripción
El diagrama RTL generado ilustra una arquitectura de dos etapas diseñada para obtener la magnitud absoluta en operaciones aritméticas. El primer módulo (uut) ejecuta la operación principal entre las entradas, generando un resultado intermedio y una señal de acarreo (co_tb). Para gestionar adecuadamente los resultados negativos, una lógica combinacional intermedia (comb) evalúa dicha señal junto con el selector de operación (sel). Si se detecta una resta con resultado negativo, esta lógica activa el segundo módulo (uut1), el cual corrige el valor intermedio aplicando el complemento a dos para entregar únicamente la magnitud absoluta en la salida final (salida).
#### 1.2 Diagrama restador

![sumres_02_page-0001](https://github.com/user-attachments/assets/6931e027-f92b-435a-844b-80e8db144c24)
"El diagrama RTL muestra la etapa de entrada del sumador/restador. Utiliza compuertas XOR y la señal de control (sel) como acarreo inicial (ci_tb) para invertir el sustraendo y sumarle 1. Esto implementa por hardware el complemento a dos ($A + \bar{B} + 1$), permitiendo realizar restas reutilizando el mismo módulo sumador."

#### 1.3 simulacion_tb.v

<img width="1918" height="638" alt="simulacion_TB" src="https://github.com/user-attachments/assets/0d168afd-dd1c-466c-890a-a850e01c7cc5" />

La imagen muestra la verificación funcional de un módulo restador/sumador de 4 bits. El enfoque principal de este segmento de la simulación es validar la operación de resta mediante el barrido de valores en las entradas.
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

