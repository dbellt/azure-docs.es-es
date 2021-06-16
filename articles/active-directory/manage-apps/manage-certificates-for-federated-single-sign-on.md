---
title: Administrar certificados de federación en Azure AD | Microsoft Docs
description: Aprenda a personalizar la fecha de expiración de los certificados de federación y a renovar certificados que expiran pronto.
services: active-directory
author: mtillman
manager: CelesteDG
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.topic: conceptual
ms.date: 04/04/2019
ms.author: mtillman
ms.reviewer: jeedes
ms.collection: M365-identity-device-management
ms.openlocfilehash: 538307a4ffe9970960a4f8cea32ed8052bb87bf9
ms.sourcegitcommit: 3bb9f8cee51e3b9c711679b460ab7b7363a62e6b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/14/2021
ms.locfileid: "112080622"
---
# <a name="manage-certificates-for-federated-single-sign-on-in-azure-active-directory"></a>Administrar certificados para inicio de sesión único federado en Azure Active Directory

En este artículo se aborda información y preguntas comunes relacionadas con los certificados que crea Azure Active Directory (Azure AD) para establecer el inicio de sesión único (SSO) federado para las aplicaciones de software como servicio (SaaS). Estas aplicaciones se agregan desde la galería de aplicaciones de Azure AD o mediante una plantilla de aplicación ajena a la galería. Configure la aplicación mediante la opción de inicio de sesión único federado.

Este artículo solo es relevante para las aplicaciones que están configuradas para usar el inicio de sesión único de Azure AD mediante la federación de [Lenguaje de marcado de aserción de seguridad](https://wikipedia.org/wiki/Security_Assertion_Markup_Language) (SAML).

## <a name="auto-generated-certificate-for-gallery-and-non-gallery-applications"></a>Generación automática de certificado para aplicaciones de la galería y ajenas a la misma

Al agregar una nueva aplicación desde la galería y configurar el inicio de sesión basado en SAML (mediante la selección de **Inicio de sesión único** > **SAML** desde la página de información general de la aplicación), Azure AD genera un certificado para la aplicación con una validez de tres años. Para descargar el certificado activo como un archivo de certificado de seguridad ( **.cer**), vuelva a esa página (**inicio de sesión basado en SAML**) y seleccione un vínculo de descarga en el encabezado **Certificado de firma de SAML**. Puede elegir entre el certificado sin formato (binario) o el certificado de Base64 (texto cifrado en Base64). Para las aplicaciones de la galería, en esta sección es posible que también se muestre un vínculo para descargar el certificado como XML de metadatos de federación (un archivo **.xml**), según el requisito de la aplicación.

![Opciones de descarga del certificado de firma de SAML activo](./media/manage-certificates-for-federated-single-sign-on/active-certificate-download-options.png)

También puede descargar un certificado activo o inactivo seleccionando el icono **Editar** (con forma de lápiz) del encabezado del **certificado de firma de SAML**, que muestra la página del **certificado de firma de SAML**. Seleccione los puntos suspensivos ( **...** ) junto al certificado que quiere descargar y, a continuación, elija el formato de certificado que quiera. Tiene la opción adicional de descargar el certificado en formato de correo con privacidad mejorada (PEM). Este formato es idéntico a Base64, pero con una extensión de nombre de archivo **.pem**, que no se reconoce en Windows como formato de certificado.

![Opciones de descarga del certificado de firma de SAML (activo e inactivo)](./media/manage-certificates-for-federated-single-sign-on/all-certificate-download-options.png)

## <a name="customize-the-expiration-date-for-your-federation-certificate-and-roll-it-over-to-a-new-certificate"></a>Personalizar la fecha de expiración para el certificado de federación y sustituirlo por un certificado nuevo

De manera predeterminada, Azure configura un certificado para que expire después de tres años cuando se crea automáticamente durante la configuración de inicio de sesión único de SAML. Dado que no se puede cambiar la fecha de un certificado después de guardarlo, tendrá que:

1. Crear un certificado nuevo con la fecha deseada.
1. Guardar el certificado nuevo.
1. Descargar el certificado nuevo en el formato correcto.
1. Cargar el certificado nuevo en la aplicación.
1. Activar el certificado nuevo en el portal de Azure Active Directory.

Las dos secciones siguientes le ayudan a realizar estos pasos.

### <a name="create-a-new-certificate"></a>Crear un nuevo certificado

En primer lugar, cree y guarde el certificado nuevo con otra fecha de expiración:

1. Inicie sesión en el [portal de Azure Active Directory](https://aad.portal.azure.com/). Aparecerá la página del **Centro de administración de Azure Active Directory**.
1. En el panel izquierdo, seleccione **Aplicaciones empresariales**. Aparecerá una lista de las aplicaciones empresariales de su cuenta.
1. Seleccione la aplicación afectada. Aparecerá una página de información general de la aplicación.
1. En el panel izquierdo de la página de información general de la aplicación, seleccione **Inicio de sesión único**.
1. Si aparece la página **Seleccione un método de inicio de sesión único**, seleccione **SAML**.
1. En la página **Configurar el inicio de sesión único con SAML (versión preliminar)** , busque el encabezado **Certificado de firma de SAML** y seleccione el icono **Editar** (con forma de lápiz). Aparece la página del **certificado de firma de SAML**, que muestra el estado (**Activo** o **Inactivo**), la fecha de expiración y la huella digital (una cadena de hash) de cada certificado.
1. Seleccione **Nuevo certificado**. Aparece una nueva fila debajo de la lista de certificados, donde se establece la fecha de expiración predeterminada en exactamente tres años después de la fecha actual. (Los cambios todavía no se han guardado, por lo que aún se puede modificar la fecha de expiración).
1. En la nueva fila del certificado, pase el ratón por encima de la columna de fecha de expiración y seleccione el icono **Seleccionar fecha** (un calendario). Aparece un control de calendario que muestra los días de un mes de la fecha de expiración actual de la nueva fila.
1. Use el control del calendario para establecer una nueva fecha. Puede establecer cualquier fecha del período entre la fecha actual y los tres años posteriores.
1. Seleccione **Guardar**. El nuevo certificado aparece ahora con el estado **Inactivo**, la fecha de expiración que ha elegido y una huella digital.
1. Seleccione la **X** para volver a la página **Configurar el inicio de sesión único con SAML (versión preliminar)** .

### <a name="upload-and-activate-a-certificate"></a>Cargar y activar un certificado

A continuación, descargue el nuevo certificado en el formato correcto, cárguelo en la aplicación y actívelo en Azure Active Directory:

1. Puede ver las instrucciones de configuración de inicio de sesión de SAML adicionales de la aplicación mediante una de las siguientes formas:

   - Seleccione el vínculo de la **guía de configuración** para verlas en una pestaña o ventana del explorador independiente.
   - Vaya al encabezado de **configuración** y seleccione **Ver instrucciones detalladas** para verlas en una barra lateral.

1. En las instrucciones, tenga en cuenta el formato de codificación necesario para la carga del certificado.
1. Siga las instrucciones de la sección anterior [Generación automática de certificado para aplicaciones de la galería y ajenas a la misma](#auto-generated-certificate-for-gallery-and-non-gallery-applications). Mediante este paso se descarga el certificado en el formato de codificación que se requiere para la carga de la aplicación.
1. Cuando quiera sustituir el certificado nuevo, vuelva a la página **Certificado de firma de SAML** y, en la fila de certificado que se acaba de guardar, seleccione los puntos suspensivos ( **...** ) y seleccione **Activar el certificado**. El estado del nuevo certificado cambia a **Activo** y el certificado activo anteriormente se cambia a un estado de **Inactivo**.
1. Siga las instrucciones de configuración de inicio de sesión de SAML de la aplicación que se mostraron anteriormente para que pueda cargar el certificado de firma de SAML en el formato de codificación correcto.

## <a name="add-email-notification-addresses-for-certificate-expiration"></a>Adición de direcciones de notificación de correo electrónico a la expiración del certificado

Azure AD enviará una notificación por correo electrónico 60, 30 y 7 días antes de que expire el certificado de SAML. Puede agregar varias direcciones de correo electrónico para recibir notificaciones. Para especificar las direcciones de correo electrónico a las que quiere que se envíen las notificaciones:

1. En la página **Certificado de firma de SAML**, vaya al encabezado **Direcciones de correo electrónico de notificación**. De manera predeterminada, este encabezado solo usa la dirección de correo electrónico del administrador que agregó la aplicación.
1. Debajo de la dirección de correo electrónico final, escriba la dirección de correo electrónico que debe recibir el aviso de expiración del certificado y, a continuación, presione ENTRAR.
1. Repita el paso anterior para cada dirección de correo electrónico que quiere agregar.
1. Para cada dirección de correo electrónico que quiere eliminar, seleccione el icono **Eliminar** (un cubo de basura) junto a la dirección de correo electrónico.
1. Seleccione **Guardar**.

Puede agregar hasta 5 direcciones de correo electrónico a la lista de notificaciones (incluida la dirección de correo electrónico del administrador que agregó la aplicación). Si necesita enviar notificaciones a más personas, use los mensajes de correo electrónico de la lista de distribución.

Recibirá el correo electrónico de notificación de aadnotification@microsoft.com. Para evitar que el correo electrónico vaya a la ubicación de correo no deseado, agregue este correo electrónico a los contactos.

## <a name="renew-a-certificate-that-will-soon-expire"></a>Renovar un certificado que expirará pronto

Si un certificado está a punto de expirar, se puede renovar mediante un procedimiento que no da como resultado ningún tiempo de inactividad considerable para los usuarios. Para renovar un certificado que va a expirar:

1. Siga las instrucciones de la sección anterior [Crear un nuevo certificado](#create-a-new-certificate), con una fecha que se superponga con el certificado existente. Esa fecha limita la cantidad de tiempo de inactividad causado por la expiración del certificado.
1. Si la aplicación puede sustituir automáticamente un certificado, siga estos pasos para definir el nuevo certificado como activo:
   1. Vuelva a la página **Certificado de firma de SAML**.
   1. En la fila de certificado recién guardada, seleccione los puntos suspensivos ( **...** ) y, a continuación, seleccione **Activar el certificado**.
   1. Omita los dos pasos siguientes.

1. Si la aplicación solo puede controlar un certificado a la vez, elija un intervalo de tiempo de inactividad para realizar el paso siguiente. (En caso contrario, si la aplicación no elige automáticamente el nuevo certificado, pero puede controlar más de un certificado de firma, puede realizar el paso siguiente en cualquier momento).
1. Antes de que expire el certificado antiguo, siga las instrucciones de la sección anterior [Cargar y activar un certificado](#upload-and-activate-a-certificate).
1. Inicie sesión en la aplicación para asegurarse de que el certificado funciona correctamente.

## <a name="related-articles"></a>Artículos relacionados

- [Tutoriales para integrar aplicaciones SaaS con Azure Active Directory](../saas-apps/tutorial-list.md)
- [Administración de aplicaciones con Azure Active Directory](what-is-application-management.md)
- [Inicio de sesión único en aplicaciones en Azure Active Directory](what-is-single-sign-on.md)
- [Depuración del inicio de sesión único basado en SAML en aplicaciones de Azure Active Directory](./debug-saml-sso-issues.md)
