# Guía de Instalación y Configuración de CLIAMP en WSL

![Interfaz de cliamp](image01.png)

Guia simple Para poder instalar [CLIAMP](https://www.cliamp.stream) en [WSL](https://learn.microsoft.com/es-es/windows/wsl/install)
Este tutorial fue generado con asistencia de Inteligencia Artificial (IA) para documentar el proceso exacto de instalación y configuración de cliamp en Windows Subsystem for Linux (WSL).

CLIAMP es una herramienta de streaming para Linux, este es un tutorial sirve solo para poder correr este programa en windows, **NO CREE EL PROGRAMA DE CLIAMP TODOS LOS CREDITOS EN (https://github.com/bjarneo/cliamp)** 

## 1 instalacion de wsl y ubuntu

1. Abrir PowerShell como Administrador.
2. Instalar WSL y Ubuntu con este comando:
   ```powershell
   wsl --install -d Ubuntu
   ```
3. Reiniciar la computadora.
4. Al reiniciar, se abrirá la terminal de Ubuntu. Configurar el nombre de usuario y la contraseña cuando lo solicite.
5. Volver a abrir PowerShell como Administrador y actualizar el motor de WSL para asegurar soporte de audio:
   ```powershell
   wsl --update
   ```

## 2 Instalacion de curl --> cliamp --> libasound2

1. Actualizar los repositorios de Ubuntu en la terminal de Linux:
   ```bash
   sudo apt update && sudo apt upgrade -y
   ```
2. Instalar `curl`, herramientas esenciales y las dependencias de audio base:
   ```bash
   sudo apt install -y curl nodejs npm libasound2 pulseaudio pulseaudio-utils
   ```
   *(Nota: Si usás Ubuntu 24.04 o superior, el sistema instalará `libasound2t64` automáticamente).*
3. Instalar el reproductor `cliamp` de forma global mediante NPM:
   ```bash
   sudo npm install -g cliamp
   ```
4. Generar los archivos internos de la aplicación:
   ```bash
   cliamp setup
   ```

## 3 arreglo de salida de audio

WSL no detecta tu tarjeta de sonido directamente. Necesitás apuntar el audio de Linux hacia el servidor de Windows (WSLg) para salir del silencio o corregir el error de conexión.

1. Vincular PulseAudio con el servidor de Windows ejecutando este bloque en tu terminal de Ubuntu:
   ```bash
   echo "export PULSE_SERVER=unix:/mnt/wslg/PulseServer" >> ~/.bashrc
   source ~/.bashrc
   ```
2. Verificar si el puente se activó correctamente:
   ```bash
   pactl info
   ```
   Si muestra los datos del servidor, el audio ya funciona. Si dice "Connection refused", ejecutá `wsl --shutdown` en PowerShell de Windows y volvé a abrir Ubuntu para reiniciar el sistema de sonido de fondo.

## 4 Arreglo opcional del Buffer

Si el streaming o las radios online sufren cortes, chasquidos o clips digitales debido a la latencia de la red de WSL, ejecutá este comando para forzar bloques de audio estables y eliminar las interferencias:

```bash
echo "export PULSE_LATENCY_MSEC=100" >> ~/.bashrc
source ~/.bashrc
```

## 5 Automatizacion opcional (Acceso directo desde Windows)

Si querés abrir `cliamp` con un clic desde el escritorio de Windows sin abrir la terminal de Ubuntu a mano, podés usar el script de automatización incluido en este repositorio.

1. Descargá el archivo **[cliamp.bat](cliamp.bat)** de este repositorio y guardalo en tu Escritorio de Windows.
2. Hacer clic derecho sobre el archivo descargado y seleccionar **Anclar a la barra de tareas** o dejarlo como acceso directo en el Escritorio.

*(Nota: Al ejecutarlo por primera vez, Windows puede mostrar una advertencia de SmartScreen. Hacé clic en "Más información" y luego en "Ejecutar de todas formas").*
