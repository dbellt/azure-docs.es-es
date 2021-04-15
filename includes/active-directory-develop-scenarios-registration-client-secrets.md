---
title: archivo de inclusión
description: archivo de inclusión para las páginas de aterrizaje del escenario de cliente confidencial (demonio, aplicación web, API web)
services: active-directory
documentationcenter: dev-center-name
author: jmprieur
manager: CelesteDG
ms.service: active-directory
ms.devlang: na
ms.topic: include
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 08/14/2020
ms.author: jmprieur
ms.openlocfilehash: 42102f38959911388cefcc141d949e59f24a2c31
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "95996020"
---
## <a name="add-a-client-secret-or-certificate"></a>Incorporación del secreto de cliente o certificado

Al igual que con cualquier aplicación cliente confidencial, debe agregar un secreto o certificado para que actúe como *credenciales* de la aplicación para que pueda autenticarse por sí misma, sin la interacción del usuario.

Puede agregar credenciales al registro de la aplicación cliente mediante [Azure Portal](#add-client-credentials-by-using-the-azure-portal) o mediante una herramienta de línea de comandos como [PowerShell](#add-client-credentials-by-using-powershell).

### <a name="add-client-credentials-by-using-the-azure-portal"></a>Incorporación de credenciales de cliente mediante Azure Portal

Para agregar credenciales al registro de aplicaciones de la aplicación cliente confidencial, siga los pasos descritos en la [Guía de inicio rápido: Registro de una aplicación en la plataforma de identidad de Microsoft](../articles/active-directory/develop/quickstart-register-app.md) correspondiente al tipo de credencial que quiere agregar:

* [Agregar un secreto de cliente](../articles/active-directory/develop/quickstart-register-app.md#add-a-client-secret)
* [Incorporación de un certificado](../articles/active-directory/develop/quickstart-register-app.md#add-a-certificate)

### <a name="add-client-credentials-by-using-powershell"></a>Incorporación de credenciales de cliente mediante PowerShell

También puede agregar credenciales al registrar la aplicación con la plataforma de identidad de Microsoft mediante PowerShell.

El ejemplo de código [active-directory-dotnetcore-daemon-v2](https://github.com/Azure-Samples/active-directory-dotnetcore-daemon-v2) en GitHub muestra cómo agregar un certificado o un secreto de aplicación al registrar una aplicación:

- Para más información sobre cómo agregar un **secreto de cliente** con PowerShell, consulte [AppCreationScripts/Configure.ps1](https://github.com/Azure-Samples/active-directory-dotnetcore-daemon-v2/blob/5199032b352a912e7cc0fce143f81664ba1a8c26/AppCreationScripts/Configure.ps1#L190).
- Para más información sobre cómo agregar un **certificado** con PowerShell, consulte [AppCreationScripts-withCert/Configure.ps1](https://github.com/Azure-Samples/active-directory-dotnetcore-daemon-v2/blob/5199032b352a912e7cc0fce143f81664ba1a8c26/AppCreationScripts-withCert/Configure.ps1#L162-L178).
