---
title: Azure Digital Twins Explorer
titleSuffix: Azure Digital Twins
description: Descripción de las funcionalidades y la finalidad de Azure Digital Twins Explorer
author: baanders
ms.author: baanders
ms.date: 4/28/2021
ms.topic: conceptual
ms.service: digital-twins
ms.openlocfilehash: a0ecf0200b56b602468f1ca6c3dbfe19a5860c60
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111966375"
---
# <a name="azure-digital-twins-explorer-preview"></a>Azure Digital Twins Explorer (versión preliminar)

**Azure Digital Twins Explorer** es una herramienta para desarrolladores que se emplea para visualizar e interactuar con los datos de la instancia de Azure Digital Twins, incluidos los [modelos](concepts-models.md) y el [grafo del gemelo](concepts-twins-graph.md). 

>[!NOTE]
>Esta herramienta se encuentra actualmente en **versión preliminar pública**.

Esta es una vista de la ventana del explorador, en la que se muestran los modelos y gemelos que se han rellenado para un grafo de ejemplo:

:::image type="content" source="media/concepts-azure-digital-twins-explorer/azure-digital-twins-explorer-demo.png" alt-text="Captura de pantalla de Azure Digital Twins Explorer que muestra modelos y gemelos de ejemplo." lightbox="media/concepts-azure-digital-twins-explorer/azure-digital-twins-explorer-demo.png":::

La interfaz visual es una excelente herramienta para explorar y comprender la forma del grafo y el conjunto de modelos, así como para realizar cambios ad hoc específicos en gemelos y relaciones individuales.

Este artículo contiene más información sobre Azure Digital Twins Explorer como, por ejemplo, sus casos de uso y un resumen general de sus características. Para obtener pasos detallados sobre el uso de cada característica, consulte [Procedimiento: Uso de Azure Digital Twins Explorer](how-to-use-azure-digital-twins-explorer.md).

## <a name="when-to-use"></a>Cuándo se usa

Azure Digital Twins Explorer es una herramienta visual diseñada para los usuarios que desean explorar su grafo del gemelo y modificar gemelos y relaciones en el contexto de este grafo.

Los desarrolladores pueden encontrar esta herramienta especialmente útil en los escenarios siguientes:
* **Exploración**: use el explorador para obtener información sobre Azure Digital Twins y la forma en que representa su entorno real. Importe modelos y grafos de ejemplo que pueda ver y editar para familiarizarse con el servicio. Para ver los pasos guiados para empezar a usar Azure Digital Twins Explorer, consulte [Inicio rápido: Introducción a Azure Digital Twins Explorer](quickstart-azure-digital-twins-explorer.md).
* **Desarrollo**: use el explorador para ver y validar el grafo del gemelo, así como para investigar propiedades específicas de modelos, gemelos y relaciones. Realice modificaciones ad hoc en el grafo y sus datos. Para obtener instrucciones detalladas sobre cómo usar cada característica, consulte [Procedimiento: Uso de Azure Digital Twins Explorer](how-to-use-azure-digital-twins-explorer.md). 

El propósito principal del explorador es ayudarle a visualizar y comprender el grafo, y actualizarlo según sea necesario. Para soluciones a gran escala y para trabajos que se deben repetir o automatizar, considere la posibilidad de usar [API y SDK](./concepts-apis-sdks.md) para interactuar con la instancia mediante código.

[!INCLUDE [digital-twins-access-explorer.md](../../includes/digital-twins-access-explorer.md)]

## <a name="features-and-capabilities"></a>Características y funcionalidades

Azure Digital Twins Explorer se organiza en paneles, cada uno con un conjunto diferente de funcionalidades para explorar y administrar los modelos, gemelos y relaciones.

Las secciones del explorador son las siguientes:
* **Modelos:** agregue, quite y vea los detalles de los modelos desde una vista de lista.
* **Grafo de modelos**: visualice los modelos y las formas en que están interconectados en forma de un grafo de modelos personalizable.
* **Grafo de gemelos**: visualice los gemelos y las relaciones como un grafo de gemelos personalizable. Cree y elimine gemelos y relaciones, y vea o edite sus propiedades.
* **Explorador de consultas**: ejecute consultas en el grafo de gemelos y vea los resultados visuales en el panel **Grafo de gemelos**.

:::image type="content" source="media/concepts-azure-digital-twins-explorer/azure-digital-twins-explorer-panels.png" alt-text="Captura de pantalla de Azure Digital Twins Explorer, con un resaltado alrededor de cada uno de los paneles descritos anteriormente." lightbox="media/concepts-azure-digital-twins-explorer/azure-digital-twins-explorer-panels.png":::

Para obtener instrucciones detalladas sobre cómo usar cada característica, consulte [Procedimiento: Uso de Azure Digital Twins Explorer](how-to-use-azure-digital-twins-explorer.md). 

## <a name="how-to-contribute"></a>Cómo contribuir

Azure Digital Twins Explorer es una herramienta de **código abierto** que agradece las contribuciones al código y la documentación. La aplicación hospedada se implementa regularmente desde un repositorio de código fuente de GitHub.

Para ver el código fuente de la herramienta y leer instrucciones detalladas sobre cómo contribuir al código, visite su repositorio de GitHub: [digital-twins-explorer](https://github.com/Azure-Samples/digital-twins-explorer).

Para ver las instrucciones para contribuir a esta documentación, visite la [Guía para colaboradores de Docs](/contribute/).

## <a name="other-considerations"></a>Otras consideraciones

### <a name="region-support"></a>Regiones admitidas

Azure Digital Twins Explorer está disponible para su uso con todas las instancias de Azure Digital Twins en todas las [regiones admitidas](https://azure.microsoft.com/global-infrastructure/services/?products=digital-twins).

Sin embargo, durante la versión preliminar pública, los datos se pueden enviar para su procesamiento a través de regiones diferentes de la región donde se hospeda la instancia. Para evitar esto en situaciones en las que la soberanía de datos es un problema, puede descargar el [código fuente abierto](#how-to-contribute) para crear una versión hospedada localmente del explorador en su propia máquina.

### <a name="billing"></a>Facturación

Azure Digital Twins Explorer es una herramienta gratuita para interactuar con el servicio Azure Digital Twins. Aunque la propia herramienta es gratuita, se puede incurrir en costos durante el uso cuando se envían solicitudes al servicio Azure Digital Twins, el cual sigue estas directrices de precios: [Precios de Azure Digital Twins](https://azure.microsoft.com/pricing/details/digital-twins/).

## <a name="next-steps"></a>Pasos siguientes 

Aprenda a usar las características de Azure Digital Twins Explorer: [Procedimiento: Uso de Azure Digital Twins Explorer](how-to-use-azure-digital-twins-explorer.md).