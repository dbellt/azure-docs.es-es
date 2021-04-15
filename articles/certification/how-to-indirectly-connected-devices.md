---
title: Certificación de conjuntos de dispositivos y dispositivos conectados indirectamente
titleSuffix: Azure Certified
description: Vea cómo enviar un dispositivo conectado indirectamente para su certificación.
author: cbroad
ms.author: cbroad
ms.date: 02/23/2021
ms.topic: how-to
ms.service: certification
ms.openlocfilehash: 3a4fd2838c0ddf6d7d03d68f105fc59471b77dea
ms.sourcegitcommit: b4fbb7a6a0aa93656e8dd29979786069eca567dc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/13/2021
ms.locfileid: "107304484"
---
# <a name="device-bundles-and-indirectly-connected-devices"></a>Conjuntos de dispositivos y dispositivos conectados indirectamente

Para admitir dispositivos que interactúan con Azure a través de un dispositivo, las ofertas de SaaS o PaaS, nuestro portal de envío (https://certify.azure.com/) y el catálogo de dispositivos (https://devicecatalog.azure.com) habilitan los conceptos de agrupación y dependencias para promocionar y habilitar el acceso de estas combinaciones de dispositivos al programa de Azure Certified Device.

En función de la línea de productos y los servicios ofrecidos, es posible que la situación requiera una combinación de estos pasos:


![Creación de las dependencias del proyecto](./media/indirect-connected-device/picture-1.png )
## <a name="sensors-and-indirect-devices"></a>Sensores y dispositivos indirectos
Muchos sensores requieren un dispositivo para conectarse a Azure. Además, puede tener varios dispositivos compatibles que funcionan con el dispositivo de sensor. **Para dar cabida a estos escenarios, primero debe certificar los dispositivos, antes de certificar el sensor que pasará información a través de ellos.**

Matriz de ejemplo de combinaciones de envío ![Ejemplo de envío](./media/indirect-connected-device/picture-2.png )

Para certificar el sensor, que requiere un dispositivo independiente:
1.  En primer lugar, [certifique el dispositivo](https://certify.azure.com) y publíquelo en el catálogo de Azure Certified Device
    - Si tiene varios dispositivos de acceso directo compatibles (como en el ejemplo anterior), envíelos por separado para la certificación y su publicación en el catálogo también
2.  Con el sensor conectado a través del dispositivo, envíe el sensor para su certificación
    * En la pestaña "Dependencias" de la sección "Detalles del dispositivo", establezca los valores siguientes
        * Tipo de dependencia = "Puerta de enlace de hardware"
        * URL de la dependencia = "Vínculo de dirección URL al dispositivo del catálogo de dispositivos"
        * Used during testing (Se usa en las pruebas) = "Sí"
        * Agregue cualquier comentario para el cliente que se deba proporcionar al usuario que vea la descripción del producto en el catálogo de dispositivos. (Por ejemplo: "Los dispositivos de la serie 100 son necesarios para que los sensores se conecten a Azure")

3.  Si tiene más dispositivos que le gustaría agregar como opcionales para este dispositivo, puede seleccionar "+ Add additional dependency" (Agregar otra dependencia). A continuación, siga las mismas instrucciones y tenga en cuenta que no se usó durante las pruebas. En los comentarios para el cliente, asegúrese de que los clientes saben que están disponibles otros dispositivos asociados a este sensor (como alternativa al dispositivo que se usó durante las pruebas).

![Texto alternativo](./media/indirect-connected-device/picture-3.png "Tipo de dependencia de hardware")

## <a name="paas-and-saas-offerings"></a>Ofertas de PaaS y SaaS
Como parte de la cartera de productos, puede haber dispositivos que certifique, pero el dispositivo también necesita otros servicios de la empresa o de otras compañías de terceros. Para agregar esta dependencia, siga estos pasos:
1. Inicie el proceso de envío del dispositivo.
2. En la pestaña "Dependencias", establezca los valores siguientes
    - Tipo de dependencia = "Servicio de software"
    - Nombre del servicio = "[nombre del producto]"
    - URL de la dependencia = "Vínculo de dirección URL a la página del producto que describe el servicio"
    - Agregue cualquier comentario para el cliente que se deba proporcionar al usuario que vea la descripción del producto en el catálogo de Azure Certified Device.
3. Si tiene otras dependencias de software, servicios o hardware que le gustaría agregar como opcionales para este dispositivo, puede seleccionar "+ Add additional dependency" (Agregar dependencia adicional) y seguir las mismas instrucciones.

![Tipo de dependencia de software](./media/indirect-connected-device/picture-4.png )

## <a name="bundled-products"></a>Productos agrupados
Las listas de productos agrupados son simplemente la certificación correcta de un dispositivo con otros componentes que se van a vender como parte de la agrupación en una lista de productos. Tiene la capacidad de enviar un dispositivo que incluye componentes adicionales, como un sensor de temperatura y un sensor de cámara (n.º 1), o bien puede enviar un sensor táctil que incluya un dispositivo de acceso directo (n.º 2). Con la característica "Componente", tiene la posibilidad de agregar varios componentes a la lista.

Si tiene previsto hacerlo, aplique formato a la imagen de la lista de productos para indicar que este producto incluye otros componentes.  Además, si su agrupación requiere servicios adicionales para la certificación, deberá identificarlos a través de la dependencia de servicios.
Matriz de ejemplo de productos agrupados

![Ejemplo de envío de agrupación](./media/indirect-connected-device/picture-5.png )

Para obtener una descripción más detallada de sobre cómo usar la funcionalidad de componentes en el portal de Azure Certified Device, consulte nuestra [documentación de ayuda](./how-to-using-the-components-feature.md). 

Si se trata de un dispositivo de acceso directo con un sensor independiente en el mismo producto, cree un componente para reflejar el dispositivo de acceso directo y otro componente para reflejar el sensor. Los componentes se pueden agregar al proyecto en la pestaña Detalles del producto de la sección Detalles del dispositivo:

![Adición de componentes](./media/indirect-connected-device/picture-6.png )

Para el dispositivo de acceso directo, establezca Tipo de componente como Producto listo para el cliente y rellene los demás campos según sea necesario para su producto. Ejemplo:

![Detalles del componente](./media/indirect-connected-device/picture-7.png )

En el caso del sensor, agregue un segundo componente y establezca Tipo de componente como Periférico y Método de datos adjuntos como Discreto. Ejemplo:

![Detalles del segundo componente](./media/indirect-connected-device/picture-8.png )

Una vez creado el componente de sensor, edite los detalles, navegue a la pestaña Sensores y, a continuación, agregue los detalles del sensor. Ejemplo:

![Detalles del sensor](./media/indirect-connected-device/picture-9.png )

Complete los detalles de los proyectos y envíe el dispositivo para la certificación de la forma habitual.

