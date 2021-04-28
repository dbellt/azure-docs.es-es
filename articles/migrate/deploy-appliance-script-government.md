---
title: Configuración de un dispositivo de Azure Migrate en Azure Government
description: Aprenda a configurar un dispositivo de Azure Migrate en Azure Government
author: vikram1988
ms.author: vibansa
ms.manager: abhemraj
ms.topic: how-to
ms.date: 03/13/2021
ms.openlocfilehash: b0bc2b69a4a1ec31cfa560d51920378fe1ab52b8
ms.sourcegitcommit: 3ed0f0b1b66a741399dc59df2285546c66d1df38
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/19/2021
ms.locfileid: "107714801"
---
# <a name="set-up-an-appliance-in-azure-government"></a>Configuración de una aplicación en Azure Government 

Siga este artículo para implementar un [dispositivo Azure Migrate](./migrate-appliance-architecture.md) para servidores en un entorno VMware, servidores en Hyper-V y servidores físicos en una nube de Azure Government. Ejecute un script para crear el dispositivo y compruebe que se pueda conectar a Azure. Si quiere configurar una aplicación en la nube pública, siga [este artículo](deploy-appliance-script.md).


> [!NOTE]
> La opción para implementar un dispositivo mediante una plantilla (para servidores en el entorno VMware y el entorno Hyper-V) no es compatible con Azure Government.


## <a name="prerequisites"></a>Requisitos previos

El script configura el dispositivo de Azure Migrate en un servidor físico o uno existente.

- El servidor que hará de dispositivo debe ejecutar Windows Server 2016, con 32 GB de memoria, ocho CPU virtuales, cerca de 80 GB de almacenamiento en disco y un conmutador virtual externo. Requiere una dirección IP estática o dinámica. 
- Antes de implementar el dispositivo, consulte los requisitos detallados del dispositivo para las [servidores en VMware](migrate-appliance.md#appliance---vmware), [en Hyper-V](migrate-appliance.md#appliance---hyper-v)y [servidores físicos](migrate-appliance.md#appliance---physical).
- No ejecute el script en un dispositivo de Azure Migrate existente.

## <a name="set-up-the-appliance-for-vmware"></a>Instalación de la aplicación para VMware

Para configurar el dispositivo para VMware, descargue un archivo comprimido de Azure Portal y extraiga el contenido. Ejecute el script de PowerShell para iniciar la aplicación web de la aplicación. Instale la aplicación y configúrela por primera vez. Después, registre la aplicación con el proyecto.

### <a name="download-the-script"></a>Descarga del script

1. En **Objetivos de migración** > **Windows, Linux y SQL Servers** >  (Servidores Windows, Linux y SQL) **Azure Migrate: Detección y evaluación**, haga clic en **Detectar**.
2. En **Detectar servidor** >  **¿Están virtualizados sus servidores?** , seleccione **Sí, con hipervisor VMware vSphere.**
3. Haga clic en **Descargar** para descargar el archivo comprimido.

### <a name="verify-file-security"></a>Comprobación de la seguridad del archivo

Compruebe que el archivo comprimido es seguro, antes de implementarlo.

1. En el servidor en el que descargó el archivo, abra una ventana de comandos de administrador.
2. Ejecute el siguiente comando para generar el código hash para el archivo ZIP.
    - ```C:\>CertUtil -HashFile <file_location> [Hashing Algorithm]```
    - Ejemplo: ```C:\>CertUtil -HashFile C:\Users\administrator\Desktop\AzureMigrateInstaller-VMWare-USGov.zip SHA256```

3. Compruebe la versión más reciente del dispositivo y el valor hash:

    **Algoritmo** | **Descargar** | **SHA256**
    --- | --- | ---
    VMware (85,8 MB) | [La versión más reciente](https://go.microsoft.com/fwlink/?linkid=2140337) | 2daaa2a59302bf911e8ef195f8add7d7c8352de77a9af0b860e2a627979085ca


### <a name="run-the-script"></a>Ejecute el script.

Funcionamiento del script:

- Instala agentes y una aplicación web.
- Instala roles de Windows, como el servicio de activación de Windows, IIS y PowerShell ISE.
- Descarga e instala un módulo de reescritura de IIS. [Más información](https://www.microsoft.com/download/details.aspx?id=7435).
- Actualiza una clave del registro (HKLM) con una configuración persistente para Azure Migrate.
- Crea archivos de registro y de configuración del siguiente modo:
    - **Archivos de configuración**: %ProgramData%\Microsoft Azure\Config
    - **Archivos de registro**: %ProgramData%\Microsoft Azure\Logs

Para ejecutar el script:

1. Extraiga el archivo comprimido en la carpeta del servidor que hospedará el dispositivo. No ejecute el script en un servidor con un dispositivo de Azure Migrate existente.
2. Inicie PowerShell en el servidor con privilegios de administrador (elevados).
3. Cambie el directorio de PowerShell a la carpeta que contiene el contenido extraído del archivo comprimido descargado.
4. Ejecute el script **AzureMigrateInstaller.ps1** como se indica a continuación:
    
    ``` PS C:\Users\Administrators\Desktop\AzureMigrateInstaller-VMWare-USGov>.\AzureMigrateInstaller.ps1 ```
1. Una vez que el script se haya ejecutado correctamente, inicia la aplicación web del dispositivo para que pueda configurarlo. Si detecta algún problema, consulte los registros del script en C:\Archivos de programa\Microsoft Azure\Logs\AzureMigrateScenarioInstaller_<em>Timestamp</em>.log.

### <a name="verify-access"></a>Comprobación del acceso

Asegúrese de que el dispositivo pueda conectarse a las direcciones URL de Azure para las [nubes gubernamentales](migrate-appliance.md#government-cloud-urls).


## <a name="set-up-the-appliance-for-hyper-v"></a>Configuración del dispositivo para Hyper-V

Para configurar la aplicación para Hyper-V, descargue un archivo comprimido de Azure Portal y extraiga el contenido. Ejecute el script de PowerShell para iniciar la aplicación web de la aplicación. Instale la aplicación y configúrela por primera vez. Después, registre la aplicación con el proyecto.

### <a name="download-the-script"></a>Descarga del script

1.  En **Objetivos de migración** > **Windows, Linux y SQL Servers** >  (Servidores Windows, Linux y SQL) **Azure Migrate: Detección y evaluación**, haga clic en **Detectar**.
2.  En **Detectar servidores** >  **¿Están sus servidores virtualizados?** , seleccione **Sí, con Hyper-V**.
3.  Haga clic en **Descargar** para descargar el archivo comprimido. 


### <a name="verify-file-security"></a>Comprobación de la seguridad del archivo

Compruebe que el archivo comprimido es seguro, antes de implementarlo.

1. En el servidor en el que descargó el archivo, abra una ventana de comandos de administrador.
2. Ejecute el siguiente comando para generar el código hash para el archivo ZIP.
    - ```C:\>CertUtil -HashFile <file_location> [Hashing Algorithm]```
    - Ejemplo: ```C:\>CertUtil -HashFile C:\Users\administrator\Desktop\AzureMigrateInstaller-HyperV-USGov.zip SHA256```

3. Compruebe la versión más reciente del dispositivo y el valor hash:

    **Escenario** | **Descargar** | **SHA256**
    --- | --- | ---
    Hyper-V (85,8 MB) | [La versión más reciente](https://go.microsoft.com/fwlink/?linkid=2140424) |  db5311de3d1d4a1167183a94e8347456db9c5749c7332ff2eb4b777798765e48

          

### <a name="run-the-script"></a>Ejecute el script.

Funcionamiento del script:

- Instala agentes y una aplicación web.
- Instala roles de Windows, como el servicio de activación de Windows, IIS y PowerShell ISE.
- Descarga e instala un módulo de reescritura de IIS. [Más información](https://www.microsoft.com/download/details.aspx?id=7435).
- Actualiza una clave del registro (HKLM) con una configuración persistente para Azure Migrate.
- Crea archivos de registro y de configuración del siguiente modo:
    - **Archivos de configuración**: %ProgramData%\Microsoft Azure\Config
    - **Archivos de registro**: %ProgramData%\Microsoft Azure\Logs

Para ejecutar el script:

1. Extraiga el archivo comprimido en la carpeta del servidor que hospedará el dispositivo. Asegúrese de no ejecutar el script en un servidor que ejecute un dispositivo de Azure Migrate existente.
2. Inicie PowerShell en el servidor con privilegios de administrador (elevados).
3. Cambie el directorio de PowerShell a la carpeta que contiene el contenido extraído del archivo comprimido descargado.
4. Ejecute el script **AzureMigrateInstaller.ps1** como se indica a continuación: 

    ``` PS C:\Users\Administrators\Desktop\AzureMigrateInstaller-HyperV-USGov>.\AzureMigrateInstaller.ps1 ``` 
1. Una vez que el script se haya ejecutado correctamente, inicia la aplicación web del dispositivo para que pueda configurarlo. Si detecta algún problema, consulte los registros del script en C:\Archivos de programa\Microsoft Azure\Logs\AzureMigrateScenarioInstaller_<em>Timestamp</em>.log.

### <a name="verify-access"></a>Comprobación del acceso

Asegúrese de que el dispositivo pueda conectarse a las direcciones URL de Azure para las [nubes gubernamentales](migrate-appliance.md#government-cloud-urls).


## <a name="set-up-the-appliance-for-physical-servers"></a>Configuración del dispositivo para servidores físicos

Para configurar el dispositivo para VMware, descargue un archivo comprimido de Azure Portal y extraiga el contenido. Ejecute el script de PowerShell para iniciar la aplicación web de la aplicación. Instale la aplicación y configúrela por primera vez. Después, registre la aplicación con el proyecto.

### <a name="download-the-script"></a>Descarga del script

1. En **Objetivos de migración** > **Windows, Linux y SQL Servers** >  (Servidores Windows, Linux y SQL) **Azure Migrate: Detección y evaluación**, haga clic en **Detectar**.
2. En **Detectar servidores** >  **¿Están sus servidores virtualizados?** , seleccione **No virtualizado/Otro**.
3. Haga clic en **Descargar** para descargar el archivo comprimido.

### <a name="verify-file-security"></a>Comprobación de la seguridad del archivo

Compruebe que el archivo comprimido es seguro, antes de implementarlo.

1. En los servidores en los que descargó el archivo, abra una ventana de comandos de administrador.
2. Ejecute el siguiente comando para generar el código hash para el archivo ZIP.
    - ```C:\>CertUtil -HashFile <file_location> [Hashing Algorithm]```
    - Ejemplo: ```C:\>CertUtil -HashFile C:\Users\administrator\Desktop\AzureMigrateInstaller-Server-USGov.zip SHA256```

3. Compruebe la versión más reciente del dispositivo y el valor hash:

    **Escenario** | **Descargar*** | *Valor hash**
    --- | --- | ---
    Físico (85 MB) | [La versión más reciente](https://go.microsoft.com/fwlink/?linkid=2140338) | cfed44bb52c9ab3024a628dc7a5d0df8c624f156ec1ecc3507116bae330b257f
          

### <a name="run-the-script"></a>Ejecute el script.

Funcionamiento del script:

- Instala agentes y una aplicación web.
- Instala roles de Windows, como el servicio de activación de Windows, IIS y PowerShell ISE.
- Descarga e instala un módulo de reescritura de IIS. [Más información](https://www.microsoft.com/download/details.aspx?id=7435).
- Actualiza una clave del registro (HKLM) con una configuración persistente para Azure Migrate.
- Crea archivos de registro y de configuración del siguiente modo:
    - **Archivos de configuración**: %ProgramData%\Microsoft Azure\Config
    - **Archivos de registro**: %ProgramData%\Microsoft Azure\Logs

Para ejecutar el script:

1. Extraiga el archivo comprimido en la carpeta del servidor que hospedará el dispositivo. Asegúrese de no ejecutar el script en un servidor que ejecute un dispositivo de Azure Migrate existente.
2. Inicie PowerShell en el servidor con privilegios de administrador (elevados).
3. Cambie el directorio de PowerShell a la carpeta que contiene el contenido extraído del archivo comprimido descargado.
4. Ejecute el script **AzureMigrateInstaller.ps1** como se indica a continuación:

    ``` PS C:\Users\Administrators\Desktop\AzureMigrateInstaller-Server-USGov>.\AzureMigrateInstaller.ps1 ```
1. Una vez que el script se haya ejecutado correctamente, inicia la aplicación web del dispositivo para que pueda configurarlo. Si detecta algún problema, consulte los registros del script en C:\Archivos de programa\Microsoft Azure\Logs\AzureMigrateScenarioInstaller_<em>Timestamp</em>.log.

### <a name="verify-access"></a>Comprobación del acceso

Asegúrese de que el dispositivo pueda conectarse a las direcciones URL de Azure para las [nubes gubernamentales](migrate-appliance.md#government-cloud-urls).

## <a name="next-steps"></a>Pasos siguientes

Después de implementar la aplicación, es preciso que la configure y la registre en el proyecto.

- Configure el dispositivo para [VMware](how-to-set-up-appliance-vmware.md#4-configure-the-appliance).
- Configure el dispositivo para [Hyper-V](how-to-set-up-appliance-hyper-v.md#configure-the-appliance).
- Configure el dispositivo para [servidores físicos](how-to-set-up-appliance-physical.md).
