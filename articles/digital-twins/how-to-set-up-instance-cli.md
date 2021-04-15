---
title: Configuración de una instancia y de la autenticación (CLI)
titleSuffix: Azure Digital Twins
description: Descubra cómo configurar una instancia del servicio Azure Digital Twins mediante la CLI.
author: baanders
ms.author: baanders
ms.date: 7/23/2020
ms.topic: how-to
ms.service: digital-twins
ms.openlocfilehash: 2969ee5339306e30044a6a80260f912d45d0c8a2
ms.sourcegitcommit: 5f482220a6d994c33c7920f4e4d67d2a450f7f08
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/08/2021
ms.locfileid: "107105071"
---
# <a name="set-up-an-azure-digital-twins-instance-and-authentication-cli"></a>Configuración de una instancia de Azure Digital Twins y autenticación (CLI)

[!INCLUDE [digital-twins-setup-selector.md](../../includes/digital-twins-setup-selector.md)]

En este artículo se describen los pasos para **configurar una nueva instancia de Azure Digital Twins**, incluidas la creación de la instancia y la configuración de la autenticación. Después de completar este artículo, tendrá una instancia de Azure Digital Twins lista para empezar a programar.

En esta versión de este artículo se realizan los pasos manualmente, uno por uno, mediante la CLI.
* Para seguir estos pasos manualmente con Azure Portal, consulte la versión del portal de este artículo: [*Procedimiento: Configuración de una instancia y autenticación (Azure Portal)*](how-to-set-up-instance-portal.md).
* Para ejecutar una configuración automatizada mediante un script de implementación de ejemplo, consulte la versión con scripts de este artículo: [*Procedimiento: Configuración de una instancia y autenticación (con scripts)*](how-to-set-up-instance-scripted.md).

[!INCLUDE [digital-twins-setup-steps.md](../../includes/digital-twins-setup-steps.md)]
[!INCLUDE [digital-twins-setup-permissions.md](../../includes/digital-twins-setup-permissions.md)]

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

## <a name="set-up-cloud-shell-session"></a>Configuración de una sesión de Cloud Shell
[!INCLUDE [Cloud Shell for Azure Digital Twins](../../includes/digital-twins-cloud-shell.md)]

## <a name="create-the-azure-digital-twins-instance"></a>Creación de una instancia de Azure Digital Twins

En esta sección, **creará una nueva instancia de Azure Digital Twins** mediante el comando de Cloud Shell. Tendrá que proporcionar:
* Un grupo de recursos en el que se implementará la instancia. Si todavía no tiene un grupo de recursos existente, puede crear uno ahora con este comando:
    ```azurecli-interactive
    az group create --location <region> --name <name-for-your-resource-group>
    ```
* Una región para la implementación. Para ver qué regiones admiten Azure Digital Twins, visite [*Productos de Azure disponibles por región*](https://azure.microsoft.com/global-infrastructure/services/?products=digital-twins).
* Un nombre para la instancia. Si su suscripción tiene otra instancia de Azure Digital Twins en la región que ya usa el nombre especificado, se le pedirá que elija un nombre diferente.

Use estos valores en el comando siguiente para crear la instancia:

```azurecli-interactive
az dt create --dt-name <name-for-your-Azure-Digital-Twins-instance> -g <your-resource-group> -l <region>
```

### <a name="verify-success-and-collect-important-values"></a>Comprobación de que la operación es correcta y recopilación de valores importantes

Si la instancia se creó correctamente, el resultado en Cloud Shell tiene un aspecto similar al siguiente, donde se muestra información sobre el recurso que ha creado:

:::image type="content" source="media/how-to-set-up-instance/cloud-shell/create-instance.png" alt-text="Ventana Comandos con la creación correcta del grupo de recursos y la instancia de Azure Digital Twins":::

Anote **hostName**, **name** y **resourceGroup** del resultado de la instancia de Azure Digital Twins. Estos son todos los valores importantes que puede necesitar a medida que sigue trabajando con la instancia de Azure Digital Twins para configurar la autenticación y los recursos de Azure relacionados. Si otros usuarios van a programar en la instancia, debe compartirlos con ellos.

> [!TIP]
> Puede ver estas propiedades, junto con todas las propiedades de la instancia, en cualquier momento si ejecuta `az dt show --dt-name <your-Azure-Digital-Twins-instance>`.

Ahora tiene lista una instancia de Azure Digital Twins. A continuación, debe proporcionar los permisos de usuario de Azure adecuados para administrarla.

## <a name="set-up-user-access-permissions"></a>Configuración de permisos de acceso de usuarios

[!INCLUDE [digital-twins-setup-role-assignment.md](../../includes/digital-twins-setup-role-assignment.md)]

Use el comando siguiente para asignar el rol (debe ser ejecutado por un usuario con [suficientes permisos](#prerequisites-permission-requirements) en la suscripción de Azure). El comando requiere que pase el *nombre principal de usuario* en la cuenta de Azure AD del usuario al que se debe asignar el rol. En la mayoría de los casos, coincidirá con el correo electrónico del usuario en la cuenta de Azure AD.

```azurecli-interactive
az dt role-assignment create --dt-name <your-Azure-Digital-Twins-instance> --assignee "<Azure-AD-user-principal-name-of-user-to-assign>" --role "Azure Digital Twins Data Owner"
```

El resultado de este comando es la información de salida acerca de la asignación de roles que se ha creado.

> [!NOTE]
> Si este comando devuelve un error que indica que la CLI **no puede encontrar el usuario ni la entidad de servicio en la base de datos de grafos**:
>
> Asigne el rol mediante el *id. de objeto* del usuario en su lugar. Esto puede ocurrir para los usuarios de [cuentas de Microsoft (MSA)](https://account.microsoft.com/account) personales. 
>
> Use la [página de Azure Portal de usuarios de Azure Active Directory](https://portal.azure.com/#blade/Microsoft_AAD_IAM/UsersManagementMenuBlade/AllUsers) para seleccionar la cuenta de usuario y abrir los detalles. Copie el *id. de objeto* del usuario:
>
> :::image type="content" source="media/includes/user-id.png" alt-text="Vista de la página de usuario en Azure Portal en la que se resalta el GUID en el campo &quot;Id. de objeto&quot;" lightbox="media/includes/user-id.png":::
>
> A continuación, repita el comando de lista de asignación de roles con el *id. de objeto* del usuario para el parámetro `assignee` anterior.

### <a name="verify-success"></a>Comprobación de que la operación se ha completado correctamente

[!INCLUDE [digital-twins-setup-verify-role-assignment.md](../../includes/digital-twins-setup-verify-role-assignment.md)]

Ahora tiene lista una instancia de Azure Digital Twins y los permisos asignados para administrarla.

## <a name="next-steps"></a>Pasos siguientes

Pruebe las llamadas individuales de la API de REST en su instancia mediante los comandos de la CLI de Azure Digital Twins: 
* [Referencia de az dt](/cli/azure/dt)
* [*Procedimiento: Uso de la CLI de Azure Digital Twins*](how-to-use-cli.md).

O bien consulte cómo conectar una aplicación cliente a la instancia mediante el código de autenticación:
* [*Procedimiento: Escritura de código de autenticación de aplicación*](how-to-authenticate-client.md)