---
title: Error de autenticación de Azure Digital Twins Explorer
description: Causas y resoluciones para el "Error de autenticación" en Azure Digital Twins Explorer.
ms.service: digital-twins
author: baanders
ms.author: baanders
ms.topic: troubleshooting
ms.date: 4/8/2021
ms.openlocfilehash: 7d04b98014abbf6bf732da292df3b7bb9172f281
ms.sourcegitcommit: a5dd9799fa93c175b4644c9fe1509e9f97506cc6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108203584"
---
# <a name="authentication-failed"></a>Error de autenticación

En este artículo se describen las causas y los pasos de resolución al recibir un "Error de autenticación" cuando se ejecuta el ejemplo de [Azure Digital Twins Explorer](/samples/azure-samples/digital-twins-explorer/digital-twins-explorer/) en el equipo local. 

## <a name="symptoms"></a>Síntomas

Al configurar y ejecutar la aplicación Azure Digital Twins Explorer, los intentos de autenticación con la aplicación reciben el siguiente mensaje de error:

:::image type="content" source="media/troubleshoot-error-azure-digital-twins-explorer-authentication/authentication-error.png" alt-text="Captura de pantalla de un mensaje de error en Azure Digital Twins Explorer con el texto siguiente: Error de autenticación. Si ejecuta la aplicación de manera local, asegúrese de que ha iniciado sesión en Azure en la máquina host. Por ejemplo, ejecute &quot;az login&quot; en un símbolo del sistema, inicie sesión en Visual Studio o VS Code o establezca variables de entorno. Si necesita más información, consulte el archivo Léame o busque DefaultAzureCredential en la documentación de Azure.Identity. Si ejecuta adt-explorer hospedado en la nube, asegúrese de que la función de Azure de hospedaje tenga configurada una identidad administrada asignada por el sistema. Consulte el archivo Léame para obtener más información.":::

## <a name="causes"></a>Causas

### <a name="cause-1"></a>Causa 1

La aplicación de Azure Digital Twins Explorer usa [DefaultAzureCredential](/dotnet/api/azure.identity.defaultazurecredential) (parte de la biblioteca `Azure.Identity`), que buscará credenciales en el entorno local.

Como indica el texto del error, este puede producirse si no se han proporcionado credenciales locales para `DefaultAzureCredential`.

Para más información sobre el uso de credenciales locales con Azure Digital Twins Explorer, consulte la sección [Configuración de credenciales locales de Azure](./quickstart-azure-digital-twins-explorer.md#set-up-local-azure-credentials) de la *guía de inicio rápido de Azure Digital Twins: exploración de un escenario de ejemplo*.

### <a name="cause-2"></a>Causa 2

Este error también puede producirse si la cuenta de Azure no tiene configurados los permisos de control de acceso basado en rol (Azure RBAC) de Azure necesarios en la instancia de Azure Digital Twins. Para acceder a los datos de la instancia, debe tener el rol **Lector de datos de Azure Digital Twins** o **Propietario de datos de Azure Digital Twins** en la instancia que está intentando leer o administrar, respectivamente. 

Para obtener más información sobre la seguridad y los roles de Azure Digital Twins, consulte [Conceptos: Soluciones de seguridad para Azure Digital Twins](concepts-security.md).

## <a name="solutions"></a>Soluciones

### <a name="solution-1"></a>Solución 1

En primer lugar, asegúrese de que ha proporcionado las credenciales necesarias a la aplicación.

#### <a name="provide-local-credentials"></a>Especificación de credenciales locales

`DefaultAzureCredential` se autentica en el servicio con la información de un inicio de sesión local de Azure. Para proporcionar sus credenciales de Azure, inicie sesión en su cuenta de Azure en una ventana local de la [CLI de Azure](/cli/azure/install-azure-cli), o bien en Visual Studio o Visual Studio Code.

Puede consultar los tipos de credenciales que `DefaultAzureCredential` acepta, así como el orden en el que se intentan usar, en la [documentación de Azure.Identity para DefaultAzureCredential](/dotnet/api/overview/azure/identity-readme#defaultazurecredential).

Si ya ha iniciado sesión localmente en la cuenta de Azure correcta y el problema no se resuelve, pruebe con la siguiente solución.

### <a name="solution-2"></a>Solución 2

Compruebe que el usuario de Azure tiene el rol **Lector de datos de Azure Digital Twins** en la instancia de Azure Digital Twins si solo está intentando leer sus datos, o el rol **Propietario de datos de Azure Digital Twins** si está intentando administrar los datos.

Tenga en cuenta que este rol es diferente del...
* nombre anterior de este rol durante la versión preliminar, *Propietario de Azure Digital Twins (versión preliminar)* (el rol es el mismo, pero el nombre ha cambiado)
* rol de *propietario* de toda la suscripción de Azure *Propietario de datos de Azure Digital Twins* es un rol dentro de Azure Digital Twins y tiene el ámbito de esa instancia individual de Azure Digital Twins.
* rol de *propietario* en Azure Digital Twins. Son dos roles de administración distintos de Azure Digital Twins; *Propietario de datos de Azure Digital Twins* es el rol que debe usar para la administración.

 Si no tiene este rol, establézcalo para solucionar el problema.

#### <a name="check-current-setup"></a>Comprobación de la configuración actual

[!INCLUDE [digital-twins-setup-verify-role-assignment.md](../../includes/digital-twins-setup-verify-role-assignment.md)]

#### <a name="fix-issues"></a>Corrección de problemas 

Si no tiene esta asignación de roles, alguien con el rol de propietario en la **suscripción a Azure** debe ejecutar el siguiente comando para proporcionar al usuario de Azure el rol adecuado en la **instancia de Azure Digital Twins**. 

Si es propietario de la suscripción, puede ejecutar este comando por su cuenta. Si no, póngase en contacto con un propietario para que ejecute este comando en su nombre. El nombre del rol es **Propietario de datos de Azure Digital Twins** para el acceso de edición o **Lector de datos de Azure Digital Twins** para el acceso de lectura.

```azurecli-interactive
az dt role-assignment create --dt-name <your-Azure-Digital-Twins-instance> --assignee "<your-Azure-AD-email>" --role "<role-name>"
```

Para más información sobre este requisito de rol y el proceso de asignación, consulte la [sección de Configuración de los permisos de acceso del usuario en el artículo](how-to-set-up-instance-CLI.md#set-up-user-access-permissions) de *Procedimiento de configuración de una instancia y de la autenticación (mediante la CLI o el portal)* .

## <a name="next-steps"></a>Pasos siguientes

Lea los pasos de configuración para crear y autenticar una nueva instancia de Azure Digital Twins:
* [Procedimiento: Configuración de una instancia y de la autenticación (CLI)](how-to-set-up-instance-cli.md)

Obtenga más información sobre la seguridad y los permisos de Azure Digital Twins:
* [Conceptos: Seguridad para las soluciones de Azure Digital Twins](concepts-security.md)
