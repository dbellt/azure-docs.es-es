---
title: Uso de la asignación de UserType con Azure AD Connect Cloud Sync
description: En este artículo se describe cómo usar la asignación del atributo UserType con Cloud Sync.
services: active-directory
author: billmath
manager: mtillman
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 05/04/2021
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: 14bbeb7339b3af41b24b25aed2332ac8367e99ae
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108776923"
---
# <a name="map-usertype-with-cloud-sync"></a>Asignación de UserType con Cloud Sync

Cloud Sync admite la sincronización del atributo UserType para objetos User. 

De forma predeterminada, el atributo UserType no está habilitado para la sincronización porque no hay ningún atributo UserType correspondiente en la instancia local de Active Directory. Debe agregar manualmente esta asignación para la sincronización. Antes de hacerlo, vea los comportamientos siguientes que exige Azure AD:

- Azure AD solo acepta dos valores para el atributo UserType: Miembro e Invitado.
- Si el atributo UserType no está asignado en Cloud Sync, los usuarios de Azure AD que se crearon mediante la sincronización de directorios tendrían el atributo UserType establecido en Miembro.

Antes de agregar una asignación del atributo UserType, primero debe decidir cómo se obtendrá el atributo UserType de Active Directory local. Los siguientes son los métodos más comunes:

 - Designar un atributo de AD local sin usar (p. ej., extensionAttribute1) que se utilizará como atributo de origen. El atributo de AD local designado debería ser de tipo cadena, tener un solo valor y contener el valor Miembro o el valor Invitado.
 - Si elige este enfoque, debe asegurarse de que el atributo designado se rellena con el valor correcto para todos los objetos User existentes en Active Directory local que se sincronizan con Azure AD antes de que se habilite la sincronización del atributo UserType.

## <a name="to-add-the-usertype-mapping"></a>Para agregar la asignación de UserType
Para agregar la asignación de UserType, siga estos pasos.

 1. En Azure Portal, seleccione **Azure Active Directory**.
 2. Seleccione **Azure AD Connect**.
 3. Seleccione **Manage cloud sync** (Administrar sincronización en la nube).
 4. En **Configuración**, seleccione su configuración.
 5. En **Administrar atributos**, seleccione **Haga clic para editar las asignaciones**.
  ![Edición de las asignaciones de atributos](media/how-to-map-usertype/usertype-1.png) 

 6. Haga clic en **Agregar asignación de atributos**.
    ![Adición de una nueva asignación de atributo](media/how-to-map-usertype/usertype-2.png) 
7. Seleccione el tipo de asignación. Puede realizar la asignación de una de estas tres maneras:
 - una asignación directa (es el caso de un atributo de AD)
 - una expresión (IIF(InStr([userPrincipalName], " @partners ") > 0,"Guest","Member"))
 - una constante (es decir, convertir todos los objetos de usuario en Invitado).
  ![Agregar usertype](media/how-to-map-usertype/usertype-3.png) 
8. En el menú desplegable Atributo de destino, seleccione UserType.
9. Haga clic en el botón **Aplicar** en la parte inferior de la página. Esto creará una asignación para el atributo UserType de Azure AD.

## <a name="next-steps"></a>Pasos siguientes 

- [Escritura de expresiones para la asignación de atributos en Azure Active Directory](reference-expressions.md)
- [Configuración de la sincronización en la nube](how-to-configure.md)
