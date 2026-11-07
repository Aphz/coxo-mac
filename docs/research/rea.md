# REA (`rea-agents` 6.8.0): qué es, cómo se usa y qué cubre para COXO

Verificación del 2026-11-07 en el MacBook Pro (macOS 13, Intel x86_64).
Leyenda: **[EJECUTADO]** = lo corrí y vi la salida; **[LEÍDO]** = sale del README,
del skill, de `--help` o del código de `dist/`, sin probarlo sobre un binario real.

## 1. Qué es

- REA ("Reverse Engineer Anything"), paquete npm `rea-agents`, licencia MIT,
  repo `github.com/morluto/rea`. Es **una CLI y un servidor MCP** que envuelven
  analizadores propios (estáticos, en Node) y motores externos (Ghidra, Hopper,
  IDA, Kuna, JADX, tshark…) y devuelven resultados JSON con "Evidence"
  (SHA-256 del artefacto, procedencia y limitaciones). [LEÍDO]
- Versión instalada: `rea --version` → `6.8.0`. [EJECUTADO]
- Binarios que aporta en `~/lytro-tools/rea/node_modules/.bin`: `rea` y
  `rea-agents`, los dos enlaces a `rea-agents/scripts/rea.mjs`. [EJECUTADO]
- Incluye un skill para agentes: `skills/reverse-engineer-anything/SKILL.md` con
  referencias (`native-and-artifacts.md`, `evidence-workflows.md`,
  `runtime-observation.md`, `connection-and-recovery.md`, etc.). [LEÍDO]
- Puentes incluidos: `bridge/ghidra/ReaGhidraBridge.java` (+ scripts de
  preparación), `bridge/hopper_bridge.py`, Cutter/Rizin, JADX, mitmproxy,
  pwntools/pwndbg, LLDB. Dependencia opcional: `frida`. [LEÍDO]
- Requiere Node 22.23.4+, 24.22.1+ o 26.12.1+. [LEÍDO]

## 2. Activación e invocación

```bash
source ~/lytro-tools/env.sh
rea --help                 # lista de comandos
rea <comando> --help       # ayuda por comando
rea providers --format json   # estado de motores (Ghidra, Hopper, IDA…)
```

`env.sh` solo exporta tres variables (sin secretos) [EJECUTADO]:

| Variable | Apunta a |
|---|---|
| `PATH` | antepone `~/lytro-tools/node/bin` (Node **v24.22.2**) y `~/lytro-tools/rea/node_modules/.bin` |
| `JAVA_HOME` | `~/lytro-tools/jdk25/Contents/Home` (**Temurin 25.0.5 LTS**, x86_64) |
| `GHIDRA_INSTALL_DIR` | `~/lytro-tools/ghidra/ghidra_12.1.4_PUBLIC` (**Ghidra 12.1.4**) |

Nota: `which java` sigue dando `/usr/bin/java` (el stub de macOS), que respeta
`JAVA_HOME`; REA usa directamente `$JAVA_HOME/bin/java`. [EJECUTADO]

Opciones globales útiles: `--format json|yaml|md|toon|jsonl`,
`--filter-output a,b.c`, `--token-limit N`, `--schema`, `--llms` (manifiesto
para LLM), `--mcp` (arranca como servidor MCP por stdio). [EJECUTADO, `--help`]

Como MCP: `rea setup` (asistente; `--dry-run`, `--client <agente>`, `--yes`)
registra el servidor y el skill en Claude Code/Codex/Cursor…; `rea mcp add` /
`rea mcp doctor`. **Hoy no está registrado como MCP en ningún agente**
(`smoke-doctor.json`: `registration:claude_code false`), así que lo usamos por
CLI. No ejecuté `setup` (modifica configuración). [LEÍDO]

## 3. Motores ("providers") disponibles en este Mac

`rea providers --format json` [EJECUTADO]:

| Proveedor | Estado | Para qué |
|---|---|---|
| `ghidra` | **available** (12.1.4, headless, decompilador nativo `mac_x86_64`, JDK 25 aceptado: "64-bit full JDK 21 or newer") | descompilación/desensamblado PE, ELF, Mach-O, etc. |
| `hopper` | available (`~/Applications/Hopper Disassembler.app`) | alternativa nativa; puede pedir diálogo demo/licencia |
| `ida` | no configurado (`REA_IDA_MCP_CONFIG`) | — |
| `kuna` | no configurado (`REA_KUNA_COMMAND`) | — |
| `rea-dotnet-static` | available | metadatos y CIL de ensamblados .NET |
| `rea-artifact-graph` | available | inventario/extracción de ZIP, MSIX/AppX, ASAR, DMG, PyInstaller… |
| `native-macos` | available | Mach-O, plist, firmas (no aplica a COXO) |

### Integración con Ghidra [LEÍDO, salvo lo marcado]

- REA lanza **`support/analyzeHeadless`** con su script puente
  (`ReaGhidraBridge.java`), importa el binario en un **proyecto temporal**, espera
  el auto-análisis completo y luego sirve consultas. El proyecto se borra al
  cerrar; nunca escribe en el binario original.
- Cada comando CLI suelto reimporta y reanaliza (en Lytro A1: 45-80 s por
  llamada en imágenes pequeñas). `--snapshot <archivo>` guarda/reutiliza un
  snapshot local del análisis para no repetirlo.
- Variables de ajuste: `REA_GHIDRA_STARTUP_TIMEOUT_MS` (plazo de arranque; en
  Lytro A1 el de 330 s no alcanzó para una imagen de 8 MB),
  `REA_GHIDRA_DECOMPILE_TIMEOUT_SECONDS`, `REA_GHIDRA_DISABLED_ANALYZERS`,
  `REA_GHIDRA_LANGUAGE_ID`, `REA_GHIDRA_COMPILER_SPEC_ID`, `REA_GHIDRA_SEED_FILE`,
  `REA_GHIDRA_NATIVEAOT_JAR`. (Nombres sacados de `dist/`.)
- Todos los chequeos Ghidra del doctor pasan: configuración, plataforma,
  arquitectura, versión, headless, decompilador nativo, java. [EJECUTADO, vía
  `rea providers`]

## 4. Comandos relevantes para COXO

De `rea --help` y ayudas por comando [EJECUTADO --help; comportamiento LEÍDO]:

**Panorama de la instalación**
- `inspect-desktop-application <dir>`: recorre la carpeta instalada (hasta 20 000
  entradas, 8 MiB de lectura) y clasifica candidatos de runtime por cabecera:
  PE nativo, PE/CLI (.NET), Electron/ASAR, Unity/IL2CPP, Unreal, Godot, GameMaker.
  No tiene detector específico de Qt; Qt se verá por los nombres de DLL/imports.
- `inspect-artifact <ruta>` / `extract-artifact <ruta>`: grafo de artefactos y
  extracción de ZIP/APK/IPA/MSIX/AppX, ASAR, DMG, Godot PCK, PyInstaller, Node SEA.

**PE nativos (x86 / x64), estáticos y sin Ghidra**
- `inspect-pe-imports` (DLLs importadas y delay-load, ordinales, IAT, imphash)
  — probado, ~4 s. [EJECUTADO]
- `inspect-native-exports`, `inspect-pe-resources` (recursos, iconos, versión),
  `inspect-pe-load-config` (TLS callbacks), `inspect-pe-certificates`
  (Authenticode), `inspect-byte-entropy` (detectar empaquetado),
  `inspect-binary-layout`, `inspect-gpu-code-objects` (CUDA/HIP embebido),
  `inspect-go-binary`, `inspect-rust-binary`.

**PE nativos con Ghidra (análisis profundo)**
- `analyze <exe|dll>` (panorama), `search <bin> <patrón> [--kind strings|procedures] [--mode regex]`,
  `trace <bin> "<literal>"` (sigue una cadena por referencias),
  `decompile <bin> <dir|nombre>`, `function <bin> <dir>` (dossier completo),
  `xrefs`, `instructions`, `inspect-native-api`, `trace-native-values`,
  `resolve-native-call-targets`, `decompile-all <bin> --output <dir>`.
  Todos aceptan `--provider ghidra` y `--snapshot`.

**.NET (PE/CLI)**
- `inspect-managed-artifact`: identidad, runtime, flags, R2R, referencias,
  recursos, atributos — probado, ~5 s. [EJECUTADO]
- `inspect-managed-members [--type Nombre.Completo]`: tipos, campos, métodos con
  firmas decodificadas, cuerpo IL (tamaño, conteo de opcodes, anclas), grafo de
  llamadas (`call_edges`), accesos a campos y literales `ldstr`. Probado sobre un
  DLL del SDK .NET 6: 119 tipos, 537 métodos, 1492 aristas de llamada. [EJECUTADO]
- `inspect-managed-native-boundaries`: **P/Invoke y fronteras managed→nativo**
  (clave para ver qué DLL nativa habla con el USB).
- `verify-managed-native-boundaries`, `compare-managed-members`,
  `project-managed-application-graph`.
- `import-managed-reconstruction`: importa salida de un descompilador externo
  (p. ej. ILSpy) y la contrasta con la evidencia. **REA no trae descompilador
  a C#.**

**Otros**
- `inspect-sqlite-database` (si la app guarda configuración/pacientes en SQLite).
- `inspect-packet-capture` / `inspect-pcap-capture`: existen (y aceptan campos
  como `usb.endpoint_address`), **pero el código los limita a Linux (y Windows
  para pcap)**: en macOS fallan. [LEÍDO en `dist/captures/TsharkCaptureProvider.js`
  y `dist/domain/pcap/pcapAnalysis.js`]

## 5. Capacidad sobre binarios Windows (resumen)

| Tipo | ¿Qué hace REA? |
|---|---|
| **EXE/DLL nativos x86/x64** | Sí. Estático propio (imports/exports/recursos/firmas/entropía) + Ghidra para pseudocódigo C, cadenas, xrefs, llamadas. Pseudocódigo ≠ fuente original. |
| **Ensamblados .NET** | Metadatos completos, firmas, **IL a nivel de opcodes/anclas**, grafo de llamadas, cadenas, P/Invoke. **No descompila a C#**; no usa ILSpy/dnSpy (solo importa su salida). NativeAOT: hay que ir por Ghidra. |
| **Drivers `.sys`** | Son PE nativos: los mismos comandos PE + Ghidra sirven. No hay módulo específico de drivers Windows (sí lo hay para módulos de kernel Linux). [LEÍDO] |
| **`.inf` / `.cat`** | Nada específico. `.inf` es texto (leer con `cat`; ahí estarán VID/PID). `.cat` es PKCS#7: no está cubierto. |
| **Instaladores MSI** | No. Solo existe un lector de ZIP/MSIX/AppX; MSI (OLE/CFB) no está entre los lectores de `dist/artifacts/`. [LEÍDO] |
| **Inno Setup / NSIS / InstallShield** | No los desempaqueta: los vería como PE con overlay. |
| **CAB** | No. |

## 6. Lo que NO cubre y con qué rellenarlo

Ninguna de estas está instalada hoy en el Mac (comprobado con `command -v`) [EJECUTADO]:

| Hueco | Herramienta sugerida (Homebrew salvo indicación) |
|---|---|
| Descompilar .NET a C# | `ilspycmd` (`dotnet tool install -g ilspycmd`; hay SDK .NET 3.1 y 6.0 en el Mac, verificar qué versión de ilspycmd corre con .NET 6) |
| MSI | `msitools` (`msiextract`, `msiinfo`) o `7zz` |
| Inno Setup | `innoextract` |
| NSIS / CAB / genérico | `sevenzip` (`7zz`), `cabextract` |
| Capturas USB (pcap de USBPcap) | Wireshark/`tshark` en el Mac directamente (REA no lo hace en macOS) |
| Análisis estático alternativo | `rizin` (REA lo puede usar si está en PATH), `binwalk`, `upx -d` si hay empaquetado UPX |
| Ejecutar/observar la app Windows | fuera de REA: Wine/CrossOver o VM (fase 3) |

## 7. Lecciones de Lytro A1 (`~/Documents/Proyectos/Lytro-A1`)

Fuente: `docs/research/firmware-analisis.md` de ese proyecto [LEÍDO]; productos
intermedios en `~/lytro-tools/analysis/` (scripts Ghidra en `gscripts/`).
- Se usó `rea inspect` y `rea search --kind procedures` sobre imágenes pequeñas:
  funcionó, 45-80 s por llamada.
- Para la imagen grande (8 MB) REA no cupo en el plazo de arranque de Ghidra;
  se pasó a **Ghidra headless directo** con proyecto persistente
  (`analyzeHeadless <proyecto> <nombre> -import ... -processor ...`, con
  `_JAVA_OPTIONS=-Xmx6g`) y scripts Java propios (`ListFuncs`, `DumpDecomp`,
  `XrefsTo`…): análisis completo 4-9 min, consultas ~25 s.
- Moraleja para COXO: REA para triage rápido y DLLs medianas; para el EXE
  principal o DLLs grandes, subir `REA_GHIDRA_STARTUP_TIMEOUT_MS`, usar
  `--snapshot`, o Ghidra headless con proyecto persistente.

## 8. Receta para la fase 2 de COXO

Supuesto: Nicolás copia `C:\Program Files\COXO...` (y el instalador, si lo hay)
a una carpeta local fuera del repo, p. ej. `~/coxo-work/install/`. Nada de esto
entra a git; solo hashes, rutas y notas.

```bash
source ~/lytro-tools/env.sh
W=~/coxo-work; I=$W/install; O=$W/rea; mkdir -p $O
cd $W   # trabajar fuera del repo

# 0. Inventario y hashes (para documentar sin subir binarios)
find "$I" -type f \( -iname '*.exe' -o -iname '*.dll' -o -iname '*.sys' -o -iname '*.inf' \) \
  -exec shasum -a 256 {} \; > $O/hashes.txt

# 1. Panorama: qué runtime es cada binario (.NET vs nativo, Electron, Unity…)
rea inspect-desktop-application "$I" --format json > $O/desktop.json

# 2. Por cada EXE/DLL: imports (Qt? DirectX? CUDA? WinUSB/libusb/SetupAPI/FTDI?)
for f in "$I"/*.exe "$I"/*.dll; do
  rea inspect-pe-imports "$f" --format json > "$O/imports_$(basename "$f").json"
done
rea inspect-pe-resources "$I/COXO.exe" --format json > $O/resources.json   # versión/fabricante
rea inspect-gpu-code-objects "$I/<dll_reconstruccion>.dll" --format json   # ¿kernels CUDA?

# 3. Si es .NET: metadatos, P/Invoke hacia DLLs nativas, cadenas
rea inspect-managed-artifact "$I/COXO.exe" --format json > $O/managed.json
rea inspect-managed-native-boundaries "$I/COXO.exe" --format json > $O/pinvoke.json
rea inspect-managed-members "$I/COXO.exe" --format json > $O/members.json
#    + C# legible (herramienta externa):  ilspycmd -p -o $W/ilspy "$I/COXO.exe"

# 4. Driver USB: VID/PID y servicio
find "$I" -iname '*.inf' -exec grep -iHE 'USB\\VID_|ServiceBinary|AddService|WinUSB|DeviceInterfaceGUID' {} +
rea inspect-pe-imports "$I/drivers/<driver>.sys" --format json

# 5. Capa de comunicación con el hardware (Ghidra vía REA), en la DLL nativa
export REA_GHIDRA_STARTUP_TIMEOUT_MS=1200000   # 20 min para binarios grandes
D="$I/<dll_usb>.dll"
rea analyze "$D" --provider ghidra --snapshot $O/usb.snap --format json > $O/usb_analyze.json
rea search "$D" "WinUsb_|DeviceIoControl|CreateFile|SetupDi|libusb" --mode regex \
    --kind procedures --provider ghidra --snapshot $O/usb.snap
rea trace "$D" "VID_" --provider ghidra --snapshot $O/usb.snap
rea decompile "$D" <direccion_o_nombre> --provider ghidra --snapshot $O/usb.snap

# 6. Instalador (si viene como .msi / Inno / NSIS): extraer con herramienta externa
#    msiextract -C $W/msi setup.msi   |  innoextract -d $W/inno setup.exe  |  7zz x setup.exe -o$W/7z
#    y repetir desde el paso 1 sobre lo extraído.

# 7. Capturas USB que traiga Nicolás (USBPcap en el ThinkPad): tshark/Wireshark en el Mac
#    tshark -r captura.pcapng -Y 'usb.idVendor' -T fields -e usb.idVendor -e usb.idProduct
```

Las rutas `COXO.exe`, `<dll_usb>.dll`, etc. son de ejemplo hasta tener el listado
real. Si `rea analyze` agota el plazo con el EXE principal, pasar a Ghidra
headless directo con proyecto persistente, como en Lytro A1.

## 9. Qué verifiqué y qué no

- **Ejecutado:** `source env.sh`; versiones de node/java/Ghidra; `rea --version`;
  `rea --help` y `--help` de unos 20 comandos; `rea providers --format json`;
  `rea inspect-managed-artifact`, `rea inspect-managed-members` y
  `rea inspect-pe-imports` sobre `Microsoft.DotNet.Cli.Utils.dll` del SDK .NET 6
  (un ensamblado .NET x64 del Mac, como sustituto de un binario COXO; salida en el
  scratchpad, nada escrito fuera); `command -v` de las herramientas complementarias.
- **Solo leído:** README, `package.json`, skill y referencias, código de
  `dist/` (lectores de artefactos, gating de pcap, variables Ghidra),
  `smoke-doctor.json`, docs de Lytro A1.
- **No probado:** Ghidra vía REA sobre un PE de Windows (no hay uno en el Mac y
  es un análisis largo); `rea setup`/registro MCP; `rea doctor` completo.
