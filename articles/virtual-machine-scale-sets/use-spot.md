---
title: Creación de un conjunto de escalado que use máquinas virtuales de acceso puntual de Azure
description: Aprenda a crear conjuntos de escalado de Azure que usen máquinas virtuales de acceso puntual para ahorrar costos.
author: JagVeerappan
ms.author: jagaveer
ms.topic: how-to
ms.service: virtual-machine-scale-sets
ms.subservice: spot
ms.date: 02/26/2021
ms.reviewer: cynthn
ms.custom: devx-track-azurecli, devx-track-azurepowershell
ms.openlocfilehash: 61bb87d84b96f988ae065a70b85d445fc8b96ccf
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107762954"
---
# <a name="azure-spot-virtual-machines-for-virtual-machine-scale-sets"></a>Máquinas virtuales de acceso puntual de Azure para conjuntos de escalado 

El uso de máquinas virtuales de acceso puntual de Azure en conjuntos de escalado permite aprovechar las ventajas de nuestra capacidad no utilizada con un importante ahorro en los costos. Siempre que Azure necesite recuperar la capacidad, la infraestructura de esta plataforma expulsará las instancias de máquinas virtuales de acceso puntual de Azure. Por lo tanto, las instancias de máquinas virtuales de acceso puntual de Azure son excelentes para cargas de trabajo que puedan soportar interrupciones, como los trabajos de procesamiento por lotes, los entornos de desarrollo y pruebas, las cargas de trabajo de proceso de gran tamaño, etc.

La cantidad de capacidad sin usar disponible varía, por ejemplo, en función del tamaño, la región o la hora del día. Al implementar instancias de máquinas virtuales de acceso puntual en los conjuntos de escalado, Azure asigna las instancias solo si hay capacidad disponible, aunque estas no tienen un Acuerdo de Nivel de Servicio. Un conjunto de escalado de máquinas virtuales de acceso puntual se implementa en un dominio de error único y no ofrece garantías de alta disponibilidad.


## <a name="pricing"></a>Precios

Los precios de las instancias de máquinas virtuales de acceso puntual de Azure son variables, en función de la región y la SKU. Para más información, consulte los precios para [Linux](https://azure.microsoft.com/pricing/details/virtual-machine-scale-sets/linux/) y [Windows](https://azure.microsoft.com/pricing/details/virtual-machine-scale-sets/windows/). 


La variabilidad en los precios permite establecer un precio máximo, en dólares estadounidenses (USD), con un máximo de 5 decimales. Por ejemplo, el valor `0.98765` correspondería a un precio máximo de 0,98765 USD por hora. Si establece el precio máximo en `-1`, la instancia no se expulsará según el precio. El precio de la instancia será el precio actual de la máquina virtual de acceso puntual de Azure o el de una instancia estándar, el que sea menor de los dos, siempre que haya capacidad y cuota disponibles.


## <a name="limitations"></a>Limitaciones

Los siguientes tamaños no se admiten en las máquinas virtuales de acceso puntual de Azure:
 - Serie B
 - Versiones de promoción de cualquier tamaño (como los tamaños de promoción Dv2, NV, NC, H)

Las máquinas virtuales de acceso puntual de Azure se pueden implementar en cualquier región, excepto en Microsoft Azure China 21Vianet.

<a name="channel"></a>

Actualmente se admiten los siguientes [tipos de ofertas](https://azure.microsoft.com/support/legal/offer-details/):

-   Contrato Enterprise
-   Código de oferta de pago por uso (003P)
-   Patrocinado (0036P y 0136P)
- Para el proveedor de servicios en la nube (CSP), consulte el [Centro de partners](/partner-center/azure-plan-get-started) o contacte con su partner directamente.

## <a name="eviction-policy"></a>Directiva de expulsión

Al crear un conjunto de escalado mediante máquinas virtuales de acceso puntual de Azure, puede establecer la directiva de expulsión en *Desasignar* (opción predeterminada) o *Eliminar*. 

La directiva *Deallocate* (Desasignar) cambia las instancias expulsadas al estado stopped-deallocated para que pueda volver a implementarlas. Sin embargo, no hay ninguna garantía de que la asignación se realizará correctamente. Las máquinas virtuales desasignadas se siguen teniendo en cuenta en la cuota de instancias del conjunto de escalado y se le cobra por los discos subyacentes. 

Si quiere que las instancias se eliminen al expulsarse, puede establecer la directiva de expulsión en *Eliminar*. Con la directiva de expulsión establecida para eliminar, puede crear nuevas máquinas virtuales mediante el aumento de la propiedad de recuento de instancias del conjunto de escalado. Las máquinas virtuales expulsadas se eliminan junto con sus discos subyacentes y, por tanto, no se le cobra el almacenamiento. También puede usar la característica de escalado automático de los conjuntos de escalado para probar y compensar automáticamente las máquinas virtuales expulsadas; sin embargo, no hay ninguna garantía de que la asignación se realice correctamente. Con el fin de evitar el costo de los discos y alcanzar los límites de cuota, se recomienda usar únicamente la característica de escalado automático en los conjuntos de escalado de Spot al establecer la directiva de expulsión en Delete (Eliminar). 

Los usuarios pueden optar por recibir notificaciones en las máquinas virtuales mediante [Azure Scheduled Events](../virtual-machines/linux/scheduled-events.md). De este modo se le notificará que se van a expulsar las máquinas virtuales y tendrá 30 segundos para terminar los trabajos y cerrar las tareas antes de que esto ocurra. 

<a name="bkmk_try"></a>
## <a name="try--restore-preview"></a>Prueba y restauración (versión preliminar)

Esta nueva característica de nivel de plataforma usará IA para intentar restaurar automáticamente las instancias de máquina virtual de acceso puntual de Azure expulsadas dentro de un conjunto de escalado para mantener el recuento de instancias de destino. 

> [!IMPORTANT]
> La característica Prueba y restauración se encuentra en versión preliminar pública.
> Esta versión preliminar se ofrece sin Acuerdo de Nivel de Servicio y no se recomienda para cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas. Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Ventajas de Prueba y restauración:
- Intenta restaurar las máquinas virtuales de acceso puntual de Azure expulsadas por motivos de capacidad.
- Se espera que las máquinas virtuales de acceso puntual de Azure restauradas se ejecuten durante más tiempo con menos probabilidad de que se produzca una expulsión desencadenada por la capacidad.
- Mejora la duración de una máquina virtual de acceso puntual de Azure, por lo que las cargas de trabajo se ejecutan durante más tiempo.
- Permite que los conjuntos de escalado de máquinas virtuales conserven el recuento de destinos de las máquinas virtuales de acceso puntual de Azure, de la misma forma que la característica que sirve para conservar el recuento de destinos que ya existen para las máquinas virtuales de pago por uso.

La característica Prueba y restauración está deshabilitada en los conjuntos de escalado en los que se usa [Escalado automático](virtual-machine-scale-sets-autoscale-overview.md). El número de máquinas virtuales del conjunto de escalado está controlado por las reglas de escalado automático.

### <a name="register-for-try--restore"></a>Registro para Prueba y restauración

Antes de poder usar la característica Prueba y restauración, debe registrar la suscripción para la versión preliminar. El registro puede tardar varios minutos en terminar. Puede usar la CLI de Azure o PowerShell para completar el registro de la característica.


**Uso de la CLI**

Utilice [az feature register](/cli/azure/feature#az_feature_register) para habilitar la versión preliminar de su suscripción. 

```azurecli-interactive
az feature register --namespace Microsoft.Compute --name SpotTryRestore 
```

El registro de la característica puede tardar hasta 15 minutos. Para comprobar el estado del registro, siga estos pasos: 

```azurecli-interactive
az feature show --namespace Microsoft.Compute --name SpotTryRestore 
```

Una vez que la característica se ha registrado para su suscripción, complete el proceso de participación mediante la propagación del cambio en el proveedor de recursos de Compute. 

```azurecli-interactive
az provider register --namespace Microsoft.Compute 
```
**Uso de PowerShell** 

Use el cmdlet [Register-AzProviderFeature](/powershell/module/az.resources/register-azproviderfeature) para habilitar la versión preliminar de su suscripción. 

```azurepowershell-interactive
Register-AzProviderFeature -FeatureName SpotTryRestore -ProviderNamespace Microsoft.Compute 
```

El registro de la característica puede tardar hasta 15 minutos. Para comprobar el estado del registro, siga estos pasos: 

```azurepowershell-interactive
Get-AzProviderFeature -FeatureName SpotTryRestore -ProviderNamespace Microsoft.Compute 
```

Una vez que la característica se ha registrado para su suscripción, complete el proceso de participación mediante la propagación del cambio en el proveedor de recursos de Compute. 

```azurepowershell-interactive
Register-AzResourceProvider -ProviderNamespace Microsoft.Compute 
```

## <a name="placement-groups"></a>Grupos de selección de ubicación

Un grupo de selección de ubicación es una construcción similar a un conjunto de disponibilidad de Azure, con sus propios dominios de error y dominios de actualización. De forma predeterminada, un conjunto de escalado consta de un único grupo de selección de ubicación con un tamaño máximo de 100 máquinas virtuales. Si la propiedad `singlePlacementGroup` de un conjunto de escalado se establece en *false*, el conjunto de escalado puede estar compuesto por varios grupos de selección de ubicación y tiene un intervalo de 0 a 1000 máquinas virtuales. 

> [!IMPORTANT]
> A menos que use InfiniBand con HPC, se recomienda establecer la propiedad del conjunto de escalado `singlePlacementGroup` en *false* para habilitar varios grupos de selección de ubicación para un mejor escalado en la región o zona. 

## <a name="deploying-azure-spot-virtual-machines-in-scale-sets"></a>Implementación de máquinas virtuales de acceso puntual de Azure en conjuntos de escalado

Para implementar máquinas virtuales de acceso puntual en conjuntos de escalado, puede establecer la nueva marca *Priority* (Prioridad) en *Spot* (Acceso puntual). Todas las máquinas virtuales del conjunto de escalado se establecerán para Spot. Para crear un conjunto de escalado con máquinas virtuales de acceso puntual de Azure, use uno de los métodos siguientes:
- [Azure Portal](#portal)
- [CLI de Azure](#azure-cli)
- [Azure PowerShell](#powershell)
- [Plantillas del Administrador de recursos de Azure](#resource-manager-templates)

## <a name="portal"></a>Portal

El proceso para crear un conjunto de escalado que use máquinas virtuales de acceso puntual de Azure es igual que el que se detalla en el [artículo de introducción](quick-create-portal.md). Cuando vaya a implementar un conjunto de escalado, puede elegir establecer la marca de Spot y la directiva de expulsión: ![Creación de un conjunto de escalado con máquinas virtuales de acceso puntual de Azure](media/virtual-machine-scale-sets-use-spot/vmss-spot-portal-max-price.png)


## <a name="azure-cli"></a>Azure CLI

El proceso para crear un conjunto de escalado con máquinas virtuales de acceso puntual de Azure es igual que el que se detalla en el [artículo de introducción](quick-create-cli.md). Basta con agregar "--Priority Spot" y `--max-price`. En este ejemplo, usamos `-1` para `--max-price`, por lo que la instancia no se expulsará según el precio.

```azurecli
az vmss create \
    --resource-group myResourceGroup \
    --name myScaleSet \
    --image UbuntuLTS \
    --upgrade-policy-mode automatic \
    --single-placement-group false \
    --admin-username azureuser \
    --generate-ssh-keys \
    --priority Spot \
    --max-price -1 
```

## <a name="powershell"></a>PowerShell

El proceso para crear un conjunto de escalado con máquinas virtuales de acceso puntual de Azure es igual que el que se detalla en el [artículo de introducción](quick-create-powershell.md).
Basta con agregar "-Priority Spot" y proporcionar un valor de `-max-price` a [New-AzVmssConfig](/powershell/module/az.compute/new-azvmssconfig).

```powershell
$vmssConfig = New-AzVmssConfig `
    -Location "East US 2" `
    -SkuCapacity 2 `
    -SkuName "Standard_DS2" `
    -UpgradePolicyMode Automatic `
    -Priority "Spot" `
    -max-price -1
```

## <a name="resource-manager-templates"></a>Plantillas de Resource Manager

El proceso para crear un conjunto de escalado que use máquinas virtuales de acceso puntual de Azure es igual que el que se detalla en el artículo de introducción para [Linux](quick-create-template-linux.md) o [Windows](quick-create-template-windows.md). 

Para implementaciones de plantilla de máquina virtual de acceso puntual de Azure, use `"apiVersion": "2019-03-01"` o posterior. 

Agregue las propiedades `priority`, `evictionPolicy` y `billingProfile` a la sección `"virtualMachineProfile":` y la propiedad `"singlePlacementGroup": false,` a la sección `"Microsoft.Compute/virtualMachineScaleSets"` de la plantilla:

```json

{
  "type": "Microsoft.Compute/virtualMachineScaleSets",
  },
  "properties": {
    "singlePlacementGroup": false,
    }

        "virtualMachineProfile": {
              "priority": "Spot",
                "evictionPolicy": "Deallocate",
                "billingProfile": {
                    "maxPrice": -1
                }
            },
```

Para eliminar la instancia después de que se haya expulsado, cambie el parámetro `evictionPolicy` a `Delete`.


## <a name="simulate-an-eviction"></a>Simulación de una expulsión

Puede [simular una expulsión](/rest/api/compute/virtualmachines/simulateeviction) de una máquina virtual de acceso puntual de Azure para probar la capacidad de respuesta de una aplicación ante una expulsión repentina. 

Reemplazar lo siguiente por su propia información: 

- `subscriptionId`
- `resourceGroupName`
- `vmName`


```rest
POST https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/virtualMachines/{vmName}/simulateEviction?api-version=2020-06-01
```

`Response Code: 204` significa que la expulsión simulada se ha realizado correctamente. 

## <a name="faq"></a>Preguntas más frecuentes

**P:** Una vez que se crea, ¿es lo mismo una instancia de máquina virtual de acceso puntual de Azure que una instancia estándar?

**R:** Sí, salvo que no hay ningún Acuerdo de Nivel de Servicio para las máquinas virtuales de acceso puntual y se pueden expulsar en cualquier momento.


**P:** ¿Qué se debe hacer si se produce la expulsión pero aún se necesita capacidad?

**R:** Si necesita capacidad de inmediato, se recomienda usar máquinas virtuales estándar en lugar de máquinas virtuales de acceso puntual.


**P:** ¿Cómo se administra la cuota de las máquinas virtuales de acceso puntual de Azure?

**R:** Las instancias de máquinas virtuales de acceso puntual de Azure y las instancias estándar tendrán grupos de cuotas independientes. La cuota de máquinas virtuales de acceso puntual de Azure se compartirá entre las instancias de máquinas virtuales y de conjuntos de escalado. Para más información, consulte [Límites, cuotas y restricciones de suscripción y servicios de Microsoft Azure](../azure-resource-manager/management/azure-subscription-service-limits.md).


**P:** ¿Puedo solicitar cuota adicional para las máquinas virtuales de acceso puntual de Azure?

**R:** Sí, podrá enviar la solicitud para aumentar su cuota en las máquinas virtuales de acceso puntual de Azure mediante el [proceso de solicitud de cuota estándar](../azure-portal/supportability/per-vm-quota-requests.md).


**P:** ¿Puedo convertir los conjuntos de escalado existentes en conjuntos de escalado de máquinas virtuales de acceso puntual de Azure?

**R:** No, solo se permite establecer la marca `Spot` durante la creación.


**P:** Si uso `low` para los conjuntos de escalado de prioridad baja, ¿necesito empezar a usar `Spot` en su lugar?

**R:** Por ahora, `low` y `Spot` funcionarán, pero debe empezar a usar `Spot`.


**P:** ¿Puedo crear un conjunto de escalado con máquinas virtuales normales y máquinas virtuales de acceso puntual de Azure?

**R:** No, un conjunto de escalado no admite más de un tipo de prioridad.


**P:**  ¿Puedo usar la escalabilidad automática con conjuntos de escalado de máquinas virtuales de acceso puntual de Azure?

**R:** Sí, en el conjunto de escalado de máquinas virtuales de acceso puntual de Azure puede establecer reglas de escalado automático. Si se expulsan las máquinas virtuales, la escalabilidad automática puede intentar crear nuevas máquinas virtuales de acceso puntual de Azure. Recuerde que, aun así, no se garantiza esta capacidad. 


**P:**  ¿Funciona el escalado automático con las dos directivas de expulsión (desasignar y eliminar)?

**R:** Sí. No obstante, se recomienda establecer la directiva de expulsión en "Eliminar" cuando se use la escalabilidad automática. Esto es porque las instancias desasignadas se cuentan para el número de capacidad en el conjunto de escalado. Cuando se usa el escalado automático, es probable que alcance el número de instancias de destino rápidamente debido a las instancias expulsadas desasignadas. Además, las operaciones de escalado pueden verse afectadas por expulsiones puntuales. Por ejemplo, las instancias de conjuntos de escalado de máquinas virtuales podrían situarse por debajo del recuento mínimo establecido debido a varias expulsiones puntuales durante las operaciones de escalado. 


**P:** ¿Dónde puedo publicar preguntas?

**R:** Puede publicar y etiquetar la pregunta con `azure-spot` en [Q&A](/answers/topics/azure-spot.html) (Preguntas y respuestas). 

## <a name="next-steps"></a>Pasos siguientes

Consulte la [página de precios del conjunto de escalado de máquinas virtuales](https://azure.microsoft.com/pricing/details/virtual-machine-scale-sets/linux/) para ver información de precios.
