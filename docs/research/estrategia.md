# ADR-001: estrategia para usar el escáner COXO desde un Mac

**Estado:** Propuesto · **Fecha:** 2026-11-07 · **Autor:** worker de estrategia (savvy-flow)
**Decide:** Nicolás, con los datos de la sección 8.

Leyenda (igual que en `inventario-coxo.md`): **[C]** confirmado con fuente; **[S]** supuesto o
inferencia. Las fuentes `[Fn]` son las de `docs/research/inventario-coxo.md`; las nuevas van como `[Gn]` al final.

## 1. Contexto

- El escáner es un **OEM de Launca** (DL-300P o Breezyscan, lo más probable), vendido con marca COXO. [C] (modelo exacto [S])
- La app oficial (Manager Setting + SCAN) solo existe para Windows. Al menos el módulo de pedidos es **.NET**. [C]
- Requisitos publicados: Windows 10/11 x64, **NVIDIA RTX 4060 o superior**, "Not compatible to AMD GPU",
  "Not compatible to Apple CPU", 2 puertos USB 3.0 y Wi-Fi siempre conectado. [C] [F4]
- La reconstrucción 3D casi seguro corre en **CUDA**: el instalador pesa unos 5,6 GB y exige un driver NVIDIA concreto. [S] fuerte
- Hasta la generación DL-206 se pedía NVIDIA (GTX 1060), así que la dependencia de NVIDIA no es nueva. [C] [F7]
- Hardware disponible:
  - MacBook Pro 15" 2017 (MacBookPro14,3): i7 de 7ª generación, Radeon Pro 560, 16 GB de RAM, macOS 13, Thunderbolt 3. [C]
  - Un segundo MacBook Pro, modelo sin confirmar. [S]
  - El ThinkPad que hoy corre la app; su GPU está sin confirmar. [S]
- El laboratorio usa exocad y recibe STL por correo. [C] origen; [S] flujo exacto

**El problema de fondo no es Windows ni x86. Es la GPU NVIDIA**, y ninguno de los Mac la tiene.

## 2. Fuerzas y restricciones

1. **Uso clínico.** El escaneo tiene que ser fiable y preciso. Una solución experimental no sirve para pacientes reales.
2. **Licencia y reglas del brief.** No romper protecciones, no redistribuir código propietario y nada irreversible en el escáner ni en el ThinkPad sin OK.
3. **Repo público.** No se publican binarios, firmware, datos de pacientes ni números de serie.
4. **Tiempo y dinero de Nicolás.** Prefiere algo que funcione pronto aunque no sea "puro Mac".
5. **USB 3.0 en tiempo real.** Unos 30 fps de imágenes de luz estructurada, sensible a la latencia y al ancho de banda.
6. **Wi-Fi permanente.** Probablemente hay validación en línea o nube [S], lo que complica el uso sin conexión y las VM aisladas.
7. **Ciclo de vida de Windows 10.** El soporte terminó en octubre de 2025 y las ESU para consumidores terminaron en octubre de 2026 [S], verificar.
   Esto importa porque el equipo maneja datos de salud.

## 3. Opciones

Escala: esfuerzo **B**ajo / **M**edio / **A**lto / **MA** (muy alto). La "probabilidad de éxito" es la de **escanear pacientes con el Mac**, estimada con los datos de hoy.

### A. Capa de compatibilidad (Wine / CrossOver / Whisky) en macOS
- **Pros:** sin reiniciar ni particionar. CrossOver tiene soporte comercial.
- **Contras:**
  - **No hay CUDA en macOS:** NVIDIA dejó de dar drivers después de 10.13 y la última versión de CUDA para macOS fue la 10.2 [C] [F18]. Además el Mac no tiene una NVIDIA que exponer.
  - Wine **no carga drivers `.sys` de kernel** de Windows. Solo hay soporte USB parcial para apps que usan WinUSB o libusb en modo usuario [S].
  - .NET Framework o WPF bajo Wine (Wine Mono o winetricks) funciona a medias [S].
  - **Whisky está discontinuado** desde abril de 2025 y solo existía para Apple Silicon [C] [G1].
  - **CrossOver 27 deja de soportar Mac Intel**; en el MBP 2017 el techo es CrossOver 26 [C] [G2].
- **Esfuerzo:** M. **Éxito (escanear):** < 5 %. Como visor o administrador de casos, si la app arranca sin GPU: ~20 % [S].

### B. Máquina virtual (Parallels / VMware Fusion / VirtualBox / UTM) con passthrough USB 3.0
- **Pros:** Windows real en una ventana. El passthrough USB 3.0 existe en Parallels y en VMware.
- **Contras:**
  - La GPU de la VM es virtual, **sin passthrough ni CUDA** en macOS; las apps con CUDA fallan [C] [F16][F17].
  - El USB isócrono o de alto caudal (cámaras a 30 fps) suele perder frames dentro de una VM [S].
  - La activación puede ligarse al hardware y pedir reactivación en la VM [S].
  - UTM en Intel usa QEMU y es lento.
- **Esfuerzo:** B-M. **Éxito (escanear):** < 5 %. Para Manager Setting o para ver casos: ~40 % [S].

### C. Boot Camp + Windows 10 en el MBP 2017 (con o sin eGPU NVIDIA por Thunderbolt 3)
No es un port: es **Windows nativo en hardware Apple**. Es el único camino en que "el Mac" ejecuta la app oficial completa.
- **C1, sin eGPU:** la Radeon Pro 560 es AMD, y el fabricante dice "Not compatible to AMD GPU". Si la reconstrucción es CUDA, falla. **Éxito:** < 10 % [S].
- **C2, con eGPU:** caja Thunderbolt 3 + RTX 4060 o superior.
  - **Pros:**
    - Windows nativo con CUDA real y driver USB nativo.
    - El ancho de banda de TB3 (≈ PCIe 3.0 x4) basta para cómputo; se pierde algo de rendimiento.
    - Es reversible: se puede borrar la partición.
  - **Contras:**
    - El "error 12" de PCIe es conocido en los MBP 15" 2016-2018 y se resuelve con parches de arranque (`apple_set_os`, automate-eGPU EFI) y desactivando dispositivos [C] [F20].
    - Apple no da soporte para eGPU en Boot Camp.
    - La CPU de 7ª generación está bajo el mínimo (10ª-13ª).
    - Windows 11 no corre de forma oficial (no hay TPM 2.0) y Windows 10 ya no recibe parches.
    - Costo aproximado de caja + GPU: USD 450-700 [S].
- **Esfuerzo:** M (hay que particionar e instalar Windows; no toca el escáner ni el ThinkPad). **Éxito (escanear):** ~50-65 % [S]. Los riesgos son el error 12 y la CPU.

### D. Reimplementación nativa (protocolo USB + reconstrucción 3D abierta en macOS)
Usaría libusb o AVFoundation para la captura y Metal, OpenCV u Open3D para la reconstrucción. Solo serviría para interoperar y no copiaría código.
- **Pros:** sería macOS de verdad y aprendería mucho. Es plausible para **capturar imágenes crudas** si el dispositivo es UVC estándar.
- **Contras:**
  - La reconstrucción **es el producto de Launca**: patrones de luz estructurada, calibración por equipo, registro en tiempo real (ICP o TSDF) e IA.
  - Llegar a precisión clínica (~10-20 µm) son **años-persona** de trabajo.
  - Si el chip es un Cypress FX3, es probable que el firmware se cargue desde el PC en cada conexión [S]. Habría que reusar un binario propietario, lo que es una zona gris.
  - La calibración puede estar cifrada o venir de la nube [S].
  - Hay riesgo legal: revisar la excepción de interoperabilidad de la Ley 17.336 de Chile [S].
  - Hay riesgo regulatorio: el resultado sería software médico no validado.
- **Esfuerzo:** MA. **Éxito (escanear con calidad clínica):** < 2 %. **Éxito (ver frames crudos en el Mac):** 10-40 % según la clase USB [S].

### E. Híbrido o remoto: el escáner en un PC Windows con NVIDIA y el Mac como pantalla y estación de trabajo
El escáner se conecta al ThinkPad (o a un mini-PC con RTX 4060). El Mac lo controla con Parsec, Moonlight + Sunshine, RDP (solo si el PC tiene Windows Pro) o el TeamViewer incluido.
Los STL o PLY llegan por carpeta compartida (SMB) o por la nube Launca. Opcionalmente, el Mac se encarga del post-proceso nativo: visor, renombrado, ZIP del pedido y envío al laboratorio con exocad.
- **Pros:**
  - Usa el software oficial **sin tocarlo**, en hardware soportado. Con un ThinkPad con NVIDIA, cuesta casi cero.
  - Funciona tanto con un Mac Intel como con uno Apple Silicon.
  - Es reversible.
- **Contras:**
  - **No elimina el PC Windows**; solo lo esconde. El PC tiene que estar junto al sillón, porque el cable USB va a la pieza de mano.
  - Escanear viendo un escritorio remoto agrega latencia: en LAN por cable, 10-30 ms con Parsec o Moonlight [S]. Por Wi-Fi puede molestar.
  - Si el ThinkPad no tiene NVIDIA (o es lento), hay que comprar un mini-PC: USD 900-1.300 [S].
- **Esfuerzo:** B. **Éxito (flujo usable desde el Mac):** ~85 % si el ThinkPad ya escanea bien [S].

### Resumen

| Opción | Esfuerzo | Costo | Éxito escaneando | Reversible | ¿Elimina el PC Windows? |
|---|---|---|---|---|---|
| A Wine/CrossOver | M | USD 0-74 | < 5 % | Sí | Sí (si funcionara) |
| B VM | B-M | USD 0-120/año | < 5 % | Sí | Sí (si funcionara) |
| C1 Boot Camp | M | USD 0 | < 10 % | Sí | Sí |
| C2 Boot Camp + eGPU | M | USD 450-700 | 50-65 % | Sí | Sí, pero hay que reiniciar a Windows |
| D Nativo | MA | Tiempo | < 2 % (clínico) | Sí | Sí, en teoría |
| E Híbrido o remoto | B | USD 0 a 1.300 | ~85 % | Sí | No |

## 4. Decisión recomendada (escalonada)

1. **Ahora: E.** Mantener el escáner en el ThinkPad, siempre que tenga NVIDIA y escanee bien hoy. Montar en el Mac:
   - (a) un escritorio remoto de baja latencia por cable o LAN, para quien quiera operar desde la pantalla del Mac;
   - (b) una carpeta compartida o la nube Launca, para que los pedidos `Order_…_stl.zip` lleguen al Mac sin pasos manuales;
   - (c) si suma valor, un flujo nativo de post-proceso en el Mac (vista previa, renombrado y envío al laboratorio).

   Este es el camino de menor riesgo clínico y de licencia.
2. **Mediano plazo, solo si el objetivo es quitar el ThinkPad del todo: evaluar C2** en el MBP 2017 (Boot Camp + eGPU RTX 4060).
   - Antes de comprar nada, probar C1 (Boot Camp sin eGPU), que no cuesta nada, para ver si la app arranca, si reconoce el USB y qué error da sin NVIDIA.
   - Si se compra, que sea a una tienda con devolución.
   - Alternativa más limpia: un mini-PC con RTX 4060, es decir, E sin el ThinkPad.
3. **Investigación acotada de D, solo si la fase 2 (REA más captura USB) muestra un protocolo simple**: clase UVC o bulk sin firmware descargable y sin cifrado. Aun así, el objetivo sería un **prototipo de captura y visualización**, no un reemplazo clínico. Límite sugerido: 2-3 sesiones.
4. **Descartar A y B para escanear.** Como mucho, B serviría para abrir Manager Setting o revisar casos en el Mac si la app arranca sin GPU. Sería un experimento de una tarde, no una solución.

## 5. Consecuencias

- Con E, Nicolás sigue manteniendo un equipo Windows. El "port" se convierte en **integrar el flujo**, no en reemplazar la app.
- Con C2, el Mac pasa a ser un Windows de escaneo cuando arranca en Boot Camp: hay que reiniciar para alternar, y la partición de Windows queda sin parches.
- La fase 2 con REA sigue siendo útil en cualquier escenario. Confirma CUDA o no CUDA y la clase USB, y decide si D vale la pena.

## 6. Riesgos

| Riesgo | Mitigación |
|---|---|
| La licencia o activación se rompe al reinstalar en otro equipo (C, B) | Preguntar antes al distribuidor y no desinstalar del ThinkPad hasta que el nuevo equipo funcione |
| Error 12 o eGPU inestable en el MBP 2017 | Probar con una caja y una GPU con devolución; seguir guías conocidas [F20]; volver a E si falla |
| Windows 10 sin parches con datos de salud | Red aislada o firewall, sin uso general, respaldo de los pedidos fuera del equipo |
| Latencia del escritorio remoto al escanear | LAN por cable, Moonlight/Sunshine o Parsec con códec por hardware; o escanear en el PC y revisar en el Mac |
| Datos de pacientes en el repo o en la nube por error | Las carpetas compartidas van fuera del repo y `.gitignore` se revisa; nunca se suben pedidos |
| D deriva a romper protecciones o reusar firmware | Límite explícito: solo observar tráfico propio; sin descifrar ni redistribuir binarios |

## 7. Qué cambia la recomendación ("si X, entonces Y")

| Dato desconocido | Si resulta… | Entonces… |
|---|---|---|
| GPU del ThinkPad | NVIDIA RTX y escanea fluido | **E** con el ThinkPad, a costo cero. C2 queda solo como opción si se quiere retirar el ThinkPad |
| GPU del ThinkPad | Sin NVIDIA (Intel o AMD) **y la app escanea igual** | La dependencia de CUDA es falsa o tiene alternativa en CPU u OpenCL: **C1** (Boot Camp sin eGPU) pasa a ser favorita, y vale probar B |
| GPU del ThinkPad | NVIDIA débil o el escaneo se corta | E con un mini-PC RTX 4060, o C2 |
| Segundo MacBook Pro | Apple Silicon (M1 o posterior) | Sin Boot Camp ni eGPU, y Windows ARM no tiene CUDA: **solo E** (o D) para ese equipo |
| Segundo MacBook Pro | Intel 2018-2019 con TB3 | Mejor candidato para C2 que el de 2017: CPU más nueva, aunque igual por debajo del mínimo |
| Clase USB y VID/PID | UVC estándar | D (captura de frames) se vuelve factible como experimento; la reconstrucción sigue siendo el muro |
| Clase USB y VID/PID | Driver propio (`cyusb3.sys` o similar) con firmware descargable | D queda descartada en la práctica; A y B pierden aún más sentido |
| Binarios (fase 2) | Sin `cudart`/`cudnn`; usa OpenCL, DirectX u ONNX en CPU | Revaluar C1 y B: la Radeon podría servir |
| Activación | Ligada al equipo o con número de reinstalaciones limitado | Priorizar E y no mover la instalación sin hablar antes con el distribuidor |
| Activación | Ligada a la cuenta o al escáner, y se reinstala libremente | C1 y C2 son de bajo riesgo de licencia |
| Objetivo real de Nicolás | "Solo quiero operar y recibir los archivos en el Mac" | E basta y se cierra el proyecto rápido |
| Objetivo real de Nicolás | "Quiero dejar de tener un PC Windows" | C2 o un mini-PC dedicado; D no llega a tiempo |

## 8. Qué necesitamos de Nicolás para decidir

1. **Objetivo:** ¿qué molesta del ThinkPad: el equipo, Windows, cargar dos notebooks, la pantalla o el rendimiento? ¿Basta con operar y recibir archivos en el Mac?
2. **GPU del ThinkPad:** modelo exacto del equipo, salida de `dxdiag` y, si existe, de `nvidia-smi`. ¿El escaneo va fluido?
3. **Modelo del segundo MacBook Pro:** Menú Apple → Acerca de este Mac (año y chip).
4. **USB del escáner:** VID/PID, categoría en el Administrador de dispositivos y nombre del driver o `.inf` (inventario §9.5). Sin número de serie.
5. **Licencia y activación:** ¿cómo se activó? ¿Qué dice el distribuidor de reinstalar en otro equipo? ¿Qué pasa al abrir la app sin internet? Solo abrirla, sin escanear.
6. **Listado de la carpeta de instalación** (inventario §9.3), para confirmar CUDA en la fase 2.
7. **Presupuesto y tolerancia:** ¿hay disposición a gastar unos USD 500-700 (eGPU) o USD 900-1.300 (mini-PC)? ¿Se acepta reiniciar el Mac en Windows para escanear?
8. **Red de la consulta:** ¿hay cable Ethernet o un Wi-Fi estable junto al sillón? Esto define si E es usable en vivo.

## 9. Próximos pasos

1. Nicolás responde la §8, en especial los puntos 1, 2 y 3, que bastan para elegir entre E y C.
2. Fase 2 (REA) sobre el listado o la copia de la instalación, para confirmar CUDA, el stack .NET y la capa USB. Resultado en `docs/research/`.
3. Si se elige E: preparar una guía paso a paso (Parsec o Moonlight, carpeta compartida, prueba de latencia con un modelo de prueba, nunca con un paciente).
4. Si se elige C: guía de Boot Camp en el MBP 2017 con un plan de reversión y, primero, la prueba C1 sin eGPU.
5. Actualizar `docs/bitacora.md` y pasar este ADR a **Aceptado** con la opción elegida.

## Fuentes nuevas

- [G1] Fin del desarrollo de Whisky (abril de 2025): https://heise.de/-10355657 · https://forums.appleinsider.com/discussion/239965
- [G2] CrossOver 27 deja los Mac Intel: https://www.parallels.com/blogs/crossover-drops-intel-mac-support-what-it-means-for-you/ · requisitos: https://www.codeweavers.com/products/crossover-mac/requirements
