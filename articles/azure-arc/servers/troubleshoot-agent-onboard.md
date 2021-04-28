---
title: Solución de problemas de conexión del agente de los servidores habilitados para Azure Arc
description: En este artículo se explica cómo solucionar problemas relacionados con el agente de Connected Machine que surgen con los servidores habilitados para Azure Arc al intentar conectarse al servicio.
ms.date: 04/12/2021
ms.topic: conceptual
ms.openlocfilehash: ae26b599a72129b5ed7f47d76d10353be5c0e8ac
ms.sourcegitcommit: 3b5cb7fb84a427aee5b15fb96b89ec213a6536c2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/14/2021
ms.locfileid: "107498006"
---
# <a name="troubleshoot-azure-arc-enabled-servers-agent-connection-issues"></a>Solución de problemas de conexión del agente de los servidores habilitados para Azure Arc

En este artículo se proporciona información sobre la solución de problemas que pueden producirse al intentar configurar el agente de Connected Machine de los servidores habilitados para Azure Arc en Windows o Linux. Se incluyen los métodos de instalación tanto interactiva como a escala al configurar la conexión al servicio. Para información general, consulte [Introducción a los servidores habilitados para Arc](./overview.md).

## <a name="agent-error-codes"></a>Códigos de error del agente

Si recibe un error al configurar el agente de servidores habilitados para Azure Arc, la tabla siguiente puede ayudarle a identificar la posible causa y los pasos sugeridos para resolver el problema. Necesitará el código de error `AZCM0000` ("0000", puede ser cualquier número de 4 cifras) impreso en la salida de la consola o del script para continuar.

| Código de error | Causa probable | Corrección sugerida |
|------------|----------------|-----------------------|
| AZCM0000 | Acción realizada correctamente | N/D |
| AZCM0001 | se ha producido un error desconocido. | Póngase en contacto con el Soporte técnico de Microsoft para más ayuda. |
| AZCM0011 | El usuario canceló la acción (CTRL+C) | Vuelva a probar el comando anterior. |
| AZCM0012 | El token de acceso proporcionado no es válido | Obtenga un nuevo token de acceso e inténtelo de nuevo. |
| AZCM0013 | Las etiquetas proporcionadas no son válidas | Compruebe que las etiquetas están entre comillas dobles, separadas por comas, y que los nombres o valores con espacios se incluyen entre comillas simples: `--tags "SingleName='Value with spaces',Location=Redmond"`.
| AZCM0014 | La nube no es válida | Especifique una nube compatible: `AzureCloud` o `AzureUSGovernment`. |
| AZCM0015 | El identificador de correlación especificado no es un GUID válido | Proporcione un identificador único global válido para `--correlation-id`. |
| AZCM0016 | Falta un parámetro obligatorio | Revise la salida para identificar qué parámetros faltan. |
| AZCM0017 | El nombre del recurso no es válido | Especifique un nombre que solo use caracteres alfanuméricos, guiones o caracteres de subrayado. El nombre no puede terminar en guion nien un carácter de subrayado. |
| AZCM0018 | El comando se ejecutó sin privilegios administrativos | Vuelva a intentar el comando con privilegios de administrador o raíz en un símbolo del sistema con privilegios elevados o una sesión de consola. |
| AZCM0041 | Las credenciales proporcionadas no son válidas | Para el inicio de sesión en dispositivos, compruebe que la cuenta de usuario especificada tiene acceso al inquilino y a la suscripción donde se creará el recurso de servidor. Para el inicio de sesión en entidades de servicio, compruebe si el identificador de cliente y el secreto son correctos, la fecha de expiración del secreto y si la entidad de servicio es del mismo inquilino donde se va a crear el recurso de servidor. |
| AZCM0042 | Error al crear el recurso de servidor habilitado para Arc | Compruebe que el usuario o la entidad de servicio especificados tienen acceso para crear recursos de servidor habilitados para Arc en el grupo de recursos especificado. |
| AZCM0043 | Error al eliminar el recurso de servidor habilitado para Arc | Compruebe que el usuario o la entidad de servicio especificados tienen acceso para eliminar recursos de servidor habilitados para Arc en el grupo de recursos especificado. Si el recurso ya no existe en Azure, use la marca `--force-local-only` para continuar. |
| AZCM0044 | Ya existe un recurso compartido con ese nombre | Especifique un nombre diferente para el parámetro `--resource-name` o elimine el servidor habilitado para Arc existente en Azure e inténtelo de nuevo. |
| AZCM0061 | No se puede acceder al servicio del agente | Compruebe que ejecuta el comando en un contexto de usuario con privilegios elevados (administrador o raíz) y que el servicio HIMDS se está ejecutando en el servidor. |
| AZCM0062 | Error de conexión al servidor | Revise otros códigos de error en la salida para información más específica. Si el error se produjo después de crear el recurso de Azure, debe eliminar el servidor de Arc del grupo de recursos antes de reintentar. |
| AZCM0063 | Error al desconectar el servidor | Revise otros códigos de error en la salida para información más específica. Si este error persiste, puede eliminar el recurso en Azure y ejecutar `azcmagent disconnect --force-local-only` en el servidor para desconectar el agente. |
| AZCM0064 | El servicio no responde | Compruebe el estado del servicio `himds` para asegurarse de que se está ejecutando. Inicie el servicio si no está en ejecución. Si se está ejecutando, espere un minuto y vuelva a intentarlo. |
| AZCM0065 | Error interno de comunicación con el agente | Para recibir asistencia, póngase en contacto con el Soporte técnico de Microsoft. |
| AZCM0066 | El servicio web del agente no responde o no está disponible | Para recibir asistencia, póngase en contacto con el Soporte técnico de Microsoft. |
| AZCM0067 | El agente ya está conectado a Azure | Siga los pasos descritos en [Desconectar el agente](manage-agent.md#unregister-machine) e inténtelo de nuevo. |
| AZCM0068 | Error interno al desconectar el servidor de Azure | Para recibir asistencia, póngase en contacto con el Soporte técnico de Microsoft. |
| AZCM0081 | Error al descargar el certificado de identidad administrada de Azure Active Directory | Si se encuentra este mensaje al intentar conectar el servidor a Azure, el agente no podrá comunicarse con el servicio Azure Arc. Elimine el recurso de Azure e intente conectarse de nuevo. |
| AZCM0101 | El comando no se ha analizado correctamente | Ejecute `azcmagent <command> --help` para revisar la sintaxis de comandos correcta. |
| AZCM0102 | No se puede recuperar el nombre de host del equipo | Ejecute `hostname` para comprobar si hay mensajes de error de nivel de sistema y póngase en contacto con el Soporte técnico de Microsoft. |
| AZCM0103 | Error al generar las claves de RSA | Para recibir asistencia, póngase en contacto con el Soporte técnico de Microsoft. |
| AZCM0104 | No se pudo leer la información del sistema | Compruebe que la identidad usada para ejecutar `azcmagent` tiene privilegios de administrador o raíz en el sistema e inténtelo de nuevo. |

## <a name="agent-verbose-log"></a>Registro detallado del agente

Antes de seguir los pasos de solución de problemas que se describen más adelante en este artículo, la información mínima que necesita es el registro detallado. Contiene la salida de los comandos de la herramienta **azcmagent**, cuando se usa el argumento verbose (-v). Los archivos de registro se escriben en `%ProgramData%\AzureConnectedMachineAgent\Log\azcmagent.log` para Windows, y en `/var/opt/azcmagent/log/azcmagent.log` para Linux.

### <a name="windows"></a>Windows

A continuación encontrará un ejemplo del comando para habilitar el registro detallado con el agente de Connected Machine para Windows al realizar una instalación interactiva.

```console
& "$env:ProgramFiles\AzureConnectedMachineAgent\azcmagent.exe" connect --resource-group "resourceGroupName" --tenant-id "tenantID" --location "regionName" --subscription-id "subscriptionID" --verbose
```

El siguiente es un ejemplo del comando para habilitar el registro detallado con el agente de Connected Machine para Windows al realizar una instalación a escala mediante una entidad de servicio.

```console
& "$env:ProgramFiles\AzureConnectedMachineAgent\azcmagent.exe" connect `
  --service-principal-id "{serviceprincipalAppID}" `
  --service-principal-secret "{serviceprincipalPassword}" `
  --resource-group "{ResourceGroupName}" `
  --tenant-id "{tenantID}" `
  --location "{resourceLocation}" `
  --subscription-id "{subscriptionID}"
  --verbose
```

### <a name="linux"></a>Linux

A continuación encontrará un ejemplo del comando para habilitar el registro detallado con el agente de Connected Machine para Linux al realizar una instalación interactiva.

>[!NOTE]
>Debe tener permisos de acceso *raíz* en máquinas Linux para ejecutar **azcmagent**.

```bash
azcmagent connect --resource-group "resourceGroupName" --tenant-id "tenantID" --location "regionName" --subscription-id "subscriptionID" --verbose
```

El siguiente es un ejemplo del comando para habilitar el registro detallado con el agente de Connected Machine para Linux al realizar una instalación a escala mediante una entidad de servicio.

```bash
azcmagent connect \
  --service-principal-id "{serviceprincipalAppID}" \
  --service-principal-secret "{serviceprincipalPassword}" \
  --resource-group "{ResourceGroupName}" \
  --tenant-id "{tenantID}" \
  --location "{resourceLocation}" \
  --subscription-id "{subscriptionID}"
  --verbose
```

## <a name="agent-connection-issues-to-service"></a>Problemas de conexión del agente al servicio

En la tabla siguiente se enumeran algunos de los errores conocidos y las sugerencias sobre cómo solucionarlos y resolverlos.

|Message |Error |Causa probable |Solución |
|--------|------|---------------|---------|
|No se puede adquirir el flujo de dispositivo del token de autorización. |`Error occurred while sending request for Device Authorization Code: Post https://login.windows.net/fb84ce97-b875-4d12-b031-ef5e7edf9c8e/oauth2/devicecode?api-version=1.0:  dial tcp 40.126.9.7:443: connect: network is unreachable.` |No se puede comunicar con el punto de conexión `login.windows.net`. | Verifique la conectividad al punto de conexión. |
|No se puede adquirir el flujo de dispositivo del token de autorización. |`Error occurred while sending request for Device Authorization Code: Post https://login.windows.net/fb84ce97-b875-4d12-b031-ef5e7edf9c8e/oauth2/devicecode?api-version=1.0:  dial tcp 40.126.9.7:443: connect: network is Forbidden`. |Un proxy o firewall bloquean el acceso al punto de conexión `login.windows.net`. | Verifique la conectividad al punto de conexión y que no esté bloqueada por un firewall o un servidor proxy. |
|No se puede adquirir el flujo de dispositivo del token de autorización.  |`Error occurred while sending request for Device Authorization Code: Post https://login.windows.net/fb84ce97-b875-4d12-b031-ef5e7edf9c8e/oauth2/devicecode?api-version=1.0:  dial tcp lookup login.windows.net: no such host`. | El objeto de directiva de grupo *Computer Configuration\ Administrative Templates\ System\ User Profiles\ Eliminar perfiles de usuario con una antigüedad superior al número de días especificado al reiniciar el sistema* está habilitado. | Compruebe que el GPO está habilitado y que tiene como destino la máquina afectada. Consulte la nota al pie <sup>[1](#footnote1)</sup> para más información. |
|No se puede adquirir el token de autorización del SPN. |`Failed to execute the refresh request. Error = 'Post https://login.windows.net/fb84ce97-b875-4d12-b031-ef5e7edf9c8e/oauth2/token?api-version=1.0: Forbidden'` |Un proxy o firewall bloquean el acceso al punto de conexión `login.windows.net`. |Verifique la conectividad al punto de conexión y que no esté bloqueada por un firewall o un servidor proxy. |
|No se puede adquirir el token de autorización del SPN. |`Invalid client secret is provided` |Secreto incorrecto o no válido de la entidad de servicio. |Verifique el secreto de la entidad de servicio. |
| No se puede adquirir el token de autorización del SPN. |`Application with identifier 'xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx' was not found in the directory 'xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx'. This can happen if the application has not been installed by the administrator of the tenant or consented to by any user in the tenant` |Entidad de servicio o identificador de inquilino incorrectos. |Verifique la entidad de servicio o el identificador del inquilino.|
|Obtener respuesta a recursos de ARM |`The client 'username@domain.com' with object id 'xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx' does not have authorization to perform action 'Microsoft.HybridCompute/machines/read' over scope '/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourcegroups/myResourceGroup/providers/Microsoft.HybridCompute/machines/MSJC01' or the scope is invalid. If access was recently granted, please refresh your credentials."}}" Status Code=403` |Credenciales o permisos incorrectos. |Verifique que usted o la entidad de servicio sean miembros del rol **Incorporación de Azure Connected Machine**. |
|No se puede aplicar AzcmagentConnectr al recurso de ARM. |`The subscription is not registered to use namespace 'Microsoft.HybridCompute'` |Los proveedores de recursos de Azure no están registrados. |Registre los [proveedores de recursos](./agent-overview.md#register-azure-resource-providers). |
|No se puede aplicar AzcmagentConnectr al recurso de ARM. |`Get https://management.azure.com/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourcegroups/myResourceGroup/providers/Microsoft.HybridCompute/machines/MSJC01?api-version=2019-03-18-preview:  Forbidden` |El servidor proxy o firewall bloquean el acceso al punto de conexión `management.azure.com`. |Verifique la conectividad al punto de conexión y que no esté bloqueada por un firewall o un servidor proxy. |

<a name="footnote1"></a><sup>1</sup>Si este GPO está habilitado y se aplica a máquinas con el agente de Connected Machine, se elimina el perfil de usuario asociado a la cuenta integrada especificada para el servicio *himds*. Como resultado, también elimina el certificado de autenticación usado para comunicarse con el servicio almacenado en caché en el almacén local de certificados durante 30 días. Antes del límite de 30 días, se realiza un intento de renovar el certificado. Para resolver este problema, siga los pasos para [anular el registro de la máquina](manage-agent.md#unregister-machine) y, luego, vuelva a registrarlo en el servicio mediante `azcmagent connect`.

## <a name="next-steps"></a>Pasos siguientes

Si su problema no aparece aquí o no puede resolverlo, intente obtener ayuda adicional mediante uno de los siguientes canales:

* Obtenga respuestas de expertos de Azure a través de [Preguntas y respuestas de Microsoft](/answers/topics/azure-arc.html).

* Póngase en contacto con [@AzureSupport](https://twitter.com/azuresupport), la cuenta oficial de Microsoft Azure para mejorar la experiencia del cliente. El Soporte técnico de Azure pone en contacto a la comunidad de Azure con respuestas, soporte técnico y expertos.

* Registrar un incidente de soporte técnico de Azure. Vaya al [sitio de Soporte técnico de Azure](https://azure.microsoft.com/support/options/) y seleccione **Obtener soporte técnico**.
