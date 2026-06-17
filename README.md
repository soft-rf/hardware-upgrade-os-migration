# KB-001: Hardware Upgrade & OS Migration — Lenovo IdeaPad S145

## 📝 Ficha del Proyecto
* **ID de Incidente/Caso:** INC-2026-0602
* **Especialista a Cargo:** Rodrigo Rabanal Fernandez
* **Activo Tecnológico:** Laptop Lenovo IdeaPad S145-15AST (Modelo 81N3)
* **Destino/Usuario:** Entorno Hogareño / Familiar
* **Estado:** Completado y Funcional (Post-Optimización)

---

## 🔍 1. Diagnóstico Inicial y Síntomas del Sistema
El equipo (fabricado en 2016) ingresó al laboratorio presentando una **degradación crítica del rendimiento** bajo el entorno original de Windows 10 Home. 

### Síntomas detectados:
* **Tiempos de boot (arranque):** Superiores a los 5-8 minutos hasta alcanzar la estabilidad del escritorio.
* **Cuello de botella:** Latencia extremadamente alta en tareas de Entrada/Salida (E/S) debido al uso de un disco mecánico (HDD) como unidad principal del sistema operativo (uso del disco al 100% constante).
* **Limitación de CPU:** Procesador AMD A9-9425 (2 Cores, 3.10 GHz) saturado por servicios en segundo plano de Windows 10 y bloatware del fabricante.

* ---

## 🛠️ 2. Solución Implementada e Ingeniería de Discos

Para maximizar el ciclo de vida del procesador AMD A9 y eliminar por completo la latencia de almacenamiento, se prescindió del entorno Windows y se migró la arquitectura a **Ubuntu 26.04 LTS**, estructurando el almacenamiento bajo el estándar **GPT** en modo **UEFI nativo**.

### 🔧 Actualización de Hardware (Hardware Upgrade)
* **Reemplazo de Componente Crítico:** Se realizó el desarme del equipo para remover la batería dañada y se instaló un módulo de batería nuevo compatible con el Modelo 81N3.
* **Instalación de Unidad de Estado Sólido:** Se colocó una unidad SSD Western Digital de 480GB en el puerto M.2 NVMe (Bus PCIe), manteniendo el HDD original de fábrica en el puerto SATA como unidad secundaria.

### 💾 Distribución y Particionado Lógico (Storage Architecture)
Para garantizar el máximo rendimiento del sistema sin perder capacidad de almacenamiento, se aplicó una estrategia de almacenamiento híbrido:

1. **Aislamiento del Sistema Operativo (Unidad SSD NVMe - `/dev/nvme0n1`):**
   * Se asignó la raíz del sistema (`/`) y la partición de arranque de firmware (`/boot/efi`) en el SSD.
   * **Resultado:** Tasas de transferencia óptimas para el inicio del sistema operativo y la ejecución de aplicaciones.

2. **Reutilización de Almacenamiento Masivo (Unidad HDD SATA - `/dev/sda`):**
   * Se eliminaron por completo las particiones propietarias residuales de Windows (`MSR`, `Recovery`, `NTFS C:`).
   * Se unificó el espacio físico en un único volumen de 1TB formateado bajo el sistema de archivos nativo de Linux (**Ext4**).
   * Se configuró el punto de montaje secundario de forma permanente en `/media/datos` para el almacenamiento de archivos del usuario familiar.

---

## 📈 3. Métricas de Rendimiento (Post-Optimización)

Para validar el éxito del proyecto, se realizaron pruebas de estrés y medición de tiempos de respuesta en el entorno final:

* **Tiempo de Arranque Inicial (Windows 10 original):** ~6 a 8 minutos hasta lograr estabilidad en el escritorio.
* **Tiempo de Arranque Final (Ubuntu 26.04 LTS):** ~1 minuto en promedio (reducción del 85% en la latencia de inicio).
* **Fluidez Operativa:** Fluida bajo carga de trabajo estándar (navegación web multi-pestaña, ofimática y reproducción de contenido multimedia). El HDD secundario no genera cuellos de botella al estar completamente aislado de los procesos del sistema central.

---

## 🚨 4. Fallas en Despliegue de OS y Troubleshooting (Resolución de Incidentes)

Durante el proceso de instalación y configuración, se detectaron y resolvieron de forma quirúrgica dos fallas críticas:

### 🛑 Incidente A: El medio de instalación USB no es detectado por la BIOS/UEFI
* **Síntomas:** Al encender el equipo e interactuar con el menú de booteo (F12), la placa madre no reconoce el pendrive con la imagen de Ubuntu.
* **Causa Raíz:** 1. El archivo `.iso` se había copiado mediante un arrastre de archivos común en un pendrive formateado en FAT32/NTFS, impidiendo que la UEFI localice un cargador de arranque (`bootloader`) válido en los sectores físicos.
  2. La función de seguridad **Secure Boot** en la placa madre estaba bloqueando de forma activa el instalador no verificado por las firmas de fábrica de hardware.
* **Resolución:** Se generó nuevamente el medio utilizando **Rufus/BalenaEtcher**, forzando la escritura de la imagen en modo ISO puro directamente sobre los bloques profundos del USB bajo esquema GPT. Posteriormente, se ingresó al firmware de la BIOS (pestaña *Security*) y se cambió el estado de **Secure Boot** a `Disabled`.

### 🛑 Incidente B: Interrupción de la instalación con error crítico `Input/output` en BusyBox
* **Síntomas:** El asistente de instalación gráfica se congelaba a mitad del proceso y redirigía a la terminal de emergencia de Linux (`BusyBox CLI`) con alertas de error de lectura/escritura física.
* **Causa Raíz:** Colapso del instalador provocado por sectores corruptos en la memoria flash del pendrive utilizado o inestabilidad en el voltaje del puerto USB frontal del chasis.
* **Resolución (Protocolo de Descarte Físico):** 1. Se reemplazó el medio de almacenamiento USB por un dispositivo de almacenamiento verificado.
  2. Se conectó el nuevo medio exclusivamente en los **puertos USB traseros** (soldados directamente a la placa madre) para asegurar un flujo eléctrico constante de 5V.
  3. Se realizó una verificación de integridad (`checksum`) a la ISO descargada desde la fuente oficial antes de flashearla. La instalación posterior concluyó con éxito.   
