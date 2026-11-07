# Prompt para arrancar la sesión «🦷 Port COXO → macOS»

Ajustes de la sesión: carpeta `~/Code/coxo-mac`, modelo **Opus 5.5**, esfuerzo **alto**, en la nube (repo `Aphz/coxo-mac`).

Pega esto como primer mensaje:

---

Eres la sesión coordinadora del proyecto **Port COXO → macOS** en este repo. Lee primero `docs/brief.md` y `README.md`. Reportas a la Sesión maestra de Nicolás.

Trabaja siempre con el orquestador `/savvy-flow` y sus subagentes: tú diriges, los workers investigan y escriben. Esfuerzo alto. Todo en español.

**Primera tanda:**
1. Verifica REA: en el Mac se activa con `source ~/lytro-tools/env.sh` (paquete npm `rea-agents` 6.8.0 en `~/lytro-tools/rea`); en la nube instálalo con `npm i rea-agents@6.8.0`. Anota en `docs/research/rea.md` cómo se invoca y qué puede hacer sobre binarios de Windows (PE, .NET, drivers).
2. Investiga en la web qué software entrega COXO para su escáner intraoral: nombre de la app, versión, requisitos, formato de salida, conexión USB, dongle o licencia en línea, y si existe port o alternativa para Mac. Escribe `docs/research/inventario-coxo.md`.
3. Redacta `docs/research/estrategia.md` como ADR con las opciones (Wine/CrossOver/Whisky, VM con passthrough USB, reimplementación nativa del protocolo, híbrido), riesgos y lo que necesitas de Nicolás para decidir.
4. Deja en `docs/bitacora.md` una entrada con lo hecho y una **lista corta de lo que Nicolás debe traer desde el ThinkPad** (listado de la carpeta de instalación, versión exacta, VID/PID del escáner, captura USB con USBPcap) para la fase de análisis con REA.

Reglas: nada irreversible en hardware ni en el ThinkPad sin OK explícito; no subir instaladores, DLLs ni binarios de COXO al repo; respetar licencias (no romper protecciones). Commits por hito. Al terminar, resumen de 10 líneas: hallazgos, estrategia recomendada y qué necesitas de Nicolás.
