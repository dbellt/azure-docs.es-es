---
title: 'Tutorial: Configuración de un entorno de Gen2: Azure Time Series Insights Gen2 | Microsoft Docs'
description: 'Tutorial: Aprenda a configurar un entorno en Azure Time Series Insights Gen2.'
author: riserrad
ms.author: riserrad
manager: edett
ms.workload: big-data
ms.service: time-series-insights
services: time-series-insights
ms.topic: tutorial
ms.date: 04/23/2021
ms.custom: seodec18
ms.openlocfilehash: f3994802e82b126f426a75a26d399f6f86e21797
ms.sourcegitcommit: ad921e1cde8fb973f39c31d0b3f7f3c77495600f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/25/2021
ms.locfileid: "107952027"
---
# <a name="tutorial-set-up-an-azure-time-series-insights-gen2-environment"></a>Tutorial: Configuración de un entorno de Azure Time Series Insights Gen2

Este tutorial le guía en el proceso de creación de un entorno de *pago por uso* (PAYG) de Azure Time Series Insights Gen2.

En este tutorial, aprenderá a:

> [!div class="checklist"]
>
> * Crear un entorno de Azure Time Series Insights Gen2.
> * Conectar el entorno de Azure Time Series Insights Gen2 a una instancia de IoT Hub.
> * Ejecutar un ejemplo del acelerador de soluciones para transmitir datos al entorno de Azure Time Series Insights Gen2.
> * Realizar un análisis básico de los datos.
> * Definir el tipo y la jerarquía de un modelo de Time Series y asociarlos con sus instancias.

>[!TIP]
> Los [aceleradores de soluciones de IoT](https://www.azureiotsolutions.com/Accelerators) proporcionan soluciones preconfiguradas de nivel empresarial que se pueden usar puede usar para acelerar el desarrollo de soluciones de IoT personalizadas.

Regístrese para obtener [suscripción gratuita a Azure](https://azure.microsoft.com/free/), en caso de que no tenga ninguna.

## <a name="prerequisites"></a>Requisitos previos

* Como mínimo, debe tener el rol **Colaborador** para la suscripción de Azure. Para más información, consulte [Asignación de roles de Azure mediante Azure Portal](../role-based-access-control/role-assignments-portal.md).

* Cree un entorno mediante [Azure Portal](#create-an-azure-time-series-insights-gen2-environment) o la [CLI](how-to-create-environment-using-cli.md).

## <a name="create-a-device-simulation"></a>Creación de una simulación de dispositivo

En esta sección, creará tres dispositivos simulados que envían datos a una instancia de Azure IoT Hub.

1. Vaya a la [página de aceleradores de soluciones de Azure IoT](https://www.azureiotsolutions.com/Accelerators). Inicie sesión con la cuenta de Azure y, a continuación, seleccione **Simulación de dispositivo**.

   [![Página de aceleradores de soluciones de Azure IoT.](media/tutorial-set-up-environment/iot-solution-accelerators-landing-page.png)](media/tutorial-set-up-environment/iot-solution-accelerators-landing-page.png#lightbox)

1. Desplácese hacia abajo para leer las secciones de [Información general](https://github.com/Azure/azure-iot-pcs-device-simulation#overview) e [Introducción](https://github.com/Azure/azure-iot-pcs-device-simulation#getting-started).

1. Siga las [instrucciones de implementación](https://github.com/Azure/azure-iot-pcs-device-simulation/blob/master/deployment/README.md) de la sección Introducción.

   Este proceso puede tardar hasta 20 minutos en completarse.

1. Una vez finalizada la implementación, se le proporcionará la dirección URL de la simulación. Mantenga abierta esta página, ya que volverá a ella más adelante.

   >[!IMPORTANT]
   > No escriba todavía su acelerador de soluciones. Mantenga abierta esta página web, ya que volverá a ella más adelante.

   [![Aprovisionamiento de la solución de simulación de dispositivos completada](media/tutorial-set-up-environment/iot-solution-accelerator-ready.png)](media/tutorial-set-up-environment/iot-solution-accelerator-ready.png#lightbox)

1. Ahora, inspeccione los recursos recién creados en Azure Portal. En la página **Grupos de recursos**, se ve que se ha creado un nuevo grupo de recursos, para lo que se ha usado el `solutionName` proporcionado en el archivo de parámetros de la plantilla de ARM. Tome nota de los recursos que se crearon para la simulación de dispositivos.

   [![Recursos de la simulación de dispositivos.](media/tutorial-set-up-environment/device-sim-solution-resources.png)](media/tutorial-set-up-environment/device-sim-solution-resources.png#lightbox)

## <a name="create-an-azure-time-series-insights-gen2-environment"></a>Creación de un entorno de Azure Time Series Insights Gen2

En esta sección se describe cómo crear un entorno de Azure Time Series Insights Gen2 y conectarlo al centro de IoT que el Acelerador de soluciones de IOT mediante [Azure Portal](https://portal.azure.com/).

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con su cuenta de suscripción de Azure.
1. Seleccione **+ Crear un recurso** en la esquina superior izquierda.
1. Seleccione la categoría **Internet de las cosas** y, después, elija **Time Series Insights**.

   [![Selección del recurso de entorno de Time Series Insights.](media/tutorial-set-up-environment/create-new-environment.png)](media/tutorial-set-up-environment/create-new-environment.png#lightbox)

1. En el panel **Creación del entorno de Time Series Insights**, en la pestaña **Aspectos básicos**, establezca los parámetros siguientes:

    | Parámetro | Acción |
    | --- | ---|
    | **Nombre del entorno** | Especifique un nombre único para el entorno de Azure Time Series Insights Gen2. |
    | **Suscripción** | Especifique la suscripción donde desea crear el entorno de Azure Time Series Insights Gen2. Es un procedimiento recomendado usar la misma suscripción que el resto de los recursos de IoT creados por el simulador de dispositivos. |
    | **Grupos de recursos** | Seleccione un grupo de recursos existente o cree un nuevo grupo de recursos para el recurso de entorno de Azure Time Series Insights Gen2. Un grupo de recursos es un contenedor para recursos de Azure. Un procedimiento recomendado es usar el mismo grupo de recursos que el resto de los recursos de IoT creados por el simulador de dispositivos. |
    | **Ubicación** | Seleccione una región del centro de datos para su entorno de Azure Time Series Insights Gen2. Para evitar una latencia adicional, es mejor crear el entorno de Azure Time Series Insights Gen2 en la misma región que el centro de IoT creado por el simulador de dispositivo. |
    | **Nivel** |  Seleccione **Gen2(L1)** . Esta es la SKU del producto de Azure Time Series Insights Gen2. |
    | **Nombre de propiedad de Id. de serie temporal** | Escriba el nombre de una propiedad que contenga valores que identifiquen de forma exclusiva las instancias de la serie temporal. El valor que especifique en el cuadro **Nombre de propiedad** como Id. de serie temporal no se puede cambiar más tarde. En este tutorial, escriba **_iothub-connection-device-id_**. Para obtener más información sobre el id. de serie temporal, incluido el id. de serie temporal compuesto, consulte [Procedimientos recomendados al elegir un id. de serie temporal](./how-to-select-tsid.md). |
    | **Nombre de cuenta de almacenamiento** | Escriba un nombre único global para la nueva cuenta de almacenamiento.|
    | **Tipo de cuenta de almacenamiento** | Seleccione el tipo de almacenamiento para una nueva cuenta de almacenamiento. Se recomienda StorageV2.|
    | **Replicación de la cuenta de almacenamiento** | Seleccione el tipo de almacenamiento para una nueva cuenta de almacenamiento. En función de la selección de ubicación, puede elegir entre LRS, GRS y ZRS. Para este tutorial, puede seleccionar LRS.|
    | **Espacio de nombres jerárquico** |Esta opción es seleccionable, una vez que seleccione el tipo de almacenamiento para que sea StorageV2. De forma predeterminada, está deshabilitado. En este tutorial, puede dejarlo en su estado predeterminado *deshabilitado*.|
    |**Habilitar almacenamiento intermedio**|Seleccione **sí** para habilitar el almacenamiento intermedio. Esta configuración se puede deshabilitar y volver a habilitar una vez que también se haya creado el entorno. |
    |**Retención de datos (en días)**|Elija la opción predeterminada de 7 días. |

    [![Nueva configuración del entorno de Azure Time Series Insights.](media/tutorial-set-up-environment/environment-configuration.png)](media/tutorial-set-up-environment/environment-configuration.png#lightbox)
    [![Nueva configuración del entorno de Azure Time Series Insights (continuación).](media/tutorial-set-up-environment/environment-configuration2.png)](media/tutorial-set-up-environment/environment-configuration2.png#lightbox)

1. Seleccione **Siguiente: Origen de eventos**.

   [![Configuración del identificador de serie temporal para el entorno.](media/tutorial-set-up-environment/time-series-id-selection.png)](media/tutorial-set-up-environment/time-series-id-selection.png#lightbox)

1. En la pestaña **Origen del evento**, puede establecer los siguientes parámetros:

   | Parámetro | Acción |
   | --- | --- |
   | **Create an event source?** ¿Crear un origen del evento? | Seleccione **Sí**.|
   | **Tipo de origen** | Seleccione **IoT Hub**. |
   | **Nombre** | Escriba un valor único para el nombre del origen de eventos. |
   | **Selección de un centro** | Elija **Seleccionar existente**. |
   | **Suscripción** | Seleccione la suscripción que usó para el simulador de dispositivos. |
   | **Nombre de la instancia de IoT Hub** | Seleccione el nombre del centro de IoT que creó para el simulador de dispositivos. |
   | **Directiva de acceso de IoT Hub** | seleccione **iothubowner**. |
   | **Grupo de consumidores de IoT Hub** | Seleccione **Nuevo**, escriba un nombre único y, a continuación, seleccione **+ Agregar**. El grupo de consumidores debe ser un valor único en Azure Time Series Insights Gen2. |
   | **Opciones de inicio** | Seleccione **Beginning now** (Empezando ahora). |
   | **Propiedad de marca de tiempo** | Este campo se utiliza para identificar la propiedad de **Marca de tiempo** en los datos de telemetría entrantes. En este tutorial, deje este cuadro vacío. Este simulador usa la marca de tiempo entrante de IoT Hub, que Azure Time Series Insights Gen2 toma como valor predeterminado. |

   :::image type="content" source="media/tutorial-set-up-environment/configure-event-source.png" alt-text="Configuración del centro de IoT creado como origen de eventos" lightbox="media/tutorial-set-up-environment/configure-event-source.png":::

1. Seleccione **Revisar + crear**.

   :::image type="content" source="media/tutorial-set-up-environment/environment-confirmation.png" alt-text="Página Revisar y crear, con el botón Crear" lightbox="media/tutorial-set-up-environment/environment-confirmation.png":::

   Puede revisar el estado de la implementación:

   [![Notificación de que la implementación ha finalizado.](media/tutorial-set-up-environment/deployment-notification.png)](media/tutorial-set-up-environment/deployment-notification.png#lightbox)

1. Expanda los detalles de la implementación.

## <a name="stream-data"></a>Datos de streaming

Ahora que ha implementado el entorno de Azure Time Series Insights Gen2, comience a transmitir datos para su análisis.

1. Una vez completada la implementación del acelerador de soluciones, se le proporcionará una dirección URL.

1. Haga clic en la dirección URL para iniciar la simulación del dispositivo.

1. Seleccione **+ New simulation** (+ Nueva simulación).

    1. Después de que se cargue la página **Configuración de simulación**, escriba los parámetros necesarios.

        | Parámetro | Acción |
        | --- | --- |
        | **Nombre** | Escriba un nombre único para un simulador. |
        | **Descripción** | Escriba una definición. |
        | **Simulation duration** (Duración de la simulación) | Establézcala en **Run indefinitely** (Ejecutar indefinidamente). |
        | **Device model** (Modelo de dispositivo) | Haga clic en +**Add a device type** (Agregar un tipo de dispositivo). <br />**Name**: Escriba **Elevator**. <br />**Amount** (Cantidad): Escriba **3**. <br /> Deje los valores predeterminados restantes. |
        | **Target IoT Hub** (IoT Hub de destino) | Establézcalo en **Use pre-provisioned IoT Hub** (Usar IoT Hub aprovisionado previamente). |

        [![Configuración de los parámetros e inicio.](media/tutorial-set-up-environment/launch-solution-accelerator.png)](media/tutorial-set-up-environment/launch-solution-accelerator.png#lightbox)

    1. Seleccione **Start simulation** (Iniciar simulación). En el panel de simulación de dispositivos, se muestran los **Dispositivos activos** y el **Total de mensajes**.

        [![Panel de simulación de Azure IoT.](media/tutorial-set-up-environment/see-active-devices-and-messages.png)](media/tutorial-set-up-environment/see-active-devices-and-messages.png#lightbox)

## <a name="analyze-data"></a>Análisis de datos

En esta sección, se va realizar un análisis básico de los datos de la serie temporal mediante el [explorador de Azure Time Series Insights Gen2](./concepts-ux-panels.md).

1. Vaya al explorador de Azure Time Series Insights Gen2; para ello, seleccione la dirección URL de la página de recursos de [Azure Portal](https://portal.azure.com/).

    [![Dirección URL del explorador de Azure Time Series Insights Gen2](media/tutorial-set-up-environment/select-explorer-url.png)](media/tutorial-set-up-environment/select-explorer-url.png#lightbox).

1. En el explorador de Azure Time Series Insights Gen2, aparecerá una barra que abarca la parte superior de la pantalla. Este es el selector de disponibilidad. Asegúrese de tener al menos dos 2 m seleccionados y, si es necesario, expanda el período de tiempo; para ello, seleccione los controladores del selector y arrástrelos a la izquierda y a la derecha.

1. **Las instancias de la serie temporal** se muestran en el lado izquierdo.

    [![Lista de instancias no primarias.](media/tutorial-set-up-environment/explorer-unparented-instances.png)](media/tutorial-set-up-environment/explorer-unparented-instances.png#lightbox)

1. Seleccione la primera instancia de Time Series. A continuación, seleccione **Show temperature** (Mostrar temperatura).

    [![Instancia de Time Series seleccionada con el comando de menú para mostrar la temperatura media.](media/tutorial-set-up-environment/select-instance-and-temperature.png)](media/tutorial-set-up-environment/select-instance-and-temperature.png#lightbox)

    Aparece un gráfico con las series temporales. Cambie el **Intervalo** a **30s**.

1. Repita el paso anterior con las otras dos instancias de series temporales para ver las tres, como se muestra en este gráfico:

    [![Gráfico de todas las series temporales.](media/tutorial-set-up-environment/explorer-add-three-instances.png)](media/tutorial-set-up-environment/explorer-add-three-instances.png#lightbox)

1. Elija el selector de intervalo en la esquina superior derecha. Aquí puede seleccionar horas de inicio y finalización específicas hasta el milisegundo, o elegir entre opciones preconfiguradas, como **Last 30 minutes** (Últimos 30 minutos). También puede cambiar la zona horaria predeterminada.

    [![Establecimiento del intervalo de tiempo en los últimos 30 minutos.](media/tutorial-set-up-environment/explorer-thirty-minute-time-range.png)](media/tutorial-set-up-environment/explorer-thirty-minute-time-range.png#lightbox)

    Ahora se muestra el progreso del acelerador de soluciones durante los **últimos 30 minutos** en el explorador de Azure Time Series Insights Gen2.

## <a name="define-and-apply-a-model"></a>Definir y aplicar un modelo

En esta sección, aplicará un modelo para estructurar los datos. Para completar el modelo, definirá los tipos, las jerarquías y las instancias. Para más información sobre el modelado de datos, vea [Modelo de serie temporal](./concepts-model-overview.md).

1. En el explorador, seleccione la pestaña **Modelo**:

   [![Vea la pestaña Modelo en el explorador.](media/tutorial-set-up-environment/select-model-view.png)](media/tutorial-set-up-environment/select-model-view.png#lightbox)

   En la pestaña **Types** (Tipos), seleccione **+ Add** (+ Agregar).

1. Escriba los siguientes parámetros:

    | Parámetro | Acción |
    | --- | ---|
    | **Nombre** | Escriba **Elevator**. |
    | **Descripción** | Escriba **This is a type definition for Elevator** (Esta es una definición de tipo de Elevator). |

1. A continuación, seleccione la pestaña **Variables**.

    1. Seleccione **+ Agregar variable** y rellene los siguientes valores para la primera variable del tipo Elevator. Creará tres variables en total.

        | Parámetro | Acción |
        | --- | --- |
        | **Nombre** | Escriba **Avg Temperature** (Temperatura media). |
        | **Variante** | Seleccione **Numérico**. |
        | **Valor** | Seleccione un valor preestablecido: Seleccione **temperatura (doble)** . <br /> Nota: Es posible que el campo de **valor** tarde unos minutos en rellenarse automáticamente después de que Azure Time Series Insights Gen2 comience a recibir eventos.|
        | **Operación de agregación** | Expanda **Opciones avanzadas**. <br /> Seleccione **AVG** (MEDIA). |

    1. Seleccione **Aplicar**. Vuelva a seleccionar **+ Add Variable** (+ Agregar variable) y establezca los valores siguientes:

        | Parámetro | Acción |
        | --- | --- |
        | **Nombre** | Escriba **Avg Vibration**. |
        | **Variante** | Seleccione **Numérico**. |
        | **Valor** | Seleccione un valor preestablecido: Seleccione **vibration (Double)** (vibración [doble]). <br /> Nota: Es posible que el campo de **valor** tarde unos minutos en rellenarse automáticamente después de que Azure Time Series Insights Gen2 comience a recibir eventos.|
        | **Operación de agregación** | Expanda **Opciones avanzadas**. <br /> Seleccione **AVG** (MEDIA). |

    1. Seleccione **Aplicar**. Vuelva a seleccionar **+ Add Variable** (+Agregar variable) y establezca los valores siguientes para la tercera y última variable:

        | Parámetro | Acción |
        | --- | --- |
        | **Nombre** | Escriba **Floor**. |
        | **Variante** | Seleccione **Categórico**. |
        | **Valor** | Seleccione un valor preestablecido: Seleccione **Floor (Double)** (Planta [doble]). <br /> Nota: Es posible que el campo de **valor** tarde unos minutos en rellenarse automáticamente después de que Azure Time Series Insights Gen2 comience a recibir eventos.|
        | **Categorías** | <span style="text-decoration: underline">Etiqueta </span>  - <span style="text-decoration: underline">Valores</span> <br /> Inferior: 1,2,3,4 <br /> Central: 5,6,7,8,9 <br /> Superior: 10,11,12,13,14,15 |
        | **Categoría predeterminada** | Escriba **Desconocido**. |

        [![Adición de variables de tipo.](media/tutorial-set-up-environment/add-type-variables.png)](media/tutorial-set-up-environment/add-type-variables.png#lightbox)

    1. Seleccione **Aplicar**.
    1. Seleccione **Guardar**. Se crean y se muestran tres variables.

        [![Después de agregar el tipo, puede revisarlo en la vista Modelo.](media/tutorial-set-up-environment/add-type-and-view.png)](media/tutorial-set-up-environment/add-type-and-view.png#lightbox)

1. Seleccione la pestaña **Jerarquías**. Seguidamente, seleccione **+ Agregar**.

   1. En el panel **Edit Hierarchy** (Editar jerarquía), establezca los parámetros siguientes:

        | Parámetro | Acción |
        | --- | ---|
        | **Nombre** | Especifique el valor de **Location Hierarchy** (Jerarquía de ubicación). |
        |**Niveles**| Escriba **Country** como nombre del primer nivel.<br />Seleccione **+ Agregar nivel**.<br />Escriba **City** para el segundo nivel y, después, seleccione **+ Agregar nivel**.<br />Escriba **Building** como nombre del tercer y último nivel. |

   1. Seleccione **Guardar**.

        [![Visualización de la nueva jerarquía en la vista Model (Modelo).](media/tutorial-set-up-environment/add-hierarchy-and-view.png)](media/tutorial-set-up-environment/add-hierarchy-and-view.png#lightbox)

1. Vaya a **Instancias**.

    1. En **Acciones** en el extremo derecho, seleccione el icono de lápiz para editar la primera instancia con los valores siguientes:

        | Parámetro | Acción |
        | --- | --- |
        | **Tipo** | Seleccione **Elevator**. |
        | **Nombre** | Escriba **Elevator 1**.|
        | **Descripción** | Escriba **Instance for Elevator 1**. |

    1. Vaya a **Instance Fields** (Campos de instancia) y escriba los siguientes valores:

        | Parámetro | Acción |
        | --- | --- |
        | **Jerarquías** | Seleccione **Location Hierarchy** (Jerarquía de ubicación). |
        | **País** | Escriba **USA**. |
        | **Ciudad** | Escriba **Seattle**. |
        | **Edificio** | Escriba **Space Needle**. |

    1. Seleccione **Guardar**.

1. Repita el paso anterior con las otras dos instancias y use los siguientes valores:

    **Para Elevator 2:**

    | Parámetro | Acción |
    | --- | --- |
    | **Tipo** | Seleccione **Elevator**. |
    | **Nombre** | Escriba **Elevator 2**.|
    | **Descripción** | Escriba **Instance for Elevator 2**. |
    | **Jerarquías** | Seleccione **Location Hierarchy** (Jerarquía de ubicación). |
    | **País** | Escriba **USA**. |
    | **Ciudad** | Escriba **Seattle**. |
    | **Edificio** | Escriba **Pacific Science Center**. |

    **Para Elevator 3:**

    | Parámetro | Acción |
    | --- | --- |
    | **Tipo** | Seleccione **Elevator**. |
    | **Nombre** | Escriba **Elevator 3**.|
    | **Descripción** | Escriba **Instance for Elevator 3**. |
    | **Jerarquías** | Seleccione **Location Hierarchy** (Jerarquía de ubicación). |
    | **País** | Escriba **USA**. |
    | **Ciudad** | Escriba **New York**. |
    | **Edificio** | Escriba **Empire State Building**. |

    [![Visualización de las instancias actualizadas.](media/tutorial-set-up-environment/iot-solution-accelerator-instances.png)](media/tutorial-set-up-environment/iot-solution-accelerator-instances.png#lightbox)

1. Vuelva a la pestaña **Analizar** para ver el panel de gráficos. En **Location Hierarchy** (Jerarquía de ubicación), expanda todos los niveles de jerarquía para mostrar las instancias de la serie temporal:

    [![Visualización de todas las jerarquías en la vista de gráfico.](media/tutorial-set-up-environment/iot-solution-accelerator-view-hierarchies.png)](media/tutorial-set-up-environment/iot-solution-accelerator-view-hierarchies.png#lightbox)

1. En **Pacific Science Center**, seleccione la instancia de Time Series **Elevator 2** y, a continuación, seleccione **Show Average Temperature** (Mostrar temperatura media).

1. En la misma instancia, **Elevator 2**, seleccione **Show Floor** (Mostrar planta).

    Con la variable de categoría, puede determinar cuánto tiempo empleó el ascensor en el planta superior, inferior y central.

    [![Visualización de Elevator 2 con la jerarquía y los datos.](media/tutorial-set-up-environment/iot-solution-accelerator-elevator-two.png)](media/tutorial-set-up-environment/iot-solution-accelerator-elevator-two.png#lightbox)

## <a name="clean-up-resources"></a>Limpieza de recursos

Ahora que ha completado el tutorial, limpie los recursos que ha creado:

1. En el menú de la izquierda de [Azure Portal](https://portal.azure.com), seleccione **Todos los recursos** y busque el grupo de recursos de Azure Time Series Insights Gen2.
1. Seleccione **Eliminar** para eliminar todo el grupo de recursos (y todos los recursos que contiene), o quite los recursos de uno en uno.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido a:

* Crear y usar un acelerador de simulación de dispositivos.
* Crear un entorno PAYG de Azure Time Series Insights Gen2.
* Conectar el entorno de Azure Time Series Insights Gen2 a una instancia de IoT Hub.
* Ejecutar un ejemplo del acelerador de soluciones para transmitir datos al entorno de Azure Time Series Insights Gen2.
* Realizar un análisis básico de los datos.
* Definir el tipo y la jerarquía de un modelo de Time Series y asociarlos con sus instancias.

Ahora que sabe cómo crear su propio entorno de Azure Time Series Insights Gen2, obtenga más información sobre los conceptos clave de Azure Time Series Insights Gen2.

Lea acerca de la ingesta de datos en Azure Time Series Insights Gen2:

> [!div class="nextstepaction"]
> [Introducción a la ingesta de datos en Azure Time Series Insights Gen2](./concepts-ingestion-overview.md)

Lea acerca del almacenamiento en Azure Time Series Insights Gen2:

> [!div class="nextstepaction"]
> [Almacenamiento de datos en Azure Time Series Insights Gen2](./concepts-storage.md)

Obtenga más información sobre los modelos de serie temporal:

> [!div class="nextstepaction"]
> [Modelo de datos en Azure Time Series Insights Gen2](./concepts-model-overview.md)

Más información sobre cómo conectar un entorno a Power BI:

> [!div class="nextstepaction"]
> [Visualización de datos de Azure Time Series Insights Gen2 en Power BI](./how-to-connect-power-bi.md)
