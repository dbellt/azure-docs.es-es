---
title: 'Habilitación de la eliminación temporal en todos los objetos de almacenes de claves: Azure Key Vault | Microsoft Docs'
description: Utilice este documento para adoptar la eliminación temporal para todos los almacenes de claves y para realizar cambios en la aplicación y en la administración con el fin de evitar errores de conflicto.
services: key-vault
author: msmbaldwin
tags: azure-resource-manager
ms.service: key-vault
ms.topic: conceptual
ms.date: 03/31/2021
ms.author: mbaldwin
ms.openlocfilehash: 7e1b2ee95864affa6e5e72e1f8354767dc95bdb1
ms.sourcegitcommit: 6686a3d8d8b7c8a582d6c40b60232a33798067be
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107753333"
---
# <a name="soft-delete-will-be-enabled-on-all-key-vaults"></a>La eliminación temporal se habilitará en todos los almacenes de claves

> [!WARNING]
> Cambio de última hora: la posibilidad de optar por no realizar la eliminación temporal pronto estará en desuso. Los usuarios y administradores de Azure Key Vault deben habilitar la eliminación temporal en sus almacenes de claves de forma inmediata.
>
> Para HSM administrado de Azure Key Vault, la eliminación temporal está habilitada de forma predeterminada y no se puede deshabilitar.

Cuando se elimina un secreto de un almacén de claves sin protección contra la eliminación temporal, el secreto se elimina de forma permanente. Los usuarios pueden optar por no realizar la eliminación temporal durante la creación del almacén de claves. Sin embargo, Microsoft pronto habilitará la protección contra la eliminación temporal en todos los almacenes de claves para proteger los secretos de la eliminación accidental o malintencionada de un usuario. Los usuarios ya no podrán optar por realizar ni desactivar la eliminación temporal.

:::image type="content" source="../media/softdeletediagram.png" alt-text="Diagrama que muestra cómo se elimina un almacén de claves con protección contra la eliminación temporal frente no tener protección.":::

Para obtener detalles completos sobre la funcionalidad de eliminación temporal, consulte [Información general sobre la eliminación temporal de Azure Key Vault](soft-delete-overview.md).

## <a name="can-my-application-work-with-soft-delete-enabled"></a>¿Puede mi aplicación funcionar con la eliminación temporal habilitada?

> [!Important] 
> Revise atentamente la siguiente información antes de activar la eliminación temporal para los almacenes de claves.

Los nombres de los almacenes de claves son únicos globalmente. Los nombres de los secretos almacenados en un almacén de claves también son únicos. No podrá volver a usar el nombre de un almacén de claves o de un objeto de almacén de claves que exista en el estado de eliminación temporal. 

Por ejemplo, si la aplicación crea mediante programación un almacén de claves denominado "Almacén A" y, posteriormente, lo elimina, el almacén de claves pasará al estado de eliminación temporal. La aplicación no podrá volver a crear otro almacén de claves denominado "Almacén A" hasta que el almacén de claves se purgue del estado de eliminación temporal. 

Además, si la aplicación crea una clave denominada `test key` en ”Almacén A” y, posteriormente, la elimina, la aplicación no podrá crear una clave denominada `test key` en ”Almacén A” hasta que el objeto `test key` se purgue del estado de eliminación temporal. 

Si se intenta eliminar un objeto del almacén de claves y se vuelve a crearlo con el mismo nombre sin purgarlo primero del estado de eliminación temporal, se pueden causar errores de conflicto. Esto puede provocar errores en las aplicaciones o la automatización. Antes de realizar los cambios necesarios en la aplicación y la administración, póngase en contacto con el equipo de desarrollo. 

### <a name="application-changes"></a>Cambios en la aplicación

Si la aplicación supone que la eliminación temporal no está habilitada y espera que los nombres del secreto o del almacén de claves eliminados estén disponibles para su reutilización inmediata, tendrá que realizar los siguientes cambios en la lógica de aplicación.

1. Elimine el almacén de claves o el secreto original.
1. Purgar el almacén de claves o el secreto con el estado de eliminación temporal.
1. Espere a que se complete la purga. La nueva creación inmediata podría producir un conflicto.
1. Volver a crear el almacén de claves con el mismo nombre.
1. Si la operación de creación sigue produciendo un error de conflicto de nombres, intente volver a crear el almacén de claves. Los registros de Azure DNS pueden tardar hasta 10 minutos en actualizarse en el peor escenario posible.

### <a name="administration-changes"></a>Cambios en la administración

A las entidades de seguridad que necesitan acceso para eliminar secretos de forma permanente se les deben conceder más permisos de directiva de acceso para purgar estos secretos y el almacén de claves.

Deshabilite cualquier directiva de Azure en los almacenes de claves que exija que la eliminación temporal esté desactivada. Es posible que necesite escalar este problema a un administrador que controle las directivas de Azure que se aplican a su entorno. Si no se deshabilita esta directiva, puede perder la capacidad de crear nuevos almacenes de claves en el ámbito de la directiva aplicada.

Si su organización está sujeta a requisitos legales de cumplimiento normativo y no puede permitir que los almacenes de claves y secretos eliminados permanezcan en un estado recuperable durante un período de tiempo prolongado, tendrá que ajustar el período de retención de la eliminación temporal para cumplir los estándares de su organización. Puede configurar el período de retención para que dure de 7 a 90 días.

## <a name="procedures"></a>Procedimientos

### <a name="audit-your-key-vaults-to-check-if-soft-delete-is-enabled"></a>Auditoría de los almacenes de claves para comprobar si la eliminación temporal está habilitada

1. Inicie sesión en Azure Portal.
1. Busque **Azure Policy**.
1. Seleccione **Definiciones**.
1. En **Categoría**, seleccione **Key Vault** en el filtro.
1. Seleccione la directiva **Key Vault should have soft delete enabled** (El almacén de claves debe tener habilitada la eliminación temporal).
1. Seleccione **Asignar**.
1. Establezca el ámbito de la suscripción.
1. Asegúrese de que el efecto de la directiva se establece en **Auditar**.
1. Seleccione **Revisar + crear**. Un examen completo de su entorno puede tardar hasta 24 horas en realizarse.
1. En el panel de **Azure Policy**, seleccione **Cumplimiento**.
1. Seleccione la directiva que ha aplicado.

Ahora ya puede filtrar y ver cuáles de los almacenes de claves tienen habilitada la eliminación temporal (recursos compatibles) y cuáles no (recursos no compatibles).

### <a name="turn-on-soft-delete-for-an-existing-key-vault"></a>Activación de la eliminación temporal para un almacén de claves existente

1. Inicie sesión en Azure Portal.
1. Busque su instancia de Key Vault.
1. En **Configuración**, seleccione **Propiedades**.
1. En **Eliminación temporal**, seleccione la opción **Enable recovery of this vault and its objects** (Habilitar la recuperación de este almacén y de sus objetos).
1. Establezca el período de retención para la eliminación temporal.
1. Seleccione **Guardar**.

### <a name="grant-purge-access-policy-permissions-to-a-security-principal"></a>Concesión de permisos de directiva de acceso de depuración a una entidad de seguridad

1. Inicie sesión en Azure Portal.
1. Busque su instancia de Key Vault.
1. Seleccione **Directivas de acceso** en **Configuración**.
1. Seleccione la entidad de servicio a la que le gustaría conceder acceso.
1. Desplácese por los menús desplegables en **Clave**, **Secreto** y **Permisos de certificado** hasta que vea las **operaciones con privilegios**. Seleccione el permiso **Purgar**.

## <a name="frequently-asked-questions"></a>Preguntas más frecuentes

### <a name="does-this-change-affect-me"></a>¿Me afecta este cambio?

Si ya tiene activada la eliminación temporal o si no elimina y vuelve a crear objetos del almacén de claves con el mismo nombre, es probable que no observe ningún cambio en el comportamiento del almacén de claves.

Si tiene una aplicación que elimina y vuelve a crear objetos del almacén de claves con las mismas convenciones de nomenclatura con frecuencia, tendrá que realizar cambios en la lógica de la aplicación para mantener el comportamiento esperado. Consulte la sección [Cambios en la aplicación](#application-changes) en este artículo.

### <a name="how-do-i-benefit-from-this-change"></a>¿Cómo puedo aprovechar este cambio?

La protección contra la eliminación temporal proporciona a su organización otra capa de protección frente a la eliminación accidental o malintencionada. Como administrador del almacén de claves, puede restringir el acceso a los permisos de recuperación y de purga.

Si un usuario elimina accidentalmente un almacén de claves o un secreto, puede concederle permisos de acceso para recuperar el secreto sin crear el riesgo de que elimine permanentemente el secreto o el almacén de claves. Este proceso de autoservicio minimizará el tiempo de inactividad en el entorno y garantizará la disponibilidad de los secretos.

### <a name="how-do-i-find-out-if-i-need-to-take-action"></a>¿Cómo puedo averiguar si es necesario realizar alguna acción?

Siga los pasos de la sección [Auditoría de los almacenes de claves para comprobar si la eliminación temporal está habilitada](#audit-your-key-vaults-to-check-if-soft-delete-is-enabled) en este artículo. Este cambio afectará a cualquier almacén de claves que no tenga activada la eliminación temporal.

### <a name="what-action-do-i-need-to-take"></a>¿Qué acción debo llevar a cabo?

Una vez que haya confirmado que no tiene que realizar cambios en la lógica de la aplicación, active la eliminación temporal en todos los almacenes de claves.

### <a name="when-do-i-need-to-take-action"></a>¿Cuándo debo realizar alguna acción?

Para asegurarse de que las aplicaciones no se vean afectadas, active la eliminación temporal en los almacenes de claves lo antes posible.

## <a name="next-steps"></a>Pasos siguientes

- Póngase en contacto con nosotros si tiene alguna pregunta sobre este cambio en [akvsoftdelete@microsoft.com](mailto:akvsoftdelete@microsoft.com).
- Consulte la [introducción a la eliminación temporal](soft-delete-overview.md).
