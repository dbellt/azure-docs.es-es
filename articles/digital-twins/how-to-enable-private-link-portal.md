---
title: Habilitación del acceso privado con Private Link (versión preliminar) mediante Azure Portal
titleSuffix: Azure Digital Twins
description: Vea cómo habilitar el acceso privado de las soluciones de Azure Digital Twins con Private Link mediante Azure Portal.
author: baanders
ms.author: baanders
ms.date: 1/25/2021
ms.topic: how-to
ms.service: digital-twins
ms.openlocfilehash: ff1589eb496cbc4482aaf93dfd4f0faeeda5e9cd
ms.sourcegitcommit: a5dd9799fa93c175b4644c9fe1509e9f97506cc6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108208876"
---
# <a name="enable-private-access-with-private-link-preview-azure-portal"></a>Habilitación del acceso privado con Private Link (versión preliminar): Azure portal

[!INCLUDE [digital-twins-private-link-selector.md](../../includes/digital-twins-private-link-selector.md)]

En este artículo se describen las distintas formas de [habilitar Private Link con un punto de conexión privado para una instancia de Azure Digital Twins](concepts-security.md#private-network-access-with-azure-private-link-preview) (actualmente en versión preliminar). La configuración de un punto de conexión privado para la instancia de Azure Digital Twins le permite protegerla y eliminar la exposición pública, además de evitar la filtración de datos desde la instancia de [Azure Virtual Network (VNet)](../virtual-network/virtual-networks-overview.md).

En este artículo se describe el proceso que utiliza [Azure Portal](https://portal.azure.com).

Estos son los pasos que se describen en este artículo: 
1. Activación de Private Link y configuración de un punto de conexión privado para una instancia de Azure Digital Twins.
1. Deshabilitación o habilitación de marcas de acceso a la red pública para restringir el acceso de la API únicamente a las conexiones de Private Link.

## <a name="prerequisites"></a>Requisitos previos

Para poder configurar un punto de conexión privado, necesitará una [red virtual de Azure (VNet)](../virtual-network/virtual-networks-overview.md)  en la que se pueda implementar el punto de conexión. Si todavía no tiene una red virtual, puede seguir uno de los [inicios rápidos](../virtual-network/quick-create-portal.md) de Azure Virtual Network para configurarla.

## <a name="add-a-private-endpoint-for-an-azure-digital-twins-instance"></a>Adición de un punto de conexión privado para una instancia de Azure Digital Twins 

Al usar [Azure Portal](https://portal.azure.com) puede activar Private Link con un punto de conexión privado para una instancia de Azure Digital Twins como parte de la configuración inicial de la instancia, o bien habilitarlo más adelante en una instancia que ya exista. 

Cualquiera de estos métodos de creación proporcionará las mismas opciones de configuración y el mismo resultado final para la instancia. En esta sección se describe cómo realizar ambos.

>[!TIP]
> También puede configurar un punto de conexión de Private Link mediante el servicio Private Link, en lugar de hacerlo con la instancia de Azure Digital Twins. Esto también proporciona las mismas opciones de configuración y el mismo resultado final.
>
> Para obtener más información sobre la configuración de recursos de Private Link, vea la documentación de Azure Private Link para [Azure Portal](../private-link/create-private-endpoint-portal.md), la [CLI de Azure](../private-link/create-private-endpoint-cli.md), [ARM](../private-link/create-private-endpoint-template.md) o [PowerShell](../private-link/create-private-endpoint-powershell.md).

### <a name="add-a-private-endpoint-during-instance-creation"></a>Adición de un punto de conexión privado durante la creación de la instancia

En esta sección, habilitará Private Link con un punto de conexión privado en una instancia de Azure Digital Twins que se está creando actualmente. Esta sección se centra en el paso de red del proceso de creación; para obtener un tutorial completo sobre la creación de una instancia de Azure Digital Twins, vea [Procedimiento: Configuración de una instancia y autenticación](how-to-set-up-instance-portal.md).

Las opciones de Private Link se encuentran en la pestaña **Redes** de configuración de la instancia.

En esta pestaña, puede habilitar puntos de conexión privados si selecciona la opción **Punto de conexión privado** para el **Método de conectividad**.

Esto agregará una sección llamada **Puntos de conexión privados** donde puede configurar los detalles del punto de conexión privado. Seleccione el botón **+ Agregar** para continuar.

:::image type="content" source="media/how-to-enable-private-link/create-instance-networking-1.png" alt-text="Captura de pantalla de Azure Portal en la que se muestra la pestaña Redes del cuadro de diálogo Crear recurso de Azure Digital Twins. Hay un resaltado alrededor del nombre de la pestaña, la opción Punto de conexión privado para Método de conectividad y el botón + Agregar para crear una conexión de punto de conexión privado." lightbox="media/how-to-enable-private-link/create-instance-networking-1.png":::

Se abrirá una página para especificar los detalles de un nuevo punto de conexión privado.

:::image type="content" source="media/how-to-enable-private-link/create-private-endpoint-full.png" alt-text="Captura de pantalla de Azure Portal en la que se muestra la página Crear punto de conexión privado. Contiene los campos que se describen a continuación.":::

1. Rellene las selecciones para la **Suscripción** y el **Grupo de recursos**. Establezca la **Ubicación** en la misma de la red virtual que va a usar. Elija un **Nombre** para el punto de conexión y, para **Subrecursos de destino**, seleccione *API*.

1. A continuación, seleccione la **Red virtual** y la **Subred** que quiera usar para implementar el punto de conexión.

1. Por último, seleccione si quiere **Integrar con la zona DNS privada**. Puede usar el valor predeterminado de **Sí**, o bien, para obtener ayuda con esta opción, puede seguir el vínculo del portal para [obtener más información sobre la integración de DNS privado](../private-link/private-endpoint-overview.md#dns-configuration).

Después de rellenar las opciones de configuración, presione **Aceptar** para finalizar.

Volverá a la pestaña **Redes** de la configuración de la instancia de Azure Digital Twins, donde el nuevo punto de conexión debe estar visible en **Conexiones de punto de conexión privado.

:::image type="content" source="media/how-to-enable-private-link/create-instance-networking-2.png" alt-text="Captura de pantalla de Azure Portal en la que se muestra la pestaña Redes del cuadro de diálogo Crear recurso de Azure Digital Twins. Hay un resaltado alrededor de la nueva conexión de punto de conexión privado y los botones de navegación (Revisar y crear, Anterior, Siguiente: Avanzado)." lightbox="media/how-to-enable-private-link/create-instance-networking-2.png":::

Después, puede usar los botones de navegación inferiores para continuar con el resto de la configuración de la instancia.

### <a name="add-a-private-endpoint-to-an-existing-instance"></a>Adición de un punto de conexión privado a una instancia existente

En esta sección, habilitará Private Link con un punto de conexión privado en una instancia de Azure Digital Twins que ya existe.

1. En primer lugar, navegue a [Azure Portal](https://portal.azure.com) en un explorador. Abra la instancia de Azure Digital Twins; para ello, busque su nombre en la barra de búsqueda del portal.

1. En el menú de la izquierda, seleccione **Redes (versión preliminar)** .

1. Cambie a la pestaña **Conexiones de punto de conexión privado**.

1. Seleccione **+ Punto de conexión privado** para abrir la configuración **Crear un punto de conexión privado**.

    :::image type="content" source="media/how-to-enable-private-link/add-endpoint-digital-twins.png" alt-text="Captura de pantalla de Azure Portal en la que se muestra la pestaña Redes (versión preliminar) de una instancia de Azure Digital Twins. La pestaña Conexiones de punto de conexión privado está resaltada y el botón + Punto de conexión privado también." lightbox="media/how-to-enable-private-link/add-endpoint-digital-twins.png":::

1. En la pestaña  **Aspectos básicos** , escriba o seleccione la **Suscripción** y el **Grupo de recursos** del proyecto, y un **Nombre** y una **Región** para el punto de conexión. La región debe ser la misma que la de la red virtual que usa.

    :::image type="content" source="media/how-to-enable-private-link/create-private-endpoint-1.png" alt-text="Captura de pantalla de Azure Portal en la que se muestra la pestaña (Básico) del cuadro de diálogo Crear punto de conexión privado. Contiene los campos descritos antes.":::

    Cuando haya terminado, seleccione el botón **Siguiente: Recurso >** para ir a la pestaña siguiente.

1. En la pestaña **Recurso**, escriba o seleccione esta información: 
    * **Método de conexión**: seleccione **Conectarse a un recurso de Azure en mi directorio** para buscar la instancia de Azure Digital Twins.
    * **Suscripción**: escriba la suscripción.
    * **Tipo de recurso**: seleccione **Microsoft.DigitalTwins/digitalTwinsInstances**.
    * **Recursos**: seleccione el nombre de la instancia de Azure Digital Twins.
    * **Recurso secundario de destino**: Seleccione **API**.

    :::image type="content" source="media/how-to-enable-private-link/create-private-endpoint-2.png" alt-text="Captura de pantalla de Azure Portal en la que se muestra la segunda pestaña (Recurso) del cuadro de diálogo Crear punto de conexión privado. Contiene los campos descritos antes.":::

    Cuando haya terminado, seleccione el botón **Siguiente: Configuración >** para ir a la pestaña siguiente.    

1. En la pestaña  **Configuración**, escriba o seleccione esta información:
    * **Red virtual**: Seleccione la red virtual.
    * **Subred**: elija una subred de la red virtual.
    * **Integración con una zona DNS privada**: seleccione si quiere **Integrar con la zona DNS privada**. Puede usar el valor predeterminado de **Sí**, o bien, para obtener ayuda con esta opción, puede seguir el vínculo del portal para [obtener más información sobre la integración de DNS privado](../private-link/private-endpoint-overview.md#dns-configuration).
    Si selecciona **Sí**, puede dejar la información de configuración predeterminada.

    :::image type="content" source="media/how-to-enable-private-link/create-private-endpoint-3.png" alt-text="Captura de pantalla de Azure Portal en la que se muestra la tercera pestaña (Configuración) del cuadro de diálogo Crear punto de conexión privado. Contiene los campos descritos antes.":::

    Cuando haya terminado, puede seleccionar el botón **Revisar y crear** para finalizar la instalación. 

1. En la pestaña **Revisar y crear**, revise las selecciones y seleccione el botón  **Crear**. 

Cuando finalice la implementación del punto de conexión, debe aparecer en las conexiones de punto de conexión privado de la instancia de Azure Digital Twins.

>[!TIP]
> El punto de conexión también se puede ver desde Private Link Center en Azure Portal.

## <a name="disable--enable-public-network-access-flags"></a>Habilitación o deshabilitación de las marcas de acceso a la red pública

Puede configurar la instancia de Azure Digital Twins para denegar todas las conexiones públicas y permitir solo las conexiones a través de puntos de conexión privados a fin de mejorar la seguridad de la red. Esta acción se realiza con una **marca de acceso a la red pública**. 

Esta directiva permite restringir el acceso de la API únicamente a las conexiones de Private Link. Cuando la marca de acceso a la red pública está establecida en *deshabilitada*, todas las llamadas API REST al plano de datos de la instancia de Azure Digital Twins desde la nube pública devolverán `403, Unauthorized`. Como alternativa, cuando la directiva se establece en *deshabilitada* y se realiza una solicitud a través de un punto de conexión privado, la llamada API se realizará correctamente.

En este artículo se indica cómo actualizar el valor de la marca de red mediante [Azure Portal](https://portal.azure.com). Para obtener instrucciones sobre cómo hacerlo con la CLI de Azure o la herramienta de comandos ARMClient, consulte la [versión para la CLI](how-to-enable-private-link-cli.md) de este artículo.

### <a name="use-the-azure-portal"></a>Uso de Azure Portal

Para deshabilitar o habilitar el acceso a la red pública en [Azure Portal](https://portal.azure.com), abra el portal y vaya a la instancia de Azure Digital Twins.

1. En el menú de la izquierda, seleccione **Redes (versión preliminar)** .

1. En la pestaña **Acceso público**, establezca **Permitir el acceso de red público**, en **Deshabilitado** o **Todas las redes**.

    :::row:::
        :::column:::
            :::image type="content" source="media/how-to-enable-private-link/network-flag-portal.png" alt-text="Captura de pantalla de Azure Portal en la que se muestra la pestaña Redes (versión preliminar) de una instancia de Azure Digital Twins. La pestaña Acceso público está resaltada y también la opción para permitir el acceso a la red pública." lightbox="media/how-to-enable-private-link/network-flag-portal.png":::
        :::column-end:::
        :::column:::
        :::column-end:::
    :::row-end:::

    Seleccione **Guardar**.

## <a name="next-steps"></a>Pasos siguientes

Obtenga más información sobre Private Link para Azure: 
* [¿Qué es el servicio Azure Private Link?](../private-link/private-link-service-overview.md)