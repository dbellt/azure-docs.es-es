---
title: Implementación de la solución Azure Sentinel para entornos de SAP | Microsoft Docs
description: Aprenda a implementar la solución Azure Sentinel para entornos de SAP.
author: batamig
ms.author: bagold
ms.service: azure-sentinel
ms.topic: tutorial
ms.custom: mvc
ms.date: 05/13/2021
ms.subservice: azure-sentinel
ms.openlocfilehash: cf7a9fb700bba135663e0684d8ba25c7ebcf92f0
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110466537"
---
# <a name="tutorial-deploy-the-azure-sentinel-solution-for-sap-public-preview"></a>Tutorial: Implementación de la solución Azure Sentinel para SAP (versión preliminar pública)

Este tutorial le guía paso a paso por el proceso de implementación de la solución Azure Sentinel para SAP.

> [!IMPORTANT]
> La solución Azure Sentinel para SAP está actualmente en VERSIÓN PRELIMINAR. En la página [Términos de uso complementarios para las Versiones preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) se incluyen términos legales adicionales que se aplican a las características de Azure que se encuentran en versión beta, versión preliminar o que todavía no se han publicado para su disponibilidad general.
>

## <a name="overview"></a>Información general

Las [soluciones de Azure Sentinel](sentinel-solutions.md) incluyen contenido de seguridad agrupado como detecciones de amenazas, libros y listas de reproducción. Las soluciones le permiten incorporar contenido de seguridad de Azure Sentinel para un conector de datos específico mediante un único proceso.

El conector de datos de Azure Sentinel para SAP le permite supervisar los sistemas SAP en busca de amenazas sofisticadas en el nivel empresarial y de aplicación.

El conector de datos de SAP genera una gran cantidad de registros de 14 aplicaciones de todo el conjunto del sistema SAP y recopila registros de Advanced Business Application Programming (ABAP) mediante llamadas a NetWeaver RFC y datos de almacenamiento de archivos mediante la interfaz de control de OSSAP. El conector de datos de SAP se agrega a la capacidad de Azure Sentinel para supervisar la infraestructura subyacente de SAP.

Para ingerir registros de SAP en Azure Sentinel, debe tener instalado el conector de datos de Azure Sentinel para SAP en el entorno de SAP.
Se recomienda usar un contenedor de Docker en una máquina virtual de Azure para la implementación, como se indica en este tutorial.

Una vez implementado el conector de datos de SAP, implemente el contenido de seguridad de la solución de SAP para obtener información fácilmente sobre el entorno de SAP de su organización y mejorar la funcionalidad de todas las operaciones de seguridad relacionadas.

En este tutorial, aprenderá:

> [!div class="checklist"]
> * Preparar el sistema SAP para la implementación del conector de datos correspondiente
> * Usar un contenedor de Docker y una máquina virtual de Azure para implementar el conector de datos de SAP
> * Implementar el contenido de seguridad de la solución SAP en Azure Sentinel

## <a name="prerequisites"></a>Requisitos previos

Para implementar el contenido de seguridad de Azure Sentinel y el conector de datos de SAP, tal y como se describe en este tutorial, debe cumplir los siguientes requisitos previos:

|Área  |Descripción  |
|---------|---------|
|**Requisitos previos de Azure**     |  **Acceso a Azure Sentinel**. Anote el identificador y la clave del área de trabajo de Azure Sentinel que se usará en este tutorial cuando se [implemente el conector de datos de SAP](#deploy-your-sap-data-connector). <br>Para ver estos detalles en Azure Sentinel, vaya a **Configuración** > **Configuración del área de trabajo** > **Agents management** (Administración de agentes). <br><br>**Capacidad para crear recursos de Azure**. Para más información, consulte la [documentación de Azure Resource Manager](/azure/azure-resource-manager/management/manage-resources-portal). <br><br>**Acceso a Azure Key Vault**. En este tutorial se describen los pasos recomendados para usar Azure Key Vault para almacenar las credenciales. Para más información, consulte la [documentación de Azure Key Vault](/azure/key-vault/).       |
|**Requisitos previos del sistema**     |   **Software**. El script de implementación del conector de datos de SAP instala automáticamente los requisitos previos de software. Para más información, consulte [Software instalado automáticamente](#automatically-installed-software). <br><br> **Conectividad del sistema**. Asegúrese de que la máquina virtual que actúa como host del conector de datos de SAP tiene acceso a: <br>- Azure Sentinel <br>- Azure Key Vault <br>- El host del entorno de SAP, a través de los siguientes puertos TCP: *32xx*, *5xx13* y *33xx*, donde *xx* es el número de instancia de SAP. <br><br>Asegúrese de que también tiene una cuenta de usuario de SAP para acceder a la página de descarga del software de SAP.<br><br>**Arquitectura del sistema**. La solución SAP se implementa en una máquina virtual como un contenedor de Docker y cada cliente de SAP requiere su propia instancia de contenedor. <br>La máquina virtual y el área de trabajo de Azure Sentinel pueden estar en diferentes suscripciones de Azure e incluso en inquilinos de Azure AD diferentes.|
|**Requisitos previos de SAP**     |   **Versiones de SAP admitidas**. Se recomienda usar [SAP_BASIS, versión 750 SP13](https://support.sap.com/en/my-support/software-downloads/support-package-stacks/product-versions.html#:~:text=SAP%20NetWeaver%20%20%20%20SAP%20Product%20Version,%20%20SAPKB710%3Cxx%3E%20%207%20more%20rows) o versiones superiores. <br>Seleccione los pasos de este tutorial para proporcionar instrucciones alternativas si trabaja con la versión [SAP_BASIS 740](https://support.sap.com/en/my-support/software-downloads/support-package-stacks/product-versions.html#:~:text=SAP%20NetWeaver%20%20%20%20SAP%20Product%20Version,%20%20SAPKB710%3Cxx%3E%20%207%20more%20rows).<br><br> **Detalles del sistema SAP**. Tome nota de los siguientes detalles del sistema SAP para usarlos en este tutorial:<br>    - Dirección IP del sistema SAP<br>- Número del sistema SAP como, por ejemplo, `00`<br>    - Identificador del sistema SAP, del sistema SAP NetWeaver. Por ejemplo, `NPL`. <br>- Identificador de cliente de SAP como, por ejemplo, `001`.<br><br>**Acceso a la instancia de SAP NetWeaver**. El acceso a las instancias de SAP debe usar una de las siguientes opciones: <br>- [Usuario y contraseña de SAP ABAP](#configure-your-sap-system). <br>- Un usuario con un certificado X509, mediante SAP CRYPTOLIB PSE. Esta opción puede requerir pasos manuales expertos.<br><br>**Soporte técnico del equipo de SAP**.  Necesitará el soporte técnico del equipo de SAP para asegurarse de que el sistema SAP esté [configurado correctamente](#configure-your-sap-system) para la implementación de la solución.   |
|     |         |


### <a name="automatically-installed-software"></a>Software instalado automáticamente

El [script de implementación del conector de datos de SAP](#deploy-your-sap-data-connector) instala el siguiente software en la máquina virtual con privilegios SUDO (raíz):

- [Unzip.](https://www.microsoft.com/en-us/p/unzip/9mt44rnlpxxt?activetab=pivot:overviewtab)
- [NetCat](https://sectools.org/tool/netcat/)
- [Python 3.6 o versiones superiores](https://www.python.org/downloads/)
- [Python3-pip](https://pypi.org/project/pip/)
- [Docker](https://www.docker.com/)

## <a name="configure-your-sap-system"></a>Configuración del sistema SAP

En este procedimiento se describe cómo asegurarse de que el sistema SAP tenga instalados los requisitos previos correctos y esté configurado para la implementación del conector de datos de Azure Sentinel para SAP.

> [!IMPORTANT]
> Realice este procedimiento junto con el equipo de SAP para garantizar las configuraciones correctas.
>

**Para configurar el sistema SAP para el conector de datos:**

1. Si usa una versión de SAP anterior a la 750, asegúrese de que las siguientes notas de SAP estén implementadas en el sistema:

    - **SPS12641084**. Para sistemas que ejecutan versiones de SAP anteriores a SAP BASIS 750 SPS13
    - **2502336**. Para sistemas que ejecutan versiones de SAP anteriores a SAP BASIS 750 SPS1
    - **2173545**. Para sistemas que ejecutan versiones de SAP anteriores a SAP BASIS 750

    Acceda a estas notas de SAP en el [sitio de SAP ONE Support Launchpad](https://support.sap.com/en/index.html) mediante una cuenta de usuario de SAP.

1. Descargue e instale una de las siguientes solicitudes de cambio de SAP desde el repositorio de GitHub de Azure Sentinel, en https://github.com/Azure/Azure-Sentinel/tree/master/Solutions/SAP/CR:

    - **Versión SAP 750 o versiones posteriores**: instale la solicitud de cambio de SAP *131 (NPLK900131)* .
    - **Versión SAP 740**: instale la solicitud de cambio de SAP *132 (NPLK900132)* .

    Cuando realice este paso, use el código de transacción de SAP **STMS_IMPORT**.

    > [!NOTE]
    > En el área **Import Options** (Opciones de importación) de SAP, puede ver la opción **Ignore Invalid Component Version** (Ignorar versión del componente no válida). Si aparece, seleccione esta opción antes de continuar.
    >

1. Para importar la solicitud de cambio *14 (NPLK900114)* de SAP, cree un rol de SAP llamado **/MSFTSEN/SENTINEL_CONNECTOR**. Use el código de transacción de SAP **STMS_IMPORT**.

    Compruebe que el rol se crea con los permisos necesarios, como:

    :::image type="content" source="media/sap/required-sap-role-authorizations.png" alt-text="Permisos de rol necesarios para el conector de datos de Azure Sentinel para SAP.":::

    Para más información, consulte [Autorizaciones para el usuario de ABAP](sap-solution-detailed-requirements.md#required-abap-authorizations).

1. Cree un usuario de RFC/NetWeaver sin cuadro de diálogo para el conector de datos de SAP y asóciele el rol **/MSFTSEN/SENTINEL_CONNECTOR** recién creado.

    - Después de asociar el rol, compruebe que los permisos de rol se distribuyen al usuario.
    - Este proceso requiere que use un nombre de usuario y una contraseña para el usuario de ABAP. Después de crear el nuevo usuario y de que este tenga los permisos necesarios, asegúrese de cambiar la contraseña del usuario de ABAP.

1. Descargue y coloque el **SDK de SAP NetWeaver RFC 7.50 para la versión Linux ON x86_64 de 64 BITS** en la máquina virtual, ya que se necesitará durante el proceso de instalación.

    Por ejemplo, busque el SDK en el sitio de [descarga de software de SAP](https://launchpad.support.sap.com/#/softwarecenter/template/products/_APP=00200682500000001943&_EVENT=DISPHIER&HEADER=Y&FUNCTIONBAR=N&EVENT=TREE&NE=NAVIGATE&ENR=01200314690100002214&V=MAINT) > **SAP NW RFC SDK** > **SAP NW RFC SDK 7.50** > **nwrfc750X_X-xxxxxxx.zip**. Asegúrese de descargar la opción **LINUX ON X86_64 de 64 BITS**. Copie el archivo, por ejemplo, mediante SCP, en la máquina virtual.

    Necesitará una cuenta de usuario de SAP para acceder a la página de descarga de software de SAP.

1. (Opcional) El archivo **Auditlog** de SAP se usa en todo el sistema y admite varios clientes SAP. Sin embargo, cada instancia de la solución Azure Sentinel para SAP solo admite un único cliente SAP.

    Por lo tanto, si tiene un sistema SAP de varios clientes, se recomienda que habilite el archivo **Auditlog** solo para el cliente donde implemente la solución SAP para evitar la duplicación de datos.


## <a name="deploy-a-linux-vm-for-your-sap-data-connector"></a>Implementación de una máquina virtual Linux para el conector de datos de SAP

En este procedimiento se describe cómo usar la CLI de Azure para implementar una máquina virtual Ubuntu Server 18.04 LTS y asignarla con una [identidad administrada por el sistema](/azure/active-directory/managed-identities-azure-resources/).

> [!TIP]
> También puede implementar el conector de datos en RHEL, en la versión 7.7 y posteriores, o en la versión 15 y posteriores de SUSE. Tenga en cuenta que el sistema operativo y los niveles de revisión deben estar completamente actualizados.
>

**Para implementar y preparar la máquina virtual Ubuntu**:

1. Use el siguiente comando como ejemplo, insertando los valores del grupo de recursos y el nombre de la máquina virtual:

    ```azurecli
    az vm create  --resource-group [resource group name]   --name [VM Name] --image UbuntuLTS  --admin-username AzureUser --data-disk-sizes-gb 10 – --size Standard_DS2_– --generate-ssh-keys  --assign-identity
    ```

1. En la nueva máquina virtual, instale:

    - [Venv](https://docs.python.org/3.8/library/venv.html), con Python versión 3.8 o superior.
    - La [CLI de Azure versión 2.8.0 o superior](/cli/azure/).

> [!IMPORTANT]
> No olvide aplicar los procedimientos recomendados de seguridad para su organización, al igual que haría con cualquier otra máquina virtual.
>

Para más información, consulte [Inicio rápido: Creación rápida de una máquina virtual con la CLI de Azure](/azure/virtual-machines/linux/quick-create-cli).

## <a name="create-key-vault-for-your-sap-credentials"></a>Creación de un almacén de claves para las credenciales de SAP

En este tutorial se usa una instancia de [Azure Key Vault](/azure/key-vault/) recién creada o dedicada para almacenar las credenciales del conector de datos de SAP.

**Para crear o dedicar una instancia de Azure Key Vault**:

1. Cree una nueva instancia de Azure Key Vault o elija una existente para dedicarla a la implementación del conector de datos de SAP.

    Por ejemplo, para crear una nueva instancia de Key Vault, ejecute los siguientes comandos. Para ello, use el nombre del grupo de recursos de Key Vault y especifique el nombre de usuario:

    ```azurecli
    kvgp=<KVResourceGroup>

    kvname=<keyvaultname>

    #Create Key Vault
    az keyvault create \
      --name $kvname \
      --resource-group $kvgp
    ```

1. Asigne una directiva de acceso que incluya los permisos GET, LIST y SET a la identidad administrada de la máquina virtual.

    En Azure Key Vault, seleccione **Directivas de acceso** > **Agregar directiva de acceso - Permisos de los secretos: Get, List y Set** > **Seleccionar la entidad de seguridad**. Escriba el [nombre de la máquina virtual](#deploy-a-linux-vm-for-your-sap-data-connector) y, a continuación, seleccione **Agregar** > **Guardar**.

    Para más información, consulte la [documentación de Key Vault](/azure/key-vault/general/assign-access-policy-portal).

1. Ejecute el siguiente comando para obtener el [identificador de entidad de seguridad de la máquina virtual](#deploy-a-linux-vm-for-your-sap-data-connector) y escriba el nombre del grupo de recursos de Azure:

    ```azurecli
    az vm show -g [resource group] -n [Virtual Machine] --query identity.principal– --out tsv
    ```

    El identificador de la entidad de seguridad se muestra para que lo use en el paso siguiente.

1. Ejecute el siguiente comando para asignar los permisos de acceso de la máquina virtual a la instancia de Key Vault y escriba el nombre del grupo de recursos y el valor del identificador de la entidad de seguridad devuelto en el paso anterior.

    ```azurecli
    az keyvault set-policy  --name $kv  --resource-group [resource group]  --object-id [Principal ID]  --secret-permissions get set
    ```

## <a name="deploy-your-sap-data-connector"></a>Implementación del conector de datos de SAP

El script de implementación del conector de datos de Azure Sentinel para SAP instala el [software necesario](#automatically-installed-software) y, después, instala el conector en la [máquina virtual recién creada](#deploy-a-linux-vm-for-your-sap-data-connector) y almacena las credenciales en el [almacén de claves dedicado](#create-key-vault-for-your-sap-credentials).

El script de implementación del conector de datos de SAP se almacena en el directorio [repositorio de GitHub de Azure Sentinel > DataConnectors > SAP](https://raw.githubusercontent.com/Azure/Azure-Sentinel/master/DataConnectors/SAP/).

Para ejecutar el script de implementación del conector de datos de SAP, necesitará los siguientes detalles:

- Los detalles del área de trabajo de Azure Sentinel, tal y como aparecen en la sección [Requisitos previos](#prerequisites).
- Los detalles del sistema SAP que aparecen en la sección [Requisitos previos](#prerequisites).
- Acceso a un usuario de máquina virtual con privilegios de SUDO.
- El usuario de SAP que creó en [Configuración del sistema SAP](#configure-your-sap-system), con el rol **/MSFTSEN/SENTINEL_CONNECTOR** aplicado.
- La ayuda del equipo de SAP.


**Para ejecutar el script de implementación de la solución de SAP**:

1. Ejecute el siguiente comando para implementar la solución de SAP en la máquina virtual.

    ```azurecli
    wget -O sapcon-sentinel-kickstart.sh https://raw.githubusercontent.com/Azure/Azure-Sentinel/master/Solutions/SAP/sapcon-sentinel-kickstart.sh && bash ./sapcon-sentinel-kickstart.sh
    ```

1. Siga las instrucciones en pantalla para escribir los detalles de SAP y de Key Vault, y completar la implementación. Aparece un mensaje de confirmación cuando se completa la implementación:

    ```azurecli
    The process has been successfully completed, thank you!
    ```

    Azure Sentinel empieza a recuperar los registros de SAP para el intervalo de tiempo configurado, hasta 24 horas antes de la hora de inicialización.

1. Se recomienda revisar los registros del sistema para asegurarse de que el conector de datos transmite datos. Ejecute:

    ```bash
    docker logs -f sapcon-[SID]
    ```

## <a name="deploy-sap-security-content"></a>Implementación del contenido de seguridad de SAP

Implemente el [contenido de seguridad de SAP](sap-solution-security-content.md) desde las áreas **Soluciones** y **Listas de seguimiento** de Azure Sentinel.

La solución **Azure Sentinel - Continuous Threat Monitoring para SAP** permite que el conector de datos de SAP aparezca en el área **Conectores de datos** de Azure Sentinel e implementa el libro **SAP - System Applications and Products** y las reglas de análisis relacionadas con SAP.

Agregue manualmente listas de seguimiento relacionadas con SAP al área de trabajo de Azure Sentinel.

**Para implementar el contenido de seguridad de la solución de SAP**:

1. En el menú de navegación de Azure Sentinel, seleccione **Soluciones (versión preliminar)** .

    La página **Soluciones** muestra una lista filtrada en la que se pueden realizar búsquedas de soluciones.

1. Seleccione **Azure Sentinel - Continuous Threat Monitoring para SAP (versión preliminar)** para abrir la página de la solución de SAP.

    :::image type="content" source="media/sap/sap-solution.png" alt-text="Azure Sentinel: solución Continuous Threat Monitoring para SAP (versión preliminar).":::

1. Seleccione **Crear** para iniciar el asistente para la implementación de soluciones y escriba los detalles de la suscripción de Azure, el grupo de recursos y el área de trabajo de Log Analytics donde desea implementar la solución.

1. Seleccione **Siguiente** para recorrer las pestañas **Conectores de datos**, **Análisis** y **Libros**, en las que podrá obtener más información sobre los componentes que se implementarán con esta solución.

    El nombre predeterminado del libro es **SAP: Aplicaciones y productos del sistema (versión preliminar)** . Cámbielo en la pestaña Libros según sea necesario.

    Para más información, consulte [Solución Azure Sentinel para SAP: Referencia de contenido de seguridad (versión preliminar pública)](sap-solution-security-content.md).

1. En la pestaña **Revisar y crear**, espere el mensaje **Validación superada** y, a continuación, seleccione **Crear** para implementar la solución.

    > [!TIP]
    > También puede seleccionar **Descargar una plantilla** para obtener un vínculo para implementar la solución como código.

1. Una vez completada la implementación, aparece un mensaje de confirmación en la parte superior derecha de la página.

    Para mostrar el contenido recién implementado, vaya a:

    - **Administración de amenazas** > **Libros**, para encontrar el libro [SAP: Aplicaciones y productos del sistema (versión preliminar)](sap-solution-security-content.md#sap---system-applications-and-products-workbook).
    - **Configuración** > **Análisis** para ver una serie de [reglas de análisis relacionadas con SAP](sap-solution-security-content.md#built-in-analytics-rules).

1. Agregue listas de seguimiento relacionadas con SAP para usarlas en búsquedas, reglas de detección, búsqueda de amenazas y cuadernos de estrategias de respuesta. Estas listas de seguimiento proporcionan la configuración de la solución de supervisión continua de amenazas de Azure Sentinel para SAP.

    1. Descargue las listas de seguimiento de SAP desde el repositorio de GitHub de Azure Sentinel en https://github.com/Azure/Azure-Sentinel/tree/master/Solutions/SAP/Analytics/Watchlists.

    1. En el área **Listas de seguimiento** de Azure Sentinel, agregue las listas de seguimiento al área de trabajo de Azure Sentinel. Use los archivos CSV descargados como orígenes y, a continuación, personalícelos según sea necesario para su entorno. 

        [ ![Listas de seguimiento relacionadas con SAP agregadas a Azure Sentinel.](media/sap/sap-watchlists.png) ](media/sap/sap-watchlists.png#lightbox)

        Para más información, consulte [Uso de listas de seguimiento de Azure Sentinel](watchlists.md) y [Listas de seguimiento de SAP disponibles](sap-solution-security-content.md#available-watchlists).

1. En Azure Sentinel, vaya al conector de datos de **Azure Sentinel Continuous Threat Monitoring para SAP** para confirmar la conexión:

    [ ![Página del conector de datos de Azure Sentinel Continuous Threat Monitoring para SAP.](media/sap/sap-data-connector.png) ](media/sap/sap-data-connector.png#lightbox)

    Los registros de SAP ABAP aparecen en la página **Registros** en **Registros personalizados**:

    [ ![Registros de SAP ABAP en Registros personalizados en Azure Sentinel.](media/sap/sap-logs-in-sentinel.png) ](media/sap/sap-logs-in-sentinel.png#lightbox)

    Para más información, consulte la [referencia sobre registros de la solución Azure Sentinel para SAP (versión preliminar pública)](sap-solution-log-reference.md).

## <a name="sap-solution-deployment-troubleshooting"></a>Solución de problemas de implementación de la solución de SAP

Después de implementar el conector de datos de SAP y el contenido de seguridad, puede experimentar los siguientes errores o problemas:

|Incidencia  |Solución alternativa  |
|---------|---------|
|Problemas de conectividad de red con el entorno de SAP o Azure Sentinel     |  Compruebe la conectividad de red según sea necesario.       |
|Credenciales de usuario incorrectas de SAP ABAP     |Compruebe sus credenciales y corríjalas aplicando los valores correctos a los valores **ABAPUSER** y **ABAPPASS** en Azure Key Vault.         |
|Faltan permisos, como el rol **/MSFTSEN/SENTINEL_CONNECTOR** no asignado al usuario de SAP según era necesario, o el rol está inactivo.     |Corrija este error asignando el rol y asegurándose de que está activo en el sistema SAP.         |
|Falta una solicitud de cambio de SAP     | Asegúrese de que ha importado la solicitud de cambio de SAP correcta, como se describe en [Configuración del sistema SAP](#configure-your-sap-system).        |
|Se ha especificado un identificador o clave incorrectos del área de trabajo de Azure Sentinel en el script de implementación     |  Para corregir este error, especifique las credenciales correctas en Azure KeyVault.       |
|Un archivo del SDK de SAP dañado o que falta     | Para corregir este error, reinstale el SDK de SAP y asegúrese de que usa la versión correcta de Linux de 64 bits.        |
|Faltan datos en el libro o las alertas     |    Asegúrese de que la directiva **Auditlog** esté habilitada correctamente en el lado de SAP, sin errores en el archivo de registro. Use la transacción **RSAU_CONFIG_LOG** para este paso.     |
|     |         |

> [!TIP]
> Es muy recomendable que revise los registros del sistema después de instalar el conector de datos. Ejecute:
>
> ```bash
> docker logs -f sapcon-[SID]
> ```
>
Para más información, consulte:

- [Ver todos los registros de ejecución de Docker](#view-all-docker-execution-logs)
- [Revisión y actualización de la configuración del conector de datos de SAP](#review-and-update-the-sap-data-connector-configuration)
- [Comandos útiles de Docker](#useful-docker-commands)

### <a name="view-all-docker-execution-logs"></a>Ver todos los registros de ejecución de Docker

Para ver todos los registros de ejecución de la implementación del conector de datos de Azure Sentinel para SAP, ejecute uno de los siguientes comandos:

```bash
docker exec -it sapcon-[SID] bash && cd /sapcon-app/sapcon/logs
```

o

```bash
docker exec –it sapcon-[SID] cat /sapcon-app/sapcon/logs/[FILE_LOGNAME]
```

Debe aparecer una salida similar a la siguiente:

```bash
Logs directory:
root@644c46cd82a9:/sapcon-app# ls sapcon/logs/ -l
total 508
-rwxr-xr-x 1 root root      0 Mar 12 09:22 ' __init__.py'
-rw-r--r-- 1 root root    282 Mar 12 16:01  ABAPAppLog.log
-rw-r--r-- 1 root root   1056 Mar 12 16:01  ABAPAuditLog.log
-rw-r--r-- 1 root root    465 Mar 12 16:01  ABAPCRLog.log
-rw-r--r-- 1 root root    515 Mar 12 16:01  ABAPChangeDocsLog.log
-rw-r--r-- 1 root root    282 Mar 12 16:01  ABAPJobLog.log
-rw-r--r-- 1 root root    480 Mar 12 16:01  ABAPSpoolLog.log
-rw-r--r-- 1 root root    525 Mar 12 16:01  ABAPSpoolOutputLog.log
-rw-r--r-- 1 root root      0 Mar 12 15:51  ABAPTableDataLog.log
-rw-r--r-- 1 root root    495 Mar 12 16:01  ABAPWorkflowLog.log
-rw-r--r-- 1 root root 465311 Mar 14 06:54  API.log # view this log to see submits of data into Azure Sentinel
-rw-r--r-- 1 root root      0 Mar 12 15:51  LogsDeltaManager.log
-rw-r--r-- 1 root root      0 Mar 12 15:51  PersistenceManager.log
-rw-r--r-- 1 root root   4830 Mar 12 16:01  RFC.log
-rw-r--r-- 1 root root   5595 Mar 12 16:03  SystemAdmin.log
```

### <a name="review-and-update-the-sap-data-connector-configuration"></a>Revisión y actualización de la configuración del conector de datos de SAP

Si desea comprobar el archivo de configuración del conector de datos de SAP y llevar a cabo actualizaciones manuales, realice los pasos siguientes:

1. En la máquina virtual, en el directorio principal del usuario, abra el archivo **~/sapcon/[SID]/systemconfig.ini**.
1. Actualice la configuración si es necesario y reinicie el contenedor:

    ```bash
    docker restart sapcon-[SID]
    ```

### <a name="useful-docker-commands"></a>Comandos útiles de Docker

Para solucionar problemas del conector de datos de SAP, puede que le resulten útiles los siguientes comandos:

|Función  |Comando  |
|---------|---------|
|**Detención del contenedor de Docker**     |  `docker stop sapcon-[SID]`       |
|**Inicio del contenedor de Docker**     |`docker start sapcon-[SID]`         |
|**Visualización de los registros del sistema de Docker**     |  `docker logs -f sapcon-[SID]`       |
|**Especificación del contenedor de Docker**     |   `docker exec -it sapcon-[SID] bash`      |
|     |         |

Para más información, consulte la [documentación sobre la CLI de Docker](https://docs.docker.com/engine/reference/commandline/docker/).

## <a name="update-your-sap-data-connector"></a>Actualización del conector de datos de SAP

Si ya tiene un contenedor de Docker en ejecución con una versión anterior del conector de datos de SAP, ejecute el script de actualización del conector de datos de SAP para obtener las características más recientes disponibles.

1. Asegúrese de que tiene las versiones más recientes de los scripts de implementación correspondientes del repositorio de Github de Azure Sentinel. Ejecute:

    ```azurecli
    - wget -O sapcon-sentinel-kickstart.sh https://raw.githubusercontent.com/Azure/Azure-Sentinel/master/Solutions/SAP/sapcon-sentinel-kickstart.sh && bash ./sapcon-sentinel-update.sh
    ```

1. Ejecute el siguiente comando en la máquina del conector de datos de SAP:

    ```azurecli
    ./ sapcon-instance-update.sh
    ```

El contenedor de Docker del conector de datos de SAP en la máquina está actualizado.

## <a name="next-steps"></a>Pasos siguientes

Obtenga más información sobre las soluciones de Azure Sentinel para SAP:

- [Implementación de la solución Azure Sentinel para SAP mediante implementaciones alternativas](sap-solution-deploy-alternate.md)
- [Requisitos detallados de SAP para la solución Azure Sentinel para SAP](sap-solution-detailed-requirements.md)
- [Referencia sobre los registros de la solución Azure Sentinel para SAP](sap-solution-log-reference.md)
- [Azure Sentinel para SAP: contenido de seguridad integrado](sap-solution-security-content.md)

Para más información, consulte [Soluciones de Azure Sentinel](sentinel-solutions.md).
