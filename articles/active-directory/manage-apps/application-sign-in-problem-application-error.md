---
title: Mensaje de error en la página de la aplicación al iniciar sesión | Microsoft Docs
description: Cómo resolver problemas con el inicio de sesión de Azure AD cuando la aplicación devuelve un mensaje de error.
services: active-directory
author: iantheninja
manager: CelesteDG
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.topic: troubleshooting
ms.date: 07/11/2017
ms.author: iangithinji
ms.reviewer: asteen
ms.collection: M365-identity-device-management
ms.openlocfilehash: 6ddebc4778d923bc3a002f14fc4b4db1b7bb730d
ms.sourcegitcommit: 2654d8d7490720a05e5304bc9a7c2b41eb4ae007
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/13/2021
ms.locfileid: "107379305"
---
# <a name="an-app-page-shows-an-error-message-after-the-user-signs-in"></a>Mensaje de error en la página de la aplicación cuando el usuario inicia sesión

En este escenario, Azure Active Directory (Azure AD) inicia la sesión del usuario. Pero la aplicación muestra un mensaje de error y no permite al usuario finalizar el flujo de inicio de sesión. El problema es que la aplicación no aceptó la respuesta de Azure AD.

Hay algunos posibles motivos de por qué la aplicación no aceptó la respuesta de Azure AD. Si el mensaje de error no identifica claramente lo que no está presente en la respuesta, intente lo siguiente:

-   Si la aplicación es la Galería de Azure AD, compruebe que ha seguido todos los pasos del artículo [Cómo depurar el inicio de sesión único basado en SAML en aplicaciones de Azure AD](./debug-saml-sso-issues.md).

-   Use una herramienta como [Fiddler](https://www.telerik.com/fiddler) para capturar la solicitud, la respuesta y el token de SAML.

-   Envíe la respuesta SAML al proveedor de la aplicación y pídale lo que falta.

## <a name="attributes-are-missing-from-the-saml-response"></a>Faltan atributos de la respuesta SAML

Para agregar un atributo en la configuración de Azure AD que se enviará en la respuesta de Azure AD, siga estos pasos:

1. Abra [**Azure Portal**](https://portal.azure.com/) e inicie sesión como administrador global o coadministrador.

2. En la parte superior del panel del lado izquierdo, seleccione **Todos los servicios** para abrir la extensión de Azure AD.

3. Escriba **Azure Active Directory** en el cuadro de búsqueda de filtrado y, luego, seleccione **Azure Active Directory**.

4. En el panel de navegación de Azure AD, seleccione **Aplicaciones empresariales**.

5. Seleccione **Todas las aplicaciones** para ver una lista de todas las aplicaciones.

   > [!NOTE]
   > Si no ve la aplicación que busca, use el control **Filtro** en la parte superior de la lista **Todas las aplicaciones**. Establezca la opción **Mostrar** en "Todas las aplicaciones".

6. Seleccione la aplicación que desea configurar para el inicio de sesión único.

7. Después de que la aplicación se cargue, seleccione **Inicio de sesión único** en el panel de navegación.

8. En la sección **Atributos de usuario**, seleccione **Ver y editar todos los demás atributos de usuario**. Aquí puede cambiar qué atributos enviar a la aplicación en el token SAML cuando los usuarios inician sesión.

   Para agregar un atributo:

   1. Seleccione **Agregar atributo**. Escriba el **Nombre** y seleccione el **Valor** de la lista desplegable.

   1.  Seleccione **Guardar**. En la tabla verá el nuevo atributo.

9. Guarde la configuración.

   La próxima vez que el usuario inicie sesión en la aplicación, Azure AD enviará el nuevo atributo en la respuesta de SAML.

## <a name="the-app-doesnt-identify-the-user"></a>La aplicación no identifica al usuario

No se puede iniciar sesión en la aplicación porque falta un atributo, como un rol, en la respuesta SAML. O bien, se produce un error porque la aplicación espera un valor o un formato diferente para el atributo **NameID** (identificador de usuario).

Si usa el [aprovisionamiento automático de usuarios de Azure AD](../app-provisioning/user-provisioning.md) para crear, mantener y quitar usuarios de la aplicación, compruebe que el usuario se haya aprovisionado en la aplicación SaaS. Para más información, consulte [No se aprovisionan usuarios en una aplicación de la galería de Azure AD](../app-provisioning/application-provisioning-config-problem-no-users-provisioned.md).

## <a name="add-an-attribute-to-the-azure-ad-app-configuration"></a>Incorporación de un atributo en la configuración de la aplicación de Azure AD

Para cambiar el valor del identificador de usuario, siga estos pasos:

1. Abra [**Azure Portal**](https://portal.azure.com/) e inicie sesión como administrador global o coadministrador.

2. Seleccione **Todos los servicios** en la parte superior del panel del lado izquierdo para abrir la extensión de Azure AD.

3. Escriba **Azure Active Directory** en el cuadro de búsqueda de filtrado y, luego, seleccione **Azure Active Directory**.

4. En el panel de navegación de Azure AD, seleccione **Aplicaciones empresariales**.

5. Seleccione **Todas las aplicaciones** para ver una lista de todas las aplicaciones.

   > [!NOTE]
   > Si no ve la aplicación que busca, use el control **Filtro** en la parte superior de la lista **Todas las aplicaciones**. Establezca la opción **Mostrar** en "Todas las aplicaciones".

6. Seleccione la aplicación para la que quiere configurar el SSO.

7. Después de que la aplicación se cargue, seleccione **Inicio de sesión único** en el panel de navegación.

8. En **Atributos de usuario**, seleccione el identificador único para los usuarios de la lista desplegable **Identificador de usuario**.

## <a name="change-the-nameid-format"></a>Cambio del formato de NameID

Si la aplicación espera otro formato para el atributo **NameID** (identificador de usuario), consulte [Edición de NameID](../develop/active-directory-saml-claims-customization.md#editing-nameid) para cambiar el formato de NameID.

Azure AD selecciona el formato del atributo **NameID** (identificador de usuario) en función del valor que se selecciona o del formato que solicite la aplicación en el elemento AuthRequest de SAML. Para obtener más información, vea la sección "NameIDPolicy" de [Protocolo SAML de inicio de sesión único](../develop/single-sign-on-saml-protocol.md#nameidpolicy).

## <a name="the-app-expects-a-different-signature-method-for-the-saml-response"></a>La aplicación espera un método de firma diferente para la respuesta de SAML

Para cambiar qué partes del token de SAML que Azure AD firma digitalmente, siga estos pasos:

1. Abra [Azure Portal](https://portal.azure.com/) e inicie sesión como administrador global o coadministrador.

2. Seleccione **Todos los servicios** en la parte superior del panel del lado izquierdo para abrir la extensión de Azure AD.

3. Escriba **Azure Active Directory** en el cuadro de búsqueda de filtrado y, luego, seleccione **Azure Active Directory**.

4. En el panel de navegación de Azure AD, seleccione **Aplicaciones empresariales**.

5. Seleccione **Todas las aplicaciones** para ver una lista de todas las aplicaciones.

   > [!NOTE]
   > Si no ve la aplicación que busca, use el control **Filtro** en la parte superior de la lista **Todas las aplicaciones**. Establezca la opción **Mostrar** en "Todas las aplicaciones".

6. Seleccione la aplicación que desea configurar para el inicio de sesión único.

7. Después de que la aplicación se cargue, seleccione **Inicio de sesión único** en el panel de navegación.

8. En **Certificado de firma de SAML**, seleccione **Mostrar configuración avanzada de firma de certificados**.

9. Seleccione la **opción de firma** que espera la aplicación entre estas opciones:

   * **Firmar respuesta SAML**
   * **Firmar respuesta y aserción SAML**
   * **Firmar aserción SAML**

   La próxima vez que el usuario inicie sesión en la aplicación, Azure AD firmará la parte de la respuesta de SAML seleccionada.

## <a name="the-app-expects-the-sha-1-signing-algorithm"></a>La aplicación espera el algoritmo de firma SHA-1

De forma predeterminada, Azure AD firma el token de SAML con el algoritmo de máxima seguridad. Le recomendamos que no cambie el algoritmo de firma para *SHA-1* a menos que la aplicación requiera SHA-1.

Para cambiar el algoritmo de firma, siga estos pasos:

1. Abra [Azure Portal](https://portal.azure.com/) e inicie sesión como administrador global o coadministrador.

2. Seleccione **Todos los servicios** en la parte superior del panel del lado izquierdo para abrir la extensión de Azure AD.

3. Escriba **Azure Active Directory** en el cuadro de búsqueda de filtrado y, luego, seleccione **Azure Active Directory**.

4. En el panel de navegación de Azure AD, seleccione **Aplicaciones empresariales**.

5. Seleccione **Todas las aplicaciones** para ver una lista de las aplicaciones.

   > [!NOTE]
   > Si no ve la aplicación que busca, use el control **Filtro** en la parte superior de la lista **Todas las aplicaciones**. Establezca la opción **Mostrar** en "Todas las aplicaciones".

6. Seleccione la aplicación que desea configurar para el inicio de sesión único.

7. Después de que la aplicación se carga, seleccione **Inicio de sesión único** en el panel de navegación de la izquierda de la aplicación.

8. En **Certificado de firma de SAML**, seleccione **Mostrar configuración avanzada de firma de certificados**.

9. Seleccione **SHA-1** como el **Algoritmo de firma**.

   La próxima vez que el usuario inicie sesión en la aplicación, Azure AD firmará el token SAML con el algoritmo SHA-1.

## <a name="next-steps"></a>Pasos siguientes
[Cómo depurar el inicio de sesión único basado en SAML en aplicaciones de Azure AD](./debug-saml-sso-issues.md).