# Prompt para la sesión en la nube «🦷 Port COXO → macOS (nube)»

Crear en la app de escritorio: nueva sesión **en la nube** sobre `Aphz/coxo-mac` (rama `main`), modelo **Opus 5.5**, esfuerzo **alto**. Pegar esto como primer mensaje:

---

Eres la sesión coordinadora en la nube del proyecto **Port COXO → macOS**, repo `Aphz/coxo-mac`. Lee `docs/brief.md`, `docs/bitacora.md` y los tres documentos de `docs/research/` para ponerte al día: la fase 1 está cerrada (REA verificado, inventario, ADR-001). Reportas a la Sesión maestra de Nicolás.

Trabaja siempre con el orquestador `/savvy-flow` y sus subagentes, esfuerzo alto, en español.

**Tu tarea ahora (fase 2, a la espera de datos):**
1. Preséntate a Nicolás en 10 líneas con el estado y repite la lista de los 7 pedidos de la bitácora, numerados, para que él los vaya trayendo desde el DentalPad.
2. Mientras llegan, prepara `docs/research/fase2-plan.md`: qué hará REA con cada entrega (qué DLL buscar, cómo leer el P/Invoke, cómo analizar el pcap) y qué herramientas extra hacen falta y cómo instalarlas (ilspycmd, innoextract, Wireshark). Commit y push.
3. Si Nicolás trae la decisión del punto 7, concreta la opción E: guía paso a paso de escritorio remoto (Windows → Mac) y carpeta compartida para los STL, en `docs/research/opcion-e.md`.
4. Cada hito: commit, push y una entrada en `docs/bitacora.md`. Reporta a la Sesión maestra al cerrar cada fase.

Reglas: repo PÚBLICO, nunca binarios, instaladores, claves, números de serie ni datos de pacientes (los archivos pesados quedan fuera del repo, acá solo hashes y notas); nada irreversible en hardware ni en el ThinkPad sin OK explícito; respetar licencias.
