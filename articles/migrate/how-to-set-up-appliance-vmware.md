---
title: Configuración de un dispositivo Azure Migrate para la evaluación de servidores en un entorno de VMware
description: Aprenda a configurar un dispositivo de Azure Migrate para evaluar y migrar servidores en el entorno de VMware.
author: vikram1988
ms.author: vibansa
ms.manager: abhemraj
ms.topic: how-to
ms.date: 04/16/2020
ms.openlocfilehash: 5b0a5d2117ea17ec003eb20084a0742e81d12ecb
ms.sourcegitcommit: 2cb7772f60599e065fff13fdecd795cce6500630
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108804095"
---
# <a name="set-up-an-appliance-for-servers-in-a-vmware-environment"></a>Configuración de un dispositivo para servidores en un entorno de VMware

En este artículo se describe cómo configurar el dispositivo de Azure Migrate para la evaluación con la herramienta [Azure Migrate: Discovery and assessment](migrate-services-overview.md#azure-migrate-discovery-and-assessment-tool).

El [dispositivo de Azure Migrate](migrate-appliance.md) es un dispositivo ligero que usa la herramienta Azure Migrate: Discovery and assessment para detectar los servidores que se ejecutan en vCenter Server y enviar los metadatos de configuración y rendimiento del servidor a Azure.

## <a name="set-up-the-appliance"></a>Configuración del dispositivo

Puede implementar el dispositivo Azure Migration mediante un par de métodos:

- Cree un servidor en una máquina virtual de vCenter Server mediante una plantilla OVA descargada. Este es el método que se describe en este artículo.
- Configure el dispositivo en un servidor existente mediante un script de instalador de PowerShell. Debe ejecutar [un script de PowerShell](deploy-appliance-script.md) si no puede usar una plantilla OVA o si está en Azure Government.

Una vez creado el dispositivo, compruebe que se pueda conectar a Azure Migrate: Discovery and assessment, regístrelo en el proyecto y configúrelo para iniciar la detección.

### <a name="deploy-by-using-an-ova-template"></a>Implementación mediante una plantilla OVA

Va a completar estos pasos, que se describen con más detalle en esta sección, para configurar el dispositivo mediante una plantilla OVA:

1. Proporcione un nombre de dispositivo y genere una clave de proyecto en el portal.
1. Descargar un archivo de plantilla OVA e importarla en vCenter Server. Comprobar que la plantilla OVA sea segura.
1. Crear el dispositivo a partir del archivo OVA. Comprobar que el dispositivo pueda conectarse a Azure Migrate.
1. Configurar el dispositivo por primera vez. 
1. Registrar el dispositivo con el proyecto mediante la clave del proyecto.

#### <a name="generate-the-project-key"></a>Generación de la clave de proyecto

1. En **Migration Goals** (Objetivos de migración), seleccione **Servers** > **Azure Migrate: Discovery and assessment** > **Discover** (Servidores > Azure Migrate: Discovery and assessment > Detectar).
1. En **Discover servers** (Detectar servidores), seleccione **Are your servers virtualized?** (¿Están virtualizados sus servidores?). > **Yes, with VMware vSphere hypervisor** (¿Están virtualizados sus servidores? > Sí, con VMware vSphere Hypervisor).
1. En **1:Generate project key** (1: Generar la clave de proyecto), especifique un nombre para el dispositivo de Azure Migrate que va a configurar para la detección de servidores en el entorno de VMware. El nombre debe ser alfanumérico y no puede tener más de 14 caracteres.
1. Para empezar a crear los recursos de Azure necesarios, seleccione **Generate key** (Generar clave). No cierre el panel **Discover** (Detectar) mientras se crean los recursos.
1. Después de que se hayan creado correctamente los recursos de Azure, se genera una *clave de proyecto*.
1. Copie la clave. Usará la clave para completar el registro del dispositivo al configurar el dispositivo.

#### <a name="download-the-ova-template"></a>Descarga de la plantilla OVA

En **2: Download Azure Migrate appliance** (2: Descargar dispositivo de Azure Migrate), seleccione el archivo .OVA y, a continuación, seleccione **Download** (Descargar).

##### <a name="verify-security"></a>Comprobación de la seguridad

Antes de implementar el archivo OVA, compruebe que el archivo sea seguro:

1. En el servidor en el que ha descargado el archivo, abra una ventana del símbolo del sistema con la opción **Ejecutar como administrador**.
1. Ejecute el siguiente comando para generar el código hash del archivo OVA:
  
    ```bash
    C:\>CertUtil -HashFile <file_location> <hashing_agorithm>
    ```
   
    Ejemplo: `C:\>CertUtil -HashFile C:\Users\Administrator\Desktop\MicrosoftAzureMigration.ova SHA256`

1. Compruebe las versiones más recientes del dispositivo y los valores hash de la nube pública de Azure:
    
    **Algoritmo** | **Descargar** | **SHA256**
    --- | --- | ---
    VMware (11,9 GB) | [La versión más reciente](https://go.microsoft.com/fwlink/?linkid=2140333) | e9c9a1fe4f3ebae81008328e8f3a7933d78ff835ecd871d1b17f367621ce3c74

#### <a name="create-the-appliance-server"></a>Creación del servidor del dispositivo

Importe el archivo descargado y, a continuación, cree un servidor en el entorno de VMware:

1. En la consola de cliente de vSphere, seleccione **File** (Archivo) > **Deploy OVF Template** (Implementar plantilla de OVF).
1. En el Asistente para implementar la plantilla OVF, seleccione **Source** (Origen) y escriba la ubicación del archivo OVA.
1. En **Name** (Nombre), escriba un nombre para el servidor. En **Location** (Ubicación), seleccione el objeto de inventario en el que se hospedará el servidor.
1. En **Host/Cluster** (Host o clúster), seleccione el host o el clúster en el que se va a ejecutar el servidor.
1. En **Storage** (Almacenamiento), seleccione el destino de almacenamiento del servidor.
1. En **Disk Format** (Formato de disco), seleccione el tamaño y el tipo de disco.
1. En **Network Mapping** (Asignación de red), seleccione la red a la que se conectará el servidor. La red requiere conectividad a Internet para enviar los metadatos a Azure Migrate.
1. Revise y confirme la configuración y, a continuación, seleccione **Finish** (Finalizar).

#### <a name="verify-appliance-access-to-azure"></a>Comprobación de que el dispositivo puede acceder a Azure

Asegúrese de que el servidor del dispositivo pueda conectarse a las direcciones URL de Azure de las [nubes públicas](migrate-appliance.md#public-cloud-urls) y las [nubes del Gobierno](migrate-appliance.md#government-cloud-urls).

<a name="4-configure-the-appliance"></a>

### <a name="configure-the-appliance"></a>Configuración del dispositivo

Para configurar el dispositivo por primera vez:

> [!NOTE]
> Si configura el dispositivo con un [script de PowerShell](deploy-appliance-script.md) en lugar de una plantilla OVA descargada, puede omitir los dos primeros pasos.

1. En el cliente de vSphere, haga clic con el botón derecho en el servidor y, a continuación, seleccione **Open Console** (Abrir consola).
1. Seleccione o escriba el idioma, la zona horaria y la contraseña del dispositivo.
1. Abra un explorador en cualquier equipo que pueda conectarse al dispositivo. A continuación, abra la dirección URL del administrador de configuración del dispositivo: `https://appliance name or IP address: 44368`.

     También puede abrir el administrador de configuración desde el escritorio del servidor del dispositivo; para ello, seleccione el acceso directo del administrador de configuración.
1. Acepte los términos de licencia y lea la información de terceros.
1. En el administrador de configuración, seleccione **Set up prerequisites** (Configurar los requisitos previos) y, a continuación, complete estos pasos:
    1. **Conectividad**: el dispositivo comprueba que el servidor tenga acceso a Internet. Si el servidor usa un proxy:
        1. Seleccione **Setup proxy** (Configurar proxy) para especificar la dirección del proxy (con el formato `http://ProxyIPAddress` o `http://ProxyFQDN`, donde *FQDN* hace referencia a un *nombre de dominio completo*) y el puerto de escucha.
        1.  Escriba las credenciales si el proxy requiere autenticación.
        1. Si ha agregado detalles del proxy o ha deshabilitado el proxy o la autenticación, seleccione **Save** (Guardar) para desencadenar la conectividad y comprobar de nuevo la conectividad.

            Solo se admite un proxy HTTP.
    1. **Sincronización de hora**: para que la detección funcione correctamente, compruebe que la hora del dispositivo esté sincronizada con la hora de Internet.
    1. **Instalación de actualizaciones**: el dispositivo garantiza que se instalan las actualizaciones más recientes. Una vez finalizada la comprobación, puede seleccionar **View appliance services** (Ver servicios del dispositivo) para ver el estado y las versiones de los servicios que se ejecutan en el servidor del dispositivo.
    1. **Instalación del VDDK**: el dispositivo comprueba si VMware vSphere Virtual Disk Development Kit (VDDK) está instalado. Si VDDK no está instalado, descargue VDDK 6.7 de VMware. Extraiga el contenido del archivo ZIP descargado en la ubicación especificada del dispositivo, tal como se indica en las *instrucciones de instalación*.

        La migración de servidores de Azure Migrate usa VDDK para replicar los servidores durante la migración a Azure. 
1. Puede *volver a ejecutar los requisitos previos* en cualquier momento durante la configuración del dispositivo para comprobar si el dispositivo cumple todos los requisitos previos:

    :::image type="content" source="./media/tutorial-discover-vmware/appliance-prerequisites.png" alt-text="Captura de pantalla que muestra cómo configurar los requisitos previos en el administrador de configuración del dispositivo.":::

#### <a name="register-the-appliance-with-azure-migrate"></a>Registro del dispositivo en Azure Migrate

1. Pegue la clave de proyecto que ha copiado desde el portal. Si no tiene la clave, vaya a **Discovery and assessment** > **Discover** > **Manage existing appliances** (Discovery and assessment > Detectar > Administrar dispositivos existentes). Seleccione el nombre del dispositivo que proporcionó al generar la clave del proyecto y, a continuación, copie la clave que se muestra.
1. Necesitará un código de dispositivo para autenticarse con Azure. Seleccione **Login** (Iniciar sesión). En **Continue with Azure Login** (Continuar con el inicio de sesión de Azure), seleccione **Copy code & Login** (Copiar código e iniciar sesión) para copiar el código del dispositivo y abrir una petición de inicio de sesión de Azure en una nueva pestaña del explorador. Asegúrese de que ha deshabilitado el bloqueador de elementos emergentes en el explorador para ver la petición.

    :::image type="content" source="./media/tutorial-discover-vmware/device-code.png" alt-text="Captura de pantalla que muestra dónde copiar el código del dispositivo e iniciar sesión.":::

1. En una nueva pestaña del explorador, pegue el código del dispositivo e inicie sesión con su nombre de usuario y contraseña de Azure. No se admite el inicio de sesión con un PIN.

    En caso de que cierre accidentalmente la pestaña de inicio de sesión sin iniciar sesión, actualice la pestaña explorador del administrador de configuración del dispositivo para mostrar el código del dispositivo y el botón **Copy code & Login** (Copiar código e iniciar sesión).
1. Después de iniciar sesión correctamente, vuelva a la pestaña del explorador que muestra el administrador de configuración del dispositivo. Si la cuenta de usuario de Azure que usó para iniciar sesión tiene los permisos necesarios para los recursos de Azure que se crearon durante la generación de la clave, se inicia el registro del dispositivo.
1. Una vez que el dispositivo se ha registrado correctamente, seleccione **View details** (Ver detalles) para ver los detalles de registro.

    :::image type="content" source="./media/tutorial-discover-vmware/appliance-registration.png" alt-text="Captura de pantalla del panel de registro con Azure Migrate que muestra que el dispositivo se ha registrado correctamente.":::

## <a name="start-continuous-discovery"></a>Inicio de detección continua

Complete los pasos de configuración en el administrador de configuración del dispositivo para preparar e iniciar la detección.

### <a name="provide-vcenter-server-details"></a>Proporción de los detalles de vCenter Server

El dispositivo debe conectarse a vCenter Server para detectar los datos de configuración y rendimiento de los servidores:

1. En **Step 1: Provide vCenter Server credentials** (Paso 1: Proporcionar las credenciales de vCenter Server), seleccione **Add credentials** (Agregar credenciales) para escribir un nombre para las credenciales. Agregue el nombre de usuario y la contraseña de la cuenta de vCenter Server que usará el dispositivo para detectar los servidores que se ejecutan en vCenter Server.
    - Tal y como se explica anteriormente en este artículo, debería tener una cuenta configurada con los permisos necesarios.
    - Si quiere limitar el ámbito de la detección a objetos específicos de VMware (centros de datos, clústeres, hosts, carpetas de clústeres o hosts, o servidores individuales de vCenter Server), consulte las instrucciones para [establecer el ámbito de detección](set-discovery-scope.md) para restringir la cuenta que usa Azure Migrate.
1. En **Step 2: Provide vCenter Server details** (Paso 2: Proporcionar los detalles de vCenter Server), seleccione **Add discovery source** (Agregar origen de detección) para seleccionar el nombre de las credenciales en la lista desplegable. Seleccione la dirección IP o el FQDN de la instancia de vCenter Server. Puede dejar el puerto predeterminado (443) o especificar un puerto personalizado en el que vCenter Server escuche. Seleccione **Guardar**.
1. El dispositivo intenta validar la conexión con el servidor que ejecuta vCenter Server mediante las credenciales. Muestra el estado de validación de la dirección IP o el FQDN de vCenter Server en la tabla de credenciales.
1. Puede *volver a validar* la conectividad con vCenter Server en cualquier momento antes de iniciar la detección.

    :::image type="content" source="./media/tutorial-discover-vmware/appliance-manage-sources.png" alt-text="Captura de pantalla que muestra la administración de credenciales y orígenes de detección de vCenter Server en el administrador de configuración del dispositivo.":::

### <a name="provide-server-credentials"></a>Proporción de las credenciales del servidor

En **Step 3: Provide server credentials to perform software inventory, agentless dependency analysis and discovery of SQL Server instances and databases** (Paso 3: Proporcionar credenciales de servidor para realizar el inventario de software, el análisis de dependencias sin agente y la detección de instancias y bases de datos de SQL Server), puede proporcionar varias credenciales de servidor. Si no desea usar ninguna de estas características del dispositivo, puede omitir este paso y continuar con la detección de vCenter Server. Puede cambiar esta opción en cualquier momento.

:::image type="content" source="./media/tutorial-discover-vmware/appliance-server-credentials-mapping.png" alt-text="Captura de pantalla que muestra cómo proporcionar credenciales para el inventario de software, el análisis de dependencias y la detección de s q l server.":::

Si quiere utilizar estas características, proporcione las credenciales del servidor siguiendo los pasos que se indican a continuación. El dispositivo intenta asignar automáticamente las credenciales a los servidores para ejecutar las características de detección.

Para agregar credenciales de servidor:

1. Seleccione **Add Credentials** (Agregar credenciales).
1. En el menú desplegable, seleccione **Credentials type** (Tipo de credenciales).
    
    Puede proporcionar credenciales de autenticación de dominio, de Windows (no de dominio), de Linux (no de dominio) y de SQL Server. Obtenga información sobre cómo [proporcionar credenciales](add-server-credentials.md) y cómo las controlamos.
1. Para cada tipo de credenciales, escriba:
    * Un nombre descriptivo.
    * Un nombre de usuario.
    * Contraseña.
    Seleccione **Guardar**.

    Si elige utilizar credenciales de dominio, también debe especificar el nombre de dominio completo. El nombre de dominio completo es obligatorio para validar la autenticidad de las credenciales con la instancia de Active Directory de ese dominio.
1. Revise los [permisos necesarios](add-server-credentials.md#required-permissions) en la cuenta para detectar las aplicaciones instaladas, realizar el análisis de dependencias sin agente y detectar las instancias y las bases de datos de SQL Server.
1. Para agregar varias credenciales a la vez, seleccione **Add more** (Agregar más) para guardar las credenciales y agregar más.
    Cuando se selecciona **Save** (Guardar) o **Add more** (Agregar más), el dispositivo valida las credenciales de domino con la instancia de Active Directory del dominio para la autenticación. La validación se realiza después de cada adición para evitar bloqueos de cuentas a medida que el dispositivo recorre en iteración para asignar las credenciales a los servidores respectivos.

Para comprobar la validación de las credenciales de dominio:

En el administrador de configuración, en la tabla de credenciales, consulte el **estado de validación** de las credenciales de dominio. Solo se validan las credenciales de dominio.

Si se produce un error en la validación, puede seleccionar el estado **con errores** para ver el error de validación. Corrija el problema y, a continuación, seleccione **Revalidate credentials** (Revalidar credenciales) para volver a intentar la validación de las credenciales.

:::image type="content" source="./media/tutorial-discover-vmware/add-server-credentials-multiple.png" alt-text="Captura de pantalla que muestra cómo proporcionar y validar varias credenciales.":::

### <a name="start-discovery"></a>Iniciar detección

Para iniciar la detección de vCenter Server, seleccione **Start discovery** (Iniciar detección) en **Step 3: Provide server credentials to perform software inventory, agentless dependency analysis and discovery of SQL Server instances and databases** (Paso 3: Proporcionar credenciales de servidor para realizar el inventario de software, el análisis de dependencias sin agente y la detección de instancias y bases de datos de SQL Server). Una vez que la detección se ha iniciado correctamente, puede comprobar el estado de detección examinando la dirección IP o el FQDN de vCenter Server en la tabla de orígenes.

## <a name="how-discovery-works"></a>Funcionamiento de la detección

* El inventario de servidores detectados tarda alrededor de 15 minutos en aparecer en Azure Portal.
* Si ha proporcionado las credenciales de los servidores, una vez que se haya completado la detección de servidores que ejecutan vCenter Server, se iniciará automáticamente el inventario de software (la detección de las aplicaciones instaladas). El inventario de software se realiza una vez cada 12 horas.
* El [inventario de software](how-to-discover-applications.md) identifica las instancias de SQL Server que se ejecutan en los servidores. Con la información que recopila, el dispositivo intenta conectarse a las instancias de SQL Server mediante las credenciales de autenticación de Windows o las credenciales de autenticación de SQL Server proporcionadas en el dispositivo. A continuación, recopila datos de las bases de datos de SQL Server y sus propiedades. La detección de SQL Server se realiza una vez cada 24 horas.
* La detección de las aplicaciones instaladas puede tardar más de 15 minutos. Todo depende del número de servidores que se detecten. Para 500 servidores, el inventario detectado tarda aproximadamente una hora en aparecer en el proyecto de Azure Migrate en el portal.
* Durante el inventario de software, las credenciales de servidor agregadas se iteran en los servidores y se validan para realizar el análisis de dependencias sin agente. Una vez completada la detección de servidores, puede habilitar desde el portal el análisis de dependencias sin agente en los servidores. Solo se pueden seleccionar los servidores en los que la validación se realiza correctamente para habilitar el análisis de dependencias sin agente.
* Los datos de las instancias y bases de datos de SQL Server comienzan a aparecer en el portal en un plazo de 24 horas después de iniciar la detección.

## <a name="next-steps"></a>Pasos siguientes

Revise los tutoriales para la [valoración de VMware](./tutorial-assess-vmware-azure-vm.md) y la [migración sin agente](tutorial-migrate-vmware.md).
