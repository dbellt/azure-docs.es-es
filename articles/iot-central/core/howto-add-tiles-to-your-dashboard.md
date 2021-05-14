---
title: Configuración del panel de Azure IoT Central | Microsoft Docs
description: Como creador, aprenda a configurar el panel de la aplicación predeterminado de Azure IoT Central con iconos.
author: philmea
ms.author: philmea
ms.date: 12/19/2020
ms.topic: how-to
ms.service: iot-central
ms.openlocfilehash: 8a8ba765a966409c06dbba636932f7777624f6d4
ms.sourcegitcommit: 2aeb2c41fd22a02552ff871479124b567fa4463c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107864260"
---
# <a name="configure-the-application-dashboard"></a>Configuración del panel de la aplicación

El **panel** es la primera página que ve cuando se conecta a una aplicación de IoT Central. Si crea la aplicación a partir de una de las [plantillas de aplicación](./concepts-app-templates.md) específicas del sector, tiene un panel predefinido como punto de partida. Si crea la aplicación a partir de una [plantilla de aplicación](./concepts-app-templates.md) personalizada, en el panel se muestran algunas sugerencias para empezar.

> [!TIP]
> Los usuarios pueden [crear varios paneles](howto-create-personal-dashboards.md) además del panel de aplicación predeterminado. Estos paneles pueden ser personales solo para el usuario o se pueden compartir entre todos los usuarios de la aplicación.  

## <a name="add-tiles"></a>Agregar iconos

En la captura de pantalla siguiente se muestra el panel en una aplicación creada a partir de la plantilla **Aplicación personalizada**. Para personalizar el panel actual, seleccione **Editar**; para agregar un panel personal o compartido personalizado, seleccione **Nuevo**:

:::image type="content" source="media/howto-add-tiles-to-your-dashboard/dashboard-sample-contoso.png" alt-text="Panel para aplicaciones basadas en la plantilla Aplicación personalizada":::

Después de seleccionar **Editar** o **Nuevo**, el panel se encuentra en modo de *edición*. Puede usar las herramientas del panel **Editar panel** para agregar, personalizar y quitar iconos del panel. Por ejemplo, para agregar un icono **Telemetría** a fin de mostrar la temperatura actual detectada por uno o más dispositivos:

1. Seleccione un **Grupo de dispositivos** y, a continuación, elija los dispositivos del menú desplegable **Dispositivos** que se deben mostrar en el icono. Ahora verá la telemetría, las propiedades y los comandos disponibles de los dispositivos.

1. Si es necesario, use el menú desplegable para seleccionar el valor de telemetría que se va a mostrar en el icono. Puede agregar más elementos al icono. Para ello, seleccione **+ Telemetría**, **+ Propiedad** o **+ Propiedad de la nube**.

:::image type="content" source="media/howto-add-tiles-to-your-dashboard/device-details.png" alt-text="Adición de un icono de telemetría de temperatura al panel":::

Cuando haya seleccionado todos los valores que se van a mostrar en el icono, haga clic en **Agregar icono.** Ahora el icono se muestra en el panel, donde puede cambiar la visualización y el tamaño, y configurarlo.

Cuando haya terminado de agregar y personalizar iconos en el panel, seleccione **Guardar** para guardar los cambios en el panel, con lo que saldrá del modo de edición.

## <a name="customize-tiles"></a>Personalización de iconos

Para editar un icono, debe estar en el modo de edición.  Las opciones de personalización disponibles dependen del [tipo de icono](#tile-types):

* El icono de regla de un icono permite cambiar la visualización. Entre las visualizaciones se incluyen gráficos de líneas, gráficos de barras, gráficos circulares, los últimos valores conocidos, indicadores clave de rendimiento (o KPI), mapas térmicos y mapas.

* El icono cuadrado permite cambiar el tamaño del icono.

* El icono de engranaje permite configurar la visualización. Por ejemplo, para una visualización de gráfico de líneas puede elegir mostrar la leyenda y los ejes, y el intervalo de tiempo que se va a trazar.


## <a name="tile-types"></a>Tipos de iconos

En la tabla siguiente se describen los distintos tipos de icono que puede agregar a un panel:

| Icono             | Descripción |
| ---------------- | ----------- |
| Markdown         | Los iconos de Markdown son iconos interactivos con un encabezado y texto de descripción a los que se les aplica formato mediante Markdown. La dirección URL puede ser un vínculo relativo a otra página de la aplicación, o bien un vínculo absoluto a un sitio externo.|
| Imagen            | Los iconos de imagen muestran una imagen personalizada y se puede hacer clic en ellos. La dirección URL puede ser un vínculo relativo a otra página de la aplicación, o bien un vínculo absoluto a un sitio externo.|
| Etiqueta            | Los iconos de etiqueta muestran texto personalizado en un panel. Puede elegir el tamaño del texto. Use un icono de etiqueta para agregar información relevante al panel como, por ejemplo, descripciones, detalles de contacto o ayuda.|
| Count            | Los iconos de recuento muestran el número de dispositivos de un grupo de dispositivos.|
| Map              | Los iconos de mapa muestran la ubicación de uno o varios dispositivos en un mapa. También puede mostrar hasta cien puntos del historial de ubicación de un dispositivo. Por ejemplo, puede mostrar una ruta muestreada de la ubicación del dispositivo durante la última semana.|
| KPI              |  Los mosaicos de KPI muestran valores de telemetría agregados para uno o más dispositivos durante un período de tiempo. Por ejemplo, se pueden usar para mostrar la temperatura máxima y la presión alcanzadas para uno o varios dispositivos durante la última hora.|
| Gráfico de líneas       | Los iconos de gráfico de líneas representan uno o varios valores de telemetría agregados para uno o más dispositivos durante un período de tiempo. Por ejemplo, puede mostrar un gráfico de líneas que trace la temperatura y presión medias de uno o varios dispositivos durante la última hora.|
| Gráfico de barras        | Los iconos de gráfico de barras representan uno o varios valores de telemetría agregados para uno o más dispositivos durante un período de tiempo. Por ejemplo, puede mostrar un gráfico de barras que muestre la temperatura y presión medias de uno o varios dispositivos durante la última hora.|
| Gráfico circular        | Los iconos de gráfico circular muestran uno o varios valores de telemetría agregados para uno o más dispositivos durante un período de tiempo.|
| Mapa térmico         | Los iconos de mapa térmico muestran información sobre uno o varios dispositivos, representada en forma de colores.|
| Último valor conocido | Los iconos de último valor conocido muestran los valores de telemetría más recientes de uno o más dispositivos. Por ejemplo, puede usar este icono para mostrar los valores más recientes de temperatura, presión y humedad de uno o varios dispositivos. |
| Historial de eventos    | Los iconos del Historial de eventos muestran los eventos ocurridos en un dispositivo durante un período de tiempo. Por ejemplo, los puede usar para mostrar todos los eventos de apertura y cierre de válvulas de uno o más dispositivos durante la última hora.|
| Propiedad         |  Los iconos de propiedades muestran el valor actual de las propiedades y las propiedades de la nube de uno o varios dispositivos. Por ejemplo, puede usar este icono para mostrar las propiedades del dispositivo, como el fabricante o la versión de firmware. |

Actualmente, puede agregar hasta 10 dispositivos a los mosaicos que admiten varios dispositivos.

### <a name="customizing-visualizations"></a>Personalización de visualizaciones

De forma predeterminada, los gráficos de líneas muestran datos de un intervalo de tiempo. El intervalo de tiempo seleccionado se divide en 50 cubos del mismo tamaño y, a continuación, los datos del dispositivo se agregan por cubo para proporcionar 50 puntos de datos durante el intervalo de tiempo seleccionado. Si desea ver los datos sin procesar, puede cambiar la selección para ver los últimos 100 valores. Para cambiar el intervalo de tiempo o para seleccionar la visualización de datos sin procesar, use le menú desplegable Mostrar intervalo en el panel **Configurar gráfico**.

:::image type="content" source="media/howto-add-tiles-to-your-dashboard/display-range.png" alt-text="Cambiar el intervalo de visualización de un gráfico de líneas":::

En el caso de los iconos que muestran valores agregados, seleccione el icono de engranaje situado junto al tipo de telemetría en el panel **Configurar gráfico** para elegir la agregación. Puede elegir entre el promedio, la suma, el valor máximo, el valor mínimo y el recuento.

En el caso de los gráficos de líneas, de barras y circulares, puede personalizar el color de los distintos valores de telemetría. Seleccione el icono de paleta situado junto a la telemetría que quiere personalizar:

:::image type="content" source="media/howto-add-tiles-to-your-dashboard/color-customization.png" alt-text="Cambio del color de un valor de telemetría":::

En el caso de los iconos que muestran propiedades de cadena o valores de telemetría, puede elegir cómo mostrar el texto. Por ejemplo, si el dispositivo almacena una dirección URL en una propiedad de cadena, puede mostrarla como un vínculo interactivo. Si la dirección URL hace referencia a una imagen, se puede representar la imagen en un último valor conocido o en el icono de propiedad. Para cambiar cómo se muestra una cadena, en la configuración del icono, seleccione el icono de engranaje situado junto a la propiedad o el tipo de telemetría:

:::image type="content" source="media/howto-add-tiles-to-your-dashboard/string-customization.png" alt-text="Cambio de la representación de una cadena en un icono":::

Para los iconos de **KPI**, **Último valor conocido** y **Propiedad**, puede usar el formato condicional para personalizar el color del icono en función de su valor actual. Para agregar formato condicional, seleccione **Configurar** en el icono y, a continuación, seleccione el icono de **Formato condicional** situado junto al valor que se va a personalizar:

:::image type="content" source="media/howto-add-tiles-to-your-dashboard/conditional-formatting-1.png" alt-text="Captura de pantalla que muestra cómo buscar la opción de configuración de un icono y, después, el icono de formato condicional":::

Agregue las reglas de formato condicional:

:::image type="content" source="media/howto-add-tiles-to-your-dashboard/conditional-formatting-2.png" alt-text="Captura de pantalla que muestra las reglas de formato condicional para un flujo medio. Hay tres reglas: menos de 20 es verde, menos de 50 es amarillo y todo lo que está por encima de 50 es rojo":::
   
En la captura de pantalla siguiente se muestra el efecto de la regla de formato condicional:

:::image type="content" source="media/howto-add-tiles-to-your-dashboard/conditional-formatting-3.png" alt-text="Captura de pantalla que muestra el fondo de color rojo en el icono de caudal de agua promedio. El número del icono es 50.54":::

### <a name="tile-formatting"></a>Formato del "icono"
Esta característica, disponible en los iconos de KPI, LKV y Propiedad, permite a los usuarios ajustar el tamaño de fuente, elegir la precisión decimal, abreviar valores numéricos o ajustar valores de encapsulado en sus iconos.

:::image type="content" source="media/howto-add-tiles-to-your-dashboard/tile-format.png" alt-text="Formato de icono":::

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha aprendido a configurar el panel de aplicaciones predeterminado de Azure IoT Central, puede [aprender a crear un panel personal](howto-create-personal-dashboards.md).
