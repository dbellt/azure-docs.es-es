---
title: 'Tutorial: Creación de una aplicación de supervisión continua de pacientes con Azure IoT Central | Microsoft Docs'
description: En este tutorial aprenderá a crear una aplicación de supervisión continua de pacientes mediante plantillas de aplicación de Azure IoT Central.
author: philmea
ms.author: philmea
ms.date: 09/24/2019
ms.topic: tutorial
ms.service: iot-central
services: iot-central
manager: eliotgra
ms.openlocfilehash: 07cd77eb5546143936af1fc963f0212112fc6eb7
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108743370"
---
# <a name="tutorial-deploy-and-walkthrough-a-continuous-patient-monitoring-app-template"></a>Tutorial: Implementación y tutorial de una plantilla de aplicación de supervisión continua de pacientes

En este tutorial se muestra cómo puede empezar a implementar una plantilla de aplicación de supervisión continua de pacientes de IoT Central. Aprenderá a implementar y usar la plantilla.

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Elija una plantilla de aplicación.
> * Recorrido por la plantilla de aplicación

## <a name="prerequisites"></a>Prerrequisitos

Se recomienda una suscripción de Azure. Como alternativa, puede usar una versión de prueba gratuita de 7 días. Si no tiene una suscripción de Azure, puede crear una en la [página de suscripción a Azure](https://aka.ms/createazuresubscription).

## <a name="create-an-application-template"></a>Elija una plantilla de aplicación.

Vaya al [sitio web del administrador de aplicaciones de Azure IoT Central](https://apps.azureiotcentral.com/). Seleccione **Build** (Crear) en la barra de navegación de la izquierda y, a continuación, seleccione la pestaña **Healthcare** (Asistencia sanitaria).

:::image type="content" source="media/app-manager-health.png" alt-text="Plantilla de aplicación de asistencia sanitaria":::

Seleccione el botón **Create app** (Crear aplicación) para empezar a crear la aplicación y, a continuación, inicie sesión con una cuenta personal, profesional o educativa de Microsoft. Esto le llevará a la página **Nueva aplicación**.

![Crear aplicación de asistencia sanitaria](media/app-manager-health-create.png)

![Crear información de facturación de una aplicación de asistencia sanitaria](media/app-manager-health-create-billinginfo.png)

Para crear la aplicación:

1. Azure IoT Central sugiere automáticamente un nombre de aplicación basado en la plantilla que haya seleccionado. Puede aceptar este nombre o escribir su propio nombre descriptivo de aplicación, como **Supervisión continua de pacientes**. Azure IoT Central también genera un prefijo de dirección URL único, en función del nombre de la aplicación. Cambiar si lo desea este prefijo de dirección URL por algo más fácil de recordar.

2. Puede elegir si desea crear la aplicación mediante el plan de precios *gratuito* o mediante uno de los planes de precios *estándar*. Las aplicaciones que crea mediante el plan de precios gratuito no tienen costo alguno durante siete días y luego expiran, y permiten hasta cinco dispositivos gratuitos. Puede cambiar una aplicación del plan de precios gratuito a un plan estándar en cualquier momento antes de que expire. Si elige el plan de precios gratuito, debe escribir su información de contacto y elegir si desea recibir información y sugerencias de Microsoft. Las aplicaciones que crea mediante un plan de tarifa estándar admiten hasta dos dispositivos gratuitos y requieren que se incluya la información de suscripción de Azure para la facturación.

3. Seleccione **Crear** en la parte inferior de la página para implementar la aplicación.

## <a name="walk-through-the-application-template"></a>Recorrido por la plantilla de aplicación

### <a name="dashboards"></a>Paneles

Después de implementar la plantilla de la aplicación, irá en primer lugar al **panel de supervisión de pacientes in situ de Lamna**. Lamna Healthcare es un sistema hospitalario ficticio que contiene dos hospitales: de Woodgrove y de Burkville. En el panel del operador del Hospital de Woodgrove, puede:

* Consultar la telemetría de los dispositivos y propiedades como el **nivel de batería** del dispositivo o su estado de **conectividad**.

* Vea el **plan de las plantas** y la ubicación para el dispositivo inteligente de revisión de constantes vitales (Smart Vitals Patch).

* **Vuelva a aprovisionarlo** para un nuevo paciente.

* Vea un ejemplo del **panel de proveedor** que un equipo de atención hospitalaria podría ver para realizar el seguimiento de sus pacientes.

* Cambie el **estado del paciente** del dispositivo para indicar si se está usando para un escenario remoto o con pacientes in situ.

:::image type="content" source="media/lamna-in-patient.png" alt-text="Estado de paciente in situ":::

También puede seleccionar **Go to remote patient dashboard** (Ir al panel de pacientes remotos) para ver el panel del operador del Hospital de Burkville. Este panel contiene un conjunto similar de acciones, telemetría e información. También puede ver varios dispositivos en uso y elegir **actualizar el firmware** de cada uno.

:::image type="content" source="media/lamna-remote.png" alt-text="Panel del operador remoto":::

### <a name="device-templates"></a>Plantillas de dispositivo

Si selecciona **Device templates**, (Plantillas de dispositivo) verá los dos tipos de dispositivo en la plantilla:

* **Smart Vitals Patch** (Revisión inteligente de constantes vitales): este dispositivo representa un parche que mide varias constantes vitales. Se utiliza para supervisar pacientes dentro y fuera del hospital. Si selecciona la plantilla, verá que, además de enviar datos del dispositivo, como el nivel de batería y la temperatura del dispositivo, el parche también envía datos del estado de los pacientes, como la frecuencia respiratoria y la presión arterial.

* **Smart Knee Brace** (Rodillera inteligente): este dispositivo representa una rodillera que los pacientes utilizan al recuperarse de una cirugía de reemplazo de una rodilla. Si selecciona esta plantilla, verá funcionalidades como los datos del dispositivo, el intervalo de movimiento y la aceleración.

:::image type="content" source="media/smart-vitals-device-template.png" alt-text="Plantilla de parche inteligente":::

### <a name="device-groups"></a>Grupos de dispositivos

Utilice grupos de dispositivos para agrupar lógicamente un conjunto de dispositivos para así realizar consultas u operaciones masivas sobre ellos.

Si selecciona la pestaña de grupos de dispositivos, verá un grupo de dispositivos predeterminado para cada plantilla de dispositivo de la aplicación. También se han creado dos grupos de dispositivos de ejemplo adicionales, llamados **Provision devices** (Aprovisionamiento de dispositivos) y **Devices with outdated firmware** (Dispositivos con firmware sin actualizar). Puede usar estos grupos de dispositivos de ejemplo como entradas para ejecutar algunos [Trabajos](#jobs) de la aplicación.

### <a name="rules"></a>Reglas

Si selecciona **Rules** (Reglas), verá las tres reglas en la plantilla:

* **Brace temperature high**(Temperatura alta de rodillera): esta regla se desencadena cuando la temperatura del dispositivo de la rodillera inteligente es mayor de 95&deg;F en un período de 5 minutos. Utilice esta regla para avisar al equipo de asistencia sanitaria y al paciente, y enfriar el dispositivo de forma remota.

* **Fall detected** (Caída detectada): esta regla se desencadena si se detecta que un paciente ha tenido una caída. Use esta regla para configurar una acción con el fin de implementar un equipo operativo para ayudar al paciente que se ha caído.

* **Patch battery low** (Revisión de poca batería): esta regla se desencadena cuando el nivel de batería del dispositivo desciende por debajo del 10 %. Utilice esta regla para desencadenar una notificación al paciente para que cargue su dispositivo.

:::image type="content" source="media/brace-temp-rule.png" alt-text="Reglas":::

### <a name="jobs"></a>Trabajos

Los trabajos le permiten ejecutar operaciones masivas en un conjunto de dispositivos mediante el uso de [grupos de dispositivos](#device-groups) como entrada. La plantilla de aplicación tiene dos trabajos de ejemplo que un operador puede ejecutar:

* **Update knee brace firmware** (Actualización del firmware de la rodillera): este trabajo busca dispositivos en el grupo de dispositivos **Devices with outdated firmware** (Dispositivos con firmware sin actualizar) y ejecuta un comando para actualizar dichos dispositivos a la versión de firmware más reciente. En este trabajo de ejemplo se supone que los dispositivos pueden controlar el comando **update** y recuperar los archivos de firmware desde la nube.  

* **Re-provision devices** (Reaprovisionamiento de dispositivos): Tiene un conjunto de dispositivos que se han devuelto recientemente al hospital. Este trabajo busca dispositivos en el grupo de dispositivos **Provision devices** (Aprovisionamiento de dispositivos) y ejecuta un comando para volver a aprovisionarlos para el siguiente conjunto de pacientes.

### <a name="devices"></a>Dispositivos

Seleccione la pestaña **Devices**  (Dispositivos) y, a continuación, seleccione una instancia del dispositivo **Smart Knee Brace** (Rodillera inteligente). Hay tres vistas para explorar la información sobre el dispositivo específico que ha seleccionado. Estas vistas se crean y publican al crear la plantilla de dispositivo para el dispositivo. Por lo tanto, estas vistas serán coherentes en todos los dispositivos que se conecten o simulen.

La vista **Dashboard** (Panel) ofrece información general sobre la telemetría y las propiedades del dispositivo orientada a los operadores.

La pestaña **Properties** (Propiedades) le permite editar las propiedades de la nube y leer y escribir las propiedades del dispositivo.

La pestaña **Commands** (Comandos) le permite ejecutar comandos en el dispositivo.

:::image type="content" source="media/knee-brace-dashboard.png" alt-text="Panel de la rodillera inteligente":::

### <a name="data-export"></a>Exportación de datos

La exportación de datos permite exportar los datos del dispositivo de forma continua a otros servicios de Azure, incluido [Azure API for FHIR](concept-continuous-patient-monitoring-architecture.md#export-to-azure-api-for-fhir).

## <a name="clean-up-resources"></a>Limpieza de recursos

Si no va a seguir usando esta aplicación, elimine su plantilla. Para ello, vaya a **Administración > Configuración de la aplicación** y haga clic en **Eliminar**.

:::image type="content" source="media/admin-delete.png" alt-text="Recursos ordenados":::

## <a name="next-steps"></a>Pasos siguientes

Vaya al siguiente artículo para aprender a crear un panel de proveedor que se conecte a la aplicación IoT Central.

> [!div class="nextstepaction"]
> [Creación de un panel de proveedor](tutorial-health-data-triage.md)