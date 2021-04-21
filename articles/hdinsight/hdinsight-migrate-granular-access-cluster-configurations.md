---
title: Configuraciones de clústeres de Azure HDInsight con acceso basado en roles pormenorizado
description: Obtenga más información sobre los cambios necesarios como parte de la migración a un acceso granular basado en rol para las configuraciones de clústeres de HDInsight.
author: tylerfox
ms.author: tyfox
ms.service: hdinsight
ms.topic: conceptual
ms.date: 04/20/2020
ms.openlocfilehash: afb30f4648f1649bf6cc6cc6a3bf02f433f49d45
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107774938"
---
# <a name="migrate-to-granular-role-based-access-for-cluster-configurations"></a>Migración a acceso basado en rol detallado para configuraciones de clúster

Se van a realizar algunos cambios importantes para permitir un acceso más pormenorizado basado en rol para obtener información confidencial. Como parte de estos cambios, es posible que deba tomar alguna medida **antes del 3 de septiembre de 2019** si usa alguna de las [entidades o de los escenarios afectados](#am-i-affected-by-these-changes).

## <a name="what-is-changing"></a>¿Qué está cambiando?

Anteriormente, los usuarios de clústeres que poseían los [roles de Azure](../role-based-access-control/rbac-and-directory-admin-roles.md) de propietario, colaborador o lector podían obtener secretos a través de la API de HDInsight, ya que estaban disponibles para cualquier persona con el permiso `*/read`. Los secretos se definen como valores que podrían utilizarse para obtener un acceso más elevado que el que debería permitir el rol de un usuario. Estos incluyen valores como las credenciales HTTP de la puerta de enlace del clúster, las claves de la cuenta de almacenamiento y las credenciales de la base de datos.

A partir del 3 de septiembre de 2019, para acceder a estos secretos será necesario el permiso `Microsoft.HDInsight/clusters/configurations/action`, lo que significa que los usuarios con el rol de lector ya no podrán acceder a ellos. Los roles que tienen este permiso son Colaborador, Propietario y el nuevo rol Operador de clúster de HDInsight (a continuación se ofrece más información).

También estamos introduciendo el nuevo rol de [operador de clústeres de HDInisght](../role-based-access-control/built-in-roles.md#hdinsight-cluster-operator), que será capaz de recuperar secretos sin que se le concedan los permisos administrativos de colaborador o propietario. Resumiendo:

| Role                                  | Anteriormente                                                                                       | A partir de ahora       |
|---------------------------------------|--------------------------------------------------------------------------------------------------|-----------|
| Lector                                | - Acceso de lectura, incluidos secretos.                                                                   | - Acceso de lectura, **excluidos** secretos | 
| Operador de clústeres de HDInsight<br>(Nuevo rol) | N/D                                                                                              | - Acceso de lectura/escritura, incluidos secretos         | 
| Colaborador                           | - Acceso de lectura/escritura, incluidos secretos.<br>- Creación y administración de todos los tipos de recursos de Azure.<br>- Ejecución de acciones de script.     | Sin cambios |
| Propietario                                 | - Acceso de lectura/escritura, incluidos secretos.<br>- Acceso total a todos los recursos<br>- Delegación del acceso a otros usuarios.<br>- Ejecución de acciones de script. | Sin cambios |

Para obtener información sobre cómo agregar la asignación del rol de operador de clústeres de HDInsight a un usuario para otorgarle acceso de lectura o escritura a los secretos del clúster, consulte la sección siguiente [Adición de la asignación del rol de operador de clústeres de HDInsight a un usuario](#add-the-hdinsight-cluster-operator-role-assignment-to-a-user).

## <a name="am-i-affected-by-these-changes"></a>¿Estos cambios me afectan?

Afecta a las entidades y los escenarios siguientes:

- [API](#api): Usuarios que utilizan los puntos de conexión `/configurations` o `/configurations/{configurationName}`.
- [Azure HDInsight Tools for Visual Studio Code](#azure-hdinsight-tools-for-visual-studio-code) 1.1.1 o anterior.
- [Azure Toolkit for IntelliJ](#azure-toolkit-for-intellij) 3.20.0 o anterior.
- [Herramientas de Azure Data Lake y de Stream Analytics para Visual Studio](#azure-data-lake-and-stream-analytics-tools-for-visual-studio), anterior a la versión 2.3.9000.1.
- [Azure Toolkit for Eclipse](#azure-toolkit-for-eclipse) versión 3.15.0 o anterior.
- [SDK para .NET](#sdk-for-net)
    - [versiones 1.x o 2.x](#versions-1x-and-2x): Usuarios que utilizan los métodos `GetClusterConfigurations`, `GetConnectivitySettings`, `ConfigureHttpSettings`, `EnableHttp` o `DisableHttp` desde la clase ConfigurationsOperationsExtensions.
    - [versiones 3.x y posteriores](#versions-3x-and-up): Usuarios que utilizan los métodos `Get`, `Update`, `EnableHttp` o `DisableHttp` desde la clase `ConfigurationsOperationsExtensions`.
- [SDK para Python](#sdk-for-python): Usuarios que utilizan los métodos `get` o `update` desde la clase `ConfigurationsOperations`.
- [SDK para Java](#sdk-for-java): Usuarios que utilizan los métodos `update` o `get` desde la clase `ConfigurationsInner`.
- [SDK para Go](#sdk-for-go): Usuarios que utilizan los métodos `Get` o `Update` desde la estructura `ConfigurationsClient`.
- [Az.HDInsight PowerShell](#azhdinsight-powershell) anterior a la versión 2.0.0.
Consulte las secciones siguientes (o use los vínculos anteriores) para ver los pasos de migración para su escenario.

### <a name="api"></a>API

Las API siguientes se puede cambiar o están en desuso:

- [**GET /configurations/{configurationName}**](/rest/api/hdinsight/hdinsight-cluster#get-configuration) (se quita la información confidencial)
    - Utilizada anteriormente para obtener tipos de configuración individuales (incluidos secretos).
    - A partir del 3 de septiembre de 2019, esta llamada API devolverá tipos de configuración individuales con secretos omitidos. Para obtener todas las configuraciones, como los secretos, use la nueva llamada POST /configurations. Para obtener solo la configuración de la puerta de enlace, use la nueva llamada POST /getGatewaySettings.
- [**GET /configurations**](/rest/api/hdinsight/hdinsight-cluster#get-configuration) (en desuso)
    - Se utilizaba para obtener todas las configuraciones (incluidos los secretos).
    - A partir del 3 de septiembre de 2019, esta llamada API quedará en desuso y ya no se admitirá. Para obtener todas las configuraciones futuras, use la nueva llamada POST /configurations. Para obtener las configuraciones con los parámetros confidenciales omitidos, utilice la llamada GET /configurations/{configurationName}.
- [**POST /configurations/{configurationName}**](/rest/api/hdinsight/hdinsight-cluster#update-gateway-settings) (en desuso)
    - Anteriormente se usaba para actualizar las credenciales de puerta de enlace.
    - A partir del 3 de septiembre de 2019, esta llamada API quedará en desuso y ya no se admitirá. Use la nueva llamada POST /updateGatewaySettings en su lugar.

Se han agregado las API de reemplazo siguientes:</span>

- [**POST /configurations**](/rest/api/hdinsight/hdinsight-cluster#list-configurations)
    - Utilice esta API para obtener todas las configuraciones, incluidos los secretos.
- [**POST /getGatewaySettings**](/rest/api/hdinsight/hdinsight-cluster#get-gateway-settings)
    - Utilice esta API para obtener la configuración de puerta de enlace.
- [**POST /updateGatewaySettings**](/rest/api/hdinsight/hdinsight-cluster#update-gateway-settings)
    - Utilice esta API para actualizar la configuración de la puerta de enlace (nombre de usuario o contraseña).

### <a name="azure-hdinsight-tools-for-visual-studio-code"></a>Azure HDInsight Tools for Visual Studio Code

Si usa la versión 1.1.1 o anterior, actualice a la [versión más reciente de Azure HDInsight Tools for Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=mshdinsight.azure-hdinsight&ssr=false) para evitar interrupciones.

### <a name="azure-toolkit-for-intellij"></a>Kit de herramientas de Azure para IntelliJ

Si usa la versión 3.20.0 o anterior, actualice a la [versión más reciente del complemento Azure Toolkit for IntelliJ](https://plugins.jetbrains.com/plugin/8053-azure-toolkit-for-intellij) para evitar interrupciones.

### <a name="azure-data-lake-and-stream-analytics-tools-for-visual-studio"></a>Herramientas de Azure Data Lake y de Stream Analytics para Visual Studio

Actualice a la versión 2.3.9000.1 o posterior de [Azure Data Lake y Stream Analytics Tools para Visual Studio](https://marketplace.visualstudio.com/items?itemName=ADLTools.AzureDataLakeandStreamAnalyticsTools&ssr=false#overview) para evitar interrupciones.  Para obtener ayuda con la actualización, consulte la documentación [Actualización de Data Lake Tools para Visual Studio](./hadoop/apache-hadoop-visual-studio-tools-get-started.md#update-data-lake-tools-for-visual-studio).

### <a name="azure-toolkit-for-eclipse"></a>Kit de herramientas de Azure para Eclipse

Si usa la versión 3.15.0 o anterior, actualice a la [versión más reciente de Azure Toolkit for Eclipse](https://marketplace.eclipse.org/content/azure-toolkit-eclipse) para evitar interrupciones.

### <a name="sdk-for-net"></a>SDK para .NET

#### <a name="versions-1x-and-2x"></a>Versiones 1.x y 2.x

Actualice a la [versión 2.1.0](https://www.nuget.org/packages/Microsoft.Azure.Management.HDInsight/2.1.0) del SDK de HDInsight para .NET. Es posible que sea necesario realizar modificaciones mínimas en el código si está utilizando un método afectado por estos cambios:

- `ClusterOperationsExtensions.GetClusterConfigurations` ya **no devuelve parámetros confidenciales** como claves de almacenamiento (sitio principal) o credenciales de HTTP (puerta de enlace).
    - Para recuperar todas las configuraciones, incluidos los parámetros confidenciales, utilice `ClusterOperationsExtensions.ListConfigurations` a partir de ahora.  Tenga en cuenta que los usuarios con el rol "Lector" no podrán usar este método. Esto permite un control granular sobre el cual los usuarios pueden acceder a información confidencial para un clúster.
    - Para recuperar solo las credenciales de la puerta de enlace HTTP, utilice `ClusterOperationsExtensions.GetGatewaySettings`.

- `ClusterOperationsExtensions.GetConnectivitySettings` ahora está en desuso y se ha reemplazado por `ClusterOperationsExtensions.GetGatewaySettings`.

- `ClusterOperationsExtensions.ConfigureHttpSettings` ahora está en desuso y se ha reemplazado por `ClusterOperationsExtensions.UpdateGatewaySettings`.

- `ConfigurationsOperationsExtensions.EnableHttp` y `DisableHttp` ahora están en desuso. Ahora HTTP siempre está habilitado, por lo que estos métodos ya no son necesarios.

#### <a name="versions-3x-and-up"></a>Versiones 3.x y posteriores

Actualice a la [versión 5.0.0](https://www.nuget.org/packages/Microsoft.Azure.Management.HDInsight/5.0.0) o posterior del SDK de HDInsight para .NET. Es posible que sea necesario realizar modificaciones mínimas en el código si está utilizando un método afectado por estos cambios:

- [`ConfigurationOperationsExtensions.Get`](/dotnet/api/microsoft.azure.management.hdinsight.configurationsoperationsextensions.get)**ya no devuelve parámetros confidenciales** como claves de almacenamiento (sitio principal) o credenciales de HTTP (puerta de enlace).
    - Para recuperar todas las configuraciones, incluidos los parámetros confidenciales, utilice [`ConfigurationOperationsExtensions.List`](/dotnet/api/microsoft.azure.management.hdinsight.configurationsoperationsextensions.list) a partir de ahora.  Tenga en cuenta que los usuarios con el rol "Lector" no podrán usar este método. Esto permite un control granular sobre el cual los usuarios pueden acceder a información confidencial para un clúster. 
    - Para recuperar solo las credenciales de la puerta de enlace HTTP, utilice [`ClusterOperationsExtensions.GetGatewaySettings`](/dotnet/api/microsoft.azure.management.hdinsight.clustersoperationsextensions.getgatewaysettings). 
- [`ConfigurationsOperationsExtensions.Update`](/dotnet/api/microsoft.azure.management.hdinsight.configurationsoperationsextensions.update) ahora está en desuso y se ha reemplazado por [`ClusterOperationsExtensions.UpdateGatewaySettings`](/dotnet/api/microsoft.azure.management.hdinsight.clustersoperationsextensions.updategatewaysettings). 
- [`ConfigurationsOperationsExtensions.EnableHttp`](/dotnet/api/microsoft.azure.management.hdinsight.configurationsoperationsextensions.enablehttp) y [`DisableHttp`](/dotnet/api/microsoft.azure.management.hdinsight.configurationsoperationsextensions.disablehttp) ahora están en desuso. Ahora HTTP siempre está habilitado, por lo que estos métodos ya no son necesarios.

### <a name="sdk-for-python"></a>SDK para Python

Actualice a la [versión 1.0.0](https://pypi.org/project/azure-mgmt-hdinsight/1.0.0/) o posterior del SDK de HDInsight para Python. Es posible que sea necesario realizar modificaciones mínimas en el código si está utilizando un método afectado por estos cambios:

- [`ConfigurationsOperations.get`](/python/api/azure-mgmt-hdinsight/azure.mgmt.hdinsight.operations.configurationsoperations#get-resource-group-name--cluster-name--configuration-name--custom-headers-none--raw-false----operation-config-)**ya no devuelve parámetros confidenciales** como claves de almacenamiento (sitio principal) o credenciales de HTTP (puerta de enlace).
    - Para recuperar todas las configuraciones, incluidos los parámetros confidenciales, utilice [`ConfigurationsOperations.list`](/python/api/azure-mgmt-hdinsight/azure.mgmt.hdinsight.operations.configurationsoperations#list-resource-group-name--cluster-name--custom-headers-none--raw-false----operation-config-) a partir de ahora.  Tenga en cuenta que los usuarios con el rol "Lector" no podrán usar este método. Esto permite un control granular sobre el cual los usuarios pueden acceder a información confidencial para un clúster. 
    - Para recuperar solo las credenciales de la puerta de enlace HTTP, utilice [`ClusterOperations.get_gateway_settings`](/python/api/azure-mgmt-hdinsight/azure.mgmt.hdinsight.operations.clustersoperations#get-gateway-settings-resource-group-name--cluster-name--custom-headers-none--raw-false----operation-config-).
- [`ConfigurationsOperations.update`](/python/api/azure-mgmt-hdinsight/azure.mgmt.hdinsight.operations.configurationsoperations#update-resource-group-name--cluster-name--configuration-name--parameters--custom-headers-none--raw-false--polling-true----operation-config-) ahora está en desuso y se ha reemplazado por [`ClusterOperations.update_gateway_settings`](/python/api/azure-mgmt-hdinsight/azure.mgmt.hdinsight.operations.clustersoperations#update-gateway-settings-resource-group-name--cluster-name--parameters--custom-headers-none--raw-false--polling-true----operation-config-).

### <a name="sdk-for-java"></a>SDK para Java

Actualice a la [versión 1.0.0](https://search.maven.org/artifact/com.microsoft.azure.hdinsight.v2018_06_01_preview/azure-mgmt-hdinsight/1.0.0/jar) o posterior del SDK de HDInsight para Java. Es posible que sea necesario realizar modificaciones mínimas en el código si está utilizando un método afectado por estos cambios:

- `ConfigurationsInner.get` ya **no devuelve parámetros confidenciales** como claves de almacenamiento (sitio principal) o credenciales de HTTP (puerta de enlace).
- `ConfigurationsInner.update` ahora está en desuso.

### <a name="sdk-for-go"></a>SDK para Go

Actualice a la [versión 27.1.0](https://github.com/Azure/azure-sdk-for-go/tree/master/services/preview/hdinsight/mgmt/2015-03-01-preview/hdinsight) o posterior del SDK de HDInsight para Go. Es posible que sea necesario realizar modificaciones mínimas en el código si está utilizando un método afectado por estos cambios:

- [`ConfigurationsClient.get`](https://godoc.org/github.com/Azure/azure-sdk-for-go/services/preview/hdinsight/mgmt/2015-03-01-preview/hdinsight#ConfigurationsClient.Get)**ya no devuelve parámetros confidenciales** como claves de almacenamiento (sitio principal) o credenciales de HTTP (puerta de enlace).
    - Para recuperar todas las configuraciones, incluidos los parámetros confidenciales, utilice [`ConfigurationsClient.list`](https://godoc.org/github.com/Azure/azure-sdk-for-go/services/preview/hdinsight/mgmt/2015-03-01-preview/hdinsight#ConfigurationsClient.List) a partir de ahora.  Tenga en cuenta que los usuarios con el rol "Lector" no podrán usar este método. Esto permite un control granular sobre el cual los usuarios pueden acceder a información confidencial para un clúster. 
    - Para recuperar solo las credenciales de la puerta de enlace HTTP, utilice [`ClustersClient.get_gateway_settings`](https://godoc.org/github.com/Azure/azure-sdk-for-go/services/preview/hdinsight/mgmt/2015-03-01-preview/hdinsight#ClustersClient.GetGatewaySettings).
- [`ConfigurationsClient.update`](https://godoc.org/github.com/Azure/azure-sdk-for-go/services/preview/hdinsight/mgmt/2015-03-01-preview/hdinsight#ConfigurationsClient.Update) ahora está en desuso y se ha reemplazado por [`ClustersClient.update_gateway_settings`](https://godoc.org/github.com/Azure/azure-sdk-for-go/services/preview/hdinsight/mgmt/2015-03-01-preview/hdinsight#ClustersClient.UpdateGatewaySettings).

### <a name="azhdinsight-powershell"></a>Az.HDInsight PowerShell
Actualice a [Az PowerShell versión 2.0.0](https://www.powershellgallery.com/packages/Az) o posterior para evitar interrupciones.  Es posible que sea necesario realizar modificaciones mínimas en el código si está utilizando un método afectado por estos cambios.
- `Grant-AzHDInsightHttpServicesAccess` ahora está en desuso y se ha reemplazado por el nuevo cmdlet `Set-AzHDInsightGatewayCredential`.
- Se ha actualizado `Get-AzHDInsightJobOutput` para admitir el acceso granular basado en roles a la clave de almacenamiento.
    - No se verán afectados los usuarios con roles de operador, colaborador o propietario de clúster de HDInsight.
    - Los usuarios con el rol de lector deben especificar el parámetro `DefaultStorageAccountKey` explícitamente.
- `Revoke-AzHDInsightHttpServicesAccess` ahora está en desuso. Ahora HTTP siempre está habilitado, por lo que este cmdlet ya no es necesario.
 Consulte la [guía de migración de az.HDInsight](https://github.com/Azure/azure-powershell/blob/master/documentation/migration-guides/Az.2.0.0-migration-guide.md#azhdinsight) para obtener más detalles.

## <a name="add-the-hdinsight-cluster-operator-role-assignment-to-a-user"></a>Adición de la asignación del rol de operador de clústeres de HDInsight a un usuario

Un usuario con el rol [Propietario](../role-based-access-control/built-in-roles.md#owner) puede asignar el rol [Operador de clúster de HDInsight](../role-based-access-control/built-in-roles.md#hdinsight-cluster-operator) a los usuarios que deseen tener acceso de lectura o escritura a los valores de configuración del clúster de HDInsight (como las credenciales de la puerta de enlace del clúster y las claves de la cuenta de almacenamiento).

### <a name="using-the-azure-cli"></a>Uso de la CLI de Azure

La forma más sencilla de agregar esta asignación de roles es con el comando `az role assignment create` en la CLI de Azure.

> [!NOTE]
> Este comando debe ejecutarlo un usuario con el rol Propietario, ya que solo ellos pueden otorgar estos permisos. `--assignee` es el nombre de la entidad de servicio o la dirección de correo electrónico del usuario a quien desea asignar el rol Operador de clúster de HDInsight. Si recibe un error de permisos insuficientes, consulte las preguntas más frecuentes a continuación.

#### <a name="grant-role-at-the-resource-cluster-level"></a>Concesión del rol en el nivel del recurso (clúster)

```azurecli-interactive
az role assignment create --role "HDInsight Cluster Operator" --assignee <user@domain.com> --scope /subscriptions/<SubscriptionId>/resourceGroups/<ResourceGroupName>/providers/Microsoft.HDInsight/clusters/<ClusterName>
```

#### <a name="grant-role-at-the-resource-group-level"></a>Concesión del rol en el nivel del grupo de recursos

```azurecli-interactive
az role assignment create --role "HDInsight Cluster Operator" --assignee user@domain.com -g <ResourceGroupName>
```

#### <a name="grant-role-at-the-subscription-level"></a>Concesión del rol en el nivel de suscripción

```azurecli-interactive
az role assignment create --role "HDInsight Cluster Operator" --assignee user@domain.com
```

### <a name="using-the-azure-portal"></a>Uso de Azure Portal

Como alternativa, puede usar Azure Portal para agregar la asignación del rol de operador de clústeres de HDInsight a un usuario. Consulte la documentación de [Asignación de roles de Azure mediante Azure Portal](../role-based-access-control/role-assignments-portal.md).

## <a name="faq"></a>Preguntas más frecuentes

### <a name="why-am-i-seeing-a-403-forbidden-response-after-updating-my-api-requests-andor-tool"></a>¿Por qué veo una respuesta 403 (prohibido) tras actualizar mis solicitudes de API o la herramienta?

Las configuraciones de clúster están ahora detrás del control de acceso basado en rol detallado y requieren el permiso `Microsoft.HDInsight/clusters/configurations/*` para acceder a ellas. Para obtener este permiso, asigne el rol Operador de clúster de HDInsight, Colaborador o Propietario al usuario o entidad de servicio que intentan acceder a las configuraciones.

### <a name="why-do-i-see-insufficient-privileges-to-complete-the-operation-when-running-the-azure-cli-command-to-assign-the-hdinsight-cluster-operator-role-to-another-user-or-service-principal"></a>¿Por qué veo el mensaje "No tiene privilegios suficientes para completar la operación" al ejecutar el comando de la CLI de Azure para asignar el rol Operador de clúster de HDInsight a otro usuario o entidad de servicio?

Además de tener el rol Propietario, el usuario o la entidad de servicio que ejecuta el comando debe tener suficientes permisos de Azure AD para buscar los identificadores de objeto del usuario asignado. Este mensaje indica que no tiene suficientes permisos de Azure AD. Intente reemplazar el argumento `-–assignee` por `–assignee-object-id` y proporcione el identificador de objeto del usuario asignado como parámetro en lugar del nombre (o el identificador de la entidad de seguridad en el caso de una identidad administrada). Consulte la sección de parámetros opcionales de la [documentación sobre el comando az role assignment create](/cli/azure/role/assignment#az_role_assignment_create) para más información.

Si sigue sin funcionar, póngase en contacto con el administrador de Azure AD para adquirir los permisos correctos.

### <a name="what-will-happen-if-i-take-no-action"></a>¿Qué ocurrirá si no se realiza ninguna acción?

A partir del 3 de septiembre de 2019, las llamadas `GET /configurations` y `POST /configurations/gateway` ya no devolverán ninguna información y la llamada a `GET /configurations/{configurationName}` ya no devolverá parámetros confidenciales, como las claves de la cuenta de almacenamiento o la contraseña del clúster. Lo mismo es cierto para los métodos de SDK y los cmdlets de PowerShell correspondientes.

Si usa una versión anterior de alguna de las herramientas de Visual Studio, VSCode, IntelliJ o Eclipse que se mencionaron anteriormente, estas dejarán de funcionar hasta que actualice.

Para más información, consulte la sección correspondiente de este documento para su escenario.
