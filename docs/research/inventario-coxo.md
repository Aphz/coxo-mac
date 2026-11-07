# Inventario: escáner intraoral COXO (fase 1)

**Fecha:** 2026-11-07 · **Autor:** worker de investigación (savvy-flow) · **Estado:** borrador para el orquestador

> Repo público: este documento no incluye números de serie, claves, datos de pacientes
> ni nombres de archivo que identifiquen pacientes. Las rutas locales son relativas a `~`.

**Leyenda de estado**
- **[C]** confirmado, con la fuente entre paréntesis (URL, archivo local o comando).
- **[S]** supuesto o inferencia, pendiente de verificar (casi siempre en el ThinkPad).

---

## 0. Resumen

| Tema | Hallazgo | Estado |
|---|---|---|
| Fabricante real | COXO **no fabrica** el escáner: es un **OEM de Guangdong Launca Medical Device Technology Co., Ltd.** (Dongguan). COXO lo distribuye con su marca, y también con la marca YSDENT. | [C] (IFU Breezyscan [F3]; hoja de requisitos de COXO que dice "Thank you for choosing Launca Oral Scanner" [F4]; instalador alojado en `launcapublish.s3.amazonaws.com` [F1]) |
| Modelos COXO actuales | **DL-300P** (= Launca DL-300P) y **Breezyscan** (= Launca Breezyscan). Variantes YSDENT de los dos. | [C] [F1][F2][F5] |
| Modelo de Nicolás | Desconocido. Lo más probable es un DL-300P (o un Launca anterior como el DL-206P), por las fechas de uso (desde 2025). | [S] |
| Software | Paquete Launca, rebautizado como "Intraoral Scanner Software Package". Tiene dos módulos: **Manager Setting** (administración) y **SCAN** (adquisición). Instalador `Setup_<fecha>_<build>`. Último build publicado para el DL-300P: `Setup_20250612_2584`, de unos 5,6 GB. | [C] [F1][F3][F5] |
| Requisitos | Windows 10/11 x64, **GPU NVIDIA GeForce RTX 4060 o superior** (no RTX xx50), **no compatible con GPU AMD ni con CPU Apple**, 16 GB de RAM, 2 puertos USB 3.0/3.2 y **Wi-Fi conectado de forma permanente**. | [C] [F4][F6] |
| Conexión | Cable USB 3.0. El Breezyscan usa USB-C ("Type-C USB3.0"). El chip USB (Cypress FX3, UVC, etc.) y el VID/PID son desconocidos. | [C] para el USB 3.0 [F3][F4]; [S] para el chip y los IDs |
| Formatos | STL, PLY y OBJ. Sistema abierto. El pedido se envía como STL o PLY por correo o por la nube Launca. | [C] [F3][F7] |
| Licencia | Sin suscripción anual según Launca. Se entra con cuentas de dentista locales; la nube Launca va ligada a un email. Exige internet ("otherwise the software will not function"). No se menciona dongle. | [C] [F4][F7] para lo citado; [S] el mecanismo exacto de activación |
| Versión macOS | **No existe.** Launca declara que no es compatible con Mac, y COXO, que no es compatible con CPU Apple ni con iMac. | [C] [F4][F7] |
| Hallazgo local clave | Exportaciones reales del escáner en el Mac (pedidos `Order_…_stl.zip` con `order.pdf` y `stl/upperjaw.stl`, etc.). Su estructura coincide con el software Launca. El PDF lo genera **iTextSharp 5.5.13.3**, una librería **.NET**. | [C] archivos locales; [S] la atribución al software COXO/Launca (probabilidad alta) |

---

## 1. Hallazgos locales en este Mac (solo lectura)

Búsqueda con `find -iname` (profundidad máxima de 5 a 6 niveles) en `~/Documents`, `~/Downloads`, `~/Desktop` y
`~/Library/Mobile Documents`. Se buscaron los patrones `*coxo*`, `*intraoral*`, `*dentscan*`, `*dental*`,
`*scan*`, `*.exe`, `*.msi`, `*.stl`, `*.ply`, `*.obj` y `*.dcm`.

### 1.1 Lo que NO hay
- Ningún instalador (`.exe`, `.msi`, `.zip` del software), driver, DLL ni manual PDF de COXO o Launca. [C]
- Ningún `.ply` de escaneo. [C]
- Nada relevante en `~/Desktop` ni en `~/Library/Mobile Documents` (fuera de lo que iCloud ya sincroniza en Documents). [C]
- Unos PDF con "Escaneo" en el nombre son documentos escaneados de una página, sin texto, y no son manuales del escáner. [C]

### 1.2 Exportaciones del escáner intraoral (con datos de paciente; se describen de forma genérica)
Ubicación: una carpeta de casos clínicos dentro de `~/Documents` (sincronizada con iCloud). Por ser repo público no se publican rutas exactas, fechas ni hashes de archivos de pacientes.

| Elemento | Detalle | Estado |
|---|---|---|
| ZIP de pedido descargado (1) | Nombre con el patrón `Order_<Clínica>_<Paciente>_<ID de 19 dígitos>_stl.zip`, unos 38 MB. | [C] |
| Contenido del ZIP | `Order60/order.pdf` (142 KB), `Order60/stl/upperjaw.stl` (24,5 MB), `lowerjaw.stl` (22,0 MB) y `buccal1.stl` (37,6 MB). | [C] (`unzip -l`) |
| Carpeta `Order88/` | `order.pdf` (~147 KB) más `stl/` con `upperjaw`, `lowerjaw`, `buccal1`, `buccal2` y `upperimplant.stl`. Los STL están solo como placeholders de iCloud y no se descargaron. | [C] |
| Otro pedido | Un segundo `Order_…_stl.zip`, solo como placeholder de iCloud. | [C] |
| Formato STL | **STL binario** con la cabecera de 80 bytes **en cero** (sin firma de software). Triángulos: 489.711 (upper), 440.408 (lower) y 751.031 (buccal1). Tamaño = 84 + 50·n, sin atributos de color. | [C] (`xxd`) |
| `order.pdf` | PDF 1.4, tamaño A3, 2 páginas. Producer: **`iTextSharp 5.5.13.3 (AGPL-version)`**. Título en la UI en español: "Pedidos electrónicos". Campos: "Archivo de modelo de maxilar superior/inferior", "Modelo de Scanbody Superior/Inferior", "Modelo bucal", "línea de margen", "foto". No aparece el nombre ni la versión del software, ni el número de serie del equipo. | [C] (`pdfinfo` y `pdftotext`, leídos en el scratchpad y luego borrados) |
| Atribución | Las categorías (upper jaw, lower jaw, buccal, upper/lower scanbody → `upperimplant`) y el flujo "Create Order / Send Order" del IFU Launca [F3] coinciden 1:1 con estos archivos. El español es uno de los 6 idiomas del software [F3]. | [S] (alta) |
| Implicancia técnica | iTextSharp es una librería .NET: al menos el módulo que genera los pedidos es **.NET (C#)**. | [C] para el PDF; [S] para la app completa |

Otros archivos 3D en la misma carpeta, que **no** vienen del escáner:
- `Modelo.stl`: cabecera `exocad GmbH 2024 - DentalCAD.v9036`. Es un diseño hecho en **exocad DentalCAD** (probablemente por el laboratorio), así que el laboratorio usa exocad. [C] cabecera; [S] origen
- `Molde Lab.stl`: cabecera "Made with Shapr3D". [C]
- Varios `.stl` más, solo como placeholders de iCloud (algunos con nombres de pacientes; aquí no se listan). [C]


### 1.3 Hardware de este Mac (para evaluar compatibilidad)
`system_profiler`: **MacBookPro14,3**, Intel Core i7 de 4 núcleos a 3,1 GHz (7ª gen. Kaby Lake, probablemente i7-7920HQ), 16 GB de RAM,
**Radeon Pro 560 de 4 GB** e Intel HD 630. No tiene GPU NVIDIA. macOS 13 (Darwin 22.6). [C] (comando local; el modelo exacto de CPU es [S])

---

## 2. Fabricante, marcas y modelos

| Dato | Estado |
|---|---|
| Foshan COXO Medical Instrument Co., Ltd. (desde 2003): fabricante de instrumental dental (motores de endodoncia, localizadores de ápice, piezas de mano, etc.). | [C] [F5] |
| El IFU del Breezyscan publicado por COXO dice: "Thank you for purchasing the Intraoral Scanner manufactured by Guangdong Launca Medical Device Technology Co., Ltd." y nombra a Launca como "Manufacturer" (Songshan Lake, Dongguan). COXO aparece al final, como distribuidor. | [C] [F3] |
| La hoja "Computer Configuration Requirements" de COXO lleva el logo de COXO y el texto "Thank you for choosing Launca Oral Scanner". | [C] [F4] |
| El instalador del DL-300P que COXO enlaza está alojado en el bucket S3 de Launca (`launcapublish.s3.amazonaws.com/Setup_20250612_2584.zip`, con URL prefirmada). | [C] [F1] |
| Modelos que COXO vende hoy: **DL-300P** y **Breezyscan**. YSDENT (otra marca de COXO) vende "YS DL-300P" y "Breezyscan YSDENT". | [C] [F5] |
| Gama Launca: DL-100/DL-150 (2018), DL-200/200P, DL-206/206P (2020), DL-300, **DL-300P**, DL-300 Wireless (2023) y **Breezyscan**. | [C] [F8][F9]; las fechas son de un folleto citado por un buscador, [S] |
| DL-300P: LED, 30 fps, profundidad de 0 a 18 mm, puntas de 20×17 y 17×14,5 mm, 180 g, "10 µm" (en otro texto, 15 µm). | [C] [F1] |
| Breezyscan: triangulación con autocalibración, **LED + láser violeta de 405 nm** (clase 1), 30 fps, profundidad de −1 a 21 mm, 130 g, **un solo cable USB-C**. | [C] [F2][F3] |
| Estudio independiente (MDPI 2025): comparó la precisión del COXO DL-300P con la de Medit i700 y Primescan. | [C] [F10] (no se leyó en detalle) |
| No es un rebrand de Shining 3D, Medit, Panda/Freqty ni Runyes. El OEM es Launca. | [C] por las pruebas de arriba |

---

## 3. Software

| Dato | Estado |
|---|---|
| Nombre comercial en COXO: "DL-300P Intraoral Scanner Software Package" y "Breezyscan Intraoral Scanner Software Package". No tiene un nombre de producto propio (en Launca se le llama "DL-300 software"). | [C] [F1][F2][F11] |
| Arquitectura según el IFU: dos aplicaciones, **Manager Setting** (cuenta de administrador: clínicas, dentistas, laboratorios, pacientes, carpetas de datos, idioma, región "China/Other Countries") y **SCAN** (login del dentista, paciente, caso, escaneo 3D, edición, exportación y envío del pedido). | [C] [F3] |
| Launca FAQ (DL-206): "the acquisition software is not stand-alone; it must be installed with Launca Management software". | [C] [F7] |
| Builds publicados en coxotec.com/download: DL-300P **`Setup_20250612_2584` [Standard], unos 5,6 GB** (notas: RTX serie 40 → driver NVIDIA 576.x; serie 50 → **driver NVIDIA 581.42 obligatorio**); "Overseas Orthodontic Version" **`Setup_20250612_2728_release`, 5,9 GB**; Breezyscan **`Setup_2864_COXO_IOS`, 296 MB** (y `Setup_2864_YSDENT`); "DL-300P One-click repair software" (enlaces regionales). | [C] [F6] |
| Versionado: fecha de compilación + número de build (`20250612_2584`). En la app se ve en "Version info" o "About". | [C] [F3][F6] |
| Funciones (blog Launca, feb-2024): AI Scan, flip, endoscopio integrado, análisis de retenciones, línea de margen, Health Report, grabación de video, Model Base para impresión. | [C] [F11] |
| Trae **TeamViewer** incluido para soporte remoto. "Anti-virus software is not compatible with our software". | [C] [F7] |
| Stack: el generador de PDF es .NET (iTextSharp). El peso de unos 5,6 GB y el driver NVIDIA obligatorio apuntan a que incluye **runtime CUDA** y librerías de reconstrucción en GPU. | [C] .NET (archivo local); [S] CUDA (fuerte, pero sin confirmar en binarios) |

---

## 4. Requisitos de sistema

Fuente principal: hoja oficial de COXO [F4] (imagen) y la página "COXO Oral Scanner Computer Specifications" [F6b].

| Componente | Requisito | Nota del fabricante | Estado |
|---|---|---|---|
| CPU | Intel Core i7/i9 de 12ª/13ª gen. (en la versión en texto, 10ª/11ª gen.), ≥ 3,0 GHz base; o AMD R7 5800H o superior | "Not compatible to Apple CPU" | [C] [F4][F6b] |
| GPU | **NVIDIA GeForce RTX 4060 o superior** | "Not compatible to AMD GPU". "Don't support RTX xx50 series GPU (eg. 3050 4050 5050)" | [C] [F4] |
| RAM | 16 GB o más | | [C] [F4] |
| Pantalla | 1920×1080 o más | | [C] [F4] |
| SO | Windows 10/11 (la versión en texto dice "Windows 7/10/11") | "Not compatible to Apple IMAC" | [C] [F4][F6b] |
| USB | "USB 3.0/3.2 (at least 2)". Breezyscan: "Type-C USB3.0" | | [C] [F3][F4] |
| Red | "ensure that the computer remains connected to WIFI and the signal is stable; otherwise, the software will not function" | | [C] [F4][F6b] |
| Energía | Conectado a la corriente y en modo de máximo rendimiento | | [C] [F4] |
| Generación anterior (DL-206, FAQ de Launca) | i7, 16 GB, **NVIDIA GTX 1060**, 2 USB (al menos 1 USB 3.0), Windows 10 y 7. "It is not compatible with Mac operating systems" | Los builds antiguos pedían menos GPU | [C] [F7] |

---

## 5. Conexión y hardware USB

| Dato | Estado |
|---|---|
| USB 3.0 por cable (DL-300P). El Breezyscan usa un solo cable USB-C "full-featured". | [C] [F2][F3][F4] |
| Existe un "DL-300 **Wireless**" en la gama Launca. COXO anuncia "First Wireless iOS", pero el DL-300P figura con "Wireless Range: N/A" en Launca. | [C] [F1][F8]; el modelo de Nicolás, [S] |
| Chip puente USB (Cypress/Infineon FX3, controlador propio, UVC): **sin información pública**. Muchos dispositivos de imagen dental usan Cypress FX3 con `cyusb3.sys` (ver [F12]), pero no hay evidencia de que Launca lo haga. | [S] |
| VID/PID: desconocidos. **Hay que leerlos en el ThinkPad** (ver §9). | [S] |
| Que el software exija NVIDIA y un procesamiento fuerte sugiere que el escáner envía imágenes crudas o patrones (no nubes de puntos ya calculadas) y que la reconstrucción 3D ocurre en el PC con GPU. | [S] (razonable para escáneres de "3D video", por verificar con captura USB) |

---

## 6. Formatos de salida e integración

| Dato | Estado |
|---|---|
| Salida: **STL, PLY, OBJ**. | [C] [F3][F9] |
| "Open system": funciona con cualquier sistema que acepte STL o PLY, incluidos laboratorios y fabricantes de alineadores. Un STL de arcada completa pesa unos 50 MB. | [C] [F7] |
| Envío del pedido por **correo** o por la **nube Launca**, en formato STL o PLY según la configuración del laboratorio en Manager Setting. El adjunto se puede cifrar con contraseña. | [C] [F3] |
| No se encontró mención de exportación DCM (3Shape) ni de integración directa con exocad o 3Shape Communicate. El laboratorio de Nicolás usa exocad (§1.2) y recibe STL. | [C] que no aparece en las fuentes revisadas; [S] que no exista |
| Estructura de exportación observada: `Order_<Clínica>_<Paciente>_<ID>_stl.zip` → `OrderNN/order.pdf` + `OrderNN/stl/{upperjaw,lowerjaw,buccal1,buccal2,upperimplant}.stl`. | [C] local; [S] que la genere esta app |

---

## 7. Licencia, cuentas y nube

| Dato | Estado |
|---|---|
| "No annual subscription fee". Las actualizaciones las entregan los distribuidores "with no additional cost". | [C] [F7] |
| Cuentas locales: el administrador, con contraseña, crea cuentas de dentista. La cuenta se activa al definir la contraseña en el primer login. | [C] [F3] |
| Nube Launca: para enviar pedidos por la nube, la cuenta del dentista debe vincularse a un email con código de verificación. "Cloud Orders" sincroniza los pedidos. | [C] [F3] |
| El software "will not function" sin Wi-Fi estable. Esto sugiere activación o validación en línea, o telemetría. | [C] la frase [F4]; [S] el motivo |
| Dongle: ninguna fuente lo menciona. Es probable que la licencia esté ligada al propio escáner (número de serie) o a la cuenta en la nube. | [S] |
| Regla del brief: no romper la protección de licencia. Cualquier camino debe usar el software oficial con su licencia, o hablar con el escáner sin tocar la protección del software. | — |

---

## 8. ¿Versión Mac? Alternativas y compatibilidad en un Mac

| Opción | Hallazgo | Estado |
|---|---|---|
| Versión macOS de Launca/COXO | **No existe.** Launca: "not compatible with Mac operating systems". COXO: "Not compatible to Apple CPU / Apple IMAC". | [C] [F4][F7] |
| SDK de Launca | No se encontró ningún SDK público. | [C] que no aparece; [S] que no exista |
| Otras marcas con app Mac | Shining 3D (Aoralscan 3, 3W, Elite, Elite Wireless, etc.) y Medit (i600, i700, i700W) tienen software para macOS, pero **solo para Apple silicon** y **solo con sus propios escáneres**. No sirven para hardware Launca. | [C] [F13][F14][F15] |
| VM (Parallels, VMware Fusion, VirtualBox) en este Mac Intel | La GPU de la VM es virtual (sin passthrough ni CUDA). Las apps que exigen NVIDIA/CUDA fallan ("Cuda is not installed"). Es inviable para la reconstrucción, aunque el USB passthrough sí funcione. | [C] [F16][F17] lo general; [S] aplicado a esta app |
| Wine / CrossOver / Whisky | macOS 13 no tiene drivers NVIDIA ni CUDA (NVIDIA no publica drivers después de 10.13, y CUDA 10.2 fue la última versión para macOS). Sin CUDA no hay reconstrucción. Además, un driver USB de kernel de Windows no corre en Wine. | [C] [F18] lo de CUDA; [S] el resto |
| Boot Camp (Windows nativo en este Mac) | El MBP 2017 admite **Windows 10** con Boot Camp. Windows 11 no corre por falta de TPM (según la comunidad). La GPU interna es **AMD Radeon Pro 560**, que el fabricante declara **no compatible**. La CPU es de 7ª gen., por debajo de la 10ª a 13ª que se pide. | [C] [F19] Win10; [C] la incompatibilidad AMD [F4]; [S] Win11 |
| Boot Camp + **eGPU NVIDIA por Thunderbolt 3** | Es la única forma de tener NVIDIA/CUDA en este Mac. Apple no da soporte oficial. En los MBP 15" 2016-2018 aparece el "error 12" de PCIe y hay que usar parches (desactivar la cámara FaceTime, scripts de reubicación PCIe). Además requiere una RTX 4060 o superior, y la CPU seguiría bajo el requisito. | [C] [F20] los problemas; [S] que el escáner funcione así |
| Reportes de usuarios corriendo escáneres Launca o COXO en Mac | No se encontró ninguno. | [C] que no hay resultados |

**Lectura para el orquestador (sin decidir estrategia):** el bloqueo principal no es Windows ni la arquitectura x86. Es
la **dependencia de una GPU NVIDIA (CUDA, muy probablemente)** del pipeline de reconstrucción, y este Mac no tiene
NVIDIA. Por eso Wine, Whisky, CrossOver y las VMs quedan prácticamente descartados para escanear (podrían servir para ver casos o
administrar pedidos, si la app arranca sin GPU, lo que está por verificar). Una reimplementación nativa exigiría replicar el
protocolo USB **y** la reconstrucción 3D (triangulación más registro en tiempo real), que hoy son una caja negra.

---

## 9. Incógnitas que solo Nicolás puede resolver desde el ThinkPad

Nada de lo siguiente es irreversible. Al compartir capturas, **tapar el número de serie y cualquier dato de paciente**.

1. **Modelo exacto del escáner:** foto de la etiqueta de la pieza de mano o de la caja (modelo, por ejemplo DL-300P, Breezyscan, DL-206P o DL-300 Wireless; marca COXO, YSDENT o Launca), con la serie tapada. ¿Conecta por cable USB-A, USB-C o es inalámbrico?
2. **Versión del software:** en Manager Setting → "Version info" o en SCAN → "About". Anotar el build (`Setup_YYYYMMDD_NNNN`).
3. **Instalación:** captura o listado (`dir /s /b` o `tree /f`, sin copiar binarios) de la carpeta de instalación (`C:\Program Files\…` o `C:\Program Files (x86)\…`, carpeta COXO o Launca) y de los accesos directos del menú Inicio. Interesa ver si hay `cudart64_*.dll`, `cudnn*`, `onnxruntime*`, `Qt5*`/`Qt6*`, `*.exe.config` (.NET) y `cyusb*`/`*.inf`.
4. **GPU, CPU, RAM y Windows del ThinkPad:** salida de `dxdiag` (guardar en .txt) o de Configuración → Sistema. Si hay NVIDIA, también `nvidia-smi` (driver y versión de CUDA). Si el ThinkPad **no** tiene NVIDIA y el escáner funciona igual, eso cambia todo el análisis.
5. **Identificación USB:** con el escáner conectado, en Administrador de dispositivos → dispositivo del escáner → Propiedades → Detalles → "Id. de hardware" (**VID/PID**), "Servicio" y "Proveedor/archivo .inf" del driver, y en qué categoría aparece (Cámaras, Dispositivos de imágenes, Controladoras USB, WinUSB). Opcional: captura de pantalla de USBTreeView.
6. **Activación y licencia:** ¿pide login de administrador o de dentista al abrir? ¿la cuenta está ligada a un email (nube Launca)? ¿hay un dongle USB aparte del escáner? ¿hay archivos `*.lic` o `license*` en la carpeta? (solo el nombre, no el contenido). ¿Qué pasa si se abre el software sin internet? Probar solo abrirlo, sin escanear a un paciente.
7. **Carpetas de datos:** en Manager Setting → General, las rutas "Scan data storage directory" y "Order data storage directory". Además, qué tipo de archivos guarda un escaneo antes de exportar (extensiones y tamaños, sin abrirlos).
8. **Envío al laboratorio:** formato configurado (STL o PLY) y método (correo o nube). ¿El laboratorio recibe los pedidos por correo (así llegaron los `Order_…_stl.zip` al Mac)?
9. **Software acompañante:** ¿está instalado TeamViewer? ¿hay antivirus desactivado por indicación del distribuidor? ¿qué "Region" está configurada (China u Other Countries)?
10. **Origen del instalador:** ¿lo entregó el distribuidor en Chile (pendrive, link) o se bajó de coxotec.com? Nombre del distribuidor y contacto de soporte. Fecha de compra aproximada.
11. **Rendimiento real:** ¿el escaneo funciona fluido en el ThinkPad? Esto ayuda a calibrar si los requisitos publicados son mínimos reales o solo recomendaciones.

---

## Fuentes

- [F1] COXO, DL-300P Intraoral Scanner: https://coxotec.com/product/dl-300p-intraoral-scanner/ (incluye el link al instalador en `launcapublish.s3.amazonaws.com`)
- [F2] COXO, Breezyscan Intraoral Scanner: https://coxotec.com/product/breezyscan-intraoral-scanner/
- [F3] COXO/Launca, *Instruction for Use, Breezyscan* (PDF de 55 pp., versión A/1): https://coxotec.com/wp-content/uploads/2026/04/Instruction-for-Use_Breezyscan-COXO_A1.pdf (sha256 de la copia leída: `07d4c5cf294f98e2d4bd15c1e55ab601a01e0df0dfb648c0fa1f70dcf6e05657`)
- [F4] COXO, hoja "Computer Configuration Requirements" (imagen): https://coxotec.com/wp-content/uploads/2025/02/computer-requirements.jpg (sha256 `fac192a9004944ea4beaa75253c39aa5df21c14b4593e8f3b30356bcd9e772f7`)
- [F5] COXO, catálogo de escáneres intraorales: https://coxotec.com/product-category/intraoral-scanner/ · YSDENT: https://coxotec.com/product-category/ysdent-products/ys-intraoral-scanner/
- [F6] COXO, Download: https://coxotec.com/download/
- [F6b] COXO, Oral Scanner Computer Specifications: https://coxotec.com/about-us/coxo-oral-scanner-computer-specifications/
- [F7] Launca, FAQs: https://launcadental.com/faqs
- [F8] Launca, DL-300P: https://launcadental.com/launca-dl-300p-intraoral-scanner-camera-product · folleto: https://www.launcadental.com/uploads/brochure-compressed3.pdf
- [F9] Launca, DL-300: https://launcadental.com/dl-300-intraoral-scanner-product
- [F10] MDPI *Medicina* 62(4):622, "Clinical Evaluation of Accuracy and Speed Across Three Intraoral Scanners": https://www.mdpi.com/1648-9144/62/4/622
- [F11] Launca, blog del software DL-300 (feb-2024): https://launcadental.com/blog/unlocking-the-potential-exploring-launca-dl-300-softwares-latest-features
- [F12] Ejemplo de dispositivo dental con Cypress FX3/`cyusb3.sys` (otra marca): https://community.infineon.com/t5/USB-low-full-high-speed/VisualiX-eHD-support-for-Windows-10/m-p/68373
- [F13] Shining 3D, Aoralscan Elite Wireless en macOS: https://www.shining3ddental.com/news-post/aoralscan-elite-wireless-now-compatible-with-macos-redefining-digital-dentistry-across-platforms
- [F14] Shining 3D, modelos de Mac compatibles: https://support.shining3ddental.com/en/support/solutions/articles/60001635307
- [F15] Medit, apps para Mac (Apple silicon): https://medit.com/the-medit-times-the-first-and-only-scanning-cad-apps-on-mac
- [F16] Parallels KB 122708, adaptador de video emulado: https://kb.parallels.com/br/122708
- [F17] Foro Parallels, app 3D que exige CUDA dentro de una VM: https://forum.parallels.com/threads/nvidia-cuda-driver.347536/
- [F18] Fin de CUDA y de los drivers NVIDIA en macOS: https://mjtsai.com/blog/?p=27347 · https://forums.developer.nvidia.com/t/when-will-the-nvidia-web-drivers-be-released-for-macos-mojave-10-14/65895/1429
- [F19] Boot Camp en MBP 2017 (Windows 10; Windows 11 sin TPM): https://discussions.apple.com/thread/8010209 · https://communities.apple.com/fr/thread/255727448
- [F20] eGPU en Boot Camp con MBP 15" y "error 12": https://georgegarside.com/blog/technology/boot-camp-windows-2004-egpu-macbook-pro-15-inch · https://egpu.io/automate-eGPU-EFI

## Anexo 2026-11-07 — Datos confirmados en el DentalPad
- Modelo: **DL-300P** [C] (Nicolás).
- Build instalado: `20250612_2496_release`, versión mostrada `V1`, actualizado 2025-06-20 [C] (captura de *Información de Versión*).
- Publicados por COXO: `20250612_2584` (Standard) y `20250612_2728_release` (ortodoncia) [C] [F6], o sea que hay builds más nuevos.
