---
title: Administración de procesos en Azure Stack Edge Pro con GPU | Microsoft Docs
description: En este artículo se explica cómo administrar la configuración de procesos en Edge, como el desencadenador, los módulos, y ver y quitar la configuración por medio de Azure Portal en Azure Stack Edge Pro con GPU.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: how-to
ms.date: 06/04/2021
ms.author: alkohli
ms.openlocfilehash: eac9154d4308700eaa6ea8faf0bf94fabc80bce4
ms.sourcegitcommit: e39ad7e8db27c97c8fb0d6afa322d4d135fd2066
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111982527"
---
# <a name="manage-compute-on-your-azure-stack-edge-pro-gpu"></a>Administración de procesos en Azure Stack Edge Pro con GPU

[!INCLUDE [applies-to-GPU-and-pro-r-and-mini-r-skus](../../includes/azure-stack-edge-applies-to-gpu-pro-r-mini-r-sku.md)]

En este artículo se describe cómo administrar el proceso a través del servicio IoT Edge en un dispositivo Azure Stack Edge Pro con GPU. Puede administrar el proceso a través de Azure Portal o mediante la interfaz de usuario web local. Use Azure Portal para administrar los módulos, los desencadenadores y la configuración de IoT Edge, y la interfaz de usuario web local para administrar la configuración de la red.



## <a name="manage-triggers"></a>Administración de desencadenadores

Los eventos son cosas que ocurren dentro del entorno de nube o en el dispositivo sobre las cuales quizás quiera tomar medidas. Por ejemplo, cuando se crea un archivo en un recurso compartido, hablamos de un evento. Los desencadenadores provocan los eventos. En Azure Stack Edge Pro, los desencadenadores pueden surgir en respuesta a eventos de archivos o según una programación.

- **Archivo**: estos desencadenadores surgen en respuesta a eventos de archivo como la creación de un archivo o su modificación.
- **Scheduled**: estos desencadenadores surgen en respuesta a una programación que puede definir con una fecha de inicio, una hora de inicio y el intervalo de repetición.


### <a name="add-a-trigger"></a>Incorporación de un desencadenador

Para crear un desencadenador, siga estos pasos en Azure Portal.

1. En Azure Portal, vaya al recurso de Azure Stack Edge y luego a **IoT Edge**. Vaya a **Desencadenadores** y seleccione **+Agregar desencadenador** en la barra de comandos.

    ![Selección de Agregar desencadenador](media/azure-stack-edge-gpu-manage-compute/add-trigger-1-m.png)

2. En la hoja **Agregar desencadenador**, escriba un nombre único para el desencadenador.
    
    <!--Trigger names can only contain numbers, lowercase letters, and hyphens. The share name must be between 3 and 63 characters long and begin with a letter or a number. Each hyphen must be preceded and followed by a non-hyphen character.-->

3. Seleccione un **Tipo** para el desencadenador. Elija **Archivo** cuando el desencadenador sea la respuesta a un evento de archivo. Seleccione **Scheduled** (Programado) cuando quiera que el desencadenador se inicie a una hora determinada y se ejecuta según un intervalo de repetición especificado. Según la selección que haga, verá un conjunto distinto de opciones.

    - **File trigger** (Desencadenador de archivo): elija un recurso compartido montado en la lista desplegable. Cuando se active un evento de archivo en este recurso compartido, el desencadenador podría invocar una función de Azure.

        ![Incorporación de recurso compartido de SMB](media/azure-stack-edge-gpu-manage-compute/add-file-trigger.png)

    - **Scheduled trigger** (Desencadenador programado): especifique la fecha y hora de inicio y el intervalo de repetición en horas, minutos o segundos. Escriba también el nombre para un tema. Un tema le brindará la flexibilidad que necesita para enrutar el desencadenador a un módulo implementado en el dispositivo.

        Una cadena de ruta de ejemplo es: `"route3&quot;: &quot;FROM /* WHERE topic = 'topicname' INTO BrokeredEndpoint(&quot;modules/modulename/inputs/input1")"`.

        ![Incorporación de un recurso compartido NFS](media/azure-stack-edge-gpu-manage-compute/add-scheduled-trigger.png)

4. Seleccione **Agregar** para crear el desencadenador. Una notificación muestra que la creación del desencadenador está en curso. Una vez creado el desencadenador, la hoja se actualiza para reflejar el desencadenador nuevo.
 
    ![Lista de desencadenadores actualizada](media/azure-stack-edge-gpu-manage-compute/add-trigger-2.png)

### <a name="delete-a-trigger"></a>Eliminación de un desencadenador

Para eliminar un desencadenador, siga estos pasos en Azure Portal.

1. En la lista de desencadenadores, seleccione el desencadenador que quiere eliminar.

    ![Seleccionar un desencadenador](media/azure-stack-edge-gpu-manage-compute/delete-trigger-1.png)

2. Haga clic con el botón derecho y seleccione **Eliminar**.

    ![Seleccionar Eliminar](media/azure-stack-edge-gpu-manage-compute/delete-trigger-2.png)

3. Cuando se le pida confirmación, haga clic en **Sí**.

    ![Confirmar eliminación](media/azure-stack-edge-gpu-manage-compute/delete-trigger-3.png)

La lista de desencadenadores se actualiza para reflejar la eliminación.

## <a name="manage-iot-edge-configuration"></a>Administración de la configuración de IoT Edge

Use Azure Portal para ver la configuración de proceso, quitar una configuración de proceso ya existente o actualizarla con el fin de sincronizar las claves de acceso del dispositivo IoT y el dispositivo IoT Edge de Azure Stack Edge Pro.

### <a name="view-iot-edge-configuration"></a>Visualización de la configuración de IoT Edge

Para ver la configuración de IoT Edge para el dispositivo, siga estos pasos en Azure Portal.

1. En Azure Portal, vaya al recurso de Azure Stack Edge y luego a **IoT Edge**. Después de habilitar el servicio IoT Edge en el dispositivo, la página Información general indica que el servicio IoT Edge se está ejecutando correctamente.

    ![Seleccionar Ver proceso](media/azure-stack-edge-gpu-manage-compute/view-compute-1.png)

2. Vaya a **Propiedades** para ver la configuración de IoT Edge en el dispositivo. Cuando configuró el proceso, creó un recurso de IoT Hub. En ese recurso de IoT Hub, se configuró un dispositivo IoT y un dispositivo IoT Edge. Solo los módulos de Linux se admiten para ejecución en el dispositivo IoT Edge.

    ![Ver configuración](media/azure-stack-edge-gpu-manage-compute/view-compute-2.png)


### <a name="remove-iot-edge-service"></a>Eliminación del servicio IoT Edge

Para quitar la configuración de IoT Edge existente del dispositivo, siga estos pasos en Azure Portal.

1. En Azure Portal, vaya al recurso de Azure Stack Edge y luego a **IoT Edge**. Vaya a **Información general** y seleccione **Quitar** en la barra de comandos.

    ![Seleccionar Quitar proceso](media/azure-stack-edge-gpu-manage-compute/remove-compute-1.png)

2. Si quita el servicio IoT Edge, la acción es irreversible y no se puede deshacer. También se eliminarán los módulos y los desencadenadores que creó. Tendrá que volver a configurar el dispositivo en caso de que necesite volver a usar IoT Edge. Cuando se le pida confirmación, seleccione **Aceptar**.

    ![Seleccionar Eliminar proceso 2](media/azure-stack-edge-gpu-manage-compute/remove-compute-2.png)

### <a name="sync-up-iot-device-and-iot-edge-device-access-keys"></a>Sincronización de las claves de acceso de dispositivo IoT y dispositivo IoT Edge

Cuando se configura el proceso en Azure Stack Edge Pro, se crea un dispositivo IoT y un dispositivo IoT Edge. A estos dispositivos se les asigna automáticamente claves de acceso simétricas. Como procedimiento recomendado de seguridad, estas claves se rotan de manera periódica a través del servicio IoT Hub.

Para rotar estas claves, puede ir al servicio IoT Hub que creó y seleccionar el dispositivo IoT o el dispositivo IoT Edge. Cada dispositivo tiene una clave de acceso primaria y una clave de acceso secundaria. Asigne la clave de acceso primaria a la clave de acceso secundario y luego vuelva a generar la clave de acceso primaria.

Si las claves del dispositivo IoT y del dispositivo IoT Edge se han rotado, deberá actualizar la configuración en Azure Stack Edge Pro para tener las claves de acceso más recientes. La sincronización permite que el dispositivo obtenga las claves más recientes del dispositivo IoT y del dispositivo IoT Edge. Azure Stack Edge Pro solo usa las claves de acceso primarias.

Para sincronizar las claves de acceso del dispositivo, siga estos pasos en Azure Portal.

1. En Azure Portal, vaya al recurso de Azure Stack Edge y luego a **Proceso de IoT Edge**. Vaya a **Información general** y seleccione **Actualizar configuración** en la barra de comandos.

    ![Seleccionar Actualizar configuración](media/azure-stack-edge-gpu-manage-compute/refresh-configuration-1.png)

2. Seleccione **Sí** cuando se pida confirmación.

    ![Seleccionar Sí cuando se pida confirmación](media/azure-stack-edge-gpu-manage-compute/refresh-configuration-2.png)

3. Salga del cuadro de diálogo cuando haya finalizado la sincronización.

## <a name="change-external-service-ips-for-containers"></a>Cambio de IP de servicio externo para contenedores

Las direcciones IP del servicio externo de Kubernetes se usan para llegar a los servicios que se exponen fuera del clúster de Kubernetes. Después de activar el dispositivo, puede establecer o modificar las direcciones IP del servicio externo para las cargas de trabajo en contenedor del dispositivo mediante el acceso a la interfaz de usuario local.


1. En la interfaz de usuario local del dispositivo, vaya a **Proceso**.
1. Seleccione el puerto cuya red está configurada para el proceso. En la hoja que se abre, especifique (nuevo) o modifique (si existen) las direcciones IP del servicio externo de Kubernetes. Estas direcciones IP se usan para cualquier servicio que deba exponerse fuera del clúster de Kubernetes. 
    - Necesita un mínimo de una dirección IP de servicio para el servicio `edgehub` que se ejecuta en el dispositivo y que usan los módulos de IoT Edge. 
    - Necesitará una dirección IP para cada módulo o contenedor de IoT Edge adicional que quiera implementar. 
    - Se trata de direcciones IP estáticas y contiguas.

    ![Cambio de las direcciones IP de servicio de Kubernetes](media/azure-stack-edge-gpu-manage-compute/change-service-ips-1.png)

1. Seleccione **Aplicar**. Una vez aplicadas las direcciones IP, no es necesario reiniciar ni volver a arrancar el dispositivo. Las nuevas direcciones IP se aplican inmediatamente.


## <a name="next-steps"></a>Pasos siguientes

- Aprenda a [solucionar problemas de IoT Edge en el dispositivo GPU de Azure Stack Edge Pro](azure-stack-edge-gpu-troubleshoot-iot-edge.md).
