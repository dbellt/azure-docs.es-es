---
title: Controlador de volumen de Azure Files para Service Fabric
description: Service Fabric admite el uso de Azure Files para realizar copias de seguridad de los volúmenes desde su contenedor.
ms.topic: conceptual
ms.date: 6/10/2018
ms.openlocfilehash: d3acb20723bc826a120a8333c0ef63c33131376a
ms.sourcegitcommit: f9e368733d7fca2877d9013ae73a8a63911cb88f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111901212"
---
# <a name="azure-files-volume-driver-for-service-fabric"></a>Controlador de volumen de Azure Files para Service Fabric

El controlador de volúmenes de Azure Files es un [complemento de volúmenes de Docker](https://docs.docker.com/engine/extend/plugins_volume/) que proporciona volúmenes basados en [Azure Files](../storage/files/storage-files-introduction.md) para los contenedores de Docker. Se empaqueta como una aplicación de Service Fabric que se puede implementar en un clúster de Service Fabric para proporcionar volúmenes para otras aplicaciones contenedoras de Service Fabric dentro del clúster.

> [!NOTE]
> La versión 6.5.661.9590 del complemento de volumen de Azure Files se ha publicado para la disponibilidad general.
>

## <a name="prerequisites"></a>Prerrequisitos
* La versión de Windows del complemento de volúmenes de Azure Files funciona en [Windows Server versión 1709](/windows-server/get-started/whats-new-in-windows-server-1709), [Windows 10 versión 1709](/windows/whats-new/whats-new-windows-10-version-1709) o sistemas operativos posteriores únicamente.

* La versión de Linux del complemento de volúmenes de Azure Files funciona en todas las versiones de sistemas operativos admitidas por Service Fabric.

* El complemento de volúmenes de Azure Files solo funciona en Service Fabric versión 6.2 y versiones más recientes.

* Siga las instrucciones en la [documentación de Azure Files](../storage/files/storage-how-to-create-file-share.md) para crear un recurso compartido de archivos para la aplicación de contenedor de Service Fabric que se usará como volumen.

* Necesitará que [Powershell con el módulo de Service Fabric](./service-fabric-get-started.md) o [SFCTL](./service-fabric-cli.md) esté instalado.

* Si usa contenedores de Hyper-V, hay que agregar los fragmentos de código siguientes en la sección ClusterManifest (clúster local) o fabricSettings de la plantilla de Azure Resource Manager (clúster de Azure) o ClusterConfig.json (clúster independiente).

En ClusterManifest, hay que agregar el siguiente código en la sección de hospedaje. En este ejemplo, el nombre del volumen es **sfazurefile** y el puerto que escucha en el clúster es **19100**. Reemplácelos por los valores correctos para el clúster.

``` xml 
<Section Name="Hosting">
  <Parameter Name="VolumePluginPorts" Value="sfazurefile:19100" />
</Section>
```

En la sección fabricSettings de la plantilla Azure Resource Manager (para las implementaciones de Azure) o ClusterConfig.json (para las implementaciones independientes), debe agregarse el siguiente fragmento de código. De nuevo, reemplace el nombre del volumen y los valores del puerto por los suyos.

```json
"fabricSettings": [
  {
    "name": "Hosting",
    "parameters": [
      {
          "name": "VolumePluginPorts",
          "value": "sfazurefile:19100"
      }
    ]
  }
]
```

## <a name="deploy-a-sample-application-using-service-fabric-azure-files-volume-driver"></a>Implementación de una aplicación de ejemplo mediante el controlador de volúmenes de Azure Files en Service Fabric

### <a name="using-azure-resource-manager-via-the-provided-powershell-script-recommended"></a>Uso de Azure Resource Manager mediante el script de PowerShell proporcionado (recomendado)

Si el clúster se basa en Azure, le recomendamos que implemente aplicaciones en este mediante el modelo de recursos de la aplicación de Azure Resource Manager a fin de simplificar su uso y contribuir a avanzar hacia el modelo de mantenimiento de la infraestructura como código. Este enfoque elimina la necesidad de realizar un seguimiento de la versión de la aplicación para el controlador de volumen de Azure Files. También permite mantener las plantillas de Azure Resource Manager separadas para cada sistema operativo admitido. El script da por supuesto que implementa la versión más reciente de la aplicación de Azure Files y toma parámetros del tipo de sistema operativo, el identificador de la suscripción del clúster y el grupo de recursos. Puede descargar el script desde el [sitio de descarga de Service Fabric](https://sfazfilevd.blob.core.windows.net/sfazfilevd/DeployAzureFilesVolumeDriver.zip). Tenga en cuenta que esto establece automáticamente el valor de ListenPort, que es el puerto en el que el complemento de volúmenes de Azure Files escucha las solicitudes del demonio de Docker a 19100. Para cambiarlo, puede agregar el parámetro denominado "listenPort". Asegúrese de que el puerto no entre en conflicto con ningún otro puerto que usen el clúster o las aplicaciones.
 

Comando de implementación de Azure Resource Manager para Windows:
```powershell
.\DeployAzureFilesVolumeDriver.ps1 -subscriptionId [subscriptionId] -resourceGroupName [resourceGroupName] -clusterName [clusterName] -windows
```

Comando de implementación de Azure Resource Manager para Linux:
```powershell
.\DeployAzureFilesVolumeDriver.ps1 -subscriptionId [subscriptionId] -resourceGroupName [resourceGroupName] -clusterName [clusterName] -linux
```

Una vez que haya ejecutado correctamente el script, puede ir directamente a la [sección de configuración de la aplicación.](#configure-your-applications-to-use-the-volume)


### <a name="manual-deployment-for-standalone-clusters"></a>Implementación manual de clústeres independientes

La aplicación de Service Fabric que proporciona los volúmenes para los contenedores puede descargarse desde el [sitio de descarga de Service Fabric](https://sfazfilevd.blob.core.windows.net/sfazfilevd/AzureFilesVolumePlugin.6.5.661.9590.zip). La aplicación puede implementarse en el clúster mediante [API de PowerShell](./service-fabric-deploy-remove-applications.md), [CLI](./service-fabric-application-lifecycle-sfctl.md) o [FabricClient](./service-fabric-deploy-remove-applications-fabricclient.md).

1. Mediante la línea de comandos, cambie el directorio al directorio raíz del paquete de aplicación descargado.

    ```powershell
    cd .\AzureFilesVolume\
    ```

    ```bash
    cd ~/AzureFilesVolume
    ```

2. A continuación, copie el paquete de aplicación en el almacén de imágenes con los valores adecuados para [ApplicationPackagePath] y [ImageStoreConnectionString]:

    ```powershell
    Copy-ServiceFabricApplicationPackage -ApplicationPackagePath [ApplicationPackagePath] -ImageStoreConnectionString [ImageStoreConnectionString] -ApplicationPackagePathInImageStore AzureFilesVolumePlugin
    ```

    ```bash
    sfctl cluster select --endpoint https://testcluster.westus.cloudapp.azure.com:19080 --pem test.pem --no-verify
    sfctl application upload --path [ApplicationPackagePath] --show-progress
    ```

3. Cargar el tipo de aplicación

    ```powershell
    Register-ServiceFabricApplicationType -ApplicationPathInImageStore AzureFilesVolumePlugin
    ```

    ```bash
    sfctl application provision --application-type-build-path [ApplicationPackagePath]
    ```

4. Cree la aplicación y preste mucha atención al valor del parámetro de la aplicación **ListenPort**. Este valor es el puerto en el que el complemento de volúmenes de Azure Files escucha las solicitudes del demonio de Docker. Asegúrese de que el puerto que se proporciona a la aplicación coincida con el valor de VolumePluginPorts en ClusterManifest y no entre en conflicto con ningún otro puerto que usen el clúster o las aplicaciones.

    ```powershell
    New-ServiceFabricApplication -ApplicationName fabric:/AzureFilesVolumePluginApp -ApplicationTypeName AzureFilesVolumePluginType -ApplicationTypeVersion 6.5.661.9590   -ApplicationParameter @{ListenPort='19100'}
    ```

    ```bash
    sfctl application create --app-name fabric:/AzureFilesVolumePluginApp --app-type AzureFilesVolumePluginType --app-version 6.5.661.9590  --parameter '{"ListenPort":"19100"}'
    ```

> [!NOTE]
> 
> Windows Server 2016 Datacenter no admite montajes de SMB de asignación en contenedores ([solo se admiten en Windows Server versión 1709](/virtualization/windowscontainers/manage-containers/container-storage)). Esta restricción evita que haya asignaciones de volumen de red y controladores de volúmenes de Azure Files en versiones anteriores a 1709.

#### <a name="deploy-the-application-on-a-local-development-cluster"></a>Implementación de la aplicación en un clúster de desarrollo local
Siga los pasos de 1 a 3 [anteriores.](#manual-deployment-for-standalone-clusters)

 El número de instancias de servicio predeterminado para la aplicación del complemento de volúmenes de Azure Files es -1; es decir, hay una instancia del servicio implementada en cada nodo del clúster. Sin embargo, al implementar la aplicación del complemento de volúmenes de Azure Files en un clúster de desarrollo local, el recuento de instancias de servicio debe especificarse como 1. Esto puede hacerse a través del parámetro de aplicación **InstanceCount**. Por lo tanto, el comando para crear la aplicación del complemento de volúmenes de Azure Files en un clúster de desarrollo local es:

```powershell
New-ServiceFabricApplication -ApplicationName fabric:/AzureFilesVolumePluginApp -ApplicationTypeName AzureFilesVolumePluginType -ApplicationTypeVersion 6.5.661.9590  -ApplicationParameter @{ListenPort='19100';InstanceCount='1'}
```

```bash
sfctl application create --app-name fabric:/AzureFilesVolumePluginApp --app-type AzureFilesVolumePluginType --app-version 6.5.661.9590  --parameter '{"ListenPort": "19100","InstanceCount": "1"}'
```

## <a name="configure-your-applications-to-use-the-volume"></a>Configuración de las aplicaciones para usar el volumen
En el siguiente fragmento de código se muestra cómo se puede especificar un volumen basado en Azure Files en el archivo de manifiesto de aplicación de la aplicación. El elemento específico de interés es la etiqueta **Volume**:

```xml
?xml version="1.0" encoding="UTF-8"?>
<ApplicationManifest ApplicationTypeName="WinNodeJsApp" ApplicationTypeVersion="1.0" xmlns="http://schemas.microsoft.com/2011/01/fabric" xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance">
    <Description>Calculator Application</Description>
    <Parameters>
      <Parameter Name="ServiceInstanceCount" DefaultValue="3"></Parameter>
      <Parameter Name="MyCpuShares" DefaultValue="3"></Parameter>
      <Parameter Name="MyStorageVar" DefaultValue="c:\tmp"></Parameter>
    </Parameters>
    <ServiceManifestImport>
        <ServiceManifestRef ServiceManifestName="NodeServicePackage" ServiceManifestVersion="1.0"/>
     <Policies>
       <ContainerHostPolicies CodePackageRef="NodeService.Code" Isolation="hyperv">
            <PortBinding ContainerPort="8905" EndpointRef="Endpoint1"/>
            <RepositoryCredentials PasswordEncrypted="false" Password="****" AccountName="test"/>
            <Volume Source="azfiles" Destination="c:\VolumeTest\Data" Driver="sfazurefile">
                <DriverOption Name="shareName" Value="" />
                <DriverOption Name="storageAccountName" Value="" />
                <DriverOption Name="storageAccountKey" Value="" />
                <DriverOption Name="storageAccountFQDN" Value="" />
            </Volume>
       </ContainerHostPolicies>
   </Policies>
    </ServiceManifestImport>
    <ServiceTemplates>
        <StatelessService ServiceTypeName="StatelessNodeService" InstanceCount="5">
            <SingletonPartition></SingletonPartition>
        </StatelessService>
    </ServiceTemplates>
</ApplicationManifest>
```

El nombre de controlador para el complemento de volúmenes de Azure Files es **sfazurefile**. Este valor se establece para el atributo **Driver** del elemento de la etiqueta **Volume** en el manifiesto de aplicación.

En la etiqueta **Volume** del fragmento de código anterior, el complemento de volúmenes de Azure Files requiere los siguientes atributos:
- **Source**: se trata del nombre del volumen. El usuario puede elegir cualquier nombre para su volumen.
- **Destination**: este atributo es la ubicación a la que se asigna el volumen dentro del contenedor en ejecución. Por lo tanto, el destino no puede ser una ubicación que ya exista dentro del contenedor.

Como se muestra en los elementos **DriverOption** en el fragmento de código anterior, el complemento de volúmenes de Azure Files admite las siguientes opciones de controlador:
- **shareName**: nombre del recurso compartido de archivos de Azure Files que proporciona el volumen para el contenedor.
- **storageAccountName**: nombre de la cuenta de almacenamiento de Azure que contiene el recurso compartido de archivos de Azure Files.
- **storageAccountKey**: clave de acceso de la cuenta de almacenamiento de Azure que contiene el recurso compartido de archivos de Azure Files.
- **storageAccountFQDN**: nombre de dominio asociado con la cuenta de almacenamiento. Si no se especifica storageAccountFQDN, el nombre de dominio se formará utilizando el sufijo predeterminado (.file.core.windows.net) con storageAccountName.  

    ```xml
    - Example1: 
        <DriverOption Name="shareName" Value="myshare1" />
        <DriverOption Name="storageAccountName" Value="myaccount1" />
        <DriverOption Name="storageAccountKey" Value="mykey1" />
        <!-- storageAccountFQDN will be "myaccount1.file.core.windows.net" -->
    - Example2: 
        <DriverOption Name="shareName" Value="myshare2" />
        <DriverOption Name="storageAccountName" Value="myaccount2" />
        <DriverOption Name="storageAccountKey" Value="mykey2" />
        <DriverOption Name="storageAccountFQDN" Value="myaccount2.file.core.chinacloudapi.cn" />
    ```

## <a name="using-your-own-volume-or-logging-driver"></a>Uso de su propio controlador de volumen o registro
Service Fabric también permite el uso de sus propios controladores de [volumen](https://docs.docker.com/engine/extend/plugins_volume/) o de [registro](https://docs.docker.com/engine/admin/logging/overview/) personalizados. Si el controlador de volumen/registro de Docker no está instalado en el clúster, puede instalarlo manualmente mediante los protocolos RDP/SSH. Puede realizar la instalación con estos protocolos mediante un [script de inicio de conjunto de escalado de máquinas virtuales](https://azure.microsoft.com/resources/templates/vmss-custom-script-windows/) o un [script SetupEntryPoint](./service-fabric-application-model.md).

A continuación se muestra un ejemplo del script para instalar el [controlador de volumen de Docker para Azure](https://docs.docker.com/docker-for-azure/persistent-data-volumes/):

```bash
docker plugin install --alias azure --grant-all-permissions docker4x/cloudstor:17.09.0-ce-azure1  \
    CLOUD_PLATFORM=AZURE \
    AZURE_STORAGE_ACCOUNT="[MY-STORAGE-ACCOUNT-NAME]" \
    AZURE_STORAGE_ACCOUNT_KEY="[MY-STORAGE-ACCOUNT-KEY]" \
    DEBUG=1
```

En las aplicaciones, para usar el controlador de volumen o de registro que ha instalado, deberá especificar los valores apropiados en los elementos **Volume** y **LogConfig** en  **ContainerHostPolicies** en el manifiesto de aplicación.

```xml
<ContainerHostPolicies CodePackageRef="NodeService.Code" Isolation="hyperv">
    <PortBinding ContainerPort="8905" EndpointRef="Endpoint1"/>
    <RepositoryCredentials PasswordEncrypted="false" Password="****" AccountName="test"/>
    <LogConfig Driver="[YOUR_LOG_DRIVER]" >
        <DriverOption Name="test" Value="vale"/>
    </LogConfig>
    <Volume Source="c:\workspace" Destination="c:\testmountlocation1" IsReadOnly="false"></Volume>
    <Volume Source="[MyStorageVar]" Destination="c:\testmountlocation2" IsReadOnly="true"> </Volume>
    <Volume Source="myvolume1" Destination="c:\testmountlocation2" Driver="[YOUR_VOLUME_DRIVER]" IsReadOnly="true">
        <DriverOption Name="[name]" Value="[value]"/>
    </Volume>
</ContainerHostPolicies>
```

Al especificar un complemento de volumen, Service Fabric crea automáticamente el volumen con los parámetros especificados. La etiqueta **Source** del elemento **Volume** es el nombre del volumen; la etiqueta **Driver** especifica el complemento de controlador de volumen. La etiqueta **Destino** es la ubicación a la que se asigna el **origen** dentro del contenedor en ejecución. Por lo tanto, el destino no puede ser una ubicación que ya exista dentro del contenedor. Se pueden especificar opciones mediante la etiqueta **DriverOption** de la manera siguiente:

```xml
<Volume Source="myvolume1" Destination="c:\testmountlocation4" Driver="azure" IsReadOnly="true">
           <DriverOption Name="share" Value="models"/>
</Volume>
```

Se admiten parámetros de la aplicación para volúmenes, tal y como se muestra en el fragmento de código de manifiesto anterior (busque `MyStorageVar` para ver un ejemplo de uso).

Si se especifica un controlador de registro de Docker, debe implementar agentes (o contenedores) para administrar los registros en el clúster. La etiqueta **DriverOption** se puede usar para especificar opciones para el controlador de registro.

## <a name="next-steps"></a>Pasos siguientes
* Para ver ejemplos de contenedor, incluido el controlador de volumen, visite los [ejemplos de contenedor de Service Fabric](https://github.com/Azure-Samples/service-fabric-containers)
* Para implementar contenedores en un clúster de Service Fabric, consulte el artículo [Cree la primera aplicación contenedora en Service Fabric en Windows](./service-fabric-get-started-containers.md).
