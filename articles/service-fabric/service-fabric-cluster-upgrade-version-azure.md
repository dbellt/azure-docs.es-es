---
title: Administración de actualizaciones del clúster de Service Fabric
description: Administre cuándo y cómo se actualiza el entorno de ejecución del clúster de Service Fabric.
ms.topic: how-to
ms.date: 03/26/2021
ms.openlocfilehash: 98c3300e5cc51c32d894397839879e25190d979b
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/30/2021
ms.locfileid: "105731174"
---
# <a name="manage-service-fabric-cluster-upgrades"></a>Administración de actualizaciones del clúster de Service Fabric

Un clúster de Azure Service Fabric es un recurso de su propiedad, pero que está parcialmente administrado por Microsoft. Aquí se muestra cómo administrar cuándo y cómo Microsoft actualiza el clúster de Azure Service Fabric.

Para más información sobre los conceptos y los procesos de actualización de clústeres, consulte [Actualización de los clústeres de Azure Service Fabric](service-fabric-cluster-upgrade.md).

## <a name="set-upgrade-mode"></a>Establecimiento del modo de actualización

Puede configurar el clúster para recibir actualizaciones automáticas de Service Fabric a medida que se publican en Microsoft o elegir manualmente de una lista de versiones admitidas en la actualidad mediante la configuración del modo de actualización del clúster. Para ello, se puede usar el control *Modo de actualización de tejido* en Azure Portal o la opción `upgradeMode` de la plantilla de implementación del clúster.

### <a name="azure-portal"></a>Azure Portal

Con Azure Portal, elegirá entre actualizaciones automáticas o manuales al crear un clúster de Service Fabric.

:::image type="content" source="media/service-fabric-cluster-upgrade/portal-new-cluster-upgrade-mode.png" alt-text="Elección entre actualizaciones automáticas o manuales al crear un clúster en Azure Portal a partir de las opciones avanzadas":::

También puede alternar entre actualizaciones automáticas y manuales desde la sección **Actualizaciones de tejido** de un recurso de clúster existente.

:::image type="content" source="./media/service-fabric-cluster-upgrade/fabric-upgrade-mode.png" alt-text="Selección de actualizaciones automáticas o manuales en la sección &quot;Actualizaciones de tejido&quot; del recurso de clúster en Azure Portal.":::

### <a name="manual-upgrades-with-azure-portal"></a>Actualizaciones manuales con Azure Portal

Al seleccionar la opción de actualización manual, lo único que se necesita para iniciar una actualización es seleccionar una de las versiones disponibles del menú desplegable y, luego, hacer clic en *Guardar*. A partir de ahí, la actualización del clúster se inicia de inmediato.

Las [directivas de mantenimiento del clúster](#custom-policies-for-manual-upgrades) (una combinación del estado del nodo y el mantenimiento de todas las aplicaciones que se ejecutan en el clúster) se cumplen mientras dura la actualización. Si no se cumplen las directivas de mantenimiento del clúster, la actualización se revierte.

Una vez que se hayan solucionado los problemas que provocó la reversión, deberá iniciar la actualización de nuevo, siguiendo los mismos pasos que antes.

### <a name="resource-manager-template"></a>Plantilla de Resource Manager

Para cambiar el modo de actualización del clúster mediante una plantilla de Resource Manager, especifique *Automática* o *Manual* en la propiedad `upgradeMode` de la definición del recurso *Microsoft.ServiceFabric/clusters*. Si elige actualizaciones manuales, establezca también `clusterCodeVersion` en una [versión de tejido compatible](#query-for-supported-cluster-versions) actualmente.

:::image type="content" source="./media/service-fabric-cluster-upgrade/ARMUpgradeMode.PNG" alt-text="Captura de pantalla que muestra una plantilla, en la que se ha aplicado sangría al texto no cifrado para reflejar la estructura, y se han resaltado las propiedades &quot;clusterCodeVersion&quot; y &quot;upgradeMode&quot;.":::

Después de una implementación correcta de la plantilla, se aplicarán los cambios al modo de actualización del clúster. Si el clúster está en modo manual, la actualización del clúster se iniciará automáticamente.

Las [directivas de mantenimiento del clúster](#custom-policies-for-manual-upgrades) (una combinación del estado del nodo y el mantenimiento de todas las aplicaciones que se ejecutan en el clúster) se cumplen mientras dura la actualización. Si no se cumplen las directivas de mantenimiento del clúster, la actualización se revierte.

Una vez que se hayan solucionado los problemas que provocó la reversión, deberá iniciar la actualización de nuevo, siguiendo los mismos pasos que antes.

## <a name="wave-deployment-for-automatic-upgrades"></a>Implementación por oleadas para actualizaciones automáticas

Con el modo de actualización automática, tiene la opción de habilitar el clúster para la implementación por oleadas. Con la implementación por oleadas, puede crear una canalización para actualizar los clústeres de prueba, ensayo y producción en secuencia, separados por el "tiempo de preparación" integrado para validar las próximas versiones de Service Fabric antes de que se actualicen los clústeres de producción.

### <a name="enable-wave-deployment"></a>Habilitación de la implementación por oleadas

> [!NOTE]
> La implementación por oleadas requiere la versión de API `2020-12-01-preview` (o posterior) del recurso *Microsoft.ServiceFabric/clusters*.

Para habilitar la implementación por oleadas para la actualización automática, determine primero la oleada que asignará el clúster:

* **Oleada 0** (`Wave0`): los clústeres se actualizan en cuanto se publica una nueva compilación de Service fabric. Prevista para clústeres de desarrollo y pruebas.
* **Oleada 1** (`Wave1`): los clústeres se actualizan una semana (siete días) después de que se lance una nueva compilación. Prevista para clústeres de preproducción y ensayo.
* **Oleada 2** (`Wave2`): los clústeres se actualizan dos semanas (14 días) después de que se lance una nueva compilación. Prevista para los clústeres de producción.

A continuación, agregue simplemente una propiedad `upgradeWave` a la plantilla de recursos del clúster con uno de los valores de oleada enumerados anteriormente. Asegúrese de que la versión de API de recursos del clúster sea `2020-12-01-preview` o posterior.

```json
{
    "apiVersion": "2020-12-01-preview",
    "type": "Microsoft.ServiceFabric/clusters",
     ...
        "fabricSettings": [...],
        "managementEndpoint": ...,
        "nodeTypes": [...],
        "provisioningState": ...,
        "reliabilityLevel": ...,
        "upgradeMode": "Automatic",
        "upgradeWave": "Wave1",
       ...
```

Una vez que implemente la plantilla actualizada, el clúster se inscribirá en la oleada especificada en el siguiente período de actualización y con posterioridad a este.

Puede [registrarse para recibir notificaciones por correo electrónico](#register-for-notifications) con vínculos a ayuda adicional por si se produce un error en una actualización del clúster.

### <a name="register-for-notifications"></a>Registro de notificaciones

Puede registrarse para recibir notificaciones cuando se produzca un error en una actualización del clúster. Se enviará un correo electrónico a las direcciones de correo electrónico designadas con más detalles sobre el error de actualización y vínculos a ayuda adicional.

> [!NOTE]
> No es necesario inscribirse en la implementación por oleadas para recibir notificaciones de errores de actualización.

Para inscribirse en las notificaciones, agregue una sección `notifications` a la plantilla de recursos del clúster y designe una o varias direcciones de correo electrónico (*destinatarios*) para recibir notificaciones:

```json
    "apiVersion": "2020-12-01-preview",
    "type": "Microsoft.ServiceFabric/clusters",
     ...
        "upgradeMode": "Automatic",
        "upgradeWave": "Wave1",
        "notifications": [
        {
            "isEnabled": true,
            "notificationCategory": "WaveProgress",
            "notificationLevel": "Critical",
            "notificationTargets": [
            {
                "notificationChannel": "EmailUser",
                "receivers": [
                    "devops@contoso.com"
                ]
            }]
        }]
```

Una vez que implemente la plantilla actualizada, se le inscribirá en las notificaciones de errores de actualización.

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

* [Administración de actualizaciones de Service Fabric](service-fabric-cluster-upgrade-version-azure.md)
* [Personalización de la configuración de un clúster de Service Fabric](service-fabric-cluster-fabric-settings.md)
* [Escalar o reducir horizontalmente el clúster](service-fabric-cluster-scale-in-out.md)
* Obtenga información sobre [actualizaciones de aplicaciones](service-fabric-application-upgrade.md)


<!--Image references-->
[CertificateUpgrade]: ./media/service-fabric-cluster-upgrade/CertificateUpgrade2.png
[AddingProbes]: ./media/service-fabric-cluster-upgrade/addingProbes2.PNG
[AddingLBRules]: ./media/service-fabric-cluster-upgrade/addingLBRules.png
[HealthPolices]: ./media/service-fabric-cluster-upgrade/Manage_AutomodeWadvSettings.PNG
[ARMUpgradeMode]: ./media/service-fabric-cluster-upgrade/ARMUpgradeMode.PNG
[Create_Manualmode]: ./media/service-fabric-cluster-upgrade/Create_Manualmode.PNG
[Manage_Automaticmode]: ./media/service-fabric-cluster-upgrade/Manage_Automaticmode.PNG
