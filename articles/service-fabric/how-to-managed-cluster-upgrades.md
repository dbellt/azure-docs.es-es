---
title: Actualización de clústeres administrados de Azure Service Fabric
description: Más información sobre las opciones para actualizar el clúster administrado de Azure Service Fabric
ms.topic: how-to
ms.date: 05/10/2021
ms.openlocfilehash: 478b39a6222906c793d826ab69edeeaddbb096bf
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111960993"
---
# <a name="manage-service-fabric-managed-cluster-upgrades"></a>Administración de actualizaciones del clúster administrado de Service Fabric

Un clúster de Azure Service Fabric es un recurso de su propiedad, pero que está parcialmente administrado por Microsoft. Aquí se muestra cuándo y cómo Microsoft actualiza el clúster administrado de Azure Service Fabric.

## <a name="set-upgrade-mode"></a>Establecimiento del modo de actualización

Los clústeres administrados de Azure Service Fabric se han configurado de manera predeterminada para recibir las actualizaciones automáticas de Service Fabric a medida que Microsoft las publica mediante una estrategia de [implementación por oleadas](#wave-deployment-for-automatic-upgrades). Como alternativa, puede configurar las actualizaciones de modo manual y elegir entre una lista de las versiones admitidas actualmente. Puede configurar estas opciones mediante el control *Actualizaciones de Fabric* de Azure Portal o mediante la opción `ClusterUpgradeMode` de la plantilla de implementación del clúster.

## <a name="wave-deployment-for-automatic-upgrades"></a>Implementación por oleadas para actualizaciones automáticas

Con la implementación por oleadas, puede crear una canalización para actualizar los clústeres de prueba, ensayo y producción en secuencia, separados por el "tiempo de preparación" integrado para validar las próximas versiones de Service Fabric antes de que se actualicen los clústeres de producción.

>NOTA: De forma predeterminada, los clústeres se establecerán en la oleada 0.

Para seleccionar una implementación por oleadas para la actualización automática, determine primero la oleada que asignará al clúster:

* **Oleada 0** (`Wave0`): los clústeres se actualizan en cuanto se publica una nueva compilación de Service fabric.
* **Oleada 1** (`Wave1`): los clústeres se actualizan después de la oleada 0 para permitir un tiempo de simulación mediante "bake". Esto sucede después de un mínimo de 7 días después de la oleada 0
* **Oleada 2** (`Wave2`): los clústeres se actualizan en último lugar para permitir un mayor tiempo de simulación mediante "bake". Esto sucede después de un mínimo de 14 días después de la oleada 0

## <a name="set-the-wave-for-your-cluster"></a>Configuración de las oleadas del clúster

Puede configurar el clúster en una de las oleadas disponibles mediante el control *Actualizaciones de Fabric* de Azure Portal o mediante la opción `ClusterUpgradeMode` de la plantilla de implementación del clúster.

### <a name="azure-portal"></a>Portal de Azure

Con Azure Portal, podrá elegir entre las oleadas automáticas disponibles al crear un clúster de Service Fabric.

:::image type="content" source="./media/how-to-managed-cluster-upgrades/portal-new-cluster-upgrade-waves-setting.png" alt-text="Elección entre las diferentes oleadas disponibles al crear un clúster en Azure Portal a partir de las opciones avanzadas":::

También puede alternar entre las diferentes oleadas automáticas disponibles en la sección **Actualizaciones de Fabric** de un recurso de clúster existente.

:::image type="content" source="./media/how-to-managed-cluster-upgrades/manage-upgrade-wave-settings.png" alt-text="Selección entre las diferentes oleadas automáticas de la sección &quot;Actualizaciones de Fabric&quot; del recurso de clúster en Azure Portal":::

### <a name="resource-manager-template"></a>Plantilla de Resource Manager

Para cambiar el modo de actualización del clúster mediante una plantilla de Resource Manager, especifique *Automática* o *Manual* en la propiedad `ClusterUpgradeMode` de la definición del recurso *Microsoft.ServiceFabric/clusters*. Si elige actualizaciones manuales, establezca también `clusterCodeVersion` en una [versión de tejido compatible](#query-for-supported-cluster-versions) actualmente.

#### <a name="manual-upgrade"></a>Actualización manual

```json
{
"apiVersion": "2021-05-01",
"type": "Microsoft.ServiceFabric/managedClusters",
"properties": {
        "ClusterUpgradeMode": "Manual",
        "ClusterCodeVersion": "7.2.457.9590"
        }
}
```

Después de una implementación correcta de la plantilla, se aplicarán los cambios al modo de actualización del clúster. Si el clúster está en modo manual, la actualización del clúster se iniciará automáticamente.

Las [directivas de mantenimiento del clúster](./service-fabric-health-introduction.md#health-policies) (una combinación del estado del nodo y el mantenimiento de todas las aplicaciones que se ejecutan en el clúster) se cumplen mientras dura la actualización. Si no se cumplen las directivas de mantenimiento del clúster, la actualización se revierte.

Una vez que se hayan solucionado los problemas que provocó la reversión, deberá iniciar la actualización de nuevo, siguiendo los mismos pasos que antes.

#### <a name="automatic-upgrade-with-wave-deployment"></a>Actualización automática con implementación por oleadas

Para configurar las actualizaciones automáticas y la implementación por oleadas solo tiene que agregar o comprobar que `ClusterUpgradeMode` está establecido en `Automatic` y que la propiedad `upgradeWave` está definida con uno de los valores de oleada enumerados anteriormente en la plantilla de Resource Manager.

```json
{
"apiVersion": "2021-05-01",
"type": "Microsoft.ServiceFabric/managedClusters",
"properties": {
        "ClusterUpgradeMode": "Automatic",
        "upgradeWave": "Wave1",
        }  
}
```

Una vez que implemente la plantilla actualizada, el clúster se inscribirá en la oleada especificada en el siguiente período de actualización y con posterioridad a este.

## <a name="custom-policies-for-manual-upgrades"></a>Directivas personalizadas para actualizaciones manuales

Puede especificar directivas de mantenimiento personalizadas para las actualizaciones manuales del clúster. Estas directivas se aplican cada vez que se selecciona una nueva versión de tiempo de ejecución, que hace que el sistema inicie la actualización del clúster. Si no reemplaza las directivas, se usarán los valores predeterminados.

Puede especificar las directivas de mantenimiento personalizadas o revisar la configuración actual en la sección **Actualizaciones de tejido** del recurso de clúster en Azure Portal; para ello, seleccione la opción *Personalizada* de **Directiva de actualización**.

:::image type="content" source="./media/service-fabric-cluster-upgrade/custom-upgrade-policy.png" alt-text="Selección de la opción de directiva de actualización &quot;Personalizada&quot; en la sección &quot;Actualizaciones de tejido&quot; del recurso de clúster en Azure Portal para establecer directivas de mantenimiento personalizadas durante la actualización":::

## <a name="query-for-supported-cluster-versions"></a>Consulta de las versiones de clúster compatibles

Puede usar la [API REST de Azure](/rest/api/azure/) para enumerar todas las versiones del entorno de ejecución de Service Fabric ([clusterVersions](/rest/api/servicefabric/sfrp-api-clusterversions_list)) disponibles para la ubicación y la suscripción especificadas.

También puede consultar las [versiones de Service Fabric](service-fabric-versions.md) para más información sobre las versiones y los sistemas operativos compatibles.

```REST
GET https://<endpoint>/subscriptions/{{subscriptionId}}/providers/Microsoft.ServiceFabric/locations/{{location}}/clusterVersions?api-version=2018-02-01

"value": [
  {
    "id": "subscriptions/########-####-####-####-############/providers/Microsoft.ServiceFabric/environments/Windows/clusterVersions/5.0.1427.9490",
    "name": "5.0.1427.9490",
    "type": "Microsoft.ServiceFabric/environments/clusterVersions",
    "properties": {
      "codeVersion": "5.0.1427.9490",
      "supportExpiryUtc": "2016-11-26T23:59:59.9999999",
      "environment": "Windows"
    }
  },
  {
    "id": "subscriptions/########-####-####-####-############/providers/Microsoft.ServiceFabric/environments/Windows/clusterVersions/4.0.1427.9490",
    "name": "5.1.1427.9490",
    "type": " Microsoft.ServiceFabric/environments/clusterVersions",
    "properties": {
      "codeVersion": "5.1.1427.9490",
      "supportExpiryUtc": "9999-12-31T23:59:59.9999999",
      "environment": "Windows"
    }
  },
  {
    "id": "subscriptions/########-####-####-####-############/providers/Microsoft.ServiceFabric/environments/Windows/clusterVersions/4.4.1427.9490",
    "name": "4.4.1427.9490",
    "type": " Microsoft.ServiceFabric/environments/clusterVersions",
    "properties": {
      "codeVersion": "4.4.1427.9490",
      "supportExpiryUtc": "9999-12-31T23:59:59.9999999",
      "environment": "Linux"
    }
  }
]
}
```

`supportExpiryUtc` en la salida informa de cuando una versión determinada expira o ha expirado. La última versión no tiene una fecha válida, sino un valor de *9999-12-31T23:59:59.9999999*, lo que significa simplemente que la fecha de expiración no se ha establecido aún.

## <a name="next-steps"></a>Pasos siguientes

* [Personalización de la configuración del clúster administrado de Service Fabric](how-to-managed-cluster-configuration.md)
* Obtenga información sobre [actualizaciones de aplicaciones](service-fabric-application-upgrade.md)

<!--Image references-->
[CertificateUpgrade]: ./media/service-fabric-cluster-upgrade/CertificateUpgrade2.png
[AddingProbes]: ./media/service-fabric-cluster-upgrade/addingProbes2.PNG
[AddingLBRules]: ./media/service-fabric-cluster-upgrade/addingLBRules.png
[Upgrade-Wave-Settings]: ./media/service-fabric-cluster-upgrade/manage-upgrade-wave-settings.png
[ARMUpgradeMode]: ./media/service-fabric-cluster-upgrade/ARMUpgradeMode.PNG
[Create_Manualmode]: ./media/service-fabric-cluster-upgrade/Create_Manualmode.PNG
[Manage_Automaticmode]: ./media/service-fabric-cluster-upgrade/Manage_Automaticmode.PNG