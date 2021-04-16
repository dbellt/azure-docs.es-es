---
title: Intercambio o cambio entre dos instancias de Azure Cloud Services (soporte extendido)
description: Intercambio o cambio entre dos instancias de Azure Cloud Services (soporte extendido)
ms.topic: how-to
ms.service: cloud-services-extended-support
author: surbhijain
ms.author: surbhijain
ms.reviewer: gachandw
ms.date: 04/01/2021
ms.custom: ''
ms.openlocfilehash: 6f96656af9afd9874cc6273a9cea9ed43e8c69cc
ms.sourcegitcommit: af6eba1485e6fd99eed39e507896472fa930df4d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/04/2021
ms.locfileid: "106294255"
---
# <a name="swapswitch-between-two-azure-cloud-services-extended-support"></a>Intercambio o cambio entre dos instancias de Azure Cloud Services (soporte extendido)
Con Cloud Services (soporte extendido) puede realizar el intercambio de dos implementaciones de servicios en la nube independientes. A diferencia de Cloud Services (clásico), el concepto de ranuras no existe con el modelo de Azure Resource Manager. Cuando decida implementar una nueva versión de un servicio en la nube (soporte extendido), puede hacer que se pueda intercambiar con otro servicio en la nube existente (soporte extendido), lo que le permita ensayar y probar la nueva versión con esta implementación. El intercambio de servicios en la nube solo se puede realizar en el momento de la implementación del segundo de ellos (del par). Si se usa el método de implementación basado en plantilla de Resource Manager, para hacerlo es preciso establecer en la propiedad SwappableCloudService del perfil de red del objeto servicio en la nube el identificador del servicio en la nube emparejado. 

```
"networkProfile": {
 "SwappableCloudService": {
              "id": "[concat(variables('swappableResourcePrefix'), 'Microsoft.Compute/cloudServices/', parameters('cloudServicesToBeSwappedWith'))]"
            },
```
> [!Note] 
> No se puede realizar el intercambio entre un servicio en la nube (clásico) y un servicio en la nube (soporte extendido)

Use **Intercambiar** para cambiar las direcciones URL por las que se direccionan los dos servicios en la nube, lo promociona un nuevo servicio en la nube (almacenado provisionalmente) a la versión de producción.
Puede intercambiar implementaciones desde la página Cloud Services o el panel.

1. En el [Portal de Azure](https://portal.azure.com), seleccione el servicio en la nube que desee actualizar. Tras esta acción, se abrirá la hoja de la instancia del servicio en la nube.
2. En la hoja, seleccione **Intercambiar**
   :::image type="content" source="media/swap-cloud-service-1.png" alt-text="La imagen muestra la opción de intercambio del servicio en la nube":::
   
3. Se abre la siguiente solicitud de confirmación
   
   :::image type="content" source="media/swap-cloud-service-2.png" alt-text="La imagen muestra el intercambio del servicio en la nube":::
   
4. Después de verificar la información de la implementación, seleccione Aceptar para intercambiar las implementaciones.
El intercambio se produce rápidamente porque lo único que cambia son las direcciones IP virtuales (VIP) de los dos servicios en la nube.

Para ahorrar costos de proceso, puede eliminar uno de los servicios en la nube (designado como entorno de ensayo para la implementación de la aplicación) después de comprobar que el servicio en la nube intercambiado funciona como cabría esperar.

A continuación se muestra la API REST para realizar un "intercambio" entre dos implementaciones de servicios en la nube del tipo soporte extendido:
```http
POST https://management.azure.com/subscriptions/subId/providers/Microsoft.Network/locations/region/setLoadBalancerFrontendPublicIpAddresses?api-version=2020-11-01
```
```
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
```
## <a name="common-questions-about-swapping-deployments"></a>Preguntas comunes sobre el intercambio de implementaciones

### <a name="what-are-the-prerequisites-for-swapping-between-two-cloud-services"></a>¿Cuáles son los requisitos previos para el intercambio entre dos servicios en la nube?
Hay dos requisitos previos clave para que un intercambio de servicios en la nube (soporte extendido) sea correcto:
* Si desea usar una dirección IP estática o reservada para uno de los servicios en la nube que se pueden intercambiar, el otro también debe usar una IP reservada. Si no lo hace así, se producirá un error en el intercambio.
* Todas las instancias de los roles se deben estar ejecutando para poder realizar el intercambio. El estado de las instancias se puede comprobar en la hoja Información general de Azure Portal. Como alternativa, puede usar el comando Get-AzRole en Windows PowerShell.

Las actualizaciones del sistema operativo invitado y las operaciones de recuperación de servicios también pueden provocar que no se puedan realizar intercambios de implementación. Para más información, consulte Solución de problemas de implementación de servicios en la nube.

### <a name="can-i-perform-a-vip-swap-in-parallel-with-another-mutating-operation"></a>¿Puedo realizar un intercambio de IP virtual en paralelo con otra operación de mutación?
No. El intercambio de IP virtual es un cambio solo de red que debe completarse antes de realizar cualquier otra operación de proceso en los servicios en la nube. La realización de una operación de actualización, eliminación o escalabilidad automática en los servicios en la nube mientras un intercambio de IP virtual está en curso o el desencadenamiento de un intercambio de IP virtual mientras otra operación de proceso está en curso pueden dejar el servicio en la nube en un estado no deseado desde el que la recuperación podría no ser posible. 

### <a name="does-a-swap-incur-downtime-for-my-application-how-should-i-handle-it"></a>¿Un intercambio conlleva tiempo de inactividad de mi aplicación? ¿Cómo puedo controlarlo?
Como se ha descrito en la sección anterior, el intercambio de dos servicios en la nube suele ser rápido, ya que no es más que un cambio de configuración en Azure Load Balancer. En algunos casos, puede tardar diez segundos o más, y dar lugar a errores de conexión transitorios. Para limitar el impacto en los clientes, considere la posibilidad de implementar la lógica de reintento del cliente.

## <a name="next-steps"></a>Pasos siguientes 
- Revise los [requisitos previos de implementación](deploy-prerequisite.md) de Cloud Services (soporte extendido).
- Vea las [preguntas más frecuentes](faq.md) sobre Cloud Services (soporte extendido).
- Implemente una instancia de Cloud Services (soporte extendido) mediante [Azure Portal](deploy-portal.md), [PowerShell](deploy-powershell.md), una [plantilla](deploy-template.md) o [Visual Studio](deploy-visual-studio.md).
