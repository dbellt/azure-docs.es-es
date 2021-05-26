---
title: 'Problemas conocidos: Azure Digital Twins'
description: Obtenga ayuda para reconocer y mitigar los problemas conocidos de Azure Digital Twins.
author: baanders
ms.author: baanders
ms.topic: troubleshooting
ms.service: digital-twins
ms.date: 07/14/2020
ms.custom: contperf-fy21q2
ms.openlocfilehash: b53afb530cc49ad72ebfcbe7659fab787cc51995
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110069947"
---
# <a name="known-issues-in-azure-digital-twins"></a>Problemas conocidos en Azure Digital Twins

En este artículo se proporciona información sobre los problemas conocidos asociados a Azure Digital Twins.

## <a name="400-client-error-bad-request-in-cloud-shell"></a>"400 Error de cliente: Solicitud incorrecta" en Cloud Shell

**Descripción del problema:** Los comandos de Cloud Shell que se ejecutan en *https://shell.azure.com* pueden producir de forma intermitente el error "400 Error de cliente: Solicitud incorrecta de dirección URL: http://localhost:50342/oauth2/token", con seguimiento de la pila completa.

| ¿Cómo me afecta esto? | Causa | Solución |
| --- | --- | --- |
| In&nbsp;Azure&nbsp;Digital&nbsp;Twins, esto afecta a los siguientes grupos de comandos:<br><br>`az dt route`<br><br>`az dt model`<br><br>`az dt twin` | Este es el resultado de un problema conocido en Cloud Shell: [Error intermitente al obtener el token de Cloud Shell con 400 Error de cliente: Solicitud incorrecta](https://github.com/Azure/azure-cli/issues/11749).<br><br>Esto presenta un problema con los tokens de autenticación de la instancia de Azure Digital Twins y la autenticación basada en la [identidad administrada](../active-directory/managed-identities-azure-resources/overview.md) predeterminada del Cloud Shell. <br><br>Esto no afecta a los comandos de Azure Digital Twins de los grupos de comandos `az dt` o `az dt endpoint`, ya que usan otro tipo de token de autenticación (basado en Azure Resource Manager), que no tiene ningún problema con la autenticación de identidad administrada de Cloud Shell. | Una forma de resolver este problema consiste en volver a ejecutar el comando `az login` en Cloud Shell y completar los pasos de inicio de sesión siguientes. Esto cambiará la sesión de la autenticación de identidad administrada, lo que evita el problema raíz. Después de esto, debería poder ejecutar el comando nuevamente.<br><br>Como alternativa, puede abrir el panel de Cloud Shell en Azure Portal y completar desde allí el trabajo de Cloud Shell.<br>:::image type="content" source="media/troubleshoot-known-issues/portal-launch-icon.png" alt-text="Imagen del icono de Cloud Shell en la barra de iconos de Azure Portal" lightbox="media/troubleshoot-known-issues/portal-launch-icon.png":::<br><br>Otra alternativa consiste en [instalar la CLI de Azure](/cli/azure/install-azure-cli) en su máquina para poder ejecutar localmente comandos de la CLI de Azure. La CLI local no experimenta este problema. |


## <a name="missing-role-assignment-after-scripted-setup"></a>Asignación de roles que faltan después de la instalación con script

**Descripción del problema:** Algunos usuarios pueden experimentar problemas con la parte de asignación de roles que se describe en el tutorial [Procedimiento: Configuración de una instancia y autenticación (con scripts)](how-to-set-up-instance-scripted.md) . El script no indica un error, pero el rol *Propietario de datos de Azure Digital Twins* no se ha asignado correctamente al usuario, lo que afectará a la posibilidad de crear otros recursos más adelante.

| ¿Cómo me afecta esto? | Causa | Solución |
| --- | --- | --- |
| Para determinar si la asignación de roles se configuró correctamente después de ejecutar el script, siga las instrucciones de la sección [Comprobación de la asignación de roles de usuario](how-to-set-up-instance-scripted.md#verify-user-role-assignment) del artículo de configuración. Si el usuario no se muestra con este rol, este problema le afecta. | En el caso de los usuarios que iniciaron sesión con una [cuenta Microsoft (MSA)](https://account.microsoft.com/account) personal, el identificador de entidad de seguridad del usuario que lo identifica en comandos de este tipo puede ser diferente del correo electrónico de inicio de sesión del usuario, lo que dificulta la detección y el uso de la asignación del rol correctamente por el script. | Para resolverlo, puede configurar la asignación de roles manualmente mediante las [instrucciones de la CLI](how-to-set-up-instance-cli.md#set-up-user-access-permissions) o las [instrucciones de Azure Portal](how-to-set-up-instance-portal.md#set-up-user-access-permissions). |

## <a name="issue-with-interactive-browser-authentication-on-azureidentity-120"></a>Problema con la autenticación interactiva del explorador en Azure.Identity 1.2.0

**Descripción del problema:** Al escribir código de autenticación en las aplicaciones de Azure Digital Twins con la versión **1.2.0** de la biblioteca [Azure.Identity](/dotnet/api/azure.identity?view=azure-dotnet&preserve-view=true), puede experimentar problemas con el método [InteractiveBrowserCredential](/dotnet/api/azure.identity.interactivebrowsercredential?view=azure-dotnet&preserve-view=true). Esto presenta una respuesta de error de "Azure.Identity.AuthenticationFailedException" al intentar autenticarse en la ventana del explorador. Es posible que la ventana del explorador no se inicie por completo o que parezca que la autenticación del usuario se realiza correctamente, mientras que la aplicación cliente sigue generando el error.

| ¿Cómo me afecta esto? | Causa | Solución |
| --- | --- | --- |
| El&nbsp;método&nbsp;afectado&nbsp;se&nbsp;usa&nbsp;en&nbsp;los&nbsp;artículos siguientes:<br><br>[Tutorial: Programación de una aplicación cliente](tutorial-code.md)<br><br>[Procedimiento: Escritura de código de autenticación de aplicación](how-to-authenticate-client.md)<br><br>[Conceptos: API y SDK de Azure Digital Twins](concepts-apis-sdks.md) | Algunos usuarios han tenido este problema con la versión **1.2.0** de la biblioteca de `Azure.Identity`. | Para resolverlo, actualice las aplicaciones para que usen una [versión posterior](https://www.nuget.org/packages/Azure.Identity) de `Azure.Identity`. Después de actualizar la versión de la biblioteca, el explorador se cargará y autenticará según lo previsto. |

## <a name="issue-with-default-azure-credential-authentication-on-azureidentity-130"></a>Problema con la autenticación de credenciales de Azure predeterminada en Azure.Identity 1.3.0

**Descripción del problema:** Al escribir código de autenticación con la versión **1.3.0** de la biblioteca [Azure.Identity](/dotnet/api/azure.identity?view=azure-dotnet&preserve-view=true), algunos usuarios tienen problemas con el método [DefaultAzureCredential](/dotnet/api/azure.identity.defaultazurecredential?view=azure-dotnet&preserve-view=true) usado en muchos ejemplos en estos documentos de Azure Digital Twins. Esto presenta una respuesta de error "Azure.Identity.AuthenticationFailedException: SharedTokenCacheCredential authentication failed" cuando el código intenta autenticarse.

| ¿Cómo me afecta esto? | Causa | Solución |
| --- | --- | --- |
| `DefaultAzureCredential` se utiliza en la mayoría de los ejemplos de la documentación de este servicio que incluyen autenticación. Si va a escribir código de autenticación mediante `DefaultAzureCredential` con la versión 1.3.0 de la biblioteca `Azure.Identity` y ve este mensaje de error, esto le afectará. | Es probable que se deba a algún problema de configuración con `Azure.Identity`. | Una estrategia para resolver este problema es excluir `SharedTokenCacheCredential` de las credenciales, como se describe en este [Problema de DefaultAzureCredential](https://github.com/Azure/azure-sdk/issues/1970) que está actualmente abierto en `Azure.Identity`.<br>Otra opción es cambiar la aplicación para que use una versión anterior de `Azure.Identity`, como la [versión 1.2.3](https://www.nuget.org/packages/Azure.Identity/1.2.3). Esto no tiene ningún impacto funcional en Azure Digital Twins y, por lo tanto, también es una solución aceptada. |

## <a name="next-steps"></a>Pasos siguientes

Obtenga más información sobre la seguridad y los permisos de Azure Digital Twins:
* [Conceptos: Seguridad para las soluciones de Azure Digital Twins](concepts-security.md)