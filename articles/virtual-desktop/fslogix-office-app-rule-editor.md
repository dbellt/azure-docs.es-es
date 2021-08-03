---
title: 'Instalación de contenedores de aplicaciones de FSLogix de Microsoft Office en Azure Virtual Desktop: Azure'
description: Cómo usar el editor de reglas de aplicación para crear un contenedor de aplicaciones de FSLogix con Office en Azure Virtual Desktop.
author: Heidilohr
ms.topic: conceptual
ms.date: 02/23/2021
ms.author: helohr
manager: femila
ms.openlocfilehash: 4e145e8ff578d7e8663e45aa550906a1594f5228
ms.sourcegitcommit: 8bca2d622fdce67b07746a2fb5a40c0c644100c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/09/2021
ms.locfileid: "111757668"
---
# <a name="install-microsoft-office-using-fslogix-application-containers"></a>Instalación de Microsoft Office mediante contenedores de aplicaciones de FSLogix

Puede instalar Microsoft Office de forma rápida y eficaz mediante el uso de un contenedor de aplicaciones de FSLogix como plantilla para las otras máquinas virtuales del grupo de hosts.

Estos son los motivos por el que el uso de un contenedor de aplicación de FSLogix puede ayudar a agilizar la instalación:

- La descarga de las aplicaciones de Office en un contenedor de aplicación reduce los requisitos del tamaño de la unidad C.
- Las instantáneas y copias de seguridad de la máquina virtual utilizan menos recursos.
- Tener una canalización automatizada mediante la actualización de una sola imagen facilita la actualización de las máquinas virtuales.
- Solo se necesita una imagen para instalar Office (y otras aplicaciones) en todas las máquinas virtuales de la implementación de Azure Virtual Desktop.

En este artículo se muestra cómo configurar un contenedor de aplicaciones de FSLogix con Office.

## <a name="requirements"></a>Requisitos

Necesitará lo siguiente para configurar el editor de reglas:

- una máquina virtual que ejecute Windows sin Office instalado
- una copia de Office
- una copia de FSLogix instalada en la implementación
- un recurso compartido de red al que todas las máquinas virtuales del grupo de hosts tengan acceso de solo lectura

## <a name="install-office"></a>Instalar Office

Para instalar Office en el VHD o VHDX, habilite el protocolo de escritorio remoto en la máquina virtual y, a continuación, siga las instrucciones que se indican en [Instalación de Office en la imagen de un disco duro virtual principal](install-office-on-wvd-master-image.md). Al instalar, asegúrese de utilizar [las licencias correctas](overview.md#requirements).

>[!NOTE]
>Azure Virtual Desktop requiere la activación de equipos compartidos (SCA).

## <a name="install-fslogix"></a>Instalación de FSLogix

Para instalar FSLogix y el editor de reglas, siga las instrucciones que se indican en [Descarga e instalación de FSLogix](/fslogix/install-ht).

## <a name="create-and-prepare-a-vhd-to-store-office"></a>Creación y preparación de un disco duro virtual para almacenar Office

A continuación, deberá crear y preparar una imagen de VHD en la que usar el editor de reglas:

1. Abra un símbolo del sistema como administrador. y ejecute el siguiente comando:

    ```cmd
        taskkill /F /IM MicrosoftEdge.exe /T
    ```

    >[!NOTE]
    > Asegúrese de mantener los espacios en blanco que aparecen en este comando.

2. Luego, ejecute el siguiente comando:

    ```cmd
    sc queryex type=service state=all | find /i "ClickToRunSvc"
    ```
    
   Si encuentra el servicio, reinicie la máquina virtual antes de continuar con el paso 3.

    ```cmd
    net stop ClickToRunSvc
    ```

3. Después, vaya a **Archivos de programa** > **FSLogix** > **Aplicaciones** y ejecute el siguiente comando para crear el VHD de destino:

    ```cmd
    frx moveto-vhd -filename <path to network share>\office.vhdx -src "C:\Program Files\Microsoft Office" -size-mbs 5000 
    ```

    El disco duro virtual que se crea con este comando debe contener la carpeta C:\\Archivos de programa\\Microsoft Office.

    >[!NOTE]
    >Si ve algún error, desinstale Office y comience de nuevo desde el paso 1.

## <a name="configure-the-rule-editor"></a>Configuración del editor de reglas

Ahora que ha preparado la imagen, debe configurar el editor de reglas y crear un archivo en el que almacenar las reglas.

1. Vaya a **Archivos de programa** > **FSLogix** > **Aplicaciones** y ejecute **RuleEditor.exe**.

2. Seleccione **File** > **New** > **Create** (Archivo > Nuevo > Crear) para crear un nuevo conjunto de reglas y, a continuación, guarde el conjunto de reglas en una carpeta local.

3. Seleccione **Blank Rule Set** (Conjunto de reglas en blanco) y haga clic en **OK** (Aceptar).

4. Seleccione el botón **+**. Se abrirá la ventana **Add Rule** (Agregar regla). Esto cambiará las opciones del cuadro de diálogo **Add Rule** (Agregar regla).

5. En el menú desplegable, seleccione **App Container (VHD) Rule** (Regla de contenedor de aplicación [VHD]).

6. Escriba **C:\\Archivos de programa\\Microsoft Office** en el campo **Folder** (Carpeta).

7. En el campo **Disk file** (Archivo del disco), seleccione **\<path\>\\office.vhd** en la sección **Create target VHD** (Crear VHD de destino).

8. Seleccione **Aceptar**.

9. Vaya a la carpeta de trabajo en **C:\\Usuarios\\\<username\>\\Documentos\\FSLogix Rule Sets** y busque los archivos .frx y .fxa. Debe trasladar estos archivos a la carpeta de reglas que se encuentra en **C: \\Archivos de programa \\FSLogix\\Aplicaciones\\Rules** para que las reglas empiecen a funcionar.

10. Seleccione **Apply Rules to System** (Aplicar reglas al sistema) para que se apliquen las reglas.

     >[!NOTE]
     > Deberá aplicar los archivos de reglas de aplicación a todos los hosts de sesión.

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre FSLogix, consulte [Documentación de FSLogix](/fslogix/).