---
title: Guía de migración de un dispositivo físico de Azure Stack Edge Pro con FPGA a Azure Stack Edge Pro con GPU
description: Esta guía contiene instrucciones para migrar las cargas de trabajo de un dispositivo Azure Stack Edge Pro con FPGA a un dispositivo Azure Stack Edge Pro con GPU.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: tutorial
ms.date: 03/11/2021
ms.author: alkohli
ms.openlocfilehash: 5ad065ff74a7cec293b83497d6258bcf62975f26
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108139298"
---
# <a name="migrate-workloads-from-an-azure-stack-edge-pro-fpga-to-an-azure-stack-edge-pro-gpu"></a>Migración de las cargas de trabajo de Azure Stack Edge Pro con FPGA a Azure Stack Edge Pro con GPU

En este artículo se describe cómo migrar las cargas de trabajo y los datos de un dispositivo Azure Stack Edge Pro con FPGA a un dispositivo Azure Stack Edge Pro con GPU. El proceso de migración comienza con una comparación de los dos dispositivos, un plan de migración y una revisión de las consideraciones de migración. El procedimiento de migración proporciona pasos detallados que finalizan con la comprobación y la limpieza del dispositivo.

[!INCLUDE [Azure Stack Edge Pro FPGA end-of-life](../../includes/azure-stack-edge-fpga-eol.md)]

## <a name="about-migration"></a>Información sobre la migración

La migración es el proceso de mover cargas de trabajo y datos de aplicación de una ubicación de almacenamiento a otra. Implica realizar una copia exacta de los datos actuales de una organización de un dispositivo de almacenamiento en otro dispositivo, preferiblemente sin interrumpir ni deshabilitar aplicaciones activas, y luego redirigir toda la actividad de entrada o salida (E/S) al nuevo dispositivo. 

En esta guía de migración se proporciona un tutorial detallado de los pasos necesarios para migrar datos de un dispositivo Azure Stack Edge Pro con FPGA a un dispositivo Azure Stack Edge Pro con GPU. Este documento está destinado a profesionales de tecnologías de la información (TI) y a los trabajadores del conocimiento responsables del funcionamiento, implementación y administración de dispositivos de Azure Stack Edge en el centro de datos.

En este artículo, se hace referencia al dispositivo Azure Stack Edge Pro con FPGA como dispositivo de *origen* y el dispositivo Azure Stack Edge Pro con GPU como dispositivo de *destino*. 

## <a name="comparison-summary"></a>Resumen de comparación

En esta sección se proporciona un resumen comparativo de las funcionalidades de los dispositivos Azure Stack Edge Pro con GPU y Azure Stack Edge Pro con FPGA. El hardware tanto en el dispositivo de origen como en el de destino es prácticamente idéntico. Es posible que solo difiera respecto a la tarjeta de aceleración de hardware y la capacidad de almacenamiento.<!--Please verify: These components MAY, but need not necessarily, differ?-->

|    Capacidad  | Azure Stack Edge Pro con GPU (dispositivo de destino)  | Azure Stack Edge Pro con FPGA (dispositivo de origen)|
|----------------|-----------------------|------------------------|
| Hardware       | Aceleración de hardware: Una o dos GPU NVIDIA T4 <br> El proceso, la memoria, la interfaz de red, la unidad de fuente de alimentación y las especificaciones de cable de alimentación son idénticos al dispositivo con FPGA.  | Aceleración de hardware: Intel Arria 10 FPGA <br> El proceso, la memoria, la interfaz de red, la unidad de fuente de alimentación y las especificaciones de cable de alimentación son idénticos al dispositivo con GPU.          |
| Almacenamiento utilizable | 4,19 TB <br> Después de reservar espacio para la resistencia de la paridad y el uso interno | 12,5 TB <br> Después de reservar espacio para el uso interno |
| Seguridad       | Certificados |                                                     |
| Cargas de trabajo      | Cargas de trabajo de IoT Edge <br> Cargas de trabajo de máquinas virtuales <br> Cargas de trabajo de Kubernetes| Cargas de trabajo de IoT Edge |
| Precios        | [Precios](https://azure.microsoft.com/pricing/details/azure-stack/edge/) | [Precios](https://azure.microsoft.com/pricing/details/azure-stack/edge/)|

## <a name="migration-plan"></a>Plan de migración

Para crear el plan de migración, tenga en cuenta la información siguiente:

- Desarrolle una programación para la migración. 
- Al migrar los datos, es posible que se produzca un tiempo de inactividad. Se recomienda programar la migración durante una ventana de mantenimiento de tiempo de inactividad, ya que el proceso provoca interrupciones. Va a configurar y restaurar las configuraciones en este tiempo de inactividad, tal y como se describe más adelante en este documento.
- Comprenda la duración total del tiempo de inactividad y comuníquelo a todas las partes interesadas.
- Identifique los datos locales que deben migrarse desde el dispositivo de origen. Como medida de precaución, asegúrese de que todos los datos del almacenamiento existente tengan una copia de seguridad reciente. 


## <a name="migration-considerations"></a>Consideraciones de migración 

Antes de continuar con la migración, tenga en cuenta la siguiente información: 

- No se puede activar un dispositivo Azure Stack Edge Pro con GPU en un recurso de Azure Stack Edge Pro con FPGA. Debe crear un nuevo recurso para el dispositivo Azure Stack Edge Pro con GPU, tal como se describe en [Creación de un pedido de Azure Stack Edge Pro con GPU](azure-stack-edge-gpu-deploy-prep.md#create-a-new-resource).
- Los modelos de Machine Learning implementados en el dispositivo de origen que usó FPGA deberán cambiarse en el dispositivo de destino con GPU. Para obtener ayuda con los modelos, póngase en contacto con el servicio de soporte técnico de Microsoft. Los modelos personalizados implementados en el dispositivo de origen que no usaban FPGA (solo usaban CPU) deben funcionar tal cual en el dispositivo de destino (con CPU).
- Los módulos de IoT Edge implementados en el dispositivo de origen pueden requerir cambios antes de que se puedan implementar correctamente en el dispositivo de destino. 
- El dispositivo de origen admite los protocolos NFS 3.0 y 4.1. El dispositivo de destino solo admite el protocolo NFS 3.0.
- El dispositivo de origen admite los protocolos SMB y NFS. El dispositivo de destino admite el almacenamiento mediante el protocolo REST con el uso de cuentas de almacenamiento, además de los protocolos SMB y NFS para recursos compartidos.
- El acceso al recurso compartido en el dispositivo de origen se realiza a través de la dirección IP, mientras que el acceso compartido en el dispositivo de destino se realiza mediante el nombre del dispositivo.

## <a name="migration-steps-at-a-glance"></a>Resumen de los pasos de la migración

En esta tabla se resume el flujo general de la migración, que describe los pasos necesarios para la migración y la ubicación en la que se deben llevar a cabo estos pasos.

| En esta fase | Realice este paso| En este dispositivo |
|---------------|-------------|----------------|
| Preparación del dispositivo de origen*       | 1. Registro de los datos de configuración <br>2. Copia de seguridad de los datos del recurso compartido <br>3. Preparación de cargas de trabajo de IoT Edge| Dispositivo de origen  |
| Preparación del dispositivo de destino*       |1. Creación de un pedido <br>2. Configuración y activación| Dispositivo de destino  |
| Migración de los datos       | 1. Migración de datos desde recursos compartidos <br>2. Reimplementación de las cargas de trabajo de IoT Edge| Dispositivo de destino  |
| Comprobación de datos            |Comprobación de los datos migrados |Dispositivo de destino  |
| Limpieza, devolución              |Borrado de datos y devolución| Dispositivo de origen  |

**Los dispositivos de origen y destino pueden estar preparados en paralelo.*

## <a name="prepare-source-device"></a>Preparación de los datos de origen

La preparación incluye la identificación de los recursos compartidos en la nube de Edge, los recursos compartidos locales de Edge y los módulos de IoT Edge implementados en el dispositivo. 

### <a name="1-record-configuration-data"></a>1. Registro de los datos de configuración

Siga estos pasos en el dispositivo de origen mediante la interfaz de usuario local.

Registre los datos de configuración en el dispositivo de *origen*. Use la [lista de comprobación de implementación](azure-stack-edge-gpu-deploy-checklist.md) para ayudarle a registrar la configuración del dispositivo. Durante la migración, usará esta información para configurar el nuevo dispositivo de destino. 

### <a name="2-back-up-share-data"></a>2. Copia de seguridad de los datos del recurso compartido

Los datos del dispositivo puede ser de uno de los siguientes tipos:

- Datos de recursos compartidos en la nube de Edge
- Datos de recursos compartidos locales

#### <a name="data-in-edge-cloud-shares"></a>Datos de recursos compartidos en la nube de Edge

Los recursos compartidos en la nube de Edge organizan los datos en capas desde el dispositivo a Azure. Siga estos pasos en el dispositivo de *origen* mediante Azure Portal. 

- Cree una lista de todos los recursos compartidos en la nube de Edge y de los usuarios que tenga en el dispositivo de origen.
- Haga una lista de todas las programaciones de ancho de banda que tenga. Volverá a crear estas programaciones de ancho de banda en el dispositivo de destino.
- En función del ancho de banda de red disponible, configure las programaciones de ancho de banda en el dispositivo para maximizar los datos en capas en la nube. Esta acción minimiza los datos locales en el dispositivo.
- Asegúrese de que los recursos compartidos estén totalmente organizados en capas en la nube. Para confirmarlo, compruebe el estado del recurso compartido en Azure Portal.  

#### <a name="data-in-edge-local-shares"></a>Datos de los recursos compartidos locales de Edge

Los datos de los recursos compartidos locales de Edge permanecen en el dispositivo. Siga estos pasos en el dispositivo de *origen* mediante Azure Portal. 

- Prepare una lista de los recursos compartidos locales de Edge que tenga en el dispositivo.
- Dado que se trata de una migración única de los datos, cree una copia de los datos del recurso compartido local de Edge en otro servidor local. Puede usar herramientas de copia como `robocopy` (SMB) o `rsync` (NFS) para copiar los datos. Opcionalmente, es posible que ya haya implementado una solución de protección de datos de terceros para hacer una copia de seguridad de los datos en los recursos compartidos locales. Las siguientes soluciones de terceros se admiten para su uso con dispositivos Azure Stack Edge Pro con FPGA:

    | Software de terceros           | Referencia a la solución                               |
    |--------------------------------|---------------------------------------------------------|
    | Cohesity                       | [https://www.cohesity.com/solution/cloud/azure/](https://www.cohesity.com/solution/cloud/azure/) <br> Para obtener más información, póngase en contacto con Cohesity.          |
    | Commvault                      | [https://www.commvault.com/azure](https://www.commvault.com/azure) <br> Para obtener más información, póngase en contacto con Commvault.          |
    | Veritas                        | [http://veritas.com/azure](http://veritas.com/azure) <br> Para obtener más información, póngase en contacto con Veritas.   |
    | Veeam                          | [https://www.veeam.com/kb4041](https://www.veeam.com/kb4041) <br> Para más información, póngase en contacto con Veeam. |


### <a name="3-prepare-iot-edge-workloads"></a>3. Preparación de cargas de trabajo de IoT Edge

- Si ha implementado módulos de IoT Edge y está usando la aceleración de FPGA, puede que tenga que modificar los módulos antes de que se ejecuten en el dispositivo GPU. Siga las instrucciones para [modificar los módulos de IoT Edge](azure-stack-edge-gpu-modify-fpga-modules-gpu.md). 

<!--- If you have deployed IoT Edge workloads, the configuration data is shared on a share on the device. Back up the data in these shares.-->


## <a name="prepare-target-device"></a>Preparación del dispositivo de destino

### <a name="1-create-new-order"></a>1. Creación de un nuevo pedido

Tiene que crear un nuevo pedido (y un recurso nuevo) para el dispositivo de *destino*. El dispositivo de destino se debe activar en el recurso de GPU y no en el de FPGA.

Para realizar la solicitud, [cree un nuevo recurso de Azure Stack Edge](azure-stack-edge-gpu-deploy-prep.md#create-a-new-resource) en Azure Portal.


### <a name="2-set-up-activate"></a>2. Configuración y activación

Debe configurar y activar el dispositivo de *destino* en el nuevo recurso que creó anteriormente. 

Siga estos pasos para configurar el dispositivo de *destino* mediante Azure Portal:

1. Recopile la información necesaria que se indica en [Lista de comprobación para implementación](azure-stack-edge-gpu-deploy-checklist.md). Puede usar la información que guardó de la configuración del dispositivo de origen. 
1. [Desempaquete el dispositivo](azure-stack-edge-gpu-deploy-install.md#unpack-the-device), [móntelo en un bastidor](azure-stack-edge-gpu-deploy-install.md#rack-the-device) y [cabléelo](azure-stack-edge-gpu-deploy-install.md#cable-the-device). 
1. [Conéctese a la interfaz de usuario local del dispositivo](azure-stack-edge-gpu-deploy-connect.md).
1. Configure la red con un conjunto diferente de direcciones IP (si usa IP estáticas) que las que usó para el dispositivo anterior. Consulte cómo [definir la configuración de red](azure-stack-edge-gpu-deploy-configure-network-compute-web-proxy.md).
1. Asigne el mismo nombre de dispositivo que el dispositivo anterior y proporcione un dominio de DNS. Consulte cómo [configurar el dispositivo](azure-stack-edge-gpu-deploy-set-up-device-update-time.md).
1. Configure los certificados en el nuevo dispositivo. Consulte cómo [configurar certificados](azure-stack-edge-gpu-deploy-configure-certificates.md).
1. Obtenga la clave de activación desde Azure Portal y active el nuevo dispositivo. Consulte cómo [activar el dispositivo](azure-stack-edge-gpu-deploy-activate.md).

Ahora está preparado para restaurar los datos del recurso compartido e implementar las cargas de trabajo que se estaban ejecutando en el dispositivo anterior.

## <a name="migrate-data"></a>Migración de los datos

Ahora se copiarán los datos del dispositivo de origen en los recursos compartidos en la nube de Edge y los recursos compartidos locales de Edge en el dispositivo de *destino*.

### <a name="1-from-edge-cloud-shares"></a>1. Desde los recursos compartidos en la nube de Edge

Siga estos pasos para sincronizar los datos en los recursos compartidos en la nube de Edge en el dispositivo de destino:

1. [Agregue recursos compartidos](./azure-stack-edge-gpu-manage-shares.md#add-a-share) correspondientes a los nombres de los recursos compartidos creados en el dispositivo de origen. Al crear recursos compartidos, asegúrese de que la opción **Seleccionar el contenedor de blobs** está establecida en **Utilizar existente** y luego seleccione el contenedor que se usó con el dispositivo anterior.
1. [Agregue usuarios](./azure-stack-edge-gpu-manage-users.md#add-a-user) que tenían acceso al dispositivo anterior.
1. [Actualice los datos del recurso compartido](./azure-stack-edge-gpu-manage-shares.md#refresh-shares) desde Azure. Al actualizar el recurso compartido, se desplegarán todos los datos en la nube desde el contenedor existente hasta los recursos compartidos.
1. Vuelva a crear las programaciones de ancho de banda para asociarlas a sus recursos compartidos. Consulte [Adición de una programación de ancho de banda](./azure-stack-edge-gpu-manage-bandwidth-schedules.md#add-a-schedule) para obtener pasos detallados.


### <a name="2-from-edge-local-shares"></a>2. Desde los recursos compartidos locales de Edge

Es posible que haya implementado una solución de copia de seguridad de otro fabricante para proteger los datos de recursos compartidos locales de las cargas de trabajo de IoT. Ahora tendrá que restaurar los datos.

Una vez configurado completamente el dispositivo de sustitución, habilítelo para el almacenamiento local. 

Siga estos pasos para recuperar los datos de recursos compartidos locales:

1. [Configure el proceso en el dispositivo](azure-stack-edge-gpu-deploy-configure-compute.md).
1. Agregue todos los recursos compartidos locales en el dispositivo de destino. Consulte los pasos detallados en [Adición de un recurso compartido local](azure-stack-edge-gpu-manage-shares.md#add-a-local-share).
1. El acceso a los recursos compartidos de SMB en el dispositivo de origen usará las direcciones IP, mientras que en el dispositivo de destino usará el nombre del dispositivo. Consulte [Conexión a un recurso compartido de SMB en Azure Stack Edge Pro con GPU](./azure-stack-edge-gpu-deploy-add-shares.md#connect-to-an-smb-share). Para conectarse a los recursos compartidos de NFS en el dispositivo de destino, deberá usar las nuevas direcciones IP asociadas con el dispositivo. Consulte [Conexión a un recurso compartido de NFS en Azure Stack Edge Pro con GPU](./azure-stack-edge-gpu-deploy-add-shares.md#connect-to-an-nfs-share). 

    Si ha copiado los datos del recurso compartido en un servidor intermedio mediante SMB o NFS, puede copiar estos datos en los recursos compartidos del dispositivo de destino. Si tanto el dispositivo de origen como el de destino están *en línea*, también puede copiar los datos directamente desde el dispositivo de origen.

    Si ha usado un software de terceros para hacer una copia de seguridad de los datos de los recursos compartidos locales, deberá ejecutar el procedimiento de recuperación proporcionado por la solución de protección de datos de su elección. Consulte las referencias en la tabla siguiente.

    | Software de terceros           | Referencia a la solución                               |
    |--------------------------------|---------------------------------------------------------|
    | Cohesity                       | [https://www.cohesity.com/solution/cloud/azure/](https://www.cohesity.com/solution/cloud/azure/) <br> Para obtener más información, póngase en contacto con Cohesity.          |
    | Commvault                      | [https://www.commvault.com/azure](https://www.commvault.com/azure) <br> Para obtener más información, póngase en contacto con Commvault. |
    | Veritas                        | [http://veritas.com/azure](http://veritas.com/azure) <br> Para obtener más información, póngase en contacto con Veritas.   |
    | Veeam                          | [https://www.veeam.com/kb4041](https://www.veeam.com/kb4041) <br> Para más información, póngase en contacto con Veeam. |

### <a name="3-redeploy-iot-edge-workloads"></a>3. Reimplementación de las cargas de trabajo de IoT Edge

Una vez preparados los módulos de IoT Edge, deberá implementar las cargas de trabajo de IoT Edge en el dispositivo de destino. Si se encuentra con algún error en la implementación de módulos de IoT Edge, consulte:

- [Incidencias habituales y soluciones para Azure IoT Edge](../iot-edge/troubleshoot-common-errors.md). 
- [Errores del entorno de ejecución de Azure IoT Edge](azure-stack-edge-gpu-troubleshoot.md#troubleshoot-iot-edge-errors).

## <a name="verify-data"></a>Comprobación de datos

Después de la migración, compruebe que todos los datos se hayan migrado y que las cargas de trabajo se hayan implementado en el dispositivo de destino.

## <a name="erase-data-return"></a>Borrado de datos y devolución

Una vez completada la migración de datos, borre los datos locales y devuelva el dispositivo de origen. Siga los pasos descritos en [Devolución del dispositivo Azure Stack Edge Pro](azure-stack-edge-return-device.md).


## <a name="next-steps"></a>Pasos siguientes

[Más información sobre cómo implementar cargas de trabajo de IoT Edge en el dispositivo Azure Stack Edge Pro con GPU](azure-stack-edge-gpu-deploy-compute-module-simple.md)