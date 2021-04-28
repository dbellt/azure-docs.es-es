---
title: Administración de IoT Central desde Azure Portal | Microsoft Docs
description: En este artículo se describe cómo crear y administrar aplicaciones de IoT Central desde el Azure Portal.
services: iot-central
ms.service: iot-central
author: dominicbetts
ms.author: dobett
ms.date: 04/17/2021
ms.topic: how-to
ms.openlocfilehash: 3e5e126815d0171a6c1627a08419b05b9a3c0c23
ms.sourcegitcommit: 79c9c95e8a267abc677c8f3272cb9d7f9673a3d7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/19/2021
ms.locfileid: "107719212"
---
# <a name="manage-iot-central-from-the-azure-portal"></a>Administración de IoT Central desde Azure Portal

[!INCLUDE [iot-central-selector-manage](../../../includes/iot-central-selector-manage.md)]

Puede usar [Azure Portal](https://portal.azure.com) para crear y administrar aplicaciones de IoT Central.

## <a name="create-iot-central-applications"></a>Creación de aplicaciones de IoT Central

[!INCLUDE [Warning About Access Required](../../../includes/iot-central-warning-contribitorrequireaccess.md)]

Para crear una aplicación, vaya a la página [Aplicación IoT Central](https://ms.portal.azure.com/#create/Microsoft.IoTCentral) en Azure Portal:

![Creación de formularios de IoT Central](media/howto-manage-iot-central-from-portal/image6a.png)

* El **nombre del recurso** es un nombre único que puede elegir para la aplicación IoT Central en el grupo de recursos de Azure.

* La **dirección URL de la aplicación** es la dirección URL que puede usar para acceder a la aplicación.

* **Ubicación** es la [zona geográfica](https://azure.microsoft.com/global-infrastructure/geographies/) en la que desea crear la aplicación. Normalmente, se debe elegir la ubicación más cercana físicamente a los dispositivos para un rendimiento óptimo. Azure IoT Central está disponible actualmente en las siguientes ubicaciones:

  * Asia Pacífico
  * Australia
  * Europa
  * Japón
  * Reino Unido
  * Estados Unidos

  Una vez que elija una ubicación, no puede mover la aplicación a otra más adelante.

Después de rellenar todos los campos, seleccione **Crear**. Para más información, consulte el inicio rápido [Creación de una aplicación de Azure IoT Central](quick-deploy-iot-central.md).

## <a name="manage-existing-iot-central-applications"></a>Administrar las aplicaciones existentes de IoT Central

Si ya tiene una aplicación de Azure IoT Central, puede eliminarla o moverla a un grupo de recursos o a una suscripción diferentes en Azure Portal.

> [!NOTE]
> Las aplicaciones creadas con el plan *gratuito* no requieren suscripciones de Azure y, por lo tanto, no aparecerán en la suscripción de Azure en Azure Portal. Solo puede ver y administrar las aplicaciones gratuitas del portal de IoT Central.

Para empezar, busque la aplicación en la barra de búsqueda de la parte superior de Azure Portal. También puede ver todas las aplicaciones buscando _aplicaciones de IoT Central_ y seleccionando el servicio:

![Captura de pantalla que muestra los resultados de la búsqueda de "aplicaciones de IoT Central" con el primer servicio seleccionado.](media/howto-manage-iot-central-from-portal/search-iot-central.png)

Al seleccionar una aplicación en los resultados de la búsqueda, Azure Portal muestra su información general. Para ir a la aplicación, puede seleccionar la **URL de la aplicación de IoT Central**:

![Captura de pantalla que muestra la página "Información general" con la opción "Dirección URL de la aplicación de IoT Central" resaltada.](media/howto-manage-iot-central-from-portal/image3.png)

Para mover la aplicación a otro grupo de recursos, seleccione **Cambiar** al lado del grupo de recursos. En la página **Mover recursos**, seleccione el grupo de recursos al que le gustaría mover esta aplicación:

![Captura de pantalla que muestra la página "Información general" con la opción "Grupo de recursos (cambiar)" resaltada.](media/howto-manage-iot-central-from-portal/image4a.png)

Para mover la aplicación a otra suscripción, seleccione **Cambiar** al lado de la suscripción. En la página **Mover recursos**, elija la suscripción a la que le gustaría mover esta aplicación:

![Portal de administración: administración de recursos](media/howto-manage-iot-central-from-portal/image5a.png)

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha aprendido a administrar aplicaciones de Azure IoT Central desde Azure Portal, le sugerimos el paso siguiente:

> [!div class="nextstepaction"]
> [Administrar su aplicación](howto-administer.md)