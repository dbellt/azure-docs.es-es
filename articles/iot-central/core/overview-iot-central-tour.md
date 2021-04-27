---
title: Un paseo por la interfaz de usuario de Azure IoT Central | Microsoft Docs
description: Familiarícese con las áreas clave de la interfaz de usuario de Azure IoT Central que se usa para crear, administrar y usar la solución de IoT.
author: ankitscribbles
ms.author: ankitgup
ms.date: 02/09/2021
ms.topic: overview
ms.service: iot-central
services: iot-central
ms.custom: mvc
manager: corywink
ms.openlocfilehash: bf60f512416007137e71119fa7474b1393099ebf
ms.sourcegitcommit: 79c9c95e8a267abc677c8f3272cb9d7f9673a3d7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/19/2021
ms.locfileid: "107718888"
---
# <a name="take-a-tour-of-the-azure-iot-central-ui"></a>Un paseo por la interfaz de usuario de Azure IoT Central

Este artículo presenta la interfaz de usuario de Azure IoT Central. Puede usar la interfaz de usuario para crear, administrar y usar una aplicación de IoT Central y sus dispositivos conectados.

## <a name="iot-central-homepage"></a>Página principal de IoT Central

La [página de inicio de IoT Central](https://aka.ms/iotcentral-get-started) es el lugar donde puede obtener más información sobre las noticias y características más recientes disponibles en IoT Central, crear nuevas aplicaciones y ver e iniciar las aplicaciones existentes.

:::image type="content" source="media/overview-iot-central-tour/iot-central-homepage.png" alt-text="Página principal de IoT Central":::

### <a name="create-an-application"></a>Crear una aplicación

En la sección **Compilación** puede examinar la lista de plantillas de IoT Central importantes para el sector, o empezar desde cero con una plantilla de aplicación personalizada.  

:::image type="content" source="media/overview-iot-central-tour/iot-central-build.png" alt-text="Página de compilación de IoT Central":::

Para más información, consulte la guía de inicio rápido [Creación de una aplicación de Azure IoT Central](quick-deploy-iot-central.md).

### <a name="launch-your-application"></a>Iniciar la aplicación

Para iniciar la aplicación de IoT Central, vaya a la dirección URL que eligió durante la creación de la aplicación. También puede ver una lista de todas las aplicaciones a las que tiene acceso en el [administrador de aplicaciones de IoT Central](https://aka.ms/iotcentral-apps).

:::image type="content" source="media/overview-iot-central-tour/app-manager.png" alt-text="Administrador de aplicaciones IoT Central":::

## <a name="navigate-your-application"></a>Navegue por su aplicación

Una vez que esté dentro de la aplicación de IoT, use el panel izquierdo para acceder a las distintas características. Para expandir o contraer el panel izquierdo, seleccione el icono de tres líneas situado en la parte superior del panel:

> [!NOTE]
> Los elementos que se ven en el panel izquierdo dependen del rol de usuario. Más información sobre [administración de usuarios y roles](howto-manage-users-roles.md). 

:::row:::
  :::column span="":::
      :::image type="content" source="media/overview-iot-central-tour/navigation-bar.png" alt-text="panel izquierdo":::

  :::column-end:::
  :::column span="2":::
     La opción **Panel** muestra todos los paneles personales y de la aplicación. 
     
     **Dispositivos** permite administrar todos los dispositivos.

     **Grupos de dispositivos** permite ver y crear colecciones de dispositivos especificados por una consulta. Los grupos de dispositivos se usan a través de la aplicación para realizar operaciones masivas.

     Las **reglas** permiten crear y editar reglas para supervisar los dispositivos. Las reglas se evalúan en función de los datos del dispositivo y desencadenan acciones personalizables.

     **Análisis** ofrece funcionalidades enriquecidas para analizar tendencias históricas y correlacionar diferentes telemetrías de los dispositivos.

     **Trabajos** le permite administrar los dispositivos a escala mediante la ejecución de operaciones masivas.

     **Plantillas de dispositivo** le permite crear y administrar las características de los dispositivos que se conectan a la aplicación.

     La **exportación de datos** le permite configurar una exportación continua a servicios externos, como Storage y Queue.

     La **administración** le permite administrar la configuración de la aplicación, la personalización, la facturación, los usuarios y los roles.

     **Mis aplicaciones** permite volver al administrador de aplicaciones de IoT Central.
     
   :::column-end:::
:::row-end:::

### <a name="search-help-theme-and-support"></a>Búsqueda, ayuda, tema y soporte técnico

El menú superior aparece en todas las páginas:

:::image type="content" source="media/overview-iot-central-tour/toolbar.png" alt-text="Barra de herramientas de IoT Central":::

* Para buscar dispositivos, escriba un valor en **Buscar**.
* Para cambiar el idioma o el tema de la interfaz de usuario, elija el icono **Configuración**. Más información sobre [administración de las preferencias de la aplicación](howto-manage-preferences.md)
* Para obtener ayuda y soporte técnico, elija el menú desplegable **Ayuda** para obtener una lista de recursos. Puede [obtener información sobre la aplicación](./howto-get-app-info.md) en el vínculo **About your app** (Acerca de su aplicación). En una aplicación del plan de precios gratuito, los recursos de soporte técnico incluyen acceso al [chat en directo](howto-show-hide-chat.md).
* Para cerrar la sesión de la aplicación, elija el icono **Cuenta**.

Puede elegir entre un tema claro o un tema oscuro para la interfaz de usuario:

> [!NOTE]
> La opción de elegir entre temas claros y oscuros no está disponible si su administrador ha configurado un tema personalizado para la aplicación.

:::image type="content" source="media/overview-iot-central-tour/themes.png" alt-text="Captura de pantalla de la elección de un tema en IoT Central.":::

### <a name="dashboard"></a>Panel

:::image type="content" source="Media/overview-iot-central-tour/dashboard.png" alt-text="Captura de pantalla del panel de IoT Central.":::

* **Panel** es la primera página que verá cuando inicie sesión en la aplicación de IoT Central. Puede crear y personalizar varios paneles de aplicaciones. Más información sobre [agregar iconos al panel](howto-add-tiles-to-your-dashboard.md)

* También se pueden crear paneles personales para supervisar lo que le interese. Para más información, consulte el artículo [Creación de paneles personales de Azure IoT Central](howto-create-personal-dashboards.md).

### <a name="devices"></a>Dispositivos

:::image type="content" source="Media/overview-iot-central-tour/devices.png" alt-text="Captura de pantalla de la página de dispositivos.":::

Esta página muestra los dispositivos que hay en la aplicación de IoT Central agrupados por _plantilla del dispositivo_.

* Una plantilla de dispositivo define un tipo de dispositivo que se puede conectar a la aplicación.
* Un dispositivo representa un dispositivo real o simulado en la aplicación.

Para más información, consulte la guía de inicio rápido [Supervisar los dispositivos](./quick-monitor-devices.md). 

### <a name="device-groups"></a>Grupos de dispositivos

:::image type="content" source="Media/overview-iot-central-tour/device-groups.png" alt-text="Página Grupo de dispositivos":::

Esta página le permite crear y ver grupos de dispositivos en la aplicación de IoT Central. Puede usar grupos de dispositivos para realizar operaciones masivas en la aplicación o analizar datos. Para más información, consulte el artículo [Uso de grupos de dispositivos en la aplicación de Azure IoT Central](tutorial-use-device-groups.md).

### <a name="rules"></a>Reglas
:::image type="content" source="Media/overview-iot-central-tour/rules.png" alt-text="Captura de pantalla de la página de reglas":::.

Esta página le permite ver y crear reglas basadas en los datos del dispositivo. Cuando se activa una regla, puede desencadenar una o varias acciones, como enviar un correo electrónico o invocar un webhook. Para más información, consulte el tutorial de [Configuración de reglas](tutorial-create-telemetry-rules.md).

### <a name="analytics"></a>Análisis

:::image type="content" source="Media/overview-iot-central-tour/analytics.png" alt-text="Captura de pantalla de análisis":::.

Análisis ofrece funcionalidades enriquecidas para analizar tendencias históricas y correlacionar diferentes telemetrías de los dispositivos. Para más información, consulte el artículo [Crear análisis para su aplicación de Azure IoT Central](howto-create-analytics.md).

### <a name="jobs"></a>Trabajos

:::image type="content" source="Media/overview-iot-central-tour/jobs.png" alt-text="Página de trabajos":::

Esta página le permite ver y crear trabajos que se pueden usar para operaciones de administración de dispositivos masivas en los dispositivos. Puede actualizar las propiedades del dispositivo, la configuración y ejecutar comandos en grupos de dispositivos. Para obtener más información, consulte el artículo [Run a job](howto-run-a-job.md) (Ejecución de un trabajo).

### <a name="device-templates"></a>Plantillas de dispositivo

:::image type="content" source="Media/overview-iot-central-tour/templates.png" alt-text="Captura de pantalla de Plantillas de dispositivo":::.

La página de plantillas de dispositivo le permite ver y crear plantillas de dispositivo en la aplicación. Para más información, consulte el tutorial [Define a new device type in your Azure IoT Central application](howto-set-up-template.md) (Definición de un nuevo tipo de dispositivo en la aplicación de Azure IoT Central).

### <a name="data-export"></a>Exportación de datos

:::image type="content" source="Media/overview-iot-central-tour/export.png" alt-text="Exportación de datos":::

La exportación de datos permite configurar flujos de datos a sistemas externos. Para más información, consulte el artículo [Exportación de datos a Azure IoT Central](./howto-export-data.md).

### <a name="administration"></a>Administración

:::image type="content" source="media/overview-iot-central-tour/administration.png" alt-text="Captura de pantalla de Administración de IoT":::.

La página de administración permite configurar y personalizar la aplicación IoT Central. Aquí puede cambiar el nombre de la aplicación, la dirección URL, las funciones, administrar usuarios y roles, crear tokens de API y exportar la aplicación. Para más información, consulte el artículo [Administer your Azure IoT Central application](howto-administer.md) (Administración de la aplicación de Azure IoT Central).

## <a name="next-steps"></a>Pasos siguientes

Ahora que ya tiene información general sobre Azure IoT Central y está familiarizado con el diseño de la interfaz de usuario, el siguiente paso sugerido es completar la guía de inicio rápido [Create an Azure IoT Central application](quick-deploy-iot-central.md) (Creación de una aplicación de Azure IoT Central).
