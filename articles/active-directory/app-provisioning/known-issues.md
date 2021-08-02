---
title: Problemas conocidos del aprovisionamiento de aplicaciones en Azure Active Directory
description: Obtenga información sobre los problemas conocidos al trabajar con el aprovisionamiento automático de aplicaciones en Azure Active Directory.
author: kenwith
ms.author: kenwith
manager: mtillman
services: active-directory
ms.service: active-directory
ms.subservice: app-provisioning
ms.workload: identity
ms.topic: troubleshooting
ms.date: 05/28/2021
ms.reviewer: arvinh
ms.openlocfilehash: 1674e3aae978c16b8ef736dc6605bd30e7201e10
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111962030"
---
# <a name="known-issues-for-application-provisioning-in-azure-active-directory"></a>Problemas conocidos del aprovisionamiento de aplicaciones en Azure Active Directory
Problemas conocidos que se deben tener en cuenta al trabajar con el aprovisionamiento de aplicaciones. Puede proporcionar comentarios sobre el servicio de aprovisionamiento de aplicaciones en UserVoice; vea [Aprovisionamiento de aplicaciones de Azure AD en UserVoice](https://aka.ms/appprovisioningfeaturerequest). Supervisamos UserVoice muy de cerca para poder mejorar el servicio. 

> [!NOTE]
> Esta no es una lista completa de problemas conocidos. Si sabe que hay un problema que no aparece en la lista, proporcione comentarios en la parte inferior de la página.

## <a name="authorization"></a>Authorization 

**No se puede guardar después de una prueba de conexión correcta**

Si puede probar correctamente una conexión, pero no puede guardar, ha superado el límite de almacenamiento permitido para las credenciales. Para más información, vea [Problema al guardar las credenciales del administrador](./user-provisioning.md).

**No se puede guardar**

La dirección URL del inquilino, el token secreto y el correo electrónico de notificación se deben rellenar para guardar. No se puede proporcionar solo uno de ellos. 

**No se puede volver a cambiar el modo de aprovisionamiento a manual**

Tras configurar el aprovisionamiento por primera vez, observará que el modo de aprovisionamiento cambia de manual a automático. No se puede volver a cambiar a manual. Sin embargo, puede desactivar el aprovisionamiento en la interfaz de usuario. Desactivar el aprovisionamiento en la interfaz de usuario tiene el mismo efecto que establecer la configuración en manual en el menú desplegable.  


## <a name="attribute-mappings"></a>Asignaciones de atributos 

**Atributo SamAccountName o userType no disponible como un atributo de origen**

Los atributos SamAccountName y userType no están disponibles como un atributo de origen de forma predeterminada. Extienda el esquema para agregar el atributo. Puede agregar los atributos a la lista de atributos de origen disponibles mediante la extensión del esquema. Para más información, vea [Falta el atributo de origen](user-provisioning-sync-attributes-for-mapping.md). 

**Faltan atributos de origen en la lista desplegable para la extensión del esquema**

A veces, pueden faltar extensiones en la lista desplegable de atributos de origen de la interfaz de usuario. Vaya a la configuración avanzada de las asignaciones de atributos y agregue manualmente los atributos. Para más información, vea [Personalización de asignaciones de atributos](customize-application-attributes.md).

**No se puede aprovisionar el atributo NULL**

Azure AD actualmente no puede aprovisionar atributos NULL. Si un atributo es NULL en el objeto de usuario, se omitirá. 

**Caracteres máximos para las expresiones de asignación de atributos**

Las expresiones de asignación de atributos pueden tener un máximo de 10 000 caracteres. 

**Filtros de ámbito no admitidos**

No se admiten extensiones de directorios, appRoleAssignments, userType y accountExpires como filtros de ámbito.


## <a name="service-issues"></a>Problemas de servicio 

**Escenarios no admitidos**

- No se admite el aprovisionamiento de contraseñas. 
- No se admite el aprovisionamiento de grupos anidados. 
- No se admite el aprovisionamiento de inquilinos B2C debido al tamaño de los inquilinos.
- No todas las aplicaciones de aprovisionamiento están disponibles en todas las nubes. Por ejemplo, Atlassian aún no está disponible en la nube de administración pública. Estamos trabajando para que los desarrolladores de aplicaciones incorporen sus aplicaciones a todas las nubes.

**El aprovisionamiento automático no está disponible en mi aplicación basada en OIDC**

Si crea un registro de aplicación, la entidad de servicio correspondiente en las aplicaciones empresariales no se habilitará para el aprovisionamiento automático de usuarios. Deberá solicitar que la aplicación se agregue a la galería, si está previsto que la usen varias organizaciones, o crear una segunda aplicación fuera de la galería para el aprovisionamiento. 

**El intervalo de aprovisionamiento es fijo**

Actualmente, el [tiempo](./application-provisioning-when-will-provisioning-finish-specific-user.md#how-long-will-it-take-to-provision-users) entre los ciclos de aprovisionamiento no es configurable. 

**Cambios que no se mueven de la aplicación de destino a Azure AD**

El servicio de aprovisionamiento de aplicaciones no reconoce los cambios realizados en las aplicaciones externas. Por tanto, no se realiza ninguna acción para la reversión. El servicio de aprovisionamiento de aplicaciones se basa en los cambios realizados en Azure AD. 

**El cambio de sincronizar todo a sincronizar asignados no funciona**

Después de cambiar el ámbito de "Sincronizar todo" a "Sync Assigned" (Sincronizar asignados), asegúrese de reiniciar también para asegurarse de que el cambio surta efecto. Puede realizar el reinicio desde la interfaz de usuario.

**El ciclo de aprovisionamiento continúa hasta la finalización**

Al establecer el aprovisionamiento `enabled = off`, o al seleccionar la opción de detención, el ciclo de aprovisionamiento actual continuará ejecutándose hasta que se complete. El servicio dejará de ejecutar todos los ciclos futuros hasta que vuelva a activar el aprovisionamiento.

**Miembro del grupo no aprovisionado**

Cuando un grupo está dentro del ámbito y un miembro está fuera, se aprovisionará el grupo. El usuario que está fuera del ámbito no se aprovisionará. Si el miembro vuelve al ámbito, el servicio no detectará el cambio inmediatamente. Al reiniciar el aprovisionamiento, se solucionará el problema. Se recomienda reiniciar periódicamente el servicio para asegurarse de que todos los usuarios se hayan aprovisionado correctamente.  

**El administrador no está aprovisionado**

Si un usuario y su administrador están ambos en el ámbito para el aprovisionamiento, el servicio aprovisionará al usuario y luego actualizará el administrador. Sin embargo, si en el día el usuario está en el ámbito y el administrador está fuera del ámbito, se aprovisionará al usuario sin la referencia del administrador. Cuando el administrador entre en el ámbito, la referencia del administrador no se actualizará hasta que se reinicie el aprovisionamiento y que el servicio vuelva a evaluar a todos los usuarios. 

## <a name="on-premises-application-provisioning"></a>Aprovisionamiento de aplicaciones locales
La siguiente información es una lista actual de limitaciones conocidas con el host del conector ECMA de Azure AD y el aprovisionamiento de aplicaciones locales.

### <a name="application-and-directories"></a>Aplicación y directorios
Todavía no se admiten las siguientes aplicaciones y directorios.

**AD DS: (escritura diferida de grupos o usuarios desde Azure AD, mediante la versión preliminar del aprovisionamiento local)**
   - Cuando un usuario se administra mediante Azure AD Connect, el origen de autoridad es Active Directory local. Por lo tanto, los atributos de usuario no se pueden cambiar en Azure AD. Esta versión preliminar no cambia el origen de autoridad para los usuarios administrados por Azure AD Connect.
   - El intento de usar Azure AD Connect y el aprovisionamiento local para aprovisionar grupos o usuarios en AD DS puede dar lugar a la creación de un bucle, donde Azure AD Connect puede sobrescribir un cambio realizado por el servicio de aprovisionamiento en la nube. Microsoft trabaja en una funcionalidad dedicada para la escritura diferida de grupos o usuarios.  Vote a favor de los comentarios de UserVoice [aquí](https://feedback.azure.com/forums/169401-azure-active-directory/suggestions/16887037-enable-user-writeback-to-on-premise-ad-from-azure) para realizar un seguimiento del estado de la versión preliminar. Como alternativa, puede usar [Microsoft Identity Manager](/microsoft-identity-manager/microsoft-identity-manager-2016) para la escritura diferida de grupos o usuarios desde Azure AD en AD.

**Conectores distintos de SQL**
   - El host del conector ECMA de Azure AD se admite oficialmente para el conector genérico de SQL (GSQL). Aunque es posible usar otros conectores, como el conector de servicios web o los conectores ECMA personalizados, **aún no se admite**.

**Azure Active Directory**
   - El aprovisionamiento local le permite tomar un usuario que ya está Azure AD y aprovisionarlo en una aplicación de terceros. **No le permite llevar a un usuario al directorio desde una aplicación de terceros.** Los clientes tendrán que confiar en nuestras integraciones nativas de RR. UU., Azure AD Connect, MIM o Microsoft Graph para llevar a los usuarios al directorio.

### <a name="attributes-and-objects"></a>Atributos y objetos 
No se admiten los siguientes atributos y objetos:
   - Atributos con varios valores
   - Atributos de referencia (por ejemplo, administrador).
   - Grupos
   - Delimitadores complejos (por ejemplo, ObjectTypeName+UserName).
   - A veces, las aplicaciones locales no están federadas con Azure AD y requieren contraseñas locales. La versión preliminar de aprovisionamiento local **no admite el aprovisionamiento de contraseñas de un solo uso ni la sincronización de contraseñas** entre Azure AD y aplicaciones de terceros.
   - El atributo virtual export_password' y las operaciones SetPassword y ChangePassword no se admiten

#### <a name="ssl-certificates"></a>Certificados SSL
   - El host del conector ECMA de Azure AD requiere actualmente que el certificado SSL sea de confianza para Azure o que el agente de aprovisionamiento se use. El asunto del certificado debe coincidir con el nombre de host en el que está instalado el host del conector ECMA de Azure AD.

#### <a name="anchor-attributes"></a>Atributos de delimitador
   - El host del conector ECMA de Azure AD no admite actualmente cambios de atributo de delimitador (cambios de nombre) ni sistemas de destino, que requieren varios atributos para formar un delimitador. 

#### <a name="attribute-discovery-and-mapping"></a>Detección y asignación de atributos
   - Los atributos que admite la aplicación de destino se detectan y exponen en Azure Portal en Asignaciones de atributos. Los atributos recién agregados se seguirán detectando. Sin embargo, si un tipo de atributo ha cambiado (por ejemplo, de cadena a booleano) y el atributo forma parte de las asignaciones, el tipo no cambiará automáticamente en Azure Portal. Los clientes tendrán que adoptar la configuración avanzada en las asignaciones y actualizar manualmente el tipo de atributo.

## <a name="next-steps"></a>Pasos siguientes
- [Funcionamiento del aprovisionamiento](how-provisioning-works.md)