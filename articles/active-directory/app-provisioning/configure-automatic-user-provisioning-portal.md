---
title: Administración de aprovisionamiento de usuarios para aplicaciones empresariales en Azure Active Directory
description: Aprenda a administrar el aprovisionamiento de cuentas de usuario para aplicaciones empresariales con Azure Active Directory.
services: active-directory
author: kenwith
manager: mtillman
ms.service: active-directory
ms.subservice: app-provisioning
ms.topic: how-to
ms.workload: identity
ms.date: 05/11/2021
ms.author: kenwith
ms.reviewer: arvinh
ms.openlocfilehash: 2403ef54dfe3e0262440944b134e8021bc4c67c1
ms.sourcegitcommit: 32ee8da1440a2d81c49ff25c5922f786e85109b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2021
ms.locfileid: "109782890"
---
# <a name="managing-user-account-provisioning-for-enterprise-apps-in-the-azure-portal"></a>Administración del aprovisionamiento de cuentas de usuario para aplicaciones empresariales en el portal de Azure

En este artículo se describen los pasos generales para administrar el aprovisionamiento y el desaprovisionamiento automáticos de cuentas de usuario en aplicaciones que lo admitan. El *aprovisionamiento de cuentas de usuario* es el acto de crear, actualizar o deshabilitar registros de cuenta de usuario en el almacén de perfiles de usuario local de una aplicación. La mayoría de las aplicaciones SaaS y en la nube almacenan el rol y los permisos de los usuarios en el propio almacén local de perfiles de usuario del usuario y la presencia de tal registro de usuario en el almacén local del usuario es *necesaria* para que funcionen el inicio de sesión único y el acceso. Para más información sobre el aprovisionamiento automático de cuentas de usuario, consulte [Automatización del aprovisionamiento y desaprovisionamiento de usuarios para aplicaciones con Azure Active Directory](user-provisioning.md).

> [!IMPORTANT]
> Azure Active Directory (Azure AD) dispone de una galería con miles de aplicaciones previamente integradas que están habilitadas para el aprovisionamiento automático con Azure AD. Para comenzar, busque el tutorial de configuración del aprovisionamiento específico de su aplicación en [Tutoriales para integrar aplicaciones SaaS con Azure Active Directory](../saas-apps/tutorial-list.md). Probablemente encontrará instrucciones paso a paso para configurar tanto la aplicación como Azure AD para crear la conexión de aprovisionamiento.

## <a name="finding-your-apps-in-the-portal"></a>Búsqueda de aplicaciones en el portal

Use el portal de Azure Active Directory para ver y administrar todas las aplicaciones que están configuradas para el inicio de sesión único en un directorio. Las aplicaciones empresariales son aplicaciones que se implementan y se usan dentro de su organización. Siga estos pasos para ver y administrar las aplicaciones empresariales:

1. Abra el [portal de Azure Active Directory](https://aad.portal.azure.com).
1. En el panel izquierdo, seleccione **Aplicaciones empresariales**. Se muestra una lista de las aplicaciones configuradas, incluidas aquellas que se han agregado desde la galería.
1. Seleccione cualquier aplicación para cargar el panel de recursos, donde puede ver los informes y administrar la configuración de la aplicación.
1. Seleccione **Aprovisionamiento** para administrar la configuración de aprovisionamiento de cuentas de usuario de la aplicación seleccionada.

   ![Pantalla Aprovisionamiento para administrar la configuración del aprovisionamiento de cuentas de usuario](./media/configure-automatic-user-provisioning-portal/enterprise-apps-provisioning.png)

## <a name="provisioning-modes"></a>Modos de aprovisionamiento

El panel **Aprovisionamiento** comienza con un menú **Modo**, que muestra los modos de aprovisionamiento admitidos para una aplicación empresarial y permite configurarlos. Las opciones disponibles incluyen:

* **Automático**: esta opción aparece si Azure AD admite el aprovisionamiento automático basado en API o el desaprovisionamiento de cuentas de usuario para esta aplicación. Seleccione este modo para mostrar una interfaz que permita a los administradores:

  * Configurar Azure AD para conectarse a la API de administración de usuarios de la aplicación
  * Crear asignaciones de cuentas y flujos de trabajo que definen cómo deben fluir los datos de la cuenta de usuario entre Azure AD y la aplicación
  * Administrar el servicio de aprovisionamiento de Azure AD

* **Manual**: esta opción se muestra si Azure AD no admite el aprovisionamiento automático de cuentas de usuario para esta aplicación. En este caso, los registros de cuenta de usuario almacenados en la aplicación deben administrarse mediante un proceso externo, según las funcionalidades de aprovisionamiento y administración de usuarios proporcionadas por la aplicación (que puede incluir el aprovisionamiento Just-In-Time de SAML).

## <a name="configuring-automatic-user-account-provisioning"></a>Configuración del aprovisionamiento automático de cuentas de usuario

Seleccione la opción **Automático** para especificar la configuración de las credenciales de administrador, las asignaciones, el inicio y parada y la sincronización.

### <a name="admin-credentials"></a>Credenciales de administrador

Expanda **Credenciales de administrador** para especificar las credenciales necesarias para que Azure AD se conecte a la API de administración de usuarios de la aplicación. La entrada necesaria varía dependiendo de la aplicación. Para más información sobre los requisitos y tipos de credenciales para aplicaciones específicas, consulte el [tutorial de configuración de la aplicación específica](user-provisioning.md).

Seleccione el botón **Probar conexión** para probar las credenciales al hacer que Azure AD intente conectarse a la aplicación de aprovisionamiento de la aplicación con las credenciales proporcionadas.

### <a name="mappings"></a>Asignaciones

Expanda **Asignaciones** para ver y modificar los atributos de usuario que fluyen entre Azure AD y la aplicación de destino cuando las cuentas de usuario se aprovisionan o se actualizan.

Hay un conjunto preconfigurado de asignaciones entre los objetos de usuario de Azure AD y los objetos de usuario de cada aplicación SaaS. Algunas aplicaciones también administran objetos de grupo. Seleccione una asignación de la tabla para abrir el editor de asignaciones, donde puede verlas y personalizarlas.

Las personalizaciones compatibles incluyen:

* Habilitar y deshabilitar asignaciones para objetos específicos, como el objeto de usuario de Azure AD en el objeto de usuario de la aplicación SaaS.
* Editar los atributos que fluyen desde el objeto de usuario de Azure AD al objeto de usuario de la aplicación. Para más información sobre la asignación de atributos, consulte la sección [Información sobre los tipos de asignaciones de atributos](customize-application-attributes.md#understanding-attribute-mapping-types).
* Filtrar las acciones de aprovisionamiento que ejecuta Azure AD en la aplicación de destino. En lugar de que Azure AD sincronice totalmente los objetos, puede limitar las acciones ejecutadas.

  Por ejemplo, si solo selecciona **Actualizar**, Azure AD únicamente actualiza las cuentas de usuario existentes en una aplicación, pero no crea otras nuevas. Si solo selecciona **Crear**, Azure únicamente crea nuevas cuentas de usuario, pero no actualiza las existentes. Esta característica permite a los administradores crear asignaciones diferentes para la creación de cuentas y la actualización de los flujos de trabajo.

* Adición de una nueva asignación de atributo. Seleccione **Agregar nueva asignación** en la parte inferior del panel **Asignación de atributos**. Rellene el formulario **Editar atributo** y seleccione **Aceptar** para agregar la nueva asignación a la lista.

### <a name="settings"></a>Configuración

Expanda **Configuración** para establecer una dirección de correo electrónico para recibir notificaciones y si desea recibir alertas sobre errores. También puede seleccionar el ámbito de los usuarios que se sincronizarán. Puede optar por sincronizar todos los usuarios y grupos o solo los que se han asignado.

### <a name="provisioning-status"></a>Estado de aprovisionamiento 

Si el aprovisionamiento se habilita por primera vez para una aplicación, active el servicio mediante el cambio de **Estado de aprovisionamiento** a **Activado**. Este cambio hace que el servicio de aprovisionamiento de Azure AD ejecute un ciclo inicial. Lee los usuarios asignados en la sección **Usuarios y grupos**, consulta la aplicación de destino para ellos y, después, ejecuta las acciones de aprovisionamiento definidas en la sección **Asignaciones** de Azure AD. Durante este proceso, el servicio de aprovisionamiento almacena datos en caché sobre las cuentas de usuario que administra, para que las cuentas no administradas dentro de las aplicaciones de destino que nunca han estado en el ámbito de asignación no se vean afectadas por las operaciones de desaprovisionamiento. Después del ciclo inicial, el servicio de aprovisionamiento sincroniza automáticamente los objetos de grupo y usuario con un intervalo de cuarenta minutos.

Cambie **Estado de aprovisionamiento** a **Desactivado** para pausar el servicio de aprovisionamiento. En este estado, Azure no crea, actualiza ni quita ningún objeto de grupo o usuario en la aplicación. Cambie el estado de nuevo a **Activado** y el servicio vuelve adonde lo dejó.
