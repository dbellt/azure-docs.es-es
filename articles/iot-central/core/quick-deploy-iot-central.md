---
title: 'Inicio rápido: Creación de una aplicación de Azure IoT Central | Microsoft Docs'
description: 'Inicio rápido: Creación de una nueva aplicación de Azure IoT Central. Cree la aplicación mediante el plan de precios gratuito o uno de los planes de precios estándar.'
author: viv-liu
ms.author: viviali
ms.date: 12/28/2020
ms.topic: quickstart
ms.service: iot-central
services: iot-central
manager: corywink
ms.openlocfilehash: ef76b1283215da9ff08efb4d151d8e0e99a228f0
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108768758"
---
# <a name="quickstart---create-an-azure-iot-central-application"></a>Inicio rápido: Creación de una aplicación de Azure IoT Central

En este inicio rápido se muestra cómo crear una aplicación de Azure IoT Central.

## <a name="prerequisite"></a>Requisito previo 

 - Una cuenta de Azure con una suscripción activa. Cree una cuenta [gratuita](https://aka.ms/createazuresubscription).
 - La suscripción de Azure debe tener acceso de colaborador

## <a name="create-an-application"></a>Crear una aplicación

Navegue al sitio de [Compilación Azure IoT Central](https://aka.ms/iotcentral). Después, inicie sesión con una cuenta Microsoft personal, profesional o educativa.

Puede crear una nueva aplicación desde la lista de plantillas de IoT Central relevantes del sector, lo que le ayudaría a empezar a trabajar rápidamente, o bien partir de cero con una plantilla de **Aplicación personalizada**. En este inicio rápido se usa la plantilla **Aplicación personalizada**.

Para crear una aplicación de Azure IoT Central a partir de la plantilla **Aplicación personalizada**:

1. Vaya a la página **Compilar**:

    :::image type="content" source="media/quick-deploy-iot-central/iotcentralcreate-new-application.png" alt-text="Página Compilación de una aplicación de IoT":::

1. Elija **Aplicación personalizada**.

1. En la página **Nueva aplicación**, asegúrese de que **Aplicación personalizada** está seleccionada en **Plantilla de aplicación**.

1. Azure IoT Central sugiere automáticamente un **nombre de aplicación** en función de la plantilla de aplicación que ha seleccionado. Puede usar este nombre o escribir su propio nombre descriptivo de la aplicación.

1. Azure IoT Central también genera un prefijo de dirección **URL** único, en función del nombre de la aplicación. Esta dirección URL se usa para tener acceso a la aplicación. Si quiere, cambie el prefijo de esta URL por algo más fácil de recordar.

    :::image type="content" source="media/quick-deploy-iot-central/iotcentralcreate-custom.png" alt-text="Página de creación de una aplicación de Azure IoT Central":::

    :::image type="content" source="media/quick-deploy-iot-central/iotcentralcreate-billinginfo.png" alt-text="Información acerca de la facturación de Azure IoT Central":::

    > [!Tip]
    > Si ha eligió **Aplicación personalizada** en la página anterior, verá la lista desplegable **Plantilla de aplicación**. Es posible que la muestre otras plantillas que la organización ha puesto a su disposición.

1. Elija si desea crear esta aplicación mediante el plan de precios de evaluación gratuita durante siete días o uno de los planes de precios Estándar:

    - Las aplicaciones que se crean mediante el plan *gratuito* no tienen costo durante siete días y admiten un máximo de cinco dispositivos. En cualquier momento antes de que expiren puede convertirlas para que usen un plan de precios estándar.
        > [!NOTE]
        > Las aplicaciones creadas con el plan *gratuito* no requieren suscripciones de Azure y, por lo tanto, no aparecerán en la suscripción de Azure en Azure Portal. Solo puede ver y administrar las aplicaciones gratuitas del portal de IoT Central.          
    - Las aplicaciones que se crean con un plan *estándar* se facturan por dispositivo, y puede elegir entre el plan de precios **Estándar 0**, **Estándar 1** o **Estándar 2**, siendo gratis los dos primeros dispositivos. Obtenga más información sobre los planes de precios gratuito y estándar en la [página de precios de Azure IoT Central](https://azure.microsoft.com/pricing/details/iot-central/). Si crea una aplicación que use un plan de precios estándar, tendrá que seleccionar su *directorio*, *suscripción de Azure* y *ubicación*:
        - *Directorio* es la instancia de Azure Active Directory en que se crea la aplicación. Azure Active Directory contiene identidades de usuario, credenciales y otra información de la organización. Si no tiene Azure Active Directory, se crea uno automáticamente al crear una suscripción de Azure.
        - Una *suscripción de Azure* permite crear instancias de los servicios de Azure. IoT Central aprovisiona los recursos de su suscripción. Si no tiene una suscripción a Azure, puede crear la de forma gratuita en la [página de suscripción a Azure](https://aka.ms/createazuresubscription). Después de crear la suscripción a Azure, vuelva a la página **Nueva aplicación**. La nueva suscripción aparece en el cuadro de lista desplegable **Suscripción de Azure**.
        - *Ubicación* es la [zona geográfica](https://azure.microsoft.com/global-infrastructure/geographies/) en la que desea crear la aplicación. Normalmente, se debe elegir la ubicación más cercana físicamente a los dispositivos para un rendimiento óptimo. Una vez que elija una ubicación, no puede mover la aplicación a otra más adelante.

1. Revise los Términos y Condiciones y seleccione **Crear** en la parte inferior de la página. Después de unos minutos, la aplicación de IoT Central estará lista para usarse:

    :::image type="content" source="media/quick-deploy-iot-central/iotcentral-application.png" alt-text="Aplicación de Azure IoT Central":::

## <a name="clean-up-resources"></a>Limpieza de recursos

[!INCLUDE [iot-central-clean-up-resources](../../../includes/iot-central-clean-up-resources.md)]

## <a name="next-steps"></a>Pasos siguientes

En esta guía de inicio rápido, ha creado una aplicación de IoT Central. Este es el siguiente paso sugerido para obtener más información sobre IoT Central:

> [!div class="nextstepaction"]
> [Agregar un dispositivo simulado a la aplicación IoT Central](./quick-create-simulated-device.md)

