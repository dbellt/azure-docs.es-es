---
title: 'Configuración de la autenticación del suscriptor de webhook: Azure Event Grid IoT Edge | Microsoft Docs'
description: Configuración de la autenticación de suscriptores de webhook
author: VidyaKukke
manager: rajarv
ms.author: vkukke
ms.reviewer: spelluru
ms.subservice: iot-edge
ms.date: 05/10/2021
ms.topic: article
ms.openlocfilehash: 8855369516067c0cf05bab8be7128b628d61ce6b
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110370503"
---
# <a name="configure-webhook-subscriber-authentication"></a>Configuración de la autenticación de suscriptores de webhook

En esta guía se proporcionan ejemplos de las posibles configuraciones de suscriptores de webhook para un módulo de Event Grid. De forma predeterminada, solo se aceptan puntos de conexión HTTPS para los suscriptores de webhook. El módulo de Event Grid provocará un rechazo si el suscriptor presenta un certificado autofirmado.

## <a name="allow-only-https-subscriber"></a>Permitir solo suscriptor HTTPS

```json
 {
  "Env": [
    "outbound__webhook__httpsOnly=true",
    "outbound__webhook__skipServerCertValidation=false",
    "outbound__webhook__allowUnknownCA=false"
  ]
}
 ```

## <a name="allow-https-subscriber-with-self-signed-certificate"></a>Permitir suscriptor HTTPS con certificado autofirmado

```json
 {
  "Env": [
    "outbound__webhook__httpsOnly=true",
    "outbound__webhook__skipServerCertValidation=false",
    "outbound__webhook__allowUnknownCA=true"
  ]
}
 ```

>[!NOTE]
>Establezca la propiedad `outbound__webhook__allowUnknownCA` en `true` solo en entornos de prueba, ya que normalmente usará certificados autofirmados. En el caso de las cargas de trabajo de producción, se recomienda que se establezcan en **false**.

## <a name="allow-https-subscriber-but-skip-certificate-validation"></a>Permitir suscriptor HTTPS pero omitir la validación de certificados

```json
 {
  "Env": [
    "outbound__webhook__httpsOnly=true",
    "outbound__webhook__skipServerCertValidation=true",
    "outbound__webhook__allowUnknownCA=false"
  ]
}
 ```

>[!NOTE]
>Establezca la propiedad `outbound__webhook__skipServerCertValidation` en `true` solo en entornos de prueba, ya que es posible que no esté presentando un certificado que deba autenticarse. En el caso de las cargas de trabajo de producción, se recomienda que se establezcan en **false**.

## <a name="allow-both-http-and-https-with-self-signed-certificates"></a>Permitir HTTP y HTTPS con certificados autofirmados

```json
 {
  "Env": [
    "outbound__webhook__httpsOnly=false",
    "outbound__webhook__skipServerCertValidation=false",
    "outbound__webhook__allowUnknownCA=true"
  ]
}
 ```

>[!NOTE]
>Establezca la propiedad `outbound__webhook__httpsOnly` en `false` solo en entornos de prueba, ya que es posible que desee empezar con un suscriptor HTTP. En el caso de las cargas de trabajo de producción, se recomienda que se establezcan en **true**.
