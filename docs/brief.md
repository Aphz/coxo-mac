# Brief: port de la app del escáner intraoral COXO a macOS

**Dueño:** Nicolás (Aphz). **Creado:** 2026-11-07 por la Sesión maestra.

## Objetivo
Lograr que el escáner intraoral COXO funcione desde un Mac (MacBook Pro) en vez
de depender del ThinkPad con Windows. La app oficial solo existe para Windows.

## Herramienta
REA = paquete npm `rea-agents` (v6.8.0), instalado en el Mac en
`~/lytro-tools/rea` (binarios en `~/lytro-tools/rea/node_modules/.bin`, se activa
con `source ~/lytro-tools/env.sh`). En la nube se instala con `npm i rea-agents@6.8.0`.
Ya se usó en el proyecto Lytro A1 (repo `Aphz/lytro-a1`).

## Fases sugeridas
1. **Inventario**: qué entrega COXO (instalador, DLLs, drivers USB, protocolo,
   formatos de salida STL/PLY/OBJ, licencias/dongle). Identificar VID/PID del escáner.
2. **Análisis con REA**: arquitectura de la app, dependencias (Qt/.NET/DirectX/CUDA?),
   capa de comunicación con el hardware, pipeline de reconstrucción 3D.
3. **Decisión de estrategia**: (a) capa de compatibilidad (Wine/CrossOver/Whisky, VM
   Parallels con passthrough USB), (b) reimplementación nativa del protocolo + motor
   abierto, (c) híbrido. Documentar en `docs/research/estrategia.md` como ADR.
4. **Prototipo** del camino elegido; pruebas reales con el escáner conectado al Mac
   (eso lo hace Nicolás en persona; la sesión prepara todo y da pasos claros).

## Reglas
- Trabajar con el orquestador `/savvy-flow` y sus subagentes (workers).
- Nada irreversible sobre el hardware ni el ThinkPad sin OK explícito de Nicolás.
- No subir al repo instaladores, DLLs ni binarios de COXO; sí hashes, rutas y notas.
- Uso personal del equipo propio; respetar licencias: no romper protecciones de
  licencia ni redistribuir código propietario.
- Nicolás alterna entre el ThinkPad (Windows, donde está la app original) y el
  MacBook Pro. Pedirle archivos o capturas concretas cuando haga falta (listado de
  `Program Files\COXO...`, capturas USB con USBPcap/Wireshark, `lsusb`/System Report).
- Reportar en español: hallazgos en `docs/research/`, avance en `docs/bitacora.md`.
