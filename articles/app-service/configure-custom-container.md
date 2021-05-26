---
title: Configuración de un contenedor personalizado
description: Aprenda a configurar un contenedor personalizado en Azure App Service. En este artículo se muestran las tareas de configuración más comunes.
ms.topic: article
ms.date: 02/23/2021
ms.custom: devx-track-azurepowershell, devx-track-azurecli
zone_pivot_groups: app-service-containers-windows-linux
ms.openlocfilehash: 7648482fcf1d3618c02e4c8c8cf18aa521013baf
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110078281"
---
# <a name="configure-a-custom-container-for-azure-app-service"></a>Configuración de un contenedor de Linux personalizado para Azure App Service

En este artículo, se explica cómo configurar un contenedor personalizado para que se ejecute en Azure App Service.

::: zone pivot="container-windows"

Esta guía contiene conceptos clave e instrucciones para la contenedorización de aplicaciones de Windows en App Service. Si nunca ha usado Azure App Service, siga primero el [inicio rápido de contenedores personalizados](quickstart-custom-container.md) y el [tutorial](tutorial-custom-container.md).

::: zone-end

::: zone pivot="container-linux"

Esta guía incluye conceptos clave e instrucciones para la creación de contenedores de aplicaciones de Linux en App Service. Si nunca ha usado Azure App Service, siga primero el [inicio rápido de contenedores personalizados](quickstart-custom-container.md) y el [tutorial](tutorial-custom-container.md). También hay un [inicio rápido para aplicaciones de varios contenedores](quickstart-multi-container.md) y un [tutorial](tutorial-multi-container-app.md).

::: zone-end

::: zone pivot="container-windows"

## <a name="supported-parent-images"></a>Imágenes principales admitidas

Si desea utilizar una imagen de Windows personalizada, debe elegir la [imagen principal (imagen base)](https://docs.docker.com/develop/develop-images/baseimages/) adecuada para la plataforma que desee:

- Para implementar aplicaciones de .NET Framework, use una imagen principal basada en la versión del [Canal de mantenimiento a largo plazo (LTSC)](/windows-server/get-started-19/servicing-channels-19#long-term-servicing-channel-ltsc) de Windows Server Core. 
- Para implementar aplicaciones de .NET Core, use una imagen principal basada en la versión del [Canal de servicio semestral (SAC)](/windows-server/get-started-19/servicing-channels-19#semi-annual-channel) de Windows Server Nano. 

La descarga de una imagen primaria tarda un tiempo en completarse durante el inicio de la aplicación. Sin embargo, puede reducir el tiempo de inicio mediante una de las siguientes imágenes primarias que ya están almacenadas en caché en Azure App Service:

- [mcr.microsoft.com/windows/servercore](https://hub.docker.com/_/microsoft-windows-servercore):2004
- [mcr.microsoft.com/windows/servercore](https://hub.docker.com/_/microsoft-windows-servercore):ltsc2019
- [mcr.microsoft.com/dotnet/framework/aspnet](https://hub.docker.com/_/microsoft-dotnet-framework-aspnet/):4.8-windowsservercore-2004
- [mcr.microsoft.com/dotnet/framework/aspnet](https://hub.docker.com/_/microsoft-dotnet-framework-aspnet/):4.8-windowsservercore-ltsc2019
- [mcr.microsoft.com/dotnet/core/runtime](https://hub.docker.com/_/microsoft-dotnet-core-runtime/):3.1-nanoserver-2004
- [mcr.microsoft.com/dotnet/core/runtime](https://hub.docker.com/_/microsoft-dotnet-core-runtime/):3.1-nanoserver-1909
- [mcr.microsoft.com/dotnet/core/runtime](https://hub.docker.com/_/microsoft-dotnet-core-runtime/):3.1-nanoserver-1903
- [mcr.microsoft.com/dotnet/core/runtime](https://hub.docker.com/_/microsoft-dotnet-core-runtime/):3.1-nanoserver-1809
- [mcr.microsoft.com/dotnet/core/aspnet](https://hub.docker.com/_/microsoft-dotnet-core-aspnet/):3.1-nanoserver-2004
- [mcr.microsoft.com/dotnet/core/aspnet](https://hub.docker.com/_/microsoft-dotnet-core-aspnet/):3.1-nanoserver-1909
- [mcr.microsoft.com/dotnet/core/aspnet](https://hub.docker.com/_/microsoft-dotnet-core-aspnet/):3.1-nanoserver-1903
- [mcr.microsoft.com/dotnet/core/aspnet](https://hub.docker.com/_/microsoft-dotnet-core-aspnet/):3.1-nanoserver-1809

::: zone-end

## <a name="change-the-docker-image-of-a-custom-container"></a>Cambio de la imagen de Docker de un contenedor personalizado

Para sustituir la imagen actual de una aplicación de contenedor personalizada en Docker por otra imagen, utilice el siguiente comando:

```azurecli-interactive
az webapp config container set --name <app-name> --resource-group <group-name> --docker-custom-image-name <docker-hub-repo>/<image>
```

## <a name="use-an-image-from-a-private-registry"></a>Uso de una imagen de un registro privado

Para utilizar una imagen de un registro privado, como Azure Container Registry, ejecute el siguiente comando:

```azurecli-interactive
az webapp config container set --name <app-name> --resource-group <group-name> --docker-custom-image-name <image-name> --docker-registry-server-url <private-repo-url> --docker-registry-server-user <username> --docker-registry-server-password <password>
```

En *\<username>* y *\<password>* , especifique las credenciales de inicio de sesión de su cuenta de registro privada.

## <a name="i-dont-see-the-updated-container"></a>No veo el contenedor actualizado

Si cambia la configuración del contenedor de Docker para que apunte a un nuevo contenedor, la aplicación puede tardar unos minutos antes de que atienda las solicitudes HTTP desde el nuevo contenedor. Mientras se extrae y se inicia el nuevo contenedor, App Service sigue atendiendo solicitudes desde el contenedor antiguo. Solo cuando se inicia el nuevo contenedor y está preparado para recibir solicitudes, App Service comienza a enviarle solicitudes.

## <a name="how-container-images-are-stored"></a>Almacenamiento de las imágenes de contenedor

La primera vez que se ejecuta una imagen de Docker personalizada en App Service, esta aplicación ejecuta una operación `docker pull` y extrae todas las capas de la imagen. Estas capas se almacenan en el disco, como si Docker se estuviera utilizando en el entorno local. Cada vez que se reinicia la aplicación, App Service vuelve a ejecutar la operación `docker pull`, pero solo extrae las capas que han cambiado. Si no se ha producido ningún cambio, utiliza las capas existentes del disco local.

Si la aplicación cambia las instancias de proceso por algún motivo (por ejemplo, porque se amplíen o reduzcan verticalmente los planes de tarifa), App Service debe extraer de nuevo todas las capas. Lo mismo ocurre si se realiza un escalado horizontal para agregar instancias adicionales. También hay casos poco frecuentes en los que las instancias de la aplicación pueden cambiar sin que se deba a una operación de escalado.

## <a name="configure-port-number"></a>Configuración del número de puerto

De forma predeterminada, App Service da por hecho que el contenedor personalizado escucha en el puerto 80. Si el contenedor escucha otro puerto, establezca la opción `WEBSITES_PORT` de la aplicación en App Service. Puede hacerlo mediante [Cloud Shell](https://shell.azure.com). En Bash:

```azurecli-interactive
az webapp config appsettings set --resource-group <group-name> --name <app-name> --settings WEBSITES_PORT=8000
```

En PowerShell:

```azurepowershell-interactive
Set-AzWebApp -ResourceGroupName <group-name> -Name <app-name> -AppSettings @{"WEBSITES_PORT"="8000"}
```

Actualmente, App Service permite que el contenedor exponga solo un puerto para las solicitudes HTTP. 

## <a name="configure-environment-variables"></a>Configuración de las variables de entorno

El contenedor personalizado puede usar variables de entorno que se deben proporcionar de forma externa. Se pueden pasar mediante [Cloud Shell](https://shell.azure.com). En Bash:

```azurecli-interactive
az webapp config appsettings set --resource-group <group-name> --name <app-name> --settings DB_HOST="myownserver.mysql.database.azure.com"
```

En PowerShell:

```azurepowershell-interactive
Set-AzWebApp -ResourceGroupName <group-name> -Name <app-name> -AppSettings @{"DB_HOST"="myownserver.mysql.database.azure.com"}
```

Cuando se ejecuta la aplicación, la configuración de la aplicación App Service se inserta automáticamente en el proceso como variables de entorno. Puede comprobar las variables de entorno de contenedor con la dirección URL `https://<app-name>.scm.azurewebsites.net/Env)`.

Si la aplicación usa imágenes de un registro privado o de Docker Hub, las credenciales para acceder al repositorio se guardan en variables de entorno: `DOCKER_REGISTRY_SERVER_URL` `DOCKER_REGISTRY_SERVER_USERNAME` y `DOCKER_REGISTRY_SERVER_PASSWORD`. Debido a los riesgos de seguridad, ninguno de estos nombres de variable reservados se expone a la aplicación.

::: zone pivot="container-windows"
En el caso de los contenedores basados en IIS o .NET Framework (4.0 o superior), App Service la inserta automáticamente en `System.ConfigurationManager` como cadenas de conexión y opciones de la aplicación .NET. En todos los demás lenguajes o plataformas, se proporcionan como variables de entorno del proceso, con uno de los siguientes prefijos correspondientes:

- `APPSETTING_`
- `SQLCONTR_`
- `MYSQLCONTR_`
- `SQLAZURECOSTR_`
- `POSTGRESQLCONTR_`
- `CUSTOMCONNSTR_`

::: zone-end

::: zone pivot="container-linux"

Este método funciona tanto para aplicaciones de contenedor único o de varios contenedores, en los que las variables de entorno se especifican en el archivo *docker-compose.yml*.

::: zone-end

## <a name="use-persistent-shared-storage"></a>Uso de almacenamiento compartido persistente

::: zone pivot="container-windows"

Puede usar el directorio *C:\home* del sistema de archivos de la aplicación para conservar los archivos entre reinicios y compartirlos por diferentes instancias. El directorio `C:\home` de la aplicación se proporciona para permitir a la aplicación de contenedor el acceso al almacenamiento persistente.

Cuando se deshabilita el almacenamiento persistente, las operaciones de escritura del directorio `C:\home` no se conservan. [Los registros de contenedor y los registros del host de Docker](#access-diagnostic-logs) se guardan en un almacenamiento compartido persistente predeterminado que no está asociado al contenedor. Cuando se habilita el almacenamiento persistente, todas las operaciones de escritura que se realizan en el directorio `C:\home` se conservan y están accesibles para todas las instancias de una aplicación escalada horizontalmente. Los registros están disponibles en `C:\home\LogFiles`.

::: zone-end

::: zone pivot="container-linux"

Puede usar el directorio */home* en el sistema de archivos de la aplicación para conservar archivos entre reinicios y compartirlos entre instancias. El directorio `/home` de la aplicación se proporciona para permitir a la aplicación de contenedor el acceso al almacenamiento persistente.

Si se deshabilita el almacenamiento persistente, no se conservarán las escrituras en el directorio `/home` entre reinicios de aplicación ni entre varias instancias. La única excepción es el directorio `/home/LogFiles`, que se usa para almacenar los registros de Docker y del contenedor. Cuando se habilita el almacenamiento persistente, se conservan todas las escrituras en el directorio `/home` y todas las instancias de una aplicación escalada horizontalmente podrán acceder a ellas.

::: zone-end

De manera predeterminada, el almacenamiento persistente está deshabilitado y esta opción no aparece en la configuración de la aplicación. Para habilitarlo, defina la opción `WEBSITES_ENABLE_APP_SERVICE_STORAGE` de la aplicación mediante [Cloud Shell](https://shell.azure.com). En Bash:

```azurecli-interactive
az webapp config appsettings set --resource-group <group-name> --name <app-name> --settings WEBSITES_ENABLE_APP_SERVICE_STORAGE=true
```

En PowerShell:

```azurepowershell-interactive
Set-AzWebApp -ResourceGroupName <group-name> -Name <app-name> -AppSettings @{"WEBSITES_ENABLE_APP_SERVICE_STORAGE"=true}
```

> [!NOTE]
> También puede [configurar su propio almacenamiento persistente](configure-connect-to-azure-storage.md).

## <a name="detect-https-session"></a>Detección de sesión de HTTPS

App Service finaliza las solicitudes TLS/SSL en los servidores front-end. Esto significa que las solicitudes TLS/SSL nunca van a llegar a la aplicación. No es necesario ni debe implementarse ninguna compatibilidad con TLS/SSL en la aplicación. 

Los servidores front-end se encuentran en centros de datos de Azure. Si usa TLS/SSL con la aplicación, el tráfico de Internet siempre se cifrará de forma segura.

::: zone pivot="container-windows"

## <a name="customize-aspnet-machine-key-injection"></a>Personalización de la inserción de claves de máquina ASP.NET

 Durante el inicio del contenedor, las claves que se generan automáticamente se insertan en el contenedor como claves de máquina de las rutinas criptográficas de ASP.NET. Para [encontrar estas claves en el contenedor](#connect-to-the-container), busque las siguientes variables de entorno: `MACHINEKEY_Decryption`, `MACHINEKEY_DecryptionKey`, `MACHINEKEY_ValidationKey` y `MACHINEKEY_Validation`. 

En cada reinicio, las nuevas claves pueden restablecer la autenticación de formularios de ASP.NET y el estado de vista, si la aplicación depende de ellos. Para evitar la generación automática de claves, [establézcalas manualmente como la configuración de la aplicación de App Service](#configure-environment-variables). 

## <a name="connect-to-the-container"></a>Conexión al contenedor

Puede conectarse directamente al contenedor de Windows para realizar tareas de diagnóstico desde `https://<app-name>.scm.azurewebsites.net/DebugConsole`. Funcionamiento:

- La consola de depuración permite ejecutar comandos interactivos, como iniciar sesiones de PowerShell, inspeccionar claves del Registro y explorar todo el sistema de archivos del contenedor.
- Funciona de forma diferente al explorador gráfico anterior, en el que solo aparecen los archivos del [almacenamiento compartido](#use-persistent-shared-storage).
- En una aplicación escalada horizontalmente, la consola de depuración está conectada a una de las instancias de contenedor. Puede seleccionar una instancia diferente en la lista desplegable **Instancia** del menú principal.
- Cualquier cambio que realice en el contenedor desde la consola *no* se conservará cuando reinicie la aplicación (excepto los cambios efectuados en el almacenamiento compartido), ya que no forma parte de la imagen de Docker. Para conservar los cambios, como la configuración del Registro y la instalación del software, haga que formen parte de Dockerfile.

## <a name="access-diagnostic-logs"></a>Acceso a los registros de diagnóstico

App Service registra tanto las acciones realizadas por el host de Docker como las actividades realizadas desde dentro del contenedor. Los registros del host de Docker (registros de la plataforma) se envían de forma predeterminada, pero los registros de la aplicación o los registros del servidor web que están dentro del contenedor deben habilitarse manualmente. Para más información, consulte [Habilitación del registro de aplicaciones](troubleshoot-diagnostic-logs.md#enable-application-logging-linuxcontainer) y [Habilitar el registro de servidor web](troubleshoot-diagnostic-logs.md#enable-web-server-logging). 

Existen varias formas de acceder a los registros de Docker:

- [En Azure Portal](#in-azure-portal)
- [Desde la consola de Kudu](#from-the-kudu-console)
- [Con la API de Kudu](#with-the-kudu-api)
- [Envío de registros a Azure Monitor](troubleshoot-diagnostic-logs.md#send-logs-to-azure-monitor-preview)

### <a name="in-azure-portal"></a>En Azure Portal

Los registros de Docker aparecen en el portal, en la página **Configuración del contenedor** de la aplicación. Los registros están truncados, pero se pueden descargar haciendo clic en **Descargar**. 

### <a name="from-the-kudu-console"></a>Desde la consola de Kudu

Vaya a `https://<app-name>.scm.azurewebsites.net/DebugConsole` y haga clic en la carpeta **LogFiles** para ver archivos de registro específicos. Para descargar todo el directorio **LogFiles**, haga clic en el icono **Descargar** situado a la izquierda del nombre del directorio. También puede acceder a esta carpeta utilizando un cliente FTP.

Desde el terminal de la consola, no se puede acceder a la carpeta `C:\home\LogFiles` de forma predeterminada, ya que el almacenamiento compartido persistente no está habilitado. Para habilitar este comportamiento en el terminal de la consola, [habilite el almacenamiento compartido persistente](#use-persistent-shared-storage).

Si intenta descargar el registro de Docker que está actualmente en uso utilizando un cliente FTP, es posible que aparezca un error por un bloqueo de archivos.

### <a name="with-the-kudu-api"></a>Con la API de Kudu

Acceda directamente a `https://<app-name>.scm.azurewebsites.net/api/logs/docker` para ver los metadatos de los registros de Docker. Es posible que aparezcan varios archivos de registro. La propiedad `href` le permite descargar el archivo de registro directamente. 

Para descargar todos los registros juntos en un archivo ZIP, acceda a `https://<app-name>.scm.azurewebsites.net/api/logs/docker/zip`.

## <a name="customize-container-memory"></a>Personalización de la memoria del contenedor

De forma predeterminada, todos los contenedores de Windows implementados en Azure App Service tienen un límite de 1 GB de RAM. Puede cambiar este valor proporcionando la configuración de la aplicación `WEBSITE_MEMORY_LIMIT_MB` mediante [Cloud Shell](https://shell.azure.com). En Bash:

```azurecli-interactive
az webapp config appsettings set --resource-group <group-name> --name <app-name> --settings WEBSITE_MEMORY_LIMIT_MB=2000
```

En PowerShell:

```azurepowershell-interactive
Set-AzWebApp -ResourceGroupName <group-name> -Name <app-name> -AppSettings @{"WEBSITE_MEMORY_LIMIT_MB"=2000}
```

El valor se define en MB y debe ser menor y igual que la memoria física total del host. Por ejemplo, en un plan de App Service con 8 GB de RAM, el total acumulado de `WEBSITE_MEMORY_LIMIT_MB` para todas las aplicaciones no debe ser superior a 8 GB. Encontrará más información sobre la cantidad de memoria disponible en cada plan de tarifa en [Precios de App Service](https://azure.microsoft.com/pricing/details/app-service/windows/), en la sección del **plan de contenedor Premium (Windows)** .

## <a name="customize-the-number-of-compute-cores"></a>Personalización del número de núcleos de proceso

De forma predeterminada, un contenedor de Windows se ejecuta con todos los núcleos disponibles del plan de tarifa elegido. Es posible, por ejemplo, que desee reducir el número de núcleos que se usa en el espacio de ensayo. Para reducir el número de núcleos que usa un contenedor, establezca la opción `WEBSITE_CPU_CORES_LIMIT` de la aplicación en el número de núcleos que desee. Puede hacerlo mediante [Cloud Shell](https://shell.azure.com). En Bash:

```azurecli-interactive
az webapp config appsettings set --resource-group <group-name> --name <app-name> --slot staging --settings WEBSITE_CPU_CORES_LIMIT=1
```

En PowerShell:

```azurepowershell-interactive
Set-AzWebApp -ResourceGroupName <group-name> -Name <app-name> -AppSettings @{"WEBSITE_CPU_CORES_LIMIT"=1}
```

> [!NOTE]
> Al actualizar la configuración de la aplicación, se desencadena el reinicio automático, lo que hace que el tiempo de inactividad sea mínimo. En el caso de las aplicaciones de producción, considere la posibilidad de cambiar a un espacio de ensayo para modificarlas allí y volver después a producción.

Para comprobar el número ajustado, vaya a la consola de Kudu (`https://<app-name>.scm.azurewebsites.net`) y escriba los siguientes comandos con PowerShell. Cada comando da como resultado un número.

```PowerShell
Get-ComputerInfo | ft CsNumberOfLogicalProcessors # Total number of enabled logical processors. Disabled processors are excluded.
Get-ComputerInfo | ft CsNumberOfProcessors # Number of physical processors.
```

Los procesadores pueden ser multithreading o tener varios núcleos. Encontrará más información sobre el número de núcleos disponibles en cada plan de tarifa en [Precios de App Service](https://azure.microsoft.com/pricing/details/app-service/windows/), en la sección del **plan de contenedor Premium (Windows)** .

## <a name="customize-health-ping-behavior"></a>Personalización del comportamiento del ping de estado

App Service considera que un contenedor se ha iniciado correctamente cuando se inicia y responde a un ping HTTP. La solicitud de ping de estado contiene el encabezado `User-Agent= "App Service Hyper-V Container Availability Check"`. Si el contenedor se inicia pero no responde al ping tras un período de tiempo determinado, App Service agrega un evento al registro de Docker, lo que indica que el contenedor no se inició. 

Si la aplicación consume muchos recursos, es posible que el contenedor no responda a tiempo al ping HTTP. Para controlar las acciones cuando se produce un error en el ping HTTP, establezca la opción `CONTAINER_AVAILABILITY_CHECK_MODE` de la aplicación. Puede hacerlo mediante [Cloud Shell](https://shell.azure.com). En Bash:

```azurecli-interactive
az webapp config appsettings set --resource-group <group-name> --name <app-name> --settings CONTAINER_AVAILABILITY_CHECK_MODE="ReportOnly"
```

En PowerShell:

```azurepowershell-interactive
Set-AzWebApp -ResourceGroupName <group-name> -Name <app-name> -AppSettings @{"CONTAINER_AVAILABILITY_CHECK_MODE"="ReportOnly"}
```

En la tabla siguiente, se muestran los valores posibles:

| Value | Descripciones |
| - | - |
| **Repair** | Reinicia el contenedor después de tres comprobaciones de disponibilidad consecutivas. |
| **ReportOnly** | Valor predeterminado. No reinicia el contenedor, pero incluye información sobre el contenedor en los registros de Docker después de tres comprobaciones de disponibilidad consecutivas. |
| **Desactivado** | No comprueba la disponibilidad. |

## <a name="support-for-group-managed-service-accounts"></a>Compatibilidad con las cuentas de servicio administradas de grupo

Actualmente, no se admiten cuentas de servicio administradas por grupos (gMSA) en los contenedores de Windows de App Service.

::: zone-end

::: zone pivot="container-linux"

## <a name="enable-ssh"></a>Habilite SSH

SSH habilita la comunicación segura entre un contenedor y un cliente. Para que un contenedor personalizado admita SSH, debe agregarlo en la propia imagen de Docker.

> [!TIP]
> Todos los contenedores de Linux integrados en App Service han agregado las instrucciones de SSH en sus repositorios de imágenes. Puede seguir las instrucciones siguientes con el [repositorio de Node.js 10.14](https://github.com/Azure-App-Service/node/blob/master/10.14) para ver cómo se habilita allí. La configuración de la imagen integrada de Node.js es ligeramente diferente, pero la misma en principio.

- Agregue [un archivo sshd_config](https://man.openbsd.org/sshd_config) al repositorio, como en el ejemplo siguiente.

    ```
    Port            2222
    ListenAddress       0.0.0.0
    LoginGraceTime      180
    X11Forwarding       yes
    Ciphers aes128-cbc,3des-cbc,aes256-cbc,aes128-ctr,aes192-ctr,aes256-ctr
    MACs hmac-sha1,hmac-sha1-96
    StrictModes         yes
    SyslogFacility      DAEMON
    PasswordAuthentication  yes
    PermitEmptyPasswords    no
    PermitRootLogin     yes
    Subsystem sftp internal-sftp
    ```

    > [!NOTE]
    > Este archivo configura OpenSSH y debe incluir los siguientes elementos:
    > - `Port` se debe establecer en 2222.
    > - `Ciphers` debe incluir al menos un elemento de esta lista: `aes128-cbc,3des-cbc,aes256-cbc`.
    > - `MACs` debe incluir al menos un elemento de esta lista: `hmac-sha1,hmac-sha1-96`.

- En Dockerfile, agregue los siguientes comandos:

    ```Dockerfile
    # Install OpenSSH and set the password for root to "Docker!". In this example, "apk add" is the install instruction for an Alpine Linux-based image.
    RUN apk add openssh \
         && echo "root:Docker!" | chpasswd 

    # Copy the sshd_config file to the /etc/ssh/ directory
    COPY sshd_config /etc/ssh/

    # Open port 2222 for SSH access
    EXPOSE 80 2222
    ```

    Esta configuración no permite realizar conexiones externas al contenedor. Solo se puede acceder al puerto 2222 del contenedor dentro de la red de puente de una red virtual privada y no es accesible para un atacante en Internet.

- En el script de inicio del contenedor, inicie el servidor SSH.

    ```bash
    /usr/sbin/sshd
    ```

## <a name="access-diagnostic-logs"></a>Acceso a los registros de diagnóstico

[!INCLUDE [Access diagnostic logs](../../includes/app-service-web-logs-access-linux-no-h.md)]

## <a name="configure-multi-container-apps"></a>Configuración de aplicaciones de varios contenedores

- [Uso del almacenamiento persistente en Docker Compose](#use-persistent-storage-in-docker-compose)
- [Limitaciones de la versión preliminar](#preview-limitations)
- [Opciones de Docker Compose](#docker-compose-options)

### <a name="use-persistent-storage-in-docker-compose"></a>Uso del almacenamiento persistente en Docker Compose

Las aplicaciones de varios contenedores como WordPress necesitan almacenamiento persistente para funcionar correctamente. Para habilitarla, la configuración de Docker Compose debe apuntar a una ubicación de almacenamiento *fuera* del contenedor. Las ubicaciones de almacenamiento dentro del contenedor no conservan los cambios más allá del reinicio de la aplicación.

Para habilitar el almacenamiento persistente, establezca la opción `WEBSITES_ENABLE_APP_SERVICE_STORAGE` de la aplicación utilizando el comando [az webapp config appsettings set](/cli/azure/webapp/config/appsettings#az_webapp_config_appsettings_set) en [Cloud Shell](https://shell.azure.com).

```azurecli-interactive
az webapp config appsettings set --resource-group <group-name> --name <app-name> --settings WEBSITES_ENABLE_APP_SERVICE_STORAGE=TRUE
```

En el archivo *docker-compose.yml*, asigne la opción `volumes` a `${WEBAPP_STORAGE_HOME}`. 

`WEBAPP_STORAGE_HOME` es una variable de entorno en App Service que se asigna al almacenamiento persistente para la aplicación. Por ejemplo:

```yaml
wordpress:
  image: <image name:tag>
  volumes:
  - ${WEBAPP_STORAGE_HOME}/site/wwwroot:/var/www/html
  - ${WEBAPP_STORAGE_HOME}/phpmyadmin:/var/www/phpmyadmin
  - ${WEBAPP_STORAGE_HOME}/LogFiles:/var/log
```

### <a name="preview-limitations"></a>Limitaciones de vista previa

La aplicación de varios contenedores está actualmente en versión preliminar. Las siguientes características de la plataforma de App Service no se admiten:

- Autenticación/autorización
- Identidades administradas
- CORS
- No se admite la integración con la red virtual en escenarios de Docker Compose.

### <a name="docker-compose-options"></a>Opciones de Docker Compose

Las listas siguientes muestran opciones de configuración admitidas y no admitidas de Docker Compose:

#### <a name="supported-options"></a>Opciones admitidas

- command
- entrypoint
- Environment
- imagen
- ports
- restart
- services
- volumes

#### <a name="unsupported-options"></a>Opciones no admitidas

- build (no permitido)
- depends_on (omitido)
- networks (omitido)
- secrets (omitido)
- puertos que no sean el 80 y 8080 (omitido)

> [!NOTE]
> Cualquier otra opción que no se mencione de forma explícita, se omitirá en la versión preliminar pública.

[!INCLUDE [robots933456](../../includes/app-service-web-configure-robots933456.md)]

::: zone-end

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Tutorial: Migración de software personalizado a Azure App Service mediante un contenedor personalizado](tutorial-custom-container.md)

::: zone pivot="container-linux"

> [!div class="nextstepaction"]
> [Tutorial: Aplicación de WordPress con varios contenedores](tutorial-multi-container-app.md)

::: zone-end

O consulte estos recursos adicionales:

[Carga de certificados en contenedores de Linux y Windows](configure-ssl-certificate-in-code.md#load-certificate-in-linuxwindows-containers)
