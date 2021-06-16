---
title: Configuración y uso de la identidad administrada de aplicación en nodos de clúster administrados de Service Fabric
description: Aprenda a configurar y usar una identidad administrada de aplicación en un clúster administrado de Azure Service Fabric implementado con plantilla de ARM.
ms.topic: how-to
ms.date: 5/10/2021
ms.openlocfilehash: 4e488f9ebb78208617ca59fdb88dbec454de10e1
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111958284"
---
# <a name="deploy-a-service-fabric-application-with-managed-identity"></a>Implementación de una aplicación de Service Fabric con identidad administrada

Para implementar una aplicación de Service Fabric con una identidad administrada, la aplicación debe implementarse a través de Azure Resource Manager, normalmente con una plantilla de Azure Resource Manager. Para obtener más información sobre cómo implementar la aplicación de Service Fabric a través de Azure Resource Manager, vea [Administración de aplicaciones y servicios como recursos de Azure Resource Manager](service-fabric-application-arm-resource.md).

> [!NOTE] 
> 
> Las aplicaciones que no se implementan como un recurso de Azure **no pueden** tener identidades administradas. 
>
> La implementación de la aplicación de Service Fabric con identidad administrada es compatible con la versión `"2021-05-01"` de la API en clústeres administrados.

Las plantillas de clúster administrado de ejemplo están disponibles aquí: [plantillas de clúster administrado de Service Fabric](https://github.com/Azure-Samples/service-fabric-cluster-templates)

## <a name="managed-identity-support-in-service-fabric-managed-cluster"></a>Compatibilidad con identidad administrada en el clúster administrado de Service Fabric

Cuando una aplicación de Service Fabric se configura con [identidades administradas para los recursos de Azure](../active-directory/managed-identities-azure-resources/overview.md) y se implementa en el clúster, desencadenará la configuración automática del *servicio de token de identidad administrada* en el clúster administrado de Service Fabric. Este servicio es responsable de la autenticación de aplicaciones de Service Fabric que usan sus identidades administradas y de la obtención de los tokens de acceso en su nombre. Una vez habilitado el servicio, puede verlo en Service Fabric Explorer, en la sección **Sistema** del panel izquierdo. Se ejecuta con el nombre **fabric:/System/ManagedIdentityTokenService**.

>[!NOTE]
>La primera vez que se implementa una aplicación con identidades administradas, debería haber una implementación única más larga debido al cambio en la configuración automática del clúster. Esto debería tardar de 15 minutos para un clúster de zona a 45 minutos para un clúster que abarca varias zonas. Si hay otras implementaciones en marcha, la configuración de MITS esperará a que se completen primero.

El recurso de aplicación admite la asignación de SystemAssigned o UserAssigned y esta asignación se puede realizar tal como se muestra en el fragmento de código siguiente.

```json
{
  "type": "Microsoft.ServiceFabric/managedclusters/applications",
  "apiVersion": "2021-05-01",
  "identity": {
    "type": "SystemAssigned",
    "userAssignedIdentities": {}
  },
}

```
[Referencia JSON completa](/azure/templates/microsoft.servicefabric/2021-05-01/managedclusters/applications?tabs=json)

## <a name="user-assigned-identity"></a>Identidad asignada por el usuario

Para habilitar la aplicación con una identidad asignada por el usuario, agregue primero la propiedad de **identidad** al recurso de aplicación de tipo **userAssigned** y las identidades asignadas por el usuario a las que se hace referencia. A continuación, agregue una sección **managedIdentities** dentro de la sección **properties** del recurso **application** que contiene una lista de nombres descriptivos para la asignación del objeto principalId para cada una de las identidades asignadas por el usuario. Para más información sobre las identidades asignadas por el usuario, consulte [Creación, enumeración o eliminación de una identidad administrada asignada por el usuario](../active-directory/managed-identities-azure-resources/how-to-manage-ua-identity-powershell.md).

### <a name="application-template"></a>Plantilla de la aplicación

Para habilitar la aplicación con la identidad asignada por el usuario, primero agregue la propiedad **identity** al recurso de la aplicación con el tipo **userAssigned** y las identidades asignadas por el usuario a las que se hace referencia y, después, agregue un objeto **managedIdentities** dentro de la sección **properties** que contiene una lista de nombres descriptivos para la asignación del objeto principalId para cada una de las identidades asignadas por el usuario.

```json
{
  "apiVersion": "2021-05-01",
  "type": "Microsoft.ServiceFabric/managedclusters/applications",
  "name": "[concat(parameters('clusterName'), '/', parameters('applicationName'))]",
  "location": "[resourceGroup().location]",
  "dependsOn": [
    "[parameters('applicationVersion')]",
    "[resourceId('Microsoft.ManagedIdentity/userAssignedIdentities/', parameters('userAssignedIdentityName'))]"
  ],
  "identity": {
    "type" : "userAssigned",
    "userAssignedIdentities": {
        "[resourceId('Microsoft.ManagedIdentity/userAssignedIdentities/', parameters('userAssignedIdentityName'))]": {}
    }
  },
  "properties": {
    "version": "[parameters('applicationVersion')]",
    "parameters": {
    },
    "managedIdentities": [
      {
        "name" : "[parameters('userAssignedIdentityName')]",
        "principalId" : "[reference(resourceId('Microsoft.ManagedIdentity/userAssignedIdentities/', parameters('userAssignedIdentityName')), '2018-11-30').principalId]"
      }
    ]
  }
}
```

En el ejemplo anterior, se usa el nombre del recurso de la identidad asignada por el usuario como nombre descriptivo de la identidad administrada para la aplicación. En los ejemplos siguientes se da por hecho que el nombre descriptivo real es "AdminUser".

### <a name="application-package"></a>Paquete de aplicación

1. Para cada identidad definida en la sección `managedIdentities` de la plantilla de Azure Resource Manager, agregue una etiqueta `<ManagedIdentity>` en el manifiesto de aplicación, en la sección **Principals**. El atributo `Name` debe coincidir con la propiedad `name` definida en la sección `managedIdentities`.

    **ApplicationManifest.xml**

    ```xml
      <Principals>
        <ManagedIdentities>
          <ManagedIdentity Name="AdminUser" />
        </ManagedIdentities>
      </Principals>
    ```

2. En la sección **ServiceManifestImport**, agregue **IdentityBindingPolicy** para el servicio que usa la identidad administrada. Esta directiva asigna la identidad `AdminUser` a un nombre de identidad específico del servicio que se debe agregar en el manifiesto de servicio más adelante.

    **ApplicationManifest.xml**

    ```xml
      <ServiceManifestImport>
        <Policies>
          <IdentityBindingPolicy ServiceIdentityRef="WebAdmin" ApplicationIdentityRef="AdminUser" />
        </Policies>
      </ServiceManifestImport>
    ```

3. Actualice el manifiesto de servicio para agregar **ManagedIdentity** dentro de la sección **Resources** con el nombre que coincide con `ServiceIdentityRef` en `IdentityBindingPolicy` del manifiesto de aplicación:

    **ServiceManifest.xml**

    ```xml
      <Resources>
        ...
        <ManagedIdentities DefaultIdentity="WebAdmin">
          <ManagedIdentity Name="WebAdmin" />
        </ManagedIdentities>
      </Resources>
    ```

## <a name="system-assigned-managed-identity"></a>Identidad administrada asignada por el sistema

### <a name="application-template"></a>Plantilla de la aplicación

Para habilitar una aplicación con una identidad administrada asignada por el sistema, agregue la propiedad **identity** al recurso de aplicación, con el tipo **systemAssigned** como se muestra en el ejemplo siguiente:

```json
    {
      "apiVersion": "2021-05-01",
      "type": "Microsoft.ServiceFabric/managedclusters/applications",
      "name": "[concat(parameters('clusterName'), '/', parameters('applicationName'))]",
      "location": "[resourceGroup().location]",
      "dependsOn": [
        "[concat('Microsoft.ServiceFabric/clusters/', parameters('clusterName'), '/applicationTypes/', parameters('applicationTypeName'), '/versions/', parameters('applicationTypeVersion'))]"
      ],
      "identity": {
        "type" : "systemAssigned"
      },
      "properties": {
        "typeName": "[parameters('applicationTypeName')]",
        "typeVersion": "[parameters('applicationTypeVersion')]",
        "parameters": {
        }
      }
    }
```
Esta propiedad declara (para Azure Resource Manager y los proveedores de recursos de identidad administrada y de Service Fabric, respectivamente) que este recurso debe tener una identidad administrada (`system assigned`) implícita.

### <a name="application-and-service-package"></a>Aplicación y paquete de servicio

1. Actualice el manifiesto de aplicación para agregar un elemento **ManagedIdentity** en la sección **Principals** que contenga una sola entrada, como se muestra a continuación:

    **ApplicationManifest.xml**

    ```xml
    <Principals>
      <ManagedIdentities>
        <ManagedIdentity Name="SystemAssigned" />
      </ManagedIdentities>
    </Principals>
    ```
    De este modo, la identidad asignada se asigna a la aplicación como un recurso para un nombre descriptivo, para la posterior asignación a los servicios que componen la aplicación. 

2. En la sección **ServiceManifestImport** correspondiente al servicio al que se está asignando la identidad administrada, agregue un elemento **IdentityBindingPolicy**, como se indica a continuación:

    **ApplicationManifest.xml**

      ```xml
        <ServiceManifestImport>
          <Policies>
            <IdentityBindingPolicy ServiceIdentityRef="WebAdmin" ApplicationIdentityRef="SystemAssigned" />
          </Policies>
        </ServiceManifestImport>
      ```

    Este elemento asigna la identidad de la aplicación al servicio; sin esta asignación, el servicio no podrá acceder a la identidad de la aplicación. En el fragmento de código anterior, la identidad `SystemAssigned` (que es una palabra clave reservada) se asigna a la definición del servicio con el nombre descriptivo `WebAdmin`.

3. Actualice el manifiesto de servicio para agregar un elemento **ManagedIdentity** en la sección **Resources** con el nombre que coincide con el valor de la opción `ServiceIdentityRef` en la definición `IdentityBindingPolicy` del manifiesto de aplicación:

    **ServiceManifest.xml**

    ```xml
      <Resources>
        ...
        <ManagedIdentities DefaultIdentity="WebAdmin">
          <ManagedIdentity Name="WebAdmin" />
        </ManagedIdentities>
      </Resources>
    ```
    Esta es la asignación equivalente de una identidad a un servicio, tal como se ha descrito anteriormente, pero desde la perspectiva de la definición del servicio. Aquí se hace referencia a la identidad mediante su nombre descriptivo (`WebAdmin`), como se declaró en el manifiesto de aplicación.

## <a name="next-steps"></a>Pasos siguientes

* [Aprovechamiento de la identidad administrada de una aplicación de Service Fabric desde el código de servicio](./how-to-managed-identity-service-fabric-app-code.md)
* [Concesión de acceso a otros recursos de Azure para una aplicación de Azure Service Fabric](./how-to-grant-access-other-resources.md)