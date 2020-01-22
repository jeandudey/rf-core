# RF Core

El *RF Core* es un procesador *Cortex M0* que maneja la radio, este se comunica
con el CPU principal (*System CPU* desde ahora) por medio de comandos. Los
comandos se envían a través del registro `RFC_DBELL.CMDR`.

El registro `CMDR` guarda un puntero a la dirección de un comando. Este puntero
es un puntero hacia una región de la memoria del *System CPU*, es decir, es
nuestro trabajo reservar memoria para los comandos.

## Modos de operación

El CC1312 en este caso solo soporta un modo de operación, el modo *prop mode*
que singifica el modo de operación propietario, existen otros modos de
operación como *IEEE 802.15.4 mode* pero son solo soportados por otros
MCUs de la misma familia.

## Comandos

Existen tres tipos de comandos:

 - Inmediatos.
 - Operaciones de radio.
 - Y directos.

Los comandos *inmediatos* y de *operaciones de radio* necesitan un lugar en
la memoria del *System CPU* para guardar esos datos, esta memoria puede ser
declarada como una variable global en C y ser reutilizada cada vez que se
necesite, es decir para cada comando inmediato o de una operación de radio
debe exister un lugar en la memoria donde se guarden sus datos, por ejemplo
declarando una variable global para cada comando. Además la memoria necesita
estar alineada en una posición de 32 bits.

Los comandos directos no necesitan de memoria en el sistema y se ejecutan
directamente al ponerlos en el registro `CMDR`, encajan en un entero sin signo
de 32 bit (`uint32_t`) y tienen un formato especial.

### Inmediatos y Operaciones de radio

#### Formato

Este tipo de comandos presentan el siguiente tipo de formato para ser ingresado
al registro `CMDR`, como podemos ver es un puntero con los dos primeros bits
en 0 (por eso la alineación, se encarga solo de eso) y el resto es donde apunta
la memoria, en este caso a la estructura de donde está el comando.

```
+-----------------------------------------+
|Puntero a la estructura del Comando |0  0|
+-----------------------------------------+
31                                  2     0
  MSB                                      LSB
```

### Directos

#### Formato

Este tipo de comandos en vez de ser un puntero, se codifica directamente en
el registro `CMDR`.

- Los primeros dos bits es el valor `b01` (para identificación).
- Del bit `2` al `8` es un *parametro opcional de extensión*.
- Del bit `8` al `16` es un *parametro opcional*.
- Del bit `16` al `31` es un el *ID del comando*.

```
+-----------------------------------------+
| ID de Comando|Parámetro |Extensión|0   1|
+-----------------------------------------+
31             16         8         2     0
  MSB                                      LSB
```

### Lista de comandos

Para utilizar el CC1312 necesitamos una serie de comandos, no mostraré los
comandos de otros dispositivos para abreviar.

Para el modo de operación *prop mode* necesitamos los siguientes comandos:

- `CMD_PROP_TX`: comando de TX simple.
  - **Tipo:** Operación.
- `CMD_PROP_RX`: comando de RX simple.
  - **Tipo:** Operación.
- `CMD_PROP_TX_ADV`: comando de TX avanzado.
  - **Tipo:** Operación.
- `CMD_PROP_RX_ADV`: comando de RX avanzado.
  - **Tipo:** Operación.
- `CMD_PROP_CS`: *carrier sense*.
  - **Tipo:** Operación.
- `CMD_PROP_RADIO_SETUP`: comando para perparar la radio, solo para 2.4 GHz.
  - **Tipo:** Operación.
- `CMD_PROP_RADIO_DIV_SETUP`: comando para preparar la radio, todas las bandas.
  - **Tipo:** Operación.
- `CMD_PROP_SET_LEN`: comando para cambiar el tamaño del paquete.
  - **Tipo:** Operación.
- `CMD_PROP_RESTART_RX`: reiniciar el RX.
  - **Tipo:** Operación.

Otros comandos necesarios son:

**ToDo:** hay que investigar los otros comandos, hay varios necesarios pero
no recuerdo exactamente.

**ToDo**: comando de set tx power.

## Dirección de capa de enlace (*link layer address*)

Hay dos lugares donde se guarda el *link layer address* (LLA desde ahora) en
la memoria del CC13x2 (aplica también para otros).

1. En el componente `FCFG1`:

Los registros `MAC_15_4_0` y `MAC_14_4_1` son dos registros de 32 bits cada uno
que juntos hacen el total de 64 bits del LLA.

```
+---------------------+
|MAC_15_4_1|MAC_15_4_0|
+---------------------+
63         32          0
```

A esta dirección se le llama **dirección primaria**, es la que viene de fábrica
y se utiliza por defecto cuando no está puesta la dirección personalizada.

2. En el componente `CCFG`:

Los registros `IEEE_MAC_0` y `IEEE_MAC_1` son dos registros de 32 bits cada uno
que juntos hacen el total de 32 bits del LLA.


```
+---------------------+
|IEEE_MAC_0|IEEE_MAC_1|
+---------------------+
63         32          0
```

A esta dirección se le llama **dirección secundaria**, es personalizable
y por defecte viene llena de todos sus bytes con `FFh` para identificar cuando
está puesta o no. Primero se verifica si esta dirección existe y si no se usa
la primaria.

## Proceso de inicio del *RF Core*

**ToDo:** información desconocida en proceso de investigación, el datasheet no
nombra un proceso especifico para hacer esto, se necesita hacer ingeniería
inversa al SDK de TI para el CC13x2, CC26x2.

Por los momentos es un poco desconocido el proceso para iniciar el *RF Core*.

1. Se inicia `RTC_UPD` en el registro `AON_RTC.CTL`.
2. Se establecen los parametros de inicio del *RF Core* en el registro
`PRCM.RFCBITS`, en el datasheet están los parametros que se pueden utilizar.
En el SDK hay unos bits especificos que usan sin explicación y son los mismos
que se van a utilizar de momento. Hasta ahora no ha habido problemas.
3. En este momento el *RF Core* está apagado y ciertos registros necesitan
iniciarse para que el *RF Core* haga el *boot*.
***En proceso de investigación***
4. Activar el IRQ de el `CPE0` y del `HW_CMD`. ***En proceso de investigación***

## Enviar paquetes

El *RF Core* utiliza paquetes para enviar comandos, en este caso el que
necesitamos es el `CMD_PROP_TX_AD`. ***En proceso de investigación***
