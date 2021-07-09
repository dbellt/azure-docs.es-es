---
title: Solución de problemas de implementación de puntos de conexión en línea administrados (versión preliminar)
titleSuffix: Azure Machine Learning
description: Aprenda a solucionar algunos errores comunes de implementación y puntuación con puntos de conexión en línea administrados.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
author: petrodeg
ms.author: petrodeg
ms.reviewer: laobri
ms.date: 05/13/2021
ms.topic: troubleshooting
ms.openlocfilehash: f493cfc21ff3f5e2aa122bbbc08f24e1a759558e
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110480951"
---
# <a name="troubleshooting-managed-online-endpoints-deployment-and-scoring-preview"></a>Solución de problemas de implementación y puntuación de puntos de conexión en línea administrados (versión preliminar)

Aprenda a resolver problemas comunes en la implementación y la puntuación de puntos de conexión en línea administrados de Azure Machine Learning (versión preliminar).

Este documento se estructura de la manera en que debe abordar la solución de problemas:

1. Use la [implementación local](#deploy-locally) para probar y depurar los modelos localmente antes de realizar la implementación en la nube.
1. Use los [registros de contenedor](#get-container-logs) para ayudar a depurar problemas.
1. Conozca los [errores de implementación comunes](#common-deployment-errors) que pueden surgir y cómo corregirlos.

En la sección [Códigos de estado HTTP](#http-status-codes) se explica cómo se asignan los errores de invocación y predicción a los códigos de estado HTTP al puntuar puntos de conexión con solicitudes REST.

[!INCLUDE [preview disclaimer](../../includes/machine-learning-preview-generic-disclaimer.md)]

## <a name="prerequisites"></a>Requisitos previos

* Una **suscripción de Azure**. Pruebe la [versión gratuita o de pago de Azure Machine Learning](https://aka.ms/AMLFree).
* La[CLI de Azure](/cli/azure/install-azure-cli).
* El artículo [Instalación, configuración y uso de la CLI 2.0 (versión preliminar)](how-to-configure-cli.md).

## <a name="deploy-locally"></a>Implementación local

Se está implementando un modelo en un entorno de Docker local con la implementación local. La implementación local es útil para pruebas y depuración antes de su implementación en la nube.

La implementación local admite la creación, la actualización y la eliminación de un punto de conexión local. También permite invocar y obtener registros desde el punto de conexión. Para usar la implementación local, agregue `--local` al comando de la CLI adecuado:

```azurecli
az ml endpoint create -n <endpoint-name> -f <spec_file.yaml> --local
```
Como parte de la implementación local, se llevan a cabo los pasos siguientes:

- Docker compila una nueva imagen de contenedor o extrae una imagen existente de la caché local de Docker. Se usa una imagen existente si hay una que coincida con la parte del entorno del archivo de especificación.
- Docker inicia un nuevo contenedor con artefactos locales montados, como archivos de modelo y código.

Para más información, consulte [Implementación y depuración locales mediante puntos de conexión locales](how-to-deploy-managed-online-endpoints.md#deploy-and-debug-locally-using-local-endpoints).

## <a name="get-container-logs"></a>Obtención de registros de contenedor

No se puede obtener acceso directo a la máquina virtual donde está implementado el modelo. Sin embargo, puede obtener registros de algunos de los contenedores que se ejecutan en ella. La cantidad de información depende del estado de aprovisionamiento de la implementación. Si el contenedor especificado está en funcionamiento, verá su salida de la consola; de lo contrario, verá un mensaje para intentarlo de nuevo más tarde.

Para ver la salida del registro del contenedor, use el siguiente comando de la CLI:

```azurecli
az ml endpoint get-logs -n <endpoint-name> -d <deployment-name> -l 100
```

o

```azurecli
    az ml endpoint get-logs --name <endpoint-name> --deployment <deployment-name> --lines 100
```

Agregue `--resource-group` y `--workspace-name` a los comandos anteriores si aún no ha establecido estos parámetros a través de `az configure`.

Para ver información sobre cómo establecer estos parámetros y si los valores actuales ya están establecidos, ejecute:

```azurecli
az ml endpoint get-logs -h
```

De forma predeterminada, los registros se extraen del servidor de inferencia. Los registros incluyen los de la consola del servidor de inferencia, que contiene instrucciones print/log del código "score.py".

> [!NOTE]
> Si usa el registro de Python, asegúrese de utilizar el orden de nivel de registro correcto para los mensajes que se van a publicar en los registros. Por ejemplo, INFO.


También puede obtener registros del contenedor de inicializadores de almacenamiento pasando `–-container storage-initializer`. Estos registros contienen información sobre si el código y los datos del modelo se descargaron correctamente en el contenedor.

Agregue `--help` o `--debug` a los comandos para ver más información. Incluya el encabezado `x-ms-client-request-id` para ayudar a solucionar problemas.

## <a name="common-deployment-errors"></a>Errores de implementación frecuentes

A continuación, se muestra una lista de errores de implementación comunes que se notifican como parte del estado de la operación de implementación.

### <a name="err_1100-not-enough-quota"></a>ERR_1100: No hay cuota suficiente

Antes de implementar un modelo, debe tener suficiente cuota de proceso. Esta cuota define la cantidad de núcleos virtuales disponibles por suscripción, área de trabajo, SKU y región. Cada implementación se resta de la cuota disponible y se vuelve a agregar después de la eliminación, en función del tipo de la SKU.

Una posible mitigación es comprobar si hay implementaciones no utilizadas que se puedan eliminar. O bien, puede enviar una [solicitud de aumento de cuota](./how-to-manage-quotas.md).

### <a name="err_1200-unable-to-download-user-container-image"></a>ERR_1200: Unable to download user container image (No se puede descargar la imagen de contenedor de usuarios)

Durante la creación de la implementación, después del aprovisionamiento de proceso, Azure intenta extraer la imagen de contenedor de usuarios de la instancia de Azure Container Registry (ACR) privada del área de trabajo. Podría haber dos posibles problemas.

- No se encuentra la imagen de contenedor de usuarios.

  Asegúrese de que la imagen de contenedor está disponible en la instancia de ACR del área de trabajo.
Por ejemplo, si la imagen es `testacr.azurecr.io/azureml/azureml_92a029f831ce58d2ed011c3c42d35acb:latest`, compruebe el repositorio con `az acr repository show-tags -n testacr --repository azureml/azureml_92a029f831ce58d2ed011c3c42d35acb --orderby time_desc --output table`.

- Hay un problema de permiso al acceder a ACR.

  Para extraer la imagen, Azure usa [identidades administradas](../active-directory/managed-identities-azure-resources/overview.md) para acceder a ACR. 

  - Si ha creado el punto de conexión asociado con SystemAssigned, se concede automáticamente el permiso de control de acceso basado en rol (RBAC) de Azure y no se necesitan más permisos.
  - Si creó el punto de conexión asociado con UserAssigned, la identidad administrada del usuario debe tener el permiso AcrPull para la instancia de ACR del área de trabajo.

Para más información sobre este error, ejecute:

```azurecli
az ml endpoint get-logs -n <endpoint-name> --deployment <deployment-name> --tail 100
```

### <a name="err_1300-unable-to-download-user-modelcode-artifacts"></a>ERR_1300: Unable to download user model\code artifacts (No se pueden descargar artefactos de código o modelo de usuario)

Después de aprovisionar el recurso de proceso, durante la creación de la implementación, Azure intenta montar los artefactos de modelo y código en el contenedor de usuario desde la cuenta de almacenamiento del área de trabajo.

- Los artefactos de modelo y código de usuario no se encuentran.

  - Asegúrese de que estén registrados en la misma área de trabajo que la implementación. Use el comando `show` para mostrar los detalles de un artefacto de modelo o código en un área de trabajo. Por ejemplo: 
  
    ```azurecli
    az ml model show --name <model-name>
    az ml code show --name <code-name> --version <version>
    ```

  - También puede comprobar si los blobs existen en la cuenta de almacenamiento del área de trabajo.

    Por ejemplo, si el blob es `https://foobar.blob.core.windows.net/210212154504-1517266419/WebUpload/210212154504-1517266419/GaussianNB.pkl`, puede usar este comando para comprobar si existe: `az storage blob exists --account-name foobar --container-name 210212154504-1517266419 --name WebUpload/210212154504-1517266419/GaussianNB.pkl --subscription <sub-name>`

- Problema de permiso al acceder a ACR.

  Para extraer blobs, Azure usa [identidades administradas](../active-directory/managed-identities-azure-resources/overview.md) para acceder a la cuenta de almacenamiento.

  - Si ha creado el punto de conexión asociado con SystemAssigned, el permiso de control de acceso basado en rol (RBAC) de Azure se concede automáticamente y no se necesitan más permisos.

  - Si creó el punto de conexión asociado con UserAssigned, la identidad administrada del usuario debe tener permiso de lector de datos de blob de almacenamiento en la cuenta de almacenamiento del área de trabajo.

Para más información sobre este error, ejecute:

```azurecli
az ml endpoint get-logs -n <endpoint-name> --deployment <deployment-name> --lines 100
```

### <a name="err_2100-unable-to-start-user-container"></a>ERR_2100: Unable to start user container (No se puede iniciar el contenedor de usuarios)

Para ejecutar el archivo `score.py` proporcionado como parte de la implementación, Azure crea un contenedor que incluye todos los recursos que necesita `score.py` y ejecuta el script de puntuación en ese contenedor.

Este error significa que este contenedor no se pudo iniciar, lo que implica que no se pudo puntuar. Podría ser que el contenedor solicite más recursos de los que `instance_type` podría admitir. Si es así, considere la posibilidad de actualizar el valor de `instance_type` de la implementación en línea.

Para conocer el motivo exacto de un error, ejecute: 

```azurecli
az ml endpoint get-logs
```

### <a name="err_2200-user-container-has-crashedterminated"></a>ERR_2200: User container has crashed\terminated (El contenedor de usuarios se bloqueó o finalizó)

Para ejecutar el archivo `score.py` proporcionado como parte de la implementación, Azure crea un contenedor que incluye todos los recursos que necesita `score.py` y ejecuta el script de puntuación en ese contenedor.  El error en este escenario es que este contenedor se bloquea al ejecutarse, lo que significa que no se pudo puntuar. Este error se produce en los casos siguientes:

- Hay un error en `score.py`. Utilice `get--logs` para ayudar a diagnosticar los problemas comunes:
    - Un paquete que se importó pero no está en el entorno de Conda
    - Un error de sintaxis
    - Un error en el método `init()`
- Los sondeos de preparación o ejecución no están configurados correctamente.
- Hay un error en la configuración del entorno del contenedor, como una dependencia que falta.

### <a name="err_5000-internal-error"></a>ERR_5000: Error interno

Aunque hacemos todo lo posible para proporcionar un servicio estable y confiable, a veces las cosas no van como se espera. Si recibe este error, significa que algo no funciona por nuestra parte y es necesario corregirlo. Envíe una [incidencia de soporte técnico de cliente](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest) con toda la información relacionada y solucionaremos el problema.  

## <a name="http-status-codes"></a>Códigos de estado HTTP

Al acceder a los puntos de conexión en línea administrados con solicitudes REST, los códigos de estado devueltos cumplen los estándares de los [códigos de estado HTTP](https://aka.ms/http-status-codes). A continuación, se muestran detalles sobre cómo se asignan los errores de predicción e invocación de puntos de conexión administrados a códigos de estado HTTP.

| status code| Frase de motivo |  Motivo de que se devuelva este código. |
| --- | --- | --- |
| 200 | Aceptar | El modelo se ejecutó correctamente, dentro del límite de latencia. |
| 401 | No autorizado | No tiene permiso para realizar la acción solicitada, como puntuación, o el token ha expirado. |
| 404 | No encontrado | La dirección URL no es correcta. |
| 408 | Tiempo de espera de solicitud | La ejecución del modelo tardó más que el tiempo de espera proporcionado en `request_timeout_ms` en el elemento `request_settings` de la configuración de implementación del modelo.|
| 413 | Carga demasiado grande | La carga de la solicitud es superior a 1,5 megabytes. |
| 424 | Error del modelo; original-code=`<original code>` | Si el contenedor del modelo devuelve una respuesta distinta de 200, Azure devuelve el error 424. |
| 424 | Carga de respuesta demasiado grande | Si el contenedor devuelve una carga superior a 1,5 megabytes, Azure devuelve el error 424. |
| 429 | Limitación de frecuencia | Intentó enviar más de 100 solicitudes por segundo al punto de conexión. |
| 429 | Demasiadas solicitudes pendientes | El modelo está recibiendo más solicitudes de las que puede manejar. Se permiten 2 *`max_concurrent_requests_per_instance`* `instance_count` solicitudes cada vez. Las solicitudes adicionales se rechazan. Puede confirmar esta configuración en la configuración de implementación del modelo en `request_settings` y `scale_settings`. Si usa la escalabilidad automática, el modelo recibe solicitudes más rápido de lo que el sistema puede escalarse verticalmente. Con la escalabilidad automática, puede intentar volver a enviar solicitudes con [retroceso exponencial](https://aka.ms/exponential-backoff). Si lo hace, puede dar tiempo a que se ajuste el sistema. |
| 500 | Error interno del servidor | Error en la infraestructura aprovisionada por Azure ML. |

## <a name="next-steps"></a>Pasos siguientes

Más información acerca de la implementación:

* [Implementación y puntuación de un modelo de aprendizaje automático con un punto de conexión en línea administrado (versión preliminar)](how-to-deploy-managed-online-endpoints.md)
* [Implementación segura para puntos de conexión en línea (versión preliminar)](how-to-safely-rollout-managed-endpoints.md)

