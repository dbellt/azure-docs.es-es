---
title: Actualización de Azure Percept DK a través de una conexión USB
description: Aprenda a actualizar Azure Percept DK a través de una conexión USB
author: mimcco
ms.author: mimcco
ms.service: azure-percept
ms.topic: how-to
ms.date: 03/18/2021
ms.custom: template-how-to
ms.openlocfilehash: e8ab657ec7b4ef9a413993c064c931b4fc1523c5
ms.sourcegitcommit: bd1a4e4df613ff24e954eb3876aebff533b317ae
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/23/2021
ms.locfileid: "107929460"
---
# <a name="how-to-update-azure-percept-dk-over-a-usb-connection"></a>Actualización de Azure Percept DK a través de una conexión USB

Aunque el uso de actualizaciones inalámbricas (OTA) es el mejor método para mantener actualizado el sistema operativo y el firmware del kit de desarrollo, hay escenarios en los que es necesaria la actualización (o "parpadeo") del kit de desarrollo a través de una conexión USB:

- No es posible realizar una actualización de OTA debido a la conectividad u otros problemas técnicos.
- El dispositivo debe restablecerse a su estado de fábrica

En esta guía se muestra cómo actualizar correctamente el firmware y el sistema operativo del kit de desarrollo a través de una conexión USB.

> [!WARNING]
> Al actualizar el kit de desarrollo a través de USB, se eliminarán todos los datos existentes en el dispositivo, incluidos los modelos y contenedores de AI.
>
> Siga todas las instrucciones en orden. Omitir los pasos podría poner el kit de desarrollo en un estado inutilizable.

## <a name="prerequisites"></a>Requisitos previos

- Una instancia de Azure Percept DK
- Un equipo host basado en Windows, Linux u OS X con Wi-Fi capacidad y un puerto USB-C o USB-A disponible
- Un cable USB-C a USB-a (opcional, vendido por separado)
- Un inicio de sesión de SSH, creado durante la [experiencia de instalación de Azure Percept DK](./quickstart-percept-dk-set-up.md)

## <a name="download-software-tools-and-update-files"></a>Descargar herramientas de software y archivos de actualización

1. [Herramienta NXP UUU](https://github.com/NXPmicro/mfgtools/releases). Descargue la **versión más reciente** del archivo uuu.exe (para Windows) o del archivo uuu (para Linux) en la pestaña **Activos**.

1. [7-Zip](https://www.7-zip.org/). Este software se utilizará para extraer el archivo de imagen sin procesar de su archivo comprimido XZ. Descargue e instale el archivo .exe adecuado.

1. [Descargue los archivos de actualización](https://go.microsoft.com/fwlink/?linkid=2155734). Todos están incluidos en un archivo ZIP que se extraerá en la sección siguiente.

1. Asegúrese de que los tres artefactos de compilación están presentes:
    - Azure-Percept-DK- *&lt;número de versión&gt;* .raw.xz
    - fast-hab-fw.raw
    - emmc_full.txt

## <a name="set-up-your-environment"></a>Configurar el entorno

1. Cree una carpeta o un directorio en el equipo host en una ubicación a la que pueda acceder fácilmente a través de la línea de comandos.

1. Copie la herramienta UUU (**uuu.exe** o **uuu**) en la nueva carpeta.

1. Extraiga los archivos de actualización descargados anteriormente en la nueva carpeta que contiene la herramienta UUU.

## <a name="update-your-device"></a>Actualizar su dispositivo

1. [SSH en el kit del dispositivo](./how-to-ssh-into-percept-dk.md).

1. A continuación, abra un símbolo del sistema de Windows (**Start** > **cmd**) o un terminal de Linux y vaya a la carpeta donde se almacenan los archivos de actualización y la herramienta UUU. Escriba el siguiente comando en el símbolo del sistema o terminal para preparar el equipo para recibir un dispositivo Flash:

    - Windows:

        ```console
        uuu -b emmc_full.txt fast-hab-fw.raw Azure-Percept-DK-<version number>.raw 
        ```

    - Linux:

        ```bash
        sudo ./uuu -b emmc_full.txt fast-hab-fw.raw Azure-Percept-DK-<version number>.raw
        ```

1. Desconecte el dispositivo Azure Percept Vision del puerto USB-C de la placa de transporte.

1. Conecte el cable USB-C suministrado al puerto USB-C de la placa de transporte y al puerto USB-C del equipo host. Si el equipo sólo tiene un puerto USB-A, conecte un cable USB-C a USB (se vende por separado) a la placa del transportista y al equipo host.

1. En el símbolo del sistema del cliente SSH, escriba los siguientes comandos:

    1. Establezca el dispositivo en modo de actualización USB:

        ```bash
        sudo flagutil    -wBfRequestUsbFlash    -v1
        ```

    1. Reinicie el dispositivo. Se iniciará la instalación de la actualización.

        ```bash
        sudo reboot -f
        ```

1. Vuelva al otro símbolo del sistema o terminal. Una vez finalizada la actualización, verá un mensaje con ```Success 1    Failure 0```:

    > [!NOTE]
    > Después de la actualización, el dispositivo se restablecerá a la configuración de fábrica y perderá la conexión Wi-Fi y el inicio de sesión de SSH.

1. Una vez finalizada la actualización, desconecte la placa base. Desconecte el cable USB del equipo.  

## <a name="next-steps"></a>Pasos siguientes

Trabaje con la [experiencia de instalación de Azure Percept DK](./quickstart-percept-dk-set-up.md) para volver a configurar el dispositivo.
