---
title: Actualización de datos de inteligencia sobre amenazas
description: El paquete de datos de inteligencia sobre amenazas se proporciona con cada nueva versión de Defender para IoT o, si es necesario, entre versiones.
ms.date: 04/17/2021
ms.topic: how-to
ms.openlocfilehash: bb38d0a2486bda336d6881ec6f4c5d680906c973
ms.sourcegitcommit: 6686a3d8d8b7c8a582d6c40b60232a33798067be
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107750471"
---
# <a name="threat-intelligence-research-and-packages"></a>Investigación y paquetes de inteligencia sobre amenazas #
## <a name="overview"></a>Información general ##

Los equipos de seguridad de Microsoft llevan a cabo operaciones de inteligencia sobre amenazas e investigación de vulnerabilidades de ICS. Estos equipos incluyen MSTIC (Microsoft Threat Intelligence Center, Centro de inteligencia sobre amenazas de Microsoft), DART (Microsoft Detection and Response Team, Equipo de detección y respuesta de Microsoft), DCU (Digital Crimes Unit, Unidad de delitos digitales) y la sección 52 (expertos del dominio de IoT/OT/ICS que realizan el seguimiento de malware, campañas y adversarios de ingeniería inversa de cero días específicos de ICS).

Los equipos proporcionan detección de seguridad, análisis y respuesta a los siguientes elementos de Microsoft:

- Infraestructura y servicios en la nube.
- Productos y dispositivos tradicionales.
- Recursos corporativos internos.

Los equipos de seguridad ofrecen las siguientes ventajas:

- Protección contra amenazas conocidas y pertinentes.
- Información que ayuda a evaluar y establecer las prioridades.
- Comprensión del contexto completo de las amenazas antes de que se produzcan.
- Datos más pertinentes, precisos y procesables.

Esta inteligencia proporciona información contextual para enriquecer el análisis de la plataforma de Microsoft y es compatible con los servicios administrados de la empresa para la respuesta a incidentes y la investigación de vulneraciones. Los paquetes de inteligencia sobre amenazas contienen firmas (incluidas las firmas de malware), CVE y otro contenido de seguridad.

## <a name="when-are-packages-delivered"></a>Cuándo se entregan los paquetes ##

Los paquetes de inteligencia sobre amenazas se proporcionan aproximadamente una vez al mes o incluso con más frecuencia, si es necesario. Los anuncios sobre los nuevos paquetes están disponibles en https://techcommunity.microsoft.com/t5/azure-defender-for-iot/bd-p/AzureDefenderIoT. 

## <a name="update-threat-intelligence-packages-to-your-sensors"></a>Actualización de paquetes de inteligencia sobre amenazas en los sensores ##

Hay tres opciones disponibles para actualizar los paquetes de inteligencia sobre amenazas en los sensores:

- Insertar los paquetes en los sensores de forma automática cuando Defender para IoT los entregue.
- Insertar manualmente el paquete de inteligencia sobre amenazas en los sensores según se requiera.
- Descargar un paquete y, a continuación, cargarlo en uno o varios sensores.

Los usuarios que tengan permisos de Lector de seguridad de Defender para IoT pueden insertar paquetes en los sensores de forma tanto automática como manual.

### <a name="automatically-push-threat-intelligence-updates-to-sensors"></a>Inserción automática de actualizaciones de inteligencia sobre amenazas en los sensores ###

Los paquetes de inteligencia sobre amenazas pueden actualizarse automáticamente en los sensores *conectados a la nube* cuando Defender para IoT los publique. Asegúrese de que se realice la actualización automática de los paquetes mediante la incorporación del sensor conectado a la nube con la opción **Automatic Threat Intelligence Updates** (Actualizaciones automáticas de inteligencia sobre amenazas) habilitada. Para obtener más información, consulte [Incorporación de sensores](getting-started.md#onboard-a-sensor).

### <a name="manually-push-threat-intelligence-updates-to-sensors"></a>Inserción manual de actualizaciones de inteligencia sobre amenazas en los sensores ###

Los sensores *conectados a la nube* pueden actualizarse de forma automática con los paquetes de inteligencia sobre amenazas. Sin embargo, puede adoptar un enfoque más conservador e insertar los paquetes en los sensores desde el portal de Azure Defender para IoT solo cuando lo considere necesario.
Esta opción le permite controlar cuándo se instala un paquete, sin necesidad de descargarlo y luego cargarlo en los sensores.

**Para insertar los paquetes de forma manual:**

1. Vaya a la página **Sites and Sensors** (Sitios y sensores) de Azure Defender para IoT.
1. Seleccione los puntos suspensivos (...) de un sensor y, a continuación, seleccione **Push Threat Intelligence update** (Insertar actualización de inteligencia sobre amenazas). El campo **Threat Intelligence update status** (Estado de actualización de inteligencia sobre amenazas) muestra el progreso de la actualización.

#### <a name="change-the-threat-intelligence-update-mode"></a>Cambio del modo de actualización de la inteligencia sobre amenazas ####

Puede cambiar el modo de actualización de la inteligencia sobre amenazas del sensor después de la incorporación inicial.

**Para cambiar el modo de actualización:**

1. Seleccione los puntos suspensivos (...) de un sensor y, a continuación, elija **Edit** (Editar).
1. Habilite o deshabilite el botón de alternancia **Automatic Threat Intelligence Updates** (Actualizaciones automáticas de inteligencia sobre amenazas).

### <a name="download-packages-and-upload-to-sensors"></a>Descarga de paquetes y carga en los sensores ###

Los paquetes pueden descargarse desde el portal de Defender para IoT y cargarse manualmente en sensores individuales. Si la consola de administración local administra los sensores, puede descargar los paquetes de inteligencia sobre amenazas en la consola de administración e incluirlos en varios sensores simultáneamente.

:::image type="content" source="media/how-to-work-with-threat-intelligence-packages/download-screen.png" alt-text="Descarga de actualizaciones desde el portal de Azure Defender para IoT.":::

Esta opción está disponible para los sensores tanto *conectados a la nube* como *administrados localmente*.

**Para cargar en un solo sensor:**

1. Vaya a la página **Updates** (Actualizaciones) de Azure Defender para IoT.

2. Descargue y guarde el paquete de **Threat Intelligence** (Inteligencia sobre amenazas).

3. Inicie sesión en la consola del sensor.

4. En el menú lateral, seleccione **System Settings** (Configuración del sistema).

5. Seleccione **Threat Intelligence Data** (Datos de inteligencia sobre amenazas) y, a continuación, seleccione **Update** (Actualizar).

6. Cargue el nuevo paquete.

**Para cargar en varios sensores simultáneamente:**

1. Vaya a la página **Updates** (Actualizaciones) de Azure Defender para IoT.

2. Descargue y guarde el paquete de **Threat Intelligence** (Inteligencia sobre amenazas).

3. Inicie sesión en la consola de administración.

4. En el menú lateral, seleccione **System Settings** (Configuración del sistema).

5. En la sección **Sensor Engine Configuration** (Configuración del motor de sensores), seleccione los sensores que deben recibir los paquetes actualizados.  

6. En la sección **Select Threat Intelligence Data** (Seleccionar datos de inteligencia sobre amenazas), seleccione el signo más ( **+** ).

7. Cargar el paquete.

## <a name="review-package-update-status-on-the-sensor"></a>Revisión del estado de actualización del paquete en el sensor ##

El estado de actualización y la información de versión del paquete se muestran en la sección **System Settings** (Configuración del sistema), **Threat Intelligence** (Inteligencia sobre amenazas) del sensor.  

## <a name="review-package-information-for-cloud-connected-sensors"></a>Revisión de la información del paquete para los sensores conectados a la nube ##

Revise la información siguiente de los paquetes de inteligencia sobre amenazas para los sensores conectados a la nube:

- Versión instalada del paquete
- Modo de actualización de inteligencia sobre amenazas
- Estado de la actualización de inteligencia sobre amenazas

Para revisar la información de inteligencia sobre amenazas:

1. Vaya a la página **Sites and Sensors** (Sitios y sensores) de Azure Defender para IoT.
1. Revise la **versión de Inteligencia sobre amenazas** instalada en cada sensor. La nomenclatura de la versión se basa en el día en el que Defender para IoT compiló el paquete.
1. Revise el **modo de inteligencia sobre amenazas**. *Automatic* (Automático) indica que los paquetes recién disponibles se instalarán de forma automática en los sensores cuando Defender para IoT los publique. *Manual* indica que puede insertar los paquetes recién disponibles directamente en los sensores según sea necesario.
1. Revise el **estado de la actualización de inteligencia sobre amenazas**. Pueden mostrarse los estados siguientes:

- Con error
- En curso
- Actualización disponible
- Aceptar

Si se produce un error en las actualizaciones de inteligencia sobre amenazas conectadas a la nube, revise la información de la conexión en las columnas **Sensor status** (Estado del sensor) y **Last connected UTC** (UTC de última conexión) en la página **Sites and Sensors** (Sitios y sensores). 

## <a name="see-also"></a>Consulte también

[Incorporación de sensores](getting-started.md#onboard-a-sensor)

[Administración de sensores desde la consola de administración](how-to-manage-sensors-from-the-on-premises-management-console.md)