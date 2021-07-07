---
title: Planes de hospedaje de Azure Static Web Apps
description: Compare los distintos planes de hospedaje de Azure Static Web Apps.
services: static-web-apps
author: craigshoemaker
ms.service: static-web-apps
ms.topic: overview
ms.date: 05/14/2021
ms.author: cshoe
ms.openlocfilehash: 2fcb9805b752273cb5320114342108ed1f844a5b
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110105428"
---
# <a name="azure-static-web-apps-hosting-plans"></a>Planes de hospedaje de Azure Static Web Apps

Azure Static Web Apps está disponible a través de dos planes diferentes: Gratis y Estándar. Consulte la [página de precios para conocer los costos del plan Estándar](https://azure.microsoft.com/pricing/details/app-service/static/).

## <a name="features"></a>Características

| Característica | Plan gratuito <br> (Para proyectos personales) | Plan Estándar <br> (Para aplicaciones de producción) |
| --- | --- | --- |
| Hospedaje web | ✔ | ✔ |
| Integración de GitHub | ✔ | ✔ |
| Integración de Azure DevOps | ✔ | ✔ |
| Contenido estático distribuido globalmente | ✔ | ✔ |
| Certificados SSL gratuitos y que se renuevan automáticamente | ✔ | ✔ |
| Entornos de ensayo | 3 por aplicación | 10 por aplicación |
| Tamaño máximo de la aplicación | 250 MB por aplicación | 500 MB por aplicación |
| Dominios personalizados | 2 por aplicación | 5 por aplicación |
| API a través de Azure Functions | Administrado | Administrado o<br>[Traiga sus propias aplicaciones de Functions](functions-bring-your-own.md) |
| Integración del proveedor de autenticación | [Preconfigurada](authentication-authorization.md)<br>(El servicio se encarga de definir este valor) | [Registros personalizados](authentication-custom.md) |
| [Acuerdo de Nivel de Servicio (SLA)](https://azure.microsoft.com/support/legal/sla/app-service-static/v1_0/) | Ninguno  | ✔ |

## <a name="selecting-a-plan"></a>Selección de un plan

Los siguientes ejemplos pueden ayudarle a decidir si el plan Estándar se adapta mejor a sus necesidades.

- Los volúmenes de tráfico esperados superan los máximos de ancho de banda.
- La aplicación Azure Functions existente que quiere usar tiene desencadenadores y enlaces más allá de los puntos de conexión HTTP o no se puede convertir en una aplicación de Functions administrada.
- Los requisitos de seguridad deben tener un [registro de proveedor personalizado](authentication-custom.md).
- El tamaño total del archivo de los recursos web del sitio supera el máximo de almacenamiento.
- Necesita soporte técnico formal para el cliente.
- Necesita más de tres [entornos de ensayo](review-publish-pull-requests.md).

Consulte la [guía de cuotas](quotas.md) para obtener más información sobre las limitaciones.

## <a name="changing-plans"></a>Cambio de planes

Puede cambiar entre los planes Gratis o Estándar a través de Azure Portal.

1. Vaya a un recurso de Static Web Apps en Azure Portal.

1. En el menú _Configuración_, seleccione **Plan de hospedaje**.

1. Seleccione el plan de hospedaje que quiera para la aplicación web estática.

1. Seleccione **Guardar**.
