---
title: Configuración del cuórum de clúster
description: 'Obtenga información sobre cómo configurar un testigo de disco, un testigo en la nube o un testigo de recurso compartido de archivos como cuórum para un clúster de conmutación por error de Windows Server en SQL Server en VM de Azure. '
services: virtual-machines
documentationCenter: na
author: cawrites
editor: ''
tags: azure-service-management
ms.service: virtual-machines-sql
ms.subservice: hadr
ms.topic: conceptual
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: iaas-sql-server
ms.date: 04/30/2021
ms.author: chadam
ms.openlocfilehash: 6b974628adeb1d1562d3735fcb55704adc8e2d98
ms.sourcegitcommit: ff1aa951f5d81381811246ac2380bcddc7e0c2b0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/07/2021
ms.locfileid: "111573287"
---
# <a name="configure-cluster-quorum-for-sql-server-on-azure-vms"></a>Configuración del cuórum de clúster para SQL Server en VM de Azure
[!INCLUDE[appliesto-sqlvm](../../includes/appliesto-sqlvm.md)]

En este artículo se enseña a configurar una de las tres opciones de cuórum para un clúster de conmutación por error de Windows Server que se ejecuta en SQL Server en máquinas virtuales (VM) de Azure: un testigo de disco, un testigo en la nube y un testigo de recurso compartido de archivos.


## <a name="overview"></a>Información general

El cuórum de un clúster viene determinado por el número de elementos de votación que deben formar parte de la pertenencia activa al clúster para que el clúster se inicie correctamente o continúe ejecutándose. La configuración de un recurso de cuórum permite que un clúster de dos nodos continúe con un solo nodo en línea. El clúster de conmutación por error de Windows Server es la tecnología subyacente para las opciones de alta disponibilidad de SQL Server en máquinas virtuales de Azure: [instancias de clúster de conmutación por error (FCI)](failover-cluster-instance-overview.md) y [grupos de disponibilidad (AG).](availability-group-overview.md) 

El testigo de disco es la opción de cuórum más resistente, pero para usar un testigo de disco en SQL Server en una máquina virtual de Azure, debe usar un disco compartido de Azure que imponga algunas limitaciones a la solución de alta disponibilidad. Por lo tanto, use un testigo de disco al configurar la instancia de clúster de conmutación por error con discos compartidos de Azure; de lo contrario, use un testigo en la nube siempre que sea posible. Si usa Windows Server 2012 R2 o una versión anterior que no admita un testigo en la nube, puede usar un testigo de recurso compartido de archivos. 

Las siguientes opciones de cuórum están disponibles para su uso con SQL Server en máquinas virtuales de Azure: 

|  |[Testigo en la nube](/windows-server/failover-clustering/deploy-cloud-witness) |[Testigo de disco](/windows-server/failover-clustering/manage-cluster-quorum#configure-the-cluster-quorum) |[Testigo de recurso compartido de archivos](/windows-server/failover-clustering/manage-cluster-quorum#configure-the-cluster-quorum)  |
|---------|---------|---------|---------|
|**Sistema operativo admitido**| Windows Server 2016+ |All | All|

Para más información sobre el cuórum, consulte la [Introducción al clúster de conmutación por error de Windows Server](hadr-windows-server-failover-cluster-overview.md). 

## <a name="cloud-witness"></a>Testigo en la nube

Se trata de un tipo de testigo de cuórum de clúster de conmutación por error que usa Microsoft Azure Storage para proporcionar un voto en el quórum de clúster. 


En la tabla siguiente encontrará información adicional y consideraciones sobre el testigo en la nube: 

| Tipo de testigo  | Descripción  | Requisitos y recomendaciones  |
| ---------    |---------        |---------                        |
| Testigo en la nube     |  <ul><li> Usa Azure Storage como testigo en la nube, contiene solo la marca de tiempo. </li><li> Resulta ideal para implementaciones en varios sitios, en varias zonas y en varias regiones.</li> <li> Crea un contenedor conocido `msft-cloud-witness` en la cuenta de almacenamiento de Microsoft. </li> <li> Escribe un único archivo de blob con el identificador único del clúster correspondiente, que se usa como nombre de archivo del archivo de blob en el contenedor. </li>      |  <ul><li>El tamaño predeterminado es de 1 MB.</li><li> Use **De uso general** para el tipo de cuenta. No se admite el almacenamiento de blobs. </li><li> Use almacenamiento estándar. Azure Premium Storage no se admite. </li><li> Los clústeres de conmutación por error usan el archivo de blob como punto de arbitraje, lo cual requiere algunas garantías de coherencia al leer los datos. Por tanto, debe seleccionar **Almacenamiento con redundancia local (LRS)** como tipo de **Replicación**.</li><li> Debe excluirse de las copias de seguridad y de los análisis antivirus</li><li> No se admite un testigo de disco con Espacios de almacenamiento directo.</li> <li> El testigo en la nube usa HTTPS (puerto predeterminado 443) para establecer la comunicación con Azure Blob service. Asegúrese de que se puede acceder al puerto HTTPS a través del proxy. </li>|

Al configurar un recurso de cuórum de testigo en la nube para el clúster de conmutación por error, tenga en cuenta lo siguiente:
- En lugar de almacenar la clave de acceso, el clúster de conmutación por error generará y almacenará de forma segura un token de seguridad de acceso compartido (SAS).
- El token de SAS generado es válido siempre que la clave de acceso siga siendo válida. Al rotar la clave de acceso principal, es importante actualizar primero el testigo en la nube (en todos los clústeres que usan esa cuenta de almacenamiento) con la clave de acceso secundaria antes de volver a generar la clave de acceso principal.
- El testigo en la nube usa la interfaz REST HTTPS del servicio de cuentas de Azure Storage. Esto significa que requiere que el puerto HTTPS esté abierto en todos los nodos del clúster.


Un testigo en la nube requiere una cuenta de Azure Storage. Para configurar una cuenta de almacenamiento, siga estos pasos: 

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
2. En el menú del concentrador, seleccione Nuevo -> Datos y almacenamiento -> Cuenta de almacenamiento.
3. En la página Crear una cuenta de almacenamiento, haga lo siguiente:
    1. Escriba un nombre para la cuenta de almacenamiento. Los nombres de las cuentas de almacenamiento deben tener entre 3 y 24 caracteres, y solo pueden incluir números y letras en minúscula. El nombre de la cuenta de almacenamiento también debe ser único dentro de Azure.
    2. En **Tipo de cuenta** seleccione **Uso general**.
    3. En **Rendimiento**, seleccione **Estándar**.
    2. En **Replicación**, seleccione **Almacenamiento con redundancia local (LRS)** .


Una vez creada la cuenta de almacenamiento, siga estos pasos para configurar el recurso de cuórum del testigo en la nube para el clúster de conmutación por error: 


# <a name="powershell"></a>[PowerShell](#tab/powershell)

El actual comando Set-ClusterQuorum de PowerShell tiene nuevos parámetros correspondientes al testigo en la nube.

Puede configurar el testigo en la nube con el cmdlet [`Set-ClusterQuorum`](/powershell/module/failoverclusters/set-clusterquorum) mediante el comando de PowerShell:

```PowerShell
Set-ClusterQuorum -CloudWitness -AccountName <StorageAccountName> -AccessKey <StorageAccountAccessKey>
```

En el caso excepcional de que tenga que usar un punto de conexión diferente, use este comando de PowerShell: 

```PowerShell
Set-ClusterQuorum -CloudWitness -AccountName <StorageAccountName> -AccessKey <StorageAccountAccessKey> -Endpoint <servername>
```

Consulte la [documentación del testigo en la nube](/windows-server/failover-clustering/deploy-cloud-witness) para obtener ayuda para buscar la clave de acceso de la cuenta de almacenamiento. 


# <a name="failover-cluster-manager"></a>[Administrador de clústeres de conmutación por error](#tab/fcm-gui)

Use el Asistente para configuración de cuórum integrado en el Administrador de clústeres de conmutación por error para configurar el testigo en la nube. Para hacerlo, siga estos pasos: 

1. Abre el Administrador de clústeres de conmutación por error.

2. Haga clic con el botón derecho en el clúster -> **Más acciones** -> **Configurar opciones de quórum de clúster**. Esto inicia el asistente para configurar el cuórum de clúster.

    ![Instantánea de la ruta de acceso del menú para Configurar opciones de cuórum de clúster en la interfaz de usuario del Administrador de clústeres de conmutación por error](./media/hadr-create-quorum-windows-failover-cluster-how-to/cloud-witness-7.png)
    
3. En la página **Seleccionar opción de configuración de cuórum**, seleccione **Seleccionar el testigo de cuórum**.

    ![Instantánea del botón de radio "Seleccionar el testigo de cuórum" en el asistente para configurar el cuórum de clúster](./media/hadr-create-quorum-windows-failover-cluster-how-to/cloud-witness-8.png)
   
4. En la página **Seleccionar testigo de cuórum**, seleccione **Configurar un testigo en la nube**.

    ![Instantánea del botón de radio adecuado para seleccionar un testigo en la nube](./media/hadr-create-quorum-windows-failover-cluster-how-to/cloud-witness-9.png)
    
5. En la página **Configurar testigo en la nube**, escriba la información de la cuenta de Azure Storage. Para obtener ayuda para encontrar esta información, consulte la [documentación del testigo en la nube](/windows-server/failover-clustering/deploy-cloud-witness). 
   1. (Parámetro obligatorio) Nombre de la cuenta de Azure Storage.
   2. (Parámetro obligatorio) Clave de acceso correspondiente a la cuenta de almacenamiento.
       1. Al crear por primera vez, use la clave de acceso principal. 
       2. Al rotar la clave de acceso principal, use la clave de acceso secundaria.
   3. (Parámetro opcional) Si piensa usar otro punto de conexión de servicio de Azure (por ejemplo, el servicio Microsoft Azure en China), actualice el nombre del servidor del punto de conexión.

    ![Instantánea del panel de configuración del testigo en la nube en el asistente para configurar el cuórum de clúster](./media/hadr-create-quorum-windows-failover-cluster-how-to/cloud-witness-10.png)
      

6. Tras la configuración correcta del testigo en la nube, puede ver el recurso de testigo recién creado en el complemento Administrador de clústeres de conmutación por error.

    ![Configuración correcta del testigo en la nube](./media/hadr-create-quorum-windows-failover-cluster-how-to/cloud-witness-11.png)
    


---


## <a name="disk-witness"></a>Testigo de disco

Se trata de un pequeño disco agrupado en el grupo de almacenamiento disponible del clúster. Este disco presenta una alta disponibilidad y puede conmutar por error entre nodos. 

El testigo de disco es la opción de cuórum recomendada cuando se usa con una solución de alta disponibilidad de almacenamiento compartido, como la instancia de clúster de conmutación por error con discos compartidos de Azure. 

En la tabla siguiente encontrará información adicional y consideraciones sobre el testigo de disco de cuórum: 

| Tipo de testigo  | Descripción  | Requisitos y recomendaciones  |
| ---------    |---------        |---------                        |
| Testigo de disco     |  <ul><li> LUN dedicado que almacena una copia de la base de datos del clúster</li><li> Recomendado para clústeres con almacenamiento compartido (no replicado)</li>       |  <ul><li>El tamaño del LUN debe ser como mínimo de 512 MB</li><li> Debe ser exclusivo para el clúster y no debe asignarse a un rol en clúster</li><li> Debe incluirse en almacenamiento en clúster y completar sin errores las pruebas de validación de almacenamiento</li><li> No puede ser un disco que sea un volumen compartido de clúster (CSV)</li><li> Disco básico con un solo volumen</li><li> No necesita tener asignada una letra de unidad</li><li> Puede formatearse como NTFS o como ReFS</li><li> De manera opcional, se puede configurar con RAID de hardware para tolerancia a errores</li><li> Debe excluirse de las copias de seguridad y de los análisis antivirus</li><li> No se admite un testigo de disco con Espacios de almacenamiento directo.</li>|

Para usar un disco compartido de Azure para el testigo de disco, primero debe crear el disco y montarlo. Para ello, siga los pasos descritos en la sección [Montaje de disco](failover-cluster-instance-azure-shared-disks-manually-configure.md#add-azure-shared-disk) de la guía de creación de una instancia de clúster de conmutación por error con discos compartidos de Azure. No es necesario que el disco sea prémium. 

Una vez montado el disco, siga los pasos a continuación para agregarlo al almacenamiento del clúster: 

1. Abre el Administrador de clústeres de conmutación por error. 
1. Seleccione **Discos** en **Almacenamiento** en el panel de navegación izquierdo. 
1. Seleccione **Agregar disco** en **Acciones** en el panel de navegación derecho. 
1. Seleccione la unidad compartida de Azure que acaba de montar y anote el nombre, por ejemplo, `Cluster Disk 3`. 

Una vez que haya agregado el disco como almacenamiento en clúster, configúrelo como testigo de disco mediante PowerShell:  


El actual comando Set-ClusterQuorum de PowerShell tiene nuevos parámetros correspondientes al testigo en la nube.

Use la ruta de acceso del recurso compartido de archivos como parámetro para el testigo de disco cuando use el cmdlet [`Set-ClusterQuorum`](/powershell/module/failoverclusters/set-clusterquorum) de PowerShell:

```PowerShell
Set-ClusterQuorum -NodeAndDiskMajority "Cluster Disk 3"
```

También puede usar el Administrador de clústeres de conmutación por error; siga los mismos pasos que para el testigo en la nube, pero elija el testigo de disco como opción de cuórum en su lugar. 


## <a name="file-share-witness"></a>Testigo de recurso compartido de archivos

Se trata de un recurso compartido de archivos SMB que se suele configurar en un servidor de archivos que ejecuta Windows Server. Este testigo mantiene la información de la agrupación en clústeres en un archivo witness.log, pero no almacena una copia de la base de datos del clúster. En Azure, puede configurar un recurso compartido de archivos en una máquina virtual independiente. 

Configure un testigo de recurso compartido de archivos si en el entorno no hay disponible o no se admite un testigo de disco o un testigo en la nube. 

En la tabla siguiente encontrará información adicional y consideraciones sobre el testigo de recurso compartido de archivos de cuórum: 

| Tipo de testigo  | Descripción  | Requisitos y recomendaciones  |
| ---------    |---------        |---------                        |
| Testigo de recurso compartido de archivos     | <ul><li>El recurso compartido de archivos SMB que se configura en un servidor de archivos que ejecute Windows Server</li><li> No almacena una copia de la base de datos del clúster</li><li> Mantiene toda la información del clúster en el archivo witness.log</li><li> Recomendado para clústeres multisitio con almacenamiento replicado </li>       |  <ul><li>Debe disponer de un mínimo de 5 MB de espacio</li><li> Debe ser dedicado para un solo clúster y no usarse para almacenar datos de usuarios o de aplicaciones</li><li> Debe tener habilitados los permisos de escritura para el objeto de equipo del nombre del clúster</li></ul><br>A continuación, encontrarás consideraciones adicionales para un servidor de archivos que hospede el testigo de recurso compartido de archivos:<ul><li>Un solo servidor de archivos se puede configurar con testigos del recurso compartido de archivos para varios clústeres.</li><li> El servidor de archivos debe encontrarse en un sitio que esté separado de la carga de trabajo del clúster. Esto ofrece las mismas oportunidades de supervivencia para cualquier clúster si se pierden las comunicaciones de red de sitio a sitio. Si el servidor de archivos se encuentra en el mismo sitio, dicho sitio se convertirá en el sitio principal y será el único sitio que podrá conectarse al recurso compartido de archivos.</li><li> El servidor de archivos se puede ejecutar en una máquina virtual si esta no está hospedada en el mismo clúster que usa el testigo de recurso compartido de archivos.</li><li> Para obtener una alta disponibilidad, el servidor de archivos se puede configurar en un clúster de conmutación por error separado. </li>      |

Una vez que haya creado el recurso compartido de archivos y configurado correctamente los permisos, monte el recurso compartido de archivos en los nodos en clúster. Puede seguir los mismos pasos generales para montar el recurso compartido de archivos, tal y como se describe en la sección [Montaje del recurso compartido de archivos](failover-cluster-instance-premium-file-share-manually-configure.md) de la guía paso a paso de creación de una instancia de clúster de conmutación por error con un recurso compartido de archivos prémium. 

Una vez que el recurso compartido de archivos se haya configurado y montado correctamente, use PowerShell para agregar el recurso compartido de archivos como recurso del testigo de cuórum: 

```powershell
Set-ClusterQuorum -FileShareWitness <UNC path to file share> -Credential $(Get-Credential)
```

Se le pedirá una cuenta y una contraseña para una cuenta local (para el recurso compartido de archivos) que no sea de administrador, y que tenga derechos de administrador completos para el recurso compartido.  El clúster mantendrá el nombre y la contraseña cifrados, y nadie podrá acceder a ellos.

También puede usar el Administrador de clústeres de conmutación por error; siga los mismos pasos que para el testigo en la nube, pero elija el testigo de recurso compartido de archivos como opción de cuórum en su lugar. 

## <a name="change-quorum-voting"></a>Cambio de la votación de cuórum


Es posible cambiar el voto de cuórum de un nodo que participa en un clúster de conmutación por error de Windows Server. 

Al modificar la configuración de voto de nodo, siga estas instrucciones: 

| Guía de votación de cuórum |
|-|
| Comience con los nodos que no tengan ningún voto de manera predeterminada. Cada nodo solo debe tener un voto con justificación explícita.|
| Habilite los votos para los nodos de clúster que hospedan la réplica principal de un grupo de disponibilidad o los propietarios preferidos de una instancia de clúster de conmutación por error. |
| Habilite los votos para los propietarios de conmutación automática por error. Cada uno de los nodos que puede hospedar una réplica principal o FCI, como resultado de una conmutación automática por error, debe tener un voto. | 
| Si un grupo de disponibilidad tiene más de una réplica secundaria, habilite votos solo para las réplicas que tienen conmutación automática por error. | 
| Deshabilite los votos para los nodos que se encuentren en sitios secundarios de recuperación ante desastres. Los nodos de los sitios secundarios no deben contribuir a la decisión de desconectar un clúster si no hay ningún problema con el sitio primario. | 
| Tenga un número impar de votos, con un mínimo de tres votos de quórum. Si es necesario, agregue un [testigo de cuórum](hadr-cluster-quorum-configure-how-to.md) para un voto adicional cuando un clúster tenga dos nodos. | 
| Vuelva a valorar las asignaciones de votos después de la conmutación por error. No es conveniente realizar la conmutación por error en una configuración de clúster que no admita un quórum correcto. |




## <a name="next-steps"></a>Pasos siguientes

Para obtener más información, consulte:

- [Configuración de alta disponibilidad y recuperación ante desastres para SQL Server en máquinas virtuales de Azure](hadr-cluster-best-practices.md)
- [Clúster de conmutación por error de Windows Server con SQL Server en máquinas virtuales de Azure](hadr-windows-server-failover-cluster-overview.md)
- [Grupos de disponibilidad Always On con SQL Server en máquinas virtuales de Azure](availability-group-overview.md)
- [Clúster de conmutación por error de Windows Server con SQL Server en máquinas virtuales de Azure](hadr-windows-server-failover-cluster-overview.md)
- [Instancias de clúster de conmutación por error con SQL Server en máquinas virtuales de Azure](failover-cluster-instance-overview.md)
- [Información general de las instancias de clúster de conmutación por error](/sql/sql-server/failover-clusters/windows/always-on-failover-cluster-instances-sql-server)
