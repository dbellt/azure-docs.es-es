---
title: Administración de reservas de Azure
description: Obtenga información sobre cómo administrar las reservas de Azure. Consulte los pasos para cambiar el ámbito de la reserva, dividirla y optimizar su uso.
ms.service: cost-management-billing
ms.subservice: reservations
author: bandersmsft
ms.reviewer: yashesvi
ms.topic: how-to
ms.date: 04/21/2021
ms.author: banders
ms.openlocfilehash: 64a6b3a313f5078368563af29ab97a871b4c3ae0
ms.sourcegitcommit: fc9fd6e72297de6e87c9cf0d58edd632a8fb2552
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/30/2021
ms.locfileid: "108288695"
---
# <a name="manage-reservations-for-azure-resources"></a>Administración de reservas para los recursos de Azure

Después de comprar una reserva de Azure, es posible que deba aplicar la reserva a otra suscripción, cambiar quién puede administrar la reserva o cambiar el ámbito de la reserva. También puede dividir una reserva en dos para aplicar algunas de las instancias que compró a otra suscripción.

Si ha adquirido Azure Reserved Virtual Machine Instances, puede cambiar la configuración de optimización de la reserva. Puede aplicar el descuento de reserva a las máquinas virtuales de la misma serie o puede reservar la capacidad del centro de datos para un tamaño específico de máquina virtual. Debería intentar optimizar las reservas para que se usen totalmente.

*El permiso necesario para administrar una reserva es independiente del permiso de suscripción.*

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

## <a name="reservation-order-and-reservation"></a>Pedido de reserva y reserva

Cuando compra una reserva, se crean dos objetos: **Pedido de reserva** y **Reserva**.

En el momento de la compra, un pedido de reserva tiene una sola reserva. Acciones como la división, combinación, reembolso parcial o intercambio crean nuevas reservas en el **Pedido de reserva**.

Para ver un pedido de reserva, vaya a **Reservas** > seleccione la reserva y, a continuación, seleccione el **Id. de pedido de reserva**.

![Ejemplo de los detalles de pedido de reserva que muestra el Id. de pedido de reserva ](./media/manage-reserved-vm-instance/reservation-order-details.png)

Una reserva hereda los permisos de su pedido de reserva. Para cambiar o devolver una reserva, es preciso agregar el usuario al pedido de reserva.

## <a name="change-the-reservation-scope"></a>Cambio del ámbito de reserva

 El descuento de la reserva se aplica a las máquinas virtuales, SQL Database, Azure Cosmos DB u otros recursos que se correspondan con la reserva y que se ejecuten dentro del ámbito de esta. El contexto de facturación depende de la suscripción que se usó para comprar la reserva.

Para cambiar el ámbito de una reserva:

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
2. Seleccione **Todos los servicios** > **Reservations**.
3. Seleccione la reserva.
4. Seleccione **Configuración** > **Configuración**.
5. Cambie el ámbito.

Si cambia el ámbito para que pase de ser compartido a único, solo podrá seleccionar suscripciones de las que sea el propietario. Únicamente se pueden seleccionar las suscripciones que pertenezcan al mismo contexto de facturación que la reserva.

El ámbito solo se aplica a suscripciones individuales con tarifas de pago por uso (ofertas MS-AZR-0003P o MS-AZR-0023P), la oferta Enterprise MS-AZR-0017P o MS-AZR-0148P o los tipos de suscripción CSP.

## <a name="who-can-manage-a-reservation-by-default"></a>Quiénes pueden administrar una reservas de manera predeterminada

De forma predeterminada, estos son los usuarios que pueden ver y administrar reservas:

- La persona que compró la reserva y el propietario de la cuenta de la suscripción de facturación tienen acceso mediante el control de acceso basado en roles de Azure al pedido de reserva.
-  Los colaboradores de facturación del Contrato Enterprise y el Contrato de cliente de Microsoft pueden administrar todas las reservas en Cost Management + Billing > Transacciones de reserva > seleccionando el banner azul.

Para que otras personas puedan administrar las reservas, tiene dos opciones:

- Delegar la administración de acceso en un pedido de reserva individual:
    1. Inicie sesión en [Azure Portal](https://portal.azure.com).
    1. Seleccione **Todos los servicios** > **Reservations** para enumerar las reservas a las que tiene acceso.
    1. Seleccione la reserva cuyo acceso quiere delegar a otros usuarios.
    1. En Detalles de reserva, seleccione el pedido de reserva.
    1. Seleccione **Access Control (IAM)** .
    1. Seleccione **Agregar asignación de roles** > **Rol** > **Propietario**. Si desea otorgar un acceso limitado, seleccione otro rol.
    1. Escriba la dirección de correo electrónico del usuario al que desea agregar como propietario.
    1. Seleccione el usuario y, después, **Guardar**.

- Agregue un usuario como administrador de facturación a un Contrato Enterprise o un Contrato de cliente de Microsoft:
    - En el caso de un Contrato Enterprise, agregue usuarios con el rol de _Administrador de empresa_ para ver y administrar todos los pedidos de reserva que se aplican al Contrato Enterprise. Los usuarios con el rol de _Administrador de empresa (solo lectura)_ solo pueden ver la reserva. Ni los administradores de departamento ni los propietarios de cuentas no pueden ver reservas, _salvo que_ se les agreguen mediante el Control de acceso (IAM). Para más información, consulte [Administración de roles de Azure Enterprise](../manage/understand-ea-roles.md).

        _Los administradores de empresa pueden tomar posesión de un pedido de reserva y pueden agregar otros usuarios a una reserva mediante el Control de acceso (IAM)._
    - En el caso de un Contrato de cliente de Microsoft, los usuarios con el rol Propietario de perfil de facturación o el rol Colaborador de perfil de facturación pueden administrar todas las compras de reserva realizadas mediante el perfil de facturación. Los lectores del perfil de facturación y los administradores de facturas pueden ver todas las reservas que se pagan con el perfil de facturación. Sin embargo, no pueden realizar cambios en las reservas.
    Para más información, consulte [Tareas y roles del perfil de facturación](../manage/understand-mca-roles.md#billing-profile-roles-and-tasks).

### <a name="how-billing-administrators-view-or-manage-reservations"></a>De qué forma ven o administran las reservas los administradores de facturación

Si es un administrador de facturación, siga estos pasos para ver y administrar todas las reservas y las transacciones de reserva.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) y vaya a **Cost Management + Billing**.
    - Los administradores de EA pueden seleccionar **Ámbitos de facturación** en el menú de la izquierda y, después, seleccionar uno de los ámbitos de la lista.
    - Los propietarios de un perfil de facturación de Contrato de cliente de Microsoft, deben seleccionar **Perfiles de facturación** en el menú de la izquierda. En la lista de perfiles de facturación, seleccione uno.
2. En el menú de la izquierda, seleccione **Productos y servicios** > **Reservas**.
3. Se muestra la lista completa de reservas de su perfil de facturación o inscripción de EA.
4. Los administradores de facturación pueden tomar posesión de una reserva; para ello, la seleccionan y, después, seleccionan **Conceder acceso** en la ventana que aparece.

## <a name="split-a-single-reservation-into-two-reservations"></a>División de una reserva única en dos

 Si se compran varias instancias de recursos en una reserva, se pueden asignar algunas de las instancias de la reserva a distintas suscripciones. De forma predeterminada, todas las instancias tienen un ámbito, que puede ser de suscripción única o compartida, o un grupo de recursos. Supongamos que ha hecho una reserva de diez instancias de máquina virtual y que ha especificado que el ámbito sea la suscripción A. Sin embargo, ahora desea cambiar dicho ámbito, desea que en siete de ellas siga siendo la suscripción A, pero en las tres restantes sea la suscripción B. Para realizar dicho cambio hay que usar la división de reservas. Tras dividir una reserva, se cancela el identificador de reserva original y se crean dos reservas. La división no afecta al pedido de la reserva (no hay ninguna transacción comercial nueva con división y las reservas nuevas tienen la misma fecha de finalización que la que se dividió).

 Una reserva se puede dividir en dos mediante PowerShell, CLI o la API.

### <a name="split-a-reservation-by-using-powershell"></a>División de una reserva con PowerShell

1. Ejecute el comando siguiente para obtener el identificador de pedido de reserva:

    ```powershell
    # Get the reservation orders you have access to
    Get-AzReservationOrder
    ```

2. Obtenga los detalles de una reserva:

    ```powershell
    Get-AzReservation -ReservationOrderId a08160d4-ce6b-4295-bf52-b90a5d4c96a0 -ReservationId b8be062a-fb0a-46c1-808a-5a844714965a
    ```

3. Divida la reserva en dos y distribuya las instancias:

    ```powershell
    # Split the reservation. The sum of the reservations, the quantity, must equal the total number of instances in the reservation that you're splitting.
    Split-AzReservation -ReservationOrderId a08160d4-ce6b-4295-bf52-b90a5d4c96a0 -ReservationId b8be062a-fb0a-46c1-808a-5a844714965a -Quantity 3,2
    ```
4. Si quiere actualizar el ámbito, ejecute el siguiente comando:

    ```powershell
    Update-AzReservation -ReservationOrderId a08160d4-ce6b-4295-bf52-b90a5d4c96a0 -ReservationId 5257501b-d3e8-449d-a1ab-4879b1863aca -AppliedScopeType Single -AppliedScope /subscriptions/15bb3be0-76d5-491c-8078-61fe3468d414
    ```

## <a name="cancel-exchange-or-refund-reservations"></a>Cancelación, intercambio o reembolso de reservas

Puede cancelar, intercambiar o reembolsar reservas con ciertas limitaciones. Para más información, consulte [Autoservicio de intercambios y reembolsos de reservas de Azure](exchange-and-refund-azure-reservations.md).

## <a name="change-optimize-setting-for-reserved-vm-instances"></a>Cambiar la configuración de optimización para instancias reservadas de máquina virtual

 Al comprar una instancia reservada de máquina virtual, puede elegir entre la flexibilidad de tamaño de instancia o la prioridad de capacidad. En la opción de flexibilidad del tamaño de instancia se aplica el descuento por la reserva a otras máquinas virtuales del mismo [grupo](../../virtual-machines/reserved-vm-instance-size-flexibility.md). La prioridad de capacidad designa la capacidad del centro de datos más importante para las implementaciones. Esta opción le ofrece una mayor confianza en su capacidad para iniciar instancias de máquinas virtuales cuando las necesite.

De forma predeterminada, cuando se comparte el ámbito de la reserva, la flexibilidad de tamaño de instancia está activada. La capacidad del centro de datos no tiene prioridad en las implementaciones de máquina virtual.

En las reservas donde el ámbito es único, puede optimizar la reserva para la prioridad de capacidad en lugar de la flexibilidad del tamaño de instancia de máquina virtual.

Para actualizar la configuración de optimización de la reserva:

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
2. Seleccione **Todos los servicios** > **Reservations**.
3. Seleccione la reserva.
4. Seleccione **Configuración** > **Configuración**.
  ![Ejemplo que muestra el elemento Configuración](./media/manage-reserved-vm-instance/add-product03.png)
5. Cambie la configuración de la **optimización**.
  ![Ejemplo que muestra la configuración Optimizar para](./media/manage-reserved-vm-instance/instance-size-flexibility-option.png)

## <a name="optimize-reservation-use"></a>Optimización del uso de reservas

El ahorro de reservas de Azure solo se produce al usar recursos sostenidos. Cuando realiza una compra de reserva, paga por lo que es esencialmente el 100 % de uso de recursos durante un período de uno o tres años. Trate de maximizar la reserva para obtener el mayor uso y ahorros posibles. En las secciones siguientes se explica cómo supervisar una reserva y optimizar su uso.

### <a name="view-reservation-use-in-the-azure-portal"></a>Visualización del uso de las reservas en Azure Portal

Una manera de ver el uso de las reservas es mediante Azure Portal.

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).
2. Seleccione **Todos los servicios** > [**Reservations**](https://portal.azure.com/#blade/Microsoft_Azure_Reservations/ReservationsBrowseBlade) y anote el valor de **Uso (%)** de una reserva.  
  ![Imagen que muestra la lista de reservas](./media/manage-reserved-vm-instance/reservation-list.png)
3. Seleccione una reserva.
4. Revise la tendencia de uso de la reserva a lo largo del tiempo.
  ![Imagen que muestra el uso de la reserva](./media/manage-reserved-vm-instance/reservation-utilization-trend.png)

### <a name="view-reservation-use-with-api"></a>Visualización del uso de las reservas con la API

Si es un cliente con Contrato Enterprise (EA), puede ver mediante programación cómo se usan las reservas en su organización. Obtiene las reservas sin usar mediante los datos de uso. Al revisar los cargos de la reserva, tenga en cuenta que los datos se dividen entre el costo real y los costos amortizados. El costo real proporciona datos para conciliar la factura mensual. También tiene los costos de compra de reserva y los detalles de solicitud de reservas. El costo amortizado es como el costo real, salvo que se prorratea el precio efectivo para el uso de reserva. Las horas de reserva sin usar se muestran en datos de costos amortizados. Para más información sobre los datos de uso para los clientes de Contrato Enterprise, consulte [Obtención del uso y los costos de reservas de Contrato Enterprise](understand-reserved-instance-usage-ea.md).

Para otros tipos de suscripciones, use [Reservations Summaries - List By Reservation Order And Reservation](/rest/api/consumption/reservationssummaries/listbyreservationorderandreservation) API.

### <a name="optimize-your-reservation"></a>Optimización de la reserva

Si ve que las reservas de su organización se están utilizando por debajo de su capacidad:

- Asegúrese de que las máquinas virtuales que crea su organización coincidan con el tamaño de la máquina virtual que se encuentra en la reserva.
- Asegúrese de que la flexibilidad de tamaño de instancia esté activada. Para obtener más información, consulte [Administrar reservas: cambiar la configuración de optimización para instancias reservadas de máquina virtual](#change-optimize-setting-for-reserved-vm-instances).
- Cambie el ámbito de la reserva para que sea _compartido_ y así poder aplicarlo de manera más amplia. Para más información, consulte [Cambio del ámbito en una reserva](#change-the-reservation-scope).
- Considere la posibilidad de intercambiar la cantidad no utilizada. Para más información, consulte [Cancelaciones y cambios](#cancel-exchange-or-refund-reservations).


## <a name="need-help-contact-us"></a>¿Necesita ayuda? Póngase en contacto con nosotros.

Si tiene alguna pregunta o necesita ayuda, [cree una solicitud de soporte técnico](https://go.microsoft.com/fwlink/?linkid=2083458).

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información acerca de Azure Reservations, consulte los siguientes artículos:
 - [Visualización del uso de las reservas](reservation-utilization.md)
 - [Cambios y reembolsos](exchange-and-refund-azure-reservations.md)
 - [Renovación de reservas](reservation-renew.md)
 - [Transferencias entre inquilinos](troubleshoot-reservation-transfers-between-tenants.md)
 - [Búsqueda de un comprador de reservas en los registros de Azure](find-reservation-purchaser-from-logs.md)
