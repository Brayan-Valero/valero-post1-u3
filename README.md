# Post-contenido — Manejo del DEBUG: Exploración, Ensamblado y Ejecución Paso a Paso en DOSBox

**Asignatura:** Arquitectura de Computadores  
**Unidad:** 3  
**Actividad:** Post-contenido — Manejo del DEBUG  
**Repositorio:** `valero-post1-u3`

> \\\*\\\*Nota de integridad académica:\\\*\\\* este repositorio contiene el desarrollo técnico, las codificaciones calculadas, las tablas de traza y las instrucciones para reproducir el laboratorio. Las capturas reales de DOSBox deben incorporarse después de ejecutar los comandos en el equipo. No se incluyen imágenes simuladas ni se afirma que una ejecución local haya sido realizada.

\---

## 1\. Introducción

El laboratorio estudia el funcionamiento del depurador DEBUG en DOSBox mediante la inspección y modificación de registros y memoria, el ensamblado de instrucciones 8086 y la ejecución controlada de pequeños programas. La primera parte aborda los comandos `R`, `F`, `D`, `E`, `A`, `U` y `T`; la segunda utiliza `A`, `U`, `D`, `T`, `LOOP`, `DEC`, `JNZ` y `G` para comparar dos mecanismos de repetición.

El trabajo se organiza alrededor de los checkpoints definidos en la guía. Las tablas muestran el estado posterior a cada instrucción ejecutada. Los valores de banderas se presentan únicamente cuando pueden determinarse a partir de la operación realizada o del estado inicial indicado; cuando una instrucción conserva las banderas, se señala expresamente.

## 2\. Revisión técnica de la guía y convenciones

La revisión de las direcciones y codificaciones conduce a las siguientes precisiones:

1. **Programa LOOP:** sus bytes son `B8 00 00 B9 04 00 05 02 00 E2 FB CD 20`, que suman **13 bytes**. La guía indica `D CS:100 L0C` y afirma que son 12 bytes, pero `L0C` solicita solamente 12 bytes y omite el último byte (`20`) de `INT 20`. Para visualizarlo completo se debe utilizar `D CS:0100 L0D`.
2. **Direcciones LOOP:** `ADD AX,0002` comienza en `0106`, `LOOP 0106` en `0109` y `INT 20` en `010B`. El desplazamiento de `LOOP` se calcula desde la dirección posterior a la instrucción (`010B`): `0106 - 010B = -5 = FB`.
3. **Programa DEC/JNZ:** ocupa 13 bytes. `JNZ` comienza en `020A`, su dirección posterior es `020C` y el destino es `0206`; por tanto, el desplazamiento es `0206 - 020C = -6 = FA`.
4. **PSP:** ocupa los desplazamientos `0000` a `00FF`. `0100` es el punto de entrada habitual de un programa COM y no pertenece al PSP. Los contenidos observados en un volcado pueden depender del entorno.
5. **Capturas:** la guía habla de ocho checkpoints, pero especifica siete nombres de archivos de imagen. Se conservan los siete nombres explícitos; la revisión final del repositorio constituye el checkpoint restante y no requiere inventar una octava captura.
6. **Contador de instrucciones:** LOOP ejecuta 11 instrucciones en total para cuatro iteraciones: dos inicializaciones, cuatro `ADD`, cuatro `LOOP` y una terminación. DEC/JNZ ejecuta 15: dos inicializaciones, cuatro `ADD`, cuatro `DEC`, cuatro `JNZ` y una terminación.

### Convenciones de ejecución

* Los valores se expresan en hexadecimal de 16 bits, salvo indicación contraria.
* Las direcciones son desplazamientos dentro del segmento indicado.
* Los bytes de palabras de 16 bits se almacenan en formato **little-endian**: primero el byte menos significativo.
* `IP siguiente` indica el valor de IP después de ejecutar la instrucción de la fila.
* En las tablas de bucles, cada fila representa una instrucción realmente ejecutada, no una iteración resumida.
* Las tablas son trazas calculadas a partir de la codificación 8086 y de los estados iniciales de referencia de la guía. Deben contrastarse con la salida real de DOSBox al tomar las capturas.

\---

# PARTE 1 — EXPLORACIÓN CON DEBUG EN DOSBox

## 3\. Configuración del entorno

La carpeta anfitriona `C:\\\\DOSWork` debe existir antes de iniciar DOSBox. En el prompt inicial se ejecutan:

```text
Z:\\\\> MOUNT C C:\\\\DOSWork
Z:\\\\> C:
C:\\\\> MD LAB3POST1
C:\\\\> CD LAB3POST1
C:\\\\LAB3POST1> DEBUG
-
```

`MOUNT` asocia una carpeta del sistema anfitrión con una unidad virtual de DOSBox. `C:` cambia a la unidad montada; `MD` crea el directorio del laboratorio; `CD` ingresa en él; y `DEBUG` inicia el depurador.

## 4\. Registros: comando R

### 4.1 Inspección del estado inicial

En el prompt `-` se ejecuta:

```text
-R
```

La guía presenta como referencia una salida semejante a:

```text
AX=0000  BX=0000  CX=0000  DX=0000  SP=FFFE  BP=0000  SI=0000  DI=0000
DS=1357  ES=1357  SS=1357  CS=1357  IP=0100   NV UP EI PL NZ NA PO NC
1357:0100 CD20          INT 20
```

`R` sin argumentos presenta los registros generales, los registros de segmento, `IP`, los indicadores de estado y la instrucción situada en `CS:IP`. Los valores concretos de segmento y banderas deben copiarse de la sesión real, ya que pueden variar según la versión y el estado de DEBUG.

### 4.2 Modificación de AX

```text
-R AX
AX 0000
:1234
-R AX
```

El depurador solicita el nuevo contenido de AX. El valor `1234h` equivale a 4660 decimal. La segunda consulta permite comprobar que AX contiene `1234` y que la modificación se limitó al registro seleccionado.

### 4.3 Función de R

`R` inspecciona el estado de los registros y permite editar un registro cuando se especifica su nombre. No es un comando para visualizar bloques de memoria ni para rellenarlos.

## 5\. Memoria: comandos F y D

### 5.1 Relleno de 64 bytes

```text
-F 200 L40 AB CD EF
```

`F` (*Fill*) escribe un patrón repetitivo en el rango indicado. `L40` representa 40h bytes, es decir, 64 bytes. En DEBUG, la dirección sin prefijo de segmento utiliza DS como segmento predeterminado. El patrón `AB CD EF` se repite hasta completar el rango desde `DS:0200`.

### 5.2 Verificación con D

```text
-D 200 L40
```

`D` (*Dump*) presenta el contenido de memoria en hexadecimal y una representación ASCII. El patrón `AB CD EF` debe repetirse en las cuatro filas de 16 bytes.

**Interpretación de las columnas del volcado D:**

1. La primera columna contiene el segmento y el desplazamiento de inicio de la fila, por ejemplo `1357:0200`.
2. La zona central presenta los bytes de memoria en hexadecimal, en el orden en que están almacenados.
3. El guion que puede separar grupos de bytes es un separador visual, no un byte adicional.
4. La columna derecha representa esos mismos bytes como caracteres ASCII cuando son imprimibles; los bytes no imprimibles aparecen como puntos.
5. Cada fila estándar contiene 16 bytes, por lo que cuatro filas representan 64 bytes.

### 5.3 Exploración del PSP

```text
-D 0 L20
```

El PSP (*Program Segment Prefix*) ocupa los primeros 100h bytes del segmento del proceso, desde `0000` hasta `00FF`. Sus primeros bytes contienen `CD 20`, la instrucción `INT 20` utilizada históricamente como mecanismo de terminación de programas COM. Los restantes campos contienen información de control preparada por DOS, por lo que no deben interpretarse como código ejecutable de manera indiscriminada. Los valores concretos del volcado pueden variar.

## 6\. Ensamblado y desensamblado: comandos A y U

### 6.1 Ensamblado en CS:0100

```text
-A 100
MOV AX,0005
MOV BX,0003
ADD AX,BX
INT 20

-U 100 109
```

La secuencia esperada es:

|Dirección|Bytes|Instrucción|Tamaño|
|-|-|-|-:|
|0100|B8 05 00|MOV AX,0005|3|
|0103|BB 03 00|MOV BX,0003|3|
|0106|03 C3|ADD AX,BX|2|
|0108|CD 20|INT 20|2|
|**Total**|||**10 bytes**|

### 6.2 Análisis de las codificaciones

* `B8 05 00`: `B8` identifica `MOV AX,imm16`; los dos bytes siguientes codifican el inmediato `0005h` en little-endian (`05 00`).
* `BB 03 00`: `BB` identifica `MOV BX,imm16`; `03 00` representa `0003h`.
* `03 C3`: codifica `ADD AX,BX`. El byte de operación `03` corresponde a la forma `ADD r16,r/m16`; el byte ModR/M `C3` selecciona AX como destino y BX como fuente.
* `CD 20`: `CD` corresponde a la instrucción `INT imm8` y `20` identifica la interrupción DOS 20h.

`A` (*Assemble*) convierte mnemónicos en bytes y los deposita en memoria. `U` (*Unassemble*) interpreta bytes almacenados y presenta las instrucciones correspondientes. `U` no ejecuta el código.

## 7\. Modificación puntual de memoria: comando E

### 7.1 Inicialización y escritura

```text
-F 300 L10 00
-D 300 L10
-E 300 78 56
-D 300 L10
```

`L10` representa 10h, es decir, 16 bytes. Primero se inicializa la región con ceros; después `E` (*Enter*) escribe `78` en `DS:0300` y `56` en `DS:0301`. El segundo `D` permite comprobar que los otros 14 bytes continúan en cero.

### 7.2 Interpretación little-endian

Los bytes consecutivos `78 56` representan la palabra de 16 bits `5678h` cuando se interpretan en little-endian. El byte menos significativo (`78`) ocupa la dirección menor (`0300`) y el más significativo (`56`) la dirección siguiente (`0301`).

## 8\. Decisión técnica 1 — Verificación no destructiva de una escritura en memoria

La verificación de una escritura debe realizarse mediante `D`, porque este comando presenta el contenido de una región de memoria sin modificarlo. Después de escribir `78 56` con `E`, el volcado permite comprobar tanto los dos bytes alterados como la conservación de los catorce bytes restantes. Aunque `E` también permite consultar el contenido cuando se invoca sin una lista de valores, su modalidad interactiva deja al operador frente a una secuencia de bytes editables; una pulsación accidental puede sobrescribir un dato y convertir una comprobación en una modificación. `F` tampoco es apropiado, porque su finalidad es escribir un patrón repetido sobre un rango, no inspeccionar el contenido existente. `R` se concentra en registros y no verifica el bloque de memoria. Por tanto, `D` es la opción adecuada: es una operación de lectura, ofrece una representación hexadecimal verificable y no introduce cambios adicionales en el estado que se pretende comprobar.

## 9\. Direccionamiento directo a memoria

### 9.1 Ensamblado en una dirección separada

Para conservar el programa anterior se utiliza otra ubicación:

```text
-A 320
MOV AX,\\\[0300]
INT 20

-U 320 324
```

Codificación esperada:

|Dirección|Bytes|Instrucción|Tamaño|
|-|-|-|-:|
|0320|A1 00 03|MOV AX,\[0300]|3|
|0323|CD 20|INT 20|2|
|**Total**|||**5 bytes**|

`A1` codifica `MOV AX,moffs16`, es decir, la lectura de una palabra desde una dirección de memoria directa. `00 03` representa el desplazamiento `0300h` en little-endian. Para que la lectura corresponda al dato escrito con `E`, DS debe apuntar al mismo segmento en el que se realizó la escritura.

### 9.2 Ejecución controlada

Se comprueba que DS corresponde al segmento utilizado para la escritura y se coloca IP en `0320`:

```text
-R IP
:0320
-T
-R AX
```

Después de ejecutar `MOV AX,\\\[0300]`, el valor esperado es `AX=5678`. La instrucción `INT 20` no necesita ejecutarse para verificar la carga.

## 10\. Decisión técnica 2 — Direccionamiento inmediato frente a directo a memoria

`MOV AX,0005` utiliza direccionamiento inmediato: la instrucción contiene el propio valor que se copiará a AX, por lo que el procesador obtiene el operando del flujo de instrucciones. En cambio, `MOV AX,\\\[0300]` utiliza direccionamiento directo a memoria: la instrucción contiene una dirección y el procesador debe leer en esa ubicación el dato que será transferido a AX. Ambas codificaciones ocupan tres bytes, pero no realizan el mismo trabajo durante la ejecución; la segunda requiere una lectura adicional del operando en memoria, además de la obtención de la instrucción. El modo inmediato resulta conveniente para constantes conocidas al ensamblar, mientras que el directo es apropiado cuando el dato puede cambiar durante la ejecución, como ocurre con los bytes escritos mediante `E`. El comando `U` permite comparar ambas codificaciones sin ejecutarlas: muestra `B8 05 00` para el inmediato y `A1 00 03` para la lectura directa, de modo que la inspección no altera los registros ni la memoria.

## 11\. Observaciones de los checkpoints de la Parte 1

### CP1 — Registros

La salida de `R` permite identificar los registros generales, los registros de segmento, el puntero de instrucción y las banderas. La modificación selectiva con `R AX` demuestra que el depurador puede alterar un registro sin solicitar la reescritura de los demás. La captura debe conservar los valores reales de la sesión.

### CP2 — Volcado de memoria

El relleno con `F` establece un patrón reconocible de 64 bytes y `D` permite verificarlo sin modificarlo. La exploración del PSP distingue la zona de control inicial del segmento de la zona de entrada habitual del programa COM.

### CP3 — Ensamblado y desensamblado

La correspondencia entre mnemónicos y bytes confirma la codificación de las cuatro instrucciones. El programa ocupa 10 bytes. `U` permite validar la secuencia ensamblada sin ejecutarla.

### CP4 — Memoria y direccionamiento

La secuencia `F`, `D`, `E`, `D` demuestra una modificación puntual. La lectura mediante `MOV AX,\\\[0300]` confirma el valor `5678h` en AX, siempre que DS señale el segmento correcto. La captura debe mostrar también la verificación de la codificación con `U`.

\---

# PARTE 2 — ENSAMBLADO Y EJECUCIÓN PASO A PASO

## 12\. Preparación del entorno

La guía utiliza un directorio independiente para la segunda parte:

```text
Z:\\\\> MOUNT C C:\\\\DOSWork
Z:\\\\> C:
C:\\\\> MD LAB3POST2
C:\\\\> CD LAB3POST2
C:\\\\LAB3POST2> DEBUG
-
```

El repositorio de GitHub sigue siendo único (`apellido-post1-u3`). `LAB3POST2` es el directorio de trabajo dentro de DOSBox y no sustituye la estructura del repositorio.

## 13\. Programa de suma

### 13.1 Ensamblado y codificación

```text
-A 100
MOV AX,000A
MOV BX,0005
MOV CX,0003
ADD AX,BX
ADD AX,CX
INT 20
```

|Dirección|Bytes|Instrucción|Tamaño|
|-|-|-|-:|
|0100|B8 0A 00|MOV AX,000A|3|
|0103|BB 05 00|MOV BX,0005|3|
|0106|B9 03 00|MOV CX,0003|3|
|0109|03 C3|ADD AX,BX|2|
|010B|03 C1|ADD AX,CX|2|
|010D|CD 20|INT 20|2|
|**Total**|||**15 bytes**|

Verificación:

```text
-U 100 10E
-R IP
:0100
```

### 13.2 Tabla de traza del programa de suma

La tabla registra el estado posterior a cada instrucción. Los estados iniciales de banderas de la guía muestran ZF=0, CF=0 y SF=0; las instrucciones `MOV` no modifican estas banderas. Las dos sumas producen resultados positivos distintos de cero y no generan acarreo.

|Instrucción ejecutada|AX|BX|CX|IP siguiente|ZF|CF|SF|
|-|-:|-:|-:|-:|-:|-:|-:|
|MOV AX,000A|000A|0000|0000|0103|0\*|0\*|0\*|
|MOV BX,0005|000A|0005|0000|0106|0\*|0\*|0\*|
|MOV CX,0003|000A|0005|0003|0109|0\*|0\*|0\*|
|ADD AX,BX|000F|0005|0003|010B|0|0|0|
|ADD AX,CX|0012|0005|0003|010D|0|0|0|
|INT 20|0012|0005|0003|010F\*\*|0|0|0|

`\\\*` Banderas conservadas por MOV; se toma como referencia el estado inicial mostrado en la guía.  
`\\\*\\\*` IP secuencial posterior a los dos bytes de INT 20; el comportamiento de retorno al DOS puede hacer que DEBUG recupere el control de otra manera. Para el resultado aritmético, el punto relevante es AX=0012 antes de la terminación.

Cálculo: `000A + 0005 = 000F`; después `000F + 0003 = 0012` (18 decimal).

## 14\. Programa con LOOP

### 14.1 Ensamblado y codificación correcta

```text
-A 100
MOV AX,0000
MOV CX,0004
ADD AX,0002
LOOP 0106
INT 20
```

|Dirección|Bytes|Instrucción|Tamaño|
|-|-|-|-:|
|0100|B8 00 00|MOV AX,0000|3|
|0103|B9 04 00|MOV CX,0004|3|
|0106|05 02 00|ADD AX,0002|3|
|0109|E2 FB|LOOP 0106|2|
|010B|CD 20|INT 20|2|
|**Total**|||**13 bytes**|

La instrucción `LOOP` utiliza un desplazamiento relativo de 8 bits con signo. El IP posterior a `LOOP` es `010B`; el destino es `0106`. La diferencia es `0106 - 010B = -5`, que en complemento a dos de 8 bits se codifica como `FB`.

Verificación del programa completo:

```text
-U 100 10C
-D CS:0100 L0D
```

El rango `0100` a `010C` inclusive contiene 13 bytes. `L0D` solicita 13 bytes. La guía usa `L0C`, que solicita solamente 12 y no muestra el byte final `20` de `INT 20`.

### 14.2 Tabla completa de traza LOOP

Cada iteración ejecuta un `ADD` y un `LOOP`. `LOOP` decrementa CX y decide el salto, pero no modifica las banderas. `ADD AX,0002` deja ZF=0, CF=0 y SF=0 en las cuatro sumas.

|Iteración|Instrucción|AX después|CX después|IP siguiente|¿LOOP salta?|
|-|-|-:|-:|-:|-|
|Inicialización|MOV AX,0000|0000|0000|0103|—|
|Inicialización|MOV CX,0004|0000|0004|0106|—|
|1|ADD AX,0002|0002|0004|0109|—|
|1|LOOP 0106|0002|0003|0106|Sí|
|2|ADD AX,0002|0004|0003|0109|—|
|2|LOOP 0106|0004|0002|0106|Sí|
|3|ADD AX,0002|0006|0002|0109|—|
|3|LOOP 0106|0006|0001|0106|Sí|
|4|ADD AX,0002|0008|0001|0109|—|
|4|LOOP 0106|0008|0000|010B|No|
|Terminación|INT 20|0008|0000|010D\*\*|—|

El programa suma `2` cuatro veces: `2 + 2 + 2 + 2 = 8`. El resultado es `AX=0008` y el contador termina en `CX=0000`.

**Conteo:** 2 instrucciones de inicialización + 4 `ADD` + 4 `LOOP` + 1 `INT 20` = **11 instrucciones ejecutadas**.

## 15\. Análisis del código máquina mediante D

El volcado completo esperado para el programa LOOP es:

```text
-D CS:0100 L0D
B8 00 00 B9 04 00 05 02 00 E2 FB CD 20
```

|Dirección|Bytes|Instrucción|Tamaño|
|-|-|-|-:|
|0100|B8 00 00|MOV AX,0000|3|
|0103|B9 04 00|MOV CX,0004|3|
|0106|05 02 00|ADD AX,0002|3|
|0109|E2 FB|LOOP 0106|2|
|010B|CD 20|INT 20|2|
|**Total**|||**13 bytes**|

El volcado muestra los bytes consecutivos almacenados en memoria. La separación visual de grupos no cambia el contenido. La longitud total se obtiene sumando el tamaño de cada instrucción: `3 + 3 + 3 + 2 + 2 = 13` bytes.

## 16\. Bucle equivalente con DEC/JNZ

### 16.1 Ensamblado y codificación

```text
-A 200
MOV AX,0000
MOV CX,0004
ADD AX,0002
DEC CX
JNZ 0206
INT 20
```

|Dirección|Bytes|Instrucción|Tamaño|
|-|-|-|-:|
|0200|B8 00 00|MOV AX,0000|3|
|0203|B9 04 00|MOV CX,0004|3|
|0206|05 02 00|ADD AX,0002|3|
|0209|49|DEC CX|1|
|020A|75 FA|JNZ 0206|2|
|020C|CD 20|INT 20|2|
|**Total**|||**13 bytes**|

`DEC CX` se codifica como `49`. `JNZ` se codifica como `75` seguido del desplazamiento relativo. El IP posterior a `JNZ` es `020C`; el destino es `0206`. La diferencia `0206 - 020C = -6` se representa como `FA`.

### 16.2 Tabla completa de traza DEC/JNZ

`DEC` decrementa CX y modifica ZF, SF y otras banderas aritméticas, pero **conserva CF**. `JNZ` consulta ZF y no modifica las banderas. Como el `ADD` inicializa las banderas relevantes en cero, CF permanece en cero durante el bucle; al decrementar CX de 1 a 0, ZF pasa a uno.

|Iteración|Instrucción|AX después|CX después|IP siguiente|¿JNZ salta?|
|-|-|-:|-:|-:|-|
|Inicialización|MOV AX,0000|0000|0000|0203|—|
|Inicialización|MOV CX,0004|0000|0004|0206|—|
|1|ADD AX,0002|0002|0004|0209|—|
|1|DEC CX|0002|0003|020A|—|
|1|JNZ 0206|0002|0003|0206|Sí|
|2|ADD AX,0002|0004|0003|0209|—|
|2|DEC CX|0004|0002|020A|—|
|2|JNZ 0206|0004|0002|0206|Sí|
|3|ADD AX,0002|0006|0002|0209|—|
|3|DEC CX|0006|0001|020A|—|
|3|JNZ 0206|0006|0001|0206|Sí|
|4|ADD AX,0002|0008|0001|0209|—|
|4|DEC CX|0008|0000|020A|—|
|4|JNZ 0206|0008|0000|020C|No|
|Terminación|INT 20|0008|0000|020E\*\*|—|

El resultado coincide con el bucle LOOP: `AX=0008`. La condición de salida se cumple cuando `DEC CX` produce cero y activa ZF; entonces `JNZ` no realiza el salto.

**Conteo:** 2 instrucciones de inicialización + 4 `ADD` + 4 `DEC` + 4 `JNZ` + 1 `INT 20` = **15 instrucciones ejecutadas**.

## 17\. Decisión técnica 3 — Selección de LOOP frente a DEC/JNZ

Para un bucle contador sencillo, cuyo cuerpo no necesita utilizar CX para otra función, `LOOP` constituye la alternativa recomendada. En este caso, `LOOP` reúne el decremento del contador y la decisión de salto en una sola instrucción de dos bytes (`E2 FB`). La solución alternativa requiere `DEC CX` y `JNZ`, que ocupan tres bytes (`49 75 FA`) y representan dos instrucciones que el procesador debe obtener y ejecutar en cada iteración. Por ello, en el ejemplo de cuatro repeticiones, LOOP requiere cuatro instrucciones menos y una codificación de control más compacta. `DEC/JNZ` puede resultar preferible cuando se necesita separar el decremento de la condición de salto, combinar el control con una lógica de banderas más amplia o utilizar una condición establecida por `CMP` en vez de depender exclusivamente de CX distinto de cero. El comando `U` permite comparar las direcciones inicial y final de cada secuencia, contar los bytes y comprobar que LOOP ocupa dos bytes mientras DEC/JNZ ocupa tres. Para el caso específico del laboratorio, LOOP ofrece menor tamaño y menor cantidad de instrucciones ejecutadas.

## 18\. Comparación del número de instrucciones ejecutadas

|Componente|LOOP|DEC/JNZ|
|-|-:|-:|
|Inicialización (`MOV AX`, `MOV CX`)|2|2|
|`ADD` del cuerpo|4|4|
|Control del bucle|4 (`LOOP`)|8 (`DEC` + `JNZ`)|
|Terminación (`INT 20`)|1|1|
|**Total**|**11**|**15**|

La diferencia es de cuatro instrucciones. En cada una de las cuatro iteraciones, LOOP realiza el decremento y la decisión mediante una única instrucción. La versión manual necesita dos instrucciones (`DEC` y `JNZ`) para cumplir esas dos funciones. Por eso añade exactamente una instrucción por iteración: `4 × 1 = 4` instrucciones adicionales.

La comparación anterior contabiliza instrucciones ejecutadas, no ciclos de reloj. El número de ciclos depende de la variante del procesador, del resultado del salto y de otros factores de ejecución.

## 19\. Verificación rápida con G

Para ejecutar el bucle DEC/JNZ hasta detenerse antes de `INT 20`, se restablece IP y se establece un punto de interrupción en `020C`:

```text
-R IP
:0200
-G 20C
-R AX
```

`G` (*Go*) ejecuta el programa sin detenerse después de cada instrucción hasta alcanzar el punto de interrupción especificado. Al detenerse en `020C`, se espera `AX=0008` y `CX=0000`. `R AX` permite consultar el acumulador después de la ejecución.

El punto `020C` corresponde a la dirección de `INT 20`, de modo que el depurador se detiene antes de ejecutar la instrucción de terminación. Si la versión de DEBUG utilizada presenta una salida diferente, se debe registrar la salida real y comprobar que IP llegó al punto previsto.

## 20\. Decisión técnica 4 — Uso de T frente a G

Para un bucle de cien iteraciones, `G` resulta más práctico que repetir manualmente el comando `T`, porque permite ejecutar una secuencia extensa hasta un punto de interrupción sin requerir una intervención por cada instrucción. El depurador puede detenerse antes de la instrucción de terminación y presentar el estado final, lo que permite comprobar rápidamente el resultado del cálculo. Sin embargo, al utilizar `G` se pierde la observación detallada de los estados intermedios: no se registra cómo cambia AX en cada suma, cómo se decrementa CX ni en qué momento se modifica ZF. Por esa razón, `T` es indispensable en los pasos destinados a construir tablas de traza, ya que ejecuta una instrucción por vez y muestra el estado posterior a cada una. `G` es adecuado para validar el resultado final de una ejecución extensa, mientras que `T` es adecuado para explicar y documentar el comportamiento interno. Después de `G 20C`, el comando `R AX` permite consultar específicamente el acumulador.

## 21\. Observaciones de los checkpoints de la Parte 2

### CP1 — Traza del programa de suma

Las instrucciones `MOV` cargan los operandos sin modificar las banderas. Las dos operaciones `ADD` acumulan los valores en AX y producen `0012h`. La tabla registra el IP posterior a cada instrucción y diferencia las banderas conservadas de las actualizadas por la suma.

### CP2 — Traza del bucle LOOP

CX inicia en cuatro y disminuye una unidad en cada ejecución de LOOP. El salto se realiza mientras CX no sea cero. En la cuarta ejecución, CX llega a cero y la ejecución continúa hacia `INT 20`. AX termina en `0008h`.

### CP3 — Traza DEC/JNZ y verificación con G

El bucle alternativo obtiene el mismo resultado mediante `DEC CX` y `JNZ`. La última operación DEC establece ZF=1 y provoca que JNZ no salte. La traza permite contar 15 instrucciones; la ejecución con G confirma el resultado final sin mostrar cada estado intermedio.

### CP4 — Repositorio finalizado

El repositorio debe contener el README, la carpeta de capturas, los siete archivos de evidencia auténtica y los seis commits descriptivos. La URL del repositorio se comparte en la plataforma del curso.

\---

## 22\. Guía para obtener las capturas reales de DOSBox

Las imágenes deben capturarse directamente desde DOSBox después de ejecutar los comandos. No deben recrearse con un editor, generarse artificialmente ni sustituirse por imágenes ilustrativas.

|Archivo|Comandos que deben aparecer|Resultado y elementos visibles|
|-|-|-|
|`CP1\\\_registros.png`|`DEBUG`, `R`, `R AX`, ingreso de `1234`, nuevo `R AX`|Prompt del laboratorio, registros, IP, banderas y AX modificado a `1234`.|
|`CP2\\\_volcado\\\_memoria.png`|`F 200 L40 AB CD EF`, `D 200 L40`, opcionalmente `D 0 L20`|Segmento:desplazamiento, cuatro filas con el patrón repetido y columna ASCII.|
|`CP3\\\_ensamblado\\\_desensamblado.png`|`A 100`, cuatro instrucciones y `U 100 109`|Los cuatro mnemónicos y sus bytes: `B8 05 00`, `BB 03 00`, `03 C3`, `CD 20`.|
|`CP4\\\_memoria\\\_direccionamiento.png`|`F 300 L10 00`, `D 300 L10`, `E 300 78 56`, `D 300 L10`, `A 320`, `U 320 324`, `T`, `R AX`|Antes y después de la escritura, bytes `78 56`, instrucción `A1 00 03` y AX=`5678`.|
|`CP1\\\_traza\\\_suma.png`|`U 100 10E`, `R IP`, varios `T`|Instrucciones, estados de AX/BX/CX, IP y banderas; resultado AX=`0012`.|
|`CP2\\\_traza\\\_loop.png`|`U 100 10C`, `R IP`, `T` para cada instrucción|Cuatro sumas, cuatro ejecuciones de LOOP, decremento de CX y resultado AX=`0008`.|
|`CP3\\\_traza\\\_dec\\\_jnz.png`|`U 200 20D`, `R IP`, `T` para cada instrucción, `R IP`, `G 20C`, `R AX`|Traza DEC/JNZ, salida de G en el punto de interrupción y AX=`0008`.|

### Recomendaciones de captura

* Mantener visible la ventana completa de DOSBox y evitar recortar los comandos o los resultados.
* Aumentar el tamaño de la ventana o la resolución de DOSBox si las líneas no caben.
* Procurar que cada captura muestre el comando ejecutado y la respuesta correspondiente.
* Guardar las imágenes en formato PNG y respetar exactamente los nombres de la tabla.
* No alterar los valores de salida ni editar la imagen para simular una ejecución.
* Si la salida no coincide con la tabla calculada, revisar segmento, IP, longitud y bytes ensamblados antes de documentar el resultado.

\---

## 23\. Estructura del repositorio

```text
apellido-post1-u3/
├── README.md
└── capturas/
    ├── CP1\\\_registros.png
    ├── CP2\\\_volcado\\\_memoria.png
    ├── CP3\\\_ensamblado\\\_desensamblado.png
    ├── CP4\\\_memoria\\\_direccionamiento.png
    ├── CP1\\\_traza\\\_suma.png
    ├── CP2\\\_traza\\\_loop.png
    └── CP3\\\_traza\\\_dec\\\_jnz.png
```

Los siete PNG se incorporan únicamente después de obtener las capturas reales. Mientras tanto, `capturas/README.md` documenta los nombres y el contenido requerido para cada evidencia.

## 24\. Commits solicitados

Desde la carpeta que contiene el proyecto:

```bash
git init
git add README.md capturas/
git commit -m "Agrega estructura inicial de la Parte 1"
```

Después de incorporar las capturas CP1 y CP2:

```bash
git add README.md capturas/CP1\\\_registros.png capturas/CP2\\\_volcado\\\_memoria.png
git commit -m "Completa exploración de registros y memoria (CP1 y CP2)"
```

Después de incorporar CP3 y CP4:

```bash
git add README.md capturas/CP3\\\_ensamblado\\\_desensamblado.png capturas/CP4\\\_memoria\\\_direccionamiento.png
git commit -m "Completa ensamblado y direccionamiento directo (CP3 y CP4)"
```

Para la Parte 2:

```bash
git add README.md
git commit -m "Agrega estructura inicial de la Parte 2"
```

Después de incorporar las capturas de suma y LOOP:

```bash
git add README.md capturas/CP1\\\_traza\\\_suma.png capturas/CP2\\\_traza\\\_loop.png
git commit -m "Completa traza suma (CP1) y traza loop (CP2)"
```

Después de incorporar la captura DEC/JNZ y completar las decisiones:

```bash
git add README.md capturas/CP3\\\_traza\\\_dec\\\_jnz.png
git commit -m "Completa bucle DEC/JNZ y decisiones técnicas (CP3)"
```

Para publicar el repositorio en GitHub, se crea primero un repositorio vacío con el nombre exacto `apellido-post1-u3` y luego se configura el remoto:

```bash
git branch -M main
git remote add origin https://github.com/USUARIO/apellido-post1-u3.git
git push -u origin main
```

`USUARIO` debe reemplazarse por el identificador real de la cuenta. No se debe copiar literalmente ese marcador.

\---

## 25\. Conclusiones generales

El laboratorio permite relacionar la representación binaria de las instrucciones con el comportamiento observable de los registros y la memoria. La inspección mediante `R` identifica el estado del procesador; `F` prepara regiones con patrones conocidos; `E` modifica bytes puntuales; y `D` permite comprobar el contenido sin alterar la memoria. El uso conjunto de `A` y `U` evidencia que cada instrucción ensamblador tiene una codificación concreta y que el desensamblado permite verificarla antes de ejecutar el programa.

La lectura de `78 56` mediante `MOV AX,\\\[0300]` demuestra la interpretación little-endian y el funcionamiento del direccionamiento directo a memoria. En la segunda parte, la traza de suma confirma el resultado `0012h`, mientras que los programas LOOP y DEC/JNZ obtienen `0008h` mediante mecanismos de control diferentes. La comparación muestra que LOOP reduce el número de instrucciones ejecutadas en este caso. Finalmente, `T` proporciona la granularidad necesaria para construir trazas y `G` facilita la verificación de resultados finales en ejecuciones más largas.

## 26\. Lista de verificación basada en la rúbrica

### Implementación — 40 %

* \[ ] DOSBox inicia y monta correctamente `C:\\\\DOSWork`.
* \[ ] Se ejecutaron `R`, `F`, `D`, `E`, `A` y `U` en la Parte 1.
* \[ ] Se ejecutaron `A`, `U`, `D`, `T`, `LOOP`, `DEC`, `JNZ` y `G` en la Parte 2.
* \[ ] Se verificó DS antes de leer el dato con direccionamiento directo.
* \[ ] Se verificaron las direcciones y los bytes con `U`.

### Funcionalidad — 25 %

* \[ ] El patrón `AB CD EF` aparece en el volcado de 64 bytes.
* \[ ] El programa de cuatro instrucciones presenta las codificaciones indicadas.
* \[ ] La lectura directa produce AX=`5678`.
* \[ ] El programa de suma produce AX=`0012`.
* \[ ] LOOP produce AX=`0008` y CX=`0000`.
* \[ ] DEC/JNZ produce AX=`0008` y CX=`0000`.
* \[ ] `G 20C` se detiene antes de `INT 20` y permite comprobar AX.

### Documentación — 20 %

* \[ ] Se explican los comandos DEBUG utilizados.
* \[ ] Se incluyen las tres tablas completas de traza.
* \[ ] Se explica el volcado de código máquina y el tamaño de cada programa.
* \[ ] Se incluyen cuatro decisiones técnicas de al menos 100 palabras cada una.
* \[ ] Se documentan las observaciones de los checkpoints.
* \[ ] Se incluyen conclusiones en tercera persona.

### Estilo y convenciones — 15 %

* \[ ] El repositorio se llama exactamente `apellido-post1-u3`.
* \[ ] Existe `README.md` en la raíz.
* \[ ] Existe la carpeta `capturas/`.
* \[ ] Se incorporaron los siete PNG auténticos con los nombres exactos.
* \[ ] Se registraron los seis commits descriptivos.
* \[ ] Se publicó el repositorio y se obtuvo su URL para la entrega.

\---

**Estado de preparación:** documentación y estructura base desarrolladas. Las capturas reales, la ejecución local de los comandos, los seis commits y la publicación en GitHub deben completarse desde el entorno del estudiante para que la evidencia represente el trabajo efectivamente realizado.

