---
title: Introducción a Microsoft Azure Stack Edge Pro FPGA | Microsoft Docs
description: Se describe Azure Stack Edge Pro FPGA, una solución de almacenamiento que usa un dispositivo físico para las transferencias a Azure basadas en red.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: overview
ms.date: 03/15/2021
ms.author: alkohli
ms.openlocfilehash: f19973979cda0fba54b2f03792e8f3ea31b4d15c
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110099377"
---
# <a name="what-is-azure-stack-edge-pro-fpga"></a>¿Qué es Azure Stack Edge Pro FGPA?

[!INCLUDE [Azure Stack Edge Pro FPGA end-of-life](../../includes/azure-stack-edge-fpga-eol.md)]

Azure Stack Edge Pro con FPGA es un dispositivo informático perimetral habilitado para la inteligencia artificial que cuenta con funcionalidades de transferencia de datos de red. En este artículo se proporciona información general acerca de la solución de Azure Stack Edge Pro con FPGA, sus ventajas, sus funcionalidades clave y los escenarios de implementación.

Azure Stack Edge Pro con FPGA es una solución de hardware como servicio. Microsoft le envía un dispositivo administrado en la nube con una matriz de puertas programables (FPGA) que permite la inferencia acelerada de IA y tiene todas las funcionalidades de una puerta de enlace de almacenamiento de red.

Azure Data Box Edge ha cambiado de nombre: ahora se denomina Azure Stack Edge.

## <a name="use-cases"></a>Casos de uso

Estos son los distintos escenarios en los que se puede usar Azure Stack Edge Pro FPGA para la inferencia rápida de Machine Learning (ML) perimetral y el preprocesamiento de datos antes de enviarlos a Azure.

- **Inferencia con Azure Machine Learning**: con Azure Stack Edge Pro FPGA, puede ejecutar modelos de ML para obtener resultados rápidos sobre los que se puede actuar antes de que los datos se envíen a la nube. El conjunto de datos completo se puede transferir opcionalmente para continuar entrenando y mejorando los modelos de Machine Learning. Para más información sobre cómo usar los modelos acelerados por hardware de Azure ML en el dispositivo de Azure Stack Edge Pro FPGA, consulte [Implementación de modelos acelerados por hardware de Azure ML en Azure Stack Edge Pro FPGA](../machine-learning/how-to-deploy-fpga-web-service.md#deploy-to-a-local-edge-server).

- **Preprocesamiento de datos**: transforme datos antes de enviarlos a Azure para crear un conjunto de datos más accionable. El preprocesamiento se puede usar para: 

    - Agregar datos.
    - Modificar datos, por ejemplo, para quitar los datos personales.
    - Subconjuntos de datos para optimizar el almacenamiento y el ancho de banda, o para un análisis más exhaustivo.
    - Analizar y reaccionar a los eventos de IoT. 

- **Transferencia de datos a través de la red a Azure**: use Azure Stack Edge Pro FPGA para transferir datos a Azure de forma rápida y sencilla a fin de poder realizar más procesos y análisis, o para fines de archivado. 

## <a name="key-capabilities"></a>Principales capacidades

Azure Stack Edge Pro FPGA presenta las siguientes funcionalidades:

|Capacidad |Descripción  |
|---------|---------|
|Inferencia de IA acelerada| Habilitado por la FPGA integrada.|
|Procesamiento       |Permite el análisis, procesamiento y filtrado de datos.|
|Alto rendimiento | Transferencias de datos y proceso de alto rendimiento.|
|Acceso a datos     | Ofrece acceso directo a los datos de Azure Storage Blob y Azure Files mediante las API en la nube para un procesamiento de datos adicional en la nube. La memoria caché local del dispositivo se usa para agilizar el acceso a los archivos usados más recientemente.|
|Administración en la nube     |El dispositivo y el servicio se administran con Azure Portal.  |
|Carga sin conexión     | El modo sin conexión es compatible con escenarios de carga sin conexión.|
|Protocolos admitidos     | Compatibilidad con los protocolos estándar SMB y NFS para la ingesta de datos. <br> Para más información sobre las versiones compatibles, consulte [Requisitos del sistema de Azure Stack Edge Pro FPGA](azure-stack-edge-system-requirements.md).|
|Actualización de datos     | Capacidad para actualizar los archivos locales con la versión más reciente de la nube.|
|Cifrado    | Compatibilidad con BitLocker para, localmente, cifrar los datos y proteger las transferencias de datos hacia la nube a través de *https*.|
|Límite de ancho de banda| Limite el uso de ancho de banda durante las horas punta.|
|ExpressRoute | Mayor seguridad a través de ExpressRoute. Use la configuración de emparejamiento en la que el tráfico de los dispositivos locales a los puntos de conexión de almacenamiento en la nube viaja a través de ExpressRoute. Para más información, consulte [Información general sobre ExpressRoute](../expressroute/expressroute-introduction.md).

## <a name="components"></a>Componentes

La solución Azure Stack Edge Pro FPGA consta de un recurso Azure Stack Edge, un dispositivo físico de Azure Stack Edge Pro FPGA y una interfaz de usuario web local.

* **Dispositivo físico de Azure Stack Edge Pro FPGA**: un servidor montado en bastidor 1U proporcionado por Microsoft que se puede configurar para enviar datos a Azure.
    
* **Recurso de Azure Stack Edge**: un recurso de Azure Portal que le permite administrar un dispositivo de Azure Stack Edge Pro FPGA desde una interfaz web a la cual puede acceder desde diferentes ubicaciones geográficas. Use el recurso Azure Stack Edge para crear y administrar recursos, administrar recursos compartidos, ver y administrar dispositivos y alertas.
  
   <!--[The Azure Stack Edge service in Azure portal](media/data-box-overview/data-box-Edge-service1.png)-->

   Como Azure Stack Edge Pro FPGA se aproxima al final de su vida útil, no se están atendiendo los pedidos de nuevos dispositivos de Azure Stack Edge Pro FPGA. Si es un cliente nuevo, le recomendamos que explore el uso de los dispositivos de Azure Stack Edge Pro-GPU para sus cargas de trabajo. Para más información, vaya a [¿Qué es Azure Stack Edge Pro con GPU?](azure-stack-edge-gpu-overview.md)? Para más información sobre cómo ordenar un dispositivo de Azure Stack Edge Pro con GPU, vaya al artículo sobre [creación de un nuevo recurso para Azure Stack Edge Pro - GPU](azure-stack-edge-gpu-deploy-prep.md?tabs=azure-portal#create-a-new-resource).

   Si es un cliente existente, puede crear un nuevo recurso de Azure Stack Edge si necesita reemplazar o restablecer el dispositivo de Azure Stack Edge Pro FPGA existente. Para obtener instrucciones, vaya a [Creación de un pedido para el dispositivo de Azure Stack Edge Pro FPGA](azure-stack-edge-deploy-prep.md#create-new-resource-for-existing-device).

* **Interfaz de usuario web local de Azure Stack Edge Pro FPGA**: use la interfaz de usuario web local para ejecutar diagnósticos, apagar y reiniciar el dispositivo de Azure Stack Edge Pro FPGA, ver registros de copia y ponerse en contacto con el Soporte técnico de Microsoft para presentar una solicitud de servicio.

    <!--![The Azure Stack Edge Pro FPGA local web UI](media/data-box-Edge-overview/data-box-Edge-local-web-ui.png)-->

    Para información sobre cómo usar la interfaz de usuario basada en web, consulte [Uso de la interfaz de usuario web para administrar Azure Stack Edge Pro FPGA](azure-stack-edge-manage-access-power-connectivity-mode.md).

## <a name="region-availability"></a>Disponibilidad en regiones

El dispositivo físico de Azure Stack Edge Pro FPGA, el recurso de Azure y la cuenta de almacenamiento de destino a la que transferirá los datos no tienen que estar en la misma región.

- **Disponibilidad de recursos**: para ver una lista de todas las regiones en las que está disponible el recurso de Azure Stack Edge, consulte [Productos de Azure disponibles por región](https://azure.microsoft.com/global-infrastructure/services/?products=databox&regions=all). Azure Stack Edge Pro FPGA también se puede implementar en la nube de Azure Government. Para más información, consulte [¿Qué es Azure Government?](../azure-government/documentation-government-welcome.md).
    
- **Cuentas de almacenamiento de destino**: las cuentas de almacenamiento que almacenan los datos están disponibles en todas las regiones de Azure. Las regiones en las que las cuentas de almacenamiento almacenan los datos de Azure Stack Edge Pro FPGA deben estar ubicadas cerca de la ubicación del dispositivo a fin de obtener un rendimiento óptimo. Una cuenta de almacenamiento que se encuentra lejos del dispositivo causa latencias largas y un rendimiento más lento.

El servicio Azure Stack Edge es un servicio no regional. Para obtener más información, consulte [Regiones y zonas de disponibilidad en Azure](../availability-zones/az-overview.md). El servicio Azure Stack Edge no depende de una región específica de Azure, lo que lo hace más resistentes a las interrupciones en toda la zona, así como a interrupciones en toda la región.

## <a name="next-steps"></a>Pasos siguientes

- Revise los [requisitos del sistema de Azure Stack Edge Pro FPGA](azure-stack-edge-system-requirements.md).
- Comprenda los [límites de Azure Stack Edge Pro FPGA](azure-stack-edge-limits.md).
- Implemente [Azure Stack Edge Pro FPGA](azure-stack-edge-deploy-prep.md) en Azure Portal.