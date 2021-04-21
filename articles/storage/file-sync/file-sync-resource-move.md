---
title: Traslado de recursos y cambios de topología de Azure File Sync
description: Obtenga información sobre cómo trasladar recursos de sincronización entre grupos de recursos, suscripciones e inquilinos de Azure Active Directory (AAD).
author: fauhse
ms.service: storage
ms.topic: how-to
ms.date: 04/13/2021
ms.author: fauhse
ms.subservice: files
ms.openlocfilehash: fb59b8d48d8411ade620304fbd143d86ac5ae919
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107796435"
---
# <a name="move-azure-file-sync-resources-to-a-different-resource-group-subscription-or-aad-tenant"></a>Traslado de recursos de Azure File Sync a otro grupo de recursos, suscripción o inquilino de AAD

En este artículo se describe cómo realizar cambios a un grupo de recursos, suscripción o inquilino de Azure Active Directory (AAD) para sus recursos de nube de Azure File Sync y cuentas de Azure Storage.

Al planear la realización de cambios en los recursos de nube de Azure File Sync, es importante tener en cuenta también a los recursos de almacenamiento. Están disponibles los siguientes recursos:

**Recursos de Azure File Sync (en orden jerárquico)**

* :::image type="icon" source="media/storage-sync-resource-move/storage-sync-resource-move-storage-sync-service.png" border="false"::: Servicio de sincronización de almacenamiento
  * :::image type="icon" source="media/storage-sync-resource-move/storage-sync-resource-move-registered-servers.png" border="false"::: Servidor registrado
  * :::image type="icon" source="media/storage-sync-resource-move/storage-sync-resource-move-sync-group.png" border="false"::: Grupo de sincronización
    * :::image type="icon" source="media/storage-sync-resource-move/storage-sync-resource-move-cloud-endpoint.png" border="false"::: Punto de conexión en la nube
    * :::image type="icon" source="media/storage-sync-resource-move/storage-sync-resource-move-server-endpoint.png" border="false"::: Punto de conexión de servidor

En Azure File Sync, el único recurso capaz de trasladarse es el recurso de servicio de sincronización de almacenamiento. Todos los subrecursos están enlazados a su elemento primario y no se pueden trasladar a otro servicio de sincronización de almacenamiento.

**Recursos de Azure Storage (en orden jerárquico)**

* :::image type="icon" source="media/storage-sync-resource-move/storage-sync-resource-move-storage-account.png" border="false"::: Cuenta de almacenamiento
    * :::image type="icon" source="media/storage-sync-resource-move/storage-sync-resource-move-file-share.png" border="false"::: Recurso compartido de archivos

En Azure Storage, el único recurso capaz de trasladarse es la cuenta de almacenamiento. Un recurso compartido de archivos de Azure no se puede trasladar a una cuenta de almacenamiento diferente, ya que es un subrecurso.

## <a name="supported-combinations"></a>Combinaciones admitidas

Al planear un traslado de recursos, la cuenta de almacenamiento y el recurso de Azure File Sync de nivel superior, denominado *servicio de sincronización de almacenamiento*, se deben considerar en conjunto.

Como procedimiento recomendado, el servicio de sincronización de almacenamiento y las cuentas de almacenamiento que tienen recursos compartidos de archivos que se sincronizarán siempre deben residir en la misma suscripción. Se admiten las siguientes combinaciones:

* Las cuentas de almacenamiento y el servicio de sincronización de almacenamiento se encuentran en **grupos de recursos distintos** (en el mismo inquilino de Azure)
* Las cuentas de almacenamiento y el servicio de sincronización de almacenamiento se encuentran en **suscripciones distintas** (en el mismo inquilino de Azure)

> [!IMPORTANT]
> Debido a diferentes combinaciones de movimientos, un servicio de sincronización de almacenamiento y las cuentas de almacenamiento pueden acabar en distintas suscripciones, que se rigen por distintos inquilinos de AAD. Incluso parecería que la sincronización funciona, pero esta no es una configuración admitida. La sincronización puede detenerse más adelante, sin la posibilidad de volver a un estado en el que funcione.

Al planear el traslado de recursos, existen diferentes consideraciones para el [traslado dentro del mismo inquilino de AAD](#move-within-the-same-azure-active-directory-tenant) y para el traslado [a otro inquilino de AAD](#move-to-a-new-azure-active-directory-tenant). Al realizar un traslado entre inquilinos de AAD, siempre debe mover los recursos de sincronización y almacenamiento a la vez.

### <a name="move-within-the-same-azure-active-directory-tenant"></a>Traslado en el mismo inquilino de Azure Active Directory

:::row:::
    :::column:::
        :::image type="content" source="media/storage-sync-resource-move/storage-sync-resource-move-small.png" alt-text="Imagen que muestra Azure Portal para un recurso del servicio de sincronización de almacenamiento, con el comando de movimiento expandido. Muestra las opciones de movimiento de la suscripción y del grupo de recursos." lightbox="media/storage-sync-resource-move/storage-sync-resource-move.png":::
    :::column-end:::
    :::column:::
        Azure Portal es un método práctico para trasladar un recurso de servicio de sincronización de almacenamiento. Navegue hasta al servicio de sincronización de almacenamiento que quiere trasladar y seleccione **Mover** desde la barra de comandos. Siga los mismos pasos para trasladar una cuenta de almacenamiento. También puede trasladar todos los recursos de un grupo de recursos de esta forma. Se recomienda trasladar el grupo de recursos completo si el servicio de sincronización de almacenamiento y todas las cuentas de almacenamiento que usa están en dicho grupo de recursos.
    :::column-end:::
:::row-end:::

> [!WARNING]
> Al mover un recurso de cuenta de almacenamiento, la sincronización se detendrá inmediatamente. Tendrá que autorizar manualmente la sincronización para acceder a las cuentas de almacenamiento pertinentes en la nueva suscripción. En la sección [Autorización de acceso al almacenamiento de Azure File Sync](#azure-file-sync-storage-access-authorization) se especificarán los pasos necesarios.

### <a name="move-to-a-new-azure-active-directory-tenant"></a>Traslado a un nuevo inquilino de Azure Active Directory

Los recursos individuales, como un servicio de sincronización de almacenamiento o las cuentas de almacenamiento, no se pueden trasladar a un inquilino de AAD diferente. Solo las suscripciones a Azure pueden trasladar inquilinos de AAD. Piense en la estructura de la suscripción en el nuevo inquilino de AAD. Puede usar una suscripción dedicada para Azure File Sync. 

1. Cree una suscripción de Azure (o bien, identifique una existente en el inquilino anterior que deba trasladarse).
1. [Realice el traslado de una suscripción en el mismo inquilino de AAD](#move-within-the-same-azure-active-directory-tenant) del servicio de sincronización de almacenamiento y todas las cuentas de almacenamiento asociadas.
1. La sincronización se detendrá. Complete el traslado del inquilino de inmediato o [restaure la funcionalidad de sincronización para acceder a las cuentas de almacenamiento que ha trasladado](#azure-file-sync-storage-access-authorization). A continuación, podrá pasar al nuevo inquilino de AAD.

Una vez que todos los recursos de Azure File Sync relacionados se han capturado en una suscripción, está listo para trasladar toda la suscripción al inquilino de AAD de destino. La [guía de transferencia de suscripción](../../role-based-access-control/transfer-subscription.md) le permite planear y ejecutar este tipo de transferencia.

> [!WARNING]
> Cuando transfiera una suscripción de un inquilino a otro, la sincronización se detendrá inmediatamente. Tendrá que autorizar manualmente la sincronización para acceder a las cuentas de almacenamiento pertinentes en la nueva suscripción. En la sección [Autorización de acceso al almacenamiento de Azure File Sync](#azure-file-sync-storage-access-authorization) se especificarán los pasos necesarios.

:::row:::
    :::column:::
        :::image type="content" source="media/storage-sync-resource-move/storage-sync-resource-move-aad-tenant.png" alt-text="Una imagen en la que se muestra la hoja Información general de suscripción en Azure Portal, donde está resaltado el comando Cambiar directorio de la barra de herramientas en el centro de la parte superior de la página." lightbox="media/storage-sync-resource-move/storage-sync-resource-move-aad-tenant-expanded.png":::
    :::column-end:::
    :::column:::
        Está listo para iniciar la migración una vez que tenga un plan y los permisos necesarios:
        1. En Azure Portal, navegue a la hoja **Información general** de la suscripción.
        1. Seleccione **Cambiar directorio**.
        1. Siga los pasos del Asistente para asignar el nuevo inquilino de AAD.
    :::column-end:::
:::row-end:::

## <a name="azure-file-sync-storage-access-authorization"></a>Autorización de acceso al almacenamiento de Azure File Sync

Cuando las cuentas de almacenamiento se trasladen a una nueva suscripción o dentro de una suscripción a un nuevo inquilino de Azure Active Directory (AAD), la sincronización se detendrá. El acceso basado en roles (RBAC) se usa para autorizar a Azure File Sync el acceso a una cuenta de almacenamiento, y las asignaciones de roles no se migran con los recursos.

### <a name="azure-file-sync-service-principal"></a>Entidad de servicio de Azure File Sync

:::row:::
    :::column:::
        :::image type="content" source="media/storage-sync-resource-move/storage-sync-resource-move-afs-rp-registered-small.png" alt-text="Imagen que muestra los proveedores de recursos registrados en la administración de suscripciones de Azure Portal." lightbox="media/storage-sync-resource-move/storage-sync-resource-move-afs-rp-registered.png":::
    :::column-end:::
    :::column:::
        La entidad de servicio de Azure File Sync debe existir en el inquilino de AAD para poder autorizar el acceso de sincronización a una cuenta de almacenamiento. </br></br> Cuando cree una nueva suscripción de Azure actualmente, el proveedor de recursos de Azure File Sync denominado *Microsoft.StorageSync* se registrará automáticamente con su suscripción. El registro del proveedor de recursos hará que esté disponible una *entidad de servicio* para la sincronización en el inquilino de Azure Active Directory que rige la suscripción. Una entidad de servicio es similar a una cuenta de usuario en AAD. Puede usar la entidad de servicio de Azure File Sync para autorizar el acceso a los recursos mediante el control de acceso basado en rol (RBAC). La sincronización de recursos única debe tener acceso a las cuentas de almacenamiento que contengan los recursos compartidos de archivos que se supone que se van a sincronizar. *Microsoft. StorageSync* debe estar asignado al rol integrado **Lector y acceso a los datos** de la cuenta de almacenamiento. </br></br> Esta asignación se realiza automáticamente a través del contexto del usuario que ha iniciado sesión cuando se agrega un recurso compartido de archivos a un grupo de sincronización; o, en otras palabras, cuando se crea un punto de conexión de nube. Cuando una cuenta de almacenamiento se traslada a una nueva suscripción o inquilino de AAD, esta asignación de roles se pierde y [se debe restablecer manualmente](#establish-sync-access-to-a-storage-account).
    :::column-end:::
:::row-end:::

> [!IMPORTANT]
> Si la suscripción a Azure de destino no se ha creado recientemente, compruebe que el proveedor de recursos *Microsoft.StorageSync* está registrado con la suscripción. Si no es así, agréguelo manualmente en la misma hoja del portal.

### <a name="establish-sync-access-to-a-storage-account"></a>Establecimiento de acceso de sincronización a una cuenta de almacenamiento

La [entidad de servicio de Azure File Sync](#azure-file-sync-service-principal) se debe usar para autorizar el acceso a una cuenta de almacenamiento mediante el control de acceso basado en rol (RBAC). *Microsoft.StorageSync* debe tener asignado el rol integrado **Lector y acceso a los datos** de la cuenta de almacenamiento. 

Normalmente, esta asignación se realiza automáticamente a través del contexto del usuario que ha iniciado sesión cuando se agrega un recurso compartido de archivos a un grupo de sincronización; o, en otras palabras, cuando se crea un punto de conexión de nube. No obstante, cuando una cuenta de almacenamiento se traslada a una nueva suscripción o inquilino de AAD, esta asignación de roles se pierde y se debe restablecer manualmente.

:::row:::
    :::column:::
        :::image type="content" source="media/storage-sync-resource-move/storage-sync-resource-move-assign-rbac.png" alt-text="Imagen que muestra la entidad de servicio Microsoft.StorageSync asignada al rol Lector y acceso a los datos en una cuenta de almacenamiento":::
    :::column-end:::
    :::column:::
        En Azure Portal, vaya a la cuenta de almacenamiento para la que necesita volver a autorizar el acceso. <ol><li>Seleccione **Control de acceso (IAM)** en la tabla de contenido de la izquierda.</li><li>Seleccione la pestaña Asignaciones de roles de la lista de los usuarios y aplicaciones (entidades de servicio) que tienen acceso a su cuenta de almacenamiento.</li><li>Seleccione **Agregar**.</li><li>En el **campo Rol**, seleccione **Lector y acceso a los datos**.</li><li>En el **campo Seleccionar**, escriba *Microsoft.StorageSync*, seleccione el rol y haga clic en **Guardar**.</li></ol>
    :::column-end:::
:::row-end:::

## <a name="move-to-a-different-azure-region"></a>Traslado a una región distinta de Azure

El recurso de Azure File Sync denominado *servicio de sincronización de almacenamiento* y las cuentas de almacenamiento que contienen recursos compartidos de archivos que se están sincronizando se implementaron en alguna región de Azure. La región se determina cuando se crea el recurso. Las regiones del servicio de sincronización de almacenamiento y la cuenta de almacenamiento deben coincidir. Estas regiones no se pueden cambiar en ningún tipo de recurso después de su creación.

Asignar una región distinta a un recurso es diferente de la [conmutación por error de una región](#region-fail-over), que puede admitirse en función de la configuración de redundancia de la cuenta de almacenamiento.

## <a name="region-fail-over"></a>Conmutación por error de región

[Azure Storage ofrece opciones de redundancia geográfica](../common/storage-redundancy.md#geo-redundant-storage) para una cuenta de almacenamiento. Estas opciones de redundancia pueden plantear problemas en las cuentas de almacenamiento que se usan con Azure File Sync. La razón principal es que la replicación entre regiones geográficamente distantes no se realiza mediante Azure File Sync, sino con una tecnología de replicación de almacenamiento integrada en el subsistema de almacenamiento de Azure. No puede conocer el estado de la aplicación, y Azure File Sync es una aplicación con archivos que se sincronizan con recursos compartidos de archivos de Azure como origen y destino en todo momento. Si elige cualquiera de estas opciones de redundancia de almacenamiento dispersas geográficamente, no perderá todos los datos en un desastre a gran escala. Sin embargo, debe [anticiparse a la pérdida de datos](../common/storage-disaster-recovery-guidance.md#anticipate-data-loss).

> [!CAUTION]
> La conmutación por error nunca es un sustituto adecuado para aprovisionar los recursos en la región de Azure correcta. Si los recursos se encuentran en la región "incorrecta", debe considerar la posibilidad de detener la sincronización y configurarla nuevamente con los nuevos recursos compartidos de archivos de Azure que se implementaron en la región deseada.

Microsoft puede iniciar una conmutación por error regional durante un evento catastrófico que haga que los centros de datos en una región de Azure se deshabiliten durante un período de tiempo prolongado. La definición del tiempo de inactividad que su empresa puede admitir podría ser menor que el tiempo que Microsoft está preparado para dejar pasar antes de iniciar una conmutación por error regional. En una situación como esta, [los clientes también puede iniciar las conmutaciones por error](../common/storage-initiate-account-failover.md).

> [!IMPORTANT]
> En el caso de una conmutación por error, debe abrir una incidencia de soporte técnico para los servicios de sincronización de almacenamiento afectados a fin de que la sincronización funcione de nuevo.

## <a name="see-also"></a>Consulte también

- [Información general sobre las guías de migración de recursos compartidos de Azure y Azure File Sync](../files/storage-files-migration-overview.md?toc=%2fazure%2fstorage%2ffilesync%2ftoc.json)
- [Solución de problemas de Azure File Sync](file-sync-troubleshoot.md)
- [Planeamiento de una implementación de Azure File Sync](file-sync-planning.md)