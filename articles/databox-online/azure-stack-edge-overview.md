---
title: Introducción a Microsoft Azure Stack Edge Pro | Microsoft Docs
description: Se describe Azure Stack Edge Pro, una solución de almacenamiento que usa un dispositivo físico para las transferencias basadas en red a Azure.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: overview
ms.date: 03/15/2021
ms.author: alkohli
ms.openlocfilehash: b4405c3d1c6588a67acfc66e458a3c9120d2110d
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108139136"
---
# <a name="what-is-azure-stack-edge-pro-with-fpga"></a>¿Qué es Azure Stack Edge Pro con FPGA?

[!INCLUDE [Azure Stack Edge Pro FPGA end-of-life](../../includes/azure-stack-edge-fpga-eol.md)]

Azure Stack Edge Pro con FPGA es un dispositivo informático perimetral habilitado para la inteligencia artificial que cuenta con funcionalidades de transferencia de datos de red. En este artículo se proporciona información general acerca de la solución de Azure Stack Edge Pro con FPGA, sus ventajas, sus funcionalidades clave y los escenarios de implementación.

Azure Stack Edge Pro con FPGA es una solución de hardware como servicio. Microsoft le envía un dispositivo administrado en la nube con una matriz de puertas programables (FPGA) que permite la inferencia acelerada de IA y tiene todas las funcionalidades de una puerta de enlace de almacenamiento de red.

Azure Data Box Edge ha cambiado de nombre: ahora se denomina Azure Stack Edge.

## <a name="use-cases"></a>Casos de uso

Estos son los distintos escenarios en los que se puede usar Azure Stack Edge Pro para la inferencia rápida de Machine Learning (ML) perimetral y el preprocesamiento de datos antes de enviarlos a Azure.

- **Inferencia con Azure Machine Learning**: con Azure Stack Edge Pro, puede ejecutar modelos de ML para obtener resultados rápidos sobre los que se puede actuar antes de que los datos se envíen a la nube. El conjunto de datos completo se puede transferir opcionalmente para continuar entrenando y mejorando los modelos de Machine Learning. Para más información sobre cómo usar los modelos acelerados por hardware de Azure ML en el dispositivo de Azure Stack Edge Pro, consulte [Implementación de modelos acelerados por hardware de Azure ML en Azure Stack Edge Pro](../machine-learning/how-to-deploy-fpga-web-service.md#deploy-to-a-local-edge-server).

- **Preprocesamiento de datos**: transforme datos antes de enviarlos a Azure para crear un conjunto de datos más accionable. El preprocesamiento se puede usar para: 

    - Agregar datos.
    - Modificar datos, por ejemplo, para quitar los datos personales.
    - Subconjuntos de datos para optimizar el almacenamiento y el ancho de banda, o para un análisis más exhaustivo.
    - Analizar y reaccionar a los eventos de IoT. 

- **Transferir datos a través de la red a Azure**: use Azure Stack Edge Pro para transferir datos a Azure de forma rápida y sencilla para poder realizar más procesos y análisis, o para fines de archivado. 

## <a name="key-capabilities"></a>Principales capacidades

Azure Stack Edge Pro tiene las siguientes funcionalidades:

|Capacidad |Descripción  |
|---------|---------|
|Inferencia de IA acelerada| Habilitado por la FPGA integrada.|
|Procesamiento       |Permite el análisis, procesamiento y filtrado de datos.|
|Alto rendimiento | Transferencias de datos y proceso de alto rendimiento.|
|Acceso a datos     | Ofrece acceso directo a los datos de Azure Storage Blob y Azure Files mediante las API en la nube para un procesamiento de datos adicional en la nube. La memoria caché local del dispositivo se usa para agilizar el acceso a los archivos usados más recientemente.|
|Administración en la nube     |El dispositivo y el servicio se administran con Azure Portal.  |
|Carga sin conexión     | El modo sin conexión es compatible con escenarios de carga sin conexión.|
|Protocolos admitidos     | Compatibilidad con los protocolos estándar SMB y NFS para la ingesta de datos. <br> Para más información sobre las versiones compatibles, consulte [Requisitos del sistema de Azure Stack Edge Pro](azure-stack-edge-system-requirements.md).|
|Actualización de datos     | Capacidad para actualizar los archivos locales con la versión más reciente de la nube.|
|Cifrado    | Compatibilidad con BitLocker para, localmente, cifrar los datos y proteger las transferencias de datos hacia la nube a través de *https*.|
|Límite de ancho de banda| Limite el uso de ancho de banda durante las horas punta.|
|ExpressRoute | Mayor seguridad a través de ExpressRoute. Use la configuración de emparejamiento en la que el tráfico de los dispositivos locales a los puntos de conexión de almacenamiento en la nube viaja a través de ExpressRoute. Para más información, consulte [Información general sobre ExpressRoute](../expressroute/expressroute-introduction.md).

## <a name="components"></a>Componentes

La solución Azure Stack Edge Pro consta de un recurso Azure Stack Edge, un dispositivo físico de Azure Stack Edge Pro y una interfaz de usuario web local.

* **Dispositivo físico de Azure Stack Edge Pro**: un servidor de montaje en bastidor 1U proporcionado por Microsoft que se puede configurar para enviar datos a Azure.
    
* **Recurso Azure Stack Edge**: un recurso de Azure Portal que le permite administrar un dispositivo de Azure Stack Edge Pro desde una interfaz web a la cual puede acceder desde diferentes ubicaciones geográficas. Use el recurso Azure Stack Edge para crear y administrar recursos, administrar recursos compartidos, ver y administrar dispositivos y alertas.
  
   <!--[The Azure Stack Edge service in Azure portal](media/data-box-overview/data-box-Edge-service1.png)-->

   Como Azure Stack Edge Pro se aproxima al final de su vida útil, no se están atendiendo los pedidos de nuevos dispositivos de Azure Stack Edge Pro. Si es un cliente nuevo, le recomendamos que explore el uso de los dispositivos de Azure Stack Edge Pro-GPU para sus cargas de trabajo. Para más información, vaya a [¿Qué es Azure Stack Edge Pro con GPU?](azure-stack-edge-gpu-overview.md)? Para más información sobre cómo ordenar un dispositivo de Azure Stack Edge Pro con GPU, vaya al artículo sobre [creación de un nuevo recurso para Azure Stack Edge Pro - GPU](azure-stack-edge-gpu-deploy-prep.md?tabs=azure-portal#create-a-new-resource).

   Si es un cliente existente, puede crear un nuevo recurso de Azure Stack Edge Pro si necesita reemplazar o restablecer el dispositivo de Azure Stack Edge Pro existente. Para obtener instrucciones, vaya al artículo sobre [creación de un pedido para el dispositivo de Azure Stack Edge Pro](azure-stack-edge-deploy-prep.md#create-new-resource-for-existing-device).

* **Interfaz de usuario web local de Azure Stack Edge Pro**: use la interfaz de usuario web local para ejecutar diagnósticos, apagar y reiniciar el dispositivo de Azure Stack Edge Pro, ver registros de copia y ponerse en contacto con el Soporte técnico de Microsoft para realizar una solicitud de servicio.

    <!--![The Azure Stack Edge Pro local web UI](media/data-box-Edge-overview/data-box-Edge-local-web-ui.png)-->

    Para más información acerca de cómo usar la interfaz de usuario basada en web, consulte [Uso de la interfaz de usuario web para administrar Azure Stack Edge Pro](azure-stack-edge-manage-access-power-connectivity-mode.md).

## <a name="region-availability"></a>Disponibilidad en regiones

El dispositivo físico de Azure Stack Edge Pro, el recurso de Azure y la cuenta de almacenamiento de destino a la que transferirá los datos no tienen que estar en la misma región.

- **Disponibilidad de recursos**: para ver una lista de todas las regiones en las que está disponible el recurso de Azure Stack Edge, consulte [Productos de Azure disponibles por región](https://azure.microsoft.com/global-infrastructure/services/?products=databox&regions=all). Azure Stack Edge Pro también se puede implementar en la nube de Azure Government. Para más información, consulte [¿Qué es Azure Government?](../azure-government/documentation-government-welcome.md).
    
- **Cuentas de almacenamiento de destino**: las cuentas de almacenamiento que almacenan los datos están disponibles en todas las regiones de Azure. Las regiones en las que las cuentas de almacenamiento almacenan los datos de Azure Stack Edge Pro deben estar ubicadas cerca de la ubicación del dispositivo para un rendimiento óptimo. Una cuenta de almacenamiento que se encuentra lejos del dispositivo causa latencias largas y un rendimiento más lento.

El servicio Azure Stack Edge es un servicio no regional. Para obtener más información, consulte [Regiones y zonas de disponibilidad en Azure](../availability-zones/az-overview.md). El servicio Azure Stack Edge no depende de una región específica de Azure, lo que lo hace más resistentes a las interrupciones en toda la zona, así como a interrupciones en toda la región.

## <a name="next-steps"></a>Pasos siguientes

- Revise los [requisitos del sistema de Azure Stack Edge Pro](azure-stack-edge-system-requirements.md).
- Consulte los [límites de Azure Stack Edge Pro](azure-stack-edge-limits.md).
- Implemente [Azure Stack Edge Pro](azure-stack-edge-deploy-prep.md) en Azure Portal.