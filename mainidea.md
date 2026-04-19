# Swap ECU moderna para Datsun 280ZX 1982 (L28E / L28ET)

Buen proyecto. El L28 es una plataforma ideal para standalone ECU porque tienes espacio de sobra, sensores baratos y la comunidad ha documentado casi todo. Te doy un roadmap realista con opciones por presupuesto.

## Primero: ¿qué motor tienes exactamente?

Esto cambia la estrategia:

- **L28E** (NA, inyección L-Jetronic analógica) — el caso más común en 280ZX 82
- **L28ET** (turbo, también L-Jetronic pero con AFM diferente) — si es turbo, vale la pena confirmar

Asumo **L28E NA** para el resto de la respuesta, pero el approach es casi idéntico para turbo (solo agregas MAP sensor de mayor rango y control de boost).

## Arquitectura objetivo

Lo que quieres lograr:

1. **Standalone ECU** (reemplaza la L-Jetronic y el módulo de ignición)
2. **Wasted-spark o COP (coil-on-plug)** — eliminas distribuidor
3. **Trigger wheel + CKP sensor** en el crank (elimina pickup del distribuidor)
4. **Inyectores modernos** de alta impedancia (saturados, fáciles de manejar)
5. **MAP sensor** en lugar del AFM (mejor respuesta, menos restricción de admisión)
6. **Wideband O2** para tuning

## Opciones de ECU por presupuesto

| ECU | Precio aprox (USD) | Pros | Contras |
|---|---|---|---|
| **Speeduino** (DIY, Arduino Mega) | $80-150 board + sensores | Open source, comunidad enorme, TunerStudio gratis, soporta COP/wasted spark, secuencial | Requiere soldar o comprar board ensamblada, documentarse |
| **rusEFI** (Proteus/Hellen) | $200-500 | Open source más moderno que Speeduino, hardware más robusto, CAN bus | Comunidad más pequeña en español |
| **MS3-Pro Ultimate** | ~$1,300 | Plug-and-play-ish, muy maduro, soporte comercial | Caro para el objetivo "sin gastar tanto" |
| **Haltech Nexus R3 / Elite 550** | $1,500-2,500 | Profesional, CAN, dashes integrables | Overkill y caro |
| **ECUMaster EMU Black** | ~$900 | Excelente relación calidad/precio pro | Más caro que Speeduino/rusEFI |

**Mi recomendación para tu caso: Speeduino o rusEFI.** Con tu perfil técnico (Python, Linux, homelab) vas a disfrutar el proceso y te ahorras $1000+. Speeduino específicamente tiene **toda** la documentación para L-series datsun porque es plataforma popular entre la comunidad JDM budget.

## Hardware de ignición (eliminar distribuidor)

Hay tres caminos:

**Opción A — Wasted spark (más barato, muy confiable)**
- 2 coil packs de 3 torres cada uno, o uno de 6 torres tipo LS/GM
- Coils de **Ford EDIS** (6 cilindros en línea EDIS6) — baratísimos en yonkes, ya son "smart coils" con ignitor integrado
- O coils LS truck (D585) con módulo de ignición externo o smart coils
- El distribuidor se tapa y se retira, o se deja como tapa ciega

**Opción B — COP (coil-on-plug) individual**
- 6 coils individuales tipo Yaris/Corolla 2ZR (baratos, smart coils, ~$15 c/u usados)
- O Audi/VW R8 coils (1.8T) — muy populares, smart, energía alta
- Necesitas fabricar brackets para montarlos directo sobre las bujías (elimina cables de bujía)
- Requiere ECU con 6 salidas de ignición (Speeduino con expansion board o rusEFI Proteus las tiene)

**Opción C — Single coil + distribuidor existente solo como repartidor**
- Mantienes el distribuidor mecánicamente pero cortas el pickup interno
- Control desde ECU con un solo coil
- **No lo recomiendo** — desperdicias la oportunidad de modernizar

**Mi pick: Opción A con coils EDIS6 o LS truck.** Costo ~$60-100 usados, confiabilidad brutal, wasted spark es más que suficiente para un L28 NA.

## Sistema de trigger (CKP)

Esto es crítico. Opciones:

1. **Trigger wheel 36-1 soldado/atornillado al damper** + sensor VR (reluctor variable) o Hall
   - Es el estándar. Ford usa 36-1, GM 24x, Nissan 36-2, etc.
   - Compras el wheel precortado (~$40-80) y un bracket para el sensor
   - Sensor VR genérico de Bosch o similar (~$20-30)
   
2. **Adaptar el CAS (Cam Angle Sensor) de un RB/VG30** al árbol del distribuidor
   - Más "JDM", pero Speeduino/rusEFI soporta nativo patrones Nissan
   - Requiere adaptador mecánico

**Pick: 36-1 en el crank con sensor Hall.** Hall es más limpio eléctricamente que VR, no requiere acondicionamiento de señal, y 36-1 es el patrón más documentado en Speeduino/rusEFI.

Si quieres ignición secuencial (no necesario para wasted spark pero sí para COP full sequential), agregas un sensor en el árbol de levas como CMP. Con un tornillo y un imán en la polea del cam + sensor Hall es suficiente.

## Inyección

- **Inyectores**: el L28E original usa low-impedance. Reemplaza por **high-impedance (>12Ω) EV1 o EV6**. Bosch 0280155884 (Ford 19lb) o Deka 60lb si planeas turbo después. ~$100-200 por set de 6 usados en buen estado.
- **Riel**: el stock sirve con adaptadores, o fabricas uno nuevo
- **Bomba**: Walbro 255 inline (~$80) reemplaza la original que seguro está cansada
- **Regulador de presión**: aftermarket ajustable (~$50)

## Sensores

Todos baratos y genéricos:

- **MAP**: MPX4250 (NA) o MPX4115, Speeduino los soporta nativo. ~$20
- **IAT**: GM open-element o Bosch genérico. ~$15
- **CLT**: GM o Bosch genérico. ~$15
- **TPS**: adaptar uno de Nissan moderno o genérico de 3 cables (~$25)
- **Wideband O2**: AEM X-Series o Innovate LC-2 (~$200). **Imprescindible** para tuning.

## Stack de software/tuning

- **TunerStudio MS** (gratis versión básica, $100 Pro con datalogging y autotune VE) — funciona tanto con Speeduino como rusEFI
- **MegaLogViewer** para analizar logs
- **VE Analyze Live** para autotune de la tabla de combustible

Con tu background Python, ten en cuenta que puedes **leer el CAN/serial de Speeduino y construir tu propio dash o telemetría** — hay librerías en Python para esto, se integra bien con tu homelab/Grafana si quieres loguear sesiones.

## Presupuesto realista (Speeduino + wasted spark + EDIS6)

| Componente | USD |
|---|---|
| Speeduino board ensamblada (v0.4.4 o similar) | $150 |
| Trigger wheel 36-1 + sensor Hall + bracket | $80 |
| Coils EDIS6 usados + arnés | $60 |
| Inyectores high-Z usados (6) | $150 |
| Bomba Walbro 255 | $80 |
| Regulador presión ajustable | $50 |
| Sensores (MAP, IAT, CLT, TPS) | $75 |
| Wideband AEM | $200 |
| Cable/conectores/relays/fusibles | $100 |
| TunerStudio Pro | $100 |
| **Total** | **~$1,045** |

Puedes bajar más si eres paciente con yonkes y eBay (~$700-800 es factible).

## Orden de ejecución sugerido

1. **Documenta el cableado stock antes de tocar nada** — fotos, multímetro, pinout del ECU original
2. **Instala trigger wheel y sensor CKP**, verifica señal con osciloscopio antes de conectar nada más
3. **Monta ECU en banco** con sensores simulados (potenciómetros) y valida que TunerStudio ve todo
4. **Cablea ignición primero**, arranca en modo "ignition only" manteniendo inyección stock temporal — o directo al full swap si prefieres
5. **Cablea inyección**, arranca con tabla VE base conservadora
6. **Tuning**: idle → cruise → WOT, con wideband
7. **Datalogs en Python/Grafana** para el toque nerd

## Gotchas específicos del L28

- El **bloque tiene una polea frontal con varias opciones** — asegúrate que el damper donde montes el 36-1 esté balanceado si lo modificas
- El **distribuidor** físicamente bombea aceite al árbol de levas en algunos L-series? **No**, en el L28 el distribuidor solo toma rotación, no tiene función de lubricación — lo puedes quitar sin problema tapando el hoyo
- **El alternador y arranque originales están bien** — no los toques
- **Considera upgrade de cabeza (N42, P79, P90)** si vas a abrir el motor, pero eso es otra historia
- **Tierra, tierra y tierra** — el Datsun 82 tiene tierras cansadas. Refresca todas antes de empezar, o vas a perseguir fantasmas eléctricos eternamente

## Ideas fuera de lo común que encajan con tu perfil

- **Dashboard digital con Raspberry Pi + pantalla táctil** leyendo serial/CAN del Speeduino, corriendo un frontend web (tu stack Python/FastAPI encaja perfecto). Puedes incluso integrarlo con tu homelab via Tailscale para ver datos del auto remotamente.
- **Telemetría en tiempo real a InfluxDB/Grafana** — loguea cada sesión de manejo
- **GPS + acelerómetro** vía otro microcontrolador para datos de pista
- **OTA updates del tune** via WiFi con ESP32 puente serial-WiFi al Speeduino

---

¿Quieres que profundice en alguno de estos puntos? Opciones naturales para seguir:

- Diagrama de cableado específico Speeduino → L28
- Comparativa detallada Speeduino vs rusEFI para tu caso
- Lista de compras exacta con links a vendedores confiables
- Plan de tuning paso a paso con TunerStudio
