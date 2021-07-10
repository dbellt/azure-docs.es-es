---
title: Creación de un registro de aplicación (CLI)
titleSuffix: Azure Digital Twins
description: Vea cómo se crea un registro de aplicaciones de Azure AD, como opción de autenticación para aplicaciones cliente, mediante la CLI.
author: baanders
ms.author: baanders
ms.date: 5/13/2021
ms.topic: how-to
ms.service: digital-twins
ms.openlocfilehash: 80d34833397e8ca1cb5cb14acd869597decd2d48
ms.sourcegitcommit: 7f59e3b79a12395d37d569c250285a15df7a1077
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/02/2021
ms.locfileid: "110989549"
---
# <a name="create-an-app-registration-to-use-with-azure-digital-twins-cli"></a>Creación de un registro de aplicaciones para usar con Azure Digital Twins (CLI)

[!INCLUDE [digital-twins-create-app-registration-selector.md](../../includes/digital-twins-create-app-registration-selector.md)]

Al trabajar con una instancia de Azure Digital Twins, es habitual interactuar con esa instancia mediante aplicaciones cliente, como una aplicación cliente personalizada o la aplicación de ejemplo [Azure Digital Twins Explorer](quickstart-azure-digital-twins-explorer.md). Dichas aplicaciones deben autenticarse con Azure Digital Twins para interactuar con él, y algunos de los [mecanismos de autenticación](how-to-authenticate-client.md) que las aplicaciones pueden usar conllevan un **registro de aplicaciones** de [Azure Active Directory (Azure AD)](../active-directory/fundamentals/active-directory-whatis.md).

Esto no es necesario en todos los escenarios de autenticación. Sin embargo, si va a usar una estrategia de autenticación o un ejemplo de código que requiera un registro de aplicaciones, en este artículo se muestra cómo configurar uno mediante la [CLI de Azure](/cli/azure/what-is-azure-cli). También se explica cómo [recopilar valores importantes](#collect-important-values) necesarios para usar el registro de aplicaciones para realizar la autenticación.

## <a name="azure-ad-app-registrations"></a>Registros de aplicaciones de Azure AD

[Azure Active Directory (Azure AD)](../active-directory/fundamentals/active-directory-whatis.md) es un servicio de administración de identidades y acceso basado en la nube de Microsoft. La configuración de un **registro de aplicaciones** en Azure AD es una manera de conceder acceso a una aplicación cliente a Azure Digital Twins.

Este registro de la aplicación es donde se configuran los permisos de acceso a las [API de Azure Digital Twins](concepts-apis-sdks.md). Más tarde, las aplicaciones cliente se pueden autenticar en el registro de aplicaciones con los **valores de identificador de cliente y de inquilino** y, como resultado, que se le concedan permisos de acceso a las API.

>[!TIP]
> Es posible que prefiera configurar un nuevo registro de aplicaciones cada vez que lo necesite, *o*, hacer esto solo una vez, y establecer un solo registro de aplicaciones que se compartirá entre todos los escenarios que lo requieran.

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

## <a name="create-manifest"></a>Cree el manifiesto.

En primer lugar, cree un archivo que contenga cierta información de servicio que el registro de aplicaciones necesitará para acceder a Azure Digital Twins API. Más adelante, este archivo se pasará al crear el registro de la aplicación, para configurar los permisos Azure Digital Twins.

Cree un archivo .json en el equipo denominado **manifest.json**. Copie este texto en el archivo:

```json
[
    {
        "resourceAppId": "0b07f429-9f4b-4714-9392-cc5e8e80c8b0",
        "resourceAccess": [
            {
                "id": "4589bd03-58cb-4e6c-b17f-b580e39652f8",
                "type": "Scope"
            }
        ]
    }
]
```

El valor estático `0b07f429-9f4b-4714-9392-cc5e8e80c8b0` es el identificador de recurso del punto de conexión de servicio Azure Digital Twins, que el registro de la aplicación necesitará para acceder a las API de Azure Digital Twins.
 
Guarde el archivo terminado.

### <a name="upload-to-cloud-shell"></a>Carga en Cloud Shell

A continuación, cargue en Cloud Shell el archivo de manifiesto que acaba de crear, con el fin de que pueda acceder a él en los comandos de Cloud Shell al configurar el registro de la aplicación.

Para cargar el archivo, vaya a la ventana Cloud Shell en el explorador. Seleccione el icono "Cargar/Descargar archivos" y elija "Cargar".

:::image type="content" source="media/how-to-set-up-instance/cloud-shell/cloud-shell-upload.png" alt-text="Captura de pantalla de Azure Cloud Shell. El icono Cargar está resaltado.":::

Vaya al archivo **manifest.json** en la máquina y pulse "Open" (Abrir). Así se cargará el archivo en la raíz del almacenamiento de Cloud Shell.

## <a name="create-the-registration"></a>Creación del registro

En esta sección, ejecutará un comando de Cloud Shell para crear un registro de aplicación con la siguiente configuración:
* Nombre elegido
* Disponible solo para las cuentas del directorio predeterminado (inquilino único)
* Una dirección URL de respuesta web de `http://localhost`
* Permisos de lectura y escritura para las API Azure Digital Twins

Ejecute el siguiente comando para crear el registro:

```azurecli-interactive
az ad app create --display-name <app-registration-name> --available-to-other-tenants false --reply-urls http://localhost --native-app --required-resource-accesses "@manifest.json"
```

La salida del comando es información sobre el registro de aplicación que ha creado. 

## <a name="verify-success"></a>Comprobación de que la operación se ha completado correctamente

Para confirmar que se han concedido los permisos de Azure Digital Twins mediante la búsqueda de los siguientes campos en la salida del comando `az ad app create` y la confirmación de que sus valores coinciden con lo que se muestra en la captura de pantalla siguiente:

:::image type="content" source="media/how-to-create-app-registration/cli-required-resource-access.png" alt-text="Captura de pantalla de la salida de Cloud Shell del comando de creación del registro de la aplicación. Los elementos de &quot;requiredResourceAccess&quot; están resaltados: hay un valor &quot;resourceAppId&quot; de 0b07f429-9f4b-4714-9392-cc5e8e80c8b0, y un valor &quot;resourceAccess > id&quot; de 4589bd03-58cb-4e6c-b17f-b580e39652f8.":::

También puede comprobar que el registro de la aplicación se creó correctamente mediante Azure Portal. Para ver las instrucciones del portal, consulte la sección acerca de la [comprobación de que la operación se ha completado correctamente (portal)](how-to-create-app-registration-portal.md#verify-success).

## <a name="collect-important-values"></a>Recopilación de valores importantes

A continuación, recopile algunos valores importantes sobre el registro de la aplicación que necesitará para usar el registro para autenticar una aplicación cliente. Estos valores incluyen:
* **nombre del recurso**
* **identificador del cliente**
* **identificador del inquilino**
* **secreto del cliente**

Para trabajar con Azure Digital Twins, el **nombre del recurso** es `http://digitaltwins.azure.net`.

En las secciones siguientes se describe cómo buscar los restantes valores.

### <a name="collect-client-id-and-tenant-id"></a>Recopilación de los identificadores de cliente y de inquilino

Para usar el registro de la aplicación para la autenticación, es posible que tenga que especificar su **identificador de aplicación (cliente)** e **identificador de directorio (inquilino).** En esta sección, recopilará estos valores para poder guardarlos y usarlos siempre que se necesiten.

Puede encontrar ambos valores en la salida del comando `az ad app create`.

Identificador de aplicación (cliente):

:::image type="content" source="media/how-to-create-app-registration/cli-app-id.png" alt-text="Captura de pantalla de la salida de Cloud Shell del comando de creación del registro de la aplicación. El valor de appId está resaltado.":::

Identificador de directorio (inquilino):

:::image type="content" source="media/how-to-create-app-registration/cli-tenant-id.png" alt-text="Captura de pantalla de la salida de Cloud Shell del comando de creación del registro de la aplicación. Está resaltado el valor GUID de odata.metadata.":::

### <a name="collect-client-secret"></a>Recopilación del secreto de cliente

Para crear un **secreto de cliente** para el registro de la aplicación, necesitará el valor de identificador de cliente del registro de la aplicación de la sección anterior. Use el valor en el siguiente comando de la CLI para crear un secreto:

```azurecli-interactive
az ad app credential reset --id <client-ID> --append
```

También puede agregar más parámetros a este comando para especificar una descripción de las credenciales, una fecha de finalización y otros detalles. Para más información sobre el comando y sus parámetros adicionales, consulte la [documentación de az ad app credential resetn](/cli/azure/ad/app/credential?view=azure-cli-latest&preserve-view=true#az_ad_app_credential_reset).

La salida de este comando es información sobre el secreto de cliente que ha creado. Copie el valor de `password` para usarlo cuando necesite el secreto de cliente para la autenticación.

:::image type="content" source="media/how-to-create-app-registration/cli-client-secret.png" alt-text="Captura de pantalla de la salida de Cloud Shell del comando de creación del registro de la aplicación. El valor de password está resaltado.":::

>[!IMPORTANT]
>Asegúrese de copiar el valor ahora y almacenarlo en un lugar seguro, ya que no se puede volver a recuperar. Si no encuentra el valor posteriormente, tendrá que crear un secreto.

## <a name="other-possible-steps-for-your-organization"></a>Otros posibles pasos para la organización

Es posible que su organización requiera otras acciones por parte de los propietarios o administradores de las suscripciones para configurar correctamente un registro de aplicaciones. Los pasos necesarios pueden variar en función de la configuración específica de su organización.

Estas son algunas posibles actividades comunes que puede que el propietario o administrador de la suscripción tenga que realizar.
* Conceda el consentimiento del administrador para el registro de aplicaciones. Es posible que la organización tenga habilitado globalmente el **Consentimiento del administrador necesario** en Azure AD para todos los registros de aplicaciones dentro de la suscripción. En ese caso, es posible que el propietario o administrador tenga que conceder permisos delegados o de aplicación adicionales.
* Active el acceso de cliente público. Para ello, anexe `--set publicClient=true` a un comando de creación o actualización para el registro.
* Establezca direcciones URL de respuesta específicas para el acceso web y de escritorio mediante el parámetro `--reply-urls`. Para más información sobre el uso de este parámetro con comandos `az ad`, consulte la [documentación de az ad app](/cli/azure/ad/app?view=azure-cli-latest&preserve-view=true).
* Permita los flujos de autenticación de OAuth2 implícitos mediante el parámetro `--oauth2-allow-implicit-flow`. Para más información sobre el uso de este parámetro con comandos `az ad`, consulte la [documentación de az ad app](/cli/azure/ad/app?view=azure-cli-latest&preserve-view=true).

Para más información sobre el registro de aplicaciones y sus distintas opciones de configuración, consulte [Registro de una aplicación en la plataforma de identidad de Microsoft](/graph/auth-register-app-v2).

## <a name="next-steps"></a>Pasos siguientes

En este artículo, ha configurado un registro de aplicaciones de Azure AD que se puede usar para autenticar las aplicaciones cliente con las API de Azure Digital Twins.

A continuación, lea sobre los mecanismos de autenticación, tanto los que usan registros de aplicaciones como los que no:
* [Procedimiento: Escritura de código de autenticación de aplicación](how-to-authenticate-client.md)