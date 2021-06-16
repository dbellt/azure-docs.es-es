---
title: Configuración del inicio de sesión único (SSO) mediante un solo clic de una aplicación de Azure Marketplace | Microsoft Docs
description: Pasos para la configuración del inicio de sesión único mediante un solo clic para una aplicación de Azure Marketplace.
services: active-directory
author: mtillman
manager: CelesteDG
ms.assetid: e0416991-4b5d-4b18-89bb-91b6070ed3ba
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: conceptual
ms.date: 06/11/2019
ms.author: mtillman
ms.collection: M365-identity-device-management
ms.openlocfilehash: 766c1446a2fad9cff1103a726043d6c61d503020
ms.sourcegitcommit: 3bb9f8cee51e3b9c711679b460ab7b7363a62e6b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/14/2021
ms.locfileid: "112077021"
---
# <a name="one-click-app-configuration-of-single-sign-on"></a>Configuración del inicio de sesión único mediante un solo clic de una aplicación

 En este tutorial, aprenderá a realizar una configuración del inicio de sesión único (SSO) mediante un solo clic para aplicaciones Azure Active Directory (Azure AD) compatibles con SAML de Azure Marketplace.

## <a name="introduction-to-one-click-sso"></a>Introducción a SSO con un clic

La característica de SSO con un clic se ha diseñado para configurar el inicio de sesión único para aplicaciones de Azure Marketplace que admiten el protocolo SAML. En la página de configuración del inicio de sesión único de Azure AD, esta opción permite configurar automáticamente los metadatos de Azure AD en la aplicación. De esta forma, se puede configurar rápidamente el inicio de sesión único con un mínimo esfuerzo manual.

## <a name="advantages-of-one-click-sso"></a>Ventajas de SSO con un clic

- Configuración del inicio de sesión único rápido de las aplicaciones de Azure Marketplace que requieren configuración manual en la aplicación.
- Una configuración del inicio de sesión único más eficaz y precisa.
- No se necesitan comunicación con el asociado ni soporte técnico para la instalación. La aplicación proporciona la interfaz de usuario para la configuración de SAML.

## <a name="prerequisites"></a>Requisitos previos

- Una suscripción activa de la aplicación en la que se va a configurar el inicio de sesión único. También se necesitan credenciales de administrador.
- La **extensión de inicio de sesión seguro de Mis aplicaciones** de Microsoft instalada en el explorador. Para más información, consulte [Acceso y uso del aplicaciones en el portal Aplicaciones](../user-help/my-apps-portal-end-user-access.md).

## <a name="one-click-sso-configuration-steps"></a>Pasos para la configuración de SSO con un clic

1. Agregue la aplicación desde Azure Marketplace.

2. Seleccione **Inicio de sesión único**.

3. Seleccione **Habilitar el inicio de sesión único**.

4. Rellene los valores de configuración obligatorios en la sección **Configuración básica de SAML**.

    > [!NOTE]
    > Si la aplicación tiene notificaciones personalizadas que se deben configurar, contrólelas antes de realizar el SSO con un clic.

5. Si la característica SSO con un clic está disponible para la aplicación de Azure Marketplace, se ve la siguiente pantalla. Es posible que tenga que instalar la **extensión de inicio de sesión seguro de Mis aplicaciones**. Para ello, seleccione **Instale la extensión**.

   ![Instalar la extensión de inicio de sesión seguro de Mis aplicaciones para el explorador](./media/one-click-sso-tutorial/install-myappssecure-extension.png)

6. Después de agregar la extensión al explorador seleccione **Setup\<Application Name\>** (Instalar). Una vez que se le redirija al portal de administración de aplicaciones, inicie sesión como administrador.

   ![Configurar nombre de la aplicación](./media/one-click-sso-tutorial/setup-sso.png)

7. La extensión de explorador configura automáticamente el inicio de sesión único en la aplicación. Para confirmar, seleccione **Sí**.

   ![Guardar los datos rellenados automáticamente](./media/one-click-sso-tutorial/save-autopopulate.png)

   > [!NOTE]
   > Si la configuración del inicio de sesión único de la aplicación requiere más pasos, siga las indicaciones para realizarlos.

8. Una vez finalizada la configuración, seleccione **Aceptar** para guardar los cambios.

   ![Guardar los datos rellenados automáticamente](./media/one-click-sso-tutorial/save-data.png)

9. Se muestra una ventana de confirmación que le indica que el inicio de sesión único se ha configurado correctamente.

   ![Inicio de sesión único configurado](./media/one-click-sso-tutorial/sso-configured.png)

10. Cuando la configuración sea correcta, se cerrará la sesión de la aplicación y volverá a Azure Portal.

11. Puede seleccionar **Probar** para probar el inicio de sesión único.

## <a name="additional-resources"></a>Recursos adicionales

* [Lista de tutoriales sobre cómo integrar aplicaciones SaaS con Azure Active Directory](../saas-apps/tutorial-list.md)
* [¿Qué es la extensión de inicio de sesión seguro de Mis aplicaciones para el explorador?](../user-help/my-apps-portal-end-user-access.md)
