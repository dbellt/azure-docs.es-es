---
title: 'Inicio rápido: Portabilidad de un número de teléfono a Azure Communication Services'
description: Obtenga información acerca de cómo portar un número de teléfono a un recurso de Communication Services.
author: mikben
manager: mikben
services: azure-communication-services
ms.author: mikben
ms.date: 06/30/2021
ms.topic: quickstart
ms.service: azure-communication-services
ms.custom: references_regions
ms.openlocfilehash: f138e0ee79e0d8adf03ba51c1197813402920a19
ms.sourcegitcommit: 98308c4b775a049a4a035ccf60c8b163f86f04ca
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/30/2021
ms.locfileid: "113112048"
---
# <a name="quickstart-port-a-phone-number"></a>Inicio rápido: Portabilidad de un número de teléfono

[!INCLUDE [Regional Availability Notice](../../includes/regional-availability-include.md)]

Comience a usar Azure Communication Services mediante la portabilidad del número de teléfono a su recurso de Azure Communication Services. Los números de teléfono y gratuitos y geográficos de Estados Unidos son válidos para la portabilidad. Para más información acerca de los tipos de números de teléfono, consulte [Tipos de número de teléfono en Azure Communication Services](../../concepts/telephony-sms/plan-solution.md).

## <a name="prerequisites"></a>Requisitos previos

- Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
- [Recurso activo de Communication Services.](../create-communication-resource.md)

## <a name="gather-your-azure-resource-details"></a>Recopilación de los detalles del recurso de Azure

Antes de iniciar una solicitud de portabilidad, vaya a Azure Portal y seleccione el recurso de Communication Services. Con el panel `Overview` (Información general) seleccionado, haga clic en el vínculo `JSON View` (Vista JSON) en la esquina superior derecha:

:::image type="content" source="./media/number-port/json-view.png" alt-text="Captura de pantalla que muestra la selección de la vista JSON.":::

Anote el **identificador de Azure** del recurso y el **identificador de recurso inmutable**:

:::image type="content" source="./media/number-port/json-properties.png" alt-text="Captura de pantalla que muestra la selección de las propiedades JSON.":::

## <a name="initiate-the-port-request"></a>Inicio de la solicitud de portabilidad

Los números de teléfono y gratuitos y geográficos de Estados Unidos son válidos para la portabilidad. Use uno de los siguientes formularios para enviar la solicitud de portabilidad:

- Para números gratuitos: [Solicitud de portabilidad de número gratuito](https://aka.ms/acs-port-form-tollfree)
- Para números geográficos de Estados Unidos: [Solicitud de portabilidad de número geográfico](https://aka.ms/acs-port-form-geographic)

Cuando haya finalizado, envíe el formulario de solicitud de portabilidad relleno a acsporting@microsoft.com. Asegúrese de que la línea del asunto del correo electrónico comienza por "ACS Port-In Request" (Solicitud de portabilidad a ACS).

## <a name="next-steps"></a>Pasos siguientes

En esta guía de inicio rápido, ha aprendido a hacer lo siguiente:

> [!div class="checklist"]
> * Obtener los metadatos del recurso de Communication Services
> * Enviar una solicitud para portar el número de teléfono

> [!div class="nextstepaction"]
> [Enviar un SMS](../telephony-sms/send.md)
> [Introducción a las llamadas](../voice-video-calling/getting-started-with-calling.md)
