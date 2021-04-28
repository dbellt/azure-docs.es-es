---
title: Información general del portal para desarrolladores en Azure API Management
titleSuffix: Azure API Management
description: Obtenga información sobre el portal para desarrolladores de API Management, un sitio web personalizable, donde los consumidores de API pueden explorarlas.
services: api-management
documentationcenter: API Management
author: mikebudzynski
ms.service: api-management
ms.topic: article
ms.date: 04/15/2021
ms.author: apimpm
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 3f5da5ed6c828278eb25a9fa4fa5cfb90e16f8f4
ms.sourcegitcommit: 260a2541e5e0e7327a445e1ee1be3ad20122b37e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/21/2021
ms.locfileid: "107815966"
---
# <a name="overview-of-the-developer-portal"></a>Información general del portal para desarrolladores

El portal para desarrolladores es un sitio web totalmente personalizable que se genera automáticamente con la documentación de las API. Aquí, los consumidores de API pueden detectar las API, aprender a usarlas, solicitar acceso y probarlas.

Como se ha presentado en este artículo, puede personalizar y ampliar el portal para desarrolladores para escenarios específicos. 

![API Management developer portal](media/api-management-howto-developer-portal/cover.png)

[!INCLUDE [premium-dev-standard-basic.md](../../includes/api-management-availability-premium-dev-standard-basic.md)]

## <a name="migration-from-the-legacy-portal"></a>Migración desde el portal heredado

> [!IMPORTANT]
> El portal para desarrolladores heredado está en desuso y solo recibirá actualizaciones de seguridad. Puede seguir utilizándolo como de costumbre hasta su retirada en octubre de 2023; entonces se quitará de todos los servicios de API Management.

La migración al nuevo portal para desarrolladores se describe en el [artículo de documentación dedicada](developer-portal-deprecated-migration.md).

## <a name="customization-and-styling"></a>Personalización y aplicación de estilos

El portal para desarrolladores se puede personalizar y puede aplicarle un estilo mediante el editor visual integrado para arrastrar y colocar elementos. Consulte [este tutorial](api-management-howto-developer-portal-customize.md) para obtener más información.

## <a name="extensibility"></a><a name="managed-vs-self-hosted"></a> Extensibilidad

El servicio de API Management incluye un portal para desarrolladores integrado y **administrado** que siempre está actualizado. Puede obtener acceso al mismo desde la interfaz de Azure Portal.

Si necesita ampliarlo con una lógica personalizada que no se admite de forma integrada, puede modificar su código base. El código base del portal está [disponible en un repositorio de GitHub](https://github.com/Azure/api-management-developer-portal). Por ejemplo, puede implementar un nuevo widget que se integra con un sistema de soporte de terceros. Al implementar una nueva funcionalidad, puede elegir una de las siguientes opciones:

- **Autohospedaje** del portal resultante fuera del servicio API Management. Cuando se autohospeda el portal, se convierte en su administrador y será responsable de sus actualizaciones. La ayuda del Soporte técnico de Azure solo está limitada a la [configuración básica de portales autohospedados](developer-portal-self-host.md).
- Abra una solicitud de incorporación de cambios para que el equipo de API Management combine la nueva funcionalidad con el código base del portal **administrado**.

Para obtener información detallada e instrucciones de extensibilidad, consulte el [repositorio de GitHub](https://github.com/Azure/api-management-developer-portal) y el tutorial para la [implementación de un widget](developer-portal-implement-widgets.md). En el tutorial para [personalizar el portal administrado](api-management-howto-developer-portal-customize.md) obtendrá una serie de pasos que le ayudarán a usar el panel administrativo del portal, que es común para las versiones **administradas** y **autohospedadas**.


## <a name="next-steps"></a>Pasos siguientes

Más información acerca del nuevo portal para desarrolladores:

- [Acceso a la versión administrada del portal para desarrolladores y su personalización](api-management-howto-developer-portal-customize.md)
- [Configuración de la versión autohospedada del portal](developer-portal-self-host.md)
- [Implementación de su propio widget](developer-portal-implement-widgets.md)

Examine otros recursos:

- [Repositorio de GitHub con el código fuente](https://github.com/Azure/api-management-developer-portal)
- [Preguntas frecuentes sobre el portal para desarrolladores](developer-portal-faq.md)
