---
title: Referencia de errores para las comprobaciones de estado del registro
description: Códigos de error y posibles soluciones de los problemas encontrados tras la ejecución del comando az acr para el diagnóstico de comprobación de mantenimiento en Azure Container Registry
ms.topic: article
ms.date: 01/25/2021
ms.openlocfilehash: 12cdd1e75a51097bab3830cdd0e6567e53f91dce
ms.sourcegitcommit: 9ad20581c9fe2c35339acc34d74d0d9cb38eb9aa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/27/2021
ms.locfileid: "110537425"
---
# <a name="health-check-error-reference"></a>Referencia de errores de la comprobación de mantenimiento

A continuación se indican los detalles de los códigos de error que devuelve el comando [az acr para la comprobación de mantenimiento][az-acr-check-health]. Para cada error, se enumeran posibles soluciones.

Para información sobre la ejecución de `az acr check-health`, consulte [Comprobación del mantenimiento de un registro de contenedor de Azure](container-registry-check-health.md).

## <a name="docker_command_error"></a>DOCKER_COMMAND_ERROR

Este error indica que no se pudo encontrar el cliente Docker para la CLI. Como resultado, no se ejecutan las siguientes comprobaciones adicionales: buscar la versión de Docker, evaluar el estado del demonio de Docker y ejecutar el comando docker pull.

*Posibles soluciones*: instale el cliente de Docker; agregue una ruta de acceso de Docker a las variables del sistema.

## <a name="docker_daemon_error"></a>DOCKER_DAEMON_ERROR

Este error indica que el estado del demonio de Docker no está disponible o que no se pudo establecer conexión con él mediante la CLI. Como resultado, las operaciones de Docker (como `docker login` y `docker pull`) no están disponibles a través de la CLI.

*Posibles soluciones*: reinicie el demonio de Docker o valide que esté instalado correctamente.

## <a name="docker_version_error"></a>DOCKER_VERSION_ERROR

Este error indica que la CLI no pudo ejecutar el comando `docker --version`.

*Posibles soluciones*: intente ejecutar el comando manualmente, asegúrese de que tiene la última versión de la CLI e investigue el mensaje de error.

## <a name="docker_pull_error"></a>DOCKER_PULL_ERROR

Este error indica que la CLI no pudo extraer una imagen de ejemplo en su entorno.

*Posibles soluciones*: valide que todos los componentes necesarios para extraer una imagen se están ejecutando correctamente.

## <a name="helm_command_error"></a>HELM_COMMAND_ERROR

Este error indica que la CLI no pudo encontrar el cliente de Helm, lo que impide la realización de otras operaciones de Helm.

*Posibles soluciones*: compruebe que el cliente de Helm está instalado y que su ruta de acceso se ha agregado a las variables de entorno del sistema.

## <a name="helm_version_error"></a>HELM_VERSION_ERROR

Este error indica que la CLI no pudo determinar la versión de Helm instalada. Esto puede ocurrir si la versión de la CLI de Azure (o si la versión de Helm) que se usa está obsoleta.

*Posibles soluciones*: actualice a la versión más reciente de la CLI de Azure o a la versión de Helm recomendada; ejecute el comando manualmente e investigue el mensaje de error.

## <a name="cmk_error"></a>CMK_ERROR

Este error significa que el Registro no puede acceder a la identidad administrada asignada por el usuario o asignada por el sistema que se usa para configurar el cifrado del Registro con una clave administrada por el cliente. Es posible que se haya eliminado la identidad administrada.  

*Posible solución*: para resolver el problema y girar la clave mediante una identidad administrada diferente, consulte pasos para solucionar los problemas relacionados con [la identidad asignada por el usuario](container-registry-customer-managed-keys.md#troubleshoot).

## <a name="connectivity_dns_error"></a>CONNECTIVITY_DNS_ERROR

Este error indica que se ha hecho ping al DNS del servidor de inicio de sesión del registro determinado pero no ha respondido, lo que significa que no está disponible. Esto puede indicar problemas de conectividad. También es posible que el registro no exista, que el usuario no tenga permisos en el registro (para recuperar correctamente su servidor de inicio de sesión) o que el registro de destino esté en una nube diferente de la que se usó en la CLI de Azure.

*Posibles soluciones*: valide la conectividad; compruebe la ortografía del registro y que el registro existe; compruebe que el usuario tiene los permisos adecuados en el registro y que la nube del registro es la misma que se usó en la CLI de Azure.

## <a name="connectivity_forbidden_error"></a>CONNECTIVITY_FORBIDDEN_ERROR

Este error indica que el punto de conexión de comprobación del registro determinado respondió con un código de estado HTTP 403 Prohibido. Esto significa que los usuarios no tienen acceso al registro, probablemente debido a una configuración de red virtual o a que no está permitido el acceso al punto de conexión público del registro. Para ver las reglas de firewall configuradas actualmente, ejecute `az acr show --query networkRuleSet --name <registry>`.

*Posibles soluciones*: quite las reglas de red virtual o agregue la dirección IP del cliente actual a la lista de permitidos.

## <a name="connectivity_challenge_error"></a>CONNECTIVITY_CHALLENGE_ERROR

Este error indica que el punto de conexión de comprobación del registro de destino no ha emitido un desafío.

*Posibles soluciones*: vuelva a intentarlo más tarde. Si el error persiste, abra una incidencia en https://aka.ms/acr/issues.

## <a name="connectivity_aad_login_error"></a>CONNECTIVITY_AAD_LOGIN_ERROR

Este error indica que el punto de conexión de comprobación del registro de destino emitió un desafío, pero el registro no admite la autenticación de Azure Active Directory.

*Posibles soluciones*: pruebe otra forma de realizar la autenticación, por ejemplo, con credenciales de administrador. Si los usuarios necesitan autenticarse con Azure Active Directory, abra una incidencia en https://aka.ms/acr/issues.

## <a name="connectivity_refresh_token_error"></a>CONNECTIVITY_REFRESH_TOKEN_ERROR

Este error indica que el servidor de inicio de sesión del registro no respondió con un token de actualización, por lo que se denegó el acceso al registro de destino. Este error puede producirse si el usuario no tiene los permisos adecuados en el registro o si las credenciales de usuario para la CLI de Azure son obsoletas.

*Posibles soluciones*: compruebe si el usuario tiene los permisos adecuados en el registro; ejecute `az login` para actualizar los permisos, los tokens y las credenciales.

## <a name="connectivity_access_token_error"></a>CONNECTIVITY_ACCESS_TOKEN_ERROR

Este error indica que el servidor de inicio de sesión del registro no respondió con un token de acceso, por lo que se denegó el acceso al registro de destino. Este error puede producirse si el usuario no tiene los permisos adecuados en el registro o si las credenciales de usuario para la CLI de Azure son obsoletas.

*Posibles soluciones*: compruebe si el usuario tiene los permisos adecuados en el registro; ejecute `az login` para actualizar los permisos, los tokens y las credenciales.

## <a name="connectivity_ssl_error"></a>CONNECTIVITY_SSL_ERROR

Este error significa que el cliente no pudo establecer una conexión segura con el registro de contenedor. Este error suele producirse si se está ejecutando o utilizando un servidor proxy.

*Posibles soluciones*: Más información sobre cómo trabajar con tipos de contenido [puede encontrarse aquí](/cli/azure/use-cli-effectively).

## <a name="login_server_error"></a>LOGIN_SERVER_ERROR

Este error indica que la CLI no pudo encontrar el servidor de inicio de sesión del registro determinado y que no se encontró ningún sufijo predeterminado para la nube actual. Este error puede producirse si el registro no existe, si el usuario no tiene los permisos adecuados en el registro, si la nube del registro y la nube de la CLI de Azure actual no coinciden o si la versión de la CLI de Azure está obsoleta.

*Posibles soluciones*: compruebe que la ortografía es correcta y que el registro existe; asegúrese de que el usuario tiene los permisos adecuados en el registro y que las nubes del registro y del entorno de la CLI coinciden; actualice la CLI de Azure a la versión más reciente.

## <a name="notary_version_error"></a>NOTARY_VERSION_ERROR

Este error significa que la CLI no es compatible con la versión instalada actualmente de Docker/Notary. Intente cambiar la versión de notary.exe a una versión anterior a 0.6.0 reemplazando manualmente el cliente de Notary de la instalación de Docker para resolver este problema.

## <a name="next-steps"></a>Pasos siguientes

Si quiere conocer las opciones para comprobar el mantenimiento de un registro, consulte [Comprobar el mantenimiento de Azure Container Registry](container-registry-check-health.md).

Vea las [preguntas más frecuentes](container-registry-faq.md) para conocer las preguntas más habituales y otros problemas conocidos sobre Azure Container Registry.





<!-- LINKS - internal -->
[az-acr-check-health]: /cli/azure/acr#az_acr_check_health
