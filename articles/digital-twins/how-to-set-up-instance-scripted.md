---
title: Configuración de una instancia y autenticación (con scripts)
titleSuffix: Azure Digital Twins
description: Vea cómo configurar una instancia del servicio de Azure Digital Twins mediante la ejecución de un script de implementación automatizada.
author: baanders
ms.author: baanders
ms.date: 7/23/2020
ms.topic: how-to
ms.service: digital-twins
ms.custom: contperf-fy21q3
ms.openlocfilehash: f56d1c305f5d18d82ae79781a7b60fd6b429357b
ms.sourcegitcommit: 6323442dbe8effb3cbfc76ffdd6db417eab0cef7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/28/2021
ms.locfileid: "110613824"
---
# <a name="set-up-an-azure-digital-twins-instance-and-authentication-scripted"></a>Configuración de una instancia de Azure Digital Twins y autenticación (con scripts)

[!INCLUDE [digital-twins-setup-selector.md](../../includes/digital-twins-setup-selector.md)]

En este artículo se describen los pasos para **configurar una nueva instancia de Azure Digital Twins**, incluidas la creación de la instancia y la configuración de la autenticación. Después de completar este artículo, tendrá una instancia de Azure Digital Twins lista para empezar a programar.

En esta versión de este artículo se completan estos pasos mediante la ejecución de un [script de implementación automatizada de ejemplo](/samples/azure-samples/digital-twins-samples/digital-twins-samples/) que simplifica el proceso. 
* Para ver los pasos manuales de la CLI que el script se ejecuta en segundo plano, consulte la versión de la CLI de este artículo: [Procedimiento: Configuración de una instancia y autenticación (CLI)](how-to-set-up-instance-cli.md) .
* Para ver los pasos manuales según Azure Portal, consulte la versión del portal de este artículo: [Procedimiento: Configuración de una instancia y autenticación (portal)](how-to-set-up-instance-portal.md) .

[!INCLUDE [digital-twins-setup-steps.md](../../includes/digital-twins-setup-steps.md)]

## <a name="prerequisites-permission-requirements"></a>Requisitos previos: Requisitos de permisos
[!INCLUDE [digital-twins-setup-permissions.md](../../includes/digital-twins-setup-permissions.md)]

## <a name="prerequisites-download-the-script"></a>Requisitos previos: Descarga del script

El script de ejemplo se escribe en PowerShell. Forma parte de los [ejemplos completos de Azure Digital Twins](/samples/azure-samples/digital-twins-samples/digital-twins-samples/), que puede descargar en la máquina. Para ello, vaya a ese vínculo de ejemplo y seleccione el botón *Browse code* (Examinar código) situado debajo del título. Esto lleva al repositorio de GitHub de los ejemplos, que se pueden descargar como un archivo .zip si se selecciona el botón *Código* y *Descargar archivo ZIP*.

:::image type="content" source="media/includes/download-repo-zip.png" alt-text="Captura de pantalla del repositorio digital-twins-samples en GitHub, donde están resaltados los pasos para descargarlo como un archivo ZIP." lightbox="media/includes/download-repo-zip.png":::

Con esto se descarga una carpeta .zip en la máquina como **digital-twins-samples-master.zip**. Vaya a la carpeta en la máquina y descomprímala para extraer los archivos.

En la carpeta descomprimida, el script de implementación se encuentra en _digital-twins-samples-master > scripts > **deploy.ps1**_.

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

## <a name="run-the-deployment-script"></a>Ejecutar el script de implementación

En este artículo se usa un ejemplo de código de Azure Digital Twins para implementar una instancia de Azure Digital Twins y la autenticación necesaria de forma semiautomática. También se puede usar como punto de partida para escribir sus propias interacciones con scripts.

Estos son los pasos para ejecutar el script de implementación en Cloud Shell.
1. Vaya a una ventana de [Azure Cloud Shell](https://shell.azure.com/) en el explorador. Inicie sesión con este comando:
    ```azurecli-interactive
    az login
    ```
    Si la CLI puede abrir el explorador predeterminado, lo hará y cargará una página de inicio de sesión de Azure. En caso contrario, abra una página del explorador en *https://aka.ms/devicelogin* y escriba el código de autorización que se muestra en el terminal.
 
2. En la barra de iconos de Cloud Shell, asegúrese de que Cloud Shell está establecido para ejecutar la versión de PowerShell.

    :::image type="content" source="media/how-to-set-up-instance/cloud-shell/cloud-shell-powershell.png" alt-text="Captura de pantalla de la ventana de Cloud Shell en Azure Portal que muestra la selección de la versión de PowerShell.":::

1. Seleccione el icono "Cargar/Descargar archivos" y elija "Cargar".

    :::image type="content" source="media/how-to-set-up-instance/cloud-shell/cloud-shell-upload.png" alt-text="Captura de pantalla de la ventana de Cloud Shell en Azure Portal que muestra la selección del icono de carga.":::

    Vaya al archivo _**deploy.ps1**_ en la máquina (en _digital-twins-samples-master > scripts > **deploy.ps1**_) y seleccione "Abrir". Con esta acción se cargará el archivo en Cloud Shell para que pueda ejecutarlo en la ventana de Cloud Shell.

4. Ejecute el script mediante el envío del comando `./deploy.ps1` en la ventana de Cloud Shell. Puede copiar el comando siguiente (recuerde que, para pegar contenido en Cloud Shell, puede usar **Control + Mayús + V** en Windows y Linux, o **Cmd + Mayús + V** en macOS. También puede utilizar el menú contextual).

    ```azurecli-interactive
    ./deploy.ps1
    ```

    El script creará una instancia de Azure Digital Twins y asignará al usuario de Azure el rol *Propietario de datos de Azure Digital Twins* en la instancia.

    Cuando el script se ejecuta a través de los pasos de configuración automatizada, se le solicitará que pase los valores siguientes:
    * Por ejemplo, el *identificador de suscripción* de la suscripción de Azure.
    * En la instancia: una *ubicación* en la que quiera implementar la instancia. Para ver qué regiones admiten Azure Digital Twins, visite [Productos de Azure disponibles por región](https://azure.microsoft.com/global-infrastructure/services/?products=digital-twins).
    * Por ejemplo, el nombre de un *grupo de recursos*. Puede usar un grupo de recursos existente o escribir un nuevo nombre para crear uno.
    * Por ejemplo, un *nombre* para la instancia de Azure Digital Twins. Si su suscripción tiene otra instancia de Azure Digital Twins en la región que ya usa el nombre especificado, se le pedirá que elija un nombre diferente.

A continuación se muestra un extracto del registro de salida del script:

:::image type="content" source="media/how-to-set-up-instance/cloud-shell/deployment-script-output.png" alt-text="Captura de pantalla de la ventana de Cloud Shell en Azure Portal que muestra un registro de entrada y salida de la ejecución del script de implementación." lightbox="media/how-to-set-up-instance/cloud-shell/deployment-script-output.png":::

Si el script se completa correctamente, la copia impresa final indicará `Deployment completed successfully`. De lo contrario, solucione el mensaje de error y vuelva a ejecutar el script. Omitirá los pasos que ya ha completado y volverá a solicitar la entrada en el punto en el que se quedó.

> [!NOTE]
> Actualmente, el script asigna el rol de administración necesario en Azure Digital Twins (*Propietario de datos de Azure Digital Twins*) al mismo usuario que ejecuta el script desde Cloud Shell. Si tiene que asignar este rol a otra persona que vaya a administrar la instancia, puede hacerlo ahora por medio de Azure Portal ([instrucciones](how-to-set-up-instance-portal.md#set-up-user-access-permissions)) o la CLI ([instrucciones](how-to-set-up-instance-cli.md#set-up-user-access-permissions)).

>[!NOTE]
>Actualmente hay un **problema conocido** con la instalación con scripts, por el que algunos usuarios (especialmente los usuarios de las [cuentas de Microsoft personales o MSA](https://account.microsoft.com/account)) pueden encontrar que no se creó la **asignación de roles para el _Propietario de datos de Azure Digital Twins_**.
>
>Puede comprobar la asignación de roles mediante la sección [Comprobación de la asignación de roles de usuario](#verify-user-role-assignment) que aparece más adelante en este artículo y, si es necesario, configurar la asignación de roles manualmente mediante [Azure Portal](how-to-set-up-instance-portal.md#set-up-user-access-permissions) o la [CLI](how-to-set-up-instance-cli.md#set-up-user-access-permissions).
>
>Para más información, consulte [Solución de problemas: Problemas conocidos en Azure Digital Twins](troubleshoot-known-issues.md#missing-role-assignment-after-scripted-setup).

## <a name="verify-success-and-collect-important-values"></a>Comprobación de que la operación es correcta y recopilación de valores importantes

Si quiere verificar la creación de los recursos y los permisos configurados por el script, puede verlos en [Azure Portal](https://portal.azure.com) con las instrucciones siguientes. Si no puede comprobar el éxito de algún paso, vuelva a intentar ese paso. Los pasos se pueden realizar de forma individual con las instrucciones de [Azure Portal](how-to-set-up-instance-portal.md) o de la [CLI](how-to-set-up-instance-cli.md).

En esta sección, también se muestra cómo puede buscar valores importantes de los recursos configurados por el script que puede necesitar mientras sigue trabajando con la instancia de Azure Digital Twins. Debe guardarlos en un lugar seguro o volver a esta sección para encontrarlos más adelante cuando los necesite. Si otros usuarios van a programar contenido en la instancia, también debe compartir dichos valores con ellos.

### <a name="verify-instance"></a>Comprobación de la instancia

Para comprobar que se ha creado la instancia, vaya a la [página de Azure Digital Twins](https://ms.portal.azure.com/#blade/HubsExtension/BrowseResource/resourceType/Microsoft.DigitalTwins%2FdigitalTwinsInstances) en Azure Portal. Para acceder a esta página, escriba *Azure Digital Twins* en la barra de búsqueda del portal.

En esta página se muestran todas las instancias de Azure Digital Twins. Busque el nombre de la instancia recién creada en la lista.

Si la comprobación no se ha realizado correctamente, puede volver a intentar la creación de una instancia mediante el [portal](how-to-set-up-instance-portal.md#create-the-azure-digital-twins-instance) o la [CLI](how-to-set-up-instance-cli.md#create-the-azure-digital-twins-instance).

### <a name="collect-instance-values"></a>Recopilación de valores de la instancia

Seleccione el nombre de la instancia en la [página Azure Digital Twins](https://ms.portal.azure.com/#blade/HubsExtension/BrowseResource/resourceType/Microsoft.DigitalTwins%2FdigitalTwinsInstances) para abrir la página *Información general* de la instancia. Anote su *Nombre*, *Grupo de recursos* y *Nombre de host*. Puede necesitarlos más adelante para identificar la instancia y conectarse a ella.

:::image type="content" source="media/how-to-set-up-instance/portal/instance-important-values.png" alt-text="Captura de pantalla de Azure Portal en la que se resaltan los valores importantes de la página de información general de la instancia de Azure Digital Twins.":::

### <a name="verify-user-role-assignment"></a>Comprobación de la asignación de roles de usuario

[!INCLUDE [digital-twins-setup-verify-role-assignment.md](../../includes/digital-twins-setup-verify-role-assignment.md)]

> [!NOTE]
> Recuerde que actualmente el script asigna este rol necesario al mismo usuario que ejecuta el script desde Cloud Shell. Si tiene que asignar este rol a otra persona que vaya a administrar la instancia, puede hacerlo ahora a través de Azure Portal ([instrucciones](how-to-set-up-instance-portal.md#set-up-user-access-permissions)) o la CLI ([instrucciones](how-to-set-up-instance-cli.md#set-up-user-access-permissions)).

Si la comprobación no se ha realizado correctamente, también puede rehacer su propia asignación de roles mediante el [portal](how-to-set-up-instance-portal.md#set-up-user-access-permissions) o la [CLI](how-to-set-up-instance-cli.md#set-up-user-access-permissions).

## <a name="next-steps"></a>Pasos siguientes

Pruebe las llamadas individuales de la API de REST en su instancia mediante los comandos de la CLI de Azure Digital Twins: 
* [Referencia de az dt](/cli/azure/dt?view=azure-cli-latest&preserve-view=true)
* [Conceptos: Conjunto de comandos de la CLI de Azure Digital Twins](concepts-cli.md)

O bien consulte cómo conectar una aplicación cliente a la instancia mediante el código de autenticación:
* [Procedimiento: Escritura de código de autenticación de aplicación](how-to-authenticate-client.md)