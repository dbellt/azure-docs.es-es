---
title: Autenticación SSH con Azure Active Directory
description: Guía de arquitectura para lograr la integración de SSH con Azure Active Directory
services: active-directory
author: BarbaraSelden
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.subservice: fundamentals
ms.topic: conceptual
ms.date: 10/19/2020
ms.author: baselden
ms.reviewer: ajburnle
ms.custom: it-pro, seodec18
ms.collection: M365-identity-device-management
ms.openlocfilehash: 315fe35a79ade39de9f541504fc2fe52754614de
ms.sourcegitcommit: 6686a3d8d8b7c8a582d6c40b60232a33798067be
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107749355"
---
# <a name="ssh"></a>SSH  

Secure Shell (SSH) es un protocolo de red que proporciona cifrado para operar los servicios de red de forma segura en una red no segura. SSH también proporciona un inicio de sesión de la línea de comandos, ejecuta comandos remotos y transfiere archivos de forma segura. Se usa normalmente en sistemas basados en UNIX como Linux®. SSH reemplaza al protocolo Telnet, que no proporciona cifrado en una red no segura. 

Azure Active Directory (Azure AD) proporciona una extensión de máquina virtual para los sistemas basados en Linux® que se ejecutan en Azure. 

## <a name="use-when"></a>Cuándo se utiliza 

* Trabajo con máquinas virtuales basadas en Linux® que requieren inicio de sesión remoto

* Ejecución de comandos remotos en sistemas basados en Linux®

* Transferencia de archivos de forma segura en una red no segura

![Diagrama de Azure AD con el protocolo SSH](./media/authentication-patterns/ssh-auth.png)

SSH con Azure AD

## <a name="components-of-system"></a>Componentes del sistema 

* **Usuario**: inicia el cliente SSH para configurar una conexión con las máquinas virtuales Linux® y proporciona las credenciales para la autenticación.

* **Explorador web**: Componente con el que interactúa el usuario. Se comunica con el proveedor de identidades (Azure AD) para autenticar y autorizar de forma segura al usuario.

* **Cliente SSH**: controla el proceso de configuración de la conexión.

* **Azure AD**: autentica la identidad del usuario mediante el flujo de dispositivo y emite un token para las máquinas virtuales Linux.

* **Máquina virtual Linux**: acepta el token y proporciona una conexión correcta.

## <a name="implement-ssh-with-azure-ad"></a>Implementación de SSH con Azure AD 

* [Inicio de sesión en una máquina virtual Linux en Azure mediante la autenticación de Azure Active Directory](../../virtual-machines/linux/login-using-aad.md) 

* [Flujo de concesión de autorización de dispositivo de OAuth 2.0 y la Plataforma de identidad de Microsoft](../develop/v2-oauth2-device-code.md)

* [Integración con Azure Active Directory (akamai.com)](https://learn.akamai.com/en-us/webhelp/enterprise-application-access/enterprise-application-access/GUID-6B16172C-86CC-48E8-B30D-8E678BF3325F.html)

