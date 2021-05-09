---
title: Preguntas frecuentes sobre Azure Cloud Services (soporte extendido)
description: Preguntas frecuentes sobre Azure Cloud Services (soporte extendido)
ms.topic: conceptual
ms.service: cloud-services-extended-support
author: gachandw
ms.author: gachandw
ms.reviewer: mimckitt
ms.date: 10/13/2020
ms.custom: ''
ms.openlocfilehash: 8707578eef3467cc24cfeb3cd1c5156d7f5b3b1d
ms.sourcegitcommit: fc9fd6e72297de6e87c9cf0d58edd632a8fb2552
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/30/2021
ms.locfileid: "108288767"
---
# <a name="frequently-asked-questions-for-azure-cloud-services-extended-support"></a>Preguntas más frecuentes sobre Azure Cloud Services (soporte extendido)
En este artículo se tratan las preguntas frecuentes relacionadas con Azure Cloud Services (soporte extendido).

## <a name="general"></a>General

### <a name="what-is-the-resource-name-for-cloud-services-classic--cloud-services-extended-support"></a>¿Cuál es el nombre del recurso de Cloud Services (clásico) y Cloud Services (soporte extendido)?
- Cloud Services (clásico): `microsoft.classiccompute/domainnames`
- Cloud Services (soporte extendido): `microsoft.compute/cloudservices`

### <a name="what-locations-are-available-to-deploy-cloud-services-extended-support"></a>¿Qué ubicaciones están disponibles para implementar Cloud Services (soporte extendido)?
Cloud Services (soporte extendido) está disponible en todas las regiones de la nube pública.

### <a name="how-does-my-quota-change"></a>¿Cómo cambia mi cuota? 
Los clientes tendrán que solicitar la cuota usando los mismos procesos que para cualquier otro producto de Azure Resource Manager. La cuota de Azure Resource Manager es regional y se necesitará una solicitud de cuota distinta para cada región.

### <a name="why-dont-i-see-a-production--staging-slot-anymore"></a>¿Por qué ya no aparece un espacio de ensayo y producción?
Cloud Services (soporte extendido) no admite el concepto lógico de servicio hospedado, que incluía dos espacios (producción y ensayo). Cada implementación es una implementación de Cloud Services (soporte extendido) independiente. Para probar y ensayar una nueva versión de un servicio en la nube, implemente un servicio en la nube (soporte extendido) y etiquételo como "con intercambio de VIP" con otro servicio en la nube (soporte extendido)

### <a name="why-cant-i-create-an-empty-cloud-service-anymore"></a>¿Por qué ya no se pueden crear instancias vacías de Cloud Services?
Ya no existe el concepto de nombres de servicio hospedado, así que ya no se puede crear una instancia vacía de Cloud Services (soporte extendido).

### <a name="does-cloud-services-extended-support-support-resource-health-check-rhc"></a>¿Admite Cloud Services (soporte extendido) Resource Health Check (RHC)?
No, Cloud Services (soporte extendido) no admite Resource Health Check (RHC).

### <a name="how-are-role-instance-metrics-changing"></a>¿Cómo cambian las métricas de las instancias de rol?
No hay cambios en las métricas de las instancias de rol. 

### <a name="how-are-web--worker-roles-changing"></a>¿Cómo cambian los roles web y de trabajo?
No hay ningún cambio en el diseño, la arquitectura o los componentes de los roles web y de trabajo. 

### <a name="how-are-role-instances-changing"></a>¿Cómo cambian las instancias de rol?
No hay ningún cambio en el diseño, la arquitectura o los componentes de las instancias de rol. 

### <a name="how-will-guest-os-updates-change"></a>¿Cómo cambiarán las actualizaciones del sistema operativo invitado?
 No hay ningún cambio en el método de implementación. Cloud Services (clásico) y Cloud Services (soporte extendido) recibirán las mismas actualizaciones.
 
### <a name="does-cloud-services-extended-support-support-stopped-allocated-and-stopped-deallocated-states"></a>¿Cloud Services (soporte extendido) admite los estados detenido-asignado y detenido-desasignado?

La implementación de Cloud Services (soporte extendido) solo admite el estado detenido-asignado, que aparece como "detenido" en Azure Portal. El estado detenido-desasignado no se admite. 

### <a name="do-cloud-services-extended-support-deployments-support-scaling-across-clusters-availability-zones-and-regions"></a>¿Las implementaciones de Cloud Services (soporte extendido) admiten el escalado entre clústeres, zonas de disponibilidad y regiones?
Las implementaciones de Cloud Services (soporte extendido) no se pueden escalar entre varios clústeres, zonas de disponibilidad y regiones. 

### <a name="how-can-i-get-the-deployment-id-for-my-cloud-service-extended-support"></a>¿Cómo se puede obtener el identificador de implementación para Cloud Services (soporte extendido)?
Se puede acceder al identificador de implementación, conocido como identificador privado, mediante la API [CloudServiceInstanceView](/rest/api/compute/cloudservices/getinstanceview#cloudserviceinstanceview). También está disponible en Azure Portal en la hoja Rol e instancias de Cloud Services (soporte extendido).

### <a name="are-there-any-pricing-differences-between-cloud-services-classic-and-cloud-services-extended-support"></a>¿Existen diferencias de precio entre Cloud Services (clásico) y Cloud Services (soporte extendido)?
Cloud Services (soporte extendido) utiliza direcciones IP públicas de Azure Key Vault y básicas (ARM). Los clientes que necesiten certificados deben usar Azure Key Vault para la administración de certificados ([más información](https://azure.microsoft.com/pricing/details/key-vault/) sobre los precios de Azure Key Vault).  Cada dirección IP pública de Cloud Services (soporte extendido) se cobra por separado ([más información](https://azure.microsoft.com/pricing/details/ip-addresses/) sobre los precios de las direcciones IP públicas). 
## <a name="resources"></a>Recursos 

### <a name="what-resources-linked-to-a-cloud-services-extended-support-deployment-need-to-live-in-the-same-resource-group"></a>¿Qué recursos vinculados a una implementación de Cloud Services (soporte extendido) deben residir en el mismo grupo de recursos?
Los equilibradores de carga, los grupos de seguridad de red y las tablas de rutas deben residir en la misma región y en el mismo grupo de recursos. 

### <a name="what-resources-linked-to-a-cloud-services-extended-support-deployment-need-to-live-in-the-same-region"></a>¿Qué recursos vinculados a una implementación de Cloud Services (soporte extendido) deben residir en la misma región?
Key Vault, la red virtual, las direcciones IP públicas, los grupos de seguridad de red y las tablas de rutas deben residir en la misma región.

### <a name="what-resources-linked-to-a-cloud-services-extended-support-deployment-need-to-live-in-the-same-virtual-network"></a>¿Qué recursos vinculados a una implementación de Cloud Services (soporte extendido) deben residir en la misma red virtual?
Las direcciones IP públicas, los equilibradores de carga, los grupos de seguridad de red y las tablas de rutas deben residir en la misma red virtual. 

## <a name="deployment-files"></a>Archivos de implementación 

### <a name="how-can-i-use-a-template-to-deploy-or-manage-my-deployment"></a>¿Cómo puedo usar una plantilla para implementar o administrar mi implementación?
Los archivos de plantilla y de parámetro se pueden pasar como parámetro mediante REST, PowerShell y la CLI. También se pueden cargar mediante Azure Portal.  

### <a name="do-i-need-to-maintain-four-files-now-template-parameter-csdef-cscfg"></a>¿Es necesario mantener cuatro archivos ahora? (plantilla, parámetro, csdef, cscfg)
Los archivos de plantilla y de parámetro solo se usan para la automatización de la implementación. Al igual que con Cloud Services (clásico), puede crear manualmente recursos dependientes en primer lugar y después una implementación de Cloud Services (soporte extendido) con PowerShell, comandos de la CLI o mediante el portal con los archivos csdef y cscfg existentes.

### <a name="how-does-my-application-code-change-on-cloud-services-extended-support"></a>¿Cómo cambia el código de la aplicación en Cloud Services (soporte extendido)?
No se requieren cambios en el código de aplicación empaquetado en cspkg. Las aplicaciones existentes seguirán funcionando como de costumbre. 

### <a name="does-cloud-services-extended-support-allow-ctp-package-format"></a>¿Permite Cloud Services (soporte extendido) el formato de paquete CTP?
El formato de paquete CTP no se admite en Cloud Services (soporte extendido). Sin embargo, permite un límite de tamaño de paquete mejorado, de 800 MB.

## <a name="migration"></a>Migración

### <a name="will-cloud-services-extended-support-mitigate-the-failures-due-to-allocation-failures"></a>¿Cloud Services (soporte extendido) puede mitigar los errores de asignación?
No, las implementaciones de Cloud Services (soporte extendido) están vinculadas a un clúster, lo mismo que sucede con Cloud Services (clásico). Por lo tanto, los errores de asignación seguirán existiendo si el clúster está lleno. 

### <a name="when-do-i-need-to-migrate"></a>¿Cuándo es necesario migrar? 
La estimación del tiempo necesario y la complejidad de la migración depende de una serie de variables. La planificación es el paso más eficaz para comprender el ámbito del trabajo, los inhibidores y la complejidad de la migración.

## <a name="networking"></a>Redes 

### <a name="why-cant-i-create-a-deployment-without-virtual-network"></a>¿Por qué no se puede crear una implementación sin una red virtual?
Las redes virtuales son un recurso necesario para cualquier implementación en Azure Resource Manager. La implementación de Cloud Services (soporte extendido) debe residir en una red virtual. 

### <a name="why-am-i-now-seeing-so-many-networking-resources"></a>¿Por qué ahora me aparecen tantos recursos de red? 
En Azure Resource Manager, los componentes de la implementación de Cloud Services (soporte extendido) se exponen como un recurso para mejorar la visibilidad y el control. En Cloud Services (clásico) se usaban recursos del mismo tipo, pero estaban ocultos. Un ejemplo de este tipo de recurso es Load Balancer público, que ahora es un recurso explícito de "solo lectura" creado automáticamente por la plataforma.

### <a name="what-restrictions-apply-for-a-subnet-with-respective-to-cloud-services-extended-support"></a>¿Qué restricciones se aplican a una subred con respecto a Cloud Services (soporte extendido)?
Una subred que contenga implementaciones de Cloud Services (soporte extendido) no se puede compartir con implementaciones de otros productos de proceso, como Virtual Machines, Virtual Machine Scale Sets, Service Fabric, etc.

### <a name="what-ip-allocation-methods-are-supported-on-cloud-services-extended-support"></a>¿Qué métodos de asignación de IP se admiten en Cloud Services (soporte extendido)?
Cloud Services (soporte extendido) es compatible con los métodos de asignación de IP estática y dinámica. En el archivo cscfg se hace referencia a las direcciones IP estáticas como direcciones IP reservadas.

### <a name="why-am-i-getting-charged-for-ip-addresses"></a>¿Por qué me cobran las direcciones IP?
A los clientes se les factura el uso de la dirección IP en Cloud Services (soporte extendido) del mismo modo que se factura a los usuarios las direcciones IP asociadas a las máquinas virtuales. 

### <a name="can-i-use-a-dns-name-with-cloud-services-extended-support"></a>¿Puedo usar un nombre DNS con Cloud Services (soporte extendido)? 
Sí. También se puede asignar un nombre DNS a Cloud Services (soporte extendido). Con Azure Resource Manager, la etiqueta DNS es una propiedad opcional de la dirección IP pública que se asigna al servicio en la nube. El formato del nombre DNS en las implementaciones basadas en Azure Resource Manager es `<userlabel>.<region>.cloudapp.azure.com`.

### <a name="can-i-update-or-change-the-virtual-network-reference-for-an-existing-cloud-service-extended-support"></a>¿Puedo actualizar o cambiar la referencia de red virtual de una instancia existente de Cloud Services (soporte extendido)? 
No. La referencia de red virtual es obligatoria durante la creación de un servicio en la nube. Para un servicio en la nube existente, no se puede cambiar la referencia de red virtual. El espacio de direcciones de red virtual mismo se puede modificar mediante las API de red virtual. 

## <a name="certificates--key-vault"></a>Certificados y Key Vault

### <a name="why-do-i-need-to-manage-my-certificates-on-cloud-services-extended-support"></a>¿Por qué es necesario administrar mis certificados en Cloud Services (soporte extendido)?
Cloud Services (soporte extendido) ha adoptado el mismo proceso que otras ofertas de proceso en las que los certificados residen en los almacenes de claves administrados por el cliente. Esto permite a los clientes tener un control completo sobre sus secretos y certificados. 

### <a name="can-i-use-one-key-vault-for-all-my-deployments-in-all-regions"></a>¿Puedo usar una instancia de Key Vault en todas las implementaciones en todas las regiones?
No. Key Vault es un recurso regional, así que los clientes necesitan una instancia de Key Vault en cada región. Sin embargo, se puede usar una instancia de Key Vault en todas las implementaciones realizadas dentro de una región determinada.

## <a name="next-steps"></a>Pasos siguientes
Para empezar a usar Cloud Services (soporte extendido), consulte [Implementación de una instancia de Cloud Services (soporte extendido) mediante PowerShell](deploy-powershell.md).