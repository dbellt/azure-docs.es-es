---
title: Ventaja híbrida de Azure y VM Linux
description: Descubra cómo la Ventaja híbrida de Azure le permite ahorrar dinero en máquinas virtuales Linux que se ejecutan en Azure.
services: virtual-machines
documentationcenter: ''
author: mathapli
manager: rochakm
ms.service: virtual-machines
ms.subservice: azure-hybrid-benefit
ms.collection: linux
ms.topic: conceptual
ms.workload: infrastructure-services
ms.date: 09/22/2020
ms.author: mathapli
ms.openlocfilehash: d21d53476c82c089fd160462093e45a8da1609c7
ms.sourcegitcommit: 49bd8e68bd1aff789766c24b91f957f6b4bf5a9b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/29/2021
ms.locfileid: "108229452"
---
# <a name="how-azure-hybrid-benefit-applies-for-linux-virtual-machines"></a>Aplicación de la Ventaja híbrida de Azure en máquinas virtuales Linux

La Ventaja híbrida de Azure es una ventaja de licencia que ayuda a disminuir considerablemente los costos de ejecutar máquinas virtuales (VM) Red Hat Enterprise Linux (RHEL) y SUSE Linux Enterprise Server (SLES) en la nube. Con esta ventaja, solo paga los costos de infraestructura de la VM, ya que el precio del software incluye la suscripción de RHEL o SLES. La ventaja está disponible para todas las imágenes de pago por uso (PAYG) de Marketplace de RHEL y SLES.

La Ventaja híbrida de Azure para máquinas virtuales Linux ya está disponible para el público.

## <a name="benefit-description"></a>Descripción de la ventaja

A través de la Ventaja híbrida de Azure, puede migrar los servidores locales RHEL y SLES a Azure mediante la conversión de las VM PAYG de RHEL y SLES existentes en Azure en facturación de Traiga su propia suscripción (BYOS). Por lo general, las VM implementadas a partir de imágenes PAYG en Azure cobrarán tanto un precio de infraestructura como un precio de software. Con la Ventaja híbrida de Azure, las VM PAYG se pueden convertir en un modelo de facturación BYOS sin necesidad de una reimplementación, lo que evita todo riesgo de tiempo de inactividad.

:::image type="content" source="./media/ahb-linux/azure-hybrid-benefit-cost.png" alt-text="Visualización de los costos de la Ventaja híbrida de Azure en máquinas virtuales de Linux.":::

Después de habilitar la ventaja en la VM de RHEL o SLES, ya no se le cobrará el precio de software adicional en el que incurre habitualmente una VM de PAYG. En su lugar, la VM empezará a acumular un cargo de BYOS, que incluye solo el precio de hardware de proceso y ningún precio de software.

También puede optar por convertir una VM que tenga habilitada la ventaja nuevamente en un modelo de facturación PAYG.

## <a name="scope-of-azure-hybrid-benefit-eligibility-for-linux-vms"></a>Ámbito de la idoneidad de Ventaja híbrida de Azure para VM Linux

La Ventaja híbrida de Azure está disponible para todas las imágenes PAYG de RHEL y SLES de Azure Marketplace. La ventaja aún no está disponible para las imágenes BYOS de RHEL o SLES ni para imágenes personalizadas de Azure Marketplace.

Las instancias de Azure Dedicated Host y las ventajas híbridas de SQL no son válidas para la Ventaja híbrida de Azure si ya usa la ventaja con VM de Linux.

## <a name="get-started"></a>Primeros pasos

### <a name="red-hat-customers"></a>Para clientes de Red Hat

La Ventaja híbrida de Azure para RHEL está disponible para los clientes de Red Hat que cumplen con ambos criterios siguientes:

- Deben tener suscripciones de RHEL activas o no utilizadas que se pueden usar en Azure.
- Deben haber habilitado una o varias de esas suscripciones para usarlas en Azure con el programa [Red Hat Cloud Access](https://www.redhat.com/en/technologies/cloud-computing/cloud-access).

> [!IMPORTANT]
> Asegúrese de que se habilitó la suscripción correcta en el programa [Cloud Access](https://www.redhat.com/en/technologies/cloud-computing/cloud-access).

Para empezar a usar la ventaja para Red Hat:

1. Habilite una o varias de las suscripciones de RHEL válidas para su uso en Azure mediante la [interfaz de cliente de Red Hat Cloud Access](https://access.redhat.com/management/cloud).

   A continuación, las suscripciones de Azure que proporciona durante el proceso de habilitación de Red Hat Cloud Access podrán usar la característica Ventaja híbrida de Azure.
1. Aplique la Ventaja híbrida de Azure a cualquiera de las máquinas virtuales RHEL de pago por uso existentes, así como a todas las máquinas virtuales RHEL nuevas que implemente desde imágenes de pago por uso de Azure Marketplace. Puede usar Azure Portal o la CLI de Azure para habilitar la ventaja.
1. Siga los [pasos siguientes](https://access.redhat.com/articles/5419341) recomendados para configurar los orígenes de actualización para las máquinas virtuales RHEL y para las directrices de cumplimiento de las suscripciones de RHEL.


### <a name="suse-customers"></a>Para clientes de SUSE

Para empezar a usar la ventaja para SUSE:

1. Regístrese en el programa de nube pública de SUSE.
1. Aplique la ventaja a las máquinas virtuales existentes o recién creadas mediante Azure Portal o la CLI de Azure.
1. Registre las VM que reciben la ventaja con un origen de actualizaciones independiente.

## <a name="enable-and-disable-the-benefit-in-the-azure-portal"></a>Habilitación y deshabilitación de la ventaja en Azure Portal

Para habilitar la ventaja en las VM existentes, visite la opción **Configuración** de la izquierda y siga los pasos que se indican. Puede habilitar la ventaja en las nuevas VM durante la experiencia de creación de la VM.

### <a name="azure-portal-example-to-enable-the-benefit-during-creation-of-vm"></a>Ejemplo en Azure Portal de habilitación de la ventaja durante la creación de una máquina virtual:
1. Visite [Microsoft Azure Portal](https://portal.azure.com/)
1. Vaya a la página "Crear una máquina virtual" en el portal.
 ![Ventaja híbrida de Azure mientras se crea la máquina virtual](./media/azure-hybrid-benefit/create-vm-ahb.png)
1. Haga clic en la casilla para habilitar la conversión de AHB y usar licencias de acceso a la nube.
 ![Casilla Ventaja híbrida de Azure mientras se crea la máquina virtual](./media/azure-hybrid-benefit/create-vm-ahb-checkbox.png)
1. Siga el siguiente conjunto de instrucciones para crear una máquina virtual.
1. Compruebe la hoja **Configuración** y verá la opción habilitada. 
![Hoja de configuración de AHB después de la operación de creación](./media/azure-hybrid-benefit/create-configuration-blade.png)

### <a name="azure-portal-example-to-enable-the-benefit-for-an-existing-vm"></a>Ejemplo en Azure Portal de habilitación de la ventaja para una máquina virtual existente:
1. Visite [Microsoft Azure Portal](https://portal.azure.com/)
1. Abra la página de la máquina virtual en la que quiera aplicar la conversión.
1. Vaya a la opción **Configuración** de la izquierda. Verá la sección Licencias. Para habilitar la conversión de AHB, active el botón de radio "Sí" y la casilla Confirmación.
![Hoja de configuración de AHB después de la operación de creación](./media/azure-hybrid-benefit/create-configuration-blade.png)


## <a name="enable-and-disable-the-benefit-in-the-azure-cli"></a>Habilitación y deshabilitación de la ventaja en la CLI de Azure

Puede usar el comando `az vm update` para actualizar las VM existentes. En el caso de las VM de RHEL, ejecute el comando con un parámetro `--license-type` de `RHEL_BYOS`. En el caso de las VM de SLES, ejecute el comando con un parámetro `--license-type` de `SLES_BYOS`.

### <a name="cli-example-to-enable-the-benefit"></a>Ejemplo de la CLI para habilitar la ventaja
```azurecli
# This will enable the benefit on a RHEL VM
az vm update -g myResourceGroup -n myVmName --license-type RHEL_BYOS

# This will enable the benefit on a SLES VM
az vm update -g myResourceGroup -n myVmName --license-type SLES_BYOS
```
### <a name="cli-example-to-disable-the-benefit"></a>Ejemplo de la CLI para deshabilitar la ventaja
Para deshabilitar la ventaja, use un valor de `--license-type` de `None`:

```azurecli
# This will disable the benefit on a VM
az vm update -g myResourceGroup -n myVmName --license-type None
```

### <a name="cli-example-to-enable-the-benefit-on-a-large-number-of-vms"></a>Ejemplo de la CLI para habilitar la ventaja en un gran número de VM
Para habilitar la ventaja en un gran número de VM, puede usar el parámetro `--ids` en la CLI de Azure.

```azurecli
# This will enable the benefit on a RHEL VM. In this example, ids.txt is an
# existing text file that contains a delimited list of resource IDs corresponding
# to the VMs using the benefit
az vm update -g myResourceGroup -n myVmName --license-type RHEL_BYOS --ids $(cat ids.txt)
```

En los ejemplos siguientes se muestran dos métodos para obtener una lista de identificadores de recursos: uno en el nivel de grupo de recursos, uno en el nivel de suscripción.

```azurecli
# To get a list of all the resource IDs in a resource group:
$(az vm list -g MyResourceGroup --query "[].id" -o tsv)

# To get a list of all the resource IDs of VMs in a subscription:
az vm list -o json | jq '.[] | {VMName: .name, ResourceID: .id}'
```

## <a name="apply-the-azure-hybrid-benefit-at-vm-create-time"></a>Aplicación de la Ventaja híbrida de Azure en el momento de la creación de la VM
Además de aplicar la Ventaja híbrida de Azure a las máquinas virtuales de pago por uso existentes, puede invocarla en el momento de la creación de la VM. Las ventajas de hacerlo son tres:
- Puede aprovisionar máquinas virtuales de PAYG y BYOS con la misma imagen y proceso.
- Permite cambios futuros en el modo de licencia, algo que no está disponible con una imagen solo de BYOS o si trae su propia máquina virtual.
- De manera predeterminada, la máquina virtual se conectará a Red Hat Update Infrastructure (RHUI) para asegurarse de que sigue estando actualizada y segura. Puede cambiar el mecanismo actualizado después de la implementación en cualquier momento.

## <a name="check-the-azure-hybrid-benefit-status-of-a-vm"></a>Comprobación del estado de la Ventaja híbrida de Azure de una VM
Puede ver el estado de la Ventaja híbrida de Azure de una máquina virtual mediante la CLI de Azure o mediante Azure Instance Metadata Service.

### <a name="azure-cli"></a>Azure CLI

Puede usar el comando `az vm get-instance-view` para este fin. Busque un campo `licenseType` en la respuesta. Si el campo `licenseType` existe y el valor es `RHEL_BYOS` o `SLES_BYOS`, la VM tiene habilitada la ventaja.

```azurecli
az vm get-instance-view -g MyResourceGroup -n MyVm
```

### <a name="azure-instance-metadata-service"></a>Servicio de metadatos de instancia de Azure

Desde dentro de la máquina virtual misma, puede consultar los metadatos atestiguados en Azure Instance Metadata Service para determinar el valor de `licenseType` de la máquina virtual. Un valor `licenseType` de `RHEL_BYOS` o `SLES_BYOS` indicará que la máquina virtual tiene habilitada la ventaja. [Obtenga más información sobre los metadatos atestiguados](./instance-metadata-service.md#attested-data).

## <a name="compliance"></a>Cumplimiento

### <a name="red-hat"></a>Red Hat

Los clientes que usan la Ventaja híbrida de Azure para RHEL aceptan los [términos legales](http://www.redhat.com/licenses/cloud_CSSA/Red_Hat_Cloud_Software_Subscription_Agreement_for_Microsoft_Azure.pdf) estándar y la [declaración de privacidad](http://www.redhat.com/licenses/cloud_CSSA/Red_Hat_Privacy_Statement_for_Microsoft_Azure.pdf) asociados a las ofertas de RHEL de Azure Marketplace.

Los clientes que usan la Ventaja híbrida de Azure para RHEL tienen tres opciones para proporcionar actualizaciones de software y revisiones a esas máquinas virtuales:

- [Red Hat Update Infrastructure](../workloads/redhat/redhat-rhui.md) (opción predeterminada)
- Red Hat Satellite Server
- Red Hat Subscription Manager

Los clientes que eligen la opción RHUI pueden seguir usándola como el origen de actualización principal para sus máquinas virtuales RHEL de la Ventaja híbrida de Azure sin necesidad de adjuntar suscripciones de RHEL a esas máquinas virtuales. Los clientes que elijan la opción RHUI son responsables de garantizar el cumplimiento de las suscripciones de RHEL.

Los clientes que elijan Red Hat Satellite Server o Red Hat Subscription Manager deben quitar la configuración de RHUI y, a continuación, adjuntar una suscripción de RHEL habilitada para Cloud Access a sus máquinas virtuales RHEL de la Ventaja híbrida de Azure.  

Para más información sobre el cumplimiento de las suscripciones de Red Hat, las actualizaciones de software y los orígenes de las máquinas virtuales RHEL de la Ventaja híbrida de Azure, consulte el [artículo de Red Hat sobre las suscripciones de RHEL con la Ventaja híbrida de Azure](https://access.redhat.com/articles/5419341).

### <a name="suse"></a>SUSE

A fin de usar Ventaja híbrida de Azure para las máquinas virtuales de SLES y para obtener información sobre cómo cambiar de PAYG de SLES a BYOS, o bien pasar de BYOS de SLES a PAYG, vea [SUSE Linux Enterprise y Ventaja híbrida de Azure](https://www.suse.com/c/suse-linux-enterprise-and-azure-hybrid-benefit/). 

## <a name="azure-hybrid-benefit-on-reserved-instances"></a>Ventaja híbrida de Azure en instancias reservadas 

Las reservas de Azure (Azure Reserved Virtual Machine Instances) le permitirán ahorrar dinero, ya que se compromete a planes de uno o tres años para varios productos. Obtenga más información sobre las [Instancias reservadas aquí](https://docs.microsoft.com/azure/cost-management-billing/reservations/save-compute-costs-reservations). La Ventaja híbrida de Azure está disponible para [Instancias reservadas de máquina virtual (RI)](https://review.docs.microsoft.com/azure/cost-management-billing/reservations/save-compute-costs-reservations#charges-covered-by-reservation). 

Esto significa que, si ha adquirido costos de proceso con una tarifa de descuento mediante RI, puede aplicar el beneficio de AHB en los costos de licencia de RHEL y SUSE. Los pasos para aplicar la ventaja AHB para una instancia de RI son exactamente iguales que los de una máquina virtual normal.
![AHB para RIs](./media/azure-hybrid-benefit/reserved-instances.png)

>[!NOTE]
>Si ya ha adquirido reservas para el software RHEL o SUSE PAYG en Azure Marketplace, espere a que finalice el tiempo de reserva antes de usar la Ventaja híbrida de Azure.


## <a name="frequently-asked-questions"></a>Preguntas más frecuentes
*P: ¿Puedo usar un tipo de licencia de `RHEL_BYOS` con una imagen de SLES o viceversa?*

A. No, no puede. Si intenta especificar un tipo de licencia que no coincida correctamente con la distribución que se ejecuta en la VM, no se actualizarán los metadatos de facturación. Pero si especifica por accidente el tipo de licencia incorrecto, pero actualiza la VM de nuevo al tipo de licencia correcto, podrá seguir habilitando la ventaja.

*P: Me registré en Red Hat Cloud Access, pero todavía no puedo habilitar la ventaja en mis VM de RHEL, ¿qué debo hacer?*

A. El registro de la suscripción a Red Hat Cloud Access puede tardar un tiempo en propagarse de Red Hat a Azure. Si sigue viendo el error después de un día laborable, póngase en contacto con Soporte técnico de Microsoft.

*P: He implementado una máquina virtual con una "imagen maestra" de BYOS de RHEL. ¿Puedo convertir la facturación de estas imágenes de BYOS a PAYG?*

A. No, no puede. La Ventaja híbrida de Azure admite la conversión solo en imágenes de pago por uso.

*P: Cargué en Azure mi propia imagen de RHEL desde el entorno local (mediante Azure Migrate, Azure Site Recovery o de otro modo). ¿Puedo convertir la facturación de estas imágenes de BYOS a PAYG?*

A. No, no puede. La funcionalidad de la Ventaja híbrida de Azure solo está disponible actualmente para las imágenes de RHEL y SLES en Azure Marketplace. 

*P: Cargué en Azure mi propia imagen de RHEL desde el entorno local (mediante Azure Migrate, Azure Site Recovery o de otro modo). ¿Debo hacer algo para beneficiarme de la Ventaja híbrida de Azure?*

A. No, no es necesario. Las imágenes de RHEL que se cargan ya se consideran BYOS y solo se cobran los costos de la infraestructura de Azure. Usted es responsable de los costos de las suscripciones de RHEL, de la misma forma que con los entornos locales. 

*P: ¿Puedo usar la Ventaja híbrida de Azure en máquinas virtuales implementadas a partir de imágenes SAP de Azure Marketplace de RHEL y SLES?*

R: Sí, puede hacerlo. Puede usar el tipo de licencia `RHEL_BYOS` para las máquinas virtuales de RHEL y `SLES_BYOS` para las conversiones de las máquinas virtuales implementadas a partir de imágenes SAP de Azure Marketplace de RHEL y SLES.

*P: ¿Puedo usar la Ventaja híbrida de Azure en conjuntos de escalado de máquinas virtuales para RHEL y SLES?*

R: Sí, la Ventaja híbrida de Azure en conjuntos de escalado de máquinas virtuales para RHEL y SLES está disponible para todos los usuarios. Puede obtener [más información sobre esta ventaja y cómo usarla aquí](https://docs.microsoft.com/azure/virtual-machine-scale-sets/azure-hybrid-benefit-linux). 

*P: ¿Puedo usar la Ventaja híbrida de Azure en instancias reservadas para RHEL y SLES?*

R: Sí, la Ventaja híbrida de Azure en instancias reservadas para RHEL y SLES está disponible para todos los usuarios. Puede obtener [más información sobre esta ventaja y cómo usarla aquí](#azure-hybrid-benefit-on-reserved-instances).

*P: ¿Puedo usar la Ventaja híbrida de Azure en una máquina virtual implementada para SQL Server en imágenes de RHEL?*

A. No, no puede. No está previsto admitir estas máquinas virtuales.

*P: ¿Puedo usar Ventaja híbrida de Azure en la suscripción del Centro de datos virtual de RHEL?*

A. Lamentablemente, no. El Centro de datos virtual no se admite en Azure, incluida AHB.  
 

## <a name="common-problems"></a>Problemas comunes
En esta sección se enumeran los problemas comunes que puede encontrar y los pasos para mitigarlos.

| Error | Mitigación |
| ----- | ---------- |
| "The action could not be completed because our records show that you have not successfully enabled Red Hat Cloud Access on your Azure subscription…" (No se pudo completar la acción porque nuestros registros muestran que no ha habilitado correctamente Red Hat Cloud Access en la suscripción a Azure…) | Para usar la ventaja con VM de RHEL, primero debe [registrar las suscripciones de Azure en Red Hat Cloud Access](https://access.redhat.com/management/cloud).

## <a name="next-steps"></a>Pasos siguientes
* [Aprenda a crear y actualizar máquinas virtuales y a agregar tipos de licencia (RHEL_BYOS, SLES_BYOS) para Ventaja híbrida de Azure con la CLI de Azure](/cli/azure/vm)
* R: Sí, la Ventaja híbrida de Azure en conjuntos de escalado de máquinas virtuales para RHEL y SLES está disponible para todos los usuarios. Puede obtener [más información sobre esta ventaja y cómo usarla aquí](https://docs.microsoft.com/azure/virtual-machine-scale-sets/azure-hybrid-benefit-linux). 
