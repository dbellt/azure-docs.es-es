---
title: 'Programa Azure Certified Device: Tutorial para probar su dispositivo'
description: Una guía paso a paso para probar el dispositivo con el servicio AICS en el portal de Azure Certified Device
author: nkuntjoro
ms.author: nikuntjo
ms.service: certification
ms.topic: tutorial
ms.date: 03/02/2021
ms.custom: template-tutorial
ms.openlocfilehash: ef82d44ef44189c0430ba9789baf3279fbe49a9c
ms.sourcegitcommit: b4fbb7a6a0aa93656e8dd29979786069eca567dc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/13/2021
ms.locfileid: "107310961"
---
# <a name="tutorial-test-and-submit-your-device"></a>Tutorial: Prueba y envío de su dispositivo

La siguiente fase principal del proceso de certificación pasa por probar el dispositivo, aunque se puede completar antes de agregar los detalles del dispositivo. A través del portal, usará Azure IoT Certification Service (IACS) para demostrar el rendimiento del dispositivo de acuerdo con nuestros requisitos de certificación. Una vez que haya superado la fase de pruebas, enviará el dispositivo para revisión final y aprobación por parte del equipo de certificación de Azure.

En este tutorial aprenderá a:

> [!div class="checklist"]
> * Conectarse a Azure IoT Hub mediante Device Provisioning Service (DPS)
> * Probar el dispositivo de acuerdo con los programas de certificación seleccionados
> * Enviar el dispositivo para su revisión por parte del equipo de certificación de Azure

## <a name="prerequisites"></a>Requisitos previos

- Debe haber iniciado sesión y tener un proyecto creado para su dispositivo en el [portal de Azure Certified Device](https://certify.azure.com). Para más información, consulte el [tutorial](tutorial-01-creating-your-project.md).
- De manera opcional, le recomendamos que prepare el dispositivo y compruebe manualmente el rendimiento de acuerdo con los requisitos de certificación. Esto es así porque si desea hacer pruebas con otro código de dispositivo o programa de certificación, tendrá que crear un nuevo proyecto.

## <a name="connecting-your-device-using-dps"></a>Conexión del dispositivo mediante DPS

Todos los dispositivos certificados deben demostrar que se pueden conectar a IoT Hub mediante DPS. Los pasos siguientes le guiarán a través del procedimiento para conectar correctamente el dispositivo y hacer pruebas en el portal.

1. Para comenzar con la fase de pruebas, seleccione el vínculo `Connect & test` en la página de resumen del proyecto:  

    ![Vínculo de conexión y prueba](./media/images/connect-and-test-link.png)

1. En función de las certificaciones seleccionadas, verá las pruebas necesarias en la página "Connect & test" (Conectar y probar). Revise la información para asegurarse de que está utilizando el programa de certificación correcto.  

    ![Página de conexión y pruebas](./media/images/connect-and-test.png)

1. Conecte su dispositivo a IoT Hub mediante Device Provisioning Service (DPS). DPS es compatible con las opciones de conectividad de claves simétricas, la certificación X.509 y un Módulo de plataforma segura (TPM). Este es un requisito de todas las certificaciones.

    - *Para obtener más información sobre cómo conectar el dispositivo a Azure IoT Hub mediante DPS, consulte la [información general sobre el aprovisionamiento de dispositivos](../iot-dps/about-iot-dps.md "Información general sobre Device Provisioning Service").*
    
1. Si usa claves simétricas, se le pedirá que configure DPS con el ámbito de identificador de DPS, el identificador de dispositivo, la clave de autenticación y el punto de conexión de DPS suministrados. De lo contrario, se le pedirá que facilite el certificado X.509 o la clave de aprobación.

1. Después de configurar el dispositivo mediante DPS, confirme la conexión. Para ello, haga clic en el botón `Connect` situado en la parte inferior de la página. Una vez que la conexión sea satisfactoria, pase a la fase de pruebas haciendo clic en el botón `Next`.  

    ![Conexión y pruebas integradas](./media/images/connected.png)

## <a name="testing-your-device"></a>Pruebas del dispositivo

Una vez que haya conectado correctamente el dispositivo a AICS, podrá hacer pruebas de certificación específicas en relación con el programa de certificación que haya solicitado.

1. **Certificación de Azure Certified Device**: en la pestaña "Select device capability" (Seleccionar funcionalidad de dispositivo), revise y seleccione las pruebas que desea hacer en el dispositivo.
1. **Certificación de IoT Plug and Play**: revise cuidadosamente los parámetros que se comprobarán durante la prueba que declaró en el modelo de dispositivo.
1. **Certificación Edge Managed**: después de comprobar la conectividad, no se requieren pasos adicionales.
1. Una vez que haya realizado los preparativos necesarios para el programa de certificación específico, seleccione `Next` para pasar a la fase de pruebas.
1. Seleccione `Run tests` en la página para empezar a ejecutar AICS con su dispositivo.
1. Después de recibir la notificación de que ha superado las pruebas, seleccione `Finish` para volver a la página de resumen.

![Prueba superada](./media/images/test-pass.png)

7. Si tiene preguntas adicionales o necesita ayuda para solucionar problemas con AICS, visite nuestra guía de solución de problemas.

> [!NOTE]
> Aunque podrá completar el proceso de certificación en línea para IoT Plug and Play y Edge Managed sin necesidad de enviar el dispositivo para revisión manual, es posible que un miembro del equipo de Azure Certified Device se ponga en contacto con usted para llevar a cabo algunas comprobaciones adicionales en el dispositivo, aparte de las realizadas a través de nuestro servicio de automatización.

## <a name="submitting-your-device-for-review"></a>Envío del dispositivo para revisión

Una vez que haya completado todos los campos obligatorios en la sección "Device details" (Detalles del dispositivo) y haya superado las pruebas automatizadas del proceso "Connect & test" (Conectar y probar), podrá notificar al equipo de Azure Certified Device que está listo para la revisión de la certificación.

1. Seleccione `Submit for review` en la página de resumen del proyecto:  

    ![Vínculo de revisión y certificación](./media/images/review-and-certify.png)

1. Confirme el envío en la ventana emergente. Una vez que se ha enviado un dispositivo, todos los detalles sobre este serán de solo lectura hasta que se solicite la edición. Consulte el artículo sobre [cómo editar la información del dispositivo después de la publicación](./how-to-edit-published-device.md).  

    ![Cuadro de diálogo para iniciar la revisión de la certificación](./media/images/start-certification-review.png)

1. Una vez que se ha enviado el proyecto, la página de resumen indicará que el proyecto se encuentra `Under Certification Review` por parte del equipo de certificación de Azure:  

    ![En revisión](./media/images/review-and-certify-under-review.png)

1. En un plazo de 5 a 7 días hábiles, recibirá una respuesta de correo electrónico del equipo de certificación de Azure en la dirección facilitada en el perfil de su empresa en relación con el estado del dispositivo enviado.

    - Envío aprobado  
        Una vez que el proyecto se haya revisado y aprobado, recibirá un correo electrónico. El correo incluirá un conjunto de archivos, como el distintivo de Azure Certified Device, las directrices de uso de distintivos y otra información dirigida a dar amplia visibilidad al mensaje de que su dispositivo está certificado. Felicidades.

    - Envío pendiente  
        En caso de que el proyecto no reciba aprobación, podrá modificar los detalles del proyecto y volver a enviar el dispositivo para la certificación una vez que esté listo. Recibirá un correo electrónico con información sobre el motivo por el que el proyecto no recibió aprobación y los pasos para volver a enviarlo con fines de certificación.

## <a name="next-steps"></a>Pasos siguientes

Felicidades. El dispositivo ha superado con éxito todas las pruebas y ha recibido aprobación a través del programa Azure Certified Device. Ahora puede publicar el dispositivo en el catálogo de Azure Certified Device, desde donde los clientes pueden comprar sus productos con la confianza de que funcionan con Azure.
> [!div class="nextstepaction"]
> [Tutorial: Publicación de su dispositivo](tutorial-04-publishing-your-device.md)

