---
title: Creación de una aplicación de IoT Central | Microsoft Docs
description: En este artículo se describen las opciones para crear una aplicación de IoT Central desde el sitio de Azure IoT Central, Azure Portal y un entorno de línea de comandos.
services: iot-central
ms.service: iot-central
author: dominicbetts
ms.author: dobett
ms.date: 05/11/2021
ms.topic: how-to
ms.openlocfilehash: 956e84e8eb7281541fbb787a6ce7557a1f5fee03
ms.sourcegitcommit: e832f58baf0b3a69c2e2781bd8e32d4f1ae932c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/27/2021
ms.locfileid: "110586498"
---
# <a name="create-an-iot-central-application"></a>Creación de una aplicación de IoT Central

Para crear una aplicación de IoT Central tiene varias opciones. Puede usar uno de los métodos basados en interfaz gráfica de usuario si prefiere un enfoque manual, o uno de los métodos mediante programación o la CLI si quiere automatizar el proceso.

Sea cual sea el enfoque que elija, las opciones de configuración son las mismas y el proceso normalmente tarda menos de un minuto en completarse.

[!INCLUDE [Warning About Access Required](../../../includes/iot-central-warning-contribitorrequireaccess.md)]

## <a name="options"></a>Opciones

En esta sección se describen las opciones disponibles al crear una aplicación de IoT Central. Según el método que elija, es posible que tenga que proporcionar las opciones en un formulario o como parámetros de una línea de comandos:

### <a name="pricing-plans"></a>Panes de tarifa

El plan *Gratuito* le permite crear una aplicación de IoT Central para realizar una prueba durante siete días. El plan gratuito:

- No requiere una suscripción a Azure.
- Solo se puede crear y administrar en el sitio de [Azure IoT Central](https://aka.ms/iotcentral).
- Permite conectar hasta cinco dispositivos.
- Se puede actualizar a un plan estándar si quiere conservar la aplicación.

Los planes *estándar*:

- Requieren una suscripción a Azure. Debe tener como mínimo acceso de **colaborador** en la suscripción a Azure. Si creó la suscripción usted mismo, automáticamente será un administrador con acceso suficiente. Para más información, consulte [¿Qué es el control de acceso basado en rol de Azure?](../../role-based-access-control/overview.md)
- Permiten crear y administrar aplicaciones de IoT Central mediante cualquiera de los métodos disponibles.
- Permiten conectar tantos dispositivos como necesite. Se le facturará por dispositivo. Para saber más, consulte [Precios de Azure IoT Central](https://azure.microsoft.com/pricing/details/iot-central/).
- No se pueden degradar a un plan gratuito, pero se pueden actualizar o degradar a otros planes estándar.

En la tabla siguiente se resumen las diferencias entre los tres planes estándar:

| Nombre del plan | Dispositivos gratuitos | Mensajes/mes | Caso de uso |
| --------- | ------------ | -------------- | -------- |
| S0        | 2            | 400            | Unos cuantos mensajes al día |
| S1        | 2            | 5\.000          | Unos cuantos mensajes por hora |
| S2        | 2            | 30,000         | Mensajes cada pocos minutos |

Para más información, consulte [Administrar la factura en una aplicación de IoT Central](howto-view-bill.md).

### <a name="application-name"></a>Nombre de la aplicación

El _nombre de la aplicación_ que elija aparece en la barra de título de cada página de la aplicación de IoT Central. También aparece en el icono de la aplicación en la página **Aplicaciones** del sitio de [Azure IoT Central](https://aka.ms/iotcentral).

El _subdominio_ que elija identifica de manera única la aplicación. El subdominio forma parte de la URL que se usa para acceder a la aplicación. La URL de una aplicación de IoT Central es similar a `https://yoursubdomain.azureiotcentral.com`.

### <a name="application-template-id"></a>Identificador de plantilla de la aplicación

La plantilla de aplicación que elija determina el contenido inicial de la aplicación, como paneles y plantillas de dispositivo. Para una aplicación personalizada, use `iotc-pnp-preview` como identificador de plantilla.

Para más información sobre las plantillas de aplicación personalizadas y centradas en el sector, consulte [¿Qué son las plantillas de aplicación?](concepts-app-templates.md)

### <a name="billing-information"></a>Información de facturación

Si elige uno de los planes estándar, debe proporcionar información de facturación:

- La suscripción a Azure que está usando.
- El directorio que contiene la suscripción que está usando.
- La ubicación en la que se hospedará la aplicación. IoT Central usa zonas geográficas de Azure como ubicaciones: Estados Unidos, Europa, Asia Pacífico, Australia, Reino Unido o Japón.

## <a name="azure-iot-central-site"></a>Sitio de Azure IoT Central

La manera más fácil de empezar a crear aplicaciones de IoT Central es en el sitio de [Azure IoT Central](https://aka.ms/iotcentral).

La sección [Compilación](https://apps.azureiotcentral.com/build) le permite seleccionar la plantilla de aplicación que quiere usar:

:::image type="content" source="media/howto-create-iot-central-application/choose-template.png" alt-text="Captura de pantalla de la página Compilación que le permite elegir una plantilla de aplicación.":::

Si selecciona **Crear aplicación**, puede proporcionar la información necesaria para crear una aplicación a partir de la plantilla:

:::image type="content" source="media/howto-create-iot-central-application/create-application.png" alt-text="Captura de pantalla que muestra la página de creación de aplicación de IoT Central.":::

La página **Aplicaciones** enumera todas las aplicaciones de IoT Central a las que tiene acceso. La lista incluye las aplicaciones que ha creado y las aplicaciones a las que se le ha concedido acceso.

> [!TIP]
> Todas las aplicaciones que cree mediante un plan de precios estándar en el sitio de Azure IoT Central usan el grupo de recursos **IOTC** de su suscripción. Los enfoques que se describen en la siguiente sección le permiten elegir un grupo de recursos para usarlo.

## <a name="other-approaches"></a>Otros enfoques

También puede usar los siguientes enfoques para crear una aplicación de IoT Central:

- [Creación de una aplicación de IoT Central desde Azure Portal](howto-manage-iot-central-from-portal.md#create-iot-central-applications)
- [Creación de una aplicación de IoT Central mediante la CLI de Azure](howto-manage-iot-central-from-cli.md#create-an-application)
- [Creación de una aplicación de IoT Central mediante PowerShell](howto-manage-iot-central-from-powershell.md#create-an-application)
- [Creación de una aplicación de IoT Central mediante programación](howto-manage-iot-central-programmatically.md)

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha aprendido a administrar aplicaciones de Azure IoT Central desde la CLI de Azure, le sugerimos el paso siguiente:

> [!div class="nextstepaction"]
> [Administrar su aplicación](howto-administer.md)
