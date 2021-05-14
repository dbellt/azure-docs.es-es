---
title: Configuración de un dispositivo de Azure Migrate con un script
description: Aprenda a configurar un dispositivo de Azure Migrate con un script
ms.topic: how-to
author: vineetvikram
ms.author: vivikram
ms.manager: abhemraj
ms.date: 03/18/2021
ms.openlocfilehash: dfa96948b7e582457a9f09eed89d5cbe3bbc762d
ms.sourcegitcommit: 1b19b8d303b3abe4d4d08bfde0fee441159771e1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2021
ms.locfileid: "109750400"
---
# <a name="set-up-an-appliance-with-a-script"></a>Configuración de un dispositivo con un script

Siga este artículo para crear una [aplicación de Azure Migrate](./migrate-appliance-architecture.md) para evaluar o migrar servidores de VMware y de Hyper-V. Ejecute un script para crear una aplicación y compruebe que se puede conectar a Azure. 

Puede implementar la aplicación para servidores de Hyper-V y VMware mediante un script o mediante una plantilla que se descarga de Azure Portal. El uso de un script es útil si no se puede crear una aplicación mediante la plantilla descargada.

- Para usar una plantilla, siga los tutoriales de [VMware](./tutorial-discover-vmware.md) o [Hyper-V](./tutorial-discover-hyper-v.md).
- Para configurar una aplicación para servidores físicos, solo puede usar un script. Siga [este artículo](how-to-set-up-appliance-physical.md).
- Para configurar una aplicación en una nube de Azure Government, siga [este artículo](deploy-appliance-script-government.md).

## <a name="prerequisites"></a>Requisitos previos

El script configura el dispositivo de Azure Migrate en un servidor existente.

- El servidor que actuará como dispositivo debe cumplir los siguientes requisitos de hardware y sistema operativo:

Escenario | Requisitos
--- | ---
VMware | Windows Server 2016, con 32 GB de memoria, ocho vCPU, alrededor de 80 GB de almacenamiento en disco
Hyper-V | Windows Server 2016, con 16 GB de memoria, ocho vCPU, alrededor de 80 GB de almacenamiento en disco

- El servidor también necesita un conmutador virtual externo. Requiere una dirección IP estática o dinámica. 
- Antes de implementar el dispositivo, consulte los requisitos detallados del dispositivo para los [servidores en VMware](migrate-appliance.md#appliance---vmware) y [en Hyper-V](migrate-appliance.md#appliance---hyper-v).
- No ejecute el script en un dispositivo de Azure Migrate existente.

## <a name="set-up-the-appliance-for-vmware"></a>Instalación de la aplicación para VMware

Para configurar el dispositivo para VMware, descargue el archivo comprimido denominado AzureMigrateInstaller-Server-Public.zip desde el portal o desde [aquí](https://go.microsoft.com/fwlink/?linkid=2140334) y extraiga el contenido. Ejecute el script de PowerShell para iniciar la aplicación web de la aplicación. Instale la aplicación y configúrela por primera vez. Después, registre la aplicación con el proyecto.

### <a name="verify-file-security"></a>Comprobación de la seguridad del archivo

Compruebe que el archivo comprimido es seguro, antes de implementarlo.

1. En el servidor en el que descargó el archivo, abra una ventana de comandos de administrador.
2. Ejecute el siguiente comando para generar el código hash para el archivo ZIP.
    - ```C:\>CertUtil -HashFile <file_location> [Hashing Algorithm]```
    - Ejemplo: ```C:\>CertUtil -HashFile C:\Users\administrator\Desktop\AzureMigrateInstaller-VMware-Public.zip SHA256```
3. Compruebe la versión y script más recientes del dispositivo para la nube pública de Azure:

    **Algoritmo** | **Descargar** | **SHA256**
    --- | --- | ---
    VMware (85,8 MB) | [La versión más reciente](https://go.microsoft.com/fwlink/?linkid=2116601) | 85b74d93dfcee43412386141808d82147916330e6669df94c7969fe1b3d0fe72

### <a name="run-the-script"></a>Ejecute el script.

Funcionamiento del script:

- Instala agentes y una aplicación web.
- Instala roles de Windows, como el servicio de activación de Windows, IIS y PowerShell ISE.
- Descarga e instala un módulo de reescritura de IIS.
- Actualiza una clave del registro (HKLM) con una configuración persistente para Azure Migrate.
- Crea archivos de registro y de configuración del siguiente modo:
    - **Archivos de configuración**: %ProgramData%\Microsoft Azure\Config
    - **Archivos de registro**: %ProgramData%\Microsoft Azure\Logs

Para ejecutar el script:

1. Extraiga el archivo comprimido en la carpeta del servidor que hospedará el dispositivo. No ejecute el script en una aplicación de Azure Migrate existente.
2. Inicie PowerShell en el servidor con privilegios de administrador (elevados).
3. Cambie el directorio de PowerShell a la carpeta que contiene el contenido extraído del archivo comprimido descargado.
4. Ejecute el script **AzureMigrateInstaller.ps1** como se indica a continuación:

    ``` PS C:\Users\administrator\Desktop\AzureMigrateInstaller-Server-Public> .\AzureMigrateInstaller.ps1 -scenario VMware ```
  
5. Una vez que el script se haya ejecutado correctamente, inicia la aplicación web del dispositivo para que pueda configurarlo. Si detecta algún problema, consulte los registros del script en C:\Archivos de programa\Microsoft Azure\Logs\AzureMigrateScenarioInstaller_<em>Timestamp</em>.log.

### <a name="verify-access"></a>Comprobación del acceso

Asegúrese de que la aplicación puede conectarse a las direcciones URL de Azure para la nube [pública](migrate-appliance.md#public-cloud-urls).

## <a name="set-up-the-appliance-for-hyper-v"></a>Configure la aplicación para Hyper-V

Para configurar el dispositivo para Hyper-V, descargue el archivo comprimido denominado AzureMigrateInstaller-Server-Public.zip desde el portal o desde [aquí](https://go.microsoft.com/fwlink/?linkid=2105112) y extraiga el contenido. Ejecute el script de PowerShell para iniciar la aplicación web de la aplicación. Instale la aplicación y configúrela por primera vez. Después, registre la aplicación con el proyecto.


### <a name="verify-file-security"></a>Comprobación de la seguridad del archivo

Compruebe que el archivo comprimido es seguro, antes de implementarlo.

1. En el servidor en el que descargó el archivo, abra una ventana de comandos de administrador.
2. Ejecute el siguiente comando para generar el código hash para el archivo ZIP.
    - ```C:\>CertUtil -HashFile <file_location> [Hashing Algorithm]```
    - Ejemplo: ```C:\>CertUtil -HashFile C:\Users\administrator\Desktop\AzureMigrateInstaller-Server-HyperV.zip SHA256```

3. Compruebe la versión y script más recientes del dispositivo para la nube pública de Azure:

    **Escenario** | **Descargar** | **SHA256**
    --- | --- | ---
    Hyper-V (85,8 MB) | [La versión más reciente](https://go.microsoft.com/fwlink/?linkid=2116657) |  9bbef62e2e22481eda4b77c7fdf05db98c3767c20f0a873114fb0dcfa6ed682a

### <a name="run-the-script"></a>Ejecute el script.

Funcionamiento del script:

- Instala agentes y una aplicación web.
- Instala roles de Windows, como el servicio de activación de Windows, IIS y PowerShell ISE.
- Descarga e instala un módulo de reescritura de IIS.
- Actualiza una clave del registro (HKLM) con una configuración persistente para Azure Migrate.
- Crea archivos de registro y de configuración del siguiente modo:
    - **Archivos de configuración**: %ProgramData%\Microsoft Azure\Config
    - **Archivos de registro**: %ProgramData%\Microsoft Azure\Logs

Para ejecutar el script:

1. Extraiga el archivo comprimido en la carpeta del servidor que hospedará el dispositivo. No ejecute el script en una aplicación de Azure Migrate existente.
2. Inicie PowerShell en el servidor con privilegios de administrador (elevados).
3. Cambie el directorio de PowerShell a la carpeta que contiene el contenido extraído del archivo comprimido descargado.
4. Ejecute el script **AzureMigrateInstaller.ps1** como se indica a continuación:

    ``` PS C:\Users\administrator\Desktop\AzureMigrateInstaller-Server-Public> .\AzureMigrateInstaller.ps1 -scenario Hyperv ```
   
5. Una vez que el script se haya ejecutado correctamente, inicia la aplicación web del dispositivo para que pueda configurarlo. Si detecta algún problema, consulte los registros del script en C:\Archivos de programa\Microsoft Azure\Logs\AzureMigrateScenarioInstaller_<em>Timestamp</em>.log.

### <a name="verify-access"></a>Comprobación del acceso

Asegúrese de que la aplicación puede conectarse a las direcciones URL de Azure para la nube [pública](migrate-appliance.md#public-cloud-urls).

## <a name="next-steps"></a>Pasos siguientes

Después de implementar la aplicación, es preciso que la configure y la registre en el proyecto.

- Configure el dispositivo para [VMware](how-to-set-up-appliance-vmware.md#4-configure-the-appliance).
- Configure el dispositivo para [Hyper-V](how-to-set-up-appliance-hyper-v.md#configure-the-appliance).
