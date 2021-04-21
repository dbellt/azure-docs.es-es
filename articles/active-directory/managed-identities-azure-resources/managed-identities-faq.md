---
title: 'Preguntas más frecuentes sobre identidades administradas para recursos de Azure: Azure AD'
description: Preguntas más frecuentes sobre identidades administradas
services: active-directory
documentationcenter: ''
author: barclayn
manager: daveba
editor: ''
ms.service: active-directory
ms.subservice: msi
ms.devlang: ''
ms.topic: conceptual
ms.tgt_pltfrm: ''
ms.workload: identity
ms.date: 04/08/2021
ms.author: barclayn
ms.openlocfilehash: 0d6527c27420617728428e440c94a60236701212
ms.sourcegitcommit: 2654d8d7490720a05e5304bc9a7c2b41eb4ae007
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/13/2021
ms.locfileid: "107376773"
---
# <a name="managed-identities-for-azure-resources-frequently-asked-questions---azure-ad"></a>Preguntas más frecuentes sobre identidades administradas para recursos de Azure: Azure AD

[!INCLUDE [preview-notice](../../../includes/active-directory-msi-preview-notice.md)]

> [!NOTE]
> Identidades administradas para recursos de Azure es el nombre con el que ahora se conoce al servicio Managed Service Identity (MSI).

## <a name="administration"></a>Administración

### <a name="how-can-you-find-resources-that-have-a-managed-identity"></a>¿Cómo se pueden encontrar los recursos que tienen una identidad administrada?

Puede encontrar la lista de recursos con una identidad administrada asignada por el sistema mediante el siguiente comando de la CLI de Azure: 

```azurecli-interactive
az resource list --query "[?identity.type=='SystemAssigned'].{Name:name,  principalId:identity.principalId}" --output table
```


### <a name="what-azure-rbac-permissions-are-required-to-managed-identity-on-a-resource"></a>¿Qué permisos RBAC de Azure son necesarios para una identidad administrada en un recurso? 

- Identidad administrada asignada por el sistema: Se necesitan permisos de escritura sobre el recurso. Por ejemplo, para las máquinas virtuales es necesario Microsoft.Compute/virtualMachines/write. Esta acción se incluye en los roles integrados específicos del recurso como, por ejemplo, [Colaborador de máquina virtual](../../role-based-access-control/built-in-roles.md#virtual-machine-contributor).
- Identidad administrada asignada por el usuario: Se necesitan permisos de escritura sobre el recurso. Por ejemplo, para las máquinas virtuales es necesario Microsoft.Compute/virtualMachines/write. Además de la asignación de roles [Operador de identidad administrada](../../role-based-access-control/built-in-roles.md#managed-identity-operator) sobre la identidad administrada.

### <a name="how-do-i-prevent-the-creation-of-user-assigned-managed-identities"></a>¿Cómo evito la creación de identidades administradas asignadas por el usuario?

Puede impedir que los usuarios creen identidades administradas asignadas por el usuario mediante [Azure Policy](../../governance/policy/overview.md)

1. Navegue hasta [Azure Portal](https://portal.azure.com) y vaya a **Directiva**.
2. Elija **Definiciones**.
3. Seleccione **+ Definición de directiva** y escriba la información necesaria.
4. En la sección de regla de la directiva, pegue
    
    ```json
    {
      "mode": "All",
      "policyRule": {
        "if": {
          "field": "type",
          "equals": "Microsoft.ManagedIdentity/userAssignedIdentities"
        },
        "then": {
          "effect": "deny"
        }
      },
      "parameters": {}
    }
    
    ```

Después de crear la directiva, asígnela al grupo de recursos que quiere usar.

1. Navegue a los grupos de recursos.
2. Busque el grupo de recursos que usa para las pruebas.
3. Elija **Directivas** en el menú izquierdo.
4. Seleccione **Asignar directiva**.
5. En la sección **Datos básicos**, proporcione:
    1. **Ámbito**: el grupo de recursos que se usa para las pruebas.
    1. **Definición de directiva**: La directiva que creamos anteriormente.
6. Deje todas las demás opciones con sus valores predeterminados y elija **Revisar y crear**.

En este punto, se producirá un error al intentar crear una identidad administrada asignada por el usuario en el grupo de recursos.

  ![Infracción de la directiva](./media/known-issues/policy-violation.png)

## <a name="concepts"></a>Conceptos

### <a name="do-managed-identities-have-a-backing-app-object"></a>¿Las identidades administradas tienen un objeto de aplicación de respaldo?

No. Las identidades administradas y los registros de aplicaciones de Azure AD no son lo mismo en el directorio.

Los registros de aplicaciones tienen dos componentes: Un objeto de aplicación y un objeto de entidad de servicio.
Las identidades administradas para los recursos de Azure solo tienen uno de estos componentes: un objeto de entidad de servicio.

Las identidades administradas no tienen un objeto de aplicación en el directorio, que es lo que se suele usar para conceder permisos de aplicación para MS Graph. En su lugar, los permisos de MS Graph para identidades administradas deben concederse directamente a la entidad de servicio.

### <a name="what-is-the-credential-associated-with-a-managed-identity-how-long-is-it-valid-and-how-often-is-it-rotated"></a>¿Cuál es la credencial asociada a una identidad administrada? ¿Qué validez tiene y con qué frecuencia se rota?

> [!NOTE]
> La forma de autenticarse las identidades administradas es un dato de implementación interno que está sujeto a cambios sin previo aviso.

Las identidades administradas usan la autenticación basada en certificados. Cada credencial de identidad administrada expira al cabo de 90 días y se revierte después de 45 días.

### <a name="what-identity-will-imds-default-to-if-dont-specify-the-identity-in-the-request"></a>¿A qué identidad se asignará el IMDS de manera predeterminada si no se especifica la identidad en la solicitud?

- Si se habilita la identidad administrada asignada por el sistema y no se especifica ninguna identidad en la solicitud, el IMDS utiliza de manera predeterminada esta identidad administrada.
- Si la identidad administrada asignada por el sistema no está habilitada, y solo existe una identidad administrada asignada por el usuario, IMDS será la identidad administrada asignada de manera predeterminada a ese único usuario.
- Si la identidad administrada asignada por el sistema no está habilitada y existen varias identidades administradas asignadas por el usuario, es necesario especificar una identidad administrada en la solicitud.

## <a name="limitations"></a>Limitaciones

### <a name="can-the-same-managed-identity-be-used-across-multiple-regions"></a>¿Se puede usar la misma identidad administrada en varias regiones?

En resumen, sí puede usar identidades administradas asignadas por el usuario en más de una región de Azure. La respuesta más larga es que, mientras que las identidades administradas asignadas por el usuario se crean como recursos regionales, la [entidad de servicio](../develop/app-objects-and-service-principals.md#service-principal-object) asociada (SP) creada en Azure AD está disponible globalmente. La entidad de servicio se puede usar desde cualquier región de Azure y su disponibilidad depende de la disponibilidad de Azure AD. Por ejemplo, si ha creado una identidad administrada asignada por el usuario en la región Centro y Sur, y esa región deja de estar disponible, este problema solo afecta a las actividades del [plano de control](../../azure-resource-manager/management/control-plane-and-data-plane.md) en la propia identidad administrada.  Las actividades realizadas por los recursos ya configuradas para usar las identidades administradas no se verán afectadas.

### <a name="does-managed-identities-for-azure-resources-work-with-azure-cloud-services"></a>¿Funcionan las identidades administradas para recursos de Azure con Azure Cloud Services?

No, no hay planes que admitan las identidades administradas para recursos de Azure en Azure Cloud Services.


### <a name="what-is-the-security-boundary-of-managed-identities-for-azure-resources"></a>¿Cuál es el límite de seguridad de las identidades administradas para recursos de Azure?

El límite de seguridad de la identidad es el recurso al que está asociada. Por ejemplo, el límite de seguridad para una máquina virtual que tenga las identidades administradas para recursos de Azure habilitadas, es la Máquina virtual. Cualquier código que se ejecute en esa máquina virtual puede llamar a las identidades administradas para los tokens de punto de conexión y solicitud de recursos de Azure. Esta experiencia es similar con otros recursos que admiten las identidades administradas para recursos de Azure.

### <a name="will-managed-identities-be-recreated-automatically-if-i-move-a-subscription-to-another-directory"></a>¿Se van a volver a crear automáticamente las identidades administradas si muevo una suscripción a otro directorio?

No. Si mueve una suscripción a otro directorio, tendrá que volver a crearlas manualmente y volver a asignar los roles de Azure.
- Para las identidades administradas asignadas por el sistema, tiene que desactivarlas y volver a activarlas. 
- Para las identidades administradas asignadas por el usuario, tiene que eliminarlas, volver a crearlas y adjuntarlas de nuevo a los recursos necesarios (por ejemplo, máquinas virtuales).

### <a name="can-i-use-a-managed-identity-to-access-a-resource-in-a-different-directorytenant"></a>¿Puedo usar una identidad administrada para acceder a un recurso en un directorio o inquilino diferente?

No. Las identidades administradas no admiten actualmente escenarios entre directorios. 

### <a name="are-there-any-rate-limits-that-apply-to-managed-identities"></a>¿Hay algún límite de frecuencia que se aplique a las identidades administradas?

Los límites de identidades administradas tienen dependencias de los límites de servicio de Azure, los de Azure Instance Metadata Service (IMDS) y los de Azure Active Directory.

- Los **límites de servicio de Azure** definen el número de operaciones de creación que se pueden realizar en los niveles de inquilino y suscripción. Las identidades administradas asignadas por el usuario también tienen [limitaciones](../../azure-resource-manager/management/azure-subscription-service-limits.md#managed-identity-limits) en cuanto a cómo se les puede asignar un nombre.
- **IMDS** En general, las solicitudes a IMDS se limitan a cinco por segundo. Las solicitudes que superen este umbral se rechazarán con respuestas 429. Las solicitudes a la categoría Identidad administrada se limitan a 20 por segundo y cinco solicitudes simultáneas. Puede leer más en el artículo [Azure Instance Metadata Service (Windows)](../../virtual-machines/windows/instance-metadata-service.md?tabs=windows#managed-identity).
- **Azure Active Directory** Cada identidad administrada cuenta para el límite de cuota de objetos de un inquilino de Azure AD, como se indica en [Restricciones y límites del servicio Azure AD](../enterprise-users/directory-service-limits-restrictions.md).


### <a name="is-it-possible-to-move-a-user-assigned-managed-identity-to-a-different-resource-groupsubscription"></a>¿Se puede trasladar una identidad administrada asignada por el usuario a otro grupo de recursos o suscripción?

No se admite el movimiento de una identidad administrada asignada por el usuario a otro grupo de recursos.

## <a name="next-steps"></a>Pasos siguientes

- Obtenga información sobre [cómo funcionan las identidades administradas con máquinas virtuales](how-managed-identities-work-vm.md).
