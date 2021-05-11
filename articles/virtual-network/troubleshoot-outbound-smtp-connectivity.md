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
ms.date: 04/28/2021
ms.author: genli
ms.openlocfilehash: 3cc4cb587a7b2d5d06c249cc8f25bc78cdb86739
ms.sourcegitcommit: 62e800ec1306c45e2d8310c40da5873f7945c657
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108165270"
---
# <a name="troubleshoot-outbound-smtp-connectivity-problems-in-azure"></a>Solución de problemas de conectividad SMTP saliente en Azure

Los mensajes de correo electrónico salientes que se envían directamente a dominios externos (como outlook.com y gmail.com) en el puerto TCP 25 desde una máquina virtual (VM) solo son posibles cuando la VM se ha implementado en determinados tipos de suscripción.

## <a name="recommended-method-of-sending-email"></a>Método recomendado para el envío de correo electrónico

Se recomienda usar servicios de retransmisión SMTP autenticados para enviar correo electrónico desde máquinas virtuales de Azure o desde Azure App Service. (Estos servicios de retransmisión se suelen conectar a través del puerto TCP 587, pero se admiten otros puertos). Estos servicios se usan para mantener la reputación de IP y dominio para minimizar la posibilidad de que dominios externos rechacen sus mensajes o los pongan en la carpeta SPAM. [SendGrid](https://sendgrid.com/partners/azure/) es un servicio de retransmisión SMTP, pero hay otros. Puede que también tenga un servicio de retransmisión SMTP autenticado en los servidores locales.

El uso de estos servicios de entrega de correo electrónico no está restringido en Azure, independientemente del tipo de suscripción.

## <a name="enterprise-agreement"></a>Contrato Enterprise

En el caso de las VM implementadas en suscripciones de Contrato Enterprise, las conexiones SMTP salientes en el puerto TCP 25 no se bloquearán. Sin embargo, no hay ninguna garantía de que los dominios externos acepten los correos electrónicos entrantes de las VM. Si los dominios externos rechazan o filtran los correos electrónicos, tiene que ponerse en contacto con los proveedores de servicios de correo electrónico de los dominios externos para resolver los problemas. El Soporte técnico de Azure no cubre estos problemas.

## <a name="pay-as-you-go"></a>Pago por uso

La plataforma Azure bloqueará las conexiones SMTP salientes en el puerto TCP 25 para las VM que se han implementado en suscripciones de pago por uso. Es posible quitar este bloque si la suscripción a Azure tiene buena reputación y tiene un historial de pagos suficiente. Para solicitar que se quite la restricción, vaya a la sección **No se puede enviar correo (puerto SMTP 25)** de la hoja **Diagnosticar y solucionar** de un recurso de Azure Virtual Network en [Azure Portal](https://portal.azure.com). 

Después de que una suscripción de pago por uso quede exenta de este bloqueo y se hayan detenido e iniciado las VM desde Azure Portal, todas las VM de esa suscripción estarán exentas en el futuro. La exención solo se aplica a la suscripción solicitada y al tráfico de la máquina virtual que se enruta directamente a Internet.

> [!NOTE]
> Microsoft se reserva el derecho a revocar estas exenciones si se determina que se produjo una infracción en los términos del servicio.

## <a name="msdn-azure-pass-azure-in-open-education-azure-for-students-visual-studio-and-free-trial"></a>MSDN, Pase para Azure, Azure bajo licencia Open, Education, Microsoft Azure for Students, Visual Studio y evaluación gratuita

La plataforma de Azure bloqueará las conexiones SMTP salientes en el puerto TCP 25 para las VM implementadas en los siguientes tipos de suscripción:

- MSDN
- Pase para Azure
- Azure bajo licencia Open
- Education
- Azure for Students
- Versión de prueba gratuita
- Cualquier suscripción de Visual Studio  

Las restricciones existen para evitar abusos. No se aceptará ninguna solicitud para quitar esta restricción.

Si usa estos tipos de suscripción, recomendamos usar el servicio de retransmisión SMTP autenticado, tal como se indicó anteriormente en este artículo, o cambiar el tipo de suscripción.

## <a name="cloud-solution-provider"></a>Proveedor de soluciones en la nube

La plataforma de Azure bloqueará las conexiones SMTP salientes en el puerto TCP 25 para las VM implementadas en suscripciones de Proveedor de soluciones en la nube. Es posible quitar este bloqueo. Para solicitar que se quite el bloqueo, vaya a la sección **No se puede enviar correo (puerto SMTP 25)** de la hoja **Diagnosticar y solucionar** de un recurso de Azure Virtual Network en Azure Portal y abra una solicitud de soporte técnico.

## <a name="microsoft-partner-network-bizspark-plus-or-azure-sponsorship"></a>Microsoft Partner Network, BizSpark Plus o Patrocinio de Azure

La plataforma de Azure bloqueará los intentos de entrega SMTP salientes en el puerto TCP 25 para las VM implementadas en las siguientes suscripciones:

- Microsoft Partner Network (MPN)
- BizSpark Plus
- Patrocinio de Azure

Es posible quitar este bloqueo. Para solicitar que se quite el bloqueo, vaya a la sección **No se puede enviar correo (puerto SMTP 25)** de la hoja **Diagnosticar y solucionar** de un recurso de Azure Virtual Network en Azure Portal y abra una solicitud de soporte técnico.

Después de que la suscripción quede exenta de este bloqueo y se hayan detenido y reiniciado las VM, todas las VM de esa suscripción estarán exentas en el futuro. La exención solo se aplica a la suscripción solicitada, y al tráfico de la VM que se enruta directamente a Internet.

> [!NOTE]
> Microsoft se reserva el derecho a revocar estas exenciones si se determina que se produjo una infracción en los términos del servicio.

## <a name="changing-subscription-type"></a>Cambio del tipo de suscripción

Si cambia el tipo de suscripción de Contrato Enterprise a otro tipo de suscripción, los cambios en las implementaciones pueden provocar el bloqueo de SMTP saliente. Si planea cambiar el tipo de suscripción de Contrato Enterprise a otro tipo de suscripción y requiere SMTP saliente en el puerto TCP 25, asegúrese de trabajar con Soporte técnico para desbloquear la suscripción antes de cambiar el tipo de suscripción.

## <a name="need-help-contact-support"></a>¿Necesita ayuda? Ponerse en contacto con soporte técnico

Si sigue necesitando ayuda, [póngase en contacto con el soporte técnico](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade) para resolver el problema rápidamente. Use este tipo de incidencia: **Técnico** > **Red virtual** > **Cannot send email (SMTP/Port 25)** [No se puede enviar correo (SMTP-Puerto 25)].
