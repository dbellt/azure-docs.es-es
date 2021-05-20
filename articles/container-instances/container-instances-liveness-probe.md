---
title: Configuración del sondeo de ejecución en la instancia de contenedor
description: Información sobre cómo configurar los sondeos de ejecución para reiniciar contenedores incorrectos en Azure Container Instances
ms.topic: article
ms.date: 07/02/2020
ms.openlocfilehash: 0d88133692fbfad4c5e81eefa57c69a843448222
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108771566"
---
# <a name="configure-liveness-probes"></a>Configuración de sondeos de ejecución

Las aplicaciones en contenedores pueden ejecutarse durante largos períodos de tiempo y provocar la interrupción de estados que quizás deban repararse con un reinicio del contenedor. Azure Container Instances admite sondeos de ejecución para que pueda configurar los contenedores dentro del grupo de contenedores para que se reinicien si una funcionalidad crítica no funciona. El sondeo de ejecución se comporta como un[sondeo de ejecución de Kubernetes](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/).

En este artículo se explica cómo implementar un grupo de contenedores que incluya un sondeo de ejecución y se muestra el reinicio automático de un contenedor incorrecto simulado.

Azure Container Instances también admite [sondeos de preparación](container-instances-readiness-probe.md), que se pueden configurar para asegurarse de que el tráfico llega a un contenedor solo cuando esté listo para ello.

> [!NOTE]
> Actualmente no se puede usar un sondeo de ejecución en un grupo de contenedores que se implementa en una red virtual.

## <a name="yaml-deployment"></a>Implementación de YAML

Cree un archivo `liveness-probe.yaml` con el siguiente fragmento de código. Este archivo define un grupo de contenedores que consta de un contenedor NGNIX que, con el tiempo, se vuelve incorrecto.

```yaml
apiVersion: 2019-12-01
location: eastus
name: livenesstest
properties:
  containers:
  - name: mycontainer
    properties:
      image: nginx
      command:
        - "/bin/sh"
        - "-c"
        - "touch /tmp/healthy; sleep 30; rm -rf /tmp/healthy; sleep 600"
      ports: []
      resources:
        requests:
          cpu: 1.0
          memoryInGB: 1.5
      livenessProbe:
        exec:
            command:
                - "cat"
                - "/tmp/healthy"
        periodSeconds: 5
  osType: Linux
  restartPolicy: Always
tags: null
type: Microsoft.ContainerInstance/containerGroups
```

Ejecute el siguiente comando para implementar este grupo de contenedores con la configuración de YAML anterior:

```azurecli-interactive
az container create --resource-group myResourceGroup --name livenesstest -f liveness-probe.yaml
```

### <a name="start-command"></a>Comando de inicio

La implementación incluye una propiedad `command` que define un comando de inicio que se ejecuta cuando el contenedor comienza a ejecutarse por primera vez. Esta propiedad acepta una matriz de cadenas. Este comando simula que el contenedor entra en un estado incorrecto.

En primer lugar, inicia una sesión de Bash y crea un archivo denominado `healthy` dentro del directorio `/tmp`. Luego se suspenderá durante 30 segundos antes de eliminar el archivo y entrará en modo de suspensión durante 10 minutos:

```bash
/bin/sh -c "touch /tmp/healthy; sleep 30; rm -rf /tmp/healthy; sleep 600"
```

### <a name="liveness-command"></a>Comando de ejecución

Esta implementación define un `livenessProbe` que admite un comando de ejecución `exec` que actúa como comprobación de la ejecución. Si este comando finaliza con un valor distinto de cero, el contenedor se eliminará y se reiniciará para indicar que el archivo `healthy` no se encontró. Si este comando finaliza correctamente con el código de salida 0, no se realizará ninguna acción.

La propiedad `periodSeconds` designa que el comando de ejecución debería ejecutarse cada 5 segundos.

## <a name="verify-liveness-output"></a>Comprobación de la salida de ejecución

Durante los primeros 30 segundos, el archivo `healthy` creado por el comando de inicio existe. Cuando el comando de ejecución comprueba la existencia del archivo `healthy`, el código de estado devuelve un 0, que indica que la operación se realizó correctamente y que no se producirá ningún reinicio.

Después de 30 segundos, el comando `cat /tmp/healthy` comenzará a generar errores, lo que dará lugar a eventos incorrectos y provocará su finalización.

Estos eventos se pueden ver desde Azure Portal o la CLI de Azure.

![Evento incorrecto del portal:][portal-unhealthy]

Al ver los eventos en Azure Portal, se desencadenarán eventos de tipo `Unhealthy` al producirse un error del comando de ejecución. El evento posterior será del tipo `Killing`, lo que significa la eliminación de un contenedor y que puede comenzar un reinicio. El recuento de reinicio del contenedor se incrementa cada vez que se produce este evento.

Los reinicios se completan en contexto para recursos como las direcciones IP públicas y se conservará el contenido específico de nodo.

![Contador de reinicios del portal][portal-restart]

Si el sondeo de ejecución sigue generando errores y desencadena demasiados reinicios, el contenedor provocará un retraso de retroceso exponencial.

## <a name="liveness-probes-and-restart-policies"></a>Sondeos de ejecución y directivas de reinicio

Las directivas de reinicio reemplazan el comportamiento de reinicio que desencadenan los sondeos de ejecución. Por ejemplo, si establece una directiva `restartPolicy = Never` *y* un sondeo de ejecución, el grupo de contenedores no se reiniciará si se produce una comprobación de ejecución errónea. En su lugar, el grupo de contenedores se adherirá a la directiva de reinicio del grupo de contenedores de `Never`.

## <a name="next-steps"></a>Pasos siguientes

Los escenarios basados en tareas pueden requerir un sondeo de ejecución para habilitar los reinicios automáticos si una función de requisito previo no funciona correctamente. Para más información sobre cómo ejecutar contenedores basados en tareas, consulte [Ejecución de tareas en contenedores en Azure Container Instances](container-instances-restart-policy.md).

<!-- IMAGES -->
[portal-unhealthy]: ./media/container-instances-liveness-probe/unhealthy-killing.png
[portal-restart]: ./media/container-instances-liveness-probe/portal-restart.png
