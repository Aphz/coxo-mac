# Bitácora

- 2026-11-07 — Repo creado por la Sesión maestra con el brief inicial. Aún sin inventario del software COXO.

## 2026-11-07 — Primera tanda (sesión coordinadora)

**Hecho** (workers savvy-flow; repo público → sin datos de pacientes, seriales ni binarios):
- `docs/research/rea.md`: REA 6.8.0 verificado (`source ~/lytro-tools/env.sh` → `rea <cmd> --format json`;
  Node 24, JDK 25, Ghidra 12.1.4 headless). Sirve para PE x86/x64 (imports/exports, pseudocódigo vía
  Ghidra) y .NET (metadatos, IL, `ldstr`, P/Invoke → clave para hallar la DLL nativa del USB). No
  descompila a C#, no abre MSI/Inno/NSIS, `.sys` solo como PE, pcap no funciona en macOS. Faltan
  ilspycmd, innoextract/msitools/7zz, Wireshark (instalar con OK).
- `docs/research/inventario-coxo.md`: el escáner es un **OEM de Launca** (COXO DL-300P / Breezyscan).
  App Launca (Manager Setting + SCAN), build `Setup_20250612_2584` (~5,6 GB), exporta STL/PLY/OBJ.
  Requisitos: **NVIDIA RTX 4060+**, "Not compatible to AMD GPU / Apple CPU", 2× USB 3.0, Wi-Fi siempre.
  Sin versión Mac. Los pedidos exportados que hay en el Mac muestran que el módulo de pedidos es .NET.
- `docs/research/estrategia.md` (ADR-001, Propuesto): el obstáculo es la **GPU NVIDIA/CUDA**, no Windows.
  Recomendación: **E** ahora (escáner en PC Windows con NVIDIA, Mac por escritorio remoto + carpeta
  compartida), **C** a mediano plazo si se quiere retirar el ThinkPad (Boot Camp en el MBP 2017, con
  eGPU NVIDIA o mini-PC), **D** (protocolo nativo) solo como experimento acotado tras la fase 2.
  A (Wine/CrossOver/Whisky) y B (VM) descartadas para escanear.

Nota: la app COXO hoy está **solo en el ThinkPad "DentalPad"** (Windows). La sesión corre en el MacBook Pro;
Nicolás la opera a veces desde el DentalPad por sesión remota, así que puede reunir estos datos allí mismo.

### Pedidos para Nicolás (desde el ThinkPad / DentalPad)
1. **Versión exacta**: en la app, *About* / *Version info* (ej. `20250612_2584`) y modelo del escáner
   (etiqueta del equipo o caja: DL-300P, Breezyscan, DL-206…). **No** enviar el número de serie.
2. **GPU del ThinkPad**: modelo del ThinkPad, `dxdiag` (pestaña Pantalla) y salida de `nvidia-smi`;
   ¿escanea fluido hoy?
3. **Listado de la instalación** (solo nombres, tamaños y versiones; sin copiar binarios al repo). En PowerShell:
   `Get-ChildItem "C:\Program Files\<carpeta Launca/COXO>" -Recurse | Select FullName,Length,@{n='Ver';e={$_.VersionInfo.FileVersion}} | Export-Csv $HOME\Desktop\coxo-listado.csv`
   (si no sabes la ruta: clic derecho en el acceso directo de la app → *Abrir ubicación del archivo*).
4. **VID/PID del escáner**: Administrador de dispositivos → dispositivo del escáner → Propiedades →
   Detalles → *Id. de hardware* (`USB\VID_xxxx&PID_yyyy`), más *Servicio*, *Clase* y nombre del driver/`.inf`.
   Captura de pantalla de esa pestaña.
5. **Captura USB** (con OK, no toca el escáner): instalar Wireshark con USBPcap y grabar ~30 s de
   *conectar escáner → abrir SCAN → iniciar escaneo → detener*, guardar `.pcapng`. No escanear a un
   paciente real durante la captura.
6. **Licencia**: cómo se activó (cuenta, código, en línea), qué pasa si arranca sin internet.
7. **Decisión**: ¿basta con operar el escáner y recibir los STL en el Mac, o quieres retirar el PC Windows?
   Modelo del otro MacBook Pro (Intel o Apple Silicon) y si te sirve reiniciar el Mac en Windows para escanear.

Los archivos 3–5 van a una carpeta fuera del repo (p. ej. iCloud/AirDrop); acá solo hashes y notas.

## 2026-11-07 — Aclaración GPU del MacBook Pro
- Nicolás creía que su MacBook tiene i7 de 3 GHz con NVIDIA dedicada. Verificado con `system_profiler` en este Mac:
  MacBookPro14,3 (15" 2017), i7 4 núcleos 3,1 GHz, **Intel HD 630 + AMD Radeon Pro 560 4 GB — sin NVIDIA**.
- Puede referirse al otro MacBook Pro. Ningún MBP vino con una NVIDIA útil para esta app (los últimos con NVIDIA,
  2012–2014, traen GT 650M/750M, muy por debajo de RTX 4060 y sin drivers actuales). La opción C (Boot Camp en
  este MBP) sigue necesitando eGPU NVIDIA. Reevaluar cuando llegue el modelo del otro MacBook (pedido 7).

## 2026-11-07 — Pedido 1 recibido (versión y modelo)
- Modelo confirmado por Nicolás: **COXO DL-300P**.
- App en el DentalPad (pantalla *Información de Versión*, en español): versión `V1`, compilación
  `20250612_2496_release`, fecha de actualización 2025-06-20.
- Es **anterior** a los builds publicados hoy en coxotec.com/download (`20250612_2584` Standard y
  `20250612_2728_release` ortodoncia). "V1" es solo la etiqueta: el número de build sigue avanzando, así que la
  app no está abandonada. No actualizar sin OK: los builds nuevos exigen un driver NVIDIA específico (576.x/581.42).
- Iconos de la barra superior: cuenta, soporte remoto, nube/sincronización y Wi-Fi (la app depende de la red).

## 2026-11-07 — Pedido 2 recibido (equipo y GPU del DentalPad)
- Equipo: **Lenovo ThinkPad P15 Gen 1** (tipo 20SU). GPU: **NVIDIA Quadro T1000 4 GB** (Turing, TU117) + Intel UHD.
  Driver NVIDIA 595.71, CUDA 13.2 (WDDM).
- **Hallazgo clave:** la app DL-300P (build 2496) funciona con una T1000, que rinde aproximadamente como una
  GTX 1650 y queda muy por debajo de la "RTX 4060+" oficial. El requisito publicado es una recomendación, no un
  mínimo duro; lo que importa es tener **NVIDIA/CUDA**, no el modelo exacto. [S] fuerte (pendiente: ¿escanea fluido?)
- Consecuencia para el ADR: la opción C (eGPU) se abarata, porque una NVIDIA modesta (Turing o posterior, 4 GB)
  podría bastar. La opción E sigue siendo la base: el DentalPad ya funciona.
