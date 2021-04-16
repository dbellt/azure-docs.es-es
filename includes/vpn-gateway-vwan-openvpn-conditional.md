---
title: archivo de inclusión
description: archivo de inclusión
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: include
ms.date: 02/14/2020
ms.author: cherylmc
ms.custom: include file
ms.openlocfilehash: 410570302eec418f1e4bcb75d6413936a96b5171
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "77471582"
---
El acceso condicional permite un control de acceso específico por aplicación. Para usar el acceso condicional, debe tener licencias de Azure AD Premium 1 o superior aplicadas a los usuarios que estarán sujetos a las reglas de acceso condicional.

1. Vaya a la página **Aplicaciones empresariales: Todas las aplicaciones** y haga clic en **Azure VPN**.

   - Haga clic en **Acceso condicional**.
   - Haga clic en **Nueva directiva** para que se abra el panel **Nuevo**.
2. En el panel **Nuevo**, vaya a **Asignaciones > Usuarios y grupos**. En **Usuarios y grupos** pestaña **Incluir**:

   - Haga clic en **Seleccionar usuarios y grupos**.
   - Active la casilla **Usuarios y grupos**.
   - Haga clic en **Seleccionar** para seleccionar un grupo o un conjunto de usuarios que se verán afectado por MFA.
   - Haga clic en **Done**(Listo).

   ![Assignments](./media/vpn-gateway-vwan-openvpn-azure-ad-mfa/mfa-ca-assignments.png)
3. En el panel **Nuevo**, vaya al panel **Controles de acceso > Conceder**:

   - Haga clic en **Conceder acceso**.
   - Haga clic en **Requerir autenticación multifactor**.
   - Haga clic en **Requerir todos los controles seleccionados**.
   - Haga clic en **Seleccionar**.
   
   ![Conceder acceso: MFA](./media/vpn-gateway-vwan-openvpn-azure-ad-mfa/mfa-ca-grant-mfa.png)
4. En la sección **Habilitar directiva**:

   - Seleccione **Activado**.
   - Haga clic en **Crear**.

   ![Habilitar directiva](./media/vpn-gateway-vwan-openvpn-azure-ad-mfa/mfa-ca-enable-policy.png)