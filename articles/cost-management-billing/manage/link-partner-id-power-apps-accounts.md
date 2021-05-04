---
title: Vinculación de un identificador de asociado a sus cuentas de Power Apps con sus credenciales de Azure
description: Este artículo ayuda a los asociados de Microsoft a utilizar sus credenciales de Azure para ayudar a los clientes a usar Microsoft Power Apps.
author: bandersmsft
ms.reviewer: akangaw
ms.service: cost-management-billing
ms.subservice: billing
ms.topic: conceptual
ms.date: 03/16/2021
ms.author: banders
ms.openlocfilehash: acb22cc4b2a461e476131a83972db3e782425a39
ms.sourcegitcommit: 6f1aa680588f5db41ed7fc78c934452d468ddb84
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/19/2021
ms.locfileid: "107727715"
---
# <a name="link-a-partner-id-to-your-power-apps-accounts"></a>Vinculación de un identificador de asociado a cuentas de Power Apps

Este artículo ayuda a los asociados de Microsoft que sean proveedores de servicios de Power Apps a asociar su servicio a los clientes de Microsoft Power Apps. Cuando usted (el asociado de Microsoft) administra, configura y da soporte técnico de los servicios de Power Apps a su cliente, tiene acceso al entorno del cliente. Puede usar sus credenciales de Azure y un vínculo de administración de asociados (PAL) para asociar su identificador de red de asociado con las credenciales de cuenta usadas para la entrega del servicio.

El PAL permite a Microsoft identificar y reconocer a los asociados que tienen clientes de Power Apps. El uso de los atributos de Microsoft en la organización de un asociado en función de los permisos de la cuenta (rol de Power Apps) y el ámbito (inquilino, grupo de recursos y recurso).

## <a name="get-access-from-your-customer"></a>Obtención de acceso del cliente

Antes de vincular su identificador de asociado, el cliente debe concederle acceso a sus recursos de Power Apps mediante una de las siguientes opciones:

- **Usuario invitado**: su cliente puede agregarle como usuario invitado y asignar roles de Power Apps. Para más información, consulte [Adición de usuarios invitados de otro directorio](../../active-directory/external-identities/what-is-b2b.md).
- **Cuenta de directorio**: el cliente puede crear una cuenta de usuario automáticamente en su propio directorio y asignarle cualquier rol de Power Apps.
- **Entidad de servicio**: el cliente puede agregar una aplicación o un script de su organización a su directorio y asignarle cualquier rol de Power Apps. La identidad de la aplicación o el script se conoce como entidad de servicio.
- **Administrador delegado**: el cliente puede delegar un grupo de recursos para que los usuarios puedan trabajar en él desde su inquilino. Para más información, consulte [Para asociados: el administrador delegado](/power-platform/admin/for-partners-delegated-administrator).

## <a name="link-customer-to-a-partner-id"></a>Vinculación de un cliente a un identificador de asociado

Cuando tenga acceso a los recursos del cliente, use Azure Portal, PowerShell o la CLI de Azure para vincular su identificador de Microsoft Partner Network (Id. de MPN) a su identificador de usuario o entidad de servicio. Vincule el identificador de asociado a cada inquilino del cliente.

### <a name="use-the-azure-portal-to-link-to-a-new-partner-id"></a>Uso de Azure Portal para vincular a un nuevo Id. de partner

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. Vaya al [vínculo a un identificador de partner](https://portal.azure.com/#blade/Microsoft_Azure_Billing/managementpartnerblade) en Azure Portal.
1. Escriba el identificador de [Microsoft Partner Network](https://partner.microsoft.com/) de su organización. Asegúrese de utilizar el **identificador de MPN asociado** que se muestra en su perfil de asociado.  
    :::image type="content" source="./media/link-partner-id-power-apps-accounts/link-partner-id.png" alt-text="Captura de pantalla que muestra la ventana Vincular a un id. de partner." lightbox="./media/link-partner-id-power-apps-accounts/link-partner-id.png" :::
1. Para vincular un identificador de asociado a otro cliente, cambie de directorio. En **Cambiar directorio**, seleccione el directorio apropiado.  
    :::image type="content" source="./media/link-partner-id-power-apps-accounts/switch-directory.png" alt-text="Captura de pantalla que muestra la ventana Directorio y suscripción, donde puede cambiar de directorio." lightbox="./media/link-partner-id-power-apps-accounts/switch-directory.png" :::

### <a name="use-powershell-to-link-to-a-new-partner-id"></a>Uso de PowerShell para vincular a un nuevo Id. de partner

Instale el módulo de Azure PowerShell [Az.ManagementPartner](https://www.powershellgallery.com/packages/Az.ManagementPartner/).

Inicie sesión en el inquilino del cliente con cualquier cuenta de usuario o la entidad de servicio. Para obtener más información, consulte [Inicio de sesión con PowerShell](/powershell/azure/authenticate-azureps).

```azurepowershell-interactive
Update-AzManagementPartner -PartnerId 12345
```

Vincule el nuevo Id. de partner. El Id. de partner es el identificador de [Microsoft Partner Network](https://partner.microsoft.com/) de su organización. Asegúrese de utilizar el **identificador de MPN asociado** que se muestra en su perfil de asociado.

```azurepowershell-interactive
new-AzManagementPartner -PartnerId 12345
```

Obtención del Id. de partner vinculado

```azurepowershell-interactive
get-AzManagementPartner
```

Actualización del Id. de partner vinculado

```azurepowershell-interactive
Update-AzManagementPartner -PartnerId 12345
```

Eliminación del Id. de partner vinculado

```azurepowershell-interactive
remove-AzManagementPartner -PartnerId 12345
```

### <a name="use-the-azure-cli-to-link-to-a-new-partner-id"></a>Uso de la CLI de Azure para vincular un nuevo Id. de partner

En primer lugar, instale la extensión de la CLI de Azure.

```azurecli-interactive
az extension add --name managementpartner
```

Inicie sesión en el inquilino del cliente con cualquier cuenta de usuario o la entidad de servicio. Para obtener más información, consulte [Inicio de sesión con la CLI de Azure](/cli/azure/authenticate-azure-cli).

```azurecli-interactive
az login --tenant TenantName
```

Vincule el nuevo Id. de partner. El Id. de partner es el identificador de [Microsoft Partner Network](https://partner.microsoft.com/) de su organización.

```azurecli-interactive
az managementpartner create --partner-id 12345
```

Obtención del Id. de partner vinculado

```azurecli-interactive
az managementpartner show
```

Actualización del Id. de partner vinculado

```azurecli-interactive
az managementpartner update --partner-id 12345
```

Eliminación del Id. de partner vinculado

```azurecli-interactive
az managementpartner delete --partner-id 12345
```

### <a name="next-steps"></a>Pasos siguientes

- Lea las [preguntas frecuentes sobre Azure Cost Management + Billing](../cost-management-billing-faq.yml), donde encontrará respuestas a las preguntas relativas a la vinculación de un identificador de asociado a cuentas de Power Apps.
- Únase al debate en la [comunidad de asociados de Microsoft ](https://aka.ms/PALdiscussion) para recibir actualizaciones o enviar comentarios.
- Lea las [preguntas más frecuentes sobre la especialización avanzada de desarrollo de aplicaciones con poco código](https://assetsprod.microsoft.com/mpn/faq-low-code-app-development-advanced-specialization.pdf) para la asociación de Power Apps basada en PAL para la especialización avanzada de desarrollo de aplicaciones con poco código.
