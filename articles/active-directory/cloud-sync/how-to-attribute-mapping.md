---
title: Asignación de atributos de sincronización en la nube de Azure AD Connect
description: En este artículo se describe cómo usar la característica de sincronización en la nube de Azure AD Connect para asignar atributos.
services: active-directory
author: billmath
manager: mtillman
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 04/30/2021
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: e861a1adc2dead2ee7c4397b9fb09aae202aaf94
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111963697"
---
# <a name="attribute-mapping-in-azure-ad-connect-cloud-sync"></a>Asignación de atributos de sincronización en la nube de Azure AD Connect

Puede usar la característica de sincronización en la nube de Azure Active Directory (Azure AD) Connect para asignar atributos entre los objetos de usuario o de grupo locales y los objetos de Azure AD. Esta capacidad se ha agregado a la configuración de sincronización en la nube.

Puede personalizar (cambiar, eliminar o crear) las asignaciones de atributos predeterminadas según sus necesidades empresariales. Para obtener una lista de los atributos que se sincronizan, consulte [Atributos sincronizados con Azure Active Directory](../hybrid/reference-connect-sync-attributes-synchronized.md?context=azure%2factive-directory%2fcloud-provisioning%2fcontext%2fcp-context/hybrid/reference-connect-sync-attributes-synchronized.md).

> [!NOTE]
> En este artículo se describe cómo usar Azure Portal para asignar atributos.  Para obtener información sobre el uso de Microsoft Graph, consulte [Transformaciones](how-to-transformation.md).

## <a name="understand-types-of-attribute-mapping"></a>Descripción de los tipos de asignación de atributos
Con la asignación de atributos, puede controlar cómo se rellenan los atributos en Azure AD. Azure AD admite cuatro tipos de asignación:

|Tipo de asignación|Descripción|
|-----|-----|
|**Direct**|El atributo de destino se rellena con el valor de un atributo del objeto vinculado en Active Directory.|
|**Constante**|El atributo de destino se rellena con una cadena específica que se ha indicado.|
|**Expression**|El atributo de destino se rellena según el resultado de una expresión similar a un script. Para más información, consulte [Generador de expresiones](how-to-expression-builder.md) y [Escritura de expresiones para la asignación de atributos en Azure Active Directory](reference-expressions.md).|
|**Ninguno**|El atributo de destino se deja sin modificar. Sin embargo, si el atributo de destino está vacío, se rellena con el valor predeterminado que especifique.|

Además de estos tipos básicos, las asignaciones de atributos personalizadas admiten el concepto de una asignación de valor *predeterminada* opcional. La asignación de valor predeterminada garantiza que un atributo de destino se rellene con un valor si no hay ningún valor en Azure AD ni en el objeto de destino. La configuración más habitual consiste en dejarlo en blanco.

## <a name="schema-updates-and-mappings"></a>Actualizaciones y asignaciones de esquemas
La sincronización en la nube actualizará ocasionalmente el esquema y la lista de atributos predeterminados que se[sincronizan.](../hybrid/reference-connect-sync-attributes-synchronized.md?context=%2fazure%2factive-directory%2fcloud-provisioning%2fcontext%2fcp-context)  Estas asignaciones de atributos predeterminados estarán disponibles para las nuevas instalaciones, pero no se agregarán automáticamente a las instalaciones existentes.  Para agregar estas asignaciones, puede seguir los pasos que se indican a continuación.


  1. Haga clic en "Agregar asignación de atributos".
  2. Seleccione la lista desplegable Atributo de destino.
  3. Debería ver los nuevos atributos que están disponibles aquí.

A continuación se muestra una lista de nuevas asignaciones que se han agregado.

Atributo agregado | Tipo de asignación | Agregado con la versión del agente
| ----- | -----| -----|
|preferredDatalocation|Directo|1.1.359.0|
|EmployeeNumber|Directo|1.1.359.0|
|UserType|Directo|1.1.359.0|

Para obtener más información sobre cómo asignar UserType, consulte [Asignación de UserType con sincronización en la nube](how-to-map-usertype.md).

## <a name="understand-properties-of-attribute-mappings"></a>Descripción de las propiedades de asignaciones de atributos

Además de la propiedad de tipo, las asignaciones de atributos admiten determinados atributos.  Estos atributos dependerán del tipo de asignación que haya seleccionado.  En las secciones siguientes se describen las asignaciones de atributos admitidas para cada uno de los tipos individuales.

### <a name="direct-mapping-attributes"></a>Atributos de asignación directa
Estos son los atributos admitidos por una asignación directa:

- **Atributo de origen**: especifica el atributo de usuario del sistema de origen (por ejemplo, Active Directory).
- **Atributo de destino**: especifica el atributo de usuario del sistema de destino (por ejemplo, Azure Active Directory).
- **Valor predeterminado si es nulo (opcional)** : valor que se enviará al sistema de destino si el atributo de origen es NULL. Este valor solo se aprovisionará cuando se cree un usuario. No se aprovisionará al actualizar un usuario existente.  
- **Aplicar esta asignación**:
  - **Siempre**: esta asignación se aplica a las acciones de creación y actualización de usuarios.
  - **Solo durante la creación**: esta asignación se aplica solo a las acciones de creación de usuarios.

 ![Captura de pantalla de asignación directa](media/how-to-attribute-mapping/mapping-7.png)

### <a name="constant-mapping-attributes"></a>Atributos de asignación de constantes
Estos son los atributos admitidos por una asignación de constantes:

- **Valor constante**: el valor que desea aplicar al atributo de destino.
- **Atributo de destino**: especifica el atributo de usuario del sistema de destino (por ejemplo, Azure Active Directory).
- **Aplicar esta asignación**:
  - **Siempre**: esta asignación se aplica a las acciones de creación y actualización de usuarios.
  - **Solo durante la creación**: esta asignación se aplica solo a las acciones de creación de usuarios.

 ![Captura de pantalla de la constante](media/how-to-attribute-mapping/mapping-9.png)

### <a name="expression-mapping-attributes"></a>Atributos de asignación de expresiones
Estos son los atributos admitidos por una asignación de expresiones:

- **Expresión**: la expresión que se va a aplicar al atributo de destino.  Para más información, consulte [Generador de expresiones](how-to-expression-builder.md) y [Escritura de expresiones para la asignación de atributos en Azure Active Directory](reference-expressions.md).
-  **Valor predeterminado si es nulo (opcional)** : valor que se enviará al sistema de destino si el atributo de origen es NULL. Este valor solo se aprovisionará cuando se cree un usuario. No se aprovisionará al actualizar un usuario existente. 
- **Atributo de destino**: especifica el atributo de usuario del sistema de destino (por ejemplo, Azure Active Directory).
 
- **Aplicar esta asignación**:
  - **Siempre**: esta asignación se aplica a las acciones de creación y actualización de usuarios.
  - **Solo durante la creación**: esta asignación se aplica solo a las acciones de creación de usuarios.

 ![Captura de pantalla de la asignación de expresión](media/how-to-attribute-mapping/mapping-10.png)

## <a name="add-an-attribute-mapping"></a>Adición de una asignación de atributos

Para usar la nueva capacidad, siga estos pasos:

1.  En Azure Portal, seleccione **Azure Active Directory**.
2.  Seleccione **Azure AD Connect**.
3.  Seleccione **Manage cloud sync** (Administrar sincronización en la nube).

    ![Captura de pantalla que muestra el vínculo para administrar la sincronización en la nube.](media/how-to-install/install-6.png)

4. En **Configuración**, seleccione su configuración.
5. Seleccione **Click to edit mappings** (Haga clic para editar las asignaciones).  Este vínculo abre la pantalla **Asignaciones de atributos**.

    ![Captura de pantalla que muestra el vínculo para agregar atributos.](media/how-to-attribute-mapping/mapping-6.png)

6.  Seleccione **Agregar atributo**.

    ![Captura de pantalla que muestra el botón para agregar un atributo, junto con listas de atributos y tipos de asignación.](media/how-to-attribute-mapping/mapping-1.png)

7. Seleccione el tipo de asignación. Este puede ser uno de los siguientes:
     - **Directa**: el atributo de destino se rellena con el valor de un atributo del objeto vinculado en Active Directory.
     - **Constante**: el atributo de destino se rellena con una cadena específica que se ha especificado.
     - **Expresión**: el atributo de destino se rellena según el resultado de una expresión similar a un script. 
     - **Ninguna**: el atributo de destino se deja sin modificar. 
 
     Para obtener más información, consulte la sección [Descripción de los tipos de asignación de atributos](#understand-types-of-attribute-mapping) anterior.
8. En función de lo que haya seleccionado en el paso anterior, habrá diferentes opciones disponibles para rellenar.  Consulte las secciones anteriores de [Descripción de las propiedades de asignaciones de atributos](#understand-properties-of-attribute-mappings) para obtener información sobre estos atributos.
9. Seleccione cuándo quiere aplicar esta asignación y, a continuación, haga clic en **Aplicar**.
11. En la pantalla **Asignación de atributos**, debería ver la nueva asignación de atributos.  
12. Seleccione **Guardar esquema**.

    ![Captura de pantalla que muestra el botón Guardar esquema.](media/how-to-attribute-mapping/mapping-3.png)

## <a name="test-your-attribute-mapping"></a>Prueba de la asignación de atributos

Para probar la asignación de atributos, puede usar el [ aprovisionamiento a petición](how-to-on-demand-provision.md): 

1. En Azure Portal, seleccione **Azure Active Directory**.
2. Seleccione **Azure AD Connect**.
3. Seleccione **Administrar aprovisionamiento**.
4. En **Configuración**, seleccione su configuración.
5. En **Validar**, seleccione el botón **Aprovisionar un usuario**. 
6. En la pantalla **Aprovisionamiento a petición**, escriba el nombre distintivo de un usuario o grupo y seleccione el botón **Aprovisionar**. 

   La pantalla muestra que el aprovisionamiento está en curso.

   ![Captura de pantalla que muestra el aprovisionamiento en curso.](media/how-to-attribute-mapping/mapping-4.png)

8. Una vez finalizado el aprovisionamiento, aparece una pantalla de éxito con cuatro marcas de verificación verdes. 

   En **Realizar una acción**, seleccione **Ver detalles**. A la derecha, debería ver el nuevo atributo sincronizado y la expresión aplicada.

   ![Captura de pantalla que muestra los detalles de éxito y exportación.](media/how-to-attribute-mapping/mapping-5.png)






## <a name="next-steps"></a>Pasos siguientes

- [¿Qué es la sincronización en la nube de Azure AD Connect?](what-is-cloud-sync.md)
- [Escritura de expresiones para la asignación de atributos](reference-expressions.md)
- [Generador de expresiones con sincronización en la nube](how-to-expression-builder.md)
- [Atributos sincronizados con Azure Active Directory](../hybrid/reference-connect-sync-attributes-synchronized.md?context=azure%2factive-directory%2fcloud-provisioning%2fcontext%2fcp-context/hybrid/reference-connect-sync-attributes-synchronized.md)