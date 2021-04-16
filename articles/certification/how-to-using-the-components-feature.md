---
title: Uso de la característica de componentes en el portal de Azure Certified Device
description: Una guía sobre cómo usar mejor la característica de componentes de la sección Detalles del dispositivo para describir el dispositivo con precisión
author: nkuntjoro
ms.author: nikuntjo
ms.service: certification
ms.topic: how-to
ms.date: 03/03/2021
ms.custom: template-how-to
ms.openlocfilehash: 220a6c2107063734201064115898611c20cab650
ms.sourcegitcommit: b4fbb7a6a0aa93656e8dd29979786069eca567dc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/13/2021
ms.locfileid: "107304467"
---
# <a name="add-components-on-the-portal"></a>Incorporación de componentes en el portal

Al completar el [tutorial para agregar detalles del dispositivo](tutorial-02-adding-device-details.md) al proyecto de certificación, se espera que describa las especificaciones de hardware del dispositivo. Para ello, los usuarios pueden resaltar varios productos de hardware independientes (denominados **componentes**) que componen el dispositivo. Esto le permite promover mejor los dispositivos que acompañan al hardware adicional y permite que los clientes encuentren el producto adecuado buscando en el catálogo en función de estas características.

## <a name="prerequisites"></a>Requisitos previos

- Debe haber iniciado sesión y tener un proyecto creado para su dispositivo en el [portal de Azure Certified Device](https://certify.azure.com). Para más información, consulte el [tutorial](tutorial-01-creating-your-project.md).

## <a name="how-to-add-components"></a>Incorporación de componentes

Todos los proyectos enviados para la certificación incluirán un componente **Producto preparado para el cliente** (que, en muchos casos, representará el producto holístico en sí). Para comprender mejor la distinción de un tipo de componente [Producto preparado para el cliente](./resources-glossary.md), consulte nuestro glosario de certificación. Todos los componentes adicionales quedan a su discreción para incluirlos para capturar su dispositivo con precisión.

1. Seleccione `Add a component` en la pestaña Detalles del producto.

    ![Incorporación de un vínculo de componente](./media/images/add-a-component-link.png)

1. Complete los campos de formulario pertinentes para el componente.

    ![Sección Detalles del componente](./media/images/component-details-section.png)

1. Guarde la información mediante el botón `Save Product Details` situado en la parte inferior de la página:  

    ![Botón Guardar detalles del producto](./media/images/save-product-details-button.png)

1. Una vez que haya guardado el componente, puede personalizar aún más las funcionalidades del hardware que admite. Seleccione el vínculo `Edit` por el nombre de componente.  

    ![Botón Editar componente](./media/images/component-edit.png)

1. Proporcione la información de funcionalidad de hardware pertinente cuando corresponda.  

    ![Imagen de secciones de componentes editables](./media/images/component-selection-area.png)  

    Los campos de componente editables (mostrados anteriormente) incluyen:

    - **General**: detalles de hardware, como procesadores y hardware seguro
    - **Conectividad**: opciones de conectividad, protocolos e interfaces, como radios y GPIO
    - **Aceleradores**: especifique la aceleración de hardware, como GPU y VPU
    - **Sensores**: especifique los sensores disponibles, como GPS y vibración
    - **Especificaciones adicionales**: información adicional sobre el dispositivo, como las dimensiones físicas y la información de almacenamiento y la batería

1. Seleccione `Save Product Details` en la parte inferior de la página Detalles del producto.

## <a name="component-use-requirements-and-recommendations"></a>Requisitos y recomendaciones de uso de componentes

Es posible que tenga dudas sobre el número de componentes que se van a incluir o el tipo de componente que se va a usar. A continuación, se muestran ejemplos de algunos escenarios de ejemplo de dispositivos que puede estar certificando y cómo puede utilizar la característica de componentes.

| Tipo de producto                                       | No. Componentes | 1 componente /Tipo de acoplamiento      | Más de 2 componentes/Tipo de acoplamiento                    |
|----------------------------------------------------|------------|----------------------------------|--------------------------------------------------|
| Producto finalizado                                   | 1          | Producto preparado para el cliente, discreto | N/D                                              |
| Producto finalizado con **periféricos que se pueden desconectar** | 2 o más  | Producto preparado para el cliente, discreto | Periférico/Discreto o integrado              |
| Producto finalizado con **componentes integrados**  | 2 o más  | Producto preparado para el cliente, discreto | Seleccionar tipo apropiado, discreto o integrado |
| Kit de desarrollo preparado para la solución                             | 2 o más  | Producto preparado para el cliente, discreto | Seleccionar tipo apropiado, discreto o integrado |

## <a name="example-component-usage"></a>Ejemplo de uso de componentes

A continuación se muestran ejemplos de cómo un OEM llamado Contoso usaría la característica de componentes para certificar su producto, denominado Falcon.

1. Falcon es un dispositivo independiente completo que no se integra en un producto más grande.
    1. No. de componentes: 1
    1. Tipo de dispositivo de componente: producto preparado para el cliente
    1. Tipo de acoplamiento: discretos

     ![Imagen del producto preparado para el cliente](./media/images/customer-ready-product.png)

1. Falcon es un dispositivo que incluye un módulo de cámara periférica integrado fabricado por INC Electronics que se conecta a través de USB a Falcon.
    1. No. de componentes: 2
    1. Tipo de dispositivo de componente: producto preparado para el cliente, periférico
    1. Tipo de acoplamiento: discreto, integrado
    
    > [!Note]
    > El componente periférico se considera integrado porque no es extraíble.

     ![Imagen del componente de periférico de ejemplo](./media/images/peripheral.png)

1. Falcon es un dispositivo que incluye un sistema integrado en el módulo de INC Electronics que usa un Apollo52 de procesador incorporado de la empresa Espressif y tiene una arquitectura ARM64.
    1. No. de componentes: 2
    1. Tipo de dispositivo de componente: producto preparado para el cliente, sistema en módulo
    1. Tipo de acoplamiento: discreto, integrado

    > [!Note]
    > El componente periférico se considera integrado porque no es extraíble. El componente SoM también incluiría información del procesador.

     ![Imagen del componente del sistema en módulo de ejemplo ](./media/images/system-on-module.png)

## <a name="additional-tips"></a>Consejos adicionales

A continuación se proporcionan más aclaraciones con respecto a nuestra directiva de uso de componentes. Si tiene alguna pregunta sobre el uso adecuado de los componentes, póngase en contacto con nuestro equipo en [iotcert@microsoft.com](mailto:iotcert@microsoft.com) y estaremos encantados de ayudarle.

1. Un proyecto **solamente** debe contener un componente de producto preparado para el cliente. Si está certificando un proyecto con dos dispositivos independientes, dichos dispositivos deben estar certificados por separado.
1. Depende principalmente de usted usar (o no usar) componentes para promover las funcionalidades de su dispositivo a clientes potenciales.
1. Durante nuestra revisión de su dispositivo, el equipo de certificación de Azure solo requerirá que se enumere al menos un componente de Producto listo para el cliente. Sin embargo, podemos solicitar modificaciones a la información del componente si los detalles no son claros o parece que faltan (por ejemplo, el fabricante del componente no se proporciona para un tipo de producto preparado para el cliente).

## <a name="next-steps"></a>Pasos siguientes

Ahora que está listo para usar nuestra característica de componentes, ya está listo para completar los detalles del dispositivo o editar el proyecto para mayor claridad.

- [Tutorial: Incorporación de detalles del dispositivo](tutorial-02-adding-device-details.md)
- [Edición del dispositivo publicado](how-to-edit-published-device.md)

