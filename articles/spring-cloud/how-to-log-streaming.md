---
title: Streaming de registros de aplicaciones de Azure Spring Cloud en tiempo real
description: Uso del streaming de registros para ver los registros de aplicaciones al instante
author: MikeDodaro
ms.author: barbkess
ms.service: spring-cloud
ms.topic: how-to
ms.date: 01/14/2019
ms.custom: devx-track-java, devx-track-azurecli
ms.openlocfilehash: b87f3221e62db6999dd67f475055f699a74c4c2a
ms.sourcegitcommit: bb9a6c6e9e07e6011bb6c386003573db5c1a4810
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110495166"
---
# <a name="stream-azure-spring-cloud-app-logs-in-real-time"></a>Streaming de registros de aplicaciones de Azure Spring Cloud en tiempo real

**Este artículo se aplica a:** ✔️ Java ✔️ C#

Azure Spring Cloud permite el streaming de registros en la CLI de Azure para obtener registros de la consola de la aplicación en tiempo real para solucionar problemas. También puede realizar el [análisis de registros y métricas con la configuración de diagnóstico](./diagnostic-services.md).

## <a name="prerequisites"></a>Prerequisites

* Instale la [extensión de la CLI de Azure](/cli/azure/install-azure-cli) para Spring Cloud, la versión mínima es 0.2.0.
* Una instancia de **Azure Spring Cloud** con una aplicación en ejecución, por ejemplo [Spring Cloud App](./quickstart.md).

> [!NOTE]
>  La extensión de la CLI de ASC se actualiza de la versión 0.2.0 a la 0.2.1. Este cambio afecta a la sintaxis del comando de la transmisión de registros `az spring-cloud app log tail`, que se reemplaza por `az spring-cloud app logs`. El comando `az spring-cloud app log tail` quedará en desuso en una futura versión. Si ha usado la versión 0.2.0, puede actualizar a 0.2.1. En primer lugar, quite la versión anterior con el comando `az extension remove -n spring-cloud`.  Luego, instale la 0.2.1 mediante el comando: `az extension add -n spring-cloud`.

## <a name="use-cli-to-tail-logs"></a>Uso de la CLI para el final de los registros

Para evitar especificar repetidamente el nombre del grupo de recursos y el nombre de la instancia de servicio, establezca el nombre del grupo de recursos y el nombre del clúster predeterminados.
```azurecli
az configure --defaults group=<service group name>
az configure --defaults spring-cloud=<service instance name>
```
En los siguientes ejemplos, el grupo de recursos y el nombre del servicio se omitirán en los comandos.

### <a name="tail-log-for-app-with-single-instance"></a>Final del registro de la aplicación con una sola instancia
Si una aplicación denominada auth-service solo tiene una instancia, puede ver el registro de la instancia de la aplicación con el siguiente comando:
```azurecli
az spring-cloud app logs -n auth-service
```
Esto devolverá registros:
```output
...
2020-01-15 01:54:40.481  INFO [auth-service,,,] 1 --- [main] o.apache.catalina.core.StandardService  : Starting service [Tomcat]
2020-01-15 01:54:40.482  INFO [auth-service,,,] 1 --- [main] org.apache.catalina.core.StandardEngine  : Starting Servlet engine: [Apache Tomcat/9.0.22]
2020-01-15 01:54:40.760  INFO [auth-service,,,] 1 --- [main] o.a.c.c.C.[Tomcat].[localhost].[/uaa]  : Initializing Spring embedded WebApplicationContext
2020-01-15 01:54:40.760  INFO [auth-service,,,] 1 --- [main] o.s.web.context.ContextLoader  : Root WebApplicationContext: initialization completed in 7203 ms

...
```

### <a name="tail-log-for-app-with-multiple-instances"></a>Final del registro de una aplicación con varias instancias
Si existen varias instancias de la aplicación denominada `auth-service`, puede ver el registro de la instancia mediante la opción `-i/--instance`. 

En primer lugar, puede obtener los nombres de instancia de la aplicación con el siguiente comando.

```azurecli
az spring-cloud app show -n auth-service --query properties.activeDeployment.properties.instances -o table
```
Con resultados:

```output
Name                                         Status    DiscoveryStatus
-------------------------------------------  --------  -----------------
auth-service-default-12-75cc4577fc-pw7hb  Running   UP
auth-service-default-12-75cc4577fc-8nt4m  Running   UP
auth-service-default-12-75cc4577fc-n25mh  Running   UP
``` 
Después, puede transmitir los registros de una instancia de la aplicación con la opción `-i/--instance`:

```azurecli
az spring-cloud app logs -n auth-service -i auth-service-default-12-75cc4577fc-pw7hb
```

También puede obtener los detalles de las instancias de la aplicación en Azure Portal.  Después de seleccionar **Aplicaciones** en el panel de navegación izquierdo del servicio Azure Spring Cloud, seleccione **App Instances**.

### <a name="continuously-stream-new-logs"></a>Streaming continuo de nuevos registros
De forma predeterminada, `az spring-cloud app logs` imprime solo los registros existentes transmitidos a la consola de la aplicación y, a continuación, se cierra. Si desea hacer streaming de los nuevos registros, agregue-f (--follow):  

```azurecli
az spring-cloud app logs -n auth-service -f
``` 
Para comprobar todas las opciones de registro admitidas:
```azurecli
az spring-cloud app logs -h 
```

### <a name="format-json-structured-logs"></a>Aplicación de formato a los registros estructurados de JSON

> [!NOTE]
> Requiere la versión 2.4.0 o posterior de la extensión spring-cloud.

Cuando el [registro de aplicaciones estructurado](./structured-app-log.md) está habilitado para la aplicación, los registros se imprimen en formato JSON. Esto dificulta la lectura. El argumento `--format-json` se puede usar para dar formato a los registros JSON en formato legible.

```shell
# Raw JSON log
$ az spring-cloud app logs -n auth-service
{"timestamp":"2021-05-26T03:35:27.533Z","logger":"com.netflix.discovery.DiscoveryClient","level":"INFO","thread":"main","mdc":{},"message":"Disable delta property : false"}
{"timestamp":"2021-05-26T03:35:27.533Z","logger":"com.netflix.discovery.DiscoveryClient","level":"INFO","thread":"main","mdc":{},"message":"Single vip registry refresh property : null"}

# Formatted JSON log
$ az spring-cloud app logs -n auth-service --format-json
2021-05-26T03:35:27.533Z  INFO [           main] com.netflix.discovery.DiscoveryClient   : Disable delta property : false
2021-05-26T03:35:27.533Z  INFO [           main] com.netflix.discovery.DiscoveryClient   : Single vip registry refresh property : null
```

El argumento `--format-json` también toma un formato personalizado opcional, utilizando la [sintaxis de cadena de formato](https://docs.python.org/3/library/string.html#format-string-syntax) del argumento de la palabra clave.

```shell
# Custom format
$ az spring-cloud app logs -n auth-service --format-json="{message}{n}"
Disable delta property : false
Single vip registry refresh property : null
```

> El formato predeterminado que se usa es
> ```
> {timestamp} {level:>5} [{thread:>15.15}] {logger{39}:<40.40}: {message}{n}{stackTrace}
> ```

## <a name="next-steps"></a>Pasos siguientes
* [Inicio rápido: Supervisión de aplicaciones de Azure Spring Cloud con registros, métricas y seguimiento](./quickstart-logs-metrics-tracing.md)
* [Análisis de registros y métricas con la configuración de diagnóstico](./diagnostic-services.md)
