---
title: Propiedades de marketing
description: Una descripción de los distintos campos de marketing recopilados en el portal y cómo aparecerán en el catálogo de Azure Certified Device.
author: nkuntjoro
ms.author: nikuntjo
ms.service: certification
ms.topic: conceptual
ms.date: 03/15/2021
ms.custom: template-concept
ms.openlocfilehash: 262616ca82e9c06baec0d7a1b81a0e3dff2ed8db
ms.sourcegitcommit: b4fbb7a6a0aa93656e8dd29979786069eca567dc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/13/2021
ms.locfileid: "107304518"
---
# <a name="marketing-properties"></a>Propiedades de marketing

En el proceso de [agregar los detalles del dispositivo](tutorial-02-adding-device-details.md), se le pedirá que proporcione información de marketing que se mostrará en el [catálogo de Azure Certified Device](https://devicecatalog.azure.com). Esta información se recopila en el portal de Azure Certified Device durante el proceso de envío de certificaciones y se usará como parámetros de filtro en el catálogo. En este artículo se proporciona una asignación entre los campos recopilados en el portal y el modo en que aparecen en el catálogo. Después de leer este artículo, los asociados comprenderán mejor qué información se debe proporcionar durante el proceso de certificación para representar mejor su producto en el catálogo.

![Información general sobre PDP](./media/concepts-marketing/pdp-overview.png)

## <a name="azure-certified-device-catalog-product-tile"></a>Icono de producto del catálogo de Azure Certified Device

Los visitantes del catálogo interactuarán primero con el dispositivo con el icono de producto del catálogo que aparece en la página de búsqueda. De esta forna, tendrá una visión general básica del dispositivo y las certificaciones que se le han concedido.

![Plantilla de icono del producto](./media/concepts-marketing/product-tile.png)

| Campo | Descripción                  | Dónde agregarlo en el portal                |
|---------------|-------------------------|----------------------------------|
| Device Name (Nombre del dispositivo) | Nombre público del dispositivo certificado         | Pestaña Aspectos básicos de Detalles del dispositivo|
| Nombre de la compañía| Nombre público de la empresa  | No se puede editar en el portal. Extraído del nombre de la cuenta de MPN |
| Product photo (Foto del producto)  | Imagen del dispositivo con la resolución mínima (200 ppp x 200 ppp)  | Detalles de marketing |
| Certification classification (Clasificación de la certificación)  | Etiqueta obligatoria de certificación de Azure Certified Device y distintivos de certificación opcionales  | Pestaña Aspectos básicos de detalles del dispositivo. Debe pasar las pruebas adecuadas de la sección Connect & test (Conectar y probar). |

## <a name="product-description-page-information"></a>Información de la página de descripción del producto

Una vez que un cliente ha hecho clic en el icono del dispositivo desde la página búsqueda del catálogo, se le dirigirá a la página Descripción del producto del dispositivo. Aquí es donde se encontrará la mayor parte de la información proporcionada durante el proceso de certificación.

En la parte superior de la página de descripción del producto se resaltan las características clave, algunas de las cuales ya se usaron en el icono del producto.

![Barra superior de PDP](./media/concepts-marketing/pdp-top.png)

| Campo | Descripción                  | Dónde agregarlo en el portal                |
|---------------|-------------------------|----------------------------------|
| Clase de dispositivo | Clasificación del factor de forma y el propósito principal de su dispositivo ([Más información](./resources-glossary.md))       | Pestaña Aspectos básicos de Detalles del dispositivo|
| Tipo de dispositivo | Clasificación del dispositivo en función de su preparación para la implementación ([Más información](./resources-glossary.md)) | Pestaña Aspectos básicos de Detalles del dispositivo |
| Disponibilidad de geoárea | Regiones en las que se puede comprar el dispositivo  | Detalles de marketing |
| Sistemas operativos  | Sistemas operativos compatibles con el dispositivo  | Pestaña Detalles del producto de Detalles del dispositivo |
| Target industries (Sectores objetivo)  | Los 3 sectores principales para los que está optimizado el dispositivo  | Detalles de marketing |
| Descripción del producto  | Campo de texto libre para que escriba su descripción de marketing del producto. Aquí puede incluir los detalles que no aparecen en el portal o agregar contexto adicional de las ventajas de usar el dispositivo. | Detalles de marketing|

El resto de la página va enfocada a la presentación de las especificaciones técnicas del dispositivo en formato de tabla, lo que ayudará a su cliente a entender mejor el producto. Por comodidad, la información que se muestra en la parte superior de la página también se muestra aquí. El resto de la tabla se encuentra dividida en los componentes especificados en el portal.

![Página inferior de PDP](./media/concepts-marketing/pdp-bottom.png)

| Campo | Descripción                  | Dónde agregarlo en el portal                |
|---------------|-------------------------|----------------------------------|
| Tipo de componente | Clasificación del factor de forma y el propósito principal de su dispositivo ([Más información](./resources-glossary.md))       | Detalles del producto de Detalles del dispositivo|
| Nombre de componente| Nombre del componente que describe | Detalles del producto de Detalles del dispositivo |
| Additional component information (Información adicional del componente) | Especificaciones de hardware adicionales, como sensores incluidos, conectividad, aceleradores, etc.  | Información adicional del componentes de Detalles del dispositivo ([Más información](./how-to-using-the-components-feature.md))  |
| Device dependency text (Texto de dependencia del dispositivo) | Texto proporcionado por el asociado que describe las diferentes dependencias que requiere el producto para conectarse a Azure ([Más información](./how-to-indirectly-connected-devices.md))   | Sección de comentarios orientados al cliente de la pestaña Dependencias de Detalles del dispositivo |
| Device dependency link (Vínculo de dependencia del dispositivo)  | Vínculo a un dispositivo certificado que requiere su producto actual  | Pestaña Dependencias de Detalles del dispositivo |

## <a name="shop-links"></a>Shop links (Vínculos de tiendas)
Tanto en el icono del producto como en la página Descripción del producto hay un botón Tienda. Cuando un cliente hace clic, se abre una ventana que le permite seleccionar un distribuidor (se le permite mostrar hasta 5 distribuidores). Una vez seleccionado, al cliente se le redirige a la dirección URL proporcionada por el asociado.

![Imagen de la experiencia emergente de la tienda](./media/concepts-marketing/shop.png)

| Campo | Descripción                  | Dónde agregarlo en el portal                |
|---------------|-------------------------|----------------------------------|
| Nombre del distribuidor. | Nombre del distribuidor que vende el producto | Detalles de marketing|
| Get Device| Vínculo a un sitio web externo donde el cliente puede comprar el dispositivo (o solicitar una oferta al distribuidor). Si el distribuidor es también el fabricante del dispositivo, puede ser el mismo que la página del fabricante del dispositivo. Si no existe una página de compra, se le redirigirá a la página del distribuidor para que el cliente se ponga en contacto con ellos directamente.  | Dirección URL de la página del producto del distribuidor en los detalles de marketing. Si no hay una página de compra, el vínculo se establecerá de forma predeterminada en dirección URL del distribuidor en los detalles de marketing. |

## <a name="external-links"></a>Vínculos externos
En la página Descripción del producto también se incluyen vínculos que llevan a sitios proporcionados por el asociado o a archivos que ayudan al cliente a comprender mejor el producto. Aparecen hacia la parte superior de la página, debajo del texto de descripción del producto. Los vínculos que se muestran variarán según los distintos tipos de dispositivos y programas de certificación.

| Vínculo | Descripción                  | Dónde agregarlo en el portal                |
|---------------|-------------------------|----------------------------------|
| Guía de introducción* | Archivo PDF con instrucciones para el usuario para conectarse y usar el dispositivo con los servicios de Azure | Agregar la sección de la guía "Introducción" del portal|
| URL del fabricante*|Vínculo a la página del fabricante. Esta página puede ser la página del producto específica del dispositivo o la página principal de la empresa si no está disponible una página de marketing. | Página de marketing del fabricante en los detalles de marketing |
| Modelo de dispositivo | Modelos públicos de DTDL para soluciones de IoT Plug and Play  | No se puede editar en el portal. El modelo de dispositivo debe cargarse en el [repositorio de modelos públicos](https://aka.ms/modelrepo)  |
| Código fuente del dispositivo | Dirección URL al código fuente de los tipos de dispositivo del kit de desarrollo| Pestaña Aspectos básicos de Detalles del dispositivo  |

 **Obligatorio para todos los dispositivos publicados*

## <a name="next-steps"></a>Pasos siguientes
Ahora que conoce cómo usamos la información que proporciona durante la certificación, está listo para certificar el dispositivo. Inicie el proyecto de certificación o vuelva a la fase detalles del dispositivo para agregar su propia información de marketing.

- [Comenzar el recorrido de certificación](./tutorial-00-selecting-your-certification.md)
- [Agregar detalles del dispositivo](./tutorial-02-adding-device-details.md)
