---
title: 'Inicio rápido: Creación de un grupo de administración con JavaScript'
description: En este inicio rápido, se usa JavaScript para crear un grupo de administración para organizar los recursos en una jerarquía de recursos.
ms.date: 05/01/2021
ms.topic: quickstart
ms.custom:
- devx-track-js
- mode-api
ms.openlocfilehash: 537f78b31790251c322f8073ed2c887a430d161b
ms.sourcegitcommit: f6b76df4c22f1c605682418f3f2385131512508d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/30/2021
ms.locfileid: "108323256"
---
# <a name="quickstart-create-a-management-group-with-javascript"></a>Inicio rápido: Creación de un grupo de administración con JavaScript

Los grupos de administración son contenedores que ayudan a administran el acceso, las directivas y el cumplimiento de varias suscripciones. Cree estos contenedores para construir una jerarquía eficaz y eficiente que pueda usarse con [Azure Policy](../policy/overview.md) y los [controles de acceso basados en roles de Azure](../../role-based-access-control/overview.md). Para obtener más información sobre los grupos de administración, consulte [Organización de los recursos con grupos de administración de Azure](overview.md).

El primer grupo de administración creado en el directorio puede tardar hasta 15 minutos en completarse. Hay procesos que se ejecutan la primera vez para configurar el servicio de grupos de administración dentro de Azure para su directorio. Recibirá una notificación cuando finalice el proceso. Para obtener más información, vea [Instalación inicial de los grupos de administración](./overview.md#initial-setup-of-management-groups).

## <a name="prerequisites"></a>Requisitos previos

- Si no tiene una suscripción a Azure, cree una cuenta [gratuita](https://azure.microsoft.com/free/) antes de empezar.

- Antes de empezar, asegúrese de tener instalada la versión 12 o posterior de [Node.js](https://nodejs.org/).

- Si la [protección de jerarquías](./how-to/protect-resource-hierarchy.md#setting---require-authorization) no está habilitada, cualquier usuario de Azure AD del inquilino puede crear un grupo de administración sin tener el permiso de escritura del grupo de administración asignado. Este nuevo grupo de administración pasa a ser un elemento secundario del grupo de administración raíz o del [grupo de administración predeterminado](./how-to/protect-resource-hierarchy.md#setting---default-management-group) y al creador se le asigna el rol de "Propietario". El servicio del grupo de administración ofrece esta funcionalidad para que las asignaciones de roles no sean necesarias en el nivel raíz. Ningún usuario tiene acceso al grupo de administración raíz cuando se crea. Para sortear el obstáculo de buscar administradores globales de Azure AD para empezar a usar grupos de administración, se permite la creación de grupos de administración iniciales en el nivel raíz.

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

## <a name="application-setup"></a>Configuración de la aplicación

Para permitir que JavaScript administre grupos de administración, debe configurarse el entorno. Este programa de instalación funciona siempre que se pueda usar JavaScript, incluido [bash en Windows 10](/windows/wsl/install-win10).

1. Configure un nuevo proyecto de Node.js ejecutando el siguiente comando.

   ```bash
   npm init -y
   ```

1. Agregue una referencia al módulo yargs.

   ```bash
   npm install yargs
   ```

1. Agregue una referencia al módulo de Azure Resource Graph.

   ```bash
   npm install @azure/arm-managementgroups
   ```

1. Agregue una referencia a la biblioteca de autenticación de Azure.

   ```bash
   npm install @azure/ms-rest-nodeauth
   ```

   > [!NOTE]
   > Compruebe en _package.json_ que `@azure/arm-managementgroups` es de la versión **1.1.0** o posterior y `@azure/ms-rest-nodeauth` de la versión **3.0.5** o posterior.

## <a name="create-the-management-group"></a>Creación del grupo de administración

1. Cree un nuevo archivo denominado _index.js_ y escriba el código siguiente.

   ```javascript
   const argv = require("yargs").argv;
   const authenticator = require("@azure/ms-rest-nodeauth");
   const managementGroups = require("@azure/arm-managementgroups");

   if (argv.groupID && argv.displayName) {
       const createMG = async () => {
          const credentials = await authenticator.interactiveLogin();
          const client = new managementGroups.ManagementGroupsAPI(credentials);
          const result = await client.managementGroups.createOrUpdate(
             groupId: argv.groupID,
             {
                 displayName: argv.displayName
             }
          );
          console.log(result);
       };

       createMG();
   }
   ```

1. En el terminal, escriba el siguiente comando:

   ```bash
   node index.js --groupID "<NEW_MG_GROUP_ID>" --displayName "<NEW_MG_FRIENDLY_NAME>"
   ```

   Asegúrese de reemplazar todos los marcadores de posición `<>` de token por el _id. de grupo de administración_ y el _nombre descriptivo del grupo de administración_, respectivamente.

   A medida que el script intenta autenticarse, se muestra un mensaje similar al siguiente en el terminal:

   > Para iniciar sesión, use un explorador web para abrir la página https://microsoft.com/devicelogin y escriba el código FGB56WJUGK para autenticarse.

   Una vez que se autentica en el explorador, el script continúa ejecutándose.

El resultado de la creación del grupo de administración se envía a la consola.

## <a name="clean-up-resources"></a>Limpieza de recursos

Si desea quitar las bibliotecas instaladas de la aplicación, ejecute el siguiente comando.

```bash
npm uninstall @azure/arm-managementgroups @azure/ms-rest-nodeauth yargs
```

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha creado un grupo de administración para organizar una jerarquía de recursos. El grupo de administración puede contener suscripciones u otros grupos de administración.

Para más información acerca de los grupos de administración y cómo administrar la jerarquía de recursos, vaya a:

> [!div class="nextstepaction"]
> [Administración de los recursos con grupos de administración](./manage.md)
