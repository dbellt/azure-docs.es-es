---
title: Creación de autorizaciones aptas
description: Al incorporar clientes a Azure Lighthouse, puede permitir que los usuarios del inquilino de administración adopten un rol superior cuando sea necesario.
ms.date: 06/11/2021
ms.topic: how-to
ms.openlocfilehash: 938b0ae8f2d105d79237164287b00ec4fdf4d607
ms.sourcegitcommit: 23040f695dd0785409ab964613fabca1645cef90
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/14/2021
ms.locfileid: "112060763"
---
# <a name="create-eligible-authorizations"></a>Creación de autorizaciones aptas

Al incorporar clientes a Azure Lighthouse, puede crear autorizaciones para conceder roles integrados de Azure concretos a los usuarios del inquilino de administración. También puede crear autorizaciones aptas que usen [Azure Active Directory (Azure AD) Privileged Identity Management (PIM)](../../active-directory/privileged-identity-management/pim-configure.md) para permitir que los usuarios del inquilino de administración tengan un rol superior temporalmente. Esto le permite conceder permisos adicionales cuando sea necesario para que los usuarios solo tengan esos permisos durante un tiempo fijo.

La creación de autorizaciones aptas permite minimizar el número de asignaciones permanentes de usuarios a roles con privilegios, lo que ayuda a reducir los riesgos de seguridad relacionados con el acceso con privilegios por parte de los usuarios del inquilino.

En este tema se explica cómo funcionan las autorizaciones aptas y cómo crearlas al [incorporar un cliente a Azure Lighthouse](onboard-customer.md).

> [!IMPORTANT]
> Las autorizaciones aptas se encuentran actualmente en una versión preliminar pública.
> Esta versión preliminar se ofrece sin Acuerdo de Nivel de Servicio y no se recomienda para cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas.
> Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="license-requirements"></a>Requisitos de licencia

La creación de autorizaciones aptas requiere una licencia Enterprise Mobility + Security E5 (EMS E5) o Azure AD Premium P2. Para obtener la licencia correcta para sus requisitos, consulte [Comparación de las características con disponibilidad general de las ediciones Gratis, Básico y Premium](https://azure.microsoft.com/pricing/details/active-directory/).

La licencia EMS E5 o Azure AD Premium P2 debe estar en manos del inquilino de administración, no del inquilino del cliente.

Los costos adicionales asociados con un rol apto solo se aplicarán durante el período de tiempo en el que el usuario tenga acceso al rol superior.

Para más información sobre las licencias para usuarios, consulte [License requirements to use Privileged Identity Management](../../active-directory/privileged-identity-management/subscription-requirements.md) (Requisitos de licencia para usar Privileged Identity Management).

## <a name="how-eligible-authorizations-work"></a>Funcionamiento de las autorizaciones aptas

Una autorización apta define una asignación de roles que requiere que el usuario active el rol cuando necesite realizar tareas con privilegios. Cuando active el rol apto, tendrá el acceso completo que concede ese rol durante el período de tiempo especificado.

Los usuarios del inquilino del cliente pueden revisar todas las asignaciones de roles, incluidas las de las autorizaciones aptas, antes del proceso de incorporación.

Una vez que un usuario activa correctamente un rol apto, tendrá ese rol superior en el ámbito delegado durante un período de tiempo preconfigurado, además de sus asignaciones de rol permanentes para ese ámbito.

Los administradores del inquilino de administración pueden revisar todas las actividades de Privileged Identity Management mediante la visualización del registro de auditoría en el inquilino de administración. Los clientes pueden ver estas acciones en el registro de actividad de Azure de la suscripción delegada.

Al crear una autorización apta, se definen tres elementos: el usuario, el rol y la directiva de acceso.

- El **usuario** puede ser un usuario individual o un grupo de Azure AD en el inquilino de administración. Si se define un grupo, cualquier miembro de ese grupo podrá elevar su propio acceso individual al rol según la directiva de acceso. No se pueden usar autorizaciones aptas con entidades de servicio.
- El **rol** puede ser cualquier rol integrado de Azure que sea compatible con la administración de recursos delegados de Azure, excepto el de administrador de acceso de los usuarios.
- La **directiva de acceso** define los requisitos de autorización multifactor (MFA) y el período de tiempo que un usuario tendrá el rol activo antes de que expire. La cantidad máxima que se puede especificar para cualquier rol son 8 horas.

A continuación se incluye más información sobre estos elementos y cómo definirlos.

## <a name="create-eligible-authorizations-using-azure-resource-manager-templates"></a>Creación de autorizaciones aptas con plantillas de Azure Resource Manager

Para incorporar el cliente a Azure Lighthouse, use una [plantilla de Azure Resource Manager junto con un archivo de parámetros correspondiente](onboard-customer.md#create-an-azure-resource-manager-template) que modifique. La plantilla que elija dependerá de si se incorpora una suscripción completa, un grupo de recursos o varios grupos de recursos de una suscripción.

> [!NOTE]
> Aunque también puede incorporar clientes mediante ofertas de servicio administrado en Azure Marketplace, actualmente no se pueden incluir autorizaciones aptas en esas ofertas.

Para incluir autorizaciones aptas al incorporar un cliente, use una de las plantillas de la [sección delegated-resource-management-eligible-authorizations de nuestro repositorio de ejemplos](https://github.com/Azure/Azure-Lighthouse-samples/tree/master/templates/delegated-resource-management-eligible-authorizations).

|Para incorporar esto (con autorizaciones aptas)  |Use esta plantilla de Azure Resource Manager  |Y modifique este archivo de parámetros |
|---------|---------|---------|
|Suscripción   |[subscription.json](https://github.com/Azure/Azure-Lighthouse-samples/blob/master/templates/delegated-resource-management-eligible-authorizations/subscription/subscription.json)  |[subscription.parameters.json](https://github.com/Azure/Azure-Lighthouse-samples/blob/master/templates/delegated-resource-management-eligible-authorizations/subscription/subscription.Parameters.json)    |
|Resource group   |[rg.json](https://github.com/Azure/Azure-Lighthouse-samples/blob/master/templates/delegated-resource-management-eligible-authorizations/rg/rg.json)  |[rg.parameters.json](https://github.com/Azure/Azure-Lighthouse-samples/blob/master/templates/delegated-resource-management-eligible-authorizations/rg/rg.parameters.json)    |
|Varios grupos de recursos de una suscripción   |[multiple-rg.json](https://github.com/Azure/Azure-Lighthouse-samples/blob/master/templates/delegated-resource-management-eligible-authorizations/rg/multiple-rg.json)  |[multiple-rg.parameters.json](https://github.com/Azure/Azure-Lighthouse-samples/blob/master/templates/delegated-resource-management-eligible-authorizations/rg/multiple-rg.parameters.json)    |

La plantilla **subscription.json**, que se puede usar para incorporar una suscripción con autorizaciones aptas, se muestra a continuación. 

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-08-01/subscriptionDeploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "mspOfferName": {
            "type": "string",
            "metadata": {
                "description": "Specify a unique name for your offer"
            },
            "defaultValue": "<to be filled out by MSP> Specify a title for your offer"
        },
        "mspOfferDescription": {
            "type": "string",
            "metadata": {
                "description": "Name of the Managed Service Provider offering"
            },
            "defaultValue": "<to be filled out by MSP> Provide a brief description of your offer"
        },
        "managedByTenantId": {
            "type": "string",
            "metadata": {
                "description": "Specify the tenant id of the Managed Service Provider"
            },
            "defaultValue": "<to be filled out by MSP> Provide your tenant id"
        },
        "authorizations": {
            "type": "array",
            "metadata": {
                "description": "Specify an array of objects, containing tuples of Azure Active Directory principalId, a Azure roleDefinitionId, and an optional principalIdDisplayName. The roleDefinition specified is granted to the principalId in the provider's Active Directory and the principalIdDisplayName is visible to customers."
            },
            "defaultValue": [
                { 
                    "principalId": "00000000-0000-0000-0000-000000000000", 
                    "roleDefinitionId": "acdd72a7-3385-48ef-bd42-f606fba81ae7",
                    "principalIdDisplayName": "PIM_Group" 
                }, 
                { 
                    "principalId": "00000000-0000-0000-0000-000000000000", 
                    "roleDefinitionId": "91c1777a-f3dc-4fae-b103-61d183457e46",
                    "principalIdDisplayName": "PIM_Group" 
                }   
            ]
        }, 
        "eligibleAuthorizations": { 
            "type": "array", 
            "metadata": { 
                "description": "Provide the authorizations that will have just-in-time role assignments on customer environments" 
            },
           "defaultValue": [ 
                { 
                        "justInTimeAccessPolicy": { 
                            "multiFactorAuthProvider": "Azure", 
                            "maximumActivationDuration": "PT8H" 
                        },
                        "principalId": "00000000-0000-0000-0000-000000000000", 
                        "principalIdDisplayName": "PIM_Group",
                        "roleDefinitionId": "36243c78-bf99-498c-9df9-86d9f8d28608" 
                        
                }                    
            ]    

        }                 
    },
    "variables": {
        "mspRegistrationName": "[guid(parameters('mspOfferName'))]",
        "mspAssignmentName": "[guid(parameters('mspOfferName'))]"
    },
    "resources": [
        {
            "type": "Microsoft.ManagedServices/registrationDefinitions",
            "apiVersion": "2020-02-01-preview",
            "name": "[variables('mspRegistrationName')]",
            "properties": {
                "registrationDefinitionName": "[parameters('mspOfferName')]",
                "description": "[parameters('mspOfferDescription')]",
                "managedByTenantId": "[parameters('managedByTenantId')]",
                "authorizations": "[parameters('authorizations')]", 
                "eligibleAuthorizations": "[parameters('eligibleAuthorizations')]" 
            }
        },
        {
            "type": "Microsoft.ManagedServices/registrationAssignments",
            "apiVersion": "2020-02-01-preview",
            "name": "[variables('mspAssignmentName')]",
            "dependsOn": [
                "[resourceId('Microsoft.ManagedServices/registrationDefinitions/', variables('mspRegistrationName'))]"
            ],
            "properties": {
                "registrationDefinitionId": "[resourceId('Microsoft.ManagedServices/registrationDefinitions/', variables('mspRegistrationName'))]"
            }
        }
    ],
    
    "outputs": {
        "mspOfferName": {
            "type": "string",
            "value": "[concat('Managed by', ' ', parameters('mspOfferName'))]"
        },
        "authorizations": {
            "type": "array",
            "value": "[parameters('authorizations')]"
        }, 
        "eligibleAuthorizations": { 
            "type": "array", 
            "value": "[parameters('eligibleAuthorizations')]" 

        } 
    }
}
```

### <a name="define-eligible-authorizations-in-your-parameters-file"></a>Definición de autorizaciones aptas en el archivo de parámetros

La [plantilla de ejemplo subscription.Parameters.json](https://github.com/Azure/Azure-Lighthouse-samples/blob/master/templates/delegated-resource-management-eligible-authorizations/subscription/subscription.Parameters.json) se puede usar para definir autorizaciones, incluidas las autorizaciones aptas, al incorporar una suscripción.

Cada una de las autorizaciones aptas debe definirse en el parámetro `eligibleAuthorizations`. En este ejemplo se incluye una autorización apta.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2018-05-01/subscriptionDeploymentParameters.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "mspOfferName": {
            "value": "Relecloud Managed Services"
        },
        "mspOfferDescription": {
            "value": "Relecloud Managed Services"
        },
        "managedByTenantId": {
            "value": "<insert the managing tenant id>"
        },
        "authorizations": {
            "value": [
                { 
                    "principalId": "00000000-0000-0000-0000-000000000000",
                    "roleDefinitionId": "acdd72a7-3385-48ef-bd42-f606fba81ae7",
                    "principalIdDisplayName": "PIM group"
                }
            ]
        }, 
        "eligibleAuthorizations":{
            "value": [
                {
                        "justInTimeAccessPolicy": {
                            "multiFactorAuthProvider": "Azure",
                            "maximumActivationDuration": "PT8H"
                        },
                        "principalId": "00000000-0000-0000-0000-000000000000", 
                        "principalIdDisplayName": "Tier 2 Support",
                        "roleDefinitionId": "b24988ac-6180-42a0-ab88-20f7382dd24c"

                }
            ]
        }
    }
}
```

Dentro del parámetro `eligibleAuthorizations`, `principalId` especifica el identificador del usuario de Azure AD o del grupo al que se aplicará esta autorización apta. No use un identificador de una cuenta de entidad de servicio, ya que actualmente no hay ninguna manera de que una cuenta de este tipo use un rol apto de nivel superior.

> [!IMPORTANT]
> Asegúrese de incluir el mismo valor de `principalId` en la sección `authorizations` de la plantilla con un rol diferente al de la autorización apta, como el de lectura (u otro rol integrado de Azure que incluya el acceso de lectura). Si no lo hace, el usuario no podrá utilizar un rol superior en Azure Portal.

`roleDefinitionId` contiene el identificador de definición de rol para un [rol integrado de Azure](../../role-based-access-control/built-in-roles.md) que el usuario podrá usar cuando sea necesario.

`justInTimeAccessPolicy` especifica dos elementos:

- `multiFactorAuthProvider` se puede establecer en **Azure**, que requerirá la autorización multifactor (MFA) de Azure, o en **Ninguno** si no se requiere esta autenticación.
- `maximumActivationDuration` establece el período de tiempo total durante el que el usuario tendrá el rol apto. Este valor debe usar el formato de duración ISO 8601. El valor mínimo es PT30M (30 minutos) y el máximo, PT8H (8 horas).

> [!NOTE]
> Nota: El acceso Just-In-Time no se aplica a los elementos `delegatedRoleDefinitionIds` que un administrador de acceso de usuario pueda [asignar a identidades administradas.](deploy-policy-remediation.md) Estas asignaciones de roles no se pueden crear como autorizaciones aptas. Del mismo modo, no puede crear una autorización apta para el rol de administrador de acceso de usuario.

## <a name="elevation-process-for-users"></a>Proceso de elevación para usuarios

Después de incorporar un cliente a Azure Lighthouse, todos los roles aptos que incluya estarán disponibles para el usuario especificado (o para los usuarios de cualquier grupo especificado).

Cada usuario puede elevar su acceso en cualquier momento. Para hacerlo, debe visitar la página **Mis clientes** de Azure Portal, seleccionar una delegación y, a continuación, seleccionar **Administrar roles elegibles**. Después, pueden seguir los [pasos para activar el rol](../../active-directory/privileged-identity-management/pim-how-to-activate-role.md) en Azure AD Privileged Identity Management.

:::image type="content" source="../media/manage-eligible-roles.png" alt-text="Captura de pantalla que muestra el botón Administrar roles elegibles en Azure Portal.":::

Una vez activado el rol apto, el usuario tendrá ese rol durante todo el tiempo especificado en la autorización apta. Después de ese período de tiempo, ya no podrá usar ese rol, a menos que repita el proceso de elevación para volver a utilizarlo.

## <a name="next-steps"></a>Pasos siguientes

- Obtenga información sobre cómo [incorporar clientes a Azure Lighthouse mediante plantillas de ARM](onboard-customer.md).
- Obtenga más información acerca de [Azure AD Privileged Identity Management](../../active-directory/privileged-identity-management/pim-configure.md).
- Obtenga más información acerca de [inquilinos, usuarios y roles en Azure Lighthouse](../concepts/tenants-users-roles.md).