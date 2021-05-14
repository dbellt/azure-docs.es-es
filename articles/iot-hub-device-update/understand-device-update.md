---
title: Introducción a Device Update para Azure IoT Hub | Microsoft Docs
description: Device Update para IoT Hub es un servicio que permite implementar actualizaciones por vía inalámbrica (OTA) para los dispositivos de IoT.
author: vimeht
ms.author: vimeht
ms.date: 2/11/2021
ms.topic: overview
ms.service: iot-hub-device-update
ms.openlocfilehash: 994dc02b0d61eff715b1368e8ea7bd592cc79d70
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108133644"
---
# <a name="device-update-for-iot-hub-preview-overview"></a>Introducción a Device Update para IoT Hub (versión preliminar)

Device Update para IoT Hub es un servicio que permite implementar actualizaciones por vía inalámbrica (OTA) para los dispositivos de IoT.

A medida que las organizaciones buscan aumentar la productividad y la eficiencia operativa, cada vez se adoptan más soluciones de Internet de las cosas (IoT), lo que hace que sea esencial que los cimientos de los dispositivos que forman estas soluciones sean la confiabilidad y la seguridad, y que sean fáciles de conectar y administrar a escala. Device Update para IoT Hub es una plataforma de un extremo a otro que los clientes pueden usar para publicar, distribuir y administrar actualizaciones de forma inalámbrica para todos los elementos, desde minúsculos sensores hasta dispositivos de nivel de puerta de enlace. 

Para sacar el máximo partido de la transformación digital habilitada para IoT, los clientes necesitan esta capacidad para operar, mantener y actualizar dispositivos a escala. Explore las ventajas de la implementación de Device Update para IoT Hub, que incluyen la capacidad de responder rápidamente a las amenazas de seguridad e implementar nuevas características para obtener objetivos empresariales sin incurrir en los costos adicionales de desarrollo y mantenimiento que suponen la creación de sus propias plataformas de actualización.

## <a name="support-for-a-wide-range-of-iot-devices"></a>Compatibilidad con una amplia gama de dispositivos de IoT


Device Update para IoT Hub se ha diseñado para ofrecer no solo una implementación de actualizaciones optimizada, sino también operaciones simplificadas mediante la integración con [Azure IoT Hub](https://azure.microsoft.com/en-us/services/iot-hub/). Esta integración facilita la adopción de Device Update en cualquier solución existente. Proporciona una solución hospedada en la nube para conectar prácticamente cualquier dispositivo. Device Update es compatible con una amplio conjunto de sistemas operativos de IoT, entre los que se incluyen Linux y [Azure RTOS](https://azure.microsoft.com/en-us/services/rtos/) (sistema operativo en tiempo real), y se puede ampliar a través de código abierto. Estamos desarrollando de forma conjunta Device Update para las ofertas de IoT Hub con nuestros asociados del sector de los semiconductores, entre los que se incluyen STMicroelectronics, NXP, Renesas y Microchip. Consulte los [ejemplos](https://github.com/azure-rtos/samples/tree/PublicPreview/ADU) de paneles clave de evaluación de semiconductores que incluyen las guías introductorias para aprender a configurar, crear e implementar las actualizaciones inalámbricas en dispositivos de clase MCU. 

Se proporcionan un archivo binario del simulador de Device Update Agent e imágenes Yocto de referencia de Raspberry Pi.
Device Update para IoT Hub también admite la actualización de dispositivos de Azure IoT Edge. Se proporciona Device Update Agent para la plataforma Ubuntu Server 18.04 (amd64). Device Update para IoT Hub también proporciona código de código abierto si no se ejecuta una de las plataformas anteriores, También puede portar el agente a la distribución que se ejecuta.

Device Update funciona con IoT Plug and Play (PnP) y puede administrar cualquier dispositivo compatible con las interfaces PnP necesarias. Para más información, consulte [Device Update para IoT Hub e IoT Plug and Play](device-update-plug-and-play.md).

## <a name="support-for-a-wide-range-of-update-artifacts"></a>Compatibilidad con un amplio conjunto de artefactos de actualización

Device Update para IoT Hub admite dos formas de actualizaciones: basada en imágenes y basada en paquetes.

Las actualizaciones basadas en paquetes son actualizaciones dirigidas que solo modifican una aplicación o un componente específicos en el dispositivo. Esto conduce a un menor consumo de ancho de banda y ayuda a reducir el tiempo necesario para descargar e instalar la actualización. Normalmente, las actualizaciones de paquetes implican menos tiempo de inactividad de los dispositivos al aplicar una actualización y evitan la sobrecarga asociada a la creación de imágenes.

Las actualizaciones con imágenes proporcionan un mayor nivel de confianza en el estado final del dispositivo. Normalmente, es más fácil replicar los resultados de una actualización basada en imágenes entre un entorno de preproducción y un entorno de producción, ya que no plantea los mismos desafíos que los paquetes y sus dependencias.
Debido a su naturaleza atómica, también puede adoptar fácilmente un modelo de conmutación por error A/B.

No hay una sola respuesta correcta, y se puede elegir una respuesta diferente en función de los casos de uso concretos. Device Update para IoT Hub admite tanto la actualización basada en imágenes como la basada en paquetes, lo que le permite elegir el modelo de actualización adecuado para el entorno del dispositivo.

## <a name="flexible-features-for-updating-devices"></a>Características flexibles para la actualización de dispositivos

Las características de Device Update para IoT Hub proporcionan una experiencia eficaz y flexible, lo que incluye:

* Experiencia de usuario de administración de actualizaciones integrada con Azure IoT Hub.
* Implementación gradual de actualizaciones a través de la agrupación de dispositivos y los controles de programación de actualizaciones.
* API de programación que habiliten la automatización y experiencias del portal personalizadas.
* Vistas rápidas del estado y del cumplimiento de las actualizaciones en conjuntos de dispositivos heterogéneos
* Compatibilidad con actualizaciones de dispositivos resistentes (A/B) para ofrecer una reversión sin problemas
* Suscripción y controles de acceso basado en roles disponibles a través del portal de Azure.com
* Almacenamiento en caché de contenido local y compatibilidad con perímetro anidado para habilitar la actualización de dispositivos desconectados de la nube
* Herramientas de generación de informes y de administración de actualizaciones detalladas 

Con los controles de implementación y administración de Device Update para IoT Hub, los usuarios pueden maximizar la productividad y ahorrar mucho tiempo. Device Update para IoT Hub brinda la posibilidad de agrupar los dispositivos y especificar en cuáles de ellos se debe implementar cada actualización. Los usuarios también pueden ver el estado de implementación de las actualizaciones y asegurarse de que todos los dispositivo las aplican correctamente.

Cuando se produce un error de actualización, Device Update para IoT Hub también permite a los usuarios no solo identificar los dispositivos que no han podido aplicar la actualización, sino también ver los detalles relacionados con el error. La capacidad para identificar los dispositivos que no se han actualizado supone un ahorro de muchas horas que se dedicaban a intentar localizar el origen.

### <a name="best-in-class-security-at-global-scale"></a>La mejor seguridad a escala global

Microsoft Azure da soporte a más de mil millones de dispositivos de IoT en todo el mundo, un número que crece rápidamente día a día. Device Update para IoT Hub se basa en esta experiencia y en la confiabilidad comprobada que muestra la plataforma de Windows Update, por lo que los dispositivos se pueden actualizar sin problemas a escala global.

Device Update para IoT Hub usa una completa seguridad de la nube al perímetro desarrollada para Microsoft Azure, para que los clientes no tengan que dedicar ni un solo minuto a crearla desde cero.


## <a name="device-update-workflows"></a>Flujos de trabajo de Device Update

La funcionalidad de Device Update se puede dividir en tres áreas: integración de agente, importación y administración.

### <a name="device-update-agent"></a>Device Update Agent

Cuando se recibe un comando de actualización en un dispositivo, se ejecutará la fase de actualización solicitada (descarga, instalación y aplicación). Durante cada una de estas fases, el estado se devuelve a Device Update a través de IoT Hub, con el fin de que se pueda ver el estado actual de cualquier implementación. Si no hay ninguna actualización en curso, el estado que se devuelve es "Idle" (Inactivo). Las implementaciones se pueden cancelar en cualquier momento.

:::image type="content" source="media/understand-device-update/client-agent-workflow.png" alt-text="Diagrama del flujo de trabajo de Device Update Agent" lightbox="media/understand-device-update/client-agent-workflow.png":::.

[Más información](device-update-agent-overview.md) sobre Device Update Agent. 

### <a name="importing"></a>Importación

La importación es el modo en que las actualizaciones se ingieren en Device Update para que se puedan implementar en los dispositivos. La actualización del dispositivo admite la implementación de una sola actualización por dispositivo, lo que hace que sea idóneo para las actualizaciones de imagen completa que actualizan toda una partición de sistema operativo a la vez, o un manifiesto APT que describe todos los paquetes que se desean actualizar en el dispositivo. Para importar actualizaciones en Device Update, lo primero que se debe hacer es crear un manifiesto de importación que describa la actualización y, después, cargar los archivos de actualización y el manifiesto de importación en una ubicación a la que se pueda acceder desde Internet. Después, puede usar Azure Portal o la [API REST de Device Update](/rest/api/deviceupdate/) para iniciar el proceso asincrónico de importación de la actualización. Device Update carga los archivos, los procesa y los pone a disposición de los dispositivos de IoT para su distribución.

En caso de que el contenido sea confidencial, proteja la descarga mediante una firma de acceso compartido (SAS), como una SAS ad hoc para Azure Blob Storage. [Más información acerca de las SAS](../storage/common/storage-sas-overview.md).

:::image type="content" source="media/understand-device-update/import-update.png" alt-text="Diagrama del flujo de trabajo de la importación de Device Update para IoT Hub" lightbox="media/understand-device-update/import-update.png":::.

[Más información](import-concepts.md) sobre la importación de actualizaciones. 

### <a name="grouping-and-deployment"></a>Agrupación e implementación

Después de importar una actualización, se pueden ver tanto las actualizaciones compatibles de los dispositivos como las clases de dispositivos.

Device Update admite el concepto de **Grupos** a través de las etiquetas de IoT Hub. La implementación de cualquier actualización primero en un grupo de prueba es una buena forma de reducir el riesgo de que surjan problemas durante un lanzamiento en producción.

En Device Update, las implementaciones son una forma de conectar el contenido correcto a un conjunto concreto de dispositivos compatibles. Device Update organiza el proceso de envío de comandos a los distintos dispositivo, les indica que descarguen e instalen las actualizaciones y obtiene su estado.

:::image type="content" source="media/understand-device-update/manage-deploy-updates.png" alt-text="Diagrama del flujo de trabajo de agrupación e implementación en Device Update para IoT Hub" lightbox="media/understand-device-update/manage-deploy-updates.png":::.

[Más información](device-update-compliance.md) acerca de los conceptos de implementación.

[Más información](device-update-groups.md) sobre los grupos de Device Update.


## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Creación de una instancia y de cuentas de Device Update](create-device-update-account.md)