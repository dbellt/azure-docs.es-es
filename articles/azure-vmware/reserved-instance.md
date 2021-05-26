---
title: Instancias reservadas de Azure VMware Solution
description: Obtenga información sobre cómo comprar una instancia reservada para Azure VMware Solution. Las instancias reservadas solo cubren la parte del uso dedicada a los procesos y tienen costos de licencias de software.
ms.topic: how-to
ms.date: 05/13/2021
ms.openlocfilehash: 2ec43374561db9b0de39f4d1c571b85a75df13ba
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110063107"
---
# <a name="save-costs-with-azure-vmware-solution"></a>Ahorro de costos con Azure VMware Solution

Al comprometerse a una instancia reservada de [Azure VMware Solution](introduction.md), ahorra dinero.  El descuento de la reserva se aplica automáticamente a los hosts de Azure VMware Solution en ejecución que coinciden con el ámbito y los atributos de la reserva. La compra de una instancia reservada cubre solo la parte de proceso de su uso e incluye los costos por licencias de software. 

## <a name="purchase-restriction-considerations"></a>Consideraciones sobre restricciones de compra

Las instancias reservadas están disponibles con algunas excepciones:

-   **Nubes**: las reservas solo están disponibles en las regiones enumeradas en la página [Productos disponibles por región](https://azure.microsoft.com/global-infrastructure/services/?products=azure-vmware).

-   **Cuota insuficiente**: una reserva cuyo ámbito sea de una suscripción única o compartida debe tener cuota de hosts disponible en la suscripción para la nueva instancia reservada. Puede [crear una solicitud de aumento de la cuota](request-host-quota-azure-vmware-solution.md) para resolver este problema.

-   **Elegibilidad de la oferta**: necesitará un [Contrato Enterprise (EA) de Azure](../cost-management-billing/manage/ea-portal-agreements.md) con Microsoft.

-   **Restricciones de capacidad**: en algunas circunstancias poco frecuentes, Azure limita la compra de nuevas reservas para las SKU de host de Azure VMware Solution debido a que la capacidad en una región es baja.

## <a name="buy-a-reservation"></a>Adquisición de una reserva

Puede comprar una instancia reservada de una instancia de host de Azure VMware Solution en [Azure Portal](https://portal.azure.com/#blade/Microsoft_Azure_Reservations/CreateBlade/referrer/documentation/filters/%7B%22reservedResourceType%22%3A%22VirtualMachines%22%7D).

Puede pagar la reserva [por adelantado o de manera mensual](../cost-management-billing/reservations/prepare-buy-reservation.md).

Estos requisitos se aplican a la compra de una instancia reservada de host dedicado:

-   Debe tener un rol de *Propietario* en al menos una suscripción de EA o en una suscripción con una tarifa de pago por uso.

-   En el caso de las suscripciones de EA, debe habilitar la opción **Agregar instancias reservadas** en [EA Portal](https://ea.azure.com/). Si está deshabilitada, debe ser un administrador de EA para la suscripción para habilitarla.

-   En el caso de una suscripción en un plan de Azure del Proveedor de soluciones en la nube (CSP), el partner debe adquirir las instancias reservadas en Azure Portal del cliente. 

### <a name="buy-reserved-instances-for-an-ea-subscription"></a>Compra de instancias reservadas para una suscripción de EA

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).

2. Seleccione **Todos los servicios** > **Reservations**.

3. Seleccione **Comprar ahora** y, a continuación, seleccione **Azure VMware Solution**.

4. Escriba los campos obligatorios. Los atributos seleccionados que coinciden con los hosts de Azure VMware Solution cumplen con los requisitos para obtener el descuento de la reserva.  Los atributos incluyen la SKU, las regiones (si procede) y el ámbito. El ámbito de la reserva determina dónde se aplica el ahorro de la reserva.

   Si tiene un contrato de EA, puede usar la opción **Agregar más** para agregar rápidamente instancias. La opción no está disponible para otros tipos de suscripciones.

   | Campo        |  Descripción |
   | ------------ | ------------ |
   | Subscription | Suscripción que se usa para pagar la reserva. Los costos de la reserva se cobran en el método de pago de la suscripción. El tipo de suscripción debe ser Contrato Enterprise (números de oferta: MS-AZR-0017P o MS-AZR-0148P) o Contrato de cliente de Microsoft o una suscripción individual con tarifas de pago por uso (números de oferta: MS-AZR-0003P o MS-AZR-0023P). Los cargos se deducirán del saldo de prepago de Azure (anteriormente llamado compromiso monetario) si hay fondos disponibles o se cobrarán como parte del uso por encima del límite. En una suscripción con tarifas de pago por uso, los cargos se facturan a la tarjeta de crédito de la suscripción o un método de pago de factura. |
   | Ámbito        | El ámbito de la reserva puede cubrir una o varias suscripciones (ámbito compartido). Si selecciona:<br><ul><li>El ámbito <b>Grupo de recursos único</b>: el descuento por reserva se aplica a los recursos coincidentes solo del grupo de recursos seleccionado.</li><li>El ámbito <b>Suscripción única</b>: el descuento por reserva se aplica a los recursos coincidentes de la suscripción seleccionada.</li><li>El ámbito <b>Compartido</b>: el descuento por reserva se aplica a los recursos coincidentes en suscripciones válidas que están en el contexto de facturación. Para los clientes de EA, el contexto de facturación es la inscripción. En el caso de suscripciones individuales con tarifas de pago por uso, el ámbito de facturación son todas las suscripciones aptas creadas por el administrador de la cuenta.</li></ul>       |
   | Region       | Región de Azure que está cubierta por la reserva.   |
   | Tamaño de host    | AV36    |
   | Término         | Un año o tres años.  |
   | Cantidad     | Número de instancias que se compran dentro de la reserva. La cantidad es el número de hosts de Azure VMware Solution en ejecución a los que se aplica el descuento de facturación.    |

### <a name="buy-reserved-instances-for-a-csp-subscription"></a>Compra de instancias reservadas para una suscripción de CSP

Los CSP que desean comprar instancias reservadas para sus clientes deben usar el procedimiento **Administración en nombre de** (AOBO) de la [documentación del Centro de partners](/partner-center/azure-plan-manage). Para obtener más información, vea el vídeo [Administración en nombre de (AOBO)](https://channel9.msdn.com/Series/cspdev/Module-11-Admin-On-Behalf-Of-AOBO).

1. Inicie sesión en el [Centro de partners](https://partner.microsoft.com).

2. Seleccione **CSP** para acceder al área **Clientes**.

3. Expanda los detalles del cliente y seleccione **Portal de administración de Microsoft Azure**. 

   :::image type="content" source="media/reserved-instances/csp-partner-center-aobo.png" alt-text="Área de clientes del Centro de partners de Microsoft" lightbox="media/reserved-instances/csp-partner-center-aobo.png":::

4. En Azure Portal, seleccione **Todos los servicios** > **Reservas**.

5. Seleccione **Comprar ahora** y, a continuación, seleccione **Azure VMware Solution**.

   :::image type="content" source="media/reserved-instances/csp-buy-reserved-instance-azure-portal.png" alt-text="Reservas de Microsoft Azure Portal" lightbox="media/reserved-instances/csp-buy-reserved-instance-azure-portal.png":::

6. Escriba los campos obligatorios. Los atributos seleccionados que coinciden con los hosts de Azure VMware Solution cumplen con los requisitos para obtener el descuento de la reserva.  Los atributos incluyen la SKU, las regiones (si procede) y el ámbito. El ámbito de la reserva determina dónde se aplica el ahorro de la reserva.

   | Campo        |  Descripción |
   | ------------ | ------------ |
   | Subscription | Suscripción que se usa para pagar la reserva. Los costos de la reserva se cobran en el método de pago de la suscripción. El tipo de suscripción debe ser válido, que en este caso es una suscripción de CSP.|
   | Ámbito        | El ámbito de la reserva puede cubrir una o varias suscripciones (ámbito compartido). Si selecciona:<br><ul><li>El ámbito <b>Grupo de recursos único</b>: el descuento por reserva se aplica a los recursos coincidentes solo del grupo de recursos seleccionado.</li><li>El ámbito <b>Suscripción única</b>: el descuento por reserva se aplica a los recursos coincidentes de la suscripción seleccionada.</li><li>El ámbito <b>Compartido</b>: el descuento por reserva se aplica a los recursos coincidentes en suscripciones válidas que están en el contexto de facturación. Para los clientes de EA, el contexto de facturación es la inscripción. En el caso de suscripciones individuales con tarifas de pago por uso, el ámbito de facturación son todas las suscripciones aptas creadas por el administrador de la cuenta.</li></ul>       |
   | Region       | Región de Azure que está cubierta por la reserva.   |
   | Tamaño de host    | AV36    |
   | Término         | Un año o tres años.  |
   | Cantidad     | Número de instancias que se compran dentro de la reserva. La cantidad es el número de hosts de Azure VMware Solution en ejecución a los que se aplica el descuento de facturación.     |

Para más información sobre cómo ver las reservas adquiridas para el cliente, consulte el artículo [Visualización de la reservas de Azure como proveedor de soluciones en la nube (CSP)](../cost-management-billing/reservations/how-to-view-csp-reservations.md).

## <a name="usage-data-and-reservation-usage"></a>Datos de uso y uso de la reserva

El uso que obtiene un descuento de reserva tienen un precio efectivo de cero. Puede ver qué instancia de Azure VMware Solution recibió el descuento de reserva para cada reserva.

Para más información sobre cómo aparecen los descuentos de reserva en los datos de uso:

- Para clientes de EA, consulte [Información sobre el uso de reservas de Azure para la inscripción Enterprise](../cost-management-billing/reservations/understand-reserved-instance-usage-ea.md).
- Para suscripciones individuales, consulte [Descripción del uso de reservas para su suscripción individual de Pago por uso](../cost-management-billing/reservations/understand-reserved-instance-usage.md).

## <a name="change-a-reservation-after-purchase"></a>Cambiar una reserva después de la compra

Puede realizar estos cambios en una reserva después de haberla comprado:

-   Actualización del ámbito de la reserva

-   Flexibilidad del tamaño de instancia (si procede)

-   Propiedad

También puede dividir una reserva en fragmentos más pequeños y combinar reservas. Ninguno de estos cambios conlleva una nueva transacción comercial ni un cambio en la fecha de finalización de la reserva.

Para más información sobre las reservas administradas por CSP, consulte [Venta de reservas de Microsoft Azure a clientes mediante el Centro de partners, Azure Portal o las API](/partner-center/azure-reservations).



>[!NOTE]
>Una vez que haya adquirido la reserva, no podrá realizar los estos tipos de cambios directamente:
>
> - La región de una reserva existente
> - SKU
> - Cantidad
> - Duration
>
>Pero sí se puede *intercambiar* una reserva si quiere realizar cambios.

## <a name="cancel-exchange-or-refund-reservations"></a>Cancelación, intercambio o reembolso de reservas

Puede cancelar, intercambiar o reembolsar reservas con ciertas limitaciones. Para más información, consulte [Autoservicio de intercambios y reembolsos de reservas de Azure](../cost-management-billing/reservations/exchange-and-refund-azure-reservations.md).

Los CSP pueden cancelar, intercambiar o reembolsar reservas, con algunas limitaciones, adquiridas por sus clientes. Para obtener más información, consulte [Administración, cancelación, intercambio o reembolso de reservas de Microsoft Azure para clientes](/partner-center/azure-reservations-manage).

## <a name="next-steps"></a>Pasos siguientes

Ahora que se han explicado las instancias reservadas de Azure VMware Solution, es posible que necesite más información sobre:

- [Creación de una evaluación de Azure VMware Solution](../migrate/how-to-create-azure-vmware-solution-assessment.md).
- [Administración de DHCP para Azure VMware Solution](configure-dhcp-azure-vmware-solution.md).
- [Supervisión y administración de máquinas virtuales de Azure VMware Solution](lifecycle-management-of-azure-vmware-solution-vms.md)
 