---
title: Migración de Azure Cloud Services (clásico) a Azure Cloud Services (soporte extendido)
description: Información general de la migración de Cloud Services (clásico) a Cloud Services (soporte extendido)
ms.topic: how-to
ms.service: cloud-services-extended-support
ms.subservice: classic-to-arm-migration
author: tanmaygore
ms.author: tagore
ms.reviewer: mimckitt
ms.date: 2/08/2021
ms.custom: ''
ms.openlocfilehash: 96315899f80d0bd02ac3d2108b7cd76876025218
ms.sourcegitcommit: b8995b7dafe6ee4b8c3c2b0c759b874dff74d96f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/03/2021
ms.locfileid: "106286791"
---
# <a name="migrate-azure-cloud-services-classic-to-azure-cloud-services-extended-support"></a>Migración de Azure Cloud Services (clásico) a Azure Cloud Services (soporte extendido)

En este artículo se proporciona información general sobre la herramienta de migración compatible con la plataforma y cómo usarla para migrar de [Azure Cloud Services (clásico)](../cloud-services/cloud-services-choose-me.md) a [Azure Cloud Services (soporte extendido)](overview.md).

La herramienta de migración emplea las mismas API y tiene la misma experiencia que la [migración de Virtual Machines (clásico)](https://docs.microsoft.com/azure/virtual-machines/migration-classic-resource-manager-overview). 

> [!IMPORTANT]
> La migración de Cloud Services (clásico) a Cloud Services (soporte extendido) mediante la herramienta de migración se encuentra actualmente en versión preliminar pública. Esta versión preliminar se ofrece sin Acuerdo de Nivel de Servicio y no se recomienda para cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas. Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Si necesita ayuda con la migración, consulte los siguientes recursos: 

- [Microsoft Q&A](https://docs.microsoft.com/answers/topics/azure-cloud-services-extended-support.html): Soporte técnico de Microsoft y de la comunidad para la migración
- [Soporte técnico para la migración de Azure](https://ms.portal.azure.com/#create/Microsoft.Support/Parameters/%7B%22pesId%22:%22e79dcabe-5f77-3326-2112-74487e1e5f78%22,%22supportTopicId%22:%22fca528d2-48bd-7c9f-5806-ce5d5b1d226f%22%7D): equipo de soporte técnico dedicado para brindar asistencia técnica durante la migración. Los clientes sin soporte técnico pueden usar la [funcionalidad de soporte técnico gratuita](https://aka.ms/cs-migration-errors) que se proporciona específicamente para esta migración.
- Si su empresa u organización se ha asociado con Microsoft o trabaja con un representante de Microsoft, como un arquitecto de soluciones en la nube o administradores técnicos de cuentas, comuníquese con ellos para obtener más recursos para la migración.
- Responda [esta encuesta](https://forms.office.com/Pages/ResponsePage.aspx?id=v4j5cvGGr0GRqy180BHbR--AgudUMwJKgRGMO84rHQtUQzZYNklWUk4xOTFXVFBPOFdGOE85RUIwVC4u) para proporcionar comentarios o enviar incidencias al equipo de producto de Cloud Services (soporte extendido). 

## <a name="migration-benefits"></a>Ventajas de la migración
La migración compatible con la plataforma proporciona las siguientes ventajas principales:
- La plataforma organiza completamente la migración, moviendo toda la implementación y los recursos asociados a Azure Resource Manager.
- Migración sin tiempo de inactividad.
- Migración fácil y rápida en comparación con otras rutas de migración, ya que minimiza las tareas manuales. 
- Conserva la dirección IP y la etiqueta DNS de Cloud Services como parte de la migración. 

Para conocer otras ventajas y por qué debe migrar, consulte [Cloud Services (soporte extendido)](overview.md) y [Azure Resource Manager](../azure-resource-manager/management/overview.md). 

## <a name="setup-access-for-migration"></a>Configuración del acceso para la migración

Para realizar esta migración, deben haberlo agregado como coadministrador de la suscripción y registrar los proveedores necesarios. 

1. Inicie sesión en Azure Portal.
3. En el menú de Central, seleccione Suscripción. Si no lo ve, haga clic en Todos los servicios.
3. Busque la entrada de la suscripción adecuada y después examine el campo MI ROL. Para un coadministrador, el valor debe ser Administrador de cuenta. Si no puede agregar un coadministrador, póngase en contacto con un administrador del servicio o coadministrador de la suscripción para que le agreguen.

4. Registre la suscripción para el espacio de nombres Microsoft.ClassicInfrastructureMigrate mediante el [portal](../azure-resource-manager/management/resource-providers-and-types.md#azure-portal), [PowerShell](../azure-resource-manager/management/resource-providers-and-types.md#azure-powershell) o la [CLI](../azure-resource-manager/management/resource-providers-and-types.md#azure-cli).

    ```powershell
    Register-AzResourceProvider -ProviderNamespace Microsoft.ClassicInfrastructureMigrate 
    ```
 
5. Registre la suscripción para la característica en versión preliminar de la migración de Cloud Services mediante el [portal](../azure-resource-manager/management/resource-providers-and-types.md#azure-portal), [PowerShell](../azure-resource-manager/management/resource-providers-and-types.md#azure-powershell) o la [CLI](../azure-resource-manager/management/resource-providers-and-types.md#azure-cli).

    ```powershell
    Register-AzProviderFeature -FeatureName CloudServices -ProviderNamespace Microsoft.Compute 
    ```

6. Compruebe el estado del registro. El registro puede tardar unos minutos en completarse. 

    ```powershell
    Get-AzProviderFeature -FeatureName CloudServices -ProviderNamespace Microsoft.Compute 
    ```

## <a name="how-is-migration-for-cloud-services-classic-different-from-virtual-machines-classic"></a>¿En qué se diferencia la migración de Cloud Services (clásico) de Virtual Machines (clásico)?
Azure Service Manager admite dos productos de proceso diferentes, [Azure Virtual Machines (clásico)](https://docs.microsoft.com/previous-versions/azure/virtual-machines/windows/classic/tutorial-classic) y [Azure Cloud Services (clásico)](../cloud-services/cloud-services-choose-me.md) o roles web o de trabajo. Los dos productos difieren en función del tipo de implementación que se encuentra dentro del servicio en la nube. Azure Cloud Services (clásico) usa el servicio en la nube que contiene implementaciones con roles web y de trabajo. Azure Virtual Machines (clásico) usa un servicio en la nube que contiene implementaciones con VM IaaS.

La lista de escenarios admitidos difiere entre Cloud Services (clásico) y Virtual Machines (clásico) debido a las diferencias en los tipos de implementación.

## <a name="migration-steps"></a>Pasos de migración
 
Los clientes pueden migrar sus implementaciones de Cloud Services (clásico) mediante las cuatro mismas operaciones que se usan para migrar Virtual Machines (clásico). 

1. **Validar migración**: Valida que escenarios no admitidos comunes no impidan la migración.
2. **Preparar migración**: Duplica los metadatos de recursos en Azure Resource Manager. Todos los recursos están bloqueados para las operaciones de creación, actualización y eliminación para asegurar que los metadatos de recursos estén sincronizados entre Azure Server Manager y Azure Resource Manager. Todas las operaciones de lectura funcionarán con las API de Cloud Services (clásico) y Cloud Services (soporte extendido).
3. **Anular migración**: Quita los metadatos de recursos de Azure Resource Manager. Desbloquea todos los recursos para las operaciones de creación, actualización y eliminación.
4. **Confirmar migración**: Quita los metadatos de recursos de Azure Service Manager. Desbloquea el recurso para las operaciones de creación, actualización y eliminación. Ya no se permite la anulación después de que se haya intentado la confirmación.

>[!NOTE]
> Preparar, anular y confirmar son idempotentes y, por tanto, si se produce un error, un reintento debería corregir el problema.

:::image type="content" source="media/in-place-migration-1.png" alt-text="La imagen muestra el diagrama de los pasos asociados con la migración.":::

Para obtener más información, consulte [Migración compatible con la plataforma de recursos de IaaS desde el modelo clásico al de Azure Resource Manager](../virtual-machines/migration-classic-resource-manager-overview.md).

## <a name="supported-resources-and-features-available-for-migration-associated-with-cloud-services-classic"></a>Recursos y características admitidos disponibles para la migración asociada a Cloud Services (clásico)
-   Cuentas de almacenamiento
-   Virtual Networks
-   Grupos de seguridad de red
-   Direcciones IP públicas reservadas
-   Listas de control de acceso de punto de conexión
-   Rutas definidas por el usuario
-   Equilibrador de carga interno
-   Migración de certificados a un almacén de claves
-   Complementos y extensión (basados en XML y JSON)
-   Tareas en el inicio y en el final
-   Implementaciones con redes aceleradas
-   Implementaciones con uno o varios roles
-   Versión Básica de Load Balancer
-   Entrada, entrada de instancia, puntos de conexión internos
-   Direcciones IP públicas dinámicas
-   Nombre DNS 
-   Reglas de tráfico de red
-   Red virtual de Hypernet

## <a name="supported-configurations--migration-scenarios"></a>Configuraciones y escenarios de migración admitidos
Estos son los principales escenarios que implican combinaciones de recursos, características y Cloud Services. Esta lista no es exhaustiva.

| Servicio | Configuración | Comentarios | 
|---|---|---|
| [Azure AD Domain Services](https://docs.microsoft.com/azure/active-directory-domain-services/migrate-from-classic-vnet) | Redes virtuales que contienen Azure Active Directory Domain Services | Se admite la red virtual que contiene la implementación de Cloud Services y Azure AD Domain Services. El cliente primero debe migrar por separado Azure AD servicios de dominio y, a continuación, migrar la red virtual a la izquierda solo con la implementación del servicio en la nube. |
| Servicio en la nube | Servicio en la nube con una implementación en un solo espacio. | Se pueden migrar servicios en la nube que contienen una implementación de espacio de producción o de ensayo. |
| Servicio en la nube | Implementación no en una red virtual visible públicamente (implementación de red virtual predeterminada) | Un servicio en la nube puede estar en una red virtual visible públicamente, en una red virtual oculta o no en una red virtual.  Se admiten para la migración los servicios en la nube en una red virtual oculta y las redes virtuales visibles públicamente. El cliente puede usar la API de validación para saber si una implementación se encuentra dentro de una red virtual predeterminada o no y, por tanto, decidir si se puede migrar. |
|Servicio en la nube | Extensiones XML (BGInfo, depurador de Visual Studio, Web Deploy y depuración remota). | Se admiten todas las extensiones XML para la migración 
| Virtual Network | Red virtual que contiene varios servicios en la nube. | La red virtual contiene varios servicios en la nube que se admiten para la migración. La red virtual y todos los servicios en la nube que contiene se migrarán juntos a Azure Resource Manager. |
| Virtual Network | Migración de redes virtuales creadas a través del portal (requiere el uso de "grupo nombre-dl-grupo-de-recursos nombre-de-VNet" en el archivo .cscfg)  | Como parte de la migración, se cambiará el nombre de la red virtual en cscfg para usar el identificador de Azure Resource Manager para la red virtual. (subscription/id-suscripción/resource-group/nombre-del-grupo-de-recursos/resource/nombre-de-vnet) <br><br>Para administrar la implementación después de la migración, actualice la copia local del archivo .cscfg para empezar a usar el identificador de Azure Resource Manager en lugar del nombre de red virtual. <br><br>Un archivo .cscfg que use el esquema de nomenclatura anterior no superará la validación. 
| Virtual Network | Migración de la implementación con roles en una subred diferente. | Se admite la migración de un servicio en la nube con roles diferentes en subredes diferentes. |
    

## <a name="resources-and-features-not-available-for-migration"></a>Recursos y características no disponibles para la migración
Estos son los principales escenarios que implican combinaciones de recursos, características y Cloud Services. Esta lista no es exhaustiva. 

| Recurso | Pasos siguientes/solución alternativa | 
|---|---|
| Reglas de escalabilidad automática | La migración se completa, pero las reglas se anulan. [Vuelva a crear las reglas](https://docs.microsoft.com/azure/cloud-services-extended-support/configure-scaling) después de la migración en Cloud Services (soporte extendido). | 
| Alertas | La migración se completa, pero las alertas se anulan. [Vuelva a crear las reglas](https://docs.microsoft.com/azure/cloud-services-extended-support/enable-alerts) después de la migración en Cloud Services (soporte extendido). | 
| VPN Gateway | Quite VPN Gateway antes de comenzar la migración y, después, vuelva a crearlo una vez que la migración se complete. | 
| Puertas de enlace de ExpressRoute (en la misma suscripción que solo la red virtual) | Quite la puerta de enlace de ExpressRoute antes de comenzar la migración y, después, vuelva a crearla una vez que la migración se complete. | 
| Quota  | La cuota no se migra. [Solicite una nueva cuota](https://docs.microsoft.com/azure/azure-resource-manager/templates/error-resource-quota#solution) en Azure Resource Manager antes de la migración para que la validación sea correcta. | 
| Grupos de afinidad | No compatible. Quite todos los grupos de afinidad antes de la migración.  | 
| Redes virtuales con el [emparejamiento de red virtual](https://docs.microsoft.com/azure/virtual-network/virtual-network-peering-overview)| Antes de migrar una red virtual que está emparejada a otra red virtual, elimine el emparejamiento, migre la red virtual a Resource Manager y vuelva a crear el emparejamiento. Esto puede provocar un tiempo de inactividad en función de la arquitectura. | 
| Redes virtuales que contienen entornos de App Service | No compatible | 
| Redes virtuales que contienen servicios de HDInsight | No compatible. 
| Redes virtuales que contienen implementaciones de Azure API Management | No compatible. <br><br> Para migrar la red virtual, cambie la red virtual de la implementación de API Management. Se trata de una operación sin tiempo de inactividad. | 
| Circuitos ExpressRoute clásicos | No compatible. <br><br>Estos circuitos se deben migrar a Azure Resource Manager antes de comenzar la migración de PaaS. Para obtener más información, consulte la [transición de los circuitos ExpressRoute del modelo de implementación clásica al modelo de implementación de Resource Manager](../expressroute/expressroute-howto-move-arm.md). |  
| Control de acceso basado en roles | Después de la migración, el URI del recurso cambia de `Microsoft.ClassicCompute` a `Microsoft.Compute`. Las directivas de RBAC deben actualizarse después de la migración. | 
| Application Gateway | No compatible. <br><br> Quite Application Gateway antes de comenzar la migración y, después, vuelva a crearlo una vez que la migración se complete en Azure Resource Manager. | 

## <a name="unsupported-configurations--migration-scenarios"></a>Configuraciones y escenarios de migración no admitidos

| Configuración/escenario  | Pasos siguientes/solución alternativa | 
|---|---|
| Migración de algunas implementaciones anteriores que no están en una red virtual | Algunas implementaciones de Cloud Services que no están en una red virtual no se admiten para la migración. <br><br> 1. Use la API de validación para comprobar si la implementación es válida para la migración. <br> 2. Si es válida, las implementaciones se moverán a Azure Resource Manager en una red virtual con el prefijo "DefaultRdfeVnet". | 
| Migración de implementaciones que contienen implementaciones de espacio de producción y de ensayo mediante direcciones IP dinámicas | La migración de un servicio en la nube de dos espacios requiere la eliminación del espacio de ensayo. Una vez eliminado el espacio de ensayo, migre el espacio de producción como instancia independiente de Cloud Services (soporte extendido) en Azure Resource Manager. A continuación, vuelva a implementar el entorno de ensayo como nueva instancia de Cloud Services (soporte extendido) y haga que sea intercambiable con el primero. | 
| Migración de implementaciones que contienen implementaciones de espacio de producción y de ensayo mediante direcciones IP reservadas | No compatible. | 
| Migración de la implementación de producción y de ensayo en una red virtual diferente|La migración de un servicio en la nube de dos espacios requiere la eliminación del espacio de ensayo. Una vez eliminado el espacio de ensayo, migre el espacio de producción como instancia independiente de Cloud Services (soporte extendido) en Azure Resource Manager. A continuación, se puede vincular una nueva implementación de Cloud Services (soporte extendido) a la implementación migrada con la propiedad intercambiable habilitada. Los archivos de implementaciones de la antigua implementación del espacio de ensayo se pueden volver a usar para crear esta nueva implementación intercambiable. | 
| Migración de Cloud Services vacío (Cloud Services sin implementación) | No compatible. | 
| Migración de una implementación que contiene el complemento de Escritorio remoto y las extensiones de Escritorio remoto | Opción 1: quite el complemento de Escritorio remoto antes de la migración. Esto requiere cambios en los archivos de implementación. A continuación, la migración se completará. <br><br> Opción 2: quite la extensión de Escritorio remoto y migre la implementación. Después de la migración, quite el complemento e instale la extensión. Esto requiere cambios en los archivos de implementación. <br><br> Quite el complemento y la extensión antes de la migración. [No se recomienda el uso de complementos](https://docs.microsoft.com/azure/cloud-services-extended-support/deploy-prerequisite#required-service-definition-file-csdef-updates) en Cloud Services (soporte extendido).| 
| Redes virtuales con implementaciones PaaS e IaaS |No compatible <br><br> Mueva las implementaciones PaaS o IaaS a una red virtual diferente. Esto provocará un tiempo de inactividad. | 
Implementaciones de Cloud Serivces que usan tamaños de rol heredados (como Small o ExtraLarge). | La migración se completará, pero los tamaños de rol se actualizarán para usar los tamaños de rol modernos. No hay ningún cambio en las propiedades de costo o SKU, y la máquina virtual no se reiniciará para este cambio. Actualice todos los artefactos de implementación para que hagan referencia a estos nuevos tamaños de rol modernos. Para obtener más información, consulte [Tamaños de VM disponibles](available-sizes.md).|
| Migración de Cloud Services a una red virtual diferente | No compatible <br><br> 1. Mueva la implementación a una red virtual clásica diferente antes de la migración. Esto provocará un tiempo de inactividad. <br> 2. Migre la nueva red virtual a Azure Resource Manager. <br><br> Or <br><br> 1. Migre la red virtual a Azure Resource Manager. <br>2. Mueva Cloud Services a una nueva red virtual. Esto provocará un tiempo de inactividad. | 
| Cloud Services en una red virtual, pero sin tener asignada una subred explícita | No compatible. La mitigación implica mover el rol a una subred, lo que requiere un reinicio de rol (tiempo de inactividad) | 


## <a name="post-migration-changes"></a>Cambios posteriores a la migración
La implementación de Cloud Services (clásico) se convierte en una implementación de Cloud Services (soporte extendido). Consulte la [documentación de Cloud Services (soporte extendido)](deploy-prerequisite.md) para obtener más información.  

### <a name="changes-to-deployment-files"></a>Cambios en los archivos de implementación 

Se realizan cambios menores en los archivos .csdef y .cscfg del cliente para que los archivos de implementación cumplan con los requisitos de Azure Resource Manager y Cloud Services (soporte extendido). Después de la migración, se recuperan los nuevos archivos de implementación o se actualizan los archivos existentes. Esto será necesario para las operaciones de actualización y eliminación.  

- Virtual Network usa el identificador de recurso de Azure Resource Manager completo en lugar de simplemente el nombre del recurso en la sección NetworkConfiguration del archivo .cscfg. Por ejemplo, `/subscriptions/subscription-id/resourceGroups/resource-group-name/providers/Microsoft.Network/virtualNetworks/vnet-name`. En el caso de las redes virtuales que pertenecen al mismo grupo de recursos que el servicio en la nube, puede optar por actualizar el archivo .cscfg para que vuelva a usar solo el nombre de la red virtual.  

- Los tamaños clásicos, como Small, Large y ExtraLarge, se sustituyen por los nuevos nombres de tamaño, Standard_A*. Los nombres de tamaño deben cambiarse a sus nuevos nombres en el archivo .csdef. Para obtener más información, consulte [Requisitos previos para implementar Cloud Services (soporte extendido)](deploy-prerequisite.md#required-service-definition-file-csdef-updates).

- Use la API Get para obtener la copia más reciente de los archivos de implementación. 
    - Obtenga la plantilla mediante el [portal](https://docs.microsoft.com/azure/azure-resource-manager/templates/export-template-portal), [PowerShell](https://docs.microsoft.com/azure/azure-resource-manager/management/manage-resource-groups-powershell#export-resource-groups-to-templates), la [CLI](https://docs.microsoft.com/azure/azure-resource-manager/management/manage-resource-groups-cli#export-resource-groups-to-templates) y la [API REST](https://docs.microsoft.com/rest/api/resources/resourcegroups/exporttemplate). 
    - Obtenga el archivo .csdef mediante [PowerShell](https://docs.microsoft.com/powershell/module/az.cloudservice/?view=azps-5.4.0#cloudservice&preserve-view=true) o la [API REST](https://docs.microsoft.com/rest/api/compute/cloudservices/rest-get-package). 
    - Obtenga el archivo .cscfg mediante [PowerShell](https://docs.microsoft.com/powershell/module/az.cloudservice/?view=azps-5.4.0#cloudservice&preserve-view=true) o la [API REST](https://docs.microsoft.com/rest/api/compute/cloudservices/rest-get-package). 
    
 

### <a name="changes-to-customers-automation-cicd-pipeline-custom-scripts-custom-dashboards-custom-tooling-etc"></a>Cambios en la automatización del cliente, la canalización de CI/CD, los scripts personalizados, los paneles personalizados, las herramientas personalizadas, etc.  

Los clientes tienen que actualizar sus herramientas y automatización para empezar a usar las nuevas API y comandos para administrar su implementación. Los clientes pueden adoptar fácilmente las nuevas características y funcionalidades de Azure Resource Manager y Cloud Services (soporte extendido) como parte de este cambio. 

- Cambios en los nombres de recursos y grupos de recursos después de la migración
    - Como parte de la migración, cambian los nombres de unos pocos recursos, como el servicio en la nube, las direcciones IP públicas, etc. Es posible que estos cambios deban reflejarse en los archivos de implementación antes de actualizar el servicio en la nube. [Obtenga más información sobre los nombres de los recursos que cambian](in-place-migration-technical-details.md#translation-of-resources-and-naming-convention-post-migration).  

- Vuelva a crear las reglas y directivas necesarias para administrar y escalar servicios en la nube 
    - Las [reglas de escalabilidad automática](configure-scaling.md) no se migran. Después de la migración, vuelva a crear las reglas de escalabilidad automática.  
    - Las [alertas](enable-alerts.md) no se migran. Después de la migración, vuelva a crear las alertas.
    - Key Vault se crea sin ninguna directiva de acceso. [Cree las directivas adecuadas](../key-vault/general/assign-access-policy-portal.md) en Key Vault para ver o administrar los certificados. Los certificados estarán visibles en configuración en la pestaña denominada Secretos.

## <a name="next-steps"></a>Pasos siguientes
- [Migración compatible con la plataforma de recursos de IaaS desde el modelo clásico al de Azure Resource Manager](../virtual-machines/migration-classic-resource-manager-overview.md)
- Migración a Cloud Services (soporte extendido) mediante [Azure Portal](in-place-migration-portal.md)
- Migración a Cloud Services (soporte extendido) mediante [PowerShell](in-place-migration-powershell.md)
