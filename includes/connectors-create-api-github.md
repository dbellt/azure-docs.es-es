---
ms.service: logic-apps
ms.topic: include
author: ecfan
ms.author: estfan
ms.date: 03/02/2018
ms.openlocfilehash: 1aabc1b33722c571c6519b6620fb62211a44c2e8
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "95999556"
---
1. En [Azure Portal](https://portal.azure.com), cree una aplicación lógica en blanco. 

2. En el Diseñador de aplicaciones lógicas, escriba "github" como filtro. 

3. Seleccione el conector y el desencadenador de GitHub que desea usar.

   ![Selección del conector y el desencadenador de GitHub](./media/connectors-create-api-github/github-connector.png)

   > [!NOTE]
   > Todos los flujos de trabajo de aplicaciones lógicas deben comenzar con un desencadenador. Puede seleccionar acciones solo cuando el flujo de trabajo de lógica ya se inicia con un desencadenador. 

4. Si anteriormente no creó una conexión, elija **Iniciar sesión** para proporcionar sus credenciales de GitHub cuando se le soliciten.  

   ![Inicio de sesión con sus credenciales de GitHub](./media/connectors-create-api-github/github-connector-sign-in-credentials.png)

   La aplicación lógica usa estas credenciales para autorizar la conexión y el acceso a los datos a la cuenta de GitHub. 

5. Indique su nombre de usuario y contraseña de GitHub y luego confirme su autorización.

   ![Provisión de credenciales y confirmación de la autorización](./media/connectors-create-api-github/github-connector-authorize.png)   

   Ya se ha creado la conexión en Azure Portal y está lista para su uso.

6. Continúe definiendo el flujo de trabajo de aplicación lógica.

   ![Adición de más acciones al flujo de trabajo de la aplicación lógica](./media/connectors-create-api-github/github-connector-logic-app.png)

