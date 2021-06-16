---
title: Puntos de conexión privados
description: Comprenda el proceso de creación de puntos de conexión privados para Azure Backup y los escenarios en los que el uso de puntos de conexión privados ayuda a preservar la seguridad de los recursos.
ms.topic: conceptual
ms.date: 05/07/2020
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: a44d6d31edb6329c11103e99f0b21aa1ea686ca3
ms.sourcegitcommit: df574710c692ba21b0467e3efeff9415d336a7e1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/28/2021
ms.locfileid: "110678329"
---
# <a name="private-endpoints-for-azure-backup"></a>Puntos de conexión privados para Azure Backup

Azure Backup permite realizar copias de seguridad de los datos y restaurarlos de forma segura desde los almacenes de Recovery Services mediante [puntos de conexión privados](../private-link/private-endpoint-overview.md). Los puntos de conexión privados usan una o más direcciones IP privadas de la red virtual para incorporar el servicio de manera eficaz a su red virtual.

Este artículo le ayudará a comprender el proceso de creación de puntos de conexión privados para Azure Backup y los escenarios en los que el uso de puntos de conexión privados ayuda a preservar la seguridad de los recursos.

## <a name="before-you-start"></a>Antes de comenzar

- Los puntos de conexión privados solo se pueden crear para almacenes de Recovery Services nuevos (que no tienen elementos registrados). Por lo tanto, los puntos de conexión privados deben crearse antes de intentar proteger los elementos en el almacén.
- Una red virtual puede contener puntos de conexión privados para varios almacenes de Recovery Services. Además, un almacén de Recovery Services puede tener puntos de conexión privados en varias redes virtuales. Sin embargo, el número máximo de puntos de conexión privados que se pueden crear para un almacén es 12.
- Una vez que se crea un punto de conexión privado para un almacén, el almacén se bloqueará. No será accesible (para copias de seguridad y restauraciones) desde redes que no contengan un punto de conexión privado para el almacén. Si se quitan todos los puntos de conexión privados para el almacén, se podrá acceder al almacén desde todas las redes.
- Una conexión de punto de conexión privado para la copia de seguridad usa un total de 11 direcciones IP privadas de la subred, incluidas las que usa Azure Backup para el almacenamiento. Este número puede ser superior (hasta veinticinco) para ciertas regiones de Azure. Por lo tanto, se recomienda disponer de suficientes direcciones IP privadas cuando intente crear puntos de conexión privados para la copia de seguridad.
- Aunque tanto Azure Backup como Azure Site Recovery usan un almacén de Recovery Services, en este artículo solo se describe el uso de puntos de conexión privados para Azure Backup.
- Actualmente, Azure Active Directory no admite puntos de conexión privados. Por tanto, las direcciones IP y los FQDN necesarios para que Azure Active Directory funcione en una región necesitarán que se les permita el acceso de salida desde la red protegida al realizar copias de seguridad de bases de datos en máquinas virtuales de Azure y copias de seguridad mediante el agente de MARS. También puede usar etiquetas NSG y etiquetas de Azure Firewall para permitir el acceso a Azure AD, según corresponda.
- No se admiten redes virtuales con directivas de red para los puntos de conexión privados. Deberá [deshabilitar las directivas de red](../private-link/disable-private-endpoint-network-policy.md) antes de continuar.
- Debe volver a registrar el proveedor de recursos de Recovery Services con la suscripción si lo registró antes del 1 de mayo de 2020. Para volver a registrar el proveedor, vaya a la suscripción en Azure Portal, haga clic en el **Proveedor de recursos** en la barra de navegación izquierda, seleccione **Microsoft.RecoveryServices** y **Volver a registrar**.
- Si el almacén tiene habilitados los puntos de conexión privados, no se puede realizar la [restauración entre regiones](backup-create-rs-vault.md#set-cross-region-restore) de las copias de seguridad de bases de datos SQL y SAP HANA.
- Al trasladar un almacén de Recovery Services que ya usa puntos de conexión privados a un nuevo inquilino, deberá actualizar el almacén de Recovery Services para volver a crear y configurar la identidad administrada del almacén y crear nuevos puntos de conexión privados según sea necesario (que deben estar en el nuevo inquilino). Si no lo hace, las operaciones de copia de seguridad y restauración comenzarán a generar errores. Además, los permisos de control de acceso basado en rol (RBAC) configurados en la suscripción deberán volver a configurarse.

## <a name="recommended-and-supported-scenarios"></a>Escenarios recomendados y admitidos

Aunque los puntos de conexión privados están habilitados para el almacén, solo se usan para la copia de seguridad y restauración de las cargas de trabajo de SQL y SAP HANA en una máquina virtual de Azure y en las copias de seguridad del agente de MARS. También puede usar el almacén para realizar copias de seguridad de otras cargas de trabajo (si bien no requerirán los puntos de conexión privados). Además de la copia de seguridad de las cargas de trabajo de SQL y SAP HANA y de las copias de seguridad mediante el agente de MARS, los puntos de conexión privados también se usan para realizar la recuperación de archivos para la copia de seguridad de máquinas virtuales de Azure. Para más información, vea la tabla siguiente:

| Copia de seguridad de cargas de trabajo en máquinas virtuales de Azure (SQL, SAP HANA), copia de seguridad mediante el agente de MARS | Se recomienda el uso de puntos de conexión privados para permitir copias de seguridad y restauración sin necesidad de agregar las listas de direcciones IP o FQDN a una lista de permitidos para Azure Backup o Azure Storage desde las redes virtuales. En ese escenario, asegúrese de que las máquinas virtuales que hospedan las bases de datos SQL pueden comunicarse con direcciones IP o FQDN de Azure AD. |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| **Copia de seguridad de la máquina virtual de Azure**                                         | La copia de seguridad de la máquina virtual no requiere que permita el acceso a direcciones IP o FQDN. Por lo tanto, no requiere puntos de conexión privados para la copia de seguridad y restauración de discos.  <br><br>   Sin embargo, la recuperación de archivos desde un almacén que contiene puntos de conexión privados se restringe a redes virtuales que contienen un punto de conexión privado para el almacén. <br><br>    Al usar discos no administrados de la ACL, asegúrese de que la cuenta de almacenamiento que contiene los discos permite el acceso a **servicios de Microsoft de confianza** si forma parte de la ACL. |
| **Copia de seguridad de Azure Files**                                      | Las copias de seguridad de Azure Files se almacenan en la cuenta de almacenamiento local. Por lo tanto, no requiere puntos de conexión privados para la copia de seguridad y restauración. |

## <a name="get-started-with-creating-private-endpoints-for-backup"></a>Introducción a la creación de puntos de conexión privados para Backup

En las siguientes secciones se tratan los pasos necesarios para crear y usar puntos de conexión privados para Azure Backup dentro de las redes virtuales.

>[!IMPORTANT]
> Se recomienda encarecidamente que siga los pasos en la misma secuencia que se mencionan en este documento. Si no lo hace, puede que el almacén sea incompatible con el uso de puntos de conexión privados y que tenga que reiniciar el proceso con un nuevo almacén.

## <a name="create-a-recovery-services-vault"></a>Creación de un almacén de Recovery Services

Los puntos de conexión privados para Backup solo se pueden crear para almacenes de Recovery Services que no tengan ningún elemento protegido (o que no hayan intentado protegerse o registrarse en él en el pasado). Por lo tanto, se recomienda crear un nuevo almacén para comenzar. Para más información sobre la creación de un nuevo almacén, consulte [Creación y configuración de un almacén de Recovery Services](backup-create-rs-vault.md).

Consulte [esta sección](#create-a-recovery-services-vault-using-the-azure-resource-manager-client) para aprender a crear un almacén mediante el cliente de Azure Resource Manager. Esto crea un almacén con su identidad administrada ya habilitada.

## <a name="enable-managed-identity-for-your-vault"></a>Habilitar la identidad administrada del almacén

Las identidades administradas permiten al almacén crear y usar puntos de conexión privados. En esta sección se habla sobre cómo habilitar la identidad administrada del almacén.

1. Vaya al almacén de Recovery Services -> **Identidad**.

    ![Cambiar el estado de la identidad a activada](./media/private-endpoints/identity-status-on.png)

1. Cambie el **Estado** a **Activado** y seleccione **Guardar**.

1. Se genera una **Id. de objeto**, que es la identidad administrada del almacén.

    >[!NOTE]
    >Una vez habilitada, la identidad administrada **NO** debe deshabilitarse (ni siquiera temporalmente). Deshabilitar la identidad administrada puede provocar un comportamiento incoherente.

## <a name="grant-permissions-to-the-vault-to-create-required-private-endpoints"></a>Concesión de permisos al almacén para crear los puntos de conexión privados necesarios

Para crear los puntos de conexión privados necesarios para Azure Backup, el almacén (la identidad administrada del almacén) debe tener permisos para los siguientes grupos de recursos:

- El grupo de recursos que contiene la red virtual de destino
- El grupo de recursos donde se crearán los puntos de conexión privados
- El grupo de recursos que contiene las zonas DNS privadas, como se explica [aquí](#create-private-endpoints-for-azure-backup) de forma detallada.

Se recomienda conceder el rol **Colaborador** para estos tres grupos de recursos en el almacén (identidad administrada). En los pasos siguientes se describe cómo hacer esto para un determinado grupo de recursos (esto se debe hacer para cada uno de los tres grupos de recursos):

1. Vaya al grupo de recursos y navegue a **Control de acceso (IAM)** en la barra de la izquierda.
1. Una vez en **Control de acceso**, vaya a **Agregar una asignación de roles**.

    ![Adición de una asignación de roles](./media/private-endpoints/add-role-assignment.png)

1. En el panel de **Agregar asignación de roles**, seleccione **Colaborador** como **Rol** y use el **Nombre** del almacén como **Entidad de seguridad**. Seleccione el almacén y, cuando haya terminado, seleccione **Guardar**.

    ![Elegir rol y entidad de seguridad](./media/private-endpoints/choose-role-and-principal.png)

Para administrar los permisos de forma más detallada, consulte [Crear roles y permisos manualmente](#create-roles-and-permissions-manually).

## <a name="create-private-endpoints-for-azure-backup"></a>Creación de puntos de conexión privados para Azure Backup

En esta sección se explica cómo crear un punto de conexión privado para el almacén.

1. Navegue hasta el almacén creado anteriormente y vaya a **Conexiones de punto de conexión privado** en la barra de navegación izquierda. Seleccione **+ Punto de conexión privado** en la parte superior para comenzar la creación de un nuevo punto de conexión privado para este almacén.

    ![Creación de un punto de conexión privado](./media/private-endpoints/new-private-endpoint.png)

1. Una vez que empiece el proceso para **Crear un punto de conexión privado**, se le pedirá que especifique los detalles para crear la conexión de punto de conexión privado.
  
    1. **Aspectos básicos**: Rellene los detalles básicos de los puntos de conexión privados. La región debe ser la misma que la del almacén y el recurso de los que se hará una copia de seguridad.

        ![Rellene los detalles básicos](./media/private-endpoints/basics-tab.png)

    1. **Recursos**: esta pestaña requiere que seleccione el recurso de PaaS para el que quiere crear la conexión. Seleccione **Microsoft.RecoveryServices/vaults** del tipo de recurso para la suscripción deseada. Una vez hecho esto, elija el nombre del almacén de Recovery Services como **Recurso** y **AzureBackup** como el **Subrecurso de destino**.

        ![Selección del recurso para la conexión](./media/private-endpoints/resource-tab.png)

    1. **Configuración**: En Configuración, especifique la red virtual y la subred en la que desea que se cree el punto de conexión privado. Esta será la red virtual donde está presente la máquina virtual.

        Para conectarse de manera privada, necesita los registros DNS necesarios. Según la configuración de la red, puede elegir una de las siguientes opciones:

          - Integrar el punto de conexión privado con una zona DNS privada: seleccione **Sí** si quiere realizar la integración.
          - Usar el servidor DNS personalizado: seleccione **No** si quiere usar su propio servidor DNS.

        La administración de registros DNS para ambas opciones se [describe más adelante](#manage-dns-records).

          ![Especificación de la red virtual y la subred](./media/private-endpoints/configuration-tab.png)

    1. Opcionalmente, puede agregar **Etiquetas** para el punto de conexión privado.
    1. Continúe para **Revisar y crear** una vez que haya terminado de escribir los detalles. Una vez finalizada la validación, seleccione **Crear** para crear el punto de conexión privado.

## <a name="approve-private-endpoints"></a>Aprobación de puntos de conexión privados

Si el usuario que crea el punto de conexión privado también es el propietario del almacén de Recovery Services, el punto de conexión privado creado anteriormente se aprobará automáticamente. De lo contrario, el propietario del almacén debe aprobar el punto de conexión privado antes de poder usarlo. En esta sección se describe la aprobación manual de los puntos de conexión privados a través de Azure Portal.

Consulte la sección [Aprobación manual de los puntos de conexión privados mediante el cliente de Azure Resource Manager](#manual-approval-of-private-endpoints-using-the-azure-resource-manager-client) para usar el cliente de Azure Resource Manager para la aprobación de puntos de conexión privados.

1. En el almacén de Recovery Services, vaya a **Conexiones de punto de conexión privado** en la barra de la izquierda.
1. Seleccione la conexión de punto de conexión privado que desea aprobar.
1. Seleccione **Aprobar** en la barra superior. También puede seleccionar **Rechazar** o **Quitar** si desea rechazar o eliminar la conexión del punto de conexión.

    ![Aprobación de puntos de conexión privados](./media/private-endpoints/approve-private-endpoints.png)

## <a name="manage-dns-records"></a>Administración de registros DNS

Tal y como se ha descrito anteriormente, necesita los registros DNS necesarios en sus servidores o zonas DNS privados para conectarse de manera privada. Puede integrar su punto de conexión privado directamente con zonas DNS privadas de Azure, o usar los servidores DNS personalizados para ello, en función de las preferencias de red. Tendrá que realizar este procedimiento con los tres servicios: Backup, blobs y colas.

### <a name="when-integrating-private-endpoints-with-azure-private-dns-zones"></a>Durante la integración de puntos de conexión privados con las zonas DNS privadas de Azure

Si decide integrar su punto de conexión privado con zonas DNS privadas, Backup agregará los registros DNS necesarios. Puede ver las zonas DNS privadas que se están usando en la **Configuración DNS** del punto de conexión privado. Si estas zonas DNS no están presentes, se crearán automáticamente al crear el punto de conexión privado. Sin embargo, debe comprobar que la red virtual (que contiene los recursos de los que se va a realizar la copia de seguridad) esté vinculada correctamente a las tres zonas DNS privadas, como se describe a continuación.

![Configuración DNS en la zona DNS privada de Azure](./media/private-endpoints/dns-configuration.png)

#### <a name="validate-virtual-network-links-in-private-dns-zones"></a>Validación de vínculos de red virtual en zonas DNS privadas

Haga lo siguiente para **cada zona DNS privada** indicada anteriormente (para Backup, blobs y colas):

1. Vaya a la opción **Vínculos de red virtual** correspondiente en la barra de navegación izquierda.
1. Debe ver una entrada para la red virtual para la que ha creado el punto de conexión privado, como se muestra a continuación:

    ![Red virtual para el punto de conexión privado](./media/private-endpoints/virtual-network-links.png)

1. Si no ve ninguna entrada, agregue un vínculo de red virtual a todas las zonas DNS que no tengan una.

    ![Agregar el vínculo de red virtual](./media/private-endpoints/add-virtual-network-link.png)

### <a name="when-using-custom-dns-server-or-host-files"></a>Durante el uso de un servidor DNS personalizado o archivos de host

Si usa servidores DNS personalizados, deberá crear las zonas DNS necesarias y agregar los registros DNS necesarios para los puntos de conexión privados a los servidores DNS. En el caso de los blobs y las colas, también puede usar reenviadores condicionales.

#### <a name="for-the-backup-service"></a>En el caso del servicio de Backup

1. En el servidor DNS, cree una zona DNS para Backup según la siguiente convención de nomenclatura:

    |Zona |Servicio |
    |---------|---------|
    |`privatelink.<geo>.backup.windowsazure.com`   |  Copia de seguridad        |

    >[!NOTE]
    > En el texto anterior, `<geo>` hace referencia al código de región (por ejemplo, *eus* y *ne* para Este de EE. UU. y Norte de Europa respectivamente). Consulte las siguientes listas para ver los códigos de región:
    >
    > - [Todas las nubes públicas](https://download.microsoft.com/download/1/2/6/126a410b-0e06-45ed-b2df-84f353034fa1/AzureRegionCodesList.docx)
    > - [China](/azure/china/resources-developer-guide#check-endpoints-in-azure)
    > - [Alemania](../germany/germany-developer-guide.md#endpoint-mapping)
    > - [US Gov](../azure-government/documentation-government-developer-guide.md)

1. A continuación, es necesario agregar los registros DNS necesarios. Para ver los registros que deben agregarse a la zona DNS de Backup, navegue hasta el punto de conexión privado que creó anteriormente y vaya a la opción **Configuración DNS** en la barra de navegación izquierda.

    ![Configuración DNS para el servidor DNS personalizado](./media/private-endpoints/custom-dns-configuration.png)

1. Agregue una entrada para cada FQDN e IP que se muestra como registros de tipo D en la zona DNS para Backup. Si usa un archivo de host para la resolución de nombres, realice las entradas correspondientes en el archivo de host para cada IP y FQDN según el formato siguiente:

    `<private ip><space><backup service privatelink FQDN>`

>[!NOTE]
>Como se muestra en la captura de pantalla anterior, los FQDN se describen como `xxxxxxxx.<geo>.backup.windowsazure.com` y no como `xxxxxxxx.privatelink.<geo>.backup. windowsazure.com`. En tales casos, asegúrese de incluir (y, si es necesario, agregar) el elemento `.privatelink.` según el formato indicado.

#### <a name="for-blob-and-queue-services"></a>En el caso de los servicios de blob y cola

En el caso de los blobs y las colas, puede usar reenviadores condicionales o crear zonas DNS en el servidor DNS.

##### <a name="if-using-conditional-forwarders"></a>Si usa reenviadores condicionales

Si usa reenviadores condicionales, agregue los reenviadores para los FQDN de blobs y colas de la siguiente manera:

|FQDN  |IP  |
|---------|---------|
|`privatelink.blob.core.windows.net`     |  168.63.129.16       |
|`privatelink.queue.core.windows.net`     | 168.63.129.16        |

##### <a name="if-using-private-dns-zones"></a>Si usa zonas DNS privadas

Si usa zonas DNS para blobs y colas, primero deberá crear estas zonas DNS y agregar después los registros D necesarios.

|Zona |Servicio  |
|---------|---------|
|`privatelink.blob.core.windows.net`     |  Blob     |
|`privatelink.queue.core.windows.net`     | Cola        |

En este momento, solo crearemos las zonas para los blobs y las colas al usar servidores DNS personalizados. La adición de registros DNS se realizará más adelante en dos pasos:

1. Cuando se registra la primera instancia de copia de seguridad; es decir, cuando se configura la copia de seguridad por primera vez.
1. Cuando se ejecuta la primera copia de seguridad.

Realizaremos estos pasos en las secciones siguientes.

## <a name="use-private-endpoints-for-backup"></a>Uso de puntos de conexión privados para Backup

Una vez que los puntos de conexión privados creados para el almacén de la red virtual se han aprobado, puede empezar a usarlos para realizar las copias de seguridad y restauraciones.

>[!IMPORTANT]
>Asegúrese de haber completado correctamente todos los pasos mencionados previamente en el documento antes de continuar. En resumen, debe haber completado los pasos de la siguiente lista:
>
>1. Se creó un (nuevo) almacén de Recovery Services
>2. Se habilitó el almacén para usar la identidad administrada asignada por el sistema
>3. Se asignaron los permisos relevantes a la identidad administrada del almacén
>4. Se creó un punto de conexión privado para el almacén
>5. Se aprobó el punto de conexión privado (si no se aprobó automáticamente)
>6. Se garantizó que todos los registros DNS se agregaron correctamente (excepto los registros de blobs y colas para los servidores personalizados, que se tratarán en las secciones siguientes).

### <a name="check-vm-connectivity"></a>Comprobación de la conectividad de VM

En la VM de la red bloqueada, asegúrese de cumplir con lo siguiente:

1. La VM debe tener acceso a AAD.
2. Ejecute **nslookup** en la dirección URL de copia de seguridad (`xxxxxxxx.privatelink.<geo>.backup. windowsazure.com`) de la VM para confirmar la conectividad. Esto debe devolver la dirección IP privada asignada a la red virtual.

### <a name="configure-backup"></a>Configuración de la copia de seguridad

Una vez que confirme que se completó correctamente la lista de comprobación anterior y el acceso, puede seguir configurando la copia de seguridad de las cargas de trabajo en el almacén. Si usa un servidor DNS personalizado, deberá agregar entradas DNS para los blobs y las colas que están disponibles después de configurar la primera copia de seguridad.

#### <a name="dns-records-for-blobs-and-queues-only-for-custom-dns-servershost-files-after-the-first-registration"></a>Registros DNS para blobs y colas (solo para servidores DNS o archivos de host personalizados) después del primer registro

Después de configurar la copia de seguridad para al menos un recurso en un almacén habilitado para puntos de conexión privados, agregue los registros DNS necesarios para los blobs y las colas tal como se describe a continuación.

1. Navegue hasta el grupo de recursos y busque el punto de conexión privado que creó.
1. Aparte del nombre de punto de conexión privado que asignó, verá que se crearon otros dos puntos de conexión privados. Estos comienzan con `<the name of the private endpoint>_ecs` y tienen el sufijo `_blob` y `_queue` respectivamente.

    ![Recursos de punto de conexión privado](./media/private-endpoints/private-endpoint-resources.png)

1. Navegue a cada uno de estos puntos de conexión privados. En la opción de configuración DNS de cada uno de los dos puntos de conexión privados, verá un registro con un FQDN y una dirección IP. Agregue ambos valores al servidor DNS personalizado, además de los descritos anteriormente.
Si usa un archivo de host, realice las entradas correspondientes en el archivo de host para cada IP o FQDN según el formato siguiente:

    `<private ip><space><blob service privatelink FQDN>`<br>
    `<private ip><space><queue service privatelink FQDN>`

    ![Configuración DNS de blob](./media/private-endpoints/blob-dns-configuration.png)

Además de lo anterior, es necesaria otra entrada después de la primera copia de seguridad, que se describe [más adelante](#dns-records-for-blobs-only-for-custom-dns-servershost-files-after-the-first-backup).

### <a name="backup-and-restore-of-workloads-in-azure-vm-sql-and-sap-hana"></a>Copia de seguridad y restauración de cargas de trabajo en VM de Azure (SQL y SAP HANA)

Una vez que se crea y se aprueba el punto de conexión privado, no se requiere ningún otro cambio del lado cliente para usar el punto de conexión privado (a menos que esté usando grupos de disponibilidad de SQL, que se describen más adelante en esta sección). Toda la comunicación y la transferencia de datos desde la red segura al almacén se realizará a través del punto de conexión privado. Sin embargo, si quita los puntos de conexión privados del almacén después de haber registrado un servidor (SQL o SAP HANA), deberá volver a registrar el contenedor con el almacén. No es necesario detener la protección de los mismos.

#### <a name="dns-records-for-blobs-only-for-custom-dns-servershost-files-after-the-first-backup"></a>Registros DNS para blobs (solo para servidores DNS o archivos de host personalizados) después de la primera copia de seguridad

Después de ejecutar la primera copia de seguridad y de usar un servidor DNS personalizado (sin reenvío condicional), es probable que se produzca un error en la copia de seguridad. Si eso ocurre, haga lo siguiente:

1. Navegue hasta el grupo de recursos y busque el punto de conexión privado que creó.
1. Además de los tres puntos de conexión privados descritos anteriormente, ahora verá un cuarto punto de conexión privado cuyo nombre empieza por `<the name of the private endpoint>_prot` y contiene el sufijo `_blob`.

    ![Punto de conexión privado con el sufijo "prot"](./media/private-endpoints/private-endpoint-prot.png)

1. Navegue hasta este nuevo punto de conexión privado. En la opción Configuración DNS, verá un registro con un FQDN y una dirección IP. Agregue los valores al servidor DNS privado, además de los descritos anteriormente.

    Si usa un archivo de host, realice las entradas correspondientes en el archivo de host para cada IP y FQDN según el formato siguiente:

    `<private ip><space><blob service privatelink FQDN>`

>[!NOTE]
>En este momento, debe poder ejecutar **nslookup** desde la VM y resolver las direcciones IP privadas cuando se realice en las direcciones URL de almacenamiento y copia de seguridad del almacén.

### <a name="when-using-sql-availability-groups"></a>Durante el uso de grupos de disponibilidad de SQL

Cuando use grupos de disponibilidad de SQL, deberá aprovisionar el reenvío condicional en el DNS de grupo de disponibilidad personalizado como se describe a continuación:

1. Inicie sesión en su controlador de dominio.
1. En la aplicación DNS, agregue reenviadores condicionales para las tres zonas DNS (Backup, blobs y colas) a la dirección IP 168.63.129.16 o la dirección IP del servidor DNS personalizado, según sea necesario. Las siguientes capturas de pantallas se muestran cuando se reenvía a la dirección IP del host de Azure. Si usa su propio servidor DNS, reemplácelo por la dirección IP del servidor DNS.

    ![Reenviadores condicionales en el administrador de DNS](./media/private-endpoints/dns-manager.png)

    ![Nuevo reenviador condicional](./media/private-endpoints/new-conditional-forwarder.png)

### <a name="backup-and-restore-through-mars-agent"></a>Copia de seguridad y restauración mediante el agente de MARS

Al usar el agente de MARS para realizar una copia de seguridad de los recursos locales, asegúrese de que la red local (que contiene los recursos de los que se va a realizar la copia de seguridad) está emparejada con la red virtual de Azure que contiene un punto de conexión privado para el almacén, para que pueda usarla. Después, puede continuar con la instalación del agente de MARS y configurar la copia de seguridad como se detalla aquí. Sin embargo, debe asegurarse de que toda la comunicación para la copia de seguridad se produzca solo a través de la red emparejada.

Pero si quita los puntos de conexión privados del almacén después de haber registrado un agente de MARS, deberá volver a registrar el contenedor con el almacén. No es necesario detener la protección de los mismos.

## <a name="deleting-private-endpoints"></a>Eliminación de puntos de conexión privados

Consulte [esta sección](/rest/api/virtualnetwork/privateendpoints/delete) para obtener información sobre cómo eliminar puntos de conexión privados.

## <a name="additional-topics"></a>Otros temas

### <a name="create-a-recovery-services-vault-using-the-azure-resource-manager-client"></a>Creación de un almacén de Recovery Services mediante el cliente de Azure Resource Manager

Puede crear el almacén de Recovery Services y habilitar su identidad administrada (se requiere la habilitación de la identidad administrada, como veremos más adelante) mediante el cliente de Azure Resource Manager. A continuación, se muestra un ejemplo de cómo hacerlo:

```rest
armclient PUT /subscriptions/<subscriptionid>/resourceGroups/<rgname>/providers/Microsoft.RecoveryServices/Vaults/<vaultname>?api-version=2017-07-01-preview @C:\<filepath>\MSIVault.json
```

El archivo JSON anterior debe tener el siguiente contenido:

Solicitud JSON:

```json
{
  "location": "eastus2",
  "name": "<vaultname>",
  "etag": "W/\"datetime'2019-05-24T12%3A54%3A42.1757237Z'\"",
  "tags": {
    "PutKey": "PutValue"
  },
  "properties": {},
  "id": "/subscriptions/<subscriptionid>/resourceGroups/<rgname>/providers/Microsoft.RecoveryServices/Vaults/<vaultname>",
  "type": "Microsoft.RecoveryServices/Vaults",
  "sku": {
    "name": "RS0",
    "tier": "Standard"
  },
  "identity": {
    "type": "systemassigned"
  }
}
```

Respuesta JSON:

```json
{
   "location": "eastus2",
   "name": "<vaultname>",
   "etag": "W/\"datetime'2020-02-25T05%3A26%3A58.5181122Z'\"",
   "tags": {
     "PutKey": "PutValue"
   },
   "identity": {
     "tenantId": "<tenantid>",
     "principalId": "<principalid>",
     "type": "SystemAssigned"
   },
   "properties": {
     "provisioningState": "Succeeded",
     "privateEndpointStateForBackup": "None",
     "privateEndpointStateForSiteRecovery": "None"
   },
   "id": "/subscriptions/<subscriptionid>/resourceGroups/<rgname>/providers/Microsoft.RecoveryServices/Vaults/<vaultname>",
   "type": "Microsoft.RecoveryServices/Vaults",
   "sku": {
     "name": "RS0",
     "tier": "Standard"
   }
 }
```

>[!NOTE]
>El almacén creado en este ejemplo mediante Azure Resource Manager ya se ha creado con una identidad administrada asignada por el sistema.

### <a name="managing-permissions-on-resource-groups"></a>Administración de permisos en grupos de recursos

La identidad administrada del almacén debe tener los permisos siguientes en el grupo de recursos y la red virtual donde se crearán los puntos de conexión privados:

- `Microsoft.Network/privateEndpoints/*` Esto es necesario para realizar operaciones CRUD en puntos de conexión privados en el grupo de recursos. Debe asignarse en el grupo de recursos.
- `Microsoft.Network/virtualNetworks/subnets/join/action` Esto es necesario en la red virtual en la que la dirección IP privada se conecta con el punto de conexión privado.
- `Microsoft.Network/networkInterfaces/read` Esto es necesario en el grupo de recursos para que se cree la interfaz de red para el punto de conexión privado.
- Rol de colaborador de zona DNS privada: este rol ya existe y se puede usar para proporcionar los permisos `Microsoft.Network/privateDnsZones/A/*` y `Microsoft.Network/privateDnsZones/virtualNetworkLinks/read`.

Puede usar uno de los métodos siguientes para crear roles con los permisos necesarios:

#### <a name="create-roles-and-permissions-manually"></a>Crear roles y permisos manualmente

Cree los siguientes archivos JSON y use el comando de PowerShell que se encuentra al final de la sección para crear roles:

//PrivateEndpointContributorRoleDef.json

```json
{
  "Name": "PrivateEndpointContributor",
  "Id": null,
  "IsCustom": true,
  "Description": "Allows management of Private Endpoint",
  "Actions": [
    "Microsoft.Network/privateEndpoints/*",
  ],
  "NotActions": [],
  "AssignableScopes": [
    "/subscriptions/00000000-0000-0000-0000-000000000000"
  ]
}
```

//NetworkInterfaceReaderRoleDef.json

```json
{
  "Name": "NetworkInterfaceReader",
  "Id": null,
  "IsCustom": true,
  "Description": "Allows read on networkInterfaces",
  "Actions": [
    "Microsoft.Network/networkInterfaces/read"
  ],
  "NotActions": [],
  "AssignableScopes": [
    "/subscriptions/00000000-0000-0000-0000-000000000000"
  ]
}
```

//PrivateEndpointSubnetContributorRoleDef.json

```json
{
  "Name": "PrivateEndpointSubnetContributor",
  "Id": null,
  "IsCustom": true,
  "Description": "Allows adding of Private Endpoint connection to Virtual Networks",
  "Actions": [
    "Microsoft.Network/virtualNetworks/subnets/join/action"
  ],
  "NotActions": [],
  "AssignableScopes": [
    "/subscriptions/00000000-0000-0000-0000-000000000000"
  ]
}
```

```azurepowershell
 New-AzRoleDefinition -InputFile "PrivateEndpointContributorRoleDef.json"
 New-AzRoleDefinition -InputFile "NetworkInterfaceReaderRoleDef.json"
 New-AzRoleDefinition -InputFile "PrivateEndpointSubnetContributorRoleDef.json"
```

#### <a name="use-a-script"></a>Usar un script

1. Inicie **Cloud Shell** en Azure Portal y seleccione **Cargar archivo** en la ventana de PowerShell.

    ![Seleccionar Cargar archivo en la ventana de PowerShell](./media/private-endpoints/upload-file-in-powershell.png)

1. Cargue el siguiente script: [VaultMsiPrereqScript](https://download.microsoft.com/download/1/2/6/126a410b-0e06-45ed-b2df-84f353034fa1/VaultMsiPrereqScript.ps1)

1. Vaya a la carpeta principal (por ejemplo: `cd /home/user`)

1. Ejecute el siguiente script:

    ```azurepowershell
    ./VaultMsiPrereqScript.ps1 -subscription <subscription-Id> -vaultPEResourceGroup <vaultPERG> -vaultPESubnetResourceGroup <subnetRG> -vaultMsiName <msiName>
    ```

    Estos son los parámetros:

    - **subscription**: **SubscriptionId que tiene el grupo de recursos en el que se va a crear el punto de conexión privado para el almacén y la subred en la que se adjuntará el punto de conexión privado del almacén

    - **vaultPEResourceGroup**: El grupo de recursos donde se creará el punto de conexión privado para el almacén

    - **vaultPESubnetResourceGroup**: El grupo de recursos de la subred a la que se unirá el punto de conexión privado

    - **vaultMsiName**: Nombre del archivo MSI del almacén, que es igual que **VaultName**

1. Complete la autenticación y el script tomará el contexto de la suscripción especificada que se indicó anteriormente. Se crearán los roles adecuados si faltan en el inquilino y se asignarán roles al archivo MSI del almacén.

### <a name="creating-private-endpoints-using-azure-powershell"></a>Creación de puntos de conexión privados mediante Azure PowerShell

#### <a name="auto-approved-private-endpoints"></a>Puntos de conexión privados aprobados automáticamente

```azurepowershell
$vault = Get-AzRecoveryServicesVault `
        -ResourceGroupName $vaultResourceGroupName `
        -Name $vaultName
  
$privateEndpointConnection = New-AzPrivateLinkServiceConnection `
        -Name $privateEndpointConnectionName `
        -PrivateLinkServiceId $vault.ID `
        -GroupId "AzureBackup"  

$vnet = Get-AzVirtualNetwork -Name $vnetName -ResourceGroupName $VMResourceGroupName
$subnet = $vnet | Select -ExpandProperty subnets | Where-Object {$_.Name -eq '<subnetName>'}


$privateEndpoint = New-AzPrivateEndpoint `
        -ResourceGroupName $vmResourceGroupName `
        -Name $privateEndpointName `
        -Location $location `
        -Subnet $subnet `
        -PrivateLinkServiceConnection $privateEndpointConnection `
        -Force
```

### <a name="manual-approval-of-private-endpoints-using-the-azure-resource-manager-client"></a>Aprobación manual de los puntos de conexión privados mediante el cliente de Azure Resource Manager

1. Use **GetVault** para obtener el id. de conexión de punto de conexión privado para el punto de conexión privado.

    ```rest
    armclient GET /subscriptions/<subscriptionid>/resourceGroups/<rgname>/providers/Microsoft.RecoveryServices/vaults/<vaultname>?api-version=2017-07-01-preview
    ```

    Esto devolverá el identificador de conexión del punto de conexión privado. El nombre de la conexión se puede recuperar utilizando la primera parte del identificador de conexión como se indica a continuación:

    `privateendpointconnectionid = {peName}.{vaultId}.backup.{guid}`

1. Obtenga el **identificador de conexión del punto de conexión privado** (y el **nombre del punto de conexión privado**, donde sea necesario) de la respuesta y reemplácelo en el siguiente JSON y el URI de Azure Resource Manager e intente cambiar el estado a "Aprobado/Rechazado/Desconectado", como se muestra en el ejemplo siguiente:

    ```rest
    armclient PUT /subscriptions/<subscriptionid>/resourceGroups/<rgname>/providers/Microsoft.RecoveryServices/Vaults/<vaultname>/privateEndpointConnections/<privateendpointconnectionid>?api-version=2020-02-02-preview @C:\<filepath>\BackupAdminApproval.json
    ```

    JSON:

    ```json
    {
    "id": "/subscriptions/<subscriptionid>/resourceGroups/<rgname>/providers/Microsoft.RecoveryServices/Vaults/<vaultname>/privateEndpointConnections/<privateendpointconnectionid>",
    "properties": {
        "privateEndpoint": {
        "id": "/subscriptions/<subscriptionid>/resourceGroups/<pergname>/providers/Microsoft.Network/privateEndpoints/pename"
        },
        "privateLinkServiceConnectionState": {
        "status": "Disconnected",  //choose state from Approved/Rejected/Disconnected
        "description": "Disconnected by <userid>"
        }
    }
    }
    ```

## <a name="frequently-asked-questions"></a>Preguntas más frecuentes

Q. ¿Puedo crear un punto de conexión privado para un almacén de Azure Backup existente?<br>
A. No, los puntos de conexión privados solo se pueden crear para nuevos almacenes de Azure Backup. Por lo tanto, el almacén no debe haber tenido ningún elemento protegido. De hecho, no se puede realizar ningún intento de proteger ningún elemento en el almacén antes de crear puntos de conexión privados.

Q. Intenté proteger un elemento en el almacén, pero se produjo un error y el almacén todavía no contiene elementos protegidos. ¿Se pueden crear puntos de conexión privados para este almacén?<br>
A. No, el almacén no debe haber tenido ningún intento de proteger ningún elemento en el pasado.

Q. Tengo un almacén que utiliza puntos de conexión privados con fines de copias de seguridad y restauración. ¿Puedo agregar o quitar puntos de conexión privados para este almacén más tarde, aunque tenga elementos de copia de seguridad protegidos?<br>
A. Sí. Si ya creó puntos de conexión privados para un almacén y hay elementos de copia de seguridad protegidos en él, más adelante podrá agregar o quitar puntos de conexión privados según sea necesario.

Q. ¿Se puede usar también el punto de conexión privado de Azure Backup en Azure Site Recovery?<br>
A. No, el punto de conexión privado para Azure Backup solo se puede usar en Azure Backup. Tendrá que crear un nuevo punto de conexión privado para Azure Site Recovery, si es compatible con el servicio.

Q. Me faltó uno de los pasos de este artículo y procedí a proteger el origen de datos. ¿Puedo seguir usando puntos de conexión privados?<br>
A. No seguir los pasos del artículo y proceder con la protección de elementos puede provocar que el almacén no pueda usar puntos de conexión privados. Por lo tanto, se recomienda que consulte esta lista de comprobación antes de continuar con la protección de los elementos.

Q. ¿Puedo usar mi propio servidor DNS en lugar de usar la zona DNS privada de Azure o una zona DNS privada integrada?<br>
A. Sí, puede usar sus propios servidores DNS. Sin embargo, asegúrese de agregar todos los registros DNS necesarios como se sugiere en esta sección.

Q. ¿Es necesario realizar algún paso adicional en el servidor después de haber seguido el proceso detallado en este artículo?<br>
A. Después de seguir el proceso detallado en este artículo, no es necesario realizar ningún trabajo adicional para usar puntos de conexión privados para copias de seguridad y restauración.

## <a name="next-steps"></a>Pasos siguientes

- Obtenga información sobre todas las [características de seguridad de Azure Backup](security-overview.md).
