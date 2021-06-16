---
title: Solución de problemas del inicio de sesión único basado en SAML en Azure Active Directory
description: Solución de problemas relacionados con una aplicación de Azure AD configurada para el inicio de sesión único basado en SAML.
services: active-directory
author: mtillman
manager: CelesteDG
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.topic: troubleshooting
ms.date: 07/11/2017
ms.author: mtillman
ms.openlocfilehash: b119df626eed15c15fdb267a0a78b6fd62b72162
ms.sourcegitcommit: 3bb9f8cee51e3b9c711679b460ab7b7363a62e6b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/14/2021
ms.locfileid: "112080550"
---
# <a name="troubleshoot-saml-based-single-sign-on-in-azure-active-directory"></a>Solución de problemas del inicio de sesión único basado en SAML en Azure Active Directory
Si se produce un problema al configurar una aplicación. Compruebe que ha seguido todos los pasos del tutorial de la aplicación. En la configuración de la aplicación, tiene documentación en línea sobre cómo configurarla. También para instrucciones detalladas paso a paso, acceda a la [Lista de tutoriales sobre cómo integrar aplicaciones SaaS con Azure Active Directory](../saas-apps/tutorial-list.md).

## <a name="cant-add-another-instance-of-the-application"></a>No se puede agregar otra instancia de la aplicación
Para agregar una segunda instancia de una aplicación, tiene que poder:
-   Configurar un identificador único para la segunda instancia. No podrá configurar el mismo identificador que usó para la primera instancia.
-   Configurar un certificado diferente al utilizado para la primera instancia.

Si la aplicación no es compatible con cualquiera de los pasos anteriores. Entonces, no podrá configurar una segunda instancia.

## <a name="cant-add-the-identifier-or-the-reply-url"></a>No se puede agregar el identificador o la dirección URL de respuesta
Si no puede configurar el identificador o la dirección URL de respuesta, confirme que los valores de identificador y la dirección URL de respuesta coincidan con los patrones configurados previamente para la aplicación.

Para conocer los patrones preconfigurados para la aplicación:
1. Abra [**Azure Portal**](https://portal.azure.com/) e inicie sesión como **administrador global** o **coadministrador.** Vaya al paso 7. Si ya está en la hoja de configuración de aplicación en Azure AD.
2. Abra la **extensión de Azure Active Directory** haciendo clic en **Todos los servicios** en la parte superior del menú de navegación izquierdo principal.
3. Escriba **"Azure Active Directory**" en el cuadro de búsqueda de filtrado y seleccione el elemento **Azure Active Directory**.
4. Haga clic en **Aplicaciones empresariales** en el menú de navegación izquierdo de Azure Active Directory.
5. Haga clic en **Todas las aplicaciones** para ver una lista de todas las aplicaciones.
   * Si no ve la aplicación que desea que aparezca aquí, use el control **Filtro** de la parte superior de la lista **Todas las aplicaciones** y establezca la opción **Mostrar** en **Todas las aplicaciones.**
6. Seleccione la aplicación que desea configurar para el inicio de sesión único.
7. Cuando se cargue la aplicación, haga clic en **Inicio de sesión único** desde el menú de navegación izquierdo de la aplicación.
8. En la lista desplegable **Modo**, seleccione **Inicio de sesión basado en SAML**.
9. Vaya al cuadro de texto **Identificador** o **URL de respuesta** en la **sección Dominio y direcciones URL.**
10. Hay tres maneras de saber los patrones admitidos para la aplicación:
    * En el cuadro de texto, verá los patrones admitidos como un marcador de posición *Ejemplo:* <https://contoso.com>.
    * Si el patrón no se admite, verá un signo de exclamación rojo al intentar escribir el valor en el cuadro de texto. Si sitúa el mouse sobre el signo de exclamación rojo, verá los patrones admitidos.
    * En el tutorial de la aplicación, también puede obtener información acerca de los modelos admitidos. En la sección **Configuración del inicio de sesión único de Azure AD**. Vaya al paso para configurar los valores en el **Dominio y direcciones URL**.

Si los valores no coinciden con los patrones preconfigurados en Azure AD. Puede:
-   Trabajar con el proveedor de la aplicación para obtener los valores que coinciden con el patrón preconfigurado en Azure AD
-   O ponerse en contacto con el equipo de Azure AD en <aadapprequest@microsoft.com> o puede dejar un comentario en el tutorial para solicitar la actualización de los patrones admitidos para la aplicación

## <a name="where-do-i-set-the-entityid-user-identifier-format"></a>¿Dónde se puede establecer el formato de EntityID (Identificador de usuario)?
No podrá seleccionar el formato de EntityID (Identificador de usuario) que Azure AD envía a la aplicación en la respuesta a la autenticación de usuario.

AD Azure selecciona el formato del atributo NameID (Identificador de usuario) en función del valor seleccionado o del formato que solicite la aplicación en el elemento AuthRequest de SAML. Para más información, visite la sección NameIDPolicy del artículo [Protocolo SAML de inicio de sesión único](../develop/single-sign-on-saml-protocol.md#authnrequest).

## <a name="cant-find-the-azure-ad-metadata-to-complete-the-configuration-with-the-application"></a>No se encuentran los metadatos de Azure AD para completar la configuración con la aplicación
Para descargar el certificado o los metadatos de la aplicación de Azure AD, siga estos pasos:
1. Abra [**Azure Portal**](https://portal.azure.com/) e inicie sesión como **administrador global** o **coadministrador.**
2. Abra la **extensión de Azure Active Directory** haciendo clic en **Todos los servicios** en la parte superior del menú de navegación izquierdo principal.
3. Escriba **"Azure Active Directory**" en el cuadro de búsqueda de filtrado y seleccione el elemento **Azure Active Directory**.
4. Haga clic en **Aplicaciones empresariales** en el menú de navegación izquierdo de Azure Active Directory.
5. Haga clic en **Todas las aplicaciones** para ver una lista de todas las aplicaciones.
   * Si no ve la aplicación que desea que aparezca aquí, use el control **Filtro** de la parte superior de la lista **Todas las aplicaciones** y establezca la opción **Mostrar** en **Todas las aplicaciones.**
6. Seleccione la aplicación para la que ha configurado el inicio de sesión único.
7. Cuando se cargue la aplicación, haga clic en **Inicio de sesión único** desde el menú de navegación izquierdo de la aplicación.
8. Vaya a la sección **Certificado de firma de SAML** y haga clic en el valor de la columna **Descargar**. Según lo que necesite la aplicación para configurar el inicio de sesión único, verá la opción para descargar el archivo XML de metadatos o el certificado.

Azure AD no proporciona una dirección URL para obtener los metadatos. Solo se pueden recuperar como un archivo XML.

## <a name="customize-saml-claims-sent-to-an-application"></a>Personalización de las notificaciones de SAML enviadas a una aplicación
Para obtener información sobre cómo personalizar las notificaciones de atributo SAML que se han enviado a su aplicación, vea [Asignación de notificaciones en Azure Active Directory](../develop/active-directory-claims-mapping.md).

## <a name="next-steps"></a>Pasos siguientes
* [Serie de guías de inicio rápido sobre la administración de aplicaciones](view-applications-portal.md)