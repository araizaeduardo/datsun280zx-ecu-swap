# Speeduino v0.4.4 → L28E | Pin Reference & Wiring Sequence

## MAPEO DE PINES SPEEDUINO

### Power & Ground (Crítico)
```
Speeduino Mega 2560
├─ GND (cualquier pin GND) → Star-point ground
├─ +5V (pin internal) → regulador local (no usar para sensores externos)
└─ Vin (Power Input) → +12V (fusible 60A) → relé ECU
```

**⚠️ NO CONECTES SENSORES ANALÓGICOS AL 5V INTERNO.** Regula localmente con regulador lineal 7805 aislado.

---

## IGNICIÓN (Wasted Spark + EDIS6)

| Pin Speeduino | Salida | Control | Función |
|---|---|---|---|
| **24** | OC1B | TCCR1B | Coil Pack 1 - Primaria 1A (Cil 1) |
| **26** | OC3A | TCCR3B | Coil Pack 1 - Primaria 1B (Cil 4) |
| **28** | OC3C | TCCR3B | Coil Pack 2 - Primaria 2A (Cil 2) |
| **30** | OC4A | TCCR4B | Coil Pack 2 - Primaria 2B (Cil 5) |
| **32** | OC4C | TCCR4B | Coil Pack 3 - Primaria 3A (Cil 3) |
| **34** | OC5A | TCCR5B | Coil Pack 3 - Primaria 3B (Cil 6) |

**Patrón de fuego (360° crank):**
```
Cilindro: 1 → 5 → 3 → 6 → 2 → 4 (L28E orden)
Wasted spark pairs:
  - 1 con 4 (180°)
  - 2 con 5 (180°)
  - 3 con 6 (180°)
```

**Conexión física EDIS6:**
```
EDIS6 Coil Pack (6-pin module)
├─ Pin 1 (+): +12V (desde relé combustible/ignición)
├─ Pin 2 (-): Ground
├─ Pin A (primaria 1A): ← Speeduino pin 24
├─ Pin B (primaria 1B): ← Speeduino pin 26
├─ Pin C (primaria 2A): ← Speeduino pin 28
├─ Pin D (primaria 2B): ← Speeduino pin 30
├─ [Pin E & F]: No usadas (ignora si es versión 4-pin)
└─ Torres secundarias: hacia bujías Cil 1, 4, 2, 5, 3, 6
```

---

## INYECCIÓN (Secuencial)

| Pin Speeduino | Salida | Control | Cilindro | Remarks |
|---|---|---|---|---|
| **11** | PB5 | TCCR1A | 1 | Inyector cilindro 1 |
| **12** | PB6 | TCCR1A | 2 | Inyector cilindro 2 |
| **13** | PB7 | TCCR1A | 3 | Inyector cilindro 3 |
| **44** | PL5 | TCCR5B | 4 | Inyector cilindro 4 |
| **45** | PL4 | TCCR5B | 5 | Inyector cilindro 5 |
| **46** | PL3 | TCCR5B | 6 | Inyector cilindro 6 |

**Nota:** Para inyección secuencial REQUIERES **CMP (Cam Position Sensor)** en A5.
- Sin CMP: usa "batch fire" (todos simultáneamente) — menos preciso pero funciona
- Con CMP: secuencial puro — mejor control, más limpio

**Conexión física inyector:**
```
Inyector EV6 / Bosch 0280155884
├─ Pin 1 (+): Desde riel, 43 PSI regulados
├─ Pin 2 (-): ← Speeduino pin (11-13 o 44-46)
└─ Resistencia interna: +12Ω (high-impedance, compatible)
```

---

## ENTRADAS ANALÓGICAS (Sensores)

| Pin Speeduino | Entrada Analógica | Sensor | Rango | Notas |
|---|---|---|---|---|
| **A0** | ADC0 | **CKP** (Crank Position) | 0-5V | Crítico para trigger. 36-1 wheel + Hall. Secuencia calibrada. |
| **A1** | ADC1 | **MAP** (Manifold Air Pressure) | 0-5V | Freescale MPX4250 (0-250 kPa). Aisla línea con capacitor 0.1µF. |
| **A2** | ADC2 | **IAT** (Intake Air Temp) | 0-5V | Thermistor NTC (GM o Bosch). Resistencia pull-up interna recomendada. |
| **A3** | ADC3 | **TPS** (Throttle Position Sensor) | 0-5V | 3-pin pot. Rango típico 0.5-4.5V (no saturar rails). |
| **A4** | ADC4 | **CLT** (Coolant Temp) | 0-5V | Thermistor NTC. Monta en bloque de agua. |
| **A5** | ADC5 | **CMP** (Cam Position) [OPCIONAL] | 0-5V | Hall sensor en polea árbol de levas. Requerido para secuencial. |
| **A6** | ADC6 | **Wideband O2** [OPCIONAL] | 0-5V | AEM UEGO o Innovate LC-2 output. CAN alternativa. |
| **A7** | ADC7 | **Barometer** [OPCIONAL] | 0-5V | Sensor BMP280 via SPI. Aisla vibraciones. |

**Conexión sensor analógico estándar:**
```
Sensor (ej. MAP)
├─ +5V (regulado aislado) → resistencia pull-up 4.7kΩ (opcional, ya integrada en muchos sensores)
├─ GND (star-point)
└─ OUT → Speeduino A1 (con capacitor 0.1µF a GND cercano)
```

---

## ENTRADAS DIGITALES (Trigger, CMP, etc)

| Pin Speeduino | Entrada Digital | Sensor | Protocolo | Notas |
|---|---|---|---|---|
| **20** (INT4) | CKP primaria | Hall 36-1 | Digital | Trigger principal. Interruption-driven. CRITICO. |
| **21** (INT5) | CMP secundaria | Hall cam | Digital | Permite inyección secuencial. Opcional si batch-fire. |
| **Int0-3** | Disponibles | - | - | Para expansiones (RPM ref, fuel pump delay, etc) |

---

## SALIDAS DIGITALES (Relés, Ventilador, Bomba)

| Pin Speeduino | Salida Digital | Control | Función | Voltaje |
|---|---|---|---|---|
| **7** | D7 | Software | **Relay combustible** | 5V lógica → relé (caja 5A/12V) |
| **8** | D8 | Software | Ventilador radiador | 5V lógica → relé (caja 20A/12V) |
| **6** | D6 | Opcional | Check engine light | 5V lógica → LED 470Ω |
| **9** | D9 | Software | Boost control [TURBO] | PWM 0-100% |

---

## EJEMPLO: CIRCUITO RELÉ COMBUSTIBLE

```
Speeduino D7 (5V lógica) → Entrada Base transistor 2N3904
                            │
                            Colector → Bobina relé 12V / 5A
                                      │
                                      Diodo de protección (1N4001 reverse)
                                      │
                    ↓ (cuando Speeduino activa D7)
                    
Batería (+12V) → fusible 30A → NC/NO relé → Bomba combustible → GND
```

**En TunerStudio:** Configura "Fuel Pump Prime" = 1000ms (priming inicial)

---

## CIRCUITO CKP TRIGGER (36-1 Hall)

```
Hall Sensor (3-pin)
├─ +5V (regulado local) → capacitor 0.1µF
├─ GND (star-point)
└─ OUT (square wave digital)
              ↓
    Optoacoplador aislador (TLP521-1) [RECOMENDADO]
              ↓
    Speeduino Pin 20 (INT4) ← 1kΩ pull-up a +5V
```

**Por qué optoacoplador?** El L28 es viejo; ruido eléctrico en el bloque puede desestabilizar la señal del sensor. Aislamiento galvánico protege el Mega.

**Alternativa si no hay ruido:** Conexión directa Hall → D20 (con Schmitt-trigger si disponible).

---

## SENSOR MAP (Presión Aire)

```
Múltiple de admisión (toma Schraeder)
    ↓ (0.125" silicona)
    
Freescale MPX4250 (0-250 kPa absoluta)
├─ Pin 1 (+5V): Desde regulador local (crucial: 0.1µF + 10µF capacitores)
├─ Pin 2 (GND): Star-point
└─ Pin 3 (OUT): → Speeduino A1 (con 0.1µF capacitor a GND cercano)

Rango de voltaje esperado (NA):
├─ Idle (20 kPa): ~0.8V
├─ Cruise (50 kPa): ~2.0V
└─ WOT (100+ kPa): ~4.0V+
```

**Calibración TunerStudio:**
```
MAP Sensor Type: Freescale 0-250 kPa
ADC Raw Min: 0
ADC Raw Max: 1024
kPa Min: 0
kPa Max: 250
```

---

## SENSOR TPS (Throttle Position)

```
Throttle body potenciómetro 3-pin
├─ Pin 1 (+5V): Desde regulador local
├─ Pin 2 (GND): Star-point
└─ Pin 3 (OUT): → Speeduino A3

Calibración:
├─ Idle (cerrado): ~0.5V
└─ WOT (abierto): ~4.5V
```

**En TunerStudio Calibration → Throttle Position:**
```
Minimum ADC: 102 (0.5V)
Maximum ADC: 921 (4.5V)
Test: Abre y cierra throttle, valida rango lineal sin saltos
```

---

## WIDEBAND O2 (AEM UEGO o Innovate LC-2)

### Opción A: Salida analógica 0-5V
```
AEM UEGO connector
├─ +12V: Batería +
├─ GND: Ground
└─ Linear output (0-5V): → Speeduino A6

Rango:
├─ Lambda 0.7 (rica): ~0.5V
├─ Lambda 1.0 (stoichiometric): ~2.5V
└─ Lambda 1.5 (pobre): ~4.5V
```

### Opción B: Serial (mejor precisión)
```
AEM UEGO serial output (TTL)
RX (from AEM) → Speeduino Serial1 pin 19 (RX1)
TX (to AEM) → Speeduino Serial1 pin 18 (TX1)
Protocolo: 9600 baud, 8N1
```

**En TunerStudio:**
- Display → Gauges → Lambda real-time
- Logging: Captura lambda vs VE table para autotune

---

## SECUENCIA DE CABLEADO (orden recomendado)

### FASE 1: Preparación (sin energía)

1. **Limpiar chassis y bloque**
   - Raspa óxido de puntos de tierra
   - Limpia con papel de lija #120 hasta metal desnudo
   - Aplica grasa anticorrosiva (Vaseline o dielectric grease)

2. **Instalar trigger wheel 36-1**
   - Remueve damper del crank
   - Monta wheel con pernos de acero (NO aluminio, se suelda)
   - Verifica balance (si muele el wheel, balancea después)
   - Acopla sensor Hall a 2-3mm de gap

3. **Preparar ECU & expansiones**
   - Descarga firmware Speeduino v0.4.4 en Arduino Mega
   - Quemador: Arduino IDE + bootloader STM32
   - Verifica con multímetro: +5V en pins 5V, GND en GND

### FASE 2: Ground Star-Point (crítico)

4. **Crear punto de distribución de ground**
   - Terminal de cobre estañada de 4 AWG en bloque (M8 bolt)
   - Conecta:
     - Ground batería (cable 4 AWG)
     - Ground motor (cable 4 AWG)
     - Ground chasis (cable 4 AWG)
     - Ground sensores (star-point interno, cable 8 AWG)
   - Asegura TODAS las conexiones. Reaprieta después de 100km.

### FASE 3: Poder ECU

5. **Cableado batería → fusible → relé → ECU**
   - Cable 4 AWG rojo desde + batería
   - Fusible 60A con porta-fusible (bajo asiento)
   - Relé 5-pin SPDT (12V coil, 40A contactos)
     - Coil: desde ignición (llave) + GND
     - NC/NO: 12V entrada → Vin Speeduino
   - Speeduino Vin + GND a relé
   - **Verifica con multímetro antes de continuar:** +12-13V en Vin cuando llave en ON

### FASE 4: Cableado Ignición

6. **EDIS6 Coil Pack → Speeduino**
   - Coil +12V desde relé combustible
   - Coil GND a star-point
   - Primaria 1A (EDIS6 A) ← Cable silicona 16 AWG ← Speeduino pin 24
   - Primaria 1B (EDIS6 B) ← Cable silicona 16 AWG ← Speeduino pin 26
   - Primaria 2A (EDIS6 C) ← Cable silicona 16 AWG ← Speeduino pin 28
   - Primaria 2B (EDIS6 D) ← Cable silicona 16 AWG ← Speeduino pin 30
   - **Trenza cables ignición con un hilo de ground adyacente (twisted pair)**
   - Protege con sleeving a 6" mínimo de cables de sensores

7. **Bujías & secundarios**
   - Torres EDIS6: Cable de bujía OEM Nissan 8mm
   - Bujías: Gap 0.035-0.040" (L28 estándar)
   - Test: Enciende llave (SIN ARRANCAR), genera spark entre cable secundario y masa

### FASE 5: Cableado Combustible

8. **Bomba Walbro 255 → Relé → Speeduino**
   - Entrada bomba (succión): 5/16" con válvula check, desde tanque
   - Salida bomba: 5/16" hasta filtro → regulador → riel
   - Retorno: 3/8" desde regulador a tanque (baja presión)
   - Relé bomba 5-pin:
     - Coil: desde Speeduino D7 + GND
     - Contacto NC: +12V batería fusible 30A → Bomba + (a través de relé)
     - Contacto NC: Bomba - → GND
   - **Presión: 43-45 PSI (NA), regulador ajustable**

9. **Inyectores**
   - Riel: +12V batería (a través de relé, OR permanente si arranca manualmente)
   - Inyector 1 (-): Speeduino pin 11
   - Inyector 2 (-): Speeduino pin 12
   - Inyector 3 (-): Speeduino pin 13
   - Inyector 4 (-): Speeduino pin 44
   - Inyector 5 (-): Speeduino pin 45
   - Inyector 6 (-): Speeduino pin 46
   - **Cables trenzados, separados 6" de ignición**

### FASE 6: Sensores

10. **CKP (Crank Position Trigger)**
    - Hall sensor a damper trigger wheel
    - +5V regulado (aislado) → Hall pin Vcc
    - GND → star-point + capacitor 0.1µF
    - OUT → Optoacoplador (TLP521) → Speeduino pin 20 (INT4)
    - Pull-up 1kΩ a +5V en INT4
    - **Verifica en banco:** Gira crank manualmente, INT4 parpadea en TunerStudio

11. **MAP (Manifold Air Pressure)**
    - Sensor MPX4250 en múltiple (toma Schraeder)
    - +5V regulado ← 10µF + 0.1µF capacitores locales
    - GND → star-point
    - OUT ← 0.1µF capacitor ← Speeduino A1
    - **Prueba:** Apunta sensor a vacio, debe leer ~20 kPa idle

12. **IAT (Intake Air Temperature)**
    - Termistor NTC (GM o Bosch) en tubo admisión (después throttle)
    - Conecta a +5V regulado vía resistencia pull-up 4.7kΩ (o integrada en sensor)
    - OUT → Speeduino A2 ← capacitor 0.1µF
    - **Rango típico:** 20°C = ~2.5V, 50°C = ~1.5V (depende tabla calibración)

13. **TPS (Throttle Position)**
    - Potenciómetro 3-pin en butterfly throttle
    - +5V regulado → pin Vcc
    - GND → star-point
    - OUT (wiper) → Speeduino A3
    - **Calibración:** Idle = ~0.5V, WOT = ~4.5V (verifica sin saturar a 5V exacto)

14. **CLT (Coolant Temperature)**
    - Termistor NTC en bloque cilindros
    - +5V regulado vía resistencia pull-up 4.7kΩ
    - OUT → Speeduino A4 ← capacitor 0.1µF
    - **Rango:** Cold start = ~0.5V (fría), 90°C = ~1.2V (caliente)

15. **CMP (Cam Position) [OPCIONAL para secuencial]**
    - Hall sensor en polea árbol de levas (ó imán + sensor)
    - +5V regulado → Hall Vcc
    - GND → star-point
    - OUT → Optoacoplador → Speeduino pin 21 (INT5)
    - **Nota:** Sin CMP, usa batch-fire (suficiente para NA)

### FASE 7: Pruebas en Banco

16. **Energiza sin motor corriendo**
    - Conecta batería, verifica luces de potencia en Speeduino
    - Abre TunerStudio, conecta serial USB
    - Comprueba:
      - Voltaje sensores (A0-A5) tienen valores lógicos
      - Trigger CKP responde (gira crank manualmente)
      - MAP lee ~20 kPa en idle
      - CLT lee temperatura ambiente aprox
    - **Si no ves valores:** revisa conexiones, multímetro, capacitores

17. **Carga configuración base L28**
    - TunerStudio: File → Load
    - Base tune (disponible en forums Speeduino): "Nissan_L28_Base.msq"
    - Ajusta parámetros críticos:
      - Rpm ramp: 50-200 rpm
      - VE table: Todos valores → 30-40% (muy pobre para testear)
      - Ignition advance: 5° BTDC (seguro)
      - Fuel pressure: 43 PSI
      - Sensor calibrations: Verifica rango (0-1023 ADC vs kPa/°C)

### FASE 8: Arranque & Tuning Inicial

18. **Arranque en frío (sin arrancar aún)**
    - Llave ON, bomba priming = 1000ms (debe oír bomba)
    - Verifica presión combustible con manómetro: 43-45 PSI
    - **Si presión baja:** revisar bomba, filtro, retorno regulador

19. **Arranque motor**
    - Llave ON (bomba priming)
    - Girador de arranque → motor debe captar ignición + combustible
    - Si arranca: espera 10 segundos, apaga (ignición corta)
    - Si no arranca: **revisa spark (bujías gap) + combustible (inyectores pulsan)**

20. **Idle & Combustión**
    - Motor idle ~800 rpm, temperatura ~60-80°C inicial
    - Aumenta gradualmente: 1000 → 1500 → 2000 rpm (5 min)
    - **Afina con TunerStudio en vivo:**
      - Si rico (humo negro): baja VE table por 5-10%
      - Si pobre (moto irregular): sube VE table por 5-10%
    - Target: lambda 1.0 (stoichiometric) idle con wideband

21. **Dataloging & tuning**
    - Abre TunerStudio → Datalog (graba 10 min funcionamiento)
    - Exporta CSV, analiza con VE Analyze
    - Ajusta tabla VE por RPM & MAP carga
    - Itera: Drive test → log → ajuste → retest

---

## ESPECIFICACIONES FINALES (Checklist)

```
✓ Trigger wheel: 36-1, gap 2-3mm, Hall sensor
✓ Coils: EDIS6 o COP individual (smart coils)
✓ Spark plugs: NGK BPR5ES, gap 0.040"
✓ Fuel pump: Walbro 255 LPH (12V)
✓ Fuel pressure: 43-45 PSI regulator (NA)
✓ Injectors: High-Z (12Ω+), EV6 or similar
✓ CKP conditioning: Optoacoplador (recomendado)
✓ Sensor grounds: Star-point, capacitores 0.1µF locales
✓ Ignition wiring: 16 AWG silicona, twisted pair con ground
✓ Sensor wiring: 22 AWG shielded, separado 6" de ignición
✓ Power: 4 AWG rojo fusible 60A, 4 AWG negro star-point
✓ MAP calibration: 0-250 kPa (MPX4250)
✓ TPS range: 0.5-4.5V (idle-WOT)
✓ Wideband: Lambda display real-time durante tuning
✓ Firmware: Speeduino v0.4.4+ con soporte L28 36-1 pattern
```

---

## TROUBLESHOOTING RÁPIDO

| Síntoma | Causa Probable | Fix |
|---|---|---|
| No hay spark | CKP no lee, ignición inversa, coil sin poder | Verifica INT4 parpadea, polarity coil primaria, +12V en bobina |
| Combustión irregular | VE table demasiada pobre, inyector sucio | Sube VE 10%, limpia inyectores ultrasónico |
| Motor no arranca | Bomba no priming, presión baja, inyectores no pulsan | Verifica relé D7 activa, 43 PSI, pulse inyectores en TunerStudio |
| Lambda falsa lectura | Wideband sueldo, cable roto | Reseata sensor AEM, verifica serial baudrate |
| Vibración excesiva | Trigger wheel desbalanceado después soldar | Rebalancea damper en taller |
| Múltiples trigger errors | Ground malo, ruido eléctrico en señal | Agrega optoacoplador, verifica twists en cables |

---

**Last Update:** 2026-04-19  
**Version:** 1.0 - L28E NA Wasted Spark  
**Reference:** Speeduino Official Docs + Nissan L-Series Community
