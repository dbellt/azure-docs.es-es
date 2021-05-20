---
title: 'Tutorial: Integración de Azure Active Directory con Amazon Web Services para conectar varias cuentas | Microsoft Docs'
description: Obtenga información sobre cómo configurar el inicio de sesión único entre Azure AD y Amazon Web Services (tutorial heredado).
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 12/24/2020
ms.author: jeedes
ms.openlocfilehash: 59faa9bb9dba0b65a820db5702ea2c76a4bf86d8
ms.sourcegitcommit: 2cb7772f60599e065fff13fdecd795cce6500630
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108802025"
---
# <a name="tutorial-azure-active-directory-integration-with-amazon-web-services"></a>Tutorial: Integración de Azure Active Directory con Amazon Web Services

En este tutorial, aprenderá a integrar Azure Active Directory (Azure AD) con Amazon Web Services (AWS) (tutorial heredado).

Esta integración proporciona las ventajas siguientes:

- Puede controlar en Azure AD quién tiene acceso a AWS.
- Puede permitir que los usuarios inicien sesión automáticamente en AWS mediante el inicio de sesión único (SSO) con sus cuentas de Azure AD.
- Puede administrar sus cuentas en una ubicación central: Azure Portal.

![Diagrama de la integración de Azure AD con AWS.](./media/aws-multi-accounts-tutorial/amazonwebservice.png)

> [!NOTE]
> Se recomienda que _no_ conecte una aplicación de AWS a todas sus cuentas de AWS. En su lugar, se recomienda usar la [integración del inicio de sesión único de Azure AD con AWS](./amazon-web-service-tutorial.md) para configurar varias instancias de la cuenta de AWS en varias instancias de aplicaciones de AWS en Azure AD.

Se recomienda que _no_ conecte una aplicación de AWS a todas sus cuentas de AWS por los siguientes motivos:

- Utilice este enfoque solo si tiene un pequeño número de cuentas y roles de AWS, porque este modelo no es escalable a medida que aumenta el número de cuentas de AWS y los roles dentro de ellas. Este enfoque no usa la funcionalidad de importación de roles de AWS con el aprovisionamiento de usuarios de Azure AD, por lo que tiene que agregar, actualizar y eliminar los roles manualmente.

- Tiene que usar el enfoque de Probador de Microsoft Graph para aplicar revisiones a todos los roles de la aplicación. No se recomienda usar el enfoque de archivo de manifiesto.

- Los clientes informan de que tras agregar aproximadamente 1200 roles de aplicación para una única aplicación de AWS, todas las operaciones en la aplicación comenzaban a generar errores relacionados con el tamaño. Hay un límite de tamaño estricto sobre el objeto de aplicación.

- Es necesario actualizar manualmente los roles a medida que se agregan a cualquiera de las cuentas. Lamentablemente, se trata de un enfoque _reemplazar_ y no de un enfoque _anexar_. Además, si crece el número de cuentas, esto se convierte en una relación _n_ &times; _n_ con las cuentas y los roles.

- Todas las cuentas de AWS utilizan el mismo archivo XML de metadatos de federación. En el momento de la sustitución del certificado, actualizar el certificado en todas las cuentas de AWS al mismo tiempo puede ser un ejercicio masivo.

## <a name="prerequisites"></a>Prerrequisitos

Para configurar la integración de Azure AD con AWS, se necesitan los siguientes elementos:

- Una suscripción de Azure AD. Si no dispone de una suscripción de Azure AD, puede obtener una [evaluación gratuita de un mes](https://azure.microsoft.com/pricing/free-trial/).
- Una suscripción de AWS habilitada para el inicio de sesión único.

> [!NOTE]
> No se recomienda que pruebe los pasos de este tutorial en un entorno de producción a menos que sea necesario.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

AWS admite el inicio de sesión único iniciado por SP e IDP.

## <a name="add-aws-from-the-gallery"></a>Adición de AWS desde la galería

Para configurar la integración de AWS en Azure AD, debe agregar AWS desde la galería a la lista de aplicaciones Software como servicio (SaaS) administradas.

1. Inicie sesión en Azure Portal con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel izquierdo, seleccione el servicio de Azure AD con el que desea trabajar.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Amazon Web Services** en el cuadro de búsqueda.
1. Seleccione **Amazon Web Services** en la lista de resultados y, a continuación, agregue la aplicación. En cuestión de segundos, la aplicación se agregará al inquilino.

1. Vaya al panel **Propiedades** y copie el valor que se muestra en el cuadro **Id. de objeto**.

   ![Captura de pantalla del cuadro Id. de objeto en el panel Propiedades.](./media/aws-multi-accounts-tutorial/tutorial-amazonwebservices-properties.png)

## <a name="configure-and-test-azure-ad-sso"></a>Configuración y prueba del inicio de sesión único de Azure AD

En esta sección, va a configurar y probar el inicio de sesión único de Azure AD con AWS con un usuario de prueba llamado "Britta Simon.

Para que el inicio de sesión único funcione, Azure AD debe saber cuál es el usuario homólogo en AWS para el usuario de Azure AD. Es decir, es necesario establecer una relación de vinculación entre el usuario de Azure AD y el mismo usuario en AWS.

Para establecer la relación de vinculación, en AWS, asigne el valor del **nombre de usuario** de Azure AD como el valor del **nombre de usuario** en AWS.

Para configurar y probar el inicio de sesión único de Azure AD con AWS, haga lo siguiente:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
1. **[Configuración del inicio de sesión único de AWS](#configure-aws-sso)** , para configurar las opciones de inicio de sesión único en la aplicación.
1. **[Prueba del inicio de sesión único](#test-sso)** para comprobar que la configuración funciona.

### <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

En esta sección, habilitará el inicio de sesión único de Azure AD en Azure Portal y configurará el inicio de sesión único en la aplicación AWS de la siguiente manera:

1. En Azure Portal, en el panel izquierdo de la página de integración de la aplicación **Amazon Web Services (AWS)** , seleccione **Inicio de sesión único**.

   ![Captura de pantalla del comando "Inicio de sesión único".](common/select-sso.png)

1. En el panel **Seleccione un método de inicio de sesión único**, seleccione el modo **SAML/WS-Fed** para habilitar el inicio de sesión único.

   ![Captura de pantalla del panel "Seleccionar un método de inicio de sesión único.](common/select-saml-option.png)

1. En el panel **Configurar el inicio de sesión único con SAML**, seleccione el botón **Editar** (icono de lápiz).

   ![Captura de pantalla del botón Editar en el panel "Configurar el inicio de sesión único con SAML".](common/edit-urls.png)

1. Se abre el panel **Configuración básica de SAML**. Omita esta sección, ya que la aplicación está preintegrada con Azure. Seleccione **Guardar**.

   La aplicación AWS espera las aserciones de SAML en un formato específico. Puede administrar los valores de estos atributos en la sección **Atributos y notificaciones del usuario** de la página **Integración de aplicaciones**.

1. En la página **Configurar el inicio de sesión único con SAML**, seleccione el botón **Editar**.

   ![Captura de pantalla del botón Editar en el panel "Atributos del usuario".](common/edit-attribute.png)

1. En la sección **Notificaciones del usuario** del panel **Atributos de usuario**, configure el atributo del token de SAML con los valores de la siguiente tabla:

   | Nombre            | Atributo de origen                                                            | Espacio de nombres                                |
   | --------------- | --------------------------------------------------------------------------- | ---------------------------------------- |
   | RoleSessionName | user.userprincipalname                                                      | `https://aws.amazon.com/SAML/Attributes` |
   | Role            | user.assignedroles                                                          | `https://aws.amazon.com/SAML/Attributes` |
   | SessionDuration | "proporcione un valor comprendido entre 900 segundos (15 minutos) y 43 200 segundos (12 horas)" | `https://aws.amazon.com/SAML/Attributes` |

   a. Seleccione **Agregar nueva notificación** y, a continuación, en el panel **Administrar las notificaciones del usuario**, haga lo siguiente:

   ![Captura de pantalla de los botones "Agregar nueva notificación" y "Guardar" en el panel "Notificaciones del usuario".](common/new-save-attribute.png)

   ![Captura de pantalla del panel "Administrar las notificaciones del usuario".](common/new-attribute-details.png)

   b. En el cuadro **Nombre**, escriba el nombre de atributo.

   c. En el cuadro de texto **Namespace** (Espacio de nombres), escriba el valor del espacio de nombres.

   d. En **Origen**, seleccione **Atributo**.

   e. En la lista desplegable **Atributo de origen**, seleccione el atributo.

   f. Seleccione **Aceptar** y después **Guardar**.

   > [!NOTE]
   > Para más información sobre los roles en Azure AD, consulte [Procedimiento: Incorporación de roles de aplicación a una aplicación y su recepción en el token](../develop/howto-add-app-roles-in-azure-ad-apps.md#app-roles-ui).

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, seleccione **Descargar** para descargar el archivo XML de metadatos de federación y guárdelo en su equipo.

   ![Captura de pantalla del vínculo "Descargar" para el archivo XML de metadatos de federación en el panel "Certificado de firma de SAML".](common/metadataxml.png)

### <a name="configure-aws-sso"></a>Configuración del inicio de sesión único de AWS

1. En una nueva ventana del explorador, inicie sesión en el sitio de la compañía de AWS como administrador.

1. Seleccione el icono **AWS Home** (Página principal de AWS).

   ![Captura de pantalla del icono "AWS Home" (Página principal de AWS).][11]

1. En el panel **AWS services** (Servicios de AWS), en **Security, Identity & Compliance** (Seguridad, identidad y cumplimiento), seleccione **IAM (Identity & Access Management)** (IAM [Administración de identidad y acceso]).

   ![Captura de pantalla del vínculo "Identity and Access Management" (Administración de identidad y acceso) en el panel "AWS Services" (Servicios de AWS).][12]

1. En el panel izquierdo, seleccione **Identity Providers** (Proveedores de identidades) y, a continuación, seleccione **Create Provider** (Crear proveedor).

   ![Captura de pantalla del botón "Create Provider" (Crear proveedor).][13]

1. En el panel **Configure Provider** (Configurar proveedor), realice los pasos siguientes:

   ![Captura de pantalla del panel "Configure Provider" (Configurar proveedor).][14]

   a. En la lista desplegable **Provider Type** (Tipo de proveedor), seleccione **SAML**.

   b. En el cuadro **Provider Name** (Nombre de proveedor), escriba un nombre de proveedor (por ejemplo, _WAAD_).

   c. Junto a la casilla **Metadata Document** (Documento de metadatos), seleccione **Choose File** (Seleccionar archivo) para cargar el archivo XML de metadatos de federación descargado de Azure Portal.

   d. Seleccione **Next Step** (Siguiente paso).

1. En el panel **Verify Provider Information** (Comprobar la información del proveedor), seleccione **Create** (Crear).

   ![Captura de pantalla del panel "Verify Provider Information" (Comprobar la información del proveedor).][15]

1. En el panel izquierdo, seleccione **Roles** (Roles) y, a continuación, seleccione **Create role** (Crear rol).

   ![Captura de pantalla del botón "Create role" (Crear rol) en el panel Roles.][16]

   > [!NOTE]
   > La longitud combinada del nombre de recurso de Amazon (ARN) de rol y el ARN del proveedor de SAML para un rol que se va a importar debe ser de 240 caracteres o menos.

1. En la página **Create role** (Crear rol), realice los pasos siguientes:

   ![Captura de pantalla del botón de entidad de confianza "SAML 2.0 federation" (Federación de SAML 2.0) en la página "Create role" (Crear rol).][19]

   a. En **Select type of trusted entity** (Seleccionar tipo de entidad de confianza), seleccione **SAML 2.0 federation** (Federación de SAML 2.0).

   b. En **Choose a SAML 2.0 Provider** (Elegir un proveedor de SAML 2.0), seleccione el proveedor de SAML que ha creado anteriormente (por ejemplo, _WAAD_).

   c. Seleccione **Allow programmatic and AWS Management Console access** (Permitir acceso mediante programación y a consola de AWS Management Console).

   d. Seleccione **Siguiente: Permisos**.

1. En el cuadro de búsqueda, escriba **Administrator Access** (Acceso de administrador), seleccione la casilla **AdministratorAccess** y, a continuación, seleccione **Next: Tags** (Siguiente: Etiquetas).

   ![Captura de pantalla de la lista "Policy name" (Nombre de directiva) con la directiva AdministratorAccess seleccionada.](./media/aws-multi-accounts-tutorial/administrator-access.png)

1. En el panel **Add tags (optional)** (Agregar etiquetas [opcional]), siga estos pasos:

   ![Captura de pantalla del panel "Add tags (optional)" (Agregar etiquetas [opcional]).](./media/aws-multi-accounts-tutorial/config2.png)

   a. En el cuadro de texto **Key** (Clave), escriba el nombre de la clave (por ejemplo, _Azureadtest_).

   b. En el cuadro de texto **Value (optional)** (Valor [opcional]), escriba el valor de la clave con el siguiente formato: `<accountname-aws-admin>`. El nombre de cuenta se debe escribir todo en minúsculas.

   c. Seleccione **Siguiente: Review** (Siguiente: revisar).

1. En el panel **Review** (Revisar), siga estos pasos:

   ![Captura de pantalla del panel Review (Revisión), con las casillas "Role name" (Nombre del rol) y "Role description" (Descripción del rol) resaltadas.][34]

   a. En el cuadro de texto **Role name** (Nombre del rol), escriba el valor con el siguiente formato: `<accountname-aws-admin>`.

   b. En el cuadro de texto **Role description** (Descripción del rol), escriba el valor que utilizó para el nombre del rol.

   c. Seleccione **Create Role** (Crear rol).

   d. Cree tantos roles como sea necesario y asígnelos al proveedor de identidades.

   > [!NOTE]
   > Puede crear otros roles de forma similar, como _accountname-finance-admin_, _accountname-read-only-user_, _accountname-devops-user_ o _accountname-tpm-user_, cada uno con una directiva diferente asignada. Puede cambiar estas políticas de roles más adelante, según los requisitos de cada cuenta de AWS. Es una buena idea mantener las mismas políticas para cada rol en todas las cuentas de AWS.

1. Asegúrese de anotar el identificador de la cuenta de AWS en el panel de propiedades de Amazon Elastic Compute Cloud (Amazon EC2) o en el panel de IAM, como se muestra en la siguiente captura de pantalla:

   ![Captura de pantalla en la que se muestra el identificador de la cuenta en el panel "Identity and Access Management" (Administración de identidad y acceso).](./media/aws-multi-accounts-tutorial/aws-accountid.png)

1. Inicie sesión en Azure Portal y vaya a **Grupos**.

1. Cree nuevos grupos con el mismo nombre que el de los roles de IAM creados anteriormente y anote el valor del cuadro de texto **Id. de objeto** de cada uno de estos nuevos grupos.

   ![Captura de pantalla de los detalles de la cuenta de un nuevo grupo.](./media/aws-multi-accounts-tutorial/copy-objectids.png)

1. Cierre la sesión de la cuenta de AWS actual e inicie sesión con la otra cuenta en la que desea configurar el inicio de sesión único con Azure AD.

1. Una vez creados todos los roles en las cuentas, se muestran en la lista **Roles** de dichas cuentas.

   ![Captura de pantalla de la lista de roles en la que se muestra el nombre, la descripción y las entidades de confianza de cada rol.](./media/aws-multi-accounts-tutorial/tutorial-amazonwebservices-listofroles.png)

A continuación, debe capturar todos los ARN de los roles y las entidades de confianza de todos los roles en todas las cuentas. Tendrá que asignarlos manualmente con la aplicación de Azure AD. Para ello:

1. Seleccione cada rol para copiar su ARN de rol y sus valores de entidades de confianza. Los necesitará para todos los roles que va a crear en Azure AD.

   ![Captura de pantalla del panel de resumen para los ARN de los roles y las entidades de confianza.](./media/aws-multi-accounts-tutorial/tutorial-amazonwebservices-role-summary.png)

1. Repita el paso anterior para todos los roles de todas las cuentas y, a continuación, almacénelos en un archivo de texto con el siguiente formato: `<Role ARN>,<Trusted entities>`.

1. Abra el [Explorador de Microsoft Graph](https://developer.microsoft.com/graph/graph-explorer) y haga lo siguiente:

   a. Inicie sesión en el sitio del Explorador de Microsoft Graph con las credenciales de administrador global o coadministrador del inquilino.

   b. Para crear los roles es preciso tener permisos suficientes. Seleccione **Modificar permisos**.

   ![Captura de pantalla del vínculo "Modificar permisos" en el panel de autenticación del Explorador de Microsoft Graph.](./media/aws-multi-accounts-tutorial/graph-explorer-new9.png)

   c. En la lista de permisos, si aún no tiene los permisos que se muestran en la siguiente captura de pantalla, seleccione cada uno de ellos y, a continuación, seleccione **Modificar permisos**.

   ![Captura de pantalla de la lista de permisos del Explorador de Microsoft Graph, con los permisos adecuados resaltados.](./media/aws-multi-accounts-tutorial/graph-explorer-new10.png)

   d. Vuelva a iniciar sesión en el Explorador de Graph y acepte las condiciones de uso del sitio.

   e. En la parte superior del panel, seleccione **GET** para el método, seleccione **beta** para la versión y, a continuación, en el cuadro de consulta, escriba una de las siguientes opciones:

   - Para capturar todas las entidades de servicio del inquilino, utilice: `https://graph.microsoft.com/beta/servicePrincipals`.
   - Si utiliza varios directorios, utilice `https://graph.microsoft.com/beta/contoso.com/servicePrincipals`, que contiene el dominio principal.

   ![Captura de pantalla del panel "Cuerpo de la solicitud" de la consulta del Explorador de Microsoft Graph.](./media/aws-multi-accounts-tutorial/graph-explorer-new1.png)

   f. En la lista de entidades de servicio, seleccione la que debe modificar.

   También puede usar Ctrl + F para buscar la aplicación en todas las entidades de servicio enumeradas. Para obtener una entidad principal de servicio específica, incluya en la consulta el identificador de objeto de la entidad de servicio, que copió anteriormente del panel de propiedades de Azure AD, como se muestra aquí:

   `https://graph.microsoft.com/beta/servicePrincipals/<objectID>`.

   ![Captura de pantalla que muestra una consulta de entidad de servicio que incluye el identificador del objeto.](./media/aws-multi-accounts-tutorial/graph-explorer-new2.png)

   g. Extraiga la propiedad appRoles del objeto de entidad de servicio.

   ![Captura de pantalla del código para extraer la propiedad appRoles del objeto de la entidad de servicio.](./media/aws-multi-accounts-tutorial/graph-explorer-new3.png)

   h. Ahora debe generar roles nuevos para la aplicación.

   i. El siguiente código JSON es un ejemplo del objeto appRoles. Cree un objeto similar para agregar los roles que desee para su aplicación.

   ```
   {
   "appRoles": [
       {
           "allowedMemberTypes": [
               "User"
           ],
           "description": "msiam_access",
           "displayName": "msiam_access",
           "id": "7dfd756e-8c27-4472-b2b7-38c17fc5de5e",
           "isEnabled": true,
           "origin": "Application",
           "value": null
       },
       {
           "allowedMemberTypes": [
               "User"
           ],
           "description": "Admin,WAAD",
           "displayName": "Admin,WAAD",
           "id": "4aacf5a4-f38b-4861-b909-bae023e88dde",
           "isEnabled": true,
           "origin": "ServicePrincipal",
           "value": "arn:aws:iam::12345:role/Admin,arn:aws:iam::12345:saml-provider/WAAD"
       },
       {
           "allowedMemberTypes": [
               "User"
           ],
           "description": "Auditors,WAAD",
           "displayName": "Auditors,WAAD",
           "id": "bcad6926-67ec-445a-80f8-578032504c09",
           "isEnabled": true,
           "origin": "ServicePrincipal",
           "value": "arn:aws:iam::12345:role/Auditors,arn:aws:iam::12345:saml-provider/WAAD"
       }    ]
   }
   ```

   > [!Note]
   > Solo puede agregar nuevos roles después de agregar _msiam_access_ para la operación de revisión. Además, puede agregar tantos roles como desee según las necesidades de su organización. Azure AD envía el _valor_ de estos roles como valor de notificación en la respuesta de SAML.

   j. En el Explorador de Microsoft Graph, cambie el método de **GET** a **PATCH**. Aplique la revisión al objeto de la entidad de servicio con los roles que desee mediante la actualización de la propiedad appRoles, como se muestra en el ejemplo anterior. Seleccione **Ejecutar consulta** para ejecutar la operación de aplicación de revisión. Un mensaje de operación correcta confirma la creación del rol en la aplicación de AWS.

   ![Captura de pantalla del panel del Explorador de Microsoft Graph, con el método cambiado a PATCH.](./media/aws-multi-accounts-tutorial/graph-explorer-new11.png)

1. Una vez que haya aplicado la revisión de la entidad de servicio con más roles, puede asignar usuarios y grupos a los roles correspondientes. Puede hacerlo en Azure Portal; para ello, vaya a la aplicación de AWS y seleccione la pestaña **Usuarios y Grupos** en la parte superior.

1. Se recomienda crear un grupo nuevo para cada rol de AWS, para poder asignar dicho rol específico en el grupo. Esta asignación uno a uno significa que un grupo está asignado a un rol. Luego, puede agregar los miembros que pertenecen al grupo.

1. Después de crear los grupos, seleccione el grupo y asígnelo a la aplicación.

   ![Captura de pantalla del panel "Usuarios y grupos".](./media/aws-multi-accounts-tutorial/graph-explorer-new5.png)

   > [!Note]
   > Al asignar grupos, no se admiten los grupos anidados.

1. Para asignar el rol al grupo, seleccione el rol y, a continuación, seleccione **Asignar**.

   ![Captura de pantalla del panel "Agregar asignación".](./media/aws-multi-accounts-tutorial/graph-explorer-new6.png)

   > [!Note]
   > Después de asignar los roles, puede verlos actualizando la sesión de Azure Portal.

### <a name="test-sso"></a>Prueba de SSO

En esta sección, va a probar la configuración de inicio de sesión único de Azure AD mediante el portal Aplicaciones de Microsoft.

Al seleccionar el icono de **AWS** en Aplicaciones, se abre la página de la aplicación de AWS con una opción para seleccionar el rol.

![Captura de pantalla de la página de AWS para probar el inicio de sesión único.](./media/aws-multi-accounts-tutorial/tutorial-amazonwebservices-test-screen.png)

También puede comprobar la respuesta de SAML para ver los roles que se usan como notificaciones.

![Captura de pantalla de la respuesta de SAML.](./media/aws-multi-accounts-tutorial/tutorial-amazonwebservices-test-saml.png)

Para más información acerca del portal Aplicaciones, consulte [Inicio de sesión e inicio de aplicaciones desde el portal Aplicaciones](../user-help/my-apps-portal-end-user-access.md).

## <a name="next-steps"></a>Pasos siguientes

Una vez configurado AWS, puede aplicar el control de sesión, que protege la filtración e infiltración de la información confidencial de la organización en tiempo real. El control de sesión procede del acceso condicional. Para más información, consulte [Cómo aplicar el control de sesión con Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-aad).

<!--Image references-->

[11]: ./media/aws-multi-accounts-tutorial/ic795031.png
[12]: ./media/aws-multi-accounts-tutorial/ic795032.png
[13]: ./media/aws-multi-accounts-tutorial/ic795033.png
[14]: ./media/aws-multi-accounts-tutorial/ic795034.png
[15]: ./media/aws-multi-accounts-tutorial/ic795035.png
[16]: ./media/aws-multi-accounts-tutorial/ic795022.png
[17]: ./media/aws-multi-accounts-tutorial/ic795023.png
[18]: ./media/aws-multi-accounts-tutorial/ic795024.png
[19]: ./media/aws-multi-accounts-tutorial/ic795025.png
[32]: ./media/aws-multi-accounts-tutorial/ic7950251.png
[33]: ./media/aws-multi-accounts-tutorial/ic7950252.png
[35]: ./media/aws-multi-accounts-tutorial/tutorial-amazonwebservices-provisioning.png
[34]: ./media/aws-multi-accounts-tutorial/config3.png
[36]: ./media/aws-multi-accounts-tutorial/tutorial-amazonwebservices-securitycredentials.png
[37]: ./media/aws-multi-accounts-tutorial/tutorial-amazonwebservices-securitycredentials-continue.png
[38]: ./media/aws-multi-accounts-tutorial/tutorial-amazonwebservices-createnewaccesskey.png
[39]: ./media/aws-multi-accounts-tutorial/tutorial-amazonwebservices-provisioning-automatic.png
[40]: ./media/aws-multi-accounts-tutorial/tutorial-amazonwebservices-provisioning-testconnection.png
[41]: ./media/aws-multi-accounts-tutorial/
