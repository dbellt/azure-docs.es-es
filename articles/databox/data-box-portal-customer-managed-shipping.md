---
title: Envío autoadministrado de Microsoft Azure Data Box | Microsoft Docs en datos
description: Describe el flujo de trabajo de envío autoadministrado para dispositivos Azure Data Box.
services: databox
author: alkohli
ms.service: databox
ms.subservice: disk
ms.topic: how-to
ms.date: 05/08/2021
ms.author: alkohli
ms.openlocfilehash: 75e5737f8ad7b0f40e8480c17e150669220007ff
ms.sourcegitcommit: 1b19b8d303b3abe4d4d08bfde0fee441159771e1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2021
ms.locfileid: "109753694"
---
# <a name="use-self-managed-shipping-for-azure-data-box-in-the-azure-portal"></a>Uso del envío autoadministrado para Azure Data Box en Azure Portal

En este artículo se describen las tareas de envío autoadministradas para pedir, recoger y descartar un dispositivo Azure Data Box. Puede administrar el dispositivo Data Box mediante Azure Portal.

## <a name="prerequisites"></a>Requisitos previos

El envío autoadministrado está disponible como opción cuando [realiza un pedido de Azure Data Box](data-box-deploy-ordered.md). Solo está disponible en las siguientes regiones:

* US Gov
* Reino Unido
* Europa Occidental
* Japón
* Singapur
* Corea del Sur
* India
* Sudáfrica
* Australia
* Brasil

## <a name="use-self-managed-shipping"></a>Uso del envío autoadministrado

Al realizar el pedido de Data Box, puede elegir la opción de envío autoadministrado.

1. En el pedido de Azure Data Box, en **Detalles del contacto**, seleccione **+ Agregar dirección de envío**.
 
   ![Envío autoadministrado, Agregar dirección de envío](media\data-box-portal-customer-managed-shipping\choose-self-managed-shipping-1.png)

2. Al elegir un tipo de envío, seleccione la opción **Envío autoadministrado**. Esta opción solo está disponible si se encuentra en una región admitida, tal como se describe en los requisitos previos.

3. Una vez proporcionada la dirección de envío, tendrá que validarla y completar el pedido.

   ![Envío autoadministrado, validar y agregar dirección](media\data-box-portal-customer-managed-shipping\choose-self-managed-shipping-2.png)

4. Cuando el dispositivo esté preparado y haya recibido una notificación por correo electrónico, podrá programar una recogida.

   En el pedido de Azure Data Box, vaya a **Información general** y seleccione **Programar recogida**.

   ![Pedido de Data Box, opción Programar recogida](media\data-box-portal-customer-managed-shipping\data-box-portal-schedule-pickup-01.png)

5. Siga las instrucciones indicadas en **Programar recogida para Azure**.

   Antes de poder obtener el código de autorización, debe enviar un correo electrónico a [adbops@microsoft.com](mailto:adbops@microsoft.com) para programar la recogida del dispositivo en el centro de datos de su región.

   ![Instrucciones de programar recogida para Azure](media\data-box-portal-customer-managed-shipping\data-box-portal-schedule-pickup-email-01.png)

   **Instrucciones para Brasil:** si va a programar una recogida de dispositivos en Brasil, incluya la siguiente información en el correo electrónico. El centro de datos programará la recogida después de recibir una `Nota Fiscal` de entrada, que puede tardar hasta 4 días laborables.

   ```
   Subject: Request Azure Data Box Disk pickup for order: <ordername>

   - Order name
   - Company name
   - Company legal name (if different) 
   - Tax ID 
   - Address 
   - Country 
   - Phone number 
   - Contact name of the person who will pick up the Data Box Disk (A government-issued photo ID will be required to validate the contact’s identity upon arrival.)   
   ```

6. Una vez programada la recogida del dispositivo, puede ver el código de autorización del dispositivo en el panel **Programar recogida para Azure**.

   ![Visualización del código de autorización del dispositivo](media\data-box-portal-customer-managed-shipping\data-box-portal-auth-01b.png)

   Tome nota de este **código de autorización**. La persona que recoja el dispositivo deberá proporcionarlo.

   Según los requisitos de seguridad, en el momento de programar la recogida es necesario proporcionar el nombre y los detalles de la persona que se encargará de la recogida. Usted o la persona de contacto deben llevar un identificador con fotografía aprobado por el Gobierno, que se validará en el centro de datos.

7. Recoja el dispositivo Data Box en el centro de datos a la hora programada.

   La persona que recoja el dispositivo debe proporcionar lo siguiente:

   * Una copia de la confirmación por correo electrónico para visitar el centro de datos remitida por Operaciones de Microsoft.

   * El código de autorización. El número de referencia es exclusivo de una recogida o una entrega y se valida en el centro de datos.

   * Identificador con fotografía aprobado por el Gobierno. El identificador se validará en el centro de datos y se deben proporcionar el nombre y los detalles de la persona que recogerá el dispositivo cuando se programe la recogida.

   > [!NOTE]
   > Si no se acude a una cita programada, deberá programar una nueva cita.

8. El pedido cambia automáticamente al estado **Recogido** una vez que el dispositivo se ha recogido en el centro de datos.

    ![Un pedido en estado Recogido](media\data-box-portal-customer-managed-shipping\data-box-portal-picked-up-boxed-01.png)

9. Después de recogerlo, copie los datos al dispositivo Data Box en su sitio. Una vez completada la copia, prepare para enviar el dispositivo Data Box. Para obtener más información, vaya a [Preparación para el envío](data-box-deploy-picked-up.md#prepare-to-ship).

   El paso **Preparación para el envío** debe completarse sin errores críticos. De lo contrario, deberá volver a ejecutar este paso después de realizar las correcciones necesarias. Una vez completado correctamente el paso **Preparación para el envío**, puede ver el código de autorización para la entrega en la interfaz de usuario local del dispositivo.

   > [!NOTE]
   > No comparta el código de autorización por correo electrónico. Este solo debe verificarse en el centro de datos durante la entrega.

   **Instrucciones para Brasil:** para programar la devolución de un dispositivo en Brasil, envíe un correo electrónico a [adbops@microsoft.com](mailto:adbops@microsoft.com) con la siguiente información:

   ```
   Subject: Request Azure Data Box Disk drop-off for order: <ordername>

   - Order name
   - Contact name of the person who will drop off the Data Box Disk (A government-issued photo ID will be required to validate the contact’s identity upon arrival.) 
   - Inbound Nota Fiscal (A copy of the inbound Nota Fiscal will be required at drop-off.)   
   ```

10. Si ha recibido una cita para una entrega, el pedido debe encontrarse en el estado **Listo para recibir el paquete en el centro de datos de Azure** en Azure Portal. Siga las instrucciones que aparecen en **Programar la entrega** para devolver el dispositivo.

    ![Instrucciones para la entrega del dispositivo](media\data-box-portal-customer-managed-shipping\data-box-portal-received-complete-02b.png)

11. Después de verificar el identificador y el código de autorización y de entregar el dispositivo en el centro de datos, el estado del pedido debe ser **Recibido**.

    ![Un pedido con el estado Recibido](media\data-box-portal-customer-managed-shipping\data-box-portal-received-complete-01.png)

12. Una vez que se recibe el dispositivo, la copia de datos continuará. El pedido se completará cuando finalice la copia.

## <a name="next-steps"></a>Pasos siguientes

* [Introducción a Azure Data Box](data-box-quickstart-portal.md)
