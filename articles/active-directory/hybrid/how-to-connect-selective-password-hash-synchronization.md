---
title: Sincronización selectiva de hash de contraseñas para Azure AD Connect
description: En este artículo se describe cómo instalar y configurar la sincronización selectiva de hash de contraseñas para usarla con Azure AD Connect.
services: active-directory
author: billmath
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 03/16/2021
ms.subservice: hybrid
ms.author: billmath
ms.reviewer: ''
ms.collection: M365-identity-device-management
ms.openlocfilehash: a8ab21b884e0842f22118efd892786cd5af9559b
ms.sourcegitcommit: 62e800ec1306c45e2d8310c40da5873f7945c657
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108161436"
---
# <a name="selective-password-hash-synchronization-configuration-for-azure-ad-connect"></a>Configuración de la sincronización selectiva de hash de contraseñas para Azure AD Connect

[La sincronización de hash de contraseñas](whatis-phs.md) es uno de los métodos de inicio de sesión utilizados para realizar la identidad híbrida. Azure AD Connect sincroniza un hash, del hash, de la contraseña de un usuario desde una instancia de Active Directory local con una instancia de Azure AD basada en la nube.  De forma predeterminada, una vez que se ha configurado, la sincronización de hash de contraseñas se realizará en todos los usuarios que se van a sincronizar.

Si quiere excluir a un subconjunto de usuarios de la sincronización de su hash de contraseñas con Azure AD, puede configurar la sincronización selectiva del hash de contraseñas siguiendo los pasos indicados en este artículo.

> [!IMPORTANT]
> Microsoft no admite la modificación ni el funcionamiento de la sincronización de Azure AD Connect con configuraciones o acciones distintas a estas que se documentan formalmente. Cualquiera de estas configuraciones o acciones podría dar lugar a un estado incoherente o no compatible con la sincronización de Azure AD Connect. Como resultado, Microsoft no puede garantizar que podamos proporcionar un soporte técnico eficiente para tales implementaciones.

## <a name="consider-your-implementation"></a>Consideración de su implementación

Para reducir el esfuerzo administrativo de configuración, primero debe tener en cuenta el número de objetos de usuario que desea excluir de la sincronización de hash de contraseñas. Compruebe cuál de los escenarios siguientes, que se excluyen mutuamente, se ajusta a sus necesidades para seleccionar la opción de configuración adecuada.
- Si el número de usuarios que se van a **excluir** es **menor** que el número de usuarios que se van a **incluir**, siga los pasos de esta [sección](#excluded-users-is-smaller-than-included-users).
- Si el número de usuarios que se van a **excluir** es **mayor** que el número de usuarios que se van a **incluir**, siga los pasos de esta [sección](#excluded-users-is-larger-than-included-users).

> [!IMPORTANT]
> Con cualquier opción de configuración elegida, se realizará automáticamente una sincronización inicial necesaria (sincronización completa) para aplicar los cambios en el siguiente ciclo de sincronización.

> [!IMPORTANT]
> La configuración de la sincronización de hash de contraseñas selectiva influye directamente en la escritura diferida de contraseñas. Los cambios o restablecimientos de contraseña que se inician en Azure Active Directory reescriben en la instancia local de Active Directory solo si el usuario está en el ámbito de la sincronización de hash de contraseñas.

### <a name="the-admindescription-attribute"></a>El atributo adminDescription

Ambos escenarios se basan en establecer el atributo adminDescription de los usuarios en un valor específico.  Esto permite aplicar las reglas y es lo que hace que funcione la sincronización selectiva de hash de contraseñas.

|Escenario|valor adminDescription|
|-----|-----|
|El número de usuarios excluidos es menor que el de usuarios incluidos.|PHSFiltered|
|El número de usuarios excluidos es mayor que el de usuarios incluidos.|PHSIncluded|

Este atributo se puede establecer mediante una de estas opciones:

- con la interfaz de usuario de Usuarios y equipos de Active Directory.
- con el cmdlet de PowerShell `Set-ADUser`.  Para obtener más información, consulte [Set-ADUser](/powershell/module/activedirectory/set-aduser).

### <a name="disable-the-synchronization-scheduler"></a>Deshabilitación del programador de sincronización

Antes de iniciar cualquiera de los dos escenarios, debe desactivar el programador de sincronización mientras realiza cambios en las reglas de sincronización.
 1. Inicie Windows PowerShell y escriba:

     `set-adsyncscheduler-synccycleenabled$false`

2. Confirme que el programador está deshabilitado mediante la ejecución del siguiente cmdlet:

    `get-adsyncscheduler`

Para más información sobre el programador de sincronización, consulte [Programador de sincronización de Azure AD Connect](how-to-connect-sync-feature-scheduler.md).

## <a name="excluded-users-is-smaller-than-included-users"></a>El número de usuarios excluidos es menor que el de usuarios incluidos.

En la siguiente sección se describe cómo habilitar la sincronización selectiva de hash de contraseña cuando el número de usuarios que se van a **excluir** es **menor** que el número de usuarios que se van a **incluir**.

> [!IMPORTANT]
> Antes de continuar, asegúrese de que el programador de sincronización está deshabilitado, tal y como se ha descrito anteriormente.

- Cree una copia modificable de **In from AD – User AccountEnabled** con la opción para **habilitar la sincronización de hash de contraseñas no seleccionada** y defina su filtro de ámbito.
- Cree otra copia modificable del valor predeterminado **In from AD – User AccountEnabled** con la opción para **habilitar la sincronización de hash de contraseñas seleccionada** y defina su filtro de ámbito.
- Nueva habilitación del programador de sincronización
- Establezca el valor del atributo, en Active Directory, que se definió como atributo de ámbito en los usuarios a los que desea permitir la sincronización de hash de contraseñas.

> [!IMPORTANT]
> Los pasos proporcionados para configurar la sincronización selectiva de hash de contraseñas solo afectarán a los objetos de usuario que tengan el atributo **adminDescription** rellenado en Active Directory con el valor de **PHSFiltered**.
Si no se rellena este atributo o el valor es distinto de **PHSFiltered**, estas reglas no se aplicarán a los objetos de usuario.

### <a name="configure-the-necessary-synchronization-rules"></a>Configuración de las reglas de sincronización necesarias

 1. Inicie el editor de reglas de sincronización y establezca los filtros **Sincronización de contraseñas** en **Activado** y **Tipo de regla** en **Estándar**.
     ![Inicio del editor de regas de sincronización](media/how-to-connect-selective-password-hash-synchronization/exclude-1.png)
 2. Seleccione la regla **In from AD – User AccountEnabled** para el conector del bosque de Active Directory en el que desea configurar la sincronización selectiva de contraseñas y haga clic en **Editar**. Seleccione **Sí** en el siguiente cuadro de diálogo para crear una copia modificable de la regla original.
     ![Selección de regla](media/how-to-connect-selective-password-hash-synchronization/exclude-2.png)
 3. La primera regla deshabilitará la sincronización de hash de contraseñas. Proporcione el siguiente nombre a la nueva regla personalizada: **In from AD - User AccountEnabled - Filter Users from PHS**.
 Cambie el valor de precedencia a un número inferior a 100 (por ejemplo, **90** o el valor más bajo disponible en su entorno).
 Asegúrese de que las casillas **Habilitar sincronización de contraseña** y **Deshabilitado** estén desactivadas.
 Haga clic en **Next**.
  ![Edición de entrada](media/how-to-connect-selective-password-hash-synchronization/exclude-3.png)
 4. En **Filtro de ámbito**, haga clic en **Agregar cláusula**.
 Seleccione **adminDescription** en la columna de atributo y **EQUAL** en la columna operador y escriba **PHSFiltered** como valor.
     ![Filtro de ámbito](media/how-to-connect-selective-password-hash-synchronization/exclude-4.png)
 5. No se requiere ningún cambio más. **Reglas de unión** y **Transformaciones** deben dejarse con los valores de configuración predeterminados copiados. Luego, haga clic en **Guardar**.
 Haga clic en **Aceptar** en el cuadro de diálogo de advertencia que informa de que se ejecutará una sincronización completa en el siguiente ciclo de sincronización del conector.
     ![Guardado de regla](media/how-to-connect-selective-password-hash-synchronization/exclude-5.png)
 6. A continuación, cree otra regla personalizada con la sincronización de hash de contraseñas habilitada. Seleccione de nuevo la regla predeterminada **In from AD – User AccountEnabled** para el bosque de Active Directory en el que desea configurar la sincronización selectiva de contraseñas y haga clic en **Editar**. Seleccione **Sí** en el siguiente cuadro de diálogo para crear una copia modificable de la regla original.
     ![Regla personalizada](media/how-to-connect-selective-password-hash-synchronization/exclude-6.png)
 7. Proporcione el siguiente nombre a la nueva regla personalizada: **In from AD - User AccountEnabled - Users included for PHS**.
 Cambie el valor de precedencia a un número inferior a la regla creada anteriormente (en este ejemplo, será **89**).
 Asegúrese de que la casilla **Habilitar sincronización de contraseña** está activada y que la casilla **Deshabilitado** está desactivada.
 Haga clic en **Next**.  
     ![Edición de regla nueva](media/how-to-connect-selective-password-hash-synchronization/exclude-7.png)
 8. En **Filtro de ámbito**, haga clic en **Agregar cláusula**.
 Seleccione **adminDescription** en la columna de atributo y **NOTEQUAL** en la columna operador y escriba **PHSFiltered** como valor.
     ![](media/how-to-connect-selective-password-hash-synchronization/exclude-8.png)
 9. No se requiere ningún cambio más. **Reglas de unión** y **Transformaciones** deben dejarse con los valores de configuración predeterminados copiados. Luego, haga clic en **Guardar**.
 Haga clic en **Aceptar** en el cuadro de diálogo de advertencia que informa de que se ejecutará una sincronización completa en el siguiente ciclo de sincronización del conector.
     ![Reglas de unión](media/how-to-connect-selective-password-hash-synchronization/exclude-9.png)
 10. Confirme la creación de las reglas. Quite los filtros **Sincronización de contraseña** **Activo** y **Tipo de regla** **Estándar**. Y debería ver las reglas nuevas que acaba de crear.
     ![Confirmación de reglas](media/how-to-connect-selective-password-hash-synchronization/exclude-10.png)

### <a name="re-enable-synchronization-scheduler"></a>Rehabilitación del programador de sincronización

Una vez que haya completado los pasos para configurar las reglas de sincronización necesarias, vuelva a habilitar el programador de sincronización con los pasos siguientes:
 1. En Windows PowerShell, ejecute:

     `set-adsyncscheduler-synccycleenabled$true`

 2. Después, confirme que se ha habilitado correctamente mediante la ejecución de:

     `get-adsyncscheduler`

Para más información sobre el programador de sincronización, consulte [Programador de sincronización de Azure AD Connect](how-to-connect-sync-feature-scheduler.md).

### <a name="edit-users-admindescription-attribute"></a>Edición del atributo **adminDescription** de los usuarios

Una vez completadas todas las configuraciones, debe editar el atributo **adminDescription** para todos los usuarios que desee **excluir** de la sincronización de hash de contraseñas en Active Directory y agregar la cadena usada en el filtro de ámbito: **PHSFiltered**.

  ![Edición de atributo](media/how-to-connect-selective-password-hash-synchronization/exclude-11.png)

También puede usar el comando de PowerShell siguiente para editar el atributo **adminDescription** de un usuario:

`set-adusermyuser-replace@{adminDescription="PHSFiltered"}`

## <a name="excluded-users-is-larger-than-included-users"></a>El número de usuarios excluidos es mayor que el de usuarios incluidos.

En la siguiente sección se describe cómo habilitar la sincronización selectiva de hash de contraseña cuando el número de usuarios que se van a **excluir** es **mayor** que el número de usuarios que se van a **incluir**.

> [!IMPORTANT]
> Antes de continuar, asegúrese de que el programador de sincronización está deshabilitado, tal y como se ha descrito anteriormente.

A continuación se muestra un resumen de las acciones que se realizarán en los pasos siguientes:

- Cree una copia modificable de **In from AD – User AccountEnabled** con la opción para **habilitar la sincronización de hash de contraseñas no seleccionada** y defina su filtro de ámbito.
- Cree otra copia modificable del valor predeterminado **In from AD – User AccountEnabled** con la opción para **habilitar la sincronización de hash de contraseñas seleccionada** y defina su filtro de ámbito.
- Nueva habilitación del programador de sincronización
- Establezca el valor del atributo, en Active Directory, que se definió como atributo de ámbito en los usuarios a los que desea permitir la sincronización de hash de contraseñas.

> [!IMPORTANT]
> Los pasos proporcionados para configurar la sincronización selectiva de hash de contraseñas solo afectarán a los objetos de usuario que tengan el atributo **adminDescription** rellenado en Active Directory con el valor de **PHSIncluded**.
Si no se rellena este atributo o el valor es distinto de **PHSIncluded**, estas reglas no se aplicarán a los objetos de usuario.

### <a name="configure-the-necessary-synchronization-rules"></a>Configuración de las reglas de sincronización necesarias

 1. Inicie el editor de reglas de sincronización y establezca los filtros **Sincronización de contraseñas** en **Activado** y **Tipo de regla** en **Estándar**.
     ![Tipo de regla](media/how-to-connect-selective-password-hash-synchronization/include-1.png)
 2. Seleccione la regla **In from AD – User AccountEnabled** para el bosque de Active Directory en el que desea configurar la sincronización selectiva de contraseñas y haga clic en **Editar**. Seleccione **Sí** en el siguiente cuadro de diálogo para crear una copia modificable de la regla original.
     ![In from AD](media/how-to-connect-selective-password-hash-synchronization/include-2.png)
 3. La primera regla deshabilitará la sincronización de hash de contraseñas. Proporcione el siguiente nombre a la nueva regla personalizada: **In from AD - User AccountEnabled - Filter Users from PHS**.
 Cambie el valor de precedencia a un número inferior a 100 (por ejemplo, **90** o el valor más bajo disponible en su entorno).
 Asegúrese de que las casillas **Habilitar sincronización de contraseña** y **Deshabilitado** estén desactivadas.
 Haga clic en **Next**.
  ![Establecimiento de precedencia](media/how-to-connect-selective-password-hash-synchronization/include-3.png)
 4. En **Filtro de ámbito**, haga clic en **Agregar cláusula**.
Seleccione **adminDescription** en la columna de atributo y **NOTEQUAL** en la columna operador y escriba **PHSIncluded** como valor.
     ![Adición de cláusula](media/how-to-connect-selective-password-hash-synchronization/include-4.png)
 5. No se requiere ningún cambio más. **Reglas de unión** y **Transformaciones** deben dejarse con los valores de configuración predeterminados copiados. Luego, haga clic en **Guardar**.
 Haga clic en **Aceptar** en el cuadro de diálogo de advertencia que informa de que se ejecutará una sincronización completa en el siguiente ciclo de sincronización del conector.
     ![Transformación](media/how-to-connect-selective-password-hash-synchronization/include-5.png)
 6. A continuación, cree otra regla personalizada con la sincronización de hash de contraseñas habilitada. Seleccione de nuevo la regla predeterminada **In from AD – User AccountEnabled** para el bosque de Active Directory en el que desea configurar la sincronización selectiva de contraseñas y haga clic en **Editar**. Seleccione **Sí** en el siguiente cuadro de diálogo para crear una copia modificable de la regla original.
     ![User AccountEnabled](media/how-to-connect-selective-password-hash-synchronization/include-6.png)
 7. Proporcione el siguiente nombre a la nueva regla personalizada: **In from AD - User AccountEnabled - Users included for PHS**.
 Cambie el valor de precedencia a un número inferior a la regla creada anteriormente (en este ejemplo, será **89**).
 Asegúrese de que la casilla **Habilitar sincronización de contraseña** está activada y que la casilla **Deshabilitado** está desactivada.
 Haga clic en **Next**.
     ![Habilitación de la sincronización de contraseñas](media/how-to-connect-selective-password-hash-synchronization/include-7.png)
 8. En **Filtro de ámbito**, haga clic en **Agregar cláusula**.
 Seleccione **adminDescription** en la columna de atributo y **EQUAL** en la columna operador y escriba **PHSIncluded** como valor.
     ![PHSIncluded](media/how-to-connect-selective-password-hash-synchronization/include-8.png)
 9. No se requiere ningún cambio más. **Reglas de unión** y **Transformaciones** deben dejarse con los valores de configuración predeterminados copiados. Luego, haga clic en **Guardar**.
 Haga clic en **Aceptar** en el cuadro de diálogo de advertencia que informa de que se ejecutará una sincronización completa en el siguiente ciclo de sincronización del conector.
     ![Guardado ahora](media/how-to-connect-selective-password-hash-synchronization/include-9.png)
 10. Confirme la creación de las reglas. Quite los filtros **Sincronización de contraseña** **Activo** y **Tipo de regla** **Estándar**. Y debería ver las reglas nuevas que acaba de crear.
     ![Sincronización activa](media/how-to-connect-selective-password-hash-synchronization/include-10.png)

### <a name="re-enable-synchronization-scheduler"></a>Rehabilitación del programador de sincronización

Una vez que haya completado los pasos para configurar las reglas de sincronización necesarias, vuelva a habilitar el programador de sincronización con los pasos siguientes:

1. En Windows PowerShell, ejecute:

   `set-adsyncscheduler-synccycleenabled$true`

2. Después, confirme que se ha habilitado correctamente mediante la ejecución de:

   `get-adsyncscheduler`

Para más información sobre el programador de sincronización, consulte [Programador de sincronización de Azure AD Connect](how-to-connect-sync-feature-scheduler.md).

### <a name="edit-users-admindescription-attribute"></a>Edición del atributo **adminDescription** de los usuarios

Una vez completadas todas las configuraciones, debe editar el atributo **adminDescription** para todos los usuarios que desee **incluir** para la sincronización de hash de contraseñas en Active Directory y agregar la cadena usada en el filtro de ámbito: **PHSIncluded**.

  ![Edición de atributos](media/how-to-connect-selective-password-hash-synchronization/include-11.png)

También puede usar el comando de PowerShell siguiente para editar el atributo **adminDescription** de un usuario:

`Set-ADUser myuser -Replace @{adminDescription="PHSIncluded"}`

## <a name="next-steps"></a>Pasos siguientes

- [¿Qué es la sincronización de hash de contraseñas?](whatis-phs.md)
- [Funcionamiento de la sincronización de hash de contraseñas](how-to-connect-password-hash-synchronization.md)