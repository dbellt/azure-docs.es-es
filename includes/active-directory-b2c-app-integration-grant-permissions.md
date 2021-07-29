---
author: msmimart
ms.service: active-directory-b2c
ms.subservice: B2C
ms.topic: include
ms.date: 06/11/2021
ms.author: mimart
ms.openlocfilehash: 6d62dda341c6acb783bd2525c2f6ba7a76b312ec
ms.sourcegitcommit: 8651d19fca8c5f709cbb22bfcbe2fd4a1c8e429f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/14/2021
ms.locfileid: "112073073"
---
1. En **Administrar**, seleccione **Permisos de API**.
1. En **Permisos configurados**, seleccione **Agregar un permiso**.
1. Seleccione la pestaña **Mis API**.
1. Seleccione la API a la que la aplicación web debe tener acceso. Por ejemplo, *my-api1*.
1. En **Permiso**, expanda **Tareas** y, a continuación, seleccione los ámbitos que definió anteriormente. Por ejemplo, *tasks.read* y *tasks.write*.
1. Seleccione **Agregar permisos**.
1. Seleccione **Conceder consentimiento de administrador para (el nombre de inquilino)** .
1. Seleccione **Sí**.
1. Seleccione **Actualizar** y, a continuación, compruebe que aparece "Concedido para..." en **Estado** para ambos ámbitos.
1. En la lista **Permisos configurados**, seleccione el ámbito y copie el nombre completo del ámbito. 

    ![Obtener el ámbito de la aplicación web](./media/active-directory-b2c-app-integration-grant-permissions/get-azure-ad-b2c-app-permissions.png)  