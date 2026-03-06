View this project on [CADLAB.io](https://cadlab.io/project/30117). 

# Q-Buck TCP6A7300DDAR EVAL BOARD MX <!-- omit in toc -->

![EDA](https://img.shields.io/badge/EDA-KiCad-blue.svg)
![Stackup](https://img.shields.io/badge/Stackup-4_Layers-purple.svg)
![Topology](https://img.shields.io/badge/Topology-Sync_Buck_Converter-red.svg)
![Input](https://img.shields.io/badge/VIN-Up_to_35V-lightgrey.svg)
![Output](https://img.shields.io/badge/VOUT-5V_%40_1.4A-brightgreen.svg)
![Frequency](https://img.shields.io/badge/fSW-500_kHz-orange.svg)
![Status](https://img.shields.io/badge/Status-Layout_WIP-yellow.svg)

Una placa de evaluación (PoC) Open Source Hardware diseñada para probar y caracterizar el convertidor Buck síncrono TCP6A7300DDAR, un circuito integrado diseñado en México por QSM Semiconductores. Esta placa está pensada como un primer paso ("Proof of Concept") para validar el comportamiento térmico, la estabilidad del lazo de control y la eficiencia del IC bajo  condiciones reales, con miras a integrarlo en proyectos educativos e industriales más grandes.

<p align="center">
  <img src="./img/PCB-render.png" alt="Render 3D de la Q-Buck TCP6A7300DDAR" width="600">
</p>

## Tabla de Contenido <!-- omit in toc -->

- [Software y Herramientas](#software-y-herramientas)
- [Características del IC (TCP6A7300DDAR)](#características-del-ic-tcp6a7300ddar)
- [Características de esta Placa de Evaluación](#características-de-esta-placa-de-evaluación)
- [Ecuaciones y Diseño Preliminar](#ecuaciones-y-diseño-preliminar)
- [📦 Lista de Materiales (BOM) y Consideraciones de Ensamblaje](#-lista-de-materiales-bom-y-consideraciones-de-ensamblaje)
- [⚠️ Limitaciones y Física del Hardware (Derating)](#️-limitaciones-y-física-del-hardware-derating)
- [📐 Arquitectura del Layout y Control de EMI](#-arquitectura-del-layout-y-control-de-emi)
  - [1. Minimización del "Hot Loop" (Ruido Magnético - Alto $di/dt$)](#1-minimización-del-hot-loop-ruido-magnético---alto-didt)
  - [2. Contención del Nodo de Conmutación (Ruido Eléctrico - Alto $dv/dt$)](#2-contención-del-nodo-de-conmutación-ruido-eléctrico---alto-dvdt)
  - [3. El Lazo de Salida](#3-el-lazo-de-salida)
  - [4. Integridad de Potencia (Power Integrity) y Desacoplo Concéntrico](#4-integridad-de-potencia-power-integrity-y-desacoplo-concéntrico)
- [Estado del Proyecto](#estado-del-proyecto)
- [Referencias y Documentación](#referencias-y-documentación)
- [License](#license)

## Software y Herramientas

Este proyecto está comprometido con el uso y fomento de herramientas de código abierto (Open Source) para el diseño de hardware.

Herramientas Actuales:

- **Diseño Electrónico (EDA):** KiCad v10 - Utilizado para la captura del esquemático, diseño del PCB (Layout) y generación de archivos Gerber.

Herramientas Planeadas (Roadmap de Validación):

- **Simulación de Circuitos (SPICE):** ngspice / LTspice para validar la respuesta transitoria y el lazo de control antes del ensamble.

- **Simulación Térmica (FEM):** Flujo de trabajo utilizando FreeCAD, Salome y ElmerFEM para analizar la disipación de calor del componente TCP6A7300DDAR y optimizar los planos de cobre del PCB bajo cargas de 1A continuos.

## Características del IC (TCP6A7300DDAR)

- Voltaje de Entrada ($V_{IN}$): 3.8V a 35V
- Corriente de Salida: 3A Continuos
- Frecuencia de Conmutación: 500 kHz
- Voltaje de Referencia ($V_{REF}$): 0.8V
- Topología: Step-Down (Buck) Síncrono

<p align="center">
  <img src="./img/circuito-basico.png" alt="Circuito basico" width="600">
</p>

## Características de esta Placa de Evaluación

- **Salida Fija/Ajustable:** Configurada por defecto a ~5V, con footprint opcional para un potenciómetro de ajuste
- **Design for Testability (DFT):** Incluye una resistencia de inyección ($R_3 = 0\Omega$) en el lazo de retroalimentación (Feedback) específicamente diseñada para inyectar pequeña señal y medir el Diagrama de Bode (Fase y Ganancia) utilizando herramientas como el Analog Discovery 3.
- **Compensación Externa:** Footprint para capacitor de Feedforward ($C_6$) para experimentar con la respuesta transitoria.

## Ecuaciones y Diseño Preliminar

El circuito actual está calculado para un escenario típico de reducción de voltaje (ej. batería de $12V/24V$ a $5V$ para lógica).

**Parámetros de Diseño:**

- $V_{IN(max)} = 35V$
- $V_{OUT} = 5V$
- $I_{OUT} = 1A$
- $f_{sw} = 500 kHz$

**1. Lazo de Retroalimentación (Feedback)**

El divisor de voltaje se calcula basándose en la referencia interna de 0.8V. Asignando $R_2 = 10k\Omega$:

$$R_1 = R_2 \left( \frac{V_{OUT}}{V_{REF}} - 1 \right)$$
$$R_1 = 10k\Omega \left( \frac{5V}{0.8V} - 1 \right) = 52.5 k\Omega$$

   >[!IMPORTANT]
   >
   > En el esquemático se utiliza el valor comercial estándar más cercano de $56k\Omega$, lo que resulta en un $V_{OUT}$ real aproximado de $5.28V$.

**2. Cálculo del Inductor ($L$)**

Se asume un rizado de corriente ($\Delta I_L$) del 30% de la corriente de salida nominal ($I_{OUT} = 2A$), resultando en $\Delta I_L = 600mA$.

$$L = \frac{V_{OUT} (V_{IN} - V_{OUT})}{V_{IN} \cdot f_{sw} \cdot \Delta I_L}$$
$$L = \frac{5V (35V - 5V)}{35V \cdot 500kHz \cdot 600mA} = 14.28 \mu H$$

   >[!IMPORTANT]
   >
   > Se elige un valor comercial estándar de $10 \mu H$, lo que aumentará ligeramente el rizado de corriente, pero mejorará la respuesta transitoria.

**3. Capacitor de Salida ($C_{OUT}$)**

Para mantener un rizado de voltaje en la salida ($\Delta V_{OUT}$) menor a $20mV$:

$$C_{OUT} = \frac{\Delta I_L}{8 \cdot f_{sw} \cdot \Delta V_{OUT}}$$
$$C_{OUT} = \frac{600mA}{8 \cdot 500kHz \cdot 20mV} = 7.5 \mu F$$

   >[!IMPORTANT]
   >
   > Se utiliza un valor estándar de $10 \mu F$ (cerámico multicapa) para un filtrado óptimo.

## 📦 Lista de Materiales (BOM) y Consideraciones de Ensamblaje

Este prototipo fue diseñado bajo una filosofía de **"tropicalización" y rápida iteración**. Todos los componentes pasivos seleccionados son de montaje superficial (SMD tamaño 0805) y están restringidos a números de parte comerciales que se pueden adquirir inmediatamente en el mercado local mexicano (ej. Unit Electronics), evitando tiempos de importación.

| Ref | Parte | Valor | Cantidad | Encapsulado | Función | Notas | Proveedor |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **U1** | TCP6A7300DDAR | | 1 | SOP-8-EP | IC Step-Down | QSM Semiconductores (3A Max) | [Link](#) |
| **L1** | SWPA6020 | $10\mu H$ | 1 | 6.0x6.0mm | Inductor de Potencia | $I_{max} = 1.4A$, $I_{sat} = 2.1A$ | [Link](https://uelectronics.com/producto/indutor-de-ferrita-10uh-1-15a-slf0403-100mtt/) |
| **C1, C4, C5** |  CC0805KRX7R9BB104 | $0.1\mu F$ | 3 | 0805 |  |  | [Link](https://uelectronics.com/producto/cc0805krx7r9bb104-capacitor-ceramico-0805-100nf-50v/) |
| **C2** | UMK212ABJ225KG-T | $2.2\mu F$ | 1 | 0805 | Capacitor de Entrada | Soporta el alto $di/dt$ del nodo $V_{IN}$ | [Link](https://uelectronics.com/producto/umk212abj225kg-t-capacitor-ceramico-0805-2-2uf-50v/) |
| **C3** |  TMK212BBJ106KG-T | $10\mu F$ | 1 | 0805 | Capacitor de Salida | Absorbe el rizado principal ($\Delta I_L$) | [Link](https://uelectronics.com/producto/tmk212bbj106kg-t-capacitor-ceramico-0805-10uf-25v/) |
| **C6** | CL21C101JBANNNC | $100pF$ | 1 | 0805 | Feedforward ($C_{FF}$) | Mejora la respuesta transitoria | [Link](https://uelectronics.com/producto/cc0805krx7r9bb104-capacitor-ceramico-0805-100nf-50v/) |
| **C7, C8** | TCC0805X7R105K500DTS | $1\mu F$ | 2 | 0805 |  |  | [Link](https://uelectronics.com/producto/capacitor-ceramico-0805-1uf-50v-tcc0805x7r105k500dts/) |
| **R1** | RC0805JR-0756KL | $56k\Omega$ | 1 | 0805 | Divisor de Voltaje | Fija la salida a $\approx 5.28V$ | [Link](https://uelectronics.com/producto/resistor-56k-ohm-1-8w-smd-0805-rs-05k6802ft/) |
| **R2** | 0805W8F1002T5E | $10k\Omega$ | 1 | 0805 | Divisor de Voltaje | Fija la salida a $\approx 5.28V$ | [Link](https://uelectronics.com/producto/resistor-10k-ohm-1-8w-smd-0805-0805w8f2202t5e/) |
| **R3** | RTT05000JTP  | $0\Omega$ (Jumper) | 1 | 0805 | Inyección de Señal | Para medición del lazo (Bode Plot) | [Link](https://uelectronics.com/producto/resistor-0-ohm-1-8w-smd-0805-rtt05000jtp/) |
| **R4** | 3362P  | $100k\Omega$ | 1 | 6.60x6.99mm | Ajuste de voltaje | Para pruebas a distintos voltajes   | [Link](https://uelectronics.com/producto/potenciometro-de-precision-3362p/) |

> [!IMPORTANT]
> Los footprints marcados como **DNP** (Do Not Populate) en el esquemático, como $C_8$ y $R_4$, se dejaron intencionalmente vacíos para permitir soldar componentes adicionales durante la fase de validación en banco de pruebas.

## ⚠️ Limitaciones y Física del Hardware (Derating)

Debido a la selección de componentes orientada a la disponibilidad local, esta placa de evaluación específica tiene las siguientes características operativas:

**1. Límite de Corriente Continua (1.4A Máximo)**
Aunque el IC TCP6A7300DDAR es capaz de entregar 3A, el inductor seleccionado (SWPA6020S100MT) dicta el límite térmico del sistema a **1.4A**.
Matemáticamente, el inductor no se saturará bajo estas condiciones. En el peor caso ($V_{IN} = 35V$, $V_{OUT} = 5V$), el rizado de corriente es de $0.857A$.
La corriente pico ($I_{peak}$) que verá el núcleo de ferrita es:
$$I_{peak} = I_{OUT} + \frac{\Delta I_L}{2} = 1.4A + 0.428A = 1.828A$$
Dado que $1.828A < 2.1A$ ($I_{sat}$ del inductor), el diseño es seguro y no provocará un cortocircuito por saturación del núcleo.

**2. Pérdida de Capacitancia por DC Bias**
Los capacitores cerámicos multicapa (MLCC) en tamaños compactos como el 0805 pierden capacitancia efectiva cuando se someten a un voltaje continuo.
El capacitor de salida $C_3$ nominal de $10\mu F$ operando a $5V$ DC presentará una capacitancia real cercana a los **$5\mu F - 6\mu F$**. Si durante las pruebas de caracterización se detecta un rizado de voltaje ($\Delta V_{OUT}$) excesivo, se debe poblar el pad auxiliar $C_8$ con un capacitor idéntico en paralelo para recuperar la capacitancia calculada.

## 📐 Arquitectura del Layout y Control de EMI

El ruteo de esta tarjeta de evaluación se diseñó siguiendo estrictamente las directrices de compatibilidad electromagnética (EMC) detalladas en las notas de aplicación (ver referencias). La estrategia física se centra en contener las dos fuentes principales de ruido en un convertidor Buck:

### 1. Minimización del "Hot Loop" (Ruido Magnético - Alto $di/dt$)
En una topología Buck, el lazo de entrada experimenta corrientes altamente discontinuas, generando transitorios de corriente ($di/dt$) severos durante la conmutación de los MOSFETs internos. 
Según la Ley de Faraday ($V = L \cdot \frac{di}{dt}$), cualquier inductancia parásita en este trayecto generará picos de voltaje destructivos (Ringing).

Para mitigar esto, el "Hot Loop" (Lazo Caliente) se redujo a su mínima expresión geométrica:
* El capacitor de derivación de entrada ($C_2$) se colocó lo más cerca físicamente posible de los pines $V_{IN}$ y $GND$ del controlador TCP6A7300DDAR.
* La conexión de retorno (GND) se realizó mediante un polígono de cobre masivo en la capa Top, conectando directamente el terminal negativo de $C_2$ con el *Exposed Pad* del IC, evitando que la corriente discontinua tenga que viajar por vías hacia planos internos, reduciendo así la inductancia parásita del lazo a unos pocos nanohenrios.

### 2. Contención del Nodo de Conmutación (Ruido Eléctrico - Alto $dv/dt$)
El nodo `SW` (físicamente la unión entre el IC y el inductor L1) es el punto de mayor estrés de voltaje en la placa. Transiciona violentamente entre $0V$ y $V_{IN}$ (hasta 35V) a una frecuencia de 500 kHz. Este alto $dv/dt$ actúa como un inyector de corriente capacitiva ($I = C \cdot \frac{dv}{dt}$) hacia cualquier pista adyacente.

Las reglas de ruteo aplicadas a este nodo fueron:
* **Área controlada:** El polígono de cobre del nodo `SW` se diseñó con el ancho suficiente para soportar los 1.4A sin sobrecalentarse, pero se mantuvo lo más pequeño posible para minimizar su capacitancia parásita hacia el plano de tierra y reducir su efecto de "antena" emisora.
* **Aislamiento de señales sensibles:** La pista de retroalimentación analógica (Feedback / FB) fue ruteada en la capa Bottom (o Capa 3), utilizando el plano interno de GND de la Capa 2 como blindaje electromagnético (escudo Faraday) para evitar que el $dv/dt$ del nodo `SW` inyecte ruido en el amplificador de error del sistema.

### 3. El Lazo de Salida
A diferencia de la entrada, la corriente que fluye desde el inductor hacia el capacitor de salida ($C_3$) es continua y presenta un $di/dt$ bajo. Por lo tanto, los polígonos de $V_{OUT}$ se diseñaron amplios para priorizar una baja caída de voltaje en DC y facilitar la disipación térmica.

### 4. Integridad de Potencia (Power Integrity) y Desacoplo Concéntrico
La ubicación de los capacitores de entrada ($C_{IN}$) no se realizó de manera arbitraria, sino aplicando el criterio de impedancia jerárquica para mitigar el ruido electromagnético de alta frecuencia. 

A frecuencias en el rango de los Megahercios (generadas por los tiempos de subida en nanosegundos de los MOSFETs), la Inductancia Equivalente en Serie (ESL) de los componentes domina la impedancia de la red. Siguiendo las recomendaciones de diseño (TI SLYT670 y SNVA803), se aplicó una topología de anillos concéntricos:
* **Filtro de Alta Frecuencia (Hot Loop Primario):** El capacitor cerámico de menor valor y menor encapsulado físico (0.1 $\mu F$) se colocó a distancia milimétrica de los pines $V_{IN}$ y $GND$ del controlador. Su bajísimo ESL y alta Frecuencia de Autorresonancia (SRF) lo convierten en el camino de menor impedancia para absorber los transitorios más violentos del conmutador.
* **Filtro Bulk (Hot Loop Secundario):** El capacitor de mayor capacitancia (2.2 $\mu F$) se ubicó inmediatamente detrás. Su función principal es suministrar la demanda energética de baja frecuencia durante el ciclo de trabajo principal de 500 kHz, operando en un lazo de corriente ligeramente mayor sin comprometer las emisiones radiadas del sistema.

## Estado del Proyecto

- [x] Cálculo de componentes pasivos.
- [x] Captura del Esquemático (KiCad).
- [ ] Diseño del PCB (Layout y consideraciones térmicas).
- [ ] Ensamble del primer prototipo.
- [ ] Pruebas de caracterización y respuesta en frecuencia (FRA).

## Referencias y Documentación

- **Cálculo de Etapa de Potencia:** [Basic Calculation of a Buck Converter's Power Stage (SLVA477B)](https://www.ti.com/lit/an/slva477b/slva477b.pdf) - Texas Instruments.
- **Análisis de Estabilidad (Bode Plot):** [How to Measure the Loop Transfer Function of Power Supplies (AN-1889 / SNVA364A)](https://www.ti.com/lit/an/snva364a/snva364a.pdf) - Texas Instruments.
- **Datasheet Principal:** TCP6A7300DDAR - QSM Semiconductores (Especificaciones preliminares provistas por el fabricante).
- **Diseño de Filtro de Entrada (EMC/EMI):** [Analysis and Design of Input Filter for DC-DC Circuit (SNVA801)](https://www.ti.com/lit/pdf/snva801) - Texas Instruments.
- **Filtro de Salida (Segunda Etapa LC):** - [Design a Second-stage Filter for Sensitive Applications (SSZT824)](https://www.ti.com/lit/pdf/sszt824) - Texas Instruments.
- **El Fundamento Matemático del ESL y la Frecuencia:** - [How to select input capacitors for a buck converter](https://www.ti.com/lit/an/slyt670/slyt670.pdf) - Texas Instruments.
- **La Aplicación Física en el Layout:** [Improve High-Current DC/DC Regulator EMI Performance for Free With Optimized Power Stage Layout](https://www.ti.com/lit/ab/snva803/snva803.pdf?ts=1772697173603) - Texas Instruments.
- **Recomendaciones de layout para convertidores DC/DC:** - [DC-DC PCB Layout Design for EMC](https://www.diodes.com/assets/App-Note-Files/AN1191-DC-DC-PCB-Layout-Design-for-EMC.pdf) - Diode incoporated.

## License

This project is licensed under the CC BY-NC-SA 4.0 License. See the LICENSE file for details.
