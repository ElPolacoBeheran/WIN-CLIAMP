# 🎵 Guía Completa: Setup de WSL, Instalación de CLIAMP y Optimización de Audio

Esta guía paso a paso te enseñará cómo habilitar WSL (Subsistema de Windows para Linux), instalar el reproductor de música de terminal **cliamp** con todas sus dependencias y configurar el entorno para eliminar chasquidos ("clips") en el streaming de audio.

---

## 💻 1. Habilitación y Configuración de WSL

Si todavía no tenés Linux corriendo en tu máquina Windows, debés activar WSL 2 (que ya incluye el soporte nativo de interfaz gráfica y audio a través de WSLg).

1. Abrí **PowerShell** en Windows como Administrador.
2. Ejecutá el comando de instalación automática:
   ```powershell
   wsl --install
   ```
   *(Este comando descargará e instalará la distribución por defecto, que es Ubuntu, junto con el Kernel de Linux más reciente).*
3. **Reiniciá tu computadora** cuando finalice el proceso.
4. Al reiniciar, se abrirá una ventana de consola pidiéndote que configures un **nombre de usuario** y una **contraseña** para tu entorno Linux.

### Actualizar el entorno WSL
Para asegurarte de tener soporte de audio estable, ejecutá este comando en PowerShell:
```powershell
wsl --update
```

---

## 📦 2. Instalación de Dependencias y CLIAMP

Una vez dentro de la terminal de tu distribución de Linux (Ubuntu), procedé a preparar el sistema e instalar el reproductor.

### Paso A: Actualizar repositorios e instalar dependencias de sonido
`cliamp` requiere Node.js (ya que está construido sobre JavaScript/NPM) y las bibliotecas de desarrollo del sistema de audio ALSA (`libasound.so.2`).

Ejecutá los siguientes comandos en tu terminal de Linux:
```bash
# Actualizar el gestor de paquetes
sudo apt update && sudo apt upgrade -y

# Instalar Node.js, NPM y la biblioteca ALSA esencial
sudo apt install -y nodejs npm libasound2 pulseaudio pulseaudio-utils
```
*(Nota: Si usás una versión muy reciente de Ubuntu como la 24.04+, el paquete de sonido puede llamarse automáticamente `libasound2t64`)*.

### Paso B: Instalación global de CLIAMP
Con el entorno de Node.js listo, instalá el reproductor utilizando el gestor de paquetes `npm`:
```bash
sudo npm install -y -g cliamp
```

### Paso C: Primer inicio (Setup inicial)
Para que el reproductor genere sus carpetas de configuración internas por primera vez, ejecutá el siguiente comando:
```bash
cliamp setup
```

---

## 🔊 3. Enlazar el Servidor de Audio y Solucionar Silencios

WSL no accede al hardware de sonido directamente; se comunica con un servidor interno que transmite el audio hacia Windows. Si el reproductor inicia pero opera en silencio, seguí estos pasos.

### Paso A: Forzar la variable del servidor de audio
Agregá la ruta del servidor de audio de WSLg a tu configuración de usuario para que Linux localice la salida de Windows:
```bash
echo "export PULSE_SERVER=unix:/mnt/wslg/PulseServer" >> ~/.bashrc
source ~/.bashrc
```

### Paso B: Verificar la conexión
Comprobá si tu terminal se comunica correctamente con el motor de audio ejecutando:
```bash
pactl info
```
Si te devuelve información del servidor (en lugar de un error de conexión rechazada), tu sistema ya está listo para reproducir sonido.

---

## ⚡ 4. Eliminar "Clips" y Cortes en el Streaming

Si al reproducir música o radios online dentro de `cliamp` experimentás microcortes o chasquidos digitales molestones, se debe a que el tamaño de carga del búfer por defecto es muy chico para la latencia interna de WSL.

### Solución A: Modificar el búfer de CLIAMP
1. Abrí el archivo de configuración del reproductor:
   ```bash
   nano ~/.config/cliamp/config.toml
   ```
2. Buscá o agregá los parámetros de búfer dentro de la sección de audio para darle más margen de carga a las pistas:
   ```toml
   [audio]
   buffer_size = 4096
   stream_buffer = 10000
   ```
3. Guardá los cambios presionando **Ctrl + O**, confirmá con **Enter** y salí con **Ctrl + X**.

### Solución B: Estabilizar la latencia de PulseAudio
Forzá al puente de audio a procesar bloques de sonido ligeramente más grandes para amortiguar las fluctuaciones de red. Agregá esta variable a tu entorno:
```bash
echo "export PULSE_LATENCY_MSEC=100" >> ~/.bashrc
source ~/.bashrc
```

---

## 🚀 5. Automatización: Acceso Directo desde Windows

Para abrir tu música rápidamente sin necesidad de abrir la consola de Ubuntu y tipear comandos de manera manual, podés programar un botón de inicio en Windows.

### Archivo 1: `Abrir_cliamp.bat`
Creá un archivo de texto en tu Escritorio, pegá este código y guardalo con el nombre `Abrir_cliamp.bat`:
```batch
@echo off
wsl -e bash -c "sleep 2 && cliamp"
```

### Archivo 2: `Abrir_cliamp_Invisible.vbs`
Creá otro archivo de texto en el mismo directorio para ocultar la ventana negra parpadeante de Windows y guardalo como `Abrir_cliamp_Invisible.vbs`:
```vbs
Set WshShell = CreateObject("WScript.Shell")
WshShell.Run "cmd.exe /c %USERPROFILE%\Desktop\Abrir_cliamp.bat", 0, False
```

### Pasos para anclar a la Barra de Tareas:
1. Hacé clic derecho en tu Escritorio ➡️ **Nuevo** ➡️ **Acceso directo**.
2. En la ruta escribe: `wscript.exe "C:\Ruta\A\Tu\Archivo\Abrir_cliamp_Invisible.vbs"` (reemplazando con tu ruta real).
3. Asignale el nombre `CLIAMP Player`.
4. Hacé clic derecho sobre el acceso directo creado y seleccioná **Anclar a la barra de tareas**.
