---
title: Descuentos de reserva para Azure App Service
description: Obtenga información sobre cómo se aplican los descuentos de reserva a instancias de Azure App Service Premium v3 y Premium v2 y unidades de escalado aisladas.
author: yashesvi
ms.reviewer: yashar
ms.service: cost-management-billing
ms.subservice: reservations
ms.topic: conceptual
ms.date: 05/13/2021
ms.author: banders
ms.openlocfilehash: b6230f86fc33dca290f6d61f923024c9352d8600
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110378515"
---
# <a name="how-reservation-discounts-apply-to-azure-app-service"></a>Aplicación de los descuentos de reserva a Azure App Service

Este artículo ayuda a entender cómo se aplican los descuentos a instancias de Azure App Service Premium v3 y Premium v2 y unidades de escalado aisladas.

## <a name="how-reservation-discounts-apply-to-premium-v3-instances"></a>Aplicación de descuentos de reserva a instancias de Premium v3

Después de comprar una instancia reservada de Premium v3 de Azure App Service, el descuento de reserva se aplica automáticamente a las instancias de App Service que coincidan con los atributos y la cantidad de la reserva. Una reserva cubre el costo de las instancias de Premium v3. 

### <a name="how-the-discount-is-applied-to-azure-app-service"></a>Aplicación de descuento a Azure App Service 

Un descuento por reserva es para *usarlo o perderlo*. Por lo tanto, si no tiene recursos coincidentes para ninguna hora, perderá una cantidad de reserva para esa hora. No se pueden arrastrar las horas reservadas no utilizadas.
Al cerrar un recurso, el descuento por reserva se aplica automáticamente a otro recurso que coincida con el ámbito especificado. Si no se encuentran recursos coincidentes en el ámbito especificado, las horas reservadas se pierden.

### <a name="reservation-discount-for-premium-v3-instances"></a>Descuento de reserva para instancias de Premium v3

El descuento de reserva de Azure se aplica a las instancias de Premium v3 en ejecución en períodos de una hora. Las reservas que haya comprado se vinculan con el uso emitido por las instancias de Premium v3 en ejecución para aplicar el descuento de reserva. Para las instancias de Premium v3 que no se pueden ejecutar la hora completa, se rellenará la reserva desde otras instancias que no usen una reserva, incluidas las que se ejecutan simultáneamente. Al final de la hora, se bloquea la aplicación de reserva para las instancias en dicha hora. En el caso de que una instancia no se ejecute durante una hora o que haya instancias simultáneas dentro de la hora que no llenen la hora de reserva, la reserva estará infrautilizada durante esa hora. En el siguiente gráfico se muestra la aplicación de una reserva a un uso de máquina virtual facturable. La ilustración se basa en una compra de reserva y dos instancias de máquina virtual coincidentes.

![Imagen que muestra la aplicación de una reserva a un uso de máquina virtual facturable](./media/reservation-discount-app-service/reserved-premium-v3-instance-application.png)

1.  Cualquier uso por encima de la línea de reservas se cobrará según las tarifas normales de pago por uso. No se le cobrará por ningún uso por debajo de la línea de reservas, ya que el pago se ha hecho como parte de la compra de la reserva.
2.  En la hora 1, la instancia 1 se ejecuta durante 0,75 horas y la instancia 2 se ejecuta durante 0,5 horas. El uso total de la hora 1 es de 1,25 horas. Se le cobrará según las tarifas de pago por uso para las 0,25 horas restantes.
3.  Para las horas 2 y 3, ambas instancias se ejecutaron durante 1 hora cada una. Una instancia está cubierta por la reserva, mientras que la otra se cobra según las tarifas de pago por uso.
4.  Durante la hora 4, la instancia 1 se ejecuta durante 0,5 horas y la instancia 2 se ejecuta durante 1 hora. La instancia 1 está cubierta al completo por la reserva y 0,5 horas de la instancia 2 también. Se le cobrará según la tarifa de pago por uso por las 0,5 horas restantes.

Para comprender y ver la aplicación de Azure Reservations en los informes de uso de facturación consulte [Información sobre el uso de reservas](understand-reserved-instance-usage-ea.md).

## <a name="how-reservation-discounts-apply-to-isolated-v2-instances"></a>Aplicación de descuentos de reserva a instancias de App Service aislado v2

Después de comprar una instancia reservada de Azure App Service aislado v2, el descuento de reserva se aplica automáticamente a las instancias de App Service que coinciden con los atributos y la cantidad de la reserva. Una reserva cubre el costo de las instancias de App Service aislado v2.

### <a name="how-the-discount-is-applied-to-azure-app-service"></a>Aplicación de descuento a Azure App Service

Un descuento por reserva es para _usarlo o perderlo_. Por lo tanto, si no tiene recursos coincidentes para ninguna hora, perderá una cantidad de reserva para esa hora. No se pueden arrastrar las horas reservadas no utilizadas. Al cerrar un recurso, el descuento por reserva se aplica automáticamente a otro recurso que coincida con el ámbito especificado. Si no se encuentran recursos coincidentes en el ámbito especificado, las horas reservadas se pierden.

### <a name="reservation-discount-for-isolated-v2-instances"></a>Descuento de reserva para instancias de App Service aislado v2

El descuento de reserva de Azure se aplica a las instancias de App Service aislado v2 en ejecución por hora. Las reservas compradas se comparan con el uso emitido por las instancias de App Service aislado v2 en ejecución a fin de aplicar el descuento de reserva. En el caso de las instancias de App Service aislado v2 que no se ejecuten la hora completa, se completa la reserva con otras instancias que no usen una reserva, incluidas las que se ejecutan simultáneamente. Al final de la hora, se bloquea la aplicación de reserva para las instancias en dicha hora. En el caso de que una instancia no se ejecute durante una hora o que haya instancias simultáneas dentro de la hora que no llenen la hora de reserva, la reserva estará infrautilizada durante esa hora.

Para comprender y ver la aplicación de Azure Reservations en los informes de uso de facturación consulte [Información sobre el uso de reservas](understand-reserved-instance-usage-ea.md).

## <a name="how-reservation-discounts-apply-to-isolated-stamps"></a>Aplicación de los descuentos de reserva a los timbres en entorno aislado

Cuando compra capacidad reservada en el impuesto sobre el timbre de App Service en entorno aislado, el descuento de reserva se aplica automáticamente al impuesto sobre el timbre de una región. El descuento de reserva se aplica al uso que emite el medidor de impuesto sobre el timbre en entorno aislado. Los trabajos, los front-ends adicionales y los demás recursos asociados al timbre se siguen facturando con la tarifa normal.

### <a name="reservation-discount-application"></a>Aplicación del descuento de reserva

El descuento en el impuesto sobre el timbre de App Service en entorno aislado se aplica a la ejecución de timbres aislados por hora. Si no tiene un timbre implementado durante una hora, se pierde la capacidad reservada para esa hora. No se transfiere.

Después de la compra, la reserva que haya adquirido se hará coincidir con un timbre en entorno aislado que se ejecuta en una región específica. Si apaga ese timbre, los descuentos de reserva se aplican automáticamente a cualquier otro timbre que se ejecute en la región. Si no existe ningún timbre, la reserva se aplicará al siguiente timbre que se cree en la región.

Si los timbres no se ejecutan durante una hora completa, la reserva se aplica automáticamente a otros timbres coincidentes de la misma región durante la misma hora.

### <a name="choose-a-stamp-type---windows-or-linux"></a>Selección del tipo de timbre: Windows o Linux

Un timbre vacío en entorno aislado emite el medidor de timbres de Windows de forma predeterminada, por ejemplo, cuando no hay ningún trabajo implementado. Cuando se implementan trabajos de Windows, sigue emitiendo el medidor. Si implementa un trabajo de Linux, el medidor cambia al medidor de timbres de Linux. El timbre emite el medidor de Windows cuando se implementan trabajos de Windows y de Linux.

Como resultado, el medidor de timbres puede cambiar entre Windows y Linux durante la vigencia del timbre. Mientras tanto, las reservas son específicas del sistema operativo. Tendrá que comprar una reserva que admita los trabajos que planea implementar en el timbre. Los timbres mixtos y solo de Windows usan la reserva de Windows. Los timbres que solo tienen trabajos de Linux usan la reserva de Linux.

El único caso en el que debe comprar una reserva de Linux es cuando planee tener _solo_ trabajos de Linux en el timbre.

### <a name="discount-examples"></a>Ejemplos de descuento

En los ejemplos siguientes se muestra cómo se aplica el descuento sobre la instancia reservada en el impuesto sobre el timbre en entorno aislado en función de las implementaciones.

- **Ejemplo 1**: compra una instancia de capacidad de timbre reservado en entorno aislado en una región sin timbres de App Service en entorno aislado. Implementa un nuevo timbre en la región y pagan las tarifas reservadas para dicho timbre.
- **Ejemplo 2**: compra una instancia de capacidad de timbre reservado en entorno aislado en una región que ya tiene implementado un timbre de App Service en entorno aislado. Empieza a recibir la tarifa reservada para el timbre implementado.
- **Ejemplo 3**: compra una instancia de capacidad de timbre reservado en entorno aislado en una región que ya tiene implementado un timbre de App Service en entorno aislado. Empieza a recibir la tarifa reservada para el timbre implementado. Después, elimina el timbre e implementa uno nuevo. Recibe la tarifa reservada para el nuevo timbre. Los descuentos no se transfieren a duraciones sin sellos implementados.
- **Ejemplo 4**: compra una instancia de capacidad de timbre reservado de Linux en entorno aislado en una región y, luego, implementa un nuevo timbre en la región. Cuando el timbre se implementa inicialmente sin trabajos, emite el medidor de timbres de Windows. No se recibe ningún descuento. Cuando el primer trabajo de Linux se implementa en el timbre, emite el medidor de timbres de Linux y se aplica el descuento de reserva. Si un trabajo de Windows se implementa posteriormente en el timbre, el medidor de timbres vuelve a Windows. Ya no recibirá un descuento por la reserva de timbre reservado de Linux en entorno aislado.

## <a name="next-steps"></a>Pasos siguientes

- Para más información sobre cómo administrar una reserva, consulte [Administración de Azure Reservations](manage-reserved-vm-instance.md).
- Para obtener más información sobre la compra anticipada de la capacidad reservada de instancias de Premium v3 y timbres en entorno aislado de App Service para ahorrar dinero, consulte [Pago por adelantado de Azure App Service con capacidad reservada](prepay-app-service.md).
- Para obtener más información acerca de Azure Reservations, consulte los siguientes artículos:
  - [¿Qué es Azure Reservations?](save-compute-costs-reservations.md)
  - [Administración de reservas en Azure](manage-reserved-vm-instance.md)
  - [Información sobre el uso de reservas de Azure para suscripciones de pago por uso](understand-reserved-instance-usage.md)
  - [Información sobre el uso de reservas para la inscripción Enterprise](understand-reserved-instance-usage-ea.md)
