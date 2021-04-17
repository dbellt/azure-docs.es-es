---
title: 'Procedimiento de migración: portal'
description: Procedimiento de migración a Cloud Services (soporte extendido) mediante Azure Portal
ms.topic: how-to
ms.service: cloud-services-extended-support
ms.subservice: classic-to-arm-migration
author: tanmaygore
ms.author: tagore
ms.reviewer: mimckitt
ms.date: 2/08/2021
ms.custom: ''
ms.openlocfilehash: 79889b08baa80dc67b30ae445004e37d9f9fe295
ms.sourcegitcommit: b8995b7dafe6ee4b8c3c2b0c759b874dff74d96f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/03/2021
ms.locfileid: "106286768"
---
# <a name="migrate-to-cloud-services-extended-support-using-the-azure-portal"></a>Migración a Cloud Services (soporte extendido) mediante Azure Portal

En este artículo se muestra cómo usar Azure Portal para migrar de [Cloud Services (clásico)](../cloud-services/cloud-services-choose-me.md) a [Cloud Services (soporte extendido)](overview.md).

> [!IMPORTANT]
> La migración de Cloud Services (clásico) a Cloud Services (soporte extendido) mediante la herramienta de migración se encuentra actualmente en versión preliminar pública. Esta versión preliminar se ofrece sin Acuerdo de Nivel de Servicio y no se recomienda para cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas. Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="before-you-begin"></a>Antes de comenzar

**Asegurarse de ser administrador de la suscripción.**

Para realizar esta migración, debe estar agregado como coadministrador de la suscripción en [Azure Portal](https://portal.azure.com).

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
2. En el menú de **Central**, seleccione **Suscripción**. Si no lo ve, haga clic en **Todos los servicios**.
3. Busque la entrada de la suscripción adecuada y después examine el campo **MI ROL**. En el caso de un coadministrador, el valor debe ser *Administrador de cuenta*.

Si no puede agregar un coadministrador, póngase en contacto con un administrador del servicio o [coadministrador](../role-based-access-control/classic-administrators.md) de la suscripción para que le agreguen.

**Registro en el proveedor de recursos de migración**

1. Regístrese con el proveedor de recursos de migración `Microsoft.ClassicInfrastructureMigrate` y la característica de versión preliminar `Cloud Services` en el espacio de nombres Microsoft.Compute mediante [Azure Portal](https://docs.microsoft.com/azure/azure-resource-manager/management/resource-providers-and-types#register-resource-provider-1).  
1. Espere cinco minutos para que se complete el registro y luego compruebe el estado de la aprobación. 

## <a name="migrate-your-cloud-service-resources"></a>Migración de los recursos de Cloud Services

1. Vaya a la [hoja del portal Cloud Services (clásico)](https://ms.portal.azure.com/#blade/HubsExtension/BrowseResourceBlade/resourceType/microsoft.classicCompute%2FdomainNames). 
2. Seleccione la instancia de Cloud Services que quiere migrar.
3. Seleccione la hoja **Migrar a ARM**.

    > [!NOTE]
    > Si va a migrar una instancia de Cloud Services (clásico) ubicada dentro de una red virtual (clásica), aparecerá un mensaje de banner en el que se le pedirá que mueva la hoja de la red virtual (clásica).
    > Se le dirigirá a la hoja de la red virtual (clásica) para completar la migración de las implementaciones tanto de la red virtual (clásica) como de Cloud Services (clásico) dentro de ella.
    > :::image type="content" source="media/in-place-migration-portal-2.png" alt-text="La imagen muestra cómo mover una red virtual clásica en Azure Portal.":::
 

4. Valide la migración. 

    Si la validación es correcta, todas las implementaciones se admiten y están listas para su preparación.  

    :::image type="content" source="media/in-place-migration-portal-1.png" alt-text="Imagen que muestra la hoja Migrar a ARM en Azure Portal":::

    Si se produce un error en la validación, se mostrará una lista de escenarios no admitidos, y será necesario corregirlos para que la migración pueda continuar. 

    :::image type="content" source="media/in-place-migration-portal-3.png" alt-text="Imagen que muestra un error de validación en Azure Portal.":::

5. Prepárese para la migración.

    Si la preparación se completa correctamente, la migración está lista para confirmarse.
    
    :::image type="content" source="media/in-place-migration-portal-4.png" alt-text="Imagen que muestra la superación de la validación en Azure Portal.":::

    Si se produce un error en la preparación, revise el error, solucione los problemas y vuelva a intentar la preparación. 

    :::image type="content" source="media/in-place-migration-portal-5.png" alt-text="Imagen que muestra un error de validación.":::

      Después de la preparación, todas las instancias de Cloud Services en una red virtual están disponibles para operaciones de lectura mediante la hoja de Cloud Services (clásico) y Cloud Services (soporte extendido) en Azure Portal. La implementación de Cloud Services (soporte extendido) ahora se puede probar para asegurar el correcto funcionamiento antes de finalizar la migración. 
 
    :::image type="content" source="media/in-place-migration-portal-6.png" alt-text="Imagen que muestra las API de prueba en la hoja del portal.":::

6.  **(Opcional)** Anule la migración. 
    
    Si decide interrumpir la migración, use el botón **Anular** para revertir los pasos anteriores. A continuación, se desbloquea la implementación de Cloud Services (clásico) para todas las operaciones CRUD.

    :::image type="content" source="media/in-place-migration-portal-7.png" alt-text="Imagen que muestra la superación de la validación.":::

    Si no se puede anular, seleccione **Retry abort** (Reintentar anulación). Un reintento debería corregir el problema. De lo contrario, póngase en contacto con el servicio de soporte técnico. 
 
    :::image type="content" source="media/in-place-migration-portal-8.png" alt-text="Imagen que muestra un mensaje de error de validación.":::

7.  Confirme la migración.

    >[!IMPORTANT]
    > Una vez que confirme la migración, no hay ninguna opción para revertir. 
    
    Escriba "sí" para aceptar y confirmar la migración. La migración ahora está completa. La implementación de Cloud Services (soporte extendido) migrada está desbloqueada para todas las operaciones. 

## <a name="next-steps"></a>Pasos siguientes
Revise la sección [Cambios posteriores a la migración](in-place-migration-overview.md#post-migration-changes) para ver los cambios en los archivos de implementación, la automatización y otros atributos de la nueva implementación de Cloud Services (soporte extendido). 
