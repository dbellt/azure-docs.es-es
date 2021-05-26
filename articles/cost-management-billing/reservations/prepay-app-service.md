---
title: Ahorros para Azure App Service con capacidad reservada
description: Aprenda a ahorrar costos en instancias reservadas de Azure App Service Premium v3 y Premium v2 y tarifas de unidades de escalado aisladas.
author: yashesvi
ms.reviewer: yashar
ms.service: cost-management-billing
ms.subservice: reservations
ms.topic: how-to
ms.date: 05/13/2021
ms.author: banders
ms.custom: references_regions
ms.openlocfilehash: f80619f1eeb7cdd970e4e3ee64491453c790149d
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110378533"
---
# <a name="save-costs-with-azure-app-service-reserved-instances"></a>Ahorro de costos con instancias reservadas de Azure App Service

En este artículo se explica cómo ahorrar con instancias reservadas de Azure App Service para instancias de Premium v3 y Premium v2 y tarifas de unidades de escalado aisladas.

## <a name="save-with-premium-v3-reserved-instances"></a>Ahorro con instancias reservadas de Premium v3

Al confirmar una instancia reservada de Premium v3 de Azure App Service, puede ahorrar dinero. El descuento de la reserva se aplica automáticamente el número de instancias en ejecución que coincidan con el ámbito y los atributos de la reserva. No es necesario asignar una reserva a una instancia para obtener los descuentos.

## <a name="determine-the-right-reserved-instance-size-before-you-buy"></a>Determinación del tamaño adecuado de una instancia reservada antes de su adquisición

Antes de adquirir una reserva, debe determinar el tamaño de la instancia reservada de Premium v3 que necesita. Las siguientes secciones le ayudarán a determinar el tamaño correcto de la instancia reservada de Premium v3.

### <a name="use-reservation-recommendations"></a>Usar recomendaciones de reserva

Puede consultar las recomendaciones de reserva para averiguar las reservas que debe adquirir.

- Se mostrarán recomendaciones de compra y la cantidad recomendada al adquirir una instancia reservada de Premium v3 en Azure Portal.
- Azure Advisor proporciona recomendaciones de compra de suscripciones individuales.
- Puede usar las API para obtener recomendaciones de compra relativas tanto a los ámbitos de suscripción tanto compartida como única. Para más información, vea [Recommendations API de compra de instancia reservada para clientes empresariales](/rest/api/billing/enterprise/billing-enterprise-api-reserved-instance-recommendation).
- En el caso de los clientes de Contrato Enterprise (EA) y del Contrato de cliente de Microsoft (MCA), las recomendaciones de compra de los ámbitos de suscripción tanto única como compartida solo están disponibles con el [paquete de contenido de Power BI de Azure Consumption Insights](/power-bi/service-connect-to-azure-consumption-insights).

### <a name="analyze-your-usage-information"></a>Analizar la información de uso

Analice su información de uso para averiguar qué reservas debe adquirir. Los datos de uso están disponibles en el archivo de uso y en las API. Úselos de manera conjunta para determinar qué reserva adquirir. Para determinar la cantidad de reservas que necesita adquirir, compruebe si hay instancias de Premium v3 que tengan un uso elevado diario.

El archivo de uso proporciona los cargos por período de facturación y el uso diario. Para más información sobre cómo descargar el archivo de uso, vea [Visualización y descarga de los datos de uso y los cargos de Azure](../understand/download-azure-daily-usage.md). Tras ello, una vez posea la información del archivo de uso, podrá [determinar qué reserva comprar](determine-reservation-purchase.md).

## <a name="buy-a-premium-v3-reserved-instance"></a>Adquisición de una instancia reservada de Premium v3

Puede comprar una instancia reservada de Premium v3 en [Azure Portal](https://portal.azure.com/#blade/Microsoft_Azure_Reservations/CreateBlade/referrer/documentation/filters/%7B%22reservedResourceType%22%3A%22VirtualMachines%22%7D). Pague la reserva [por adelantado o mensualmente](prepare-buy-reservation.md). Estos requisitos se aplican a la compra de una instancia reservada de Premium v3:

- Debe tener un rol de propietario en al menos una suscripción de EA o en una suscripción con una tarifa de pago por uso.
- En el caso de las suscripciones de EA, la opción **Agregar instancias reservadas** debe estar habilitada en el [portal de EA](https://ea.azure.com/). O bien, si esa opción está deshabilitada, debe ser un administrador de EA en la suscripción.
- En el caso del programa Proveedor de soluciones en la nube (CSP), solo los agentes de administración o de ventas pueden adquirir reservas.

Para comprar una instancia:

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).
2. Seleccione **Todos los servicios** > **Reservations**.
3. Seleccione **Agregar** para comprar una nueva reserva y, luego, haga clic en **Instancia**.
4. Rellene los campos obligatorios. Las instancias reservadas de Premium v3 en ejecución que coinciden con los atributos seleccionados cumplen los requisitos para obtener el descuento de reserva. El número real de instancias reservadas de Premium v3 que obtienen el descuento depende del ámbito y la cantidad seleccionados.

Si tiene un Contrato Enterprise, puede usar la opción **Agregar más** para agregar rápidamente instancias adicionales. La opción no está disponible para otros tipos de suscripciones.

| **Campo** | **Descripción** |
|---|---|
| Subscription | Suscripción que se usa para pagar la reserva. Los costos de la reserva se cobran en el método de pago de la suscripción. El tipo de suscripción debe ser Contrato Enterprise (números de oferta: MS-AZR-0017P o MS-AZR-0148P) o Contrato de cliente de Microsoft o una suscripción individual con tarifas de pago por uso (números de oferta: MS-AZR-0003P o MS-AZR-0023P). Los cargos se deducen del saldo de compromiso monetario, si está disponible, o se cobran como uso por encima del límite. En una suscripción con tarifas de pago por uso, los cargos se cobran con el método de pago de factura o la tarjeta de crédito de la suscripción. |
| Ámbito | El ámbito de la reserva puede cubrir una o varias suscripciones (ámbito compartido). Si selecciona: <ul><li>**Single resource group scope** (Ámbito de grupo de recursos único): aplica el descuento por reserva a los recursos coincidentes solo en el grupo de recursos seleccionado. </li><li>**Single subscription scope** (Ámbito de suscripción única): aplica el descuento por reserva a los recursos coincidentes de la suscripción seleccionada.</li><li>**Ámbito compartido**: aplica el descuento por reserva a los recursos coincidentes en suscripciones aptas que están en el contexto de facturación. Para los clientes de EA, el contexto de facturación es la inscripción. En el caso de suscripciones individuales con tarifas de pago por uso, el ámbito de facturación son todas las suscripciones aptas creadas por el administrador de la cuenta.</li></ul> |
| Region | Región de Azure que está cubierta por la reserva. |
| Tamaño de instancias reservadas de Premium v3 | El tamaño de las instancias reservadas de Premium v3. |
| Término | Un año o tres años. También hay un período de 5 años disponible solo para las instancias reservadas de Premium v3 HBv2. |
| Cantidad | Número de instancias que se compran dentro de la reserva. La cantidad es el número de instancias reservadas de Premium v3 en ejecución a las que se aplica el descuento de facturación. Por ejemplo, si tiene 10 instancias reservadas de Premium v3 Standard\_D2 en la región Este de EE UU., debería especificar 10 como cantidad para maximizar la ventaja para todas las instancias reservadas de Premium v3 en ejecución. |

## <a name="buy-an-isolated-v2-reserved-instance"></a>Compra de una instancia reservada de App Service aislado v2

Puede comprar una instancia reservada de App Service aislado v2 en [Azure Portal](https://portal.azure.com/#blade/Microsoft_Azure_Reservations/CreateBlade/referrer/documentation/filters/%7B%22reservedResourceType%22%3A%22VirtualMachines%22%7D). Pague la reserva [por adelantado o mensualmente](prepare-buy-reservation.md). Estos requisitos se aplican a la compra de una instancia reservada de App Service aislado v2:

- Debe tener un rol de propietario en al menos una suscripción de EA o en una suscripción con una tarifa de pago por uso.
- En el caso de las suscripciones de EA, la opción **Agregar instancias reservadas** debe estar habilitada en el [portal de EA](https://ea.azure.com/). O bien, si esa opción está deshabilitada, debe ser un administrador de EA en la suscripción.
- En el caso del programa Proveedor de soluciones en la nube (CSP), solo los agentes de administración o de ventas pueden adquirir reservas.

Para comprar una instancia:

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).
2. Seleccione **Todos los servicios** > **Reservations**.
3. Seleccione **Agregar** para comprar una nueva reserva y, luego, haga clic en **Instancia**.
4. Rellene los campos obligatorios. Las instancias reservadas de App Service aislado v2 en ejecución que coinciden con los atributos seleccionados cumplen los requisitos para obtener el descuento de reserva. El número real de instancias reservadas de App Service aislado v2 que obtienen el descuento depende del ámbito y la cantidad seleccionados.

Si tiene un Contrato Enterprise, puede usar la opción **Agregar más** para agregar rápidamente instancias adicionales. La opción no está disponible para otros tipos de suscripciones.

| **Campo** | **Descripción** |
| --- | --- |
| Subscription | Suscripción que se usa para pagar la reserva. Los costos de la reserva se cobran en el método de pago de la suscripción. El tipo de suscripción debe ser Contrato Enterprise (números de oferta: MS-AZR-0017P o MS-AZR-0148P) o Contrato de cliente de Microsoft o una suscripción individual con tarifas de pago por uso (números de oferta: MS-AZR-0003P o MS-AZR-0023P). Los cargos se deducen del saldo de compromiso monetario, si está disponible, o se cobran como uso por encima del límite. En una suscripción con tarifas de pago por uso, los cargos se cobran con el método de pago de factura o la tarjeta de crédito de la suscripción. |
| Ámbito | El ámbito de la reserva puede cubrir una o varias suscripciones (ámbito compartido). Si selecciona:<UL><LI>**Single resource group scope** (Ámbito de grupo de recursos único): aplica el descuento por reserva a los recursos coincidentes solo en el grupo de recursos seleccionado.</li><li>**Single subscription scope** (Ámbito de suscripción única): aplica el descuento por reserva a los recursos coincidentes de la suscripción seleccionada.</li><li>**Ámbito compartido**: aplica el descuento por reserva a los recursos coincidentes en suscripciones aptas que están en el contexto de facturación. Para los clientes de EA, el contexto de facturación es la inscripción. En el caso de suscripciones individuales con tarifas de pago por uso, el ámbito de facturación son todas las suscripciones aptas creadas por el administrador de la cuenta.</li> |
| Region | Región de Azure que está cubierta por la reserva. |
| Tamaño de instancia reservada de App Service aislado v2 | Tamaño de las instancias reservadas de App Service aislado v2. |
| Término | Un año o tres años. También hay un período de 5 años disponible solo para las instancias reservadas de App Service aislado v2 HBv2. |
| Cantidad | Número de instancias que se compran dentro de la reserva. La cantidad es el número de instancias reservadas de App Service aislado v2 en ejecución que pueden obtener el descuento de facturación. Por ejemplo, si tiene 10 instancias reservadas de App Service aislado v2 Standard\_D2 en la región Este de EE UU., especificaría 10 como cantidad para maximizar el beneficio para todas las instancias reservadas de App Service aislado v2 en ejecución. |

## <a name="save-with-isolated-stamp-fees"></a>Ahorro en el impuesto sobre el timbre en entorno aislado

Para ahorrar dinero en los impuestos sobre el timbre de Azure App Service aislado, puede confirmar una reserva para el uso que hará del timbre durante un plazo de tres años. Para comprar la capacidad reservada del impuesto sobre el timbre en entorno aislado, debe elegir la región de Azure en la que se implementará el timbre y el número de timbres que quiere comprar.

Cuando compra una reserva, el uso del impuesto sobre el timbre en entorno aislado que coincide con los atributos de reserva deja de pagarse según las tarifas de pago por uso. La reserva se aplica automáticamente al número de timbres en entornos aislados que coinciden con el ámbito y la región de la capacidad reservada. No es necesario que asigne una reserva a un timbre aislado. La reserva no se aplica a los trabajos, por lo que los demás recursos asociados con el timbre se cobran por separado.

Cuando expira la capacidad reservada, los timbres en entornos aislados siguen ejecutándose, pero se facturan a la tarifa de pago por uso. Las reservas no se renuevan automáticamente.

## <a name="determine-the-right-isolated-stamp-reservation-to-purchase"></a>Determinación de la reserva de compra adecuada de timbre en entorno aislado

Al comprar una reserva, confirma el uso que hará de las cantidades reservadas durante los tres años siguientes. Compruebe los datos de uso para determinar cuántos timbres de App Service aislado usa de forma coherente y es posible que use en un futuro.

Además, asegúrese de comprender cómo el timbre en entorno aislado emite el medidor de Linux o de Windows.

- De manera predeterminada, un timbre en entorno aislado vacío emite el medidor de timbres de Windows. Por ejemplo, sin trabajos implementados. Si los trabajos de Windows se implementan en el timbre, seguirá emitiendo este medidor.
- Si implementa un trabajo de Linux, el medidor cambia al medidor de timbres de Linux.
- El timbre emite el medidor de Windows en los casos en que se implementan trabajos de Windows y Linux.

Por este motivo, el medidor de timbres puede cambiar entre Windows y Linux durante la vigencia del timbre.

Compre reservas de timbres de Windows si tiene uno o más trabajos de Windows en el timbre. El único caso en el que debe comprar una reserva de timbres de Linux es si planea tener _solo_ trabajos de Linux en el timbre.

## <a name="buy-isolated-stamp-reserved-capacity"></a>Compra de la capacidad reservada de timbres en entornos aislados

Puede comprar la capacidad reservada de timbres en entornos aislados en [Azure Portal](https://portal.azure.com/#blade/Microsoft_Azure_Reservations/CreateBlade/referrer/documentation/filters/%7B%22reservedResourceType%22%3A%22AppService%22%7D). Pague la reserva [por adelantado o mensualmente](./prepare-buy-reservation.md). Para comprar la capacidad reservada, debe tener el rol de propietario al menos en una suscripción Enterprise o individual con precios de pago por uso.

- En el caso de las suscripciones Enterprise, la opción **Agregar instancias reservadas** debe estar habilitada en el [portal de EA](https://ea.azure.com/). O bien, si esa opción está deshabilitada, debe ser un administrador del Contrato Enterprise.
- En el caso del programa del Proveedor de soluciones en la nube (CSP), los únicos que pueden comprar la capacidad reservada de Azure Synapse Analytics son los agentes de administración o de ventas.

**Para comprar:**

1. Vaya a [Azure Portal](https://portal.azure.com/#blade/Microsoft_Azure_Reservations/CreateBlade/referrer/documentation/filters/%7B%22reservedResourceType%22%3A%22AppService%22%7D).
1. Seleccione una suscripción. Utilice la lista **Suscripción** para elegir la que se va a usar para pagar la capacidad reservada. Los costos de la capacidad reservada se cobran en el método de pago de la suscripción. El tipo de suscripción debe ser Contrato Enterprise (números de oferta: MS-AZR-0017P o MS-AZR-0148P) o Pago por uso (números de oferta: MS-AZR-0003P o MS-AZR-0023P) o una suscripción a CSP.
    - En el caso de una suscripción Enterprise, los cargos se deducirán del saldo de prepago de la inscripción de Azure (anteriormente llamado compromiso monetario) o se cobrarán como parte del uso por encima del límite.
    - Para la suscripción Pago por uso, los cargos se cobran en el método de pago de tarjeta de crédito o factura de la suscripción.
1. Utilice un **ámbito** para elegir el ámbito de la suscripción.
    - **Single resource group scope** (Ámbito de grupo de recursos único): aplica el descuento por reserva a los recursos coincidentes solo en el grupo de recursos seleccionado.
    - **Single subscription scope** (Ámbito de suscripción única): aplica el descuento por reserva a los recursos coincidentes de la suscripción seleccionada.
    - **Ámbito compartido**: aplica el descuento por reserva a los recursos coincidentes en suscripciones aptas que están en el contexto de facturación. Para los clientes con Contrato Enterprise, el contexto de facturación es la inscripción. En el caso de suscripciones individuales con tarifas de pago por uso, el ámbito de facturación son todas las suscripciones aptas creadas por el administrador de la cuenta.
1. Seleccione una **región** para elegir una región de Azure que abarque la capacidad reservada y agregue la reserva al carro.
1. Seleccione un tipo de plan aislado y, luego, haga clic en **Seleccionar**.  
    ![Ejemplo ](./media/prepay-app-service/app-service-isolated-stamp-select.png)
1. Escriba la cantidad de timbres de App Service aislado que quiera reservar. Por ejemplo, una cantidad de tres le proporcionaría tres timbres reservados por región. Haga clic en **Siguiente: Review + Buy** (Revisar y comprar).
1. Revise la información y haga clic en **Comprar ahora**.

Una vez termine la compra, vaya a [Reservas](https://portal.azure.com/#blade/Microsoft_Azure_Reservations/ReservationsBrowseBlade) para ver el estado de la compra y supervisarlo en cualquier momento.

## <a name="cancel-exchange-or-refund-reservations"></a>Cancelación, intercambio o reembolso de reservas

Puede cancelar, intercambiar o reembolsar reservas con ciertas limitaciones. Para más información, consulte [Autoservicio de intercambios y reembolsos de reservas de Azure](exchange-and-refund-azure-reservations.md).

## <a name="discount-application-shown-in-usage-data"></a>Cómo se muestra la aplicación del descuento en los datos de uso

Los datos de uso que obtienen un descuento de reserva tienen un precio efectivo de cero. Los datos de uso muestran el descuento de reserva de cada instancia de timbre en cada reserva.

Para obtener más información sobre cómo se muestra el descuento de reserva en los datos de uso, consulte [Obtención del uso y los costos de reservas de Contrato Enterprise](understand-reserved-instance-usage-ea.md) si es un cliente del Contrato Enterprise (EA). De lo contrario, consulte el artículo [Descripción del uso de reservas de Azure para su suscripción individual con precios de pago por uso](understand-reserved-instance-usage.md).

## <a name="next-steps"></a>Pasos siguientes

- Para obtener más información acerca de Azure Reservations, consulte los siguientes artículos:
  - [¿Qué es Azure Reservations?](save-compute-costs-reservations.md)
  - [Understand how an Azure App Service Isolated Stamp reservation discount is applied](reservation-discount-app-service.md) (Información de cómo se aplica un descuento en la reserva del timbre de Azure App Service aislado).
  - [Información sobre el uso de reservas para la inscripción Enterprise](understand-reserved-instance-usage-ea.md)
