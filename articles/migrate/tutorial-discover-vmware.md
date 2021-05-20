---
title: 'Detección de los servidores que se ejecutan en el entorno de VMware con Azure Migrate: Discovery and assessment'
description: 'Aprenda a detectar los servidores locales, las aplicaciones y las dependencias de un entorno de VMware con la herramienta Azure Migrate: Discovery and assessment.'
author: vineetvikram
ms.author: vivikram
ms.manager: abhemraj
ms.topic: tutorial
ms.date: 03/25/2021
ms.custom: mvc
ms.openlocfilehash: 42140e61146d8682d193f89b2a691b8a13260533
ms.sourcegitcommit: 2cb7772f60599e065fff13fdecd795cce6500630
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108803704"
---
# <a name="tutorial-discover-servers-running-in-a-vmware-environment-with-azure-migrate-discovery-and-assessment"></a>Tutorial: Detección de los servidores que se ejecutan en el entorno de VMware con Azure Migrate: Discovery and assessment

Como parte del recorrido de la migración a Azure, puede detectar el inventario y las cargas de trabajo locales.

En este tutorial aprenderá a detectar los servidores que se ejecutan en el entorno de VMware con la herramienta Azure Migrate: Discovery and assessment, un dispositivo de Azure Migrate ligero. El dispositivo se implementará como un servidor que se ejecuta en la instancia de vCenter Server para detectar ininterrumpidamente los servidores y sus metadatos de rendimiento, las aplicaciones que se ejecutan en dichos servidores, las dependencias de los servidores, y las instancias y bases de datos de SQL Server.

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Configurar una cuenta de Azure.
> * Preparar el entorno de VMware para la detección.
> * Crear un proyecto.
> * Configurar el dispositivo de Azure Migrate.
> * Iniciar la detección continua.

> [!NOTE]
> Los tutoriales muestran la manera más rápida de probar un escenario. Utilizan las opciones predeterminadas siempre que sea posible.  

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/pricing/free-trial/) antes de empezar.

## <a name="prerequisites"></a>Requisitos previos

Antes de empezar este tutorial, compruebe que dispone de estos requisitos previos:

Requisito | Detalles
--- | ---
**Host vCenter Server/ESXi** | Necesita una servidor ejecute las versiones 6.7, 6.5, 6.0 o 5.5 de vCenter Server.<br /><br /> Los servidores deben hospedarse en un host ESXi que ejecute la versión 5.5 u otra posterior.<br /><br /> En la instancia de vCenter Server, permita las conexiones entrantes en el puerto TCP 443 para que el dispositivo pueda recopilar los metadatos de configuración y rendimiento.<br /><br /> De forma predeterminada, el dispositivo se conecta a vCenter Server en el puerto 443. Si el servidor que ejecuta vCenter Server escucha en otro puerto, puede modificar el puerto al proporcionar los detalles de vCenter Server en el administrador de configuración del dispositivo.<br /><br /> En los hosts ESXi, asegúrese de que el acceso de entrada esté permitido en el puerto TCP 443 para realizar la detección de las aplicaciones instaladas y el análisis de dependencias sin agente en los servidores.
**Dispositivo con Azure Migrate** | vCenter Server debe disponer de estos recursos para asignárselos a un servidor que hospede el dispositivo de Azure Migrate:<br /><br /> - 32 GB de RAM, 8 vCPU y aproximadamente 80 GB de almacenamiento en disco.<br /><br /> - Un conmutador virtual externo y acceso a Internet en el servidor del dispositivo, ya sea directamente o mediante un proxy.
**Servidores** | Todas las versiones de los sistemas operativos Windows y Linux pueden detectar los metadatos de configuración y rendimiento. <br /><br /> Se admiten todas las versiones de los sistemas operativos Windows o Linux para detectar aplicaciones en los servidores. Consulte [Requisitos para el análisis de dependencias (sin agentes)](migrate-support-matrix-vmware.md#dependency-analysis-requirements-agentless).<br /><br /> Para realizar la detección de las aplicaciones instaladas y el análisis de dependencias sin agente, las herramientas de VMware (versión 10.2.1 y posteriores) deben estar instaladas y en ejecución en los servidores. Los servidores de Windows deben tener instalada la versión 2.0 de PowerShell u otra posterior.<br /><br /> Para detectar las instancias y bases de datos de SQL Server, consulte [Requisitos para la detección de instancias y bases de datos de SQL Server](migrate-support-matrix-vmware.md#sql-server-instance-and-database-discovery-requirements) y los mecanismos de autenticación de Windows.

## <a name="prepare-an-azure-user-account"></a>Preparación de una cuenta de usuario de Azure

Para crear un proyecto y registrar el dispositivo de Azure Migrate, debe tener una cuenta de Azure que tenga estos permisos:

- Permisos de Colaborador o Propietario en la suscripción de Azure.
- Permisos para registrar aplicaciones de Azure Active Directory (Azure AD).
- Permisos de Propietario o de Colaborador y Administrador de acceso de usuario en la suscripción de Azure para crear una instancia de Azure Key Vault que se usará durante la migración de servidores sin agente.

Si ha creado una cuenta de Azure gratuita, es el propietario de la suscripción de Azure de manera predeterminada. Si no es el propietario de la suscripción, colabore con él para asignar los permisos.

Para establecer permisos de Colaborador o Propietario en la suscripción de Azure:

1. En Azure Portal, busque "suscripciones". En **Servicios**, en los resultados de la búsqueda, seleccione **Suscripciones**.

    :::image type="content" source="./media/tutorial-discover-vmware/search-subscription.png" alt-text="Captura de pantalla que muestra cómo buscar una suscripción de Azure en el cuadro de búsqueda.":::

1. En **Suscripciones**, seleccione la suscripción en la que quiere crear el proyecto.
1. En el menú izquierdo, seleccione **Control de acceso (IAM)** .
1. En la pestaña **Comprobar acceso**, en **Comprobar acceso**, busque la cuenta de usuario que desea usar.
1. En el panel **Agregar una asignación de roles**, seleccione **Agregar**.

    :::image type="content" source="./media/tutorial-discover-vmware/azure-account-access.png" alt-text="Captura de pantalla que muestra cómo buscar una cuenta de usuario para comprobar el acceso y agregar una asignación de roles.":::
    
1. En **Agregar asignación de roles**, seleccione el rol Colaborador o Propietario, y seleccione la cuenta. Seleccione **Guardar**.

    :::image type="content" source="./media/tutorial-discover-vmware/assign-role.png" alt-text="Captura de pantalla que muestra la página Agregar asignación de roles para asignar un rol a la cuenta.":::

Para conceder a la cuenta los permisos necesarios para registrar aplicaciones de Azure AD:

1. En Azure Portal, vaya a **Azure Active Directory** > **Usuarios** > **Configuración de usuario**.
1. En **Configuración de usuario**, compruebe que los usuarios de Azure AD puedan registrar aplicaciones (establecido en **Sí** de forma predeterminada).

    :::image type="content" source="./media/tutorial-discover-vmware/register-apps.png" alt-text="Captura de pantalla que muestra la comprobación de la configuración de usuario para registrar aplicaciones.":::

9. Si **Registros de aplicaciones** se ha establecido en **No**, solicite al administrador global o del inquilino que asigne los permisos necesarios. Como alternativa, el administrador del inquilino o el administrador global pueden asignar el rol Desarrollador de aplicaciones a una cuenta para permitir que los usuarios registren aplicaciones de Azure AD. [Más información](../active-directory/fundamentals/active-directory-users-assign-role-azure-portal.md).

## <a name="prepare-vmware"></a>Preparación de VMware

En vCenter Server, compruebe que la cuenta tenga permisos para crear una máquina virtual mediante un archivo de instalación de máquina virtual de Open Virtualization Appliance (OVA) de VMware. Debe tener estos permisos para implementar el dispositivo de Azure Migrate como una máquina virtual de VMware mediante un archivo OVA.

Azure Migrate debe tener una cuenta de solo lectura de vCenter Server para detectar y evaluar los servidores que se ejecutan en el entorno de VMware. Si también desea ejecutar la detección de aplicaciones instaladas y el análisis de dependencias sin agente, la cuenta debe tener habilitados permisos en VMware para las operaciones de invitado de máquina virtual.

### <a name="create-an-account-to-access-vcenter-server"></a>Creación de una cuenta para acceder a vCenter Server

En el cliente web de VMware vSphere, configure una cuenta de solo lectura que se usará para vCenter Server:

1. En una cuenta que tenga privilegios de administrador, en el cliente web de vSphere, en el menú **Home** (Inicio), seleccione **Administration** (Administración).
1. En **Single Sign-On** (Inicio de sesión único), seleccione **Users and Groups** (Usuarios y grupos).
1. En **Users** (Usuarios), seleccione **New User** (Nuevo usuario).
1. Escriba los detalles de la cuenta y, a continuación, seleccione **OK** (Aceptar).
1. En el menú **Administration** (Administración), en **Access Control** (Control de acceso), seleccione **Global Permissions** (Permisos globales).
1. Seleccione la cuenta de usuario y, a continuación, seleccione **Read-only** (Solo lectura) para asignar el rol a la cuenta. Seleccione **Aceptar**.
1. Para poder iniciar la detección de las aplicaciones instaladas y el análisis de dependencias sin agente, en el menú **Access Control** (Control de acceso), seleccione **Roles**. En el panel **Roles**, en **Roles**, seleccione **Read-only** (Solo lectura). En **Privileges** (Privilegios), seleccione **Guest operations** (Operaciones de invitado). Para propagar los privilegios a todos los objetos de la instancia de vCenter Server, seleccione la casilla **Propagate to children** (Propagar a elementos secundarios).

    :::image type="content" source="./media/tutorial-discover-vmware/guest-operations.png" alt-text="Captura de pantalla que muestra el cliente web de v Sphere y cómo crear una nueva cuenta y seleccionar roles de usuario y privilegios.":::

> [!NOTE]
> Puede establecer el ámbito de la cuenta de vCenter Server para limitar la detección a los centros de datos, los clústeres, los hosts, una carpeta de clústeres o de hosts, o servidores individuales de vCenter Server. Obtenga información sobre cómo [establecer el ámbito de la cuenta de usuario de vCenter Server](set-discovery-scope.md).

### <a name="create-an-account-to-access-servers"></a>Creación de una cuenta para acceder a los servidores

La cuenta de usuario de los servidores debe tener los permisos necesarios para iniciar la detección de aplicaciones instaladas, el análisis de dependencias sin agente y la detección de instancias y bases de datos de SQL Server. Puede proporcionar la información de la cuenta de usuario en el administrador de configuración del dispositivo. El dispositivo no instala ningún agente en los servidores.

* En los servidores Windows, cree una cuenta (local o de dominio) que tenga permisos administrativos en los servidores. Para detectar instancias y bases de datos de SQL Server, la cuenta de Windows o de SQL Server debe ser miembro del rol de servidor sysadmin. Obtenga información sobre cómo [asignar el rol requerido a la cuenta de usuario](/sql/relational-databases/security/authentication-access/server-level-roles).
* En los servidores Linux, cree una cuenta con privilegios raíz. O bien, puede crear una cuenta que tenga los permisos CAP_DAC_READ_SEARCH y CAP_SYS_PTRACE en los archivos /bin/netstat y /bin/ls.

> [!NOTE]
> Puede agregar varias credenciales de servidor en el administrador de configuración del dispositivo de Azure Migrate para iniciar la detección de las aplicaciones instaladas, el análisis de dependencias sin agente y detectar instancias y bases de datos de SQL Server. Puede agregar varias credenciales de autenticación de dominio, de Windows (no de dominio), de Linux (no de dominio) o de SQL Server. Obtenga información sobre cómo [agregar credenciales de servidor](add-server-credentials.md).

## <a name="set-up-a-project"></a>Configuración de un proyecto

Para configurar un nuevo proyecto:

1. En Azure Portal, seleccione **Todos los servicios** y, a continuación, busque **Azure Migrate**.
1. En **Servicios**, seleccione **Azure Migrate**.
1. En **Información general**, seleccione una de las siguientes opciones en función de los objetivos de migración: **Windows, Linux y SQL Server**, **SQL Server (solo)** , o bien **Explorar más escenarios**. 
1. Seleccione **Create project** (Crear proyecto).
1. En **Crear proyecto**, seleccione su suscripción y grupo de recursos de Azure. Cree un grupo de recursos si no tiene ninguno.
1. En **Project Details** (Detalles del proyecto), especifique el nombre del proyecto y la región geográfica en la que quiere crearlo. Consulte [Ubicaciones geográficas admitidas (nube pública)](migrate-support-matrix.md#supported-geographies-public-cloud) y [Ubicaciones geográficas admitidas (Azure Government)](migrate-support-matrix.md#supported-geographies-azure-government).

    :::image type="content" source="./media/tutorial-discover-vmware/new-project.png" alt-text="Captura de pantalla que muestra cómo agregar los detalles del proyecto para un nuevo proyecto de Azure Migrate.":::

1. Seleccione **Crear**.
1. Espere unos minutos a que se implemente el proyecto. La herramienta **Azure Migrate: Discovery and assessment** se agrega de forma predeterminada al nuevo proyecto.

> [!NOTE]
> Si ya ha creado un proyecto, puede usarlo para registrar más dispositivos para detectar y evaluar más servidores. Obtenga información sobre cómo [administrar proyectos](create-manage-projects.md#find-a-project).

## <a name="set-up-the-appliance"></a>Configuración del dispositivo

La herramienta Azure Migrate: Discovery and assessment usa un dispositivo ligero de Azure Migrate. El dispositivo de Azure Migrate detecta los servidores y envía los metadatos de configuración y rendimiento a Azure Migrate. Configure el dispositivo mediante la implementación de una plantilla OVA que se puede descargar del proyecto.

> [!NOTE]
> Si no puede configurar el dispositivo mediante la plantilla OVA, puede hacerlo mediante la ejecución de un script de PowerShell en un servidor existente que ejecute Windows Server 2016. Obtenga información sobre cómo [utilizar PowerShell para configurar un dispositivo de Azure Migrate](deploy-appliance-script.md#set-up-the-appliance-for-vmware).

### <a name="deploy-by-using-an-ova-template"></a>Implementación mediante una plantilla OVA

Va a completar estos pasos, que se describen con más detalle en esta sección, para configurar el dispositivo mediante una plantilla OVA:

1. Proporcione un nombre de dispositivo y genere una clave de proyecto en el portal.
1. Descargar un archivo de plantilla OVA e importarla en vCenter Server. Comprobar que la plantilla OVA sea segura.
1. Crear el dispositivo a partir del archivo OVA. Comprobar que el dispositivo pueda conectarse a Azure Migrate.
1. Configurar el dispositivo por primera vez. 
1. Registrar el dispositivo con el proyecto mediante la clave del proyecto.

#### <a name="generate-the-project-key"></a>Generación de la clave de proyecto

1. En **Migration Goals** (Objetivos de migración), seleccione **Windows, Linux and SQL Servers** > **Azure Migrate: Discovery and assessment** > **Discover** (Servidores Windows, Linux y SQL > Azure Migrate: Discovery and assessment > Detectar).
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

1. Compruebe las versiones más recientes del dispositivo y los valores hash:

    - Para la nube pública de Azure:
    
        **Algoritmo** | **Descargar** | **SHA256**
        --- | --- | ---
        VMware (11,9 GB) | [La versión más reciente](https://go.microsoft.com/fwlink/?linkid=2140333) | e9c9a1fe4f3ebae81008328e8f3a7933d78ff835ecd871d1b17f367621ce3c74

    - Para Azure Government:
    
        **Algoritmo** | **Descargar** | **SHA256**
        --- | --- | ---
        VMware (85,8 MB) | [La versión más reciente](https://go.microsoft.com/fwlink/?linkid=2140337) | 2daaa2a59302bf911e8ef195f8add7d7c8352de77a9af0b860e2a627979085ca

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

1. Pegue la clave de proyecto que ha copiado desde el portal. Si no tiene la clave, vaya a **Azure Migrate: Discovery and assessment** > **Discover** > **Manage existing appliances** (Azure Migrate: Discovery and assessment > Detectar > Administrar dispositivos existentes). Seleccione el nombre del dispositivo que proporcionó al generar la clave del proyecto y, a continuación, copie la clave que se muestra.
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

> [!NOTE]
> Azure Migrate cifra la comunicación entre el dispositivo de Azure Migrate y las instancias de SQL Server de origen cuando la propiedad [TrustServerCertificate](/dotnet/api/system.data.sqlclient.sqlconnectionstringbuilder.trustservercertificate) está establecida en `true`. La capa de transporte usa SSL para cifrar el canal y omitir la cadena de certificados para validar la confianza. El servidor del dispositivo se debe configurar para [confiar en la entidad de certificación raíz del certificado](/sql/database-engine/configure-windows/enable-encrypted-connections-to-the-database-engine).
>
> Si no se ha aprovisionado ningún certificado en el servidor cuando se inicia, SQL Server genera un certificado autofirmado que se utiliza para cifrar los paquetes de inicio de sesión. [Más información](/sql/database-engine/configure-windows/enable-encrypted-connections-to-the-database-engine).
>

## <a name="how-discovery-works"></a>Funcionamiento de la detección

* El inventario de servidores detectados tarda alrededor de 15 minutos en aparecer en Azure Portal.
* Si ha proporcionado las credenciales de los servidores, una vez que se haya completado la detección de servidores que ejecutan vCenter Server, se iniciará automáticamente el inventario de software (la detección de las aplicaciones instaladas). El inventario de software se realiza una vez cada 12 horas.
* El [inventario de software](how-to-discover-applications.md) identifica las instancias de SQL Server que se ejecutan en los servidores. Con la información que recopila, el dispositivo intenta conectarse a las instancias de SQL Server mediante las credenciales de autenticación de Windows o las credenciales de autenticación de SQL Server proporcionadas en el dispositivo. A continuación, recopila datos de las bases de datos de SQL Server y sus propiedades. La detección de SQL Server se realiza una vez cada 24 horas.
* La detección de las aplicaciones instaladas puede tardar más de 15 minutos. Todo depende del número de servidores que se detecten. Para 500 servidores, el inventario detectado tarda aproximadamente una hora en aparecer en el proyecto de Azure Migrate en el portal.
* Durante el inventario de software, las credenciales de servidor agregadas se iteran en los servidores y se validan para realizar el análisis de dependencias sin agente. Una vez completada la detección de servidores, puede habilitar desde el portal el análisis de dependencias sin agente en los servidores. Solo se pueden seleccionar los servidores en los que la validación se realiza correctamente para habilitar el análisis de dependencias sin agente.
* Los datos de las instancias y bases de datos de SQL Server comienzan a aparecer en el portal en un plazo de 24 horas después de iniciar la detección.

## <a name="next-steps"></a>Pasos siguientes

- Obtenga información sobre cómo [evaluar los servidores para la migración a máquinas virtuales de Azure](./tutorial-assess-vmware-azure-vm.md).
- Obtenga información sobre cómo [evaluar los servidores que ejecutan SQL Server para la migración a Azure SQL](./tutorial-assess-sql.md).
- Revise los [datos que recopila el dispositivo de Azure Migrate](migrate-appliance.md#collected-data---vmware) durante la detección.
