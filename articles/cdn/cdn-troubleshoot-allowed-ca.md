---
title: Entidad de certificación permitida para habilitar HTTPS personalizado
titleSuffix: Azure Content Delivery Network
description: Si utiliza su propio certificado para habilitar HTTPS en un dominio personalizado, debe usar una entidad de certificación permitida (CA) para crearlo.
services: cdn
author: asudbring
ms.service: azure-cdn
ms.topic: troubleshooting
ms.date: 02/04/2021
ms.author: allensu
ms.custom: mvc
ms.openlocfilehash: f98e28c89fa70831108cfbbbaca6e2f316d1b039
ms.sourcegitcommit: e39ad7e8db27c97c8fb0d6afa322d4d135fd2066
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/11/2021
ms.locfileid: "112003633"
---
# <a name="allowed-certificate-authorities-for-enabling-custom-https"></a>Entidades de certificación permitidas para habilitar HTTPS personalizado

Al [habilitar la característica HTTPS mediante su propio certificado](cdn-custom-ssl.md?tabs=option-2-enable-https-with-your-own-certificate#tlsssl-certificates) para un dominio personalizado de Azure CDN (Content Delivery Network), se necesitan requisitos de certificado específicos. 

* El perfil **Azure CDN Estándar de Microsoft** requiere un certificado de una de las entidades de certificación (CA) aprobadas en la lista siguiente. Si se utiliza un certificado de una entidad de certificación no aprobada o si se usa un certificado autofirmado, la solicitud se rechazará. 

* Los perfiles **Azure CDN Estándar de Verizon** y **Azure CDN Prémium de Verizon** aceptan cualquier certificado válido de cualquier entidad de certificación válida. Los perfiles de Verizon no admiten certificados autofirmados.

> [!NOTE]
> La opción de usar su propio certificado para habilitar la característica HTTPS del dominio personalizado *no* está disponible para los perfiles **Azure CDN Estándar de Akamai**. 
>

[!INCLUDE [cdn-front-door-allowed-ca](../../includes/cdn-front-door-allowed-ca.md)]

