---
title: Guía del administrador de Azure IoT Central
description: Azure IoT Central es una plataforma de aplicaciones de IoT que simplifica la creación de soluciones de IoT. En este artículo se proporciona información general el rol del administrador en IoT Central.
author: philmea
ms.author: philmea
ms.date: 03/25/2021
ms.topic: conceptual
ms.service: iot-central
services: iot-central
ms.custom: mvc
ms.openlocfilehash: 658ebe027565a3acaf427a7b3dbf3963701069d8
ms.sourcegitcommit: 2aeb2c41fd22a02552ff871479124b567fa4463c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107868652"
---
# <a name="iot-central-administrator-guide"></a>Guía del administrador de IoT Central

*Este artículo está destinado a los administradores*.

Una aplicación IoT Central le permite supervisar y administrar millones de dispositivos a lo largo de su ciclo de vida. Esta guía está orientada a administradores que administran aplicaciones IoT Central.

En IoT Central, un administrador:

- Administra usuarios y roles en la aplicación.
- Administra la seguridad como la autenticación de dispositivos.
- Configuración de la aplicación.
- Actualiza las aplicaciones.
- Exporta y comparte aplicaciones.
- Supervisión del estado de la aplicación.

## <a name="users-and-roles"></a>Usuarios y roles

IoT Central usa un sistema de control de acceso basado en roles para administrar los permisos de usuario dentro de una aplicación. IoT Central tiene tres roles integrados para administradores, generadores de soluciones y operadores. Un administrador puede crear roles personalizados con conjuntos de permisos específicos. Un administrador es responsable de agregar los usuarios a una aplicación y asignarlos a los roles.

Para más información, consulte [Administrar usuarios y roles en la aplicación de IoT Central](howto-manage-users-roles.md).

## <a name="application-security"></a>Seguridad de las aplicaciones

Los dispositivos que se conectan a la aplicación IoT Central suelen usar certificados X.509 o firmas de acceso compartido (SAS) como credenciales. El administrador administra los certificados de grupo o las claves de las que se derivan las credenciales del dispositivo.

Para obtener más información, consulte [inscripción de grupos X.509](concepts-get-connected.md#x509-group-enrollment), [inscripción de grupos SAS](concepts-get-connected.md#sas-group-enrollment)y [Cómo poner en funcionamiento los certificados de dispositivo X.509](how-to-roll-x509-certificates.md).

Además, el administrador puede crear y administrar los tokens de API que utiliza una aplicación cliente para autenticarse con la aplicación IoT Central. Las aplicaciones cliente usan la API de REST para interactuar con IoT Central.

## <a name="configure-an-application"></a>Configuración de una aplicación

El administrador puede configurar el comportamiento y la apariencia de una aplicación IoT Central. Para obtener más información, consulte:

- [Cambio del nombre y la dirección URL de la aplicación](howto-administer.md#change-application-name-and-url)
- [Personalización de la interfaz de usuario](howto-customize-ui.md)
- [Traslado de una aplicación a un plan de precios diferente](howto-view-bill.md)
- [Configurar cargas de archivos](howto-configure-file-uploads.md)

## <a name="export-an-application"></a>Exportar una aplicación

Un administrador puede:

- Crear una copia de la aplicación si solo necesita crear una copia duplicada de ella. Por ejemplo, puede que necesite una copia duplicada para las pruebas.
- Crear una plantilla de aplicación a partir de una aplicación existente si tiene previsto crear varias copias.

Para más información, consulte [Exportar la aplicación de Azure IOT](howto-use-app-templates.md).

## <a name="migrate-to-a-new-version"></a>Migrar a una nueva versión

Un administrador puede migrar una aplicación a una versión más reciente. Actualmente, una aplicación recién creada es una aplicación V3. Es posible que un administrador tenga que migrar una aplicación V2 a una aplicación V3.

Para más información, consulte [Migrar aplicación de IOT central V2 a V3](howto-migrate.md).

## <a name="monitor-application-health"></a>Supervisión del estado de la aplicación

Un administrador puede usar métricas IoT Central para evaluar el estado de los dispositivos conectados y el estado de ejecución de las exportaciones de datos.

Para ver las métricas, un administrador puede usar gráficos en el Azure Portal, una API de REST o consultas de PowerShell o CLI de Azure.

Para más información, vea [Supervisión del estado general de una aplicación de IoT Central](howto-monitor-application-health.md).

## <a name="tools"></a>Herramientas

Muchas de las herramientas que se usan como administrador están disponibles en la sección **Administración** de cada aplicación IOT central. También puede usar las siguientes herramientas para realizar algunas tareas administrativas:

- [CLI de Azure](howto-manage-iot-central-from-cli.md)
- [Azure PowerShell](howto-manage-iot-central-from-powershell.md)
- [Azure Portal](howto-manage-iot-central-from-portal.md)

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha aprendido a administrar la aplicación de Azure IoT Central, el siguiente paso sugerido es aprender a [administrar usuarios y roles](howto-manage-users-roles.md) en Azure IoT Central.
