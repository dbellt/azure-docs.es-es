---
title: Capacidades de transacción de marketplace comercial de Microsoft
description: En este artículo se describen las consideraciones de precios, facturación y pago de la opción de transacción de marketplace comercial.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: conceptual
ms.date: 04/06/2021
ms.author: mingshen
author: mingshen-ms
ms.openlocfilehash: 0a25e1b50455cad5bdbe5b76b2a291f2a1c11940
ms.sourcegitcommit: 5f482220a6d994c33c7920f4e4d67d2a450f7f08
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/08/2021
ms.locfileid: "107107013"
---
# <a name="commercial-marketplace-transact-capabilities"></a>Capacidades de transacción de marketplace comercial

En este artículo se describen los aspectos que se deben tener en cuenta en relación con los precios, la facturación y el pago de marketplace comercial de Microsoft.

## <a name="transactions-by-listing-option"></a>Opción de transacciones por publicación

El editor o Microsoft es responsable de administrar las transacciones de licencia de software para las ofertas en marketplace comercial. La opción de publicación que elija para su oferta determinará quién administra la transacción. Para obtener más información sobre la disponibilidad y explicaciones de cada opción de publicación, consulte [Introduction a las opciones de lista](determine-your-listing-type.md).

### <a name="contact-me-free-trial-and-byol-options"></a>Opciones Ponerse en contacto conmigo, Evaluación gratuita y BYOL

Los editores pueden elegir las opciones _Ponerse en contacto conmigo_ y _Evaluación gratuita_ con fines promocionales y de captación de usuarios. En el caso de algunos tipos de oferta, los publicadores pueden elegir la opción _Traiga su propia licencia_ (BYOL) para permitir que los clientes compren una suscripción a la oferta mediante una licencia que hayan adquirido directamente. Con estas opciones, Microsoft no participa directamente en las transacciones de licencias de software del publicador y no hay ninguna tarifa de transacción asociada, de modo que los publicadores conservan todos los ingresos.

Con estas opciones, los publicadores son responsables de ayudar en todos los aspectos de la transacción de la licencia de software. Por ejemplo, el pedido, la entrega, la cumplimentación, la medición del uso, la facturación, el pago y el cobro. Con la opción de publicación Ponerse en contacto conmigo, los editores se quedan con el 100 % de las cuotas de licencias de software que se cobran al cliente.

### <a name="transact-publishing-option"></a>Opción de publicación de transacción

La opción de vender a través de Microsoft permite aprovechar las ventajas de las funcionalidades de comercio de Microsoft y proporciona una experiencia completa que se inicia con la detección y la evaluación y finaliza con la compra y la implementación. Una oferta _comercializable_ es aquella en la que Microsoft facilita el intercambio de dinero por una licencia de software en nombre del publicador. Las ofertas de transacción se facturan a una suscripción de Microsoft existente o a una tarjeta de crédito, lo que permite a Microsoft hospedar las transacciones de marketplace en la nube en nombre del publicador.

Puede elegir la opción de transacción al crear una nueva oferta en el Centro de partners. Esta opción solo se mostrará si la transacción está disponible para el tipo de oferta.

## <a name="transact-overview"></a>Información general sobre transacciones

Al usar la opción de transacción, Microsoft permite la venta de software de terceros y la implementación de algunos tipos de oferta en la suscripción de Azure del cliente. Al seleccionar un modelo de precio para una oferta, el publicador debe tener en cuenta la facturación de los precios de la infraestructura y sus propias cuotas de licencia de software.

La opción de publicación de transacción se admite actualmente para los siguientes tipos de oferta:

| Tipo de oferta | Cadencia de facturación | Facturación de uso medido | Modelo de precios |
| ------------ | ------------- | ------------- | ------------- |
| Aplicación de Azure<br>(Aplicación administrada) | Mensual | Sí | Basado en uso |
| Máquina virtual de Azure | Mensualmente* | No | Basado en uso, BYOL |
| Software como servicio (SaaS) | Mensual y anual | Sí | Tarifa plana, por usuario, basada en uso |
|||||

`*` La máquina virtual de Azure ofrece compatibilidad con planes de facturación basados en el uso. Estos planes se facturan mensualmente por el uso por horas de la suscripción por núcleo, tamaño de núcleo, mercado y uso de tamaño de núcleo.

### <a name="metered-billing"></a>Facturación de uso medido

El _servicio de medición de Marketplace_ le permite especificar cargos de pago por uso (basados en el consumo) además de cargos mensuales o anuales incluidos en el contrato (derecho). Puede aplicar costos de uso por las dimensiones del servicio de medición de Marketplace que especifique, como el ancho de banda, los vales o los correos electrónicos procesados. Para más información sobre la facturación de uso medido para ofertas de SaaS, consulte [Facturación de uso medido para SaaS mediante el servicio de medición de Marketplace comercial](./partner-center-portal/saas-metered-billing.md). Para más información sobre la facturación de uso medido para ofertas de Aplicación de Azure, consulte [Facturación según uso de aplicaciones administradas](./partner-center-portal/azure-app-metered-billing.md).

### <a name="billing-infrastructure-costs"></a>Costos de infraestructura de facturación

Para las **máquinas virtuales** y **aplicaciones de Azure**, los precios de uso de la infraestructura de Azure se facturan a la suscripción a Azure del cliente. Los precios de uso de la infraestructura se cotizan y presentan por separado de los precios de licencias del proveedor de software en la factura del cliente.

Para **aplicaciones SaaS**, el publicador debe tener en cuenta los precios de uso de la infraestructura de Azure y los precios de licencias de software como elemento único de costo. Se representa como un honorario plano para el cliente. La administración y el uso de la infraestructura de Azure se le factura directamente al editor. El cliente no ve los honorarios de uso de infraestructura reales. En general, los publicadores optan por agrupar los honorarios de uso de la infraestructura de Azure en los precios de las licencias de software. Las cuotas de las licencias de software no se basan en el uso medido ni en el consumo del usuario.

## <a name="pricing-models"></a>Modelos de precios

En función de la opción de transacción usada, los cargos de suscripción son los siguientes:

- **Obténgalo ahora (de forma gratuita)** : las licencias de software son gratuitas. Las ofertas gratuitas no se pueden pasar a ofertas de pago. Los clientes deben solicitar una oferta de pago.
- **Traiga su propia licencia** (BYOL): si se incluye una oferta en el Marketplace comercial, los cargos aplicables por las licencias de software se administran directamente entre el publicador y el cliente. Microsoft solo carga las tarifas de uso de la infraestructura de Azure aplicables a la cuenta de suscripción de Azure del cliente.
- **Precios de suscripción**: las cuotas de licencia de software se presentan como cuotas de suscripción periódicas mensuales o anuales que se facturan como tarifa plana o por puesto. Las cuotas de suscripción periódicas no se prorratean por cancelaciones del cliente a mitad del período o por la falta de uso de los servicios. Las cuotas de suscripción periódicas se pueden prorratear si el cliente cambia a una versión superior o inferior de su suscripción a mitad del período.
- **Precios basados en el uso**: en el caso de ofertas de máquinas virtuales de Azure, se factura a los clientes en función del alcance de su uso de la oferta. Si son imágenes de máquina virtual, a los clientes se les cobra una tarifa por hora de Azure Marketplace, establecida por el publicador, por el uso de las máquinas virtuales implementadas a partir de las imágenes. La tarifa por horas puede ser uniforme o variar según los distintos tamaños de máquina virtual. Las horas parciales se cobran por minuto. Los planes se facturan mensualmente.
- **Precios de uso medido**: con las ofertas de aplicaciones de Azure y SaaS, los publicadores pueden usar el [servicio de medición de Marketplace](./partner-center-portal/marketplace-metering-service-apis.md) para facturar el consumo según las dimensiones personalizadas de los medidores que configuren. Estos cambios se suman a los cargos mensuales o anuales incluidos en el contrato (derecho). Algunos ejemplos de dimensiones de medidores personalizadas son el ancho de banda, los vales o los correos electrónicos procesados. Los publicadores pueden definir una o más dimensiones de uso medido para cada plan, con un máximo de 30 por oferta. Los publicadores son responsables de hacer un seguimiento del uso individual de los clientes, con cada medidor definido en la oferta. Los eventos se deben notificar a Microsoft en el plazo de una hora. Microsoft cobra a los clientes en función de la información de uso notificada por los anunciantes para el período de facturación aplicable.
- **Evaluación gratuita**: no se cobra por las licencias de software que cubren un período de 30 días a seis meses, en función del tipo de oferta. Si los publicadores proporcionan una evaluación gratuita en varios planes dentro de la misma oferta, los clientes pueden cambiar a una versión de evaluación gratuita de otro plan, pero el período de prueba no se reinicia. En el caso de ofertas de máquina virtual, se cobra a los clientes los costos de la infraestructura de Azure por el uso de la oferta durante un período de prueba. Tras la expiración del período de prueba, se cobra automáticamente a los clientes el último plan que probaron según las tarifas estándar, a menos que lo cancelen antes del final del período de prueba.

> [!NOTE]
> Las ofertas que se facturan según el uso nos on aptas para reembolsos una vez que se ha usado la solución.

Los editores que quieran cambiar las tarifas de uso asociadas a una oferta deben quitar primero la oferta (o el plan específico dentro de esta) del marketplace comercial. La eliminación debe realizarse de acuerdo con los requisitos del [contrato para editores de Microsoft](https://go.microsoft.com/fwlink/?LinkID=699560). Luego, el editor puede publicar una nueva oferta (o plan dentro de una oferta) que incluya las nuevas cuotas de uso. Para información sobre cómo quitar una oferta o plan, consulte [Detención de la venta de una oferta o un plan](./partner-center-portal/update-existing-offer.md#stop-selling-an-offer-or-plan).

### <a name="free-contact-me-and-bring-your-own-license-byol-pricing"></a>Precios de Gratis, Ponerse en contacto conmigo y traiga su propia licencia (BYOL)

Al publicar una oferta con la opción Obténgalo ahora (de forma gratuita), Ponerse en contacto conmigo o BYOL, Microsoft no desempeña ningún papel en la transacción de ventas con las tarifas de licencia de software. El publicador conserva el 100 % de las cuotas de licencia de software.

### <a name="usage-based-and-subscription-pricing"></a>Precios de suscripción y basados en el uso

Al publicar una oferta basada en el uso o una transacción de una suscripción, Microsoft proporciona la tecnología y los servicios para procesar las compras, las devoluciones y los contracargos de las licencias de software. En este escenario, el publicador autoriza a Microsoft a actuar como agente para estos fines. El publicador permite a Microsoft facilitar la transacción de licencias de software. El publicador mantiene su designación como vendedor, proveedor, distribuidor y licenciatario.

Microsoft permite a los clientes pedir, licenciar y usar el su software, sujeto a los términos y condiciones de marketplace comercial de Microsoft y de su contrato de licencia para el usuario final. Debe proporcionar su propio contrato de licencia para el usuario final o seleccionar el [Contrato estándar](./standard-contract.md) al crear la oferta.

### <a name="free-software-trials"></a>Evaluaciones gratuitas del software

En escenarios de publicación con transacciones, puede hacer que una licencia de software esté disponible de forma gratuita entre 30 y 120 días, en función de la suscripción. Se cobrará a los clientes por el uso de la infraestructura de Azure aplicable.

### <a name="examples-of-pricing-and-store-fees"></a>Ejemplos de precios y tarifas de la tienda

**Basado en uso**

Los precios basados en el uso tienen la siguiente estructura de costos:

| **Costo de su licencia** | **1,00 USD por hora** |
|---------|---------|
| Costo de uso de Azure (D1/1 núcleo) | 0,14 USD por hora |
| _Microsoft factura al cliente_ | _1,14 USD por hora_ |
||

En este escenario, Microsoft factura 1,14 USD por hora de uso de la imagen de máquina virtual publicada.

| **Microsoft factura** | **1,14 USD por hora**  |
|---------|---------|
| Microsoft le paga el 80 % del costo de licencia | 0,80 USD por hora |
| Microsoft se queda con el 20 % del costo de licencia  |  0,20 USD por hora |
| Microsoft se queda con el 100 % costo de uso de Azure | 0,14 USD por hora |
||

**Traiga su propia licencia (BYOL)**

BYOL tiene la siguiente estructura de costos:

| **Costo de su licencia** | **Usted negocia y factura el precio de licencia** |
|---------|---------|
|Costo de uso de Azure (D1/1 núcleo)    |   0,14 USD por hora     |
| _Microsoft factura al cliente_ | _0,14 USD por hora_ |
||

En este escenario, Microsoft factura 0,14 USD por hora de uso de la imagen de máquina virtual publicada.

| **Microsoft factura** | **0,14 USD por hora** |
|---------|---------|
| Microsoft se queda con el costo de uso de Azure | 0,14 USD por hora |
| Microsoft se queda con 0 % del costo de licencia | 0,00 USD por hora |
||

**Suscripción de aplicación SaaS**

Las suscripciones de SaaS pueden tener un precio de tarifa plana o cobrarse por usuario, de forma mensual o anual. Si habilita la opción de **venta mediante Microsoft** de una oferta de SaaS, obtendrá la siguiente estructura de costos:

| **Costo de su licencia** | **100,00 USD al mes** |
|--------------|---------|
| Costo de uso de Azure (D1/1 núcleo) | Facturado directamente al publicador, no al cliente |
| _Microsoft factura al cliente_ | _100,00 USD al mes (el anunciante debe contar con los costos de paso a través o en que se incurran en el precio de la licencia)_ |
||

En este escenario, Microsoft factura 100,00 USD por su licencia de software y le paga 80,00 USD o 90,00 USD en función de si la oferta es apta para una tarifa de servicio de tienda reducida.

| **Microsoft factura** | **100,00 USD al mes** |
|---------|---------|
| Microsoft le paga el 80 % del costo de licencia <br> \* Microsoft le paga el 90 % del costo de licencia de las aplicaciones SaaS aptas. | 80,00 USD al mes <br> \* 90,00 USD al mes |
| Microsoft se queda con el 20 % del costo de licencia <br> \* Microsoft se queda con el 10 % del costo de licencia de las aplicaciones SaaS aptas. | 20,00 USD al mes <br> \* 10,00 USD |

### <a name="commercial-marketplace-service-fees"></a>Tarifas de servicio de Marketplace comercial

Se cobra una cuota de servicio de almacenamiento estándar del 20 % cuando los clientes compran su oferta de transacción en Marketplace comercial. Para obtener más información sobre esta cuota, consulte la sección 5c del [Contrato de Microsoft Publisher](https://go.microsoft.com/fwlink/?LinkID=699560).

En el caso de ciertas ofertas de transacciones que publique en Marketplace comercial, puede optar por una tarifa de servicio reducida en tienda del 10 %. Para que una oferta sea apta, Microsoft debe designarla como _incentivo de venta conjunta de Azure IP_. Para recibir el precio reducido del servicio Marketplace, se deben cumplir los criterios de idoneidad al menos cinco días hábiles antes del final de cada mes natural. Una vez que se cumplan los criterios de idoneidad, se concederá el precio reducido del servicio a todas las transacciones vigentes el primer día del mes siguiente hasta que se pierda el estado _Venta conjunta con incentivos de IP de Azure_. Para obtener más información sobre la elegibilidad de la venta conjunta de IP, vea [Requisitos para el estado de la venta conjunta](/legal/marketplace/certification-policies#3000-requirements-for-co-sell-status).

La cuota reducida del servicio Marketplace se aplica a SaaS, VM y aplicaciones administradas con incentivos de venta conjunta de Azure IP, así como a cualquier otra solución de IaaS en la que se pueda llevar a cabo una transacción cualificada que esté disponible a través de Marketplace comercial. Las ofertas de SaaS de pago asociadas con una aplicación de Microsoft Teams o al menos con dos complementos de Microsoft 365 (Excel, PowerPoint, Word, Outlook y SharePoint) y publicadas en Microsoft AppSource también son aptas para este descuento.

### <a name="customer-invoicing-payment-billing-and-collections"></a>Facturación, pago y cobro al cliente

**Facturación y pago**: el publicador puede usar el método de facturación preferido del cliente para comunicar los precios de las licencias del software de suscripción o de [pago por uso](https://azure.microsoft.com/pricing/purchase-options/pay-as-you-go/).

**Contrato Enterprise**: si el método de facturación preferido del cliente es el contrato Enterprise de Microsoft, las tarifas de las licencias de software se facturarán mediante este método de facturación como un costo detallado, independiente de los costos de uso específicos de Azure.

**Tarjetas de crédito y factura mensual**: los clientes pueden pagar con tarjeta de crédito y una factura mensual. En este caso, los precios de las licencias de software se facturarán igual que en el escenario de contrato Enterprise, como un costo detallado, independiente de los costos de uso específicos de Azure.

**Créditos gratuitos y compromiso monetario**: algunos clientes eligen pagar Azure por adelantado con un compromiso monetario en el contrato Enterprise, o han recibido créditos gratuitos para su uso con Azure. Aunque estos créditos se pueden usar para pagar por el uso de Azure, no se puede usar para pagar los honorarios de las licencias de software del publicador.

**Facturación y cobro**: la facturación de las licencias de software del publicador se presenta mediante el método de facturación seleccionado por el cliente y sigue la escala de tiempo de facturación. A los clientes sin un contrato Enterprise vigente se les facturan mensualmente las licencias de software de Marketplace. A los clientes con un contrato Enterprise se les factura mensualmente a través de una factura que se presenta cada trimestre.

Cuando se han seleccionado los modelos de precios de suscripción o de pago por uso (también llamados basados en uso), Microsoft actúa como agente del publicador y es responsable de todos los aspectos de facturación, pago y cobro.

### <a name="publisher-payout-and-reporting"></a>Pagos e informes al publicador

Los honorarios de las licencias de software cobrados por Microsoft como agente están sujetos a una tarifa de transacción del 20 %, a menos que se especifique lo contrario, y se deducen en el momento del pago al publicador.

Normalmente, los clientes compran con contrato Enterprise o mediante contrato de pago por uso con tarjeta de crédito. El tipo de contrato determina los plazos de facturación, cobro y pago.

>[!NOTE]
>Todos los informes y la información para la opción de publicación de la transacción están disponibles a través de la sección Análisis del Centro de partners.

#### <a name="billing-questions-and-support"></a>Soporte técnico y preguntas sobre facturación

Para más información y conocer las directivas legales, consulte el [contrato para editores de Microsoft](https://go.microsoft.com/fwlink/?LinkID=699560). Para obtener ayuda si tiene preguntas sobre facturación, póngase en contacto con el [equipo de soporte técnico para publicadores de Marketplace comercial](https://aka.ms/marketplacepublishersupport).

## <a name="transact-requirements"></a>Requisitos de las transacciones

En esta sección se tratan los requisitos de transacción para distintos tipos de oferta.

### <a name="requirements-for-all-offer-types"></a>Requisitos para todos los tipos de ofertas

- Se necesitan una cuenta de Microsoft y la información financiera para la opción de publicación de transacciones, independientemente del modelo de precios de la oferta.
- Entre la información financiera obligatoria se incluyen el perfil fiscal y la cuenta de pago.

Para más información sobre cómo configurar estas cuentas, consulte [Administración de la cuenta de marketplace comercial en el Centro de partners](manage-account.md).

### <a name="requirements-for-specific-offer-types"></a>Requisitos para tipos de ofertas específicos

La posibilidad de realizar la transacción a través de Microsoft solo está disponible para los siguientes tipos de ofertas de Marketplace comercial. En esta lista se proporcionan los requisitos para que estos tipos de ofertas sean procesables en Marketplace comercial.

- **Aplicación de Azure (plantilla de soluciones y planes de aplicaciones administradas)** : en algunos casos, el precio por uso de la infraestructura de Azure se traslada al cliente por separado de los precios de las licencias de software, pero en el mismo extracto de facturación. En cambio, si configura un plan de aplicación administrada para los cargos por infraestructura de ISV, los recursos de Azure se facturan al editor y el cliente recibe una tarifa plana que incluye el costo de la infraestructura, las licencias de software y los servicios de administración.

- **Máquina virtual de Azure**: seleccione entre los modelos de precios gratuito, BYOL o basado en uso. En la factura de Azure para el cliente, Microsoft presenta los precios de las licencias de software del publicador por separado de los precios de la infraestructura de Azure subyacente. Los precios de la infraestructura de Azure corresponden al uso del software del publicador.

- **Aplicación SaaS**: debe ser una solución para varios inquilinos, utilice [Azure Active Directory](https://azure.microsoft.com/services/active-directory/) para la autenticación e intégrela con las [API de cumplimiento de SaaS](partner-center-portal/pc-saas-fulfillment-api-v2.md). Se administra y se factura directamente al publicador por el uso de la infraestructura de Azure, por lo que se deben tener en cuenta las tarifas de uso de la infraestructura de Azure y las tarifas de las licencias de software como un único elemento de costo. Para ver instrucciones detalladas, consulte [Planeamiento de una oferta de SaaS en el marketplace comercial](plan-saas-offer.md#plans).

## <a name="private-plans"></a>Planes privados

Puede crear un plan privado para una oferta, con precios negociados específicos para el negocio, o bien con configuraciones personalizadas.

Los planes privados le permiten ofrecer un precio mayor o menor a determinados clientes con respecto a la oferta disponible públicamente. Los planes privados pueden usarse para descontar o agregar una prima a una oferta. Los planes privados pueden estar disponibles para uno o más clientes mediante la inclusión en una lista de su suscripción de Azure en el nivel de plan.

## <a name="next-steps"></a>Pasos siguientes

- Revise los patrones de publicación por tienda en línea para ver ejemplos de cómo la solución se asigna a un tipo de oferta y configuración.
- [Guía de publicación por tipo de oferta](publisher-guide-by-offer-type.md).
