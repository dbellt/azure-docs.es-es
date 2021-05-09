---
title: Solución de problemas de conectividad SMTP saliente en Azure | Microsoft Docs
description: Sepa cuál es el método recomendado para enviar correo electrónico y cómo solucionar problemas de conectividad SMTP saliente en Azure.
services: virtual-network
author: genlin
manager: dcscontentpm
editor: ''
ms.service: virtual-network
ms.devlang: na
ms.topic: troubleshooting
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 01/04/2021
ms.author: genli
ms.openlocfilehash: a8f378300a93fca8c7c283ad0ad0959ad93ec0bc
ms.sourcegitcommit: 19dcad80aa7df4d288d40dc28cb0a5157b401ac4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107896466"
---
# <a name="troubleshoot-outbound-smtp-connectivity-problems-in-azure"></a>Solución de problemas de conectividad SMTP saliente en Azure

A partir del 15 de noviembre de 2017, los mensajes de correo electrónico salientes que se envían directamente a dominios externos (como outlook.com y gmail.com) desde una máquina virtual (VM) solo están disponibles a ciertos tipos de suscripción de Azure. Las conexiones SMTP salientes que usan el puerto TCP 25 se bloquearon. (El puerto 25 se usa principalmente para la entrega de correo electrónico sin autenticación).

Este cambio de comportamiento solo se aplica a suscripciones e implementaciones creadas a partir del 15 de noviembre de 2017.

## <a name="recommended-method-of-sending-email"></a>Método recomendado para el envío de correo electrónico

Se recomienda usar servicios de retransmisión SMTP autenticados para enviar correo electrónico desde máquinas virtuales de Azure o desde Azure App Service. (Estos servicios de retransmisión se suelen conectar a través del puerto TCP 587, pero se admiten otros puertos). Estos servicios se usan para mantener la reputación de IP o dominio y minimizar la posibilidad de que proveedores de correo electrónico de terceros rechacen el mensaje. [SendGrid](https://sendgrid.com/partners/azure/) es un servicio de retransmisión SMTP, pero hay otros. También puede tener un servicio de retransmisión SMTP seguro que se ejecute de manera local que pueda usar.

El uso de estos servicios de entrega de correo electrónico no está restringido en Azure, independientemente del tipo de suscripción.

## <a name="enterprise-agreement"></a>Contrato Enterprise

Para los usuarios de Azure de Contrato Enterprise, no hay ningún cambio en la capacidad técnica para enviar correo electrónico sin usar una retransmisión autenticada. Tanto los usuarios nuevos como existentes de Contrato Enterprise puede probar la entrega de correo electrónico saliente desde máquinas virtuales de Azure directamente a proveedores de correo electrónico externos sin ninguna restricción desde la plataforma de Azure. No hay ninguna garantía de que los proveedores de correo electrónico acepten correo electrónico entrante de cualquier usuario determinado. Sin embargo, la plataforma de Azure no bloqueará los intentos de entrega de máquinas virtuales de las suscripciones de Contrato Enterprise. Tendrá que trabajar directamente con los proveedores de correo electrónico para corregir los problemas de entrega de mensajes o de filtrado de correo no deseado que implique a proveedores específicos.

## <a name="pay-as-you-go"></a>Pago por uso

Si se suscribió antes del 15 de noviembre de 2017 con una suscripción de pago por uso, no habrá ningún cambio en la capacidad técnica de probar la entrega de correo electrónico saliente. Seguirá teniendo la posibilidad de probar la entrega de correo electrónico saliente desde máquinas virtuales de Azure dentro de estas suscripciones directamente a proveedores de correo electrónico externos sin ninguna restricción desde la plataforma de Azure. De nuevo, no hay ninguna garantía de que los proveedores de correo electrónico acepten correo electrónico entrante de cualquier usuario determinado. Los usuarios tendrán que trabajar directamente con proveedores de correo electrónico para corregir los problemas de entrega de mensajes o de filtrado de correo no deseado que implique a proveedores específicos.

Para las suscripciones de pago por uso creadas después del 15 de noviembre de 2017, habrá restricciones técnicas para bloquear el correo electrónico que se envía directamente desde máquinas virtuales dentro de estas suscripciones. Si desea poder enviar correo electrónico desde máquinas virtuales de Azure directamente a proveedores de correo electrónico externos (sin usar una retransmisión SMTP autenticada) y tiene una cuenta con buena reputación y un historial de pagos, puede solicitar que se quite la restricción. Puede hacerlo en la sección **Cannot send email (SMTP-Port 25)** [No se puede enviar correo (SMTP-Puerto 25)] de la hoja **Diagnose and Solve** (Diagnosticar y solucionar) de un recurso de Azure Virtual Network en Azure Portal. Si la solicitud se acepta, se habilitará la suscripción o recibirá las instrucciones sobre los pasos siguientes. 

Después de que una suscripción de pago por uso quede exenta y se hayan detenido e iniciado las máquinas virtuales desde Azure Portal, todas las máquinas virtuales de esa suscripción estarán exentas en el futuro. La exención solo se aplica a la suscripción solicitada y al tráfico de la máquina virtual que se enruta directamente a Internet.

> [!NOTE]
> Microsoft se reserva el derecho a revocar estas exenciones si se determina que se produjo una infracción en los términos del servicio.

## <a name="msdn-azure-pass-azure-in-open-education-azure-for-students-visual-studio-and-free-trial"></a>MSDN, Pase para Azure, Azure bajo licencia Open, Education, Microsoft Azure for Students, Visual Studio y evaluación gratuita

Si creó uno de los siguientes tipos de suscripción después del 15 de noviembre de 2017, tendrá restricciones técnicas que bloquean el correo electrónico enviado desde las máquinas virtuales de la suscripción directamente a los proveedores de correo electrónico:
- MSDN
- Pase para Azure
- Azure bajo licencia Open
- Education
- Azure for Students
- Versión de prueba gratuita
- Cualquier suscripción de Visual Studio  

Las restricciones existen para evitar abusos. No se aceptará ninguna solicitud para quitar esta restricción.

Si usa estos tipos de suscripción, recomendamos usar los servicios de retransmisión SMTP tal como se indicó anteriormente en este artículo, o cambiar el tipo de suscripción.

## <a name="cloud-solution-provider"></a>Proveedor de soluciones en la nube

Si usa recursos de Azure a través de un proveedor de soluciones en la nube, puede realizar una solicitud para quitar la restricción en la sección **Cannot send mail (SMTP-Port 25))** [No se puede enviar correo (SMTP-Puerto 25)] del panel **Diagnose and Solve** (Diagnosticar y solucionar) de un recurso de red virtual en Azure Portal. Si la solicitud se acepta, se habilitará la suscripción o recibirá las instrucciones sobre los pasos siguientes.

## <a name="microsoft-partner-network-bizspark-plus-or-azure-sponsorship"></a>Microsoft Partner Network, BizSpark Plus o Patrocinio de Azure

Para las suscripciones de los tipos siguientes creadas después del 15 de noviembre de 2017, habrá restricciones técnicas para bloquear el correo electrónico que se envía directamente desde máquinas virtuales dentro de estas suscripciones:

- Microsoft Partner Network (MPN)
- BizSpark Plus
- Patrocinio de Azure

Si quiere poder enviar correo electrónico desde VM de Azure directamente a proveedores de correo electrónico externos (sin usar retransmisión SMTP autenticada), puede realizar una solicitud mediante la apertura de un caso de soporte técnico con el siguiente tipo de incidencia: **Técnico** > **Red virtual** > **Cannot send email (SMTP/Port 25)** [No se puede enviar correo (SMTP-Puerto 25)]. Asegúrese de agregar detalles sobre por qué la implementación tiene que enviar correo directamente a los proveedores de correo en lugar de usar una retransmisión autenticada. Las solicitudes se revisarán y aprobarán a discreción de Microsoft. Las solicitudes se pueden conceder solo después de completar comprobaciones antifraudes adicionales. 

Después de que una suscripción quede exenta y se hayan detenido e iniciado las máquinas virtuales en Azure Portal, todas las máquinas virtuales de esa suscripción estarán exentas en el futuro. La exención solo se aplica a la suscripción solicitada y al tráfico de la máquina virtual que se enruta directamente a Internet.

## <a name="changing-subscription-type"></a>Cambio del tipo de suscripción

Si cambia el tipo de suscripción o el proveedor de soluciones de nube o la suscripción de pago por uso está aprobada, debe detener, desasignar y luego reiniciar la máquina virtual para que la nueva directiva surta efecto. Del mismo modo, si tiene un tipo de suscripción permitido de forma predeterminada y, luego, cambia a un tipo de suscripción no permitido, el puerto 25 podría bloquearse después de los cambios en la implementación.

## <a name="need-help-contact-support"></a>¿Necesita ayuda? Ponerse en contacto con soporte técnico

Si sigue necesitando ayuda, [póngase en contacto con el soporte técnico](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade) para resolver el problema rápidamente. Use este tipo de incidencia: **Técnico** > **Red virtual** > **Cannot send email (SMTP/Port 25)** [No se puede enviar correo (SMTP-Puerto 25)].
