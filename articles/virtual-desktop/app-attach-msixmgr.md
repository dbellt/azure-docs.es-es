---
title: 'Uso de la herramienta MSIXMGR: Azure'
description: Cómo usar la herramienta MSIXMGR para Azure Virtual Desktop.
author: Heidilohr
ms.topic: how-to
ms.date: 02/23/2021
ms.author: helohr
manager: femila
ms.openlocfilehash: 7b997f817e8e430d6f2985f73e4005da86dcda10
ms.sourcegitcommit: 8bca2d622fdce67b07746a2fb5a40c0c644100c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/09/2021
ms.locfileid: "111745536"
---
# <a name="using-the-msixmgr-tool"></a>Uso de la herramienta MSIXMGR

La herramienta MSIXMGR se utiliza para expandir aplicaciones empaquetadas con MSIX en imágenes MSIX. La herramienta toma una aplicación empaquetada con MSIX (.MSIX) y la expande en un archivo VHD, VHDx o CIM. La imagen MSIX resultante se almacena en la cuenta de Azure Storage utilizada por la implementación de Azure Virtual Desktop. En este artículo se muestra cómo usar la herramienta MSIXMGR.

>[!NOTE]
>Para garantizar la compatibilidad, asegúrese de que los archivos CIM que almacenan las imágenes MSIX se generan en la versión del sistema operativo que se ejecuta en los grupos de hosts de Azure Virtual Desktop. MSIXMGR puede crear archivos CIM, pero solo puede usar esos archivos con un grupo de hosts que ejecute Windows 10 20H2.

## <a name="requirements"></a>Requisitos

Antes de seguir las instrucciones de este artículo, deberá hacer lo siguiente:

- [Descargar la herramienta MSIXMGR](https://aka.ms/msixmgr)
- Obtener una aplicación empaquetada con MSIX (archivo .MSIX)
- Obtener permisos administrativos en la máquina donde va a crear la imagen MSIX

## <a name="create-an-msix-image"></a>Creación de una imagen MSIX

La expansión es el proceso de tomar una aplicación empaquetada con MSIX (.MSIX) y descomprimirla en una imagen MSIX (.VHD(x) o archivo CIM).

Para expandir un archivo MSIX:

1. [Descargue la herramienta MSIXMGR](https://aka.ms/msixmgr) si todavía no lo ha hecho.

2. Descomprima el archivo MSIXMGR.zip en una carpeta local.

3. Abra un símbolo del sistema con privilegios elevados.

4. Busque la carpeta local del paso 2.

5. Ejecute el siguiente comando en el símbolo del sistema para crear una imagen MSIX.

    ```cmd
    msixmgr.exe -Unpack -packagePath <path to package> -destination <output folder> [-applyacls] [-create] [-vhdSize <size in MB>] [-filetype <CIM | VHD | VHDX>] [-rootDirectory <rootDirectory>]
    ```

    Recuerde reemplazar los valores de marcador de posición por los valores correspondientes. Por ejemplo:

    ```cmd
    msixmgr.exe -Unpack -packagePath "C:\Users\%username%\Desktop\packageName_3.51.1.0_x64__81q6ced8g4aa0.msix" -destination "c:\temp\packageName.vhdx" -applyacls -create -vhdSize 200 -filetype "vhdx" -rootDirectory apps
    ```

6. Ahora que ha creado la imagen, vaya a la carpeta de destino y asegúrese de que ha creado correctamente la imagen MSIX (.VHDX).

## <a name="create-an-msix-image-in-a-cim-file"></a>Creación de una imagen MSIX en un archivo CIM

También puede usar el comando del [paso 5](#create-an-msix-image) para crear archivos CIM y VHDX; para ello, reemplace el tipo de archivo y la ruta de acceso de destino.

Por ejemplo, aquí se muestra cómo se usaría ese comando para crear un archivo CIM:

```cmd
msixmgr.exe -Unpack -packagePath "C:\Users\ssa\Desktop\packageName_3.51.1.0_x64__81q6ced8g4aa0.msix" -destination "c:\temp\packageName.cim" -applyacls -create -vhdSize 200 -filetype "cim" -rootDirectory apps
```

Aquí se muestra cómo se usaría este comando para crear un archivo VHDX:

```cmd
msixmgr.exe -Unpack -packagePath "C:\Users\ssa\Desktop\packageName_3.51.1.0_x64__81q6ced8g4aa0.msix" -destination "c:\temp\packageName.vhdx" -applyacls -create -vhdSize 200 -filetype "vhdx" -rootDirectory apps
```

## <a name="next-steps"></a>Pasos siguientes

Más información sobre la asociación de aplicaciones MSIX en [¿Qué es la asociación de aplicaciones en formato .MSIX?](what-is-app-attach.md)

Para obtener información sobre cómo configurar la asociación de la aplicación, consulte estos artículos:

- [Configuración de la asociación de aplicaciones en formato .MSIX con Azure Portal](app-attach-azure-portal.md)
- [Configuración de la asociación de aplicaciones en formato MSIX (versión preliminar) con PowerShell](app-attach-powershell.md)
- [Creación de scripts de PowerShell para la asociación de aplicaciones en formato MSIX (versión preliminar)](app-attach.md)
- [Preparación de una imagen de MSIX para Azure Virtual Desktop](app-attach-image-prep.md)
- [Configuración de un recurso compartido de archivos para asociar aplicaciones en formato MSIX (versión preliminar)](app-attach-file-share.md)

Si tiene alguna pregunta sobre la asociación de aplicaciones MSIX, consulte [Preguntas frecuentes sobre la asociación de aplicaciones en formato MSIX](app-attach-faq.md) y [Glosario sobre la conexión de aplicaciones en formato MSIX](app-attach-glossary.md).
