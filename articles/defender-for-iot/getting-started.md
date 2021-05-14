---
title: 'Inicio rápido: Introducción'
description: En este inicio rápido, aprenderá a usar el flujo de trabajo básico para la implementación de Defender para IoT.
ms.topic: quickstart
ms.date: 04/17/2021
ms.openlocfilehash: b1e7686e1d68d5a3f239320930d69f22c78e13cb
ms.sourcegitcommit: 6686a3d8d8b7c8a582d6c40b60232a33798067be
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107750453"
---
# <a name="quickstart-get-started-with-defender-for-iot"></a>Inicio rápido: Introducción a Defender para IoT

En este artículo se proporciona información general de los pasos para configurar Azure Defender para IoT. El proceso requiere que realice lo siguiente:

- Registre la suscripción y los sensores en el portal de Azure Defender para IoT.
- Instale el software de la consola de administración local y el sensor.
- Realice la activación inicial del sensor y la consola de administración.

## <a name="permission-requirements"></a>Requisitos de permisos

Algunos de los pasos de configuración requieren permisos de usuario específicos.

Los permisos de usuario administrativo son necesarios para activar el sensor y la consola de administración, cargar certificados SSL/TLS y generar nuevas contraseñas.

En la tabla siguiente se describen los permisos de acceso de usuarios para las herramientas del portal de Azure Defender para IoT:

| Permiso | Lector de seguridad | Administrador de seguridad | Colaborador de la suscripción | Propietario de la suscripción |
|--|--|--|--|--|
| Visualización de detalles y acceso a software, archivos de activación y paquetes de inteligencia sobre amenazas  | ✓ | ✓ | ✓ | ✓ |
| Incorporación de sensores  |  |  ✓ | ✓ | ✓ |
| Actualizar precios  |  |  ✓ | ✓ | ✓ |
| Recuperar contraseña  | ✓  |  ✓ | ✓ | ✓ |

## <a name="identify-the-solution-infrastructure"></a>Identificación de la infraestructura de la solución

**Aclaración de las necesidades de configuración de red**

Investigue la arquitectura de redes, el ancho de banda supervisado y otros detalles de la red. Para más información, vea [Acerca de la configuración de red de Azure Defender para IoT](how-to-set-up-your-network.md).

**Aclaración de qué sensores y dispositivos de la consola de administración son necesarios para controlar la carga de red**

Azure Defender para IoT admite implementaciones físicas y virtuales. En el caso de las implementaciones físicas, puede comprar varios dispositivos certificados. Para más información, vea [Identificación de los dispositivos necesarios](how-to-identify-required-appliances.md).

Se recomienda calcular el número aproximado de dispositivos que se van a supervisar. Más adelante, cuando registre su suscripción de Azure en el portal, se le pedirá que escriba este número. Los números se pueden agregar en intervalos de 1000 segundos. El número de dispositivos supervisados se denomina *dispositivos confirmados*.

## <a name="register-with-azure-defender-for-iot"></a>Registro en Azure Defender para IoT

El registro incluye:

- Incorporación de las suscripciones de Azure a Defender para IoT
- Definición de los dispositivos confirmados
- Descarga de un archivo de activación para la consola de administración local

Para registrarse:

1. Vaya al portal de Azure Defender para IoT.

1. Seleccione **Onboard subscription** (Incorporar suscripción).

1. En la página **Precios**, seleccione una suscripción o cree una y agregue el número de dispositivos confirmados.

1. Seleccione la pestaña **Download the on-premises management console** (Descargar la consola de administración local) y guarde el archivo de activación descargado. Este archivo contiene los dispositivos confirmados agregados que ha definido. El archivo se cargará en la consola de administración después del inicio de sesión inicial.

Para obtener información sobre cómo retirar una suscripción, consulte [Retirada de una suscripción](how-to-manage-subscriptions.md#offboard-a-subscription).

## <a name="install-and-set-up-the-on-premises-management-console"></a>Instalación y configuración de la consola de administración local

Después de adquirir su dispositivo de la consola de administración local:

- Descargue el paquete ISO del portal de Azure Defender para IoT.
- Instale el software.
- Active y lleve a cabo la configuración inicial de la consola de administración.

Para instalar y configurar:

1. Seleccione **Introducción** en el portal de Defender para IoT.
1. Seleccione la pestaña **On-premises management console** (Consola de administración local).
1. Elija una versión y seleccione **Descargar**.
1. Instale el software de la consola de administración local. Para más información, vea [Instalación de Defender para IoT](how-to-install-software.md).
1. Active y configure la consola de administración. Para más información, vea [Activación y configuración de la consola de administración local](how-to-activate-and-set-up-your-on-premises-management-console.md).

## <a name="onboard-a-sensor"></a>Incorporación de sensores ##

Incorpore un sensor; para ello, debe registrarlo en Azure Defender para IoT y descargar un archivo de activación de sensor:

1. Defina un nombre para el sensor y asócielo con una suscripción.
1. Elija un modo de conexión del sensor:

   - **Sensores conectados a la nube**: la información que detectan los sensores se muestra en la consola del sensor. Además, se proporciona información de alertas mediante un centro de IoT, que puede compartirse con otros servicios de Azure, como Azure Sentinel.  También puede elegir insertar automáticamente paquetes de inteligencia sobre amenazas desde el portal de Azure Defender para IoT a los sensores. Para más información, consulte [Investigación y paquetes de inteligencia sobre amenazas](how-to-work-with-threat-intelligence-packages.md).

   - **Locally managed sensors** (Sensores administrados de forma local): la información que detectan los sensores se muestra en la consola del sensor. Si trabaja en una red aislada y quiere una vista unificada de toda la información detectada por varios sensores administrados de forma local, trabaje con la consola de administración local.

1. Descargue un archivo de activación del sensor.

Para más información sobre la incorporación, consulte [Incorporación y administración de sensores en el portal de Defender para IoT](how-to-manage-sensors-on-the-cloud.md).

## <a name="install-and-set-up-the-sensor"></a>Instalación y configuración del sensor

Descargue el paquete ISO del portal de Azure Defender para IoT, instale el software y configure el sensor.

1. Seleccione **Introducción** en el portal de Defender para IoT.

1. Seleccione **Set up sensor** (Configurar sensor).

1. Elija una versión y seleccione **Descargar**.

1. Instale el software del sensor. Para más información, vea [Instalación de Defender para IoT](how-to-install-software.md).

1. Active y configure el sensor. Para más información, vea [Inicio de sesión y activación de un sensor](how-to-activate-and-set-up-your-sensor.md).

## <a name="connect-sensors-to-an-on-premises-management-console"></a>Conecte los sensores a una consola de administración local.

Conecte los sensores a una consola de administración para garantizar lo siguiente:

- Los sensores envían información de inventario de alertas y dispositivos a la consola de administración local.

- La consola de administración local puede realizar copias de seguridad de los sensores, administrar las alertas que los sensores detectan, investigar desconexiones de los sensores y llevar a cabo otras actividades en sensores conectados.

Se recomienda agrupar varios sensores que supervisan las mismas redes en una zona. Al hacerlo, se fusionará la información recopilada por varios sensores.

Para más información, vea [Conexión de los sensores a la consola de administración local](how-to-activate-and-set-up-your-on-premises-management-console.md#connect-sensors-to-the-on-premises-management-console).

## <a name="populate-azure-sentinel-with-alert-information-optional"></a>Rellenar Azure Sentinel con la información de alertas (opcional)

Configure Azure Sentinel para enviar ahí la información sobre las alertas. Vea [Conexión de los datos de Defender para IoT con Azure Sentinel](how-to-configure-with-sentinel.md).

## <a name="next-steps"></a>Pasos siguientes ##

[Bienvenido a Azure Defender para IoT](overview.md)

[Arquitectura de Azure Defender para IoT](architecture.md)
