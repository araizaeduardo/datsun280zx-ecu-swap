# Datsun 280ZX Speeduino L28E - Retrofit ECU

Documentación técnica completa para integrar una unidad de control del motor (ECU) basada en Speeduino en un Datsun 280ZX clásico con motor Nissan L28E.

## Descripción General

Este repositorio contiene esquemas de cableado, mapeo de pines, procedimientos de calibración de sensores y un flujo de trabajo de sintonización completo para integrar Speeduino v0.4.4 (Arduino Mega 2560) en un Datsun 280ZX con gestión de motor L28E.

**Configuración Objetivo:**
- **Plataforma ECU:** Speeduino v0.4.4 en Arduino Mega 2560
- **Motor:** Nissan L28E (6 cilindros, naturalmente aspirado)
- **Encendido:** Configuración de chispa desperdiciada (wasted spark) con bobina EDIS6
- **Sistema de Combustible:** Inyección secuencial (opcional; batch fire aceptable para NA)
- **Retroalimentación de Sensores:** O2 de banda ancha, MAP, IAT, TPS, CLT, CKP, CMP

## Contenido

### [`SPEEDUINO_L28_PIN_REFERENCE.md`](SPEEDUINO_L28_PIN_REFERENCE.md)
La referencia técnica completa que abarca:

- **Mapeo de Pines** — Los 48 pines Speeduino con función, registros de control y conexiones físicas
- **Sistema de Encendido** — Patrón de fuego wasted spark (1→5→3→6→2→4), cableado de bobina EDIS6, circuitos primarios/secundarios
- **Sistema de Combustible** — Temporización de inyectores secuenciales, secuencia de priming de bomba, regulación de presión (43–45 PSI NA)
- **Sensores Analógicos** — CKP (disparador de cigüeñal), MAP (0–250 kPa), IAT, TPS (rango 0.5–4.5V), CLT, O2 de banda ancha
- **E/S Digital** — Lógica de relés para bomba de combustible, ventilador de radiador, luz de revisión del motor
- **Secuencia de Cableado** — Instalación en 8 fases desde punto de tierra hasta pruebas en banco y sintonización
- **Tablas de Calibración** — Rangos ADC de sensores, configuración TunerStudio, puntos de partida de tabla VE
- **Solución de Problemas** — 6 modos de falla comunes con causas raíz y soluciones

### [`speeduino_l28_wiring.html`](speeduino_l28_wiring.html)
Referencia de cableado visual y esquemas de diagrama para búsqueda rápida.

### [`CLAUDE.md`](CLAUDE.md)
Guía de desarrollo y mantenimiento de documentación para futuros colaboradores.

## Inicio Rápido

### Requisitos Previos

- Arduino Mega 2560 con bootloader Speeduino
- Motor Nissan L28E (o similar 6-cil. sin distribuidor)
- Rueda de disparador de cigüeñal dentada 36-1 + sensor Hall
- Módulo de encendido EDIS6 (o 6x bobinas inteligentes individuales)
- Bomba de combustible Walbro 255 LPH
- 6x inyectores de alta impedancia (12Ω+)
- Sensores: CKP, MAP, IAT, TPS, CLT (CMP opcional para secuencial)
- Sensor O2 de banda ancha + controlador
- Software TunerStudio para sintonización de ECU

### Ruta de Instalación

1. **Lee la secuencia de cableado** (Fases 1–7 en `SPEEDUINO_L28_PIN_REFERENCE.md`)
2. **Prepara el hardware** — Monta rueda de disparador, instala Speeduino, prepara punto de tierra
3. **Cablea todos los sistemas** — Encendido, combustible, sensores en orden prescrito (crítico para depuración)
4. **Verifica en banco** — Comprobación multímetro de cada voltaje e entrada de sensor antes de encender motor
5. **Carga tune base** — Usa configuración base L28 de la comunidad Speeduino (~30–40% tabla VE)
6. **Primer arranque** — Arranque en frío, prime combustible, monitorea datos en vivo TunerStudio
7. **Sintoniza ralentí** — Ajusta tabla VE ±5–10% por iteración; objetivo lambda 1.0 con banda ancha
8. **Datalog e itera** — Exporta CSV, herramienta VE Analyze, refina por celdas RPM/MAP

## Principios de Diseño Críticos

### Punto de Tierra Star-Point (No Negociable)
Todas las tierras de sensores y retornos de potencia convergen en un único punto en el bloque del motor. No distribuyas retornos de tierra—causa fallos esporádicos, errores de disparador y ruido de sensor.

### Aislamiento de Señal
- **Cableado de encendido:** 16 AWG silicona, par trenzado con tierra, mínimo 6" de separación del cable del sensor
- **Disparador CKP:** Aislado ópticamente (TLP521) para rechazar ruido eléctrico del bloque L28
- **Cableado de sensor:** 22 AWG blindado, capacitor 0.1µF en entrada ECU

### Autoridad de Calibración
Restricciones operacionales clave (inmutables sin cambio de hardware):
| Parámetro | Valor | Sensor |
|-----------|-------|--------|
| Rango MAP | 0–250 kPa | Freescale MPX4250 |
| Rango TPS | 0.5–4.5V | Potenciómetro 3-pin en acelerador |
| Disparador CKP | Rueda 36-1, brecha 2–3mm | Sensor Hall |
| Presión combustible (NA) | 43–45 PSI | Regulador en Walbro 255 |
| Línea base encendido | 5° BTDC | Arranque seguro; optimiza vía log |
| Objetivo lambda | 1.0 estequiométrico | Retroalimentación banda ancha en sintonización |

## Lista de Verificación de Cableado

- [ ] Punto de tierra star-point: 4 AWG desde batería, bloque motor, chasis, sensores (todos en una terminal M8)
- [ ] Potencia Speeduino: 4 AWG rojo → fusible 60A → relé 5-pin → Vin + GND
- [ ] Encendido (EDIS6): Pines 24, 26, 28, 30, 32, 34 → Primarias A, B, C, D, (E), (F)
- [ ] Inyectores de combustible: Pines 11–13, 44–46 → Lados de tierra de conectores EV6
- [ ] Relé bomba combustible: Speeduino D7 → relé 5-pin 12V → Walbro 255
- [ ] Disparador CKP: Sensor Hall → Optoacoplador → Pin 20 (INT4) + pull-up 1kΩ
- [ ] Sensor MAP: Freescale MPX4250 → A1 con bypass 0.1µF
- [ ] IAT, TPS, CLT, CMP: A2, A3, A4, A5 (A5 opcional) con capacitores locales 0.1µF

## Flujo de Trabajo de Sintonización

1. **Verificación en banco** (motor sin correr)
   - Multímetro: +12–13V en Vin, todos los ADCs de sensores dentro de rango
   - TunerStudio: Pin 20 CKP parpadea cuando se gira cigüeñal manualmente
   - Presión combustible: 43–45 PSI en riel (llave ON, priming de bomba)

2. **Arranque en frío** (motor apagado, 1–2 seg)
   - Confirma chispa en bujías (revisa brecha primero)
   - Confirma niebla de combustible de inyectores
   - Escucha breve combustión, apaga inmediatamente

3. **Estabilización de ralentí** (rampa 5 min)
   - 800 → 1000 → 1500 → 2000 RPM, monitorea aumento temperatura
   - Ajusta vivo tabla VE si corre rico (humo negro) o pobre (aspereza)
   - Objetivo: ralentí suave, lambda 1.0 con banda ancha

4. **Datalog e itera**
   - Registra 10–15 min de conducción normal (ralentí, crucero, aceleración ligera)
   - Exporta CSV, analiza con herramienta VE Analyze
   - Ajusta celdas que estén ±10% de estequiométrico
   - Retest, log, refina hasta lambda consistente

## Solución de Problemas

| Síntoma | Causa Raíz | Solución |
|---------|-----------|----------|
| Sin chispa | CKP no lee, polaridad encendido invertida, bobina sin poder | Verifica INT4 parpadea; revisa +12V EDIS6 y polaridad cableado primario |
| Ralentí áspero | Tabla VE muy pobre; inyector obstruido; fuga vacío | Sube VE 10%; limpia inyectores ultrasónico; prueba presión admisión |
| No arranca | Bomba no primes; presión baja; inyectores no pulsan | Verifica relé D7 activa; revisa 43 PSI; alcance pines inyector |
| Lectura falsa banda ancha | Sensor desconectado; desajuste baudrate serial; sensor sucio | Reseat AEM UEGO; verifica 9600 baud TunerStudio; reemplaza si hollín |
| Errores disparador (múltiples) | Lazo tierra; EMI en línea CKP; rueda disparador desequilibrada | Verifica grounding punto estrella; agrega optoacoplador si no hay; rebalancea rueda |

## Comunidad y Recursos

- **Speeduino Oficial:** https://speeduino.com/
- **Foros Speeduino:** https://speeduino.com/forum/
- **Comunidad Nissan L-Series:** Foros Datsun Roadster/Fairlady (comunidades autos vintage)
- **TunerStudio:** https://www.tunerstudio.com/

## Licencia

Esta documentación se proporciona tal cual para uso educativo y personal. Speeduino es de código abierto bajo licencia GPLv3.

## Contribuyendo

Para mejoras, correcciones o configuraciones de sensores adicionales, asegúrate de:
1. Todos los mapeos de pines coincidan con pinout ATmega2560 Speeduino v0.4.4
2. Los rangos de calibración se validen con hardware real
3. Los esquemas de cableado se prueben en motor en funcionamiento
4. Las entradas de solución de problemas incluyan síntoma → causa raíz → solución

---

**Última Actualización:** 2026-04-19  
**Versión:** 1.0  
**Hardware Objetivo:** Datsun 280ZX + Speeduino v0.4.4 + L28E NA
