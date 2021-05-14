---
title: Creación visual
description: Información acerca de cómo utilizar la creación visual de Azure Data Factory
ms.service: data-factory
ms.topic: conceptual
author: nabhishek
ms.author: abnarain
ms.date: 09/08/2020
ms.openlocfilehash: 3c5bbe128b03e1d4bfe562132656fee0df51423e
ms.sourcegitcommit: b4032c9266effb0bf7eb87379f011c36d7340c2d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107905399"
---
# <a name="visual-authoring-in-azure-data-factory"></a>Creación visual de Azure Data Factory

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

La experiencia de la interfaz de usuario (UX) de Azure Data Factory le permite crear e implementar visualmente recursos para la factoría de datos sin tener que escribir código. Puede arrastrar y colocar las actividades en un lienzo de canalización, realizar ejecuciones de prueba, depurar de forma iterativa e implementar y supervisar ejecuciones de canalizaciones.

Actualmente, la experiencia de usuario de Azure Data Factory solo se admite en Microsoft Edge y Google Chrome.

## <a name="authoring-canvas"></a>Lienzo de creación

Para abrir el **lienzo de creación**, haga clic en el icono de lápiz. 

![Lienzo de creación](media/author-visually/authoring-canvas.png)

Aquí crea las canalizaciones, las actividades, los conjuntos de datos, los servicios vinculados, los flujos de datos, los desencadenadores y los entornos de ejecución de integración que componen la fábrica. Para empezar a crear una canalización mediante el lienzo de creación, consulte [Copia de datos mediante la actividad de copia](tutorial-copy-data-portal.md). 

La experiencia de creación visual predeterminada funciona directamente con el servicio Data Factory. La integración de Git de Azure Repos o GitHub también es compatible para admitir el control del código fuente y la colaboración para trabajar en canalizaciones de factoría de datos. Para más información sobre las diferencias entre estas experiencias de creación, consulte [Control de código fuente en Azure Data Factory](source-control.md).

### <a name="properties-pane"></a>Propiedades, panel

En el caso de los recursos de nivel superior, como canalizaciones, conjuntos de datos y flujos de datos, las propiedades de alto nivel se pueden editar en el panel Propiedades del lado derecho del lienzo. El panel Propiedades contiene propiedades como el nombre, la descripción, las anotaciones y otras propiedades de alto nivel. Los subrecursos, como las actividades de canalización y las transformaciones de flujo de datos, se editan mediante el panel situado en la parte inferior del lienzo. 

![Propiedades, panel](media/author-visually/properties-pane.png)

El panel Propiedades solo se abre de forma predeterminada en la creación de recursos. Para editarlo, haga clic en el icono del panel Propiedades situado en la esquina superior derecha del lienzo.

### <a name="related-resources"></a>Recursos relacionados

En el panel de propiedades, si selecciona la pestaña **Relacionado**, podrá ver los recursos que dependen del recurso seleccionado. Aparecerán todos los recursos que hagan referencia al recurso actual.

![Recursos relacionados](media/author-visually/related-resources.png)

Por ejemplo, en la imagen anterior, hay una canalización y dos flujos de datos que utilizan el conjunto de datos que está seleccionado actualmente.

## <a name="management-hub"></a>Centro de administración

El centro de administración, al que accede la pestaña *Administrar* en la experiencia de usuario de Azure Data Factory, es un portal que hospeda acciones de administración globales para Data Factory. Aquí puede administrar las conexiones a almacenes de datos y procesos externos, la configuración del control de código fuente y la configuración del desencadenador. Para obtener más información, consulte las capacidades del [centro de administración](author-management-hub.md).

![Administración de servicios vinculados](media/author-management-hub/management-hub-linked-services.png)

## <a name="expressions-and-functions"></a>Expresiones y funciones

Se pueden usar expresiones y funciones en lugar de valores estáticos para especificar muchas propiedades en Azure Data Factory.

Para especificar una expresión para un valor de propiedad, seleccione **Agregar contenido dinámico** o haga clic en **Alt + P** mientras se centra en el campo.

![Incorporación de contenido dinámico](media/author-visually/dynamic-content-1.png)

Así se abre el **Generador de expresiones de Data Factory**, donde puede generar expresiones a partir de variables del sistema admitidas, salidas de actividad, funciones y variables o parámetros especificados por el usuario. 

![Generador de expresiones](media/author-visually/dynamic-content-2.png)

Para más información sobre el lenguaje de expresión, consulte [Expresiones y funciones de Azure Data Factory](control-flow-expression-language-functions.md).

## <a name="provide-feedback"></a>Envío de comentarios

Seleccione **Comentarios** para comentar sobre las características o para notificar a Microsoft sobre los problemas con la herramienta:

![Comentarios](media/author-visually/provide-feedback.png)

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre la supervisión y la administración de canalizaciones, consulte el artículo [Supervisión y administración de canalizaciones mediante programación](monitor-programmatically.md).
