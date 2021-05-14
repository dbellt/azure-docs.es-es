---
title: Configuración de un dispositivo de Azure Migrate para servidores físicos
description: Aprenda a configurar un dispositivo de Azure Migrate para la detección y evaluación del servidor físico.
author: vineetvikram
ms.author: vivikram
ms.manager: abhemraj
ms.topic: how-to
ms.date: 03/13/2021
ms.openlocfilehash: 2c185fc20c68dab549461f64d9ff8f0540a2b06a
ms.sourcegitcommit: 1b19b8d303b3abe4d4d08bfde0fee441159771e1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2021
ms.locfileid: "109753104"
---
# <a name="set-up-an-appliance-for-physical-servers"></a>Configuración de un dispositivo para servidores físicos

En este artículo se describe cómo configurar el dispositivo de Azure Migrate si está evaluando los servidores físicos con Azure Migrate: Herramienta de detección y evaluación.

El dispositivo de Azure Migrate es un dispositivo ligero que Azure Migrate: Detección y evaluación usa para hacer lo siguiente:

- Descubrir servidores locales.
- Enviar metadatos y datos de rendimiento de los servidores detectados a Azure Migrate: Detección y evaluación.

[Más información](migrate-appliance.md) sobre el dispositivo de Azure Migrate.


## <a name="appliance-deployment-steps"></a>Pasos de implementación del dispositivo

Para configurar el dispositivo:

- Proporcione un nombre de dispositivo y genere una clave de proyecto en el portal.
- Descargue un archivo comprimido con el script del instalador de Azure Migrate desde Azure Portal.
- Extraiga el contenido del archivo comprimido. Inicie la consola de PowerShell con privilegios administrativos.
- Ejecute el script de PowerShell para iniciar la aplicación web del dispositivo.
- Configure el dispositivo por primera vez y regístrelo en el proyecto mediante la clave del proyecto.

### <a name="generate-the-project-key"></a>Generación de la clave de proyecto

1. En **Objetivos de migración** > **Windows, Linux y SQL Servers** >  (Servidores Windows, Linux y SQL) **Azure Migrate: Discovery and assessment**, seleccione **Detectar**.
2. En **Detectar máquinas** >  **¿Los servidores están virtualizados?** , seleccione **Físico o de otro tipo (AWS, GCP, Xen, etc.)** .
3. En **1: Generar la clave del proyecto**, proporcione un nombre para el dispositivo de Azure Migrate que configurará para la detección de los servidores virtuales o físicos de GCP. Este nombre debe ser alfanumérico y no puede tener más de 14 caracteres.
1. Haga clic en **Generar clave** para iniciar la creación de los recursos de Azure necesarios. No cierre la página Detectar servidores durante la creación de los recursos.
1. Después de la creación correcta de los recursos de Azure, se genera una **clave de proyecto**.
1. Copie la clave, ya que la necesitará para completar el registro del dispositivo durante su configuración.

### <a name="download-the-installer-script"></a>Descarga del script del instalador

En **2: Descargar el dispositivo de Azure Migrate**, haga clic en **Descargar**.

   ![Selecciones para Detectar máquinas](./media/tutorial-assess-physical/servers-discover.png)


   ![Selecciones para Generar clave](./media/tutorial-assess-physical/generate-key-physical.png)

### <a name="verify-security"></a>Comprobación de la seguridad

Compruebe que el archivo comprimido es seguro, antes de implementarlo.

1. En los servidores en los que descargó el archivo, abra una ventana de comandos de administrador.
2. Ejecute el siguiente comando para generar el código hash para el archivo ZIP:
    - ```C:\>CertUtil -HashFile <file_location> [Hashing Algorithm]```
    - Ejemplo de uso para la nube pública: ```C:\>CertUtil -HashFile C:\Users\administrator\Desktop\AzureMigrateInstaller-Server-Public.zip SHA256 ```
    - Ejemplo de uso para la nube gubernamental: ```  C:\>CertUtil -HashFile C:\Users\administrator\Desktop\AzureMigrateInstaller-Server-USGov.zip MD5 ```
3.  Compruebe la versión más reciente del dispositivo y la [configuración](tutorial-discover-physical.md#verify-security) de los valores hash.
 

## <a name="run-the-azure-migrate-installer-script"></a>Ejecución del script del instalador de Azure Migrate
El script del instalador hace lo siguiente:

- Instala los agentes y una aplicación web para la detección y evaluación de los servidores físicos.
- Instala los roles de Windows, incluido el servicio de activación de Windows, IIS y PowerShell ISE.
- Descarga e instala un módulo de reescritura de IIS.
- Actualiza una clave del registro (HKLM) con detalles de configuración persistentes para Azure Migrate.
- Crea los siguientes archivos en la ruta de acceso:
    - **Archivos de configuración**:%Programdata%\Microsoft Azure\Config
    - **Archivos de registro**:%Programdata%\Microsoft Azure\Logs

Ejecute el script como se indica a continuación:

1. Extraiga el archivo comprimido en la carpeta del servidor que hospedará el dispositivo.  No ejecute el script en un servidor con un dispositivo de Azure Migrate existente.
2. Inicie PowerShell en el servidor anterior con privilegios administrativos (elevados).
3. Cambie el directorio de PowerShell a la carpeta en la que se ha extraído el contenido del archivo comprimido descargado.
4. Ejecute el script denominado **AzureMigrateInstaller.ps1** ejecutando el comando siguiente:

    - Para la nube pública: 
    
        ``` PS C:\Users\administrator\Desktop\AzureMigrateInstaller-Server-Public> .\AzureMigrateInstaller.ps1 ```
    - Para Azure Government: 
    
        ``` PS C:\Users\Administrators\Desktop\AzureMigrateInstaller-Server-USGov>.\AzureMigrateInstaller.ps1 ```

    El script iniciará la aplicación web del dispositivo cuando finalice correctamente.

En caso de que surja algún problema, puede acceder a los registros de script en C:\ProgramData\Microsoft Azure\Logs\AzureMigrateScenarioInstaller_<em>Timestamp</em>.log para solucionarlo.



### <a name="verify-appliance-access-to-azure"></a>Comprobación de que el dispositivo puede acceder a Azure

Asegúrese de que el dispositivo pueda conectarse a las direcciones URL de Azure para las nubes [públicas](migrate-appliance.md#public-cloud-urls) y [gubernamentales](migrate-appliance.md#government-cloud-urls).

### <a name="configure-the-appliance"></a>Configuración del dispositivo

Configure el dispositivo por primera vez.

1. Abra un explorador en cualquier máquina que pueda conectarse al dispositivo y abra la dirección URL de la aplicación web del dispositivo: **https://*nombre o dirección IP del dispositivo*: 44368**.

   Como alternativa, puede abrir la aplicación desde el escritorio, para lo que debe hacer clic en el acceso directo de la aplicación.
2. Acepte los **términos de licencia** y lea la información de terceros.
1. En la aplicación web > **Set up prerequisites** (Configurar los requisitos previos ), realice las siguientes operaciones:
    - **Connectivity** (Conectividad): la aplicación comprueba que el servidor tenga acceso a Internet. Si el servidor usa un proxy:
        - Haga clic en **Configurar proxy** y especifique la dirección del proxy (en el formato http://ProxyIPAddress o http://ProxyFQDN) ) y el puerto de escucha.
        - Especifique las credenciales si el proxy requiere autenticación.
        - Solo se admite un proxy HTTP.
        - Si ha agregado detalles del proxy o ha deshabilitado el proxy o la autenticación, haga clic en **Guardar** para desencadenar la comprobación de conectividad.
    - **Time sync** (Sincronización de hora): Se comprueba la hora. Para que la detección del servidor funcione correctamente, la hora del dispositivo debe estar sincronizada con la hora de Internet.
    - **Instalación de actualizaciones**: la herramienta Azure Migrate: Discovery and assessment comprueba que el dispositivo tenga instaladas las actualizaciones más recientes. Una vez finalizada la comprobación, puede hacer clic en **Ver servicios del dispositivo** para ver el estado y las versiones de los componentes que se ejecutan en el dispositivo.

### <a name="register-the-appliance-with-azure-migrate"></a>Registro del dispositivo en Azure Migrate

1. Pegue la **clave de proyecto** copiada desde el portal. Si no tiene la clave, vaya a **Azure Migrate: Discovery and assessment> Discover> Manage existing appliances** (Azure Migrate: Discovery and assessment > Detectar > Administrar los dispositivos existentes), seleccione el nombre del dispositivo que proporcionó al generar la clave y copie la clave correspondiente.
1. Necesitará un código de dispositivo para autenticarse con Azure. Al hacer clic en **Iniciar sesión** se abrirá un modal con el código del dispositivo, tal como se muestra a continuación.

    ![Modal que muestra el código del dispositivo](./media/tutorial-discover-vmware/device-code.png)

1. Haga clic en **Copiar código e Iniciar sesión** para copiar el código del dispositivo y abrir un símbolo del sistema de inicio de sesión de Azure en una nueva pestaña del explorador. Si no aparece, asegúrese de que ha deshabilitado el bloqueador de elementos emergentes en el explorador.
1. En la nueva pestaña, pegue el código del dispositivo e inicie sesión con su nombre de usuario y contraseña de Azure.
   
   No se admite el inicio de sesión con un PIN.
3. En caso de que cierre accidentalmente la pestaña de inicio de sesión sin iniciar sesión, deberá actualizar la pestaña explorador del administrador de configuración del dispositivo para volver a habilitar el botón Iniciar sesión.
1. Una vez que haya iniciado sesión correctamente, vuelva a la pestaña anterior con el administrador de configuración del dispositivo.
4. Si la cuenta de usuario de Azure que se usa para el registro tiene los [permisos](./tutorial-discover-physical.md) adecuados en los recursos de Azure creados durante la generación de la clave, se iniciará el registro del dispositivo.
1. Una vez que el dispositivo se ha registrado correctamente, puede ver los detalles de registro haciendo clic en **Ver detalles**.


## <a name="start-continuous-discovery"></a>Inicio de detección continua

Ahora, conecte desde el dispositivo a los servidores físicos que se van a detectar e inicie la detección.

1. En **Paso 1: proporcionar credenciales para la detección de servidores físicos o virtuales de Windows y Linux**, haga clic en **Agregar credenciales**.
1. En el caso de Windows Server, seleccione el tipo de origen **Windows Server**, especifique un nombre descriptivo para las credenciales, agregue el nombre de usuario y la contraseña. Haga clic en **Guardar**.
1. Si usa la autenticación basada en contraseña para el servidor Linux, seleccione el tipo de origen **Servidor Linux (basado en contraseña)** , especifique un nombre descriptivo para las credenciales y agregue el nombre de usuario y la contraseña. Haga clic en **Guardar**.
1. Si usa la autenticación basada en clave SSH para el servidor Linux, puede seleccionar el tipo de origen **Servidor Linux (basado en clave SSH)** , especifique un nombre descriptivo para las credenciales, agregue el nombre de usuario. busque el archivo de clave privada SSH y selecciónela. Haga clic en **Guardar**.

    - Azure Migrate admite la clave privada SSH generada por el comando ssh-keygen mediante los algoritmos RSA, DSA, ECDSA y ed25519.
    - Actualmente, Azure Migrate no admite la clave SSH basada en frase de contraseña. Use una clave SSH sin frase de contraseña.
    - Actualmente, Azure Migrate no admite el archivo de clave privada SSH generado por PuTTy.
    - Azure Migrate admite el formato OpenSSH del archivo de clave privada SSH, como se muestra a continuación:
    
    ![Formato compatible con clave privada SSH](./media/tutorial-discover-physical/key-format.png)
1. Si quiere agregar varias credenciales a la vez, haga clic en **Agregar más** para guardar y agregar más credenciales. Se admiten varias credenciales para la detección de servidores físicos.
1. En **Paso 2: Proporcionar los detalles del servidor virtual o físico**, haga clic en **Agregar origen de detección** para especificar la **dirección IP o el FQDN** del servidor y el nombre descriptivo de las credenciales para conectarse al servidor.
1. Puede **Agregar un solo elemento** cada vez o **Agregar varios elementos** de una sola vez. También hay una opción para proporcionar los detalles del servidor a través de **Importar CSV**.

    ![Selecciones para Agregar origen de detección](./media/tutorial-assess-physical/add-discovery-source-physical.png)

    - Si elige **Agregar un solo elemento**, puede elegir el tipo de sistema operativo, especificar el nombre descriptivo de las credenciales, agregar la **dirección IP o el FQDN** del servidor y hacer clic en **Guardar**.
    - Si elige **Agregar varios elementos**, puede agregar varios registros a la vez mediante la especificación de la **dirección IP o el FQDN** del servidor con el nombre descriptivo de las credenciales en el cuadro de texto. Compruebe los registros agregados y haga clic en **Guardar**.
    - Si elige **importar CSV** _(opción seleccionada de manera predeterminada)_ , puede descargar un archivo de plantilla CSV, rellenar el archivo con la **dirección IP o el FQDN** del servidor y el nombre descriptivo de las credenciales. A continuación, importe el archivo en el dispositivo, **compruebe** los registros del archivo y haga clic en **Guardar**.

1. Al hacer clic en Guardar, el dispositivo intentará validar la conexión a los servidores agregados y mostrar el **estado de validación** en la tabla en cada servidor.
    - Si se produce un error de validación para un servidor, haga clic en **Error en la validación** en la columna Estado de la tabla para revisar el error. Corrija el problema y vuelva a validar.
    - Para quitar un servidor, haga clic en **Eliminar**.
1. Puede **volver a validar** la conectividad a los servidores en cualquier momento antes de iniciar la detección.
1. Haga clic en **Iniciar detección** para dar comienzo a la detección de los servidores validados correctamente. Una vez que la detección se ha iniciado correctamente, puede comprobar el estado de detección en cada servidor de la tabla.


De esta forma comienza la detección. Los metadatos de los servidores detectados tardan alrededor de 2 minutos por servidor en aparecer en Azure Portal.

## <a name="verify-servers-in-the-portal"></a>Comprobación de los servidores en el portal

Una vez finalizada la detección, puede verificar que los servidores aparezcan en el portal.

1. Abra el panel de Azure Migrate.
2. En la página **Azure Migrate - Windows, Linux and SQL Servers** >  (Azure Migrate: servidores Windows, Linux y SQL) **Azure Migrate: Discovery and assessment**, haga clic en el icono que muestra el recuento de **servidores detectados**.


## <a name="next-steps"></a>Pasos siguientes

Pruebe la [evaluación de los servidores físicos](tutorial-assess-physical.md) con Azure Migrate: Detección y evaluación.
