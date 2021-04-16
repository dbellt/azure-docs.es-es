---
title: 'Programa Azure Certified Device: Tutorial sobre la incorporación de detalles del dispositivo'
description: Una guía paso a paso para agregar detalles del dispositivo al proyecto en el portal de Azure Certified Device
author: nkuntjoro
ms.author: nikuntjo
ms.service: certification
ms.topic: tutorial
ms.date: 03/02/2021
ms.custom: template-tutorial
ms.openlocfilehash: f4f3d045a2530fa54d22bec789918454cba80097
ms.sourcegitcommit: b4fbb7a6a0aa93656e8dd29979786069eca567dc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/13/2021
ms.locfileid: "107310434"
---
# <a name="tutorial-add-device-details"></a>Tutorial: Incorporación de detalles del dispositivo

Ya ha creado el proyecto para el dispositivo y está listo para comenzar el proceso de certificación. Primero, agregue los detalles del dispositivo. Este paso incluye las especificaciones técnicas que sus clientes podrán ver en el catálogo de Azure Certified Device y la información comercial que usarán para hacer la compra una vez que tomen una decisión.

En este tutorial aprenderá a:

> [!div class="checklist"]
> * Agregar detalles del dispositivo mediante las características de componentes y dependencias
> * Cargar una guía de introducción para el dispositivo
> * Especificar la información comercial para que los clientes compren el dispositivo
> * Identificar certificaciones del sector (opcional)

## <a name="prerequisites"></a>Requisitos previos

* Debe haber iniciado sesión y tener un proyecto creado para su dispositivo en el [portal de Azure Certified Device](https://certify.azure.com). Para más información, consulte el [tutorial](tutorial-01-creating-your-project.md).
* Debe disponer de una guía de introducción para el dispositivo en formato PDF. Se le proporcionarán plantillas de introducción para que las utilice en función del programa de certificación y el idioma que prefiera. Las plantillas están disponibles en nuestra ubicación de GitHub de [introducción a las plantillas](https://aka.ms/GSTemplate "Introducción a las plantillas").

## <a name="adding-technical-device-details"></a>Incorporación de detalles técnicos del dispositivo

La primera sección de la página del proyecto, denominada "Input device details" (Introducción de detalles del dispositivo), le permite proporcionar información sobre las principales funciones de hardware del dispositivo, como el nombre del dispositivo, la descripción, el procesador, el sistema operativo, las opciones de conectividad, las interfaces de hardware, los protocolos del sector, las dimensiones físicas, etc. Aunque muchos de los campos son opcionales, la mayor parte de esta información estará disponible para los posibles clientes en el catálogo de Azure Certified Device si decide publicar el dispositivo una vez que se haya certificado.

1. Haga clic en `Add`, en la sección "Input device details" (Introducción de detalles del dispositivo) de la página de resumen del proyecto para abrir la sección de detalles del dispositivo. Verá cinco secciones para completar.

![Imagen de la página de detalles del proyecto](./media/images/device-details-menu.png)

2. Revise la información que proporcionó anteriormente al crear el proyecto en la pestaña `Basics`.
1. Revise las certificaciones que solicita para el dispositivo en la pestaña `Certifications`.
1. Abra la pestaña `Product details` y seleccione al menos un sistema operativo.
1. Agregue **al menos** un componente específico que describa el dispositivo. Puede obtener directrices adicionales sobre el uso de componentes [aquí](how-to-using-the-components-feature.md).
1. Haga clic en `Save`. Después, podrá editar el dispositivo componente y agregar detalles más avanzados.
1. Muestre detalles adicionales del dispositivo no capturados en los detalles del componente en `Additional product details`.
1. Si ha marcado `Other` en cualquiera de los campos del componente o se da alguna circunstancia especial sobre la que desea llamar la atención del equipo de certificación de Azure, incluya un comentario aclaratorio en la sección `Comments for reviewer`.
1. Use la pestaña `Dependencies` para mostrar las dependencias si el dispositivo requiere hardware o servicios adicionales para enviar datos a Azure. Puede obtener directrices adicionales sobre cómo mostrar las dependencias [aquí](how-to-indirectly-connected-devices.md).
1. Una vez que esté satisfecho con la información que ha proporcionado, puede usar la pestaña `Review` para obtener información general de solo lectura del conjunto completo de detalles del dispositivo que se ha especificado.
1. Haga clic en `Project summary`, en la parte superior de la página, para volver a la página de resumen.

![Revisión de la página de detalles del proyecto](./media/images/sample-device-details.png)

## <a name="uploading-a-get-started-guide"></a>Carga de una guía de introducción

La guía de introducción es un documento PDF que ayuda a simplificar la instalación, configuración y administración de su producto. La finalidad es facilitar a los clientes la conexión y el uso de dispositivos en Azure con ayuda de su dispositivo. Como parte del proceso de certificación, es necesario que nuestros asociados proporcionen **una** guía de introducción para su programa de certificación más destacado.

1. Asegúrese bien de haber proporcionado toda la información solicitada en el PDF de la guía de introducción según las [plantillas](https://aka.ms/GSTemplate)proporcionadas. La plantilla que use debe venir determinada por el distintivo de certificación que solicite. (Por ejemplo, un dispositivo de IoT Plug and Play usará la plantilla correspondiente. Los dispositivos para los que *solo* se solicite la certificación base de referencia Azure Certified Device usarán la plantilla correspondiente).
1. Haga clic en `Add`, en la sección de la guía de "introducción" de la página de resumen del proyecto.

![Imagen del botón GSG](./media/images/gsg-menu.png)

2. Haga clic en "Elegir archivo" para cargar el PDF.
1. Revise el documento en la vista previa para comprobar el formato.
1. Para guardar la carga, haga clic en el botón "Guardar".
1. Haga clic en `Project summary`, en la parte superior de la página, para volver a la página de resumen.

## <a name="providing-marketing-details"></a>Incorporación de la información comercial

En esta sección, proporcionará la información comercial de su dispositivo para el cliente. Estos campos se mostrarán en el catálogo de Azure Certified Device si decide publicar el dispositivo certificado.

1. Haga clic en `Add` en la sección "Add marketing details" (Agregar información comercial) para abrir la página de información comercial.

![Imagen de la sección de información comercial](./media/images/marketing-details.png)

1. Cargue una fotografía del producto en formato JPEG o PNG para utilizarla en el catálogo.
1. Escriba una breve descripción del dispositivo para mostrarla en la página de descripción del producto del catálogo.
1. Indique la disponibilidad geográfica del dispositivo.
1. Incluya un vínculo a la página comercial del fabricante del dispositivo. Debe ser un vínculo a un sitio con información adicional sobre el dispositivo.
    > [!Note]
    > Asegúrese de que todas las direcciones URL proporcionadas sean válidas o que estarán activas en el momento de aprobar la publicación.*)

1. Indique un máximo de 3 sectores objetivo para los que el dispositivo se ha optimizado.
1. Proporcione información para un máximo de 5 distribuidores del dispositivo. Se puede incluir el propio sitio del fabricante.

    > [!Note]
    > Si no se proporciona una URL de la página del producto del distribuidor, el botón `Shop` del catálogo se establecerá de forma predeterminada en el vínculo proporcionado para `Distributor page`, que puede no ser específico del dispositivo. La URL del distribuidor debería dirigir idealmente a una página específica en la que un cliente puede comprar un dispositivo, pero no es obligatorio. Si el distribuidor es también el fabricante del dispositivo, esta URL puede coincidir con la página del fabricante del dispositivo.*)

1. Haga clic en `Save` para confirmar la información.
1. Haga clic en `Project summary`, en la parte superior de la página, para volver a la página de resumen.

## <a name="declaring-additional-industry-certifications"></a>Declaración de certificaciones adicionales del sector

También puede promover otras certificaciones del sector que haya recibido para el dispositivo. Estas certificaciones pueden aportar mayor claridad sobre el uso previsto del dispositivo y se podrán buscar en el catálogo de Azure Certified Device.

1. Haga clic en `Add`, en la sección "Provide industry certifications" (Proporcionar certificaciones del sector).
1. Haga clic en `Add a certification` para seleccionar los programas de certificación más comunes del sector. Si el producto ha obtenido una certificación que no aparece en nuestra lista, puede especificar un valor de cadena personalizado. Para ello, seleccione `Other (please specify)`.
1. De manera opcional, puede proporcionar una descripción o notas al revisor. Sin embargo, estas notas no estarán disponibles públicamente para verlas en el catálogo.
1. Haga clic en `Save` para confirmar la información.
1. Haga clic en `Project summary`, en la parte superior de la página, para volver a la página de resumen.

## <a name="next-steps"></a>Pasos siguientes

Ya ha completado el proceso de descripción del dispositivo. Esto ayudará al equipo de revisión de Azure Certified Device y a su cliente a comprender mejor el producto. Una vez que esté satisfecho con la información que ha proporcionado, podrá pasar a la fase de pruebas del proceso de certificación.
> [!div class="nextstepaction"]
> [Tutorial: Pruebas del dispositivo](tutorial-03-testing-your-device.md)
