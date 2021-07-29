---
title: Envío autoadministrado de Microsoft Azure Data Box Disk | Microsoft Docs en datos
description: Describe el flujo de trabajo de envío autoadministrado para dispositivos Azure Data Box Disk.
services: databox
author: alkohli
ms.service: databox
ms.subservice: disk
ms.topic: how-to
ms.date: 06/22/2021
ms.author: alkohli
ms.openlocfilehash: 66ff9018371de309b61824895492335e0d3bb763
ms.sourcegitcommit: 096e7972e2a1144348f8d648f7ae66154f0d4b39
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/23/2021
ms.locfileid: "112517661"
---
# <a name="use-self-managed-shipping-for-azure-data-box-disk-in-the-azure-portal"></a>Uso del envío autoadministrado para Azure Data Box Disk en Azure Portal

En este artículo se describen las tareas de envío autoadministradas para pedir, recoger y entregar un dispositivo Azure Data Box Disk. Puede administrar Data Box Disk mediante Azure Portal.

## <a name="prerequisites"></a>Requisitos previos

El envío autoadministrado está disponible como opción cuando [realiza un pedido de Azure Data Box Disk](data-box-disk-deploy-ordered.md). Solo está disponible en las siguientes regiones:

* US Gov
* Reino Unido
* Europa Occidental
* Australia
* Japón
* Singapur
* Corea del Sur
* Sudáfrica
* India (versión preliminar)
* Brasil

## <a name="use-self-managed-shipping"></a>Uso del envío autoadministrado

Al realizar un pedido de Data Box Disk, puede elegir la opción de envío autoadministrado.

1. En el pedido de Azure Data Box Disk, en **Detalles del contacto**, seleccione **+ Agregar dirección de envío**.

   ![Captura de pantalla del asistente para pedidos que muestra el paso Detalles de contacto, con la opción Agregar dirección de envío seleccionada.](media\data-box-portal-customer-managed-shipping\choose-self-managed-shipping-1.png)

2. Al elegir el tipo de envío, seleccione la opción **Envío autoadministrado**. Esta opción solo está disponible si se encuentra en una región admitida, tal como se describe en los requisitos previos.

3. Una vez proporcionada la dirección de envío, tendrá que validarla y completar el pedido.

   ![Captura de pantalla del cuadro de diálogo Agregar dirección de envío, con las opciones Enviar con y la opción Agregar dirección de envío seleccionadas.](media\data-box-portal-customer-managed-shipping\choose-self-managed-shipping-2.png)

4. Cuando el dispositivo esté preparado y haya recibido una notificación por correo electrónico, podrá programar una recogida. En el pedido de Azure Data Box Disk, vaya a **Información general** y seleccione **Programar recogida**.

   ![Pedir un dispositivo Data Box para su recogida](media\data-box-disk-portal-customer-managed-shipping\data-box-disk-user-pickup-01b.png)

5. Siga las instrucciones indicadas en **Programar recogida para Azure**. Antes de poder obtener el código de autorización, debe enviar un correo electrónico a [adbops@microsoft.com](mailto:adbops@microsoft.com) para programar la recogida del dispositivo en el centro de datos de su región.

   ![Programación de la recogida](media\data-box-disk-portal-customer-managed-shipping\data-box-disk-user-pickup-02c.png)

   **Instrucciones para Brasil:** si va a programar una recogida de dispositivos en Brasil, incluya la siguiente información en el correo electrónico. El centro de datos programará la recogida después de recibir una `Nota Fiscal` de entrada, que puede tardar hasta 4 días laborables.

   ```
   Subject: Request Azure Data Box Disk pickup for order: <ordername>

   - Order name
   - Company name
   - Company legal name (if different) 
   - CNPJ (Business Tax ID, format: 00.000.000/0000-00) or CPF (Individual Tax ID, format: 000.000.000-00)
   - Address 
   - Country 
   - Phone number 
   - Contact name of the person who will pick up the Data Box Disk (A government-issued photo ID will be required to validate the contact’s identity upon arrival.)   
   ```

6. Una vez que haya programado la recogida del dispositivo, puede ver el código de autorización en **Programar recogida para Azure**.

   ![Captura de pantalla del cuadro de diálogo Programar recogida para Azure con el código de autorización del cuadro de texto Recogida seleccionado.](media\data-box-disk-portal-customer-managed-shipping\data-box-disk-authcode-01b.png)

   Tome nota de este código de autorización. La persona que recoja el dispositivo deberá tenerlo.

   Según los requisitos de seguridad, en el momento de programar la recogida es necesario proporcionar el nombre y los detalles de la persona que se encargará de la recogida. Usted o la persona de contacto deben llevar un documento de identidad con fotografía aprobado por el Gobierno, que se validará en el centro de datos.

7. Recoja el dispositivo Data Box Disk en el centro de datos a la hora programada.

   La persona que recoja el dispositivo debe proporcionar lo siguiente:

   * Una copia de la confirmación por correo electrónico para visitar el centro de datos remitida por Operaciones de Microsoft.

   * El código de autorización. El número de referencia es exclusivo de una recogida o una entrega y se valida en el centro de datos.

   * Identificador con fotografía aprobado por el Gobierno. El identificador se validará en el centro de datos y se deben proporcionar el nombre y los detalles de la persona que recogerá el dispositivo cuando se programe la recogida.

   > [!NOTE]
   > Si no se acude a una cita programada, deberá programar una nueva cita.

8. El pedido cambia automáticamente al estado **Recogido** una vez que el dispositivo se ha recogido en el centro de datos.

   ![Picked up (Recogido)](media\data-box-disk-portal-customer-managed-shipping\data-box-disk-ready-disk-01b.png)

9. Después de recogerlo, podrá copiar los datos en Data Box Disk en su sitio. Una vez completada la copia, prepare para enviar el dispositivo Data Box Disk.

   Una vez finalizada la copia de los datos, póngase en contacto con el departamento de operaciones para programar una cita para la entrega. Tendrá que compartir los detalles de la persona que acudirá al centro de datos para entregar el disco. El centro de recursos también tendrá que verificar el código de autorización en el momento de la entrega. Encontrará el código de autorización para la entrega en Azure Portal en la sección **Schedule drop off** (Programar entrega).

   > [!NOTE]
   > No comparta el código de autorización por correo electrónico. Este solo debe verificarse en el centro de datos durante la entrega.

   **Instrucciones para Brasil:** para programar la devolución de un dispositivo en Brasil, envíe un correo electrónico a [adbops@microsoft.com](mailto:adbops@microsoft.com) con la siguiente información:

   ```
   Subject: Request Azure Data Box Disk drop-off for order: <ordername>

   - Order name
   - Contact name of the person who will drop off the Data Box Disk (A government-issued photo ID will be required to validate the contact’s identity upon arrival.) 
   - Inbound Nota Fiscal (A copy of the inbound Nota Fiscal will be required at drop-off.)   
   ```

10. Una vez recibida la cita para una entrega, el pedido debe encontrarse en el estado **Listo para recibir el paquete en el centro de datos de Azure** en Azure Portal.

    ![Captura de pantalla del cuadro de diálogo Agregar dirección de envío, con las opciones Enviar con y la opción Agregar dirección de envío seleccionadas.](media\data-box-disk-portal-customer-managed-shipping\data-box-disk-authcode-dropoff-02b.png)

11. Después de verificar el identificador y el código de autorización y de entregar el dispositivo en el centro de datos, el estado del pedido debe ser **Recibido**.

    ![Recepción completada](media\data-box-disk-portal-customer-managed-shipping\data-box-disk-received-01a.png)

11. Una vez que se recibe el dispositivo, la copia de datos continuará. El pedido se completará cuando finalice la copia.

## <a name="next-steps"></a>Pasos siguientes

* [Inicio rápido: Implementación de Azure Data Box Disk mediante Azure Portal](data-box-disk-quickstart-portal.md)
