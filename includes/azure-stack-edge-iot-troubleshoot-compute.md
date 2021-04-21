---
author: v-dalc
ms.service: databox
ms.author: alkohli
ms.topic: include
ms.date: 03/23/2021
ms.openlocfilehash: 0d912d0ac3f0fcf4c52116e67909038a1973304b
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/30/2021
ms.locfileid: "105105151"
---
Uso de las respuestas del entorno de ejecución del agente de IoT Edge para solucionar los problemas relacionados con el proceso. A continuación, se ofrece una lista de las posibles respuestas:

* 200 - CORRECTO
* 400 - La configuración de implementación tiene un formato incorrecto o no es válida.
* 417 - El dispositivo no tiene un conjunto de configuración de implementación.
* 412 - La versión del esquema de la configuración de implementación no es válida.
* 406 - El dispositivo de IoT Edge está sin conexión o no envía informes de estado.
* 500 - Error en el entorno en tiempo de ejecución de Azure IoT Edge.

Para más información, consulte [Agente de IoT Edge](../articles/iot-edge/iot-edge-runtime.md?preserve-view=true&view=iotedge-2018-06#iot-edge-agent).

El siguiente error está relacionado con el servicio de IoT Edge en el dispositivo Azure Stack Edge Pro.

### <a name="compute-modules-have-unknown-status-and-cant-be-used"></a>Los módulos de proceso tienen el estado Desconocido y no se pueden usar

#### <a name="error-description"></a>Descripción del error

Todos los módulos del dispositivo muestran el estado Desconocido y no se pueden usar. El estado Desconocido persiste tras un reinicio.<!--Original Support ticket relates to trying to deploy a container app on a Hub. Based on the work item, I assume the error description should not be that specific, and that the error applies to Azure Stack Edge Devices, which is the focus of this troubleshooting.-->

#### <a name="suggested-solution"></a>Solución propuesta

Elimine el servicio de IoT Edge y, a continuación, vuelva a implementar los módulos. Para obtener más información, consulte [Eliminación del servicio de IoT Edge](../articles/databox-online/azure-stack-edge-gpu-manage-compute.md#remove-iot-edge-service).


### <a name="modules-show-as-running-but-are-not-working"></a>Los módulos aparecen con el estado en ejecución, pero no funcionan

#### <a name="error-description"></a>Descripción del error

El estado de tiempo de ejecución del módulo indica que está en ejecución, pero no se ven los resultados esperados. 

Esta condición podría deberse a un problema con la configuración de la ruta del módulo que no funciona o que `edgehub` no enruta los mensajes según lo esperado. Puede comprobar los registros `edgehub`. Si ve que hay errores, como que no se puede conectar al servicio IoT Hub, el motivo más común son los problemas de conectividad. Los problemas de conectividad pueden deberse a que el puerto AMPQ que el servicio IoT Hub usa como puerto predeterminado para la comunicación está bloqueado o el servidor proxy web está bloqueando estos mensajes.

#### <a name="suggested-solution"></a>Solución propuesta

Siga estos pasos:
1. Para resolver el error, vaya al recurso IoT Hub del dispositivo y, a continuación, seleccione el dispositivo perimetral. 
1. Vaya a **Establecer módulos > Configuración de tiempo de ejecución**. 
1. Agregue la variable de entorno `Upstream protocol` y asígnele el valor `AMQPWS`. Los mensajes configurados en este caso se envían a través de WebSockets por el puerto 443.

### <a name="modules-show-as-running-but-do-not-have-an-ip-assigned"></a>Los módulos aparecen con el estado en ejecución, pero no tienen una dirección IP asignada

#### <a name="error-description"></a>Descripción del error

El estado de tiempo de ejecución del módulo indica que está en ejecución, pero la aplicación en contenedor no tiene una dirección IP asignada. 

Esta condición se debe a que el intervalo de direcciones IP que ha proporcionado para direcciones IP del servicio externo de Kubernetes no es suficiente. Debe ampliar este intervalo para asegurarse de que abarca todos los contenedores o máquinas virtuales que ha implementado.

#### <a name="suggested-solution"></a>Solución propuesta

En la interfaz de usuario web local del dispositivo, realice los pasos siguientes:
1. Vaya a la página **Proceso**. Seleccione el puerto para el que ha habilitado la red de proceso. 
1. Especifique un intervalo estático y contiguo de direcciones IP para las **direcciones IP del servicio externo de Kubernetes**. Necesita una dirección IP para el servicio `edgehub`. Además, necesita una dirección IP para cada módulo de IoT Edge y para cada máquina virtual que vaya a implementar. 
1. Seleccione **Aplicar**. El intervalo de IP cambiado debe surtir efecto de inmediato.

Para obtener más información, consulte [Cambio de IP de servicio externo para contenedores](../articles/databox-online/azure-stack-edge-gpu-manage-compute.md#change-external-service-ips-for-containers).

### <a name="configure-static-ips-for-iot-edge-modules"></a>Configuración de direcciones IP estáticas para módulos IoT Edge

#### <a name="problem-description"></a>Descripción del problema

Kubernetes asigna direcciones IP dinámicas a cada módulo IoT Edge en el dispositivo Azure Stack Edge Pro con GPU. Se necesita un método para configurar direcciones IP estáticas para los módulos.

#### <a name="suggested-solution"></a>Solución propuesta

Puede especificar direcciones IP fijas para los módulos IoT Edge a través de la sección K8s-experimental, como se describe a continuación: 

```yaml
{
  "k8s-experimental": {
    "serviceOptions" : {
      "loadBalancerIP" : "100.23.201.78",
      "type" : "LoadBalancer"
    }
  }
}
```
### <a name="expose-kubernetes-service-as-cluster-ip-service-for-internal-communication"></a>Exposición de Kubernetes Service como servicio IP de clúster para la comunicación interna

#### <a name="problem-description"></a>Descripción del problema

De manera predeterminada, el tipo de servicio de IoT es de tipo equilibrador de carga y se le asignan direcciones IP orientadas externamente. Es posible que no quiera una dirección IP externa para la aplicación. Es posible que tenga que exponer los pods dentro del clúster de Kubernetes para el acceso como otros pods y no como un servicio de equilibrador de carga expuesto externamente. 

#### <a name="suggested-solution"></a>Solución propuesta

Puede usar las opciones de creación a través de la sección K8s-experimental. La siguiente opción de servicio debe funcionar con enlaces de puerto.

```yaml
{
"k8s-experimental": {
  "serviceOptions" : {
    "type" : "ClusterIP"
    }
  }
}
```