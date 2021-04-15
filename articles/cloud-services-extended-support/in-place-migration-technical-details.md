---
title: Detalles técnicos y requisitos para migrar A Azure Cloud Services (soporte extendido)
description: Proporciona detalles técnicos y requisitos para la migración desde Azure Cloud Services (clásico) a Azure Cloud Services (soporte extendido)
author: tanmaygore
ms.service: cloud-services-extended-support
ms.subservice: classic-to-arm-migration
ms.reviwer: mimckitt
ms.topic: how-to
ms.date: 02/06/2020
ms.author: tagore
ms.openlocfilehash: 4ff7d9aa2075b675a7ecd979c08d5621bbdd831a
ms.sourcegitcommit: b8995b7dafe6ee4b8c3c2b0c759b874dff74d96f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/03/2021
ms.locfileid: "106286790"
---
# <a name="technical-details-of-migrating-to-azure-cloud-services-extended-support"></a>Detalles técnicos de la migración a Azure Cloud Services (soporte extendido)   

En este artículo se describen los detalles técnicos relacionados con la herramienta de migración en relación con Cloud Services (clásico). 

> [!IMPORTANT]
> La migración de Cloud Services (clásico) a Cloud Services (soporte extendido) mediante la herramienta de migración se encuentra actualmente en versión preliminar pública. Esta versión preliminar se ofrece sin Acuerdo de Nivel de Servicio y no se recomienda para cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas. Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="details-about-feature--scenarios-supported-for-migration"></a>Detalles acerca de las características o escenarios admitidos para la migración 

### <a name="extensions-and-plugin-migration"></a>Migración de extensiones y complementos 
- Se migrarán todas las extensiones habilitadas y admitidas. 
- No se migrarán las extensiones deshabilitadas. 
- Los complementos son un concepto heredado y deben quitarse antes de la migración. Se admiten para la migración y, después de la migración, pero si es necesario habilitar la extensión, es necesario quitar primero el complemento antes de instalar la extensión. Los complementos y extensiones de Escritorio remoto se ven afectados por esto.   
 
### <a name="certificate-migration"></a>Migración de certificados
- En Cloud Services (soporte extendido), los certificados se almacenan en una instancia de Key Vault. Como parte de la migración, se crea una instancia de Key Vault para los clientes que tienen el nombre de la instancia de Cloud Services, y se transfieren todos los certificados de Azure Service Manager a Key Vault. 
- La referencia a esta instancia de Key Vault se especifica en la plantilla o se pasa a través de PowerShell o la CLI de Azure. 

### <a name="service-configuration-and-service-definition-files"></a>Archivos de configuración del servicio y de definición de servicio
- Los archivos .cscfg y .csdef deben actualizarse para Cloud Services (soporte extendido) con cambios menores. 
- Los nombres de recursos, como las SKU de redes virtuales y de VM, son diferentes. Consulte [Traducción de recursos y convención de nomenclatura después de la migración](#translation-of-resources-and-naming-convention-post-migration).
- Los clientes pueden recuperar sus nuevas implementaciones a través de [PowerShell](https://docs.microsoft.com/powershell/module/az.cloudservice/?view=azps-5.4.0#cloudservice&preserve-view=true) y la [API REST](https://docs.microsoft.com/rest/api/compute/cloudservices/get). 

### <a name="cloud-service-and-deployments"></a>Cloud Services e implementaciones
- Cada implementación de Cloud Services (soporte extendido) es un Cloud Service independiente. Las implementaciones ya no se agrupan en un servicio en la nube mediante espacios.
- Si tiene dos espacios en su instancia de Cloud Services (clásico), debe eliminar un espacio (ensayo) y usar la herramienta de migración para mover el otro espacio (producción) a Azure Resource Manager. 
- La dirección IP pública de la implementación de Cloud Services sigue siendo la misma después de la migración a Azure Resource Manager y se expone como recurso de IP de SKU básico (dinámico o estático). 
- El nombre DNS y el dominio (cloudapp.azure.net) para el servicio en la nube migrado siguen siendo los mismos. 

### <a name="virtual-network-migration"></a>Migración de redes virtuales
- Si una implementación de Cloud Services está en una red virtual, durante la migración todos los Cloud Services y los recursos de red virtual asociados se migran juntos. 
- Después de la migración, la red virtual se coloca en un grupo de recursos diferente que la instancia de Cloud Services. 
- En el caso de las redes virtuales con varias instancias de Cloud Services, cada instancia de Cloud Services se migra una después de otra. 

### <a name="migration-of-deployments-not-in-a-virtual-network"></a>Migración de implementaciones que no están en una red virtual
- En 2017, Azure comenzó a crear automáticamente nuevas implementaciones (sin la red virtual especificada por el cliente) en una red virtual "predeterminada" creada por la plataforma. Estas redes virtuales predeterminadas están ocultas a los clientes.   
- Como parte de la migración, esta red virtual predeterminada se expondrá a los clientes una vez que esté en Azure Resource Manager. Para administrar o actualizar la implementación en Azure Resource Manager, los clientes tienen que agregar esta información de red virtual en la sección NetworkConfiguration del archivo .cscfg.    
- La red virtual predeterminada, cuando se migra a Azure Resource Manager, se coloca en el mismo grupo de recursos que la implementación de Cloud Services.
- Las implementaciones de Cloud Services creadas antes de este momento no estarán en ninguna red virtual y no se podrán migrar con la herramienta. Considere la posibilidad de volver a implementar estas implementaciones de Cloud Services directamente en Azure Resource Manager. 
- Para comprobar si una implementación es válida para la migración, ejecute la API de validación en la implementación. El resultado de la API de validación contendrá un mensaje de error que menciona explícitamente si esta implementación es válida para la migración.     

### <a name="load-balancer"></a>Load Balancer   
- En el caso de una instancia de Cloud Services que usa un punto de conexión público, un equilibrador de carga creado por la plataforma asociado a la instancia de Cloud Services se expone dentro de la suscripción del cliente en Azure Resource Manager. El equilibrador de carga es un recurso de solo lectura, y las actualizaciones solo se restringen a través de los archivos de configuración del servicio (.cscfg) y de definición de servicio (.csdef). 

### <a name="key-vault"></a>Key Vault
- Como parte de la migración, Azure crea automáticamente una nueva instancia de Key Vault y migra todos los certificados a ella. La herramienta no permite usar una instancia de Key Vault existente. 
- Cloud Services (soporte extendido) requiere una instancia de Key Vault ubicada en la misma región y suscripción. Esta instancia de Key Vault se crea automáticamente como parte de la migración. 


## <a name="translation-of-resources-and-naming-convention-post-migration"></a>Traducción de recursos y convención de nomenclatura después de la migración
Como parte de la migración, se cambian los nombres de los recursos, y pocas características de Cloud Services se exponen como recursos de Azure Resource Manager. En la tabla se resumen los cambios específicos a una migración de Cloud Services.

| Cloud Services (clásico) <br><br> Nombre del recurso | Cloud Services (clásico) <br><br> Sintaxis| Cloud Services (soporte extendido) <br><br> Nombre del recurso| Cloud Services (soporte extendido) <br><br> Sintaxis | 
|---|---|---|---|
| Servicio en la nube | `cloudservicename` | Sin asociar| Sin asociar |
| Implementación (creada en el portal) <br><br> Implementación (creada fuera del portal)  | `deploymentname` | Cloud Services (soporte extendido) | `deploymentname` |  
| Virtual Network | `vnetname` <br><br> `Group resourcegroupname vnetname` <br><br> Sin asociar |  Virtual Network (creada fuera del portal) <br><br> Virtual Network (creada en el portal) <br><br> Virtual Network (predeterminado) | `vnetname` <br><br> `group-resourcegroupname-vnetname` <br><br> `DefaultRdfevirtualnetwork_vnetid`|
| Sin asociar | Sin asociar | Key Vault | `cloudservicename` | 
| Sin asociar | Sin asociar | Grupo de recursos para implementaciones de Cloud Services | `cloudservicename-migrated` | 
| Sin asociar | Sin asociar | Grupo de recursos para Virtual Network | `vnetname-migrated` <br><br> `group-resourcegroupname-vnetname-migrated`|
| Sin asociar | Sin asociar | IP pública (dinámica) | `cloudservicenameContractContract` | 
| Nombre de IP reservada | `reservedipname` | IP reservada (creada fuera del portal) <br><br> IP reservada (creada en el portal) | `reservedipname` <br><br> `group-resourcegroupname-reservedipname` | 
| Sin asociar| Sin asociar | Load Balancer | `deploymentname-lb`|



## <a name="migration-issues-and-how-to-handle-them"></a>Problemas de migración y cómo abordarlo. 

### <a name="migration-stuck-in-an-operation-for-a-long-time"></a>La migración se ha bloqueado en una operación durante mucho tiempo. 
- La confirmación, preparación y anulación pueden tardar mucho tiempo en función del número de implementaciones. Se agotará el tiempo de espera de las operaciones después de 24 horas.   
- Las operaciones de confirmación, preparación y anulación son idempotentes. La mayoría de los problemas se pueden corregir volviendo a intentar las operaciones. Podría haber errores transitorios, que pueden desaparecer en pocos minutos, se recomienda volver a intentar la operación después de un intervalo. Si la migración se realiza mediante Azure Portal y la operación está bloqueada en un estado "en curso", use PowerShell para volver a intentarla. 
- Póngase en contacto con soporte técnico para que le ayude a migrar o revertir la implementación desde el back-end. 

### <a name="migration-failed-in-an-operation"></a>Error de la migración en una operación. 
- Si se produce un error en la validación, se debe a que la implementación o la red virtual contienen un escenario, una característica o un recurso no admitidos. Use la lista de escenarios no admitidos para encontrar la solución alternativa en los documentos.  
- La operación de preparación primero realiza una validación que incluye algunas validaciones costosas (que no se cubren en la validación). El error de preparación puede deberse a un escenario no admitido. Busque el escenario y la solución alternativa en los documentos públicos. Se debe llamar a la anulación para volver al estado original y desbloquear la implementación para las operaciones de actualización y eliminación.
- Si se produce un error en la anulación, vuelva a intentar la operación. Si hay errores en los reintentos, póngase en contacto con el soporte técnico.
- Si se produce un error en la confirmación, vuelva a intentar la operación. Si hay errores en los reintentos, póngase en contacto con el soporte técnico. Incluso en caso de error en la confirmación, no debería haber ningún problema del plano de datos en la implementación. La implementación debería poder controlar el tráfico del cliente sin ningún problema. 

### <a name="portal-refreshed-after-prepare-experience-restarted-and-commit-or-abort-not-visible-anymore"></a>El portal se actualizó después de la preparación. La experiencia se reinició, y la confirmación o anulación ya no son visibles. 
- El portal almacena la información de migración localmente y, por tanto, después de la actualización, se iniciará desde la fase de validación incluso si la instancia de Cloud Services se encuentra en la fase de preparación.  
- Puede usar el portal para seguir los pasos de validación y preparación de nuevo para mostrar el botón Anular o Confirmar. No producirá ningún error.
- Los clientes pueden usar PowerShell o la API REST para anular o confirmar. 

### <a name="how-much-time-can-the-operations-takebr"></a>¿Cuánto tiempo pueden tardar las operaciones?<br>
La validación está diseñada para ser rápida. La preparación es la operación de más larga duración y tarda algún tiempo en función del número total de instancias de rol que se van a migrar. La anulación y confirmación también pueden llevar tiempo, pero tardarán menos tiempo en comparación con la preparación. Se agotará el tiempo de espera de todas las operaciones después de 24 horas. 
