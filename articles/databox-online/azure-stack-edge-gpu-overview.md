---
title: Introducción a Microsoft Azure Stack Edge Pro con GPU | Microsoft Docs
description: Se describe Azure Stack Edge Pro con GPU, una solución de almacenamiento que usa un dispositivo físico para la transferencia a través de la red a Azure.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: overview
ms.date: 04/19/2021
ms.author: alkohli
ms.openlocfilehash: b56b79c29838c00fd79a0ed91a1c3ff0686c56ce
ms.sourcegitcommit: eda26a142f1d3b5a9253176e16b5cbaefe3e31b3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2021
ms.locfileid: "109737985"
---
# <a name="what-is-azure-stack-edge-pro-with-gpu"></a>¿Qué es Azure Stack Edge Pro con GPU?

Azure Stack Edge Pro con GPU es un dispositivo informático perimetral habilitado para la inteligencia artificial que cuenta con funcionalidades de transferencia de datos de red. En este artículo se proporciona información general acerca de la solución de Azure Stack Edge Pro, sus ventajas, sus funcionalidades clave y los escenarios en los que puede implementar este dispositivo.

Azure Stack Edge Pro con GPU es una solución de hardware como servicio. Microsoft envía un dispositivo administrado en la nube que actúa como puerta de enlace de almacenamiento de red y tiene una unidad de procesamiento gráfico (GPU) integrada que habilita la inferencia de IA acelerada. 

## <a name="use-cases"></a>Casos de uso

Estos son los distintos escenarios en los que se puede usar Azure Stack Edge Pro GPU para la inferencia rápida de Machine Learning (ML) perimetral y el preprocesamiento de datos antes de enviarlos a Azure.

- **Inferencia con Azure Machine Learning**: con Azure Stack Edge Pro GPU, puede ejecutar modelos de ML para obtener resultados rápidos sobre los que se puede actuar antes de que los datos se envíen a la nube. El conjunto de datos completo se puede transferir opcionalmente para continuar entrenando y mejorando los modelos de Machine Learning. Para más información sobre cómo usar los modelos acelerados por hardware de Azure ML en el dispositivo de Azure Stack Edge Pro GPU, consulte [Implementación de modelos acelerados por hardware de Azure ML en Azure Stack Edge Pro GPU](../machine-learning/how-to-deploy-fpga-web-service.md#deploy-to-a-local-edge-server).

- **Preprocesamiento de datos**: transforme los datos antes de enviarlos a Azure a través de opciones de proceso como cargas de trabajo en contenedores y máquinas virtuales para crear un conjunto de datos más útil. El preprocesamiento se puede usar para: 

    - Agregar datos.
    - Modificar datos, por ejemplo, para quitar los datos personales.
    - Subconjuntos de datos para optimizar el almacenamiento y el ancho de banda, o para un análisis más exhaustivo.
    - Analizar y reaccionar a los eventos de IoT. 

- **Transferencia de datos a través de la red a Azure**: use Azure Stack Edge Pro GPU para transferir datos a Azure de forma rápida y sencilla a fin de poder realizar más procesos y análisis, o para fines de archivado. 

## <a name="key-capabilities"></a>Principales capacidades

Azure Stack Edge Pro GPU presenta las siguientes funcionalidades:

|Capacidad |Descripción  |
|---------|---------|
|Inferencia de IA acelerada| Habilitado por la GPU integrada (una o dos, en función del modelo).|
|Proceso perimetral      |Admite cargas de trabajo en contenedores y máquinas virtuales para permitir el análisis, procesamiento y filtrado de datos. |
|Acceso a datos     | Ofrece acceso directo a los datos de Azure Storage Blob y Azure Files mediante las API en la nube para un procesamiento de datos adicional en la nube. La memoria caché local del dispositivo se usa para agilizar el acceso a los archivos usados más recientemente.|
|Administración en la nube     |El dispositivo y el servicio se administran con Azure Portal.  |
|Carga sin conexión     | El modo sin conexión es compatible con escenarios de carga sin conexión.|
|Protocolos de transferencia de archivos compatibles      | Compatibilidad con los protocolos estándar SMB, NFS y REST para la ingesta de datos. <br> Para más información sobre las versiones compatibles, consulte [Requisitos del sistema de Azure Stack Edge Pro GPU](azure-stack-edge-system-requirements.md).|
|Actualización de datos     | Capacidad para actualizar los archivos locales con la versión más reciente de la nube.|
|Cifrado    | Compatibilidad con BitLocker para, localmente, cifrar los datos y proteger las transferencias de datos hacia la nube a través de *https*.|
|Límite de ancho de banda| Limite el uso de ancho de banda durante las horas punta.|

<!--|ExpressRoute | Added security through ExpressRoute. Use peering configuration where traffic from local devices to the cloud storage endpoints travels over the ExpressRoute. For more information, see [ExpressRoute overview](../expressroute/expressroute-introduction.md).|-->

## <a name="components"></a>Componentes

La solución Azure Stack Edge Pro GPU consta de un recurso Azure Stack Edge, un dispositivo físico de Azure Stack Edge Pro GPU y una interfaz de usuario web local.

* **Dispositivo físico de Azure Stack Edge Pro GPU**: un servidor de montaje en bastidor 1U proporcionado por Microsoft que se puede configurar para enviar datos a Azure.
    
* **Recurso Azure Stack Edge**: un recurso de Azure Portal que le permite administrar un dispositivo de Azure Stack Edge Pro GPU desde una interfaz web a la cual puede acceder desde diferentes ubicaciones geográficas. Use el recurso Azure Stack Edge para crear y administrar recursos, ver y administrar dispositivos y alertas y administrar recursos compartidos.  

    Para más información, vaya a [Creación de un pedido para el dispositivo de Azure Stack Edge Pro GPU](azure-stack-edge-gpu-deploy-prep.md#create-a-new-resource).

* **Interfaz de usuario web local de Azure Stack Edge Pro GPU**: una interfaz de usuario web local basada en explorador que se encuentra en su dispositivo Azure Stack Edge Pro GPU y cuya finalidad principal es la configuración inicial del dispositivo. Además, use la interfaz de usuario web local para ejecutar diagnósticos, apagar y reiniciar el dispositivo de Azure Stack Edge Pro GPU, ver registros de copia y ponerse en contacto con el Soporte técnico de Microsoft para realizar una solicitud de servicio.

    [!INCLUDE [azure-stack-edge-gateway-local-web-ui-languages](../../includes/azure-stack-edge-gateway-local-web-ui-languages.md)]
    
    Para más información acerca de cómo usar la interfaz de usuario basada en web, consulte [Uso de la interfaz de usuario web para administrar Azure Stack Edge Pro GPU](azure-stack-edge-manage-access-power-connectivity-mode.md).

## <a name="region-availability"></a>Disponibilidad en regiones

El dispositivo físico de Azure Stack Edge Pro GPU, el recurso de Azure y la cuenta de almacenamiento de destino a la que transferirá los datos no tienen que estar en la misma región.

- **Disponibilidad**: para esta versión, el recurso está disponible en las regiones Este de EE. UU., Oeste de Europa y Sudeste Asiático.

- **Disponibilidad del dispositivo**: para ver una lista de todos los países y regiones en los que el dispositivo Azure Stack Edge Pro GPU está disponible, vaya a la sección **Disponibilidad** de la pestaña **Azure Stack Edge Pro** de [Precios de Azure Stack Edge Pro GPU](https://azure.microsoft.com/pricing/details/azure-stack/edge/#azureStackEdgePro).
    
- **Cuentas de almacenamiento de destino**: las cuentas de almacenamiento que almacenan los datos están disponibles en todas las regiones de Azure. Las regiones en las que las cuentas de almacenamiento almacenan los datos de Azure Stack Edge Pro GPU deben estar ubicadas cerca de la ubicación del dispositivo a fin de obtener un rendimiento óptimo. Una cuenta de almacenamiento que se encuentra lejos del dispositivo causa latencias largas y un rendimiento más lento.

El servicio Azure Stack Edge es un servicio no regional. Para obtener más información, consulte [Regiones y zonas de disponibilidad en Azure](../availability-zones/az-overview.md). El servicio Azure Stack Edge no depende de una región específica de Azure, lo que lo hace más resistentes a las interrupciones en toda la zona, así como a interrupciones en toda la región.

## <a name="next-steps"></a>Pasos siguientes

- Revise los [requisitos del sistema de Azure Stack Edge Pro GPU](azure-stack-edge-gpu-system-requirements.md).

- Consulte los [límites de Azure Stack Edge Pro GPU](azure-stack-edge-limits.md).
- Implemente [Azure Stack Edge Pro GPU](azure-stack-edge-gpu-deploy-prep.md) en Azure Portal.
