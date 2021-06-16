---
title: Escalado y reducción vertical de un grupo de servidores de Hiperescala de Azure Database for PostgreSQL mediante la CLI (azdata o kubectl)
description: Escalado y reducción vertical de un grupo de servidores de Hiperescala de Azure Database for PostgreSQL mediante la CLI (azdata o kubectl)
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
author: TheJY
ms.author: jeanyd
ms.reviewer: mikeray
ms.date: 06/02/2021
ms.topic: how-to
ms.openlocfilehash: 490eaaef7e4e8569e4422b2ef2659ced583f7ff2
ms.sourcegitcommit: c385af80989f6555ef3dadc17117a78764f83963
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/04/2021
ms.locfileid: "111407406"
---
# <a name="scale-up-and-down-an-azure-database-for-postgresql-hyperscale-server-group-using-cli-azdata-or-kubectl"></a>Escalado y reducción vertical de un grupo de servidores de Hiperescala de Azure Database for PostgreSQL mediante la CLI (azdata o kubectl)



En algunas ocasiones, es posible que necesite cambiar las características o la definición de un grupo de servidores. Por ejemplo:

- Para escalar o reducir verticalmente el número de núcleos virtuales que usa cada nodo de coordinación o de trabajo.
- Para escalar o reducir verticalmente la memoria que usa cada nodo de coordinación o de trabajo.

En esta guía se explica cómo escalar los núcleos virtuales y la memoria.

El escalado o la reducción vertical de la configuración de la memoria o los núcleos virtuales del grupo de servidores significa que puede establecer un valor mínimo o máximo para cada una de las opciones de configuración de memoria y de núcleos virtuales. Si quiere configurar el grupo de servidores para que use un número específico de núcleos virtuales o una cantidad determinada de memoria, los mínimos deben establecerse en los mismos valores que los máximos.

[!INCLUDE [azure-arc-data-preview](../../../includes/azure-arc-data-preview.md)]

## <a name="show-the-current-definition-of-the-server-group"></a>Visualización de la definición actual del grupo de servidores

Para mostrar la definición actual del grupo de servidores y ver cuál es la configuración de los núcleos virtuales y de la memoria en este momento, ejecute cualquiera de los siguientes comandos:

### <a name="cli-with-azdata"></a>CLI con azdata

```console
azdata arc postgres server show -n <server group name>
```
### <a name="cli-with-kubectl"></a>CLI con kubectl

```console
kubectl describe postgresql/<server group name> -n <namespace name>
```

Devuelve la configuración del grupo de servidores. Si ha creado el grupo de servidores con la configuración predeterminada, debería ver una definición como la siguiente:

```json
Spec:
  Dev:  false
  Engine:
    Extensions:
      Name:   citus
    Version:  12
  Scale:
    Workers:  2
  Scheduling:
    Default:
      Resources:
        Requests:
          Memory:  256Mi
...
```

## <a name="interpret-the-definition-of-the-server-group"></a>Interpretación de la definición del grupo de servidores

En la definición de un grupo de servidores, la sección que contiene la configuración de los núcleos virtuales mínimos o máximos por nodo y de la memoria mínima o máxima por nodo es la sección **"scheduling"** (programación). En esta sección, los valores máximos se conservan en una subsección denominada **"limits"** (límites), y los mínimos, en una subsección denominada **"requests"** (solicitudes).

Si establece un valor mínimo distinto del máximo, esta configuración garantizará que al grupo de servidores se le asignen los recursos solicitados que necesita. No superará los límites establecidos.

Los recursos (los núcleos virtuales y la memoria) que usará realmente el grupo de servidores dependerán de los valores máximos configurados, y de las cargas de trabajo y los recursos disponibles en el clúster. Si no se limitan los valores con un máximo, el grupo de servidores puede llegar a usar todos los recursos que el clúster de Kubernetes asigna a los nodos de Kubernetes en los que está programado el grupo de servidores.

Esa configuración de núcleos virtuales y memoria se aplica a cada uno de los roles de las instancias de Postgres que componen el grupo de servidores de Hiperescala de PostgreSQL: coordinador y trabajos. Puede definir solicitudes y límites por rol. Puede definir valores de solicitudes y límites diferentes para cada rol. También pueden ser similares en función de sus necesidades.

En una configuración predeterminada, solo la memoria mínima se establece en 256 MiB, ya que es la cantidad mínima de memoria que se recomienda para ejecutar Hiperescala de PostgreSQL.

> [!NOTE]
> La configuración de un valor mínimo no significa que el grupo de servidores usará ese mínimo. Significa que, si el grupo de servidores lo necesita, seguro que se le asignará al menos este mínimo. Por ejemplo, imaginemos que hemos establecido `--minCpu 2`. Eso no significa que el grupo de servidores usará al menos dos núcleos virtuales en todo momento, sino que puede empezar a usar menos de dos núcleos virtuales si no necesita más, y garantiza que se le asignarán al menos dos núcleos virtuales si los necesita más adelante. Esto implica que el clúster de Kubernetes asigna los recursos a otras cargas de trabajo de forma que pueda asignar esos dos núcleos virtuales al grupo de servidores si es necesario. Además, escalar y reducir verticalmente no es una operación en línea, ya que requiere el reinicio de los pods de Kubernetes.

>[!NOTE]
>Antes de modificar la configuración del sistema, asegúrese de familiarizarse con el modelo de recursos de Kubernetes [aquí](https://github.com/kubernetes/community/blob/master/contributors/design-proposals/scheduling/resources.md#resource-quantities).

## <a name="scale-up-and-down-the-server-group"></a>Escalado y reducción verticales del grupo de servidores

El escalado vertical hace referencia al aumento de los valores de los núcleos virtuales o la configuración de memoria del grupo de servidores.
La reducción vertical hace referencia a la disminución de los valores de los núcleos virtuales o la configuración de memoria del grupo de servidores.

Tenga en cuenta la configuración especificada para el clúster de Kubernetes a la hora de establecer los valores mínimos y máximos. Asegúrese de que no está estableciendo valores que el clúster de Kubernetes no puede satisfacer. Esto podría provocar errores o un comportamiento impredecible, como la falta de disponibilidad de la instancia de base de datos. Por ejemplo, si el estado del grupo de servidores permanece como _actualizando_ durante mucho tiempo después de cambiar la configuración, esto puede indicar que ha establecido los parámetros siguientes en valores que el clúster de Kubernetes no puede satisfacer. En ese caso, revierta el cambio o consulte la sección de solución de problemas.

¿Qué configuración debe establecer?
- Para establecer el mínimo de núcleos virtuales, establezca `--cores-request`.
- Para establecer el máximo de núcleos virtuales, establezca `--cores-limit`.
- Para establecer el mínimo de memoria, establezca `--memory-request`.
- Para establecer el máximo de memoria, establezca `--memory-limit`.

¿Cómo se indica a qué rol se aplica la configuración?
- Para configurar el valor para el rol de coordinador, especifique `coordinator=<value>`.
- Para configurar la configuración del rol de trabajo (la configuración especificada se establecerá en el mismo valor en todos los trabajos), especifique `worker=<value>`.


> [!CAUTION]
> Con Kubernetes, la configuración de un límite sin configurar los valores de solicitud correspondientes obliga a que el valor de la solicitud sea el mismo valor que el límite. Esto podría provocar la falta de disponibilidad del grupo de servidores, ya que es posible que los pods no se vuelvan a programar si no hay un nodo de Kubernetes disponible con recursos suficientes. Por lo tanto, para evitar esta situación, en los ejemplos siguientes se muestra cómo establecer los valores de solicitud y de límite.


**La sintaxis general es:**

```console
azdata arc postgres server edit -n <servergroup name> --memory-limit/memory-request/cores-request/cores-limit <coordinator=val1,worker=val2>
```

El valor que se indica para la configuración de memoria es un número seguido de una unidad de volumen. Por ejemplo, para indicar 1 Gb, debe indicar 1024Mi o 1Gi.
Para indicar un número de núcleos, basta con pasar un número sin unidad. 

### <a name="examples-using-the-azdata-cli"></a>Ejemplos de uso de la CLI de azdata





**Configuración del rol de coordinador para que no supere los 2 núcleos y del rol de trabajo para que no supere los 4 núcleos:**
```console
 azdata arc postgres server edit -n postgres01 --cores-request coordinator=1, --cores-limit coordinator=2
 azdata arc postgres server edit -n postgres01 --cores-request worker=1, --cores-limit worker=4
```

o bien
```console
azdata arc postgres server edit -n postgres01 --cores-request coordinator=1,worker=1 --cores-limit coordinator=4,worker=4
```

> [!NOTE]
> Para obtener más información sobre estos parámetros, ejecute `azdata arc postgres server edit --help`.

### <a name="example-using-kubernetes-native-tools-like-kubectl"></a>Ejemplo de uso de herramientas nativas de Kubernetes, como `kubectl`

Ejecute el comando: 
```console
kubectl edit postgresql/<servergroup name> -n <namespace name>
```

Esto le llevará al editor `vi`, donde podrá navegar y cambiar la configuración. Use lo siguiente para asignar el valor deseado al nombre del campo en la especificación:

> [!CAUTION]
> A continuación, se proporciona un ejemplo para ilustrar cómo podría editar la configuración. Antes de actualizar la configuración, asegúrese de establecer los parámetros en los valores que el clúster de Kubernetes pueda respetar.

Por ejemplo, si quiere establecer la siguiente configuración para los roles de coordinador y de trabajo en los valores siguientes:
- Mínimo de núcleos virtuales = `2` 
- Máximo de núcleos virtuales = `4`
- Mínimo de memoria = `512Mb`
- Máximo de memoria = `1Gb` 

Establecería la definición del grupo de servidores para que coincida con la configuración siguiente:

```json
  scheduling:
    default:
      resources:
        requests:
          memory: 256Mi
    roles:
      coordinator:
        resources:
          limits:
            cpu: "4"
            memory: 1Gi
          requests:
            cpu: "2"
            memory: 512Mi
      worker:
        resources:
          limits:
            cpu: "4"
            memory: 1Gi
          requests:
            cpu: "2"
            memory: 512Mi
```

Si no está familiarizado con el editor `vi`, puede ver una descripción de los comandos que puede necesitar [aquí](https://www.computerhope.com/unix/uvi.htm):
- Modo de edición: `i`
- Desplazamiento: flechas
- Detener la edición: `esc`
- Salir sin guardar: `:qa!`
- Salir después de guardar: `:qw!`


## <a name="reset-to-default-values"></a>Restablecimiento de los valores predeterminados
Para restablecer los valores predeterminados de los parámetros de núcleo, límites de memoria y solicitudes, edítelos y pase una cadena vacía en lugar de un valor real. Por ejemplo, si quiere restablecer el parámetro de límite de núcleos, ejecute los siguientes comandos:

```console
azdata arc postgres server edit -n postgres01 --cores-request coordinator='',worker=''
azdata arc postgres server edit -n postgres01 --cores-limit coordinator='',worker=''
```

o bien 
```console
azdata arc postgres server edit -n postgres01 --cores-request coordinator='',worker='' --cores-limit coordinator='',worker=''
```

## <a name="next-steps"></a>Pasos siguientes

- [Escalado horizontal del grupo de servidores de Hiperescala de Azure Database for PostgreSQL](scale-out-in-postgresql-hyperscale-server-group.md)
- [Conceptos de almacenamiento de Kubernetes y configuración de almacenamiento](storage-configuration.md)
- [Modelo de recursos de Kubernetes](https://github.com/kubernetes/community/blob/master/contributors/design-proposals/scheduling/resources.md#resource-quantities)
