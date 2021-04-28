---
title: Cambio de una implementación a otra en Azure Cloud Services (soporte extendido)
description: Aprenda a cambiar de una implementación a otra en Azure Cloud Services (soporte extendido).
ms.topic: how-to
ms.service: cloud-services-extended-support
author: surbhijain
ms.author: surbhijain
ms.reviewer: gachandw
ms.date: 04/01/2021
ms.custom: ''
ms.openlocfilehash: f5e01075ffb460c7ddd70b40a6b19f7ea70dd776
ms.sourcegitcommit: 6686a3d8d8b7c8a582d6c40b60232a33798067be
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107748845"
---
# <a name="swap-or-switch-deployments-in-azure-cloud-services-extended-support"></a>Cambio de una implementación a otra en Azure Cloud Services (soporte extendido)

En Azure Cloud Services (soporte extendido) puede intercambiar dos implementaciones de servicios en la nube independientes. A diferencia de Azure Cloud Services (clásico), el modelo Azure Resource Manager en Azure Cloud Services (soporte extendido) no usa ranuras de implementación. En Azure Cloud Services (soporte extendido), al implementar una nueva versión de un servicio en la nube, se puede hacer que el servicio en la nube pueda "intercambiarse" con un servicio en la nube existente en Azure Cloud Services (soporte extendido).

Después de intercambiar las implementaciones, puede agregar al "stage" y probar la nueva versión mediante la implementación del nuevo servicio en la nube. En efecto, el intercambio promueve un nuevo servicio en la nube que se ha pasado a la versión de producción.

> [!NOTE]
> No se puede cambiar de una implementación mediante Azure Cloud Services (clásica) a una implementación mediante Azure Cloud Services (soporte extendido).

Si implementa el segundo de un par de servicios en la nube, es preciso que uno de ellos no se pueda cambiar por el segundo.

Para cambiar de una implementación a otra puede usar una plantilla de Azure Resource Manager, Azure Portal o la API REST.

## <a name="arm-template"></a>Plantilla ARM

Si usa el método de la implementación mediante una plantilla de Resource Manager, para que los servicios en la nube se puedan intercambiar, establezca la propiedad `SwappableCloudService` en `networkProfile` del objeto `cloudServices` en el identificador del servicio en la nube emparejado:

```json
"networkProfile": {
 "SwappableCloudService": {
              "id": "[concat(variables('swappableResourcePrefix'), 'Microsoft.Compute/cloudServices/', parameters('cloudServicesToBeSwappedWith'))]"
            },
        }
```

## <a name="azure-portal"></a>Azure Portal

Para cambiar de una implementación a otra en Azure Portal:

1. En el menú del portal, seleccione **Cloud Services (soporte extendido)** o **Panel**.
1. Seleccione el servicio en la nube que desee migrar.
1. En la hoja **Información general** del servicio en la nube, seleccione **Intercambiar**:

   :::image type="content" source="media/swap-cloud-service-portal-swap.png" alt-text="Captura de pantalla que muestra la pestaña Intercambiar del servicio en la nube.":::

1. En el panel de confirmación del intercambio, compruebe la información de la implementación y seleccione **Aceptar** para cambiar una implementación por otra:

   :::image type="content" source="media/swap-cloud-service-portal-confirm.png" alt-text="Captura de pantalla que muestra la confirmación de la información de cambio de implementación.":::

Las implementaciones se intercambian rápidamente porque lo único que cambia es la dirección IP virtual del servicio en la nube que se implementa.

Para ahorrar costos de proceso, puede eliminar uno de los servicios en la nube (designado como entorno de ensayo para la implementación de la aplicación) después de comprobar que el servicio en la nube intercambiado funciona como cabría esperar.

## <a name="rest-api"></a>API REST

Para usar la API REST para cambiar a una nueva implementación de servicios en la nube en Azure Cloud Services (soporte extendido), use el siguiente comando y la configuración de JSON:

```http
POST https://management.azure.com/subscriptions/subId/providers/Microsoft.Network/locations/region/setLoadBalancerFrontendPublicIpAddresses?api-version=2020-11-01
```

```json
{
  "frontendIPConfigurations": [
    {
    "id": "#LBFE1#",
    "properties": {
    "publicIPAddress": {
    "id": "#PIP2#"
    }
      }
    },
   {
    "id": "#LBFE2#",
    "properties": {
    "publicIPAddress": {
    "id": "#PIP1#"
     }
       }
    }
  ]
 }
}
```

## <a name="common-questions-about-swapping-deployments"></a>Preguntas comunes sobre el intercambio de implementaciones

Consulte estas respuestas a preguntas comunes sobre los intercambios de implementaciones en Azure Cloud Services (soporte extendido).

### <a name="what-are-the-prerequisites-for-swapping-to-a-new-cloud-services-deployment"></a>¿Cuáles son los requisitos previos para cambiar a una nueva implementación de servicios en la nube?

Para que realizar un cambio correcto a otra implementación en Azure Cloud Services (soporte extendido), es preciso cumplir dos requisitos previos clave:

* Si desea usar una dirección IP estática o reservada para uno de los servicios en la nube intercambiables, el otro servicio en la nube también debe usar una dirección IP reservada. Si no lo hace así, se producirá un error en el intercambio.
* Para que el intercambio se realice correctamente, todas las instancias de los roles deben estar en ejecución. Para comprobar el estado de las instancias, en Azure Portal, vaya a **Información general** en el servicio en la nube recién implementado, o bien use el comando `Get-AzRole` en Windows PowerShell.

Las actualizaciones del sistema operativo invitado y las operaciones de recuperación del servicio también provocar que no se puedan realizar un cambio de una implementación por otra. Para más información, consulte [Solución de problemas de implementación de Azure Cloud Services](../cloud-services/cloud-services-troubleshoot-deployment-problems.md).

### <a name="can-i-make-a-vip-swap-in-parallel-with-another-mutating-operation"></a>¿Puedo realizar un intercambio de VIP en paralelo con otra operación de mutación?

No. Un intercambio de VIP es un cambio de solo red que debe finalizar antes de iniciar cualquier otra operación de proceso en un servicio en la nube. El inicio de una operación de actualización, eliminación o escalabilidad automática para un servicio en la nube mientras un intercambio de VIP está en curso o el desencadenamiento de un intercambio de VIP mientras hay otra operación de proceso en curso podría poner el servicio en la nube en un estado de error irrecuperable.

### <a name="does-a-swap-incur-downtime-for-my-application-and-how-should-i-handle-it"></a>¿Un intercambio conlleva tiempo de inactividad en mi aplicación?, ¿cómo puedo controlarlo?

Los intercambios de servicios en la nube normalmente son rápidos, ya que son solo un cambio en la configuración de Azure Load Balancer. En algunos casos, el intercambio puede tardar 10 segundos, o más, y provocar errores de conexión transitorios. Para limitar el efecto del intercambio en los usuarios, considere la posibilidad de implementar la lógica de reintento de cliente.

## <a name="next-steps"></a>Pasos siguientes 

* Consulte los [requisitos previos de implementación](deploy-prerequisite.md) de Azure Cloud Services (soporte extendido).
* Consulte las [preguntas más frecuentes](faq.md) sobre Azure Cloud Services (soporte extendido).
* Implemente un servicio en la nube de Azure Cloud Services en la nube (soporte extendido) mediante una de estas opciones:
  * [Azure Portal](deploy-portal.md)
  * [PowerShell](deploy-powershell.md)
  * [Plantilla ARM](deploy-template.md)
  * [Visual Studio](deploy-visual-studio.md)
