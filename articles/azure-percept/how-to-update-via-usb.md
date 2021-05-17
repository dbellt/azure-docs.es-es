---
title: Actualización de Azure Percept DK a través de una conexión por cable USB-C
description: Aprenda a actualizar Azure Percept DK a través de una conexión por cable USB-C
author: mimcco
ms.author: mimcco
ms.service: azure-percept
ms.topic: how-to
ms.date: 03/18/2021
ms.custom: template-how-to
ms.openlocfilehash: 39889455cd41883f5782f9fb140b400d6ed09ed7
ms.sourcegitcommit: 32ee8da1440a2d81c49ff25c5922f786e85109b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2021
ms.locfileid: "109786742"
---
# <a name="how-to-update-azure-percept-dk-over-a-usb-c-cable-connection"></a>Actualización de Azure Percept DK a través de una conexión por cable USB-C

En esta guía se muestra cómo actualizar correctamente el firmware y el sistema operativo del kit de desarrollo a través de una conexión USB. Este es un resumen de lo que va a hacer durante este procedimiento.
1. Descarga del paquete de actualización en un equipo host
1. Ejecución del comando que transfiere el paquete de actualización al kit de desarrollo
1. Establecimiento del kit de desarrollo en "modo USB" (mediante SSH) para que el equipo host pueda detectarlo y recibir el paquete de actualización
1. Conexión del kit de desarrollo al equipo host mediante un cable USB-C
1. Espera a que se complete la purga

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

1. [Herramienta NXP UUU](https://github.com/NXPmicro/mfgtools/releases). Descargue la **versión más reciente** del archivo uuu.exe versión (para Windows) o del archivo uuu (para Linux) en la pestaña **Recursos**. UUU es una herramienta creada por NXP que se usa para actualizar los paneles de desarrollo NXP.

1. [Descargue los archivos de actualización](https://go.microsoft.com/fwlink/?linkid=2155734). Todos están incluidos en un archivo ZIP que se extraerá en la sección siguiente.

1. Asegúrese de que los tres artefactos de compilación están presentes:
    - Azure-Percept-DK- *&lt;número de versión&gt;* .raw
    - fast-hab-fw.raw
    - emmc_full.txt

## <a name="set-up-your-environment"></a>Configurar el entorno

1. Cree una carpeta o un directorio en el equipo host en una ubicación a la que pueda acceder fácilmente a través de la línea de comandos.

1. Copie la herramienta UUU (**uuu.exe** o **uuu**) en la nueva carpeta.

1. Extraiga los archivos de actualización descargados anteriormente en la nueva carpeta que contiene la herramienta UUU.

## <a name="update-your-device"></a>Actualizar su dispositivo

Este procedimiento usa el único puerto USB-C del kit de desarrollo para la actualización.  Si el equipo tiene un puerto USB-C, puede desconectar el dispositivo Azure Percept Vision y usar ese cable.  Si el equipo solo tiene un puerto USB-A, desconecte el dispositivo Azure Percept Vision del puerto USB-C del kit de desarrollo y conecte un cable USB-C a USB-A (se vende por separado) al kit de desarrollo y al equipo host.

1. Abra un símbolo del sistema de Windows (Inicio > cmd) o un terminal de Linux y **vaya a la carpeta donde se almacenan los archivos de actualización y la herramienta UUU**. 

1. Escriba el comando siguiente en el símbolo del sistema o terminal.

    - Windows:

        ```console
        uuu -b emmc_full.txt fast-hab-fw.raw Azure-Percept-DK-<version number>.raw 
        ```

    - Linux:

        ```bash
        sudo ./uuu -b emmc_full.txt fast-hab-fw.raw Azure-Percept-DK-<version number>.raw
        ```

1. La ventana del símbolo del sistema mostrará un mensaje que indica "**Waiting for Known USB Device to Appear...** " (Esperando a que aparezca un dispositivo USB conocido...). La herramienta UUU está esperando a que el equipo host detecte el kit de desarrollo. Es correcto continuar con los pasos siguientes.

1. Conecte el cable USB-C suministrado al puerto USB-C del kit de desarrollo y al puerto USB-C del equipo host. Si el equipo solo tiene un puerto USB-A, conecte un cable USB-C a USB-A (se vende por separado) al kit de desarrollo y al equipo host.

1. Conéctese al kit de desarrollo a través de SSH. Si necesita ayuda para SSH, [siga estas instrucciones](./how-to-ssh-into-percept-dk.md).

1. En el terminal SSH, escriba los siguientes comandos:

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

1. Una vez finalizada la actualización, desconecte el kit de desarrollo. Desconecte el cable USB del equipo.  

## <a name="next-steps"></a>Pasos siguientes

Trabaje con la [experiencia de instalación de Azure Percept DK](./quickstart-percept-dk-set-up.md) para volver a configurar el dispositivo.
