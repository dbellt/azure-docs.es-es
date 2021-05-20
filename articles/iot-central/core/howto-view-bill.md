---
title: Administración de la factura y conversión del plan de tarifa gratis en la aplicación de Azure IoT Central | Microsoft Docs
description: Aprenda a administrar su factura y a pasar del plan de tarifa gratis a un plan de tarifa estándar en su aplicación de Azure IoT Central.
author: dominicbetts
ms.author: dobett
ms.date: 11/23/2019
ms.topic: how-to
ms.service: iot-central
services: iot-central
manager: philmea
ms.openlocfilehash: e8c268965f94f5d362fb4874566e10f39ee80b18
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108747240"
---
# <a name="manage-your-bill-in-an-iot-central-application"></a>Administrar la factura en una aplicación de IoT Central

En este artículo se describe cómo se puede administrar la facturación de Azure IoT Central. Puede trasladar su aplicación del plan de tarifa gratis a un plan de tarifa estándar y también actualizar o degradar el plan de tarifa.

Para acceder a la sección **Administración**, debe disponer del rol *Administrador* o de un *rol de usuario personalizado* que le permite ver la facturación. Si crea una aplicación de Azure IoT Central, se le asigna automáticamente el rol **Administrador**.

## <a name="move-from-free-to-standard-pricing-plan"></a>Cambio del plan de tarifa gratis al estándar

- Las aplicaciones que usan el plan de tarifa gratis mantienen la gratuidad durante siete días antes de expirar. Para evitar la pérdida de datos, puede trasladarlas a un plan de tarifa estándar en cualquier momento antes de que expiren.
- Las aplicaciones que usan un plan de tarifa estándar se cobran por dispositivo, con los dos primeros dispositivos gratis, y por aplicación.

Obtenga más información sobre los precios en la [página de precios de Azure IoT Central](https://azure.microsoft.com/pricing/details/iot-central/).

En la sección de precios, puede trasladar la aplicación del plan de tarifa gratuito a un plan de precios estándar.

Para completar el proceso de autoservicio, siga estos pasos:

1. Vaya a la página **Precios** de la sección **Administración**.

    :::image type="content" source="media/howto-view-bill/freetrialbilling.png" alt-text="Evaluación gratuita":::

1. Seleccione **Convert to a paid plan** (Convertir en plan de pago).

    :::image type="content" source="media/howto-view-bill/convert.png" alt-text="Conversión de la evaluación gratuita":::

1. Seleccione la instancia de Azure Active Directory correspondiente y, a continuación, la suscripción de Azure que se usará con la aplicación sujeta a un plan de pago.

1. Después de seleccionar **Convertir**, la aplicación usará un plan de pago y se le comenzará a facturar.

> [!Note]
> De forma predeterminada, su suscripción se convertirá a un plan de tarifa *Estándar 2*.

## <a name="how-to-change-your-application-pricing-plan"></a>Cómo cambiar el plan de tarifa de la aplicación

Las aplicaciones que usan un plan de tarifa estándar se cobran por dispositivo, con los dos primeros dispositivos gratis, y por aplicación.

En la sección de precios, puede actualizar o degradar el plan de tarifa de Azure IoT en cualquier momento.

1. Vaya a la página **Precios** de la sección **Administración**.

    :::image type="content" source="media/howto-view-bill/pricing.png" alt-text="Actualización del plan de precios":::

1. Seleccione el **Plan** y, luego, seleccione **Guardar** para actualizarlo o degradarlo.

## <a name="view-your-bill"></a>Ver la factura

1. Seleccione la instancia de Azure Active Directory correspondiente y, a continuación, la suscripción de Azure que se usará con la aplicación sujeta a un plan de pago.

1. Después de seleccionar **Convertir**, la aplicación usará un plan de pago y se le comenzará a facturar.

> [!Note]
> De forma predeterminada, su suscripción se convertirá a un plan de tarifa *Estándar 2*.

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha aprendido a administrar su factura en la aplicación Azure IoT Central, el siguiente paso sugerido es aprender sobre la [personalización de la interfaz de usuario de la aplicación](howto-customize-ui.md) en Azure IoT Central.