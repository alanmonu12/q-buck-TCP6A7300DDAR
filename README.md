# Q-Buck TCP6A7300DDAR EVAL BOARD MX <!-- omit in toc -->

Una placa de evaluación (PoC) Open Source Hardware diseñada para probar y caracterizar el convertidor Buck síncrono TCP6A7300DDAR, un circuito integrado diseñado en México por QSM Semiconductores.

Esta placa está pensada como un primer paso ("Proof of Concept")
para validar el comportamiento térmico, la estabilidad del lazo de control y la eficiencia del IC bajo 
condiciones reales, con miras a integrarlo en proyectos 
educativos e industriales más grandes.

## Tabla de Contenido <!-- omit in toc -->

- [Software y Herramientas](#software-y-herramientas)
- [Características del IC (TCP6A7300DDAR)](#características-del-ic-tcp6a7300ddar)
- [Características de esta Placa de Evaluación](#características-de-esta-placa-de-evaluación)
- [Ecuaciones y Diseño Preliminar](#ecuaciones-y-diseño-preliminar)
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


## Estado del Proyecto

- [x] Cálculo de componentes pasivos.
- [x] Captura del Esquemático (KiCad).
- [ ] Diseño del PCB (Layout y consideraciones térmicas).
- [ ] Ensamble del primer prototipo.
- [ ] Pruebas de caracterización y respuesta en frecuencia (FRA).

## Referencias y Documentación

- **Cálculo de Etapa de Potencia:** [Basic Calculation of a Buck Converter's Power Stage (SLVA477B)](https://www.ti.com/lit/an/slva477b/slva477b.pdf) - Texas Instruments. Documento base para las ecuaciones de selección del inductor ($L$) y capacitores de entrada/salida ($C_{IN}$, $C_{OUT}$).
- **Análisis de Estabilidad (Bode Plot):** [How to Measure the Loop Transfer Function of Power Supplies (AN-1889 / SNVA364A)](https://www.ti.com/lit/an/snva364a/snva364a.pdf) - Texas Instruments. Metodología de la industria para la inyección de pequeña señal y medición de Margen de Fase y Ganancia.
- **Datasheet Principal:** TCP6A7300DDAR - QSM Semiconductores (Especificaciones preliminares provistas por el fabricante).

## License

This project is licensed under the CC BY-NC-SA 4.0 License. See the LICENSE file for details.
