---
title: Implementación de una aplicación .NET de un contenedor en Azure Service Fabric
description: Aprenda a incluir una aplicación .NET existente en un contenedor mediante Visual Studio y a depurar contenedores en Service Fabric localmente. La aplicación incluida en el contenedor se inserta en un registro de contenedor de Azure y se implementa en un clúster de Service Fabric. Cuando se implementa en Azure, la aplicación utiliza Azure SQL DB para conservar los datos.
ms.topic: tutorial
ms.date: 07/08/2019
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: b0e75d4054bc814c3e826db4519f9e882964ef4e
ms.sourcegitcommit: df574710c692ba21b0467e3efeff9415d336a7e1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/28/2021
ms.locfileid: "110677377"
---
# <a name="tutorial-deploy-a-net-application-in-a-windows-container-to-azure-service-fabric"></a>Tutorial: Implementación de una aplicación .NET de un contenedor de Windows en Azure Service Fabric

En este tutorial se muestra cómo incluir una aplicación ASP.NET existente en un contenedor y empaquetarla como una aplicación de Service Fabric.  Ejecute los contenedores localmente en el clúster de desarrollo de Service Fabric y, a continuación, implemente la aplicación en Azure.  La aplicación conserva los datos en [Azure SQL Database](../azure-sql/database/sql-database-paas-overview.md).

En este tutorial, aprenderá a:

> [!div class="checklist"]
>
> * Inclusión de una aplicación existente en un contenedor con Visual Studio
> * Creación de una base de datos de Azure SQL Database
> * Creación de un registro de contenedor de Azure
> * Implementación de una aplicación de Service Fabric en Azure

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="prerequisites"></a>Requisitos previos

1. Si no tiene una suscripción a Azure, [cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
2. Habilite las características **Hyper-V** y **Contenedores** de Windows.
3. Instale [Docker Desktop for Windows](https://store.docker.com/editions/community/docker-ce-desktop-windows?tab=description) para ejecutar contenedores en Windows 10.
4. Instale [la versión en tiempo de ejecución de Service Fabric 6.2 o posterior](service-fabric-get-started.md) y la [versión 3.1 del SDK de Service Fabric](service-fabric-get-started.md) o posterior.
5. Instale [Visual Studio 2019 versión 16.1](https://www.visualstudio.com/) o posterior con las cargas de trabajo de **desarrollo de Azure** y de **desarrollo de ASP.NET y web**.
6. Instale [Azure PowerShell][link-azure-powershell-install].

## <a name="download-and-run-fabrikam-fiber-callcenter"></a>Descarga y ejecución de Fabrikam Fiber CallCenter

1. Descargue la aplicación de ejemplo [Fabrikam Fiber CallCenter][link-fabrikam-github].  Haga clic en el vínculo para **descargar archivo**.  Desde el directorio *sourceCode* del archivo *fabrikam.zip*, extraiga el archivo *sourceCode.zip* y, a continuación, extraiga el directorio *VS2015* en el equipo.

2. Compruebe que la aplicación Fabrikam Fiber CallCenter se compila y se ejecuta sin error.  Inicie Visual Studio como **administrador** y abra el archivo [FabrikamFiber.CallCenter.sln][link-fabrikam-github].  Presione F5 para depurar y ejecutar la aplicación.

   ![Captura de pantalla de la página principal de la aplicación Fabrikam Fiber CallCenter que se ejecuta en el host local. La página muestra un panel con una lista de llamadas de soporte técnico.][fabrikam-web-page]

## <a name="create-an-azure-sql-db"></a>Creación de una instancia de Azure SQL DB

Cuando se ejecuta la aplicación de Fabrikam Fiber CallCenter en producción, los datos deben conservarse en una base de datos. Actualmente no hay ninguna forma de garantizar la persistencia de datos en un contenedor; por lo tanto, no se pueden almacenar datos de producción de SQL Server en un contenedor.

Recomendamos [Azure SQL Database](../azure-sql/database/powershell-script-content-guide.md). Para configurar y ejecutar una base de datos de SQL Server en Azure, ejecute el script siguiente.  Modifique las variables del script según sea necesario. *clientIP* es la dirección IP del equipo de desarrollo. Tome nota del nombre del servidor que genere el script.

```powershell
$subscriptionID="<subscription ID>"

# Sign in to your Azure account and select your subscription.
Login-AzAccount -SubscriptionId $subscriptionID

# The data center and resource name for your resources.
$dbresourcegroupname = "fabrikam-fiber-db-group"
$location = "southcentralus"

# The server name: Use a random value or replace with your own value (do not capitalize).
$servername = "fab-fiber-$(Get-Random)"

# Set an admin login and password for your database.
# The login information for the server.
$adminlogin = "ServerAdmin"
$password = "Password@123"

# The IP address of your development computer that accesses the SQL DB.
$clientIP = "<client IP>"

# The database name.
$databasename = "call-center-db"

# Create a new resource group for your deployment and give it a name and a location.
New-AzResourceGroup -Name $dbresourcegroupname -Location $location

# Create the SQL server.
New-AzSqlServer -ResourceGroupName $dbresourcegroupname `
    -ServerName $servername `
    -Location $location `
    -SqlAdministratorCredentials $(New-Object -TypeName System.Management.Automation.PSCredential -ArgumentList $adminlogin, $(ConvertTo-SecureString -String $password -AsPlainText -Force))

# Create the firewall rule to allow your development computer to access the server.
New-AzSqlServerFirewallRule -ResourceGroupName $dbresourcegroupname `
    -ServerName $servername `
    -FirewallRuleName "AllowClient" -StartIpAddress $clientIP -EndIpAddress $clientIP

# Create the database in the server.
New-AzSqlDatabase  -ResourceGroupName $dbresourcegroupname `
    -ServerName $servername `
    -DatabaseName $databasename `
    -RequestedServiceObjectiveName "S0"

Write-Host "Server name is $servername"
```

> [!TIP]
> Si está detrás de un firewall corporativo, la dirección IP del equipo de desarrollo puede no ser una dirección IP expuesta a Internet. Para comprobar que la base de datos tiene la dirección IP correcta para la regla de firewall, vaya a [Azure Portal](https://portal.azure.com) y busque la base de datos en la sección de bases de datos SQL. Haga clic en su nombre y, a continuación, en la sección de información general, haga clic en "Establecer el firewall del servidor". La "dirección IP del cliente" es la dirección IP de la máquina de desarrollo. Asegúrese de que coincide con la dirección IP en la regla "AllowClient".

## <a name="update-the-web-config"></a>Actualización de web config

En el proyecto **FabrikamFiber.Web**, actualice la cadena de conexión en el archivo **web.config** para que apunte a la instancia de SQL Server del contenedor.  Actualice la parte *Server* de la cadena de conexión con el nombre del servidor creado por el script anterior. Debe ser algo parecido a "fab-fiber-751718376.database.windows.net". En el siguiente XML, es preciso actualizar solo el `connectionString` atributo; los atributos `providerName` y `name` no necesitan cambiarse.

```xml
<add name="FabrikamFiber-Express" connectionString="Server=<server name>,1433;Initial Catalog=call-center-db;Persist Security Info=False;User ID=ServerAdmin;Password=Password@123;MultipleActiveResultSets=False;Encrypt=True;TrustServerCertificate=False;Connection Timeout=30;" providerName="System.Data.SqlClient" />
<add name="FabrikamFiber-DataWarehouse" connectionString="Server=<server name>,1433;Initial Catalog=call-center-db;Persist Security Info=False;User ID=ServerAdmin;Password=Password@123;MultipleActiveResultSets=False;Encrypt=True;TrustServerCertificate=False;Connection Timeout=30;" providerName="System.Data.SqlClient" />
  
```

>[!NOTE]
>Puede utilizar cualquier servidor SQL Server que prefiera para la depuración local, siempre que sea accesible desde el host. Sin embargo, **localdb** no admite la comunicación `container -> host`. Si desea utilizar una base de datos SQL distinta para compilar una versión de compilación de la aplicación web, agregue otra cadena de conexión al archivo *web.release.config*.

## <a name="containerize-the-application"></a>Incluir la aplicación en contenedores

1. Haga clic con el botón derecho en el proyecto **FabrikamFiber.Web** > **Agregar** > **Compatibilidad con el orquestador de contenedores**.  Seleccione **Service Fabric** como orquestador de contenedores y haga clic en **Aceptar**.

2. Si se le solicita, haga clic en **Sí** para conmutar Docker a los contenedores Windows ahora.

   Se crea un nuevo proyecto de aplicación de Service Fabric **FabrikamFiber.CallCenterApplication** en la solución.  Se agrega un archivo Dockerfile al proyecto **FabrikamFiber.Web** existente.  También se agrega un directorio **PackageRoot** al proyecto **FabrikamFiber.Web**, que contiene el manifiesto del servicio y la configuración para el nuevo servicio de FabrikamFiber.Web.

   El contenedor ahora está listo para compilarse y empaquetarse en una aplicación de Service Fabric. Cuando la imagen de contenedor esté en la máquina, puede insertarla en cualquier registro de contenedor y arrastrarla a cualquier host para que se ejecute.

## <a name="run-the-containerized-application-locally"></a>Ejecución de la aplicación en contenedores localmente

Presione **F5** para ejecutar y depurar la aplicación en un contenedor en el clúster de desarrollo local de Service Fabric. Haga clic en **Sí** si ve un cuadro de mensaje que pregunta si desea conceder al grupo "ServiceFabricAllowedUsers" permisos de lectura y ejecución en el directorio del proyecto de Visual Studio.

Si la ejecución de F5 genera una excepción como la siguiente, la dirección IP correcta no se habrá agregado al firewall de base de datos de Azure.

```text
System.Data.SqlClient.SqlException
HResult=0x80131904
Message=Cannot open server 'fab-fiber-751718376' requested by the login. Client with IP address '123.456.789.012' is not allowed to access the server.  To enable access, use the Windows Azure Management Portal or run sp_set_firewall_rule on the master database to create a firewall rule for this IP address or address range.  It may take up to five minutes for this change to take effect.
Source=.Net SqlClient Data Provider
StackTrace:
<Cannot evaluate the exception stack trace>
```

Para agregar la dirección IP adecuada al firewall de base de datos de Azure, ejecute el siguiente comando.

```powershell
# The IP address of your development computer that accesses the SQL DB.
$clientIPNew = "<client IP from the Error Message>"

# Create the firewall rule to allow your development computer to access the server.
New-AzSqlServerFirewallRule -ResourceGroupName $dbresourcegroupname `
    -ServerName $servername `
    -FirewallRuleName "AllowClientNew" -StartIpAddress $clientIPNew -EndIpAddress $clientIPNew
```

## <a name="create-a-container-registry"></a>Creación de un Registro de contenedor

Ahora que la aplicación se ejecuta localmente, empiece a preparar la implementación en Azure.  Las imágenes de contenedor deben estar almacenadas en un registro de contenedor.  Cree una instancia de [Azure Container Registry](../container-registry/container-registry-intro.md) mediante el script siguiente. El nombre del registro de contenedor es visible para otras suscripciones de Azure, por lo que debe ser único.
Antes de implementar la aplicación en Azure, inserte la imagen del contenedor en este registro.  Cuando la aplicación se implementa en el clúster de Azure, la imagen del contenedor se extrae de este registro.

```powershell
# Variables
$acrresourcegroupname = "fabrikam-acr-group"
$location = "southcentralus"
$registryname="fabrikamregistry$(Get-Random)"

New-AzResourceGroup -Name $acrresourcegroupname -Location $location

$registry = New-AzContainerRegistry -ResourceGroupName $acrresourcegroupname -Name $registryname -EnableAdminUser -Sku Basic
```

## <a name="create-a-service-fabric-cluster-on-azure"></a>Creación de un clúster de Service Fabric en Azure

Las aplicaciones de Service Fabric se ejecutan en un clúster o en un conjunto de máquinas físicas o virtuales conectadas a la red.  Antes de implementar la aplicación en Azure, cree un clúster de Service Fabric en Azure.

Puede:

* Crear un clúster de prueba desde Visual Studio. Esta opción permite crear un clúster seguro directamente desde Visual Studio con las configuraciones preferidas.
* [Crear un clúster seguro a partir de una plantilla](service-fabric-tutorial-create-vnet-and-windows-cluster.md)

Este tutorial crea un clúster desde Visual Studio, lo cual es ideal para escenarios de prueba. Si crea un clúster de alguna otra manera o utiliza un clúster existente, puede copiar y pegar el punto de conexión o seleccionarlo de la suscripción.

Antes de comenzar, abra FabrikamFiber.Web -> PackageRoot -> ServiceManifest.xml en el Explorador de soluciones. Tome nota del puerto del front-end web enumerado en **Punto de conexión**.

Al crear el clúster:

1. Haga clic con el botón derecho en el proyecto de aplicación **FabrikamFiber.CallCenterApplication** en el Explorador de soluciones y elija **Publicar**.
2. Inicie sesión con su cuenta de Azure para poder acceder a sus suscripciones.
3. Debajo de la lista desplegable **Punto de conexión**, seleccione la opción **Crear nuevo clúster...** .
4. En el cuadro de diálogo **Crear clúster**, modifique los siguientes valores:

    a. Especifique el nombre del clúster en el campo **Nombre del clúster**, así como la suscripción y la ubicación que desee utilizar. Tome nota del nombre del grupo de recursos del clúster.

    b. Opcional: puede modificar el número de nodos. De forma predeterminada tiene tres nodos, lo mínimo necesario para probar escenarios de Service Fabric.

    c. Seleccione la pestaña **Certificado**. En ella, escriba la contraseña que se utilizará para proteger el certificado del clúster. Este certificado le ayuda a proteger el clúster. También puede modificar la ruta de acceso a la ubicación en que desea guardar el certificado. Visual Studio también puede importar el certificado automáticamente, ya que este es un paso necesario para publicar la aplicación en el clúster.

    >[!NOTE]
    >Tome nota de la ruta de acceso de la carpeta a la que se ha importado este certificado. El siguiente paso después de crear el clúster es importar el certificado.

    d. Seleccione la pestaña **Detalle de máquina virtual**. Especifique la contraseña que desea usar para las máquinas virtuales (VM) que componen el clúster. El nombre de usuario y la contraseña se pueden utilizar para conectarse a las máquinas virtuales de forma remota. También debe seleccionar el tamaño de la máquina virtual y puede cambiar la imagen de la misma si fuera necesario.

    > [!IMPORTANT]
    > Elija una SKU que admita contenedores en ejecución. El sistema operativo Windows Server de los nodos del clúster debe ser compatible con el sistema operativo Windows Server del contenedor. Para más información, consulte [Sistema operativo del contenedor Windows Server y compatibilidad con el sistema operativo del host](service-fabric-get-started-containers.md#windows-server-container-os-and-host-os-compatibility). De forma predeterminada, este tutorial crea una imagen de Docker basada en Windows Server 2016 LTSC. Los contenedores basados en esta imagen se ejecutarán en clústeres creados con Windows Server 2016 Datacenter con contenedores. Sin embargo, si crea un clúster o usa uno existente basado en una versión diferente de Windows Server, debe cambiar la imagen del sistema operativo en la que se basa el contenedor. Abra el archivo **dockerfile** en el proyecto **FabrikamFiber.Web**, convierta en comentario cualquiera de las instrucciones `FROM` existentes basadas en una versión anterior de Windows Server y agregue una instrucción `FROM` basada en la etiqueta de la versión deseada desde la [página DockerHub de Windows Server Core](https://hub.docker.com/_/microsoft-windows-servercore). Para más información sobre las versiones de Windows Server Core, las escalas de tiempo de compatibilidad y el control de versiones, consulte la [información de versión de Windows Server Core](/windows-server/get-started/windows-server-release-info). 

    e. En la pestaña **Opciones avanzadas**, aparece el puerto de la aplicación que se abrirá en el equilibrador de carga cuando se implemente el clúster. Este es el puerto que anotó antes de comenzar a crear el clúster. También puede agregar una clave de Application Insights existente que se va a utilizar para enrutar los archivos de registro de aplicaciones.

    f. Cuando haya acabado de modificar la configuración, haga clic en el botón **Crear**.

5. La creación tarda varios minutos en completarse; la ventana de salida indicará el momento en que el clúster está totalmente creado.

## <a name="install-the-imported-certificate"></a>Instalación del certificado importado

Instale el certificado importado como parte del paso de creación del clúster, en la ubicación del almacén del **usuario actual** y proporcione la contraseña de clave privada que utilizó.

Para confirmar la instalación, abra **Administrar certificados de usuario** desde el panel de control y confirme que el certificado se ha instalado en **Certificados: usuario actual** -> **Personal** -> **Certificados**. El certificado debe ser similar a *[nombre del clúster]* . *[ubicación del clúster]* .cloudapp.azure.com (por ejemplo, *fabrikamfibercallcenter.southcentralus.cloudapp.azure.com*. 

## <a name="allow-your-application-running-in-azure-to-access-sql-database"></a>Permiso a la aplicación que se ejecuta en Azure para acceder a SQL Database

Anteriormente, creó una regla de firewall de SQL para conceder acceso a la aplicación que se ejecuta localmente.  A continuación, tendrá que permitir a la aplicación que se ejecuta en Azure que acceda a SQL Database.  Cree un [punto de conexión de servicio de red virtual](../azure-sql/database/vnet-service-endpoint-rule-overview.md) para el clúster de Service Fabric y, a continuación, cree una regla para permitir que ese punto de conexión tenga acceso a SQL Database. Asegúrese de especificar la variable del grupo de recursos del clúster que anotó al crear el clúster.

```powershell
# Create a virtual network service endpoint
$clusterresourcegroup = "<cluster resource group>"
$resource = Get-AzResource -ResourceGroupName $clusterresourcegroup -ResourceType Microsoft.Network/virtualNetworks | Select-Object -first 1
$vnetName = $resource.Name

Write-Host 'Virtual network name: ' $vnetName

# Get the virtual network by name.
$vnet = Get-AzVirtualNetwork `
  -ResourceGroupName $clusterresourcegroup `
  -Name              $vnetName

Write-Host "Get the subnet in the virtual network:"

# Get the subnet, assume the first subnet contains the Service Fabric cluster.
$subnet = Get-AzVirtualNetworkSubnetConfig -VirtualNetwork $vnet | Select-Object -first 1

$subnetName = $subnet.Name
$subnetID = $subnet.Id
$addressPrefix = $subnet.AddressPrefix

Write-Host "Subnet name: " $subnetName " Address prefix: " $addressPrefix " ID: " $subnetID

# Assign a Virtual Service endpoint 'Microsoft.Sql' to the subnet.
$vnet = Set-AzVirtualNetworkSubnetConfig `
  -Name            $subnetName `
  -AddressPrefix   $addressPrefix `
  -VirtualNetwork  $vnet `
  -ServiceEndpoint Microsoft.Sql | Set-AzVirtualNetwork

$vnet.Subnets[0].ServiceEndpoints;  # Display the first endpoint.

# Add a SQL DB firewall rule for the virtual network service endpoint
$subnet = Get-AzVirtualNetworkSubnetConfig `
  -Name           $subnetName `
  -VirtualNetwork $vnet;

$VNetRuleName="ServiceFabricClusterVNetRule"
$vnetRuleObject1 = New-AzSqlServerVirtualNetworkRule `
  -ResourceGroupName      $dbresourcegroupname `
  -ServerName             $servername `
  -VirtualNetworkRuleName $VNetRuleName `
  -VirtualNetworkSubnetId $subnetID;
```

## <a name="deploy-the-application-to-azure"></a>Implementación de la aplicación en Azure

Ahora que la aplicación está lista, puede implementarla en un clúster de Azure directamente desde Visual Studio.  En el Explorador de soluciones, haga clic con el botón derecho en el proyecto de aplicación **FabrikamFiber.CallCenterApplication** y elija **Publicar**.  En **Punto de conexión**, seleccione el punto de conexión del clúster que creó anteriormente.  En **Azure Container Registry**, seleccione el registro de contenedor que creó anteriormente.  Haga clic en **Publicar** para implementar la aplicación en el clúster de Azure.

![Publicación de la aplicación][publish-app]

Siga el progreso de la implementación en la ventana de salida. Cuando la aplicación esté implementada, abra un explorador y escriba la dirección del clúster y el puerto de la aplicación. Por ejemplo, `http://fabrikamfibercallcenter.southcentralus.cloudapp.azure.com:8659/`.

![Captura de pantalla de la página principal de la aplicación Fabrikam Fiber CallCenter que se ejecuta en azure.com. La página muestra un panel con una lista de llamadas de soporte técnico.][fabrikam-web-page-deployed]

Si la página no se puede cargar o no solicita el certificado, intente abrir la ruta de acceso del explorador (por ejemplo, `https://fabrikamfibercallcenter.southcentralus.cloudapp.azure.com:19080/Explorer` y seleccione el certificado recién instalado).

## <a name="set-up-continuous-integration-and-deployment-cicd-with-a-service-fabric-cluster"></a>Configuración de la integración e implementación continuas (CI/CD) con un clúster de Service Fabric

Para aprender a usar Azure DevOps para configurar la implementación de la aplicación con CI/CD en un clúster de Service Fabric, consulte [Tutorial: Implementación de una aplicación con CI/CD en un clúster de Service Fabric](service-fabric-tutorial-deploy-app-with-cicd-vsts.md). El proceso que se describe en el tutorial es el mismo para este proyecto (FabrikamFiber), solo debe omitir la descarga del ejemplo de votación y usar FabrikamFiber como nombre del repositorio, en lugar de Voting.

## <a name="clean-up-resources"></a>Limpieza de recursos

Si ha terminado, asegúrese de quitar todos los recursos que haya creado.  La manera más sencilla consiste en quitar los grupos de recursos que contienen el clúster de Service Fabric, Azure SQL DB y Azure Container Registry.

```powershell
$dbresourcegroupname = "fabrikam-fiber-db-group"
$acrresourcegroupname = "fabrikam-acr-group"
$clusterresourcegroupname="fabrikamcallcentergroup"

# Remove the Azure SQL DB
Remove-AzResourceGroup -Name $dbresourcegroupname

# Remove the container registry
Remove-AzResourceGroup -Name $acrresourcegroupname

# Remove the Service Fabric cluster
Remove-AzResourceGroup -Name $clusterresourcegroupname
```

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido a:

> [!div class="checklist"]
>
> * Inclusión de una aplicación existente en un contenedor con Visual Studio
> * Creación de una base de datos de Azure SQL Database
> * Creación de un registro de contenedor de Azure
> * Implementación de una aplicación de Service Fabric en Azure

En la siguiente parte del tutorial aprenderá a [implementar una aplicación de contenedor con CI/CD en un clúster de Service Fabric](service-fabric-tutorial-deploy-container-app-with-cicd-vsts.md).

[link-fabrikam-github]: https://aka.ms/fabrikamcontainer
[link-azure-powershell-install]: /powershell/azure/install-Az-ps
[link-servicefabric-create-secure-clusters]: service-fabric-cluster-creation-via-arm.md
[link-visualstudio-cd-extension]: https://aka.ms/cd4vs
[link-servicefabric-containers]: service-fabric-get-started-containers.md
[link-azure-portal]: https://portal.azure.com
[link-sf-clustertemplate]: https://aka.ms/securepreviewonelineclustertemplate
[link-azure-pricing-calculator]: https://azure.microsoft.com/pricing/calculator/
[link-azure-subscription]: https://azure.microsoft.com/free/
[link-vsts-account]: https://www.visualstudio.com/team-services/pricing/
[link-azure-sql]: /azure/sql-database/

[fabrikam-web-page]: media/service-fabric-host-app-in-a-container/fabrikam-web-page.png
[fabrikam-web-page-deployed]: media/service-fabric-host-app-in-a-container/fabrikam-web-page-deployed.png
[publish-app]: media/service-fabric-host-app-in-a-container/publish-app.png
