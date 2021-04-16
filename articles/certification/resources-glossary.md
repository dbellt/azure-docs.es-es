---
title: Glosario del programa Azure Certified Device
description: Una lista de términos comunes usados en el programa Azure Certified Device
author: nkuntjoro
ms.author: nikuntjo
ms.service: certification
ms.topic: conceptual
ms.date: 03/03/2021
ms.custom: template-concept
ms.openlocfilehash: f58c29c6bd2c22f37d2285ac6e93c6f54e47c4e0
ms.sourcegitcommit: b4fbb7a6a0aa93656e8dd29979786069eca567dc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/13/2021
ms.locfileid: "107305096"
---
# <a name="azure-certified-device-program-glossary"></a>Glosario del programa Azure Certified Device

En esta guía se proporcionan definiciones de los términos de uso común en el programa Azure Certified Device y en su portal. Consulte este glosario para obtener aclaraciones sobre el proceso de certificación. Por su comodidad, este glosario se clasifica según los principales conceptos de certificación que le pueden plantear dudas.

## <a name="device-class"></a>Clase de dispositivo

Al crear el proyecto de certificación, se le pedirá que especifique una clase de dispositivo. La clase de dispositivo hace referencia al factor de forma o a la clasificación que mejor represente el dispositivo.

- **Puerta de enlace**

    Dispositivo que procesa los datos enviados a través de una red de IoT.

- **Sensor**

    Dispositivo que detecta los cambios de un entorno y responde a ellos, y se conecta a las puertas de enlace para procesar los cambios.

- **Otros**

    Si selecciona Otros, agregue una descripción de la clase de dispositivo con sus propias palabras. Con el tiempo, es posible que podamos seguir agregando nuevos valores a esta lista, especialmente a medida que continuamos supervisando los comentarios de nuestros asociados.

## <a name="device-type"></a>Tipo de dispositivo

También se le pedirá que seleccione uno de los dos tipos de dispositivo durante el proceso de certificación.

- **Producto terminado**

    Dispositivo preparado para la solución y listo para la implementación en producción. Normalmente, en un factor de forma terminado con el firmware y un sistema operativo. Pueden ser dispositivos de uso general que requieren personalización adicional o dispositivos especializados que no requieren modificaciones para su uso.
- **Kit de desarrollo preparado para la solución**

    Un kit de desarrollo que contiene hardware y software ideales para una sencilla creación de prototipos, normalmente sin un factor de forma terminado. Suele incluir código de ejemplo y tutoriales para habilitar la creación rápida de prototipos.

## <a name="component-type"></a>Tipo de componente

En la sección Detalles del dispositivo, describa el dispositivo mediante la lista de componentes por tipo de componente. Puede consultar más instrucciones sobre los componentes [aquí](./how-to-using-the-components-feature.md).

- **Producto preparado para el cliente**

    Una representación de los componentes del dispositivo primario o general. Es diferente de un **producto terminado**, que es una clasificación del dispositivo como listo para que el cliente lo utilice sin más desarrollo. Un producto terminado contendrá un componente del producto preparado para el cliente.
- **Panel de desarrollo**

    Un panel integrado o desconectable con microprocesador para facilitar la personalización.
- **Periférico**

    Una adición integrada o desconectable al producto (por ejemplo, un accesorio). Normalmente se trata de dispositivos que se conectan al dispositivo principal, pero no contribuye a las funciones principales del dispositivo. En su lugar, proporciona funciones adicionales. La memoria, la RAM, el almacenamiento, los discos duros y las CPU no se consideran dispositivos periféricos (en su lugar, deben aparecer en las especificaciones adicionales del componente del producto preparado para el cliente).
- **Sistema en módulo**  

    Circuito de nivel de placa que integra una función del sistema en un único módulo.

## <a name="component-attachment-method"></a>Método de acoplamiento de componentes

El método de acoplamiento de componentes es otro detalle de los componentes que informa al cliente sobre cómo se integra el componente en el producto general.

- **Integrado**
 
    Se refiere a cuando un componente del dispositivo forma parte del chasis principal del producto. Normalmente, esto hace referencia a un tipo de componente periférico que no se puede quitar del dispositivo.  
    Ejemplo: un sensor de temperatura integrado en un chasis de puerta de enlace.

- **Discrete**

    Se refiere a cuando un componente **no** forma parte del chasis principal del producto.  
    Ejemplo: un sensor de temperatura externo que debe estar conectado al dispositivo.


## <a name="next-steps"></a>Pasos siguientes

Este glosario le guiará en el proceso de certificación del proyecto en el portal. Ya está listo para empezar con el proyecto.
- [Tutorial: Creación de un proyecto](./tutorial-01-creating-your-project.md)
