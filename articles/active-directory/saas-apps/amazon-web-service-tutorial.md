---
title: 'Tutorial: Integración del inicio de sesión único de Azure Active Directory (SSO) con AWS Single-Account Access | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y AWS Single-Account Access.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 03/05/2021
ms.author: jeedes
ms.openlocfilehash: cb8ae273dbe9f7b26cafe6b291a0fd6b5e4ef093
ms.sourcegitcommit: 2cb7772f60599e065fff13fdecd795cce6500630
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108802205"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-aws-single-account-access"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con AWS Single-Account Access

En este tutorial, aprenderá a integrar AWS Single-Account Access con Azure Active Directory (Azure AD). Al integrar AWS Single-Account Access con Azure AD, podrá:

* Controlar en Azure AD quién tiene acceso a AWS Single-Account Access.
* Permitir que los usuarios inicien sesión automáticamente en AWS Single-Account Access con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

## <a name="understanding-the-different-aws-applications-in-the-azure-ad-application-gallery"></a>Descripción de las distintas aplicaciones de AWS de la galería de aplicaciones de Azure AD
Use la información siguiente para decidir entre el uso de las aplicaciones AWS Single Sign-On y AWS Single-Account Access de la galería de aplicaciones de Azure AD.

**AWS Single Sign-On**

[AWS Single Sign-On](./aws-single-sign-on-tutorial.md) se agregó a la galería de aplicaciones de Azure AD en febrero de 2021. Esta aplicación facilita la administración del acceso de forma centralizada a varias cuentas y aplicaciones AWS, con inicio de sesión mediante Microsoft Azure AD. Federe Microsoft Azure AD con AWS SSO una vez y use AWS SSO para administrar los permisos de todas las cuentas de AWS desde un solo lugar. AWS SSO aprovisiona automáticamente los permisos y los mantiene actualizados a medida que se actualizan las directivas y se obtiene acceso a las asignaciones. Los usuarios finales pueden autenticarse con sus credenciales de Azure AD para acceder a la consola de AWS, a la interfaz de la línea de comandos y a las aplicaciones integradas con AWS SSO.

**AWS Single-Account Access**

Los clientes han usado [AWS Single-Account Access]() durante los últimos años. Esta aplicación permite federar Azure AD con una sola cuenta de AWS y usar Azure AD para administrar el acceso a los roles de AWS IAM. Los administradores de AWS IAM definen roles y directivas en cada cuenta de AWS. En cada una de estas cuentas, los administradores de Azure AD se federan con AWS IAM, asignan usuarios o grupos a la cuenta y configuran Azure AD para enviar aserciones que autoricen el acceso a los roles.  

| Característica | AWS Single Sign-On | AWS Single-Account Access |
|:--- |:---:|:---:|
|Acceso condicional| Admite una única directiva de acceso condicional para todas las cuentas de AWS. | Admite una única directiva de acceso condicional para todas las cuentas o directivas personalizadas por cuenta.|
| Acceso a la CLI | Compatible | Compatible|
| Privileged Identity Management | Todavía no se admite | Todavía no se admite |
| Administración de cuentas centralizada | Centralice la administración de cuentas en AWS. | Centralice la administración de cuentas en Azure AD (probablemente haga falta una aplicación empresarial de Azure AD por cuenta). |
| Certificado SAML| Un solo certificado| Certificados diferentes por aplicación o cuenta | 

## <a name="aws-single-account-access-architecture"></a>Arquitectura de AWS Single-Account Access
![Diagrama de la relación de Azure AD y AWS](./media/amazon-web-service-tutorial/tutorial_amazonwebservices_image.png)

Puede configurar varios identificadores para varias instancias. Por ejemplo:

* `https://signin.aws.amazon.com/saml#1`

* `https://signin.aws.amazon.com/saml#2`

Con estos valores, Azure AD quitará el valor de **#** y enviará el valor correcto `https://signin.aws.amazon.com/saml` como dirección URL del público en el token de SAML.

Se recomienda este enfoque por las siguientes razones:

- Cada aplicación le proporciona un certificado X509 único. Cada instancia de aplicación de AWS puede tener una fecha de expiración del certificado diferente que se puede administrar de forma individual en cada cuenta de AWS. En este caso, la sustitución general del certificado es sencilla.

- Puede habilitar el aprovisionamiento de usuarios con la aplicación AWS en Azure AD y, luego, nuestro servicio capturará todos los roles de esa cuenta de AWS. No tiene que agregar ni actualizar manualmente los roles de AWS en la aplicación.

- Puede asignar el propietario de la aplicación de forma individual para la aplicación. Esta persona puede administrar la aplicación directamente en Azure AD.

> [!Note]
> Asegúrese de usar solo una aplicación de la galería.

## <a name="prerequisites"></a>Prerrequisitos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único (SSO) en AWS.

> [!Note]
> Los roles no deben editarse manualmente en Azure AD al realizar importaciones de roles.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* AWS Single-Account Access admite SSO iniciado por **SP e IDP**.

> [!NOTE]
> El identificador de esta aplicación es un valor de cadena fijo, por lo que solo se puede configurar una instancia en un inquilino.

## <a name="adding-aws-single-account-access-from-the-gallery"></a>Incorporación de AWS Single-Account Access desde la galería

Para configurar la integración de AWS Single-Account Access en Azure AD, será preciso que agregue AWS Single-Account Access desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en Azure Portal con una cuenta personal, profesional o educativa de Microsoft.
1. En Azure Portal, busque y seleccione **Azure Active Directory**.
1. En el menú de información general de Azure Active Directory, elija **Aplicaciones empresariales** > **Todas las aplicaciones**.
1. Para agregar una aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **AWS Single-Account Access** en el cuadro de búsqueda.
1. Seleccione **AWS Single-Account Access** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-sso-for-aws-single-account-access"></a>Configuración y prueba del inicio de sesión único de Azure AD para AWS Single-Account Access

Configure y pruebe el inicio de sesión único de Azure AD con AWS Single-Account Access mediante un usuario de prueba llamado **B. Simon**. Para que el inicio de sesión único funcione, es preciso establecer una relación de vinculación entre un usuario de Azure AD y el usuario relacionado de AWS Single-Account Access.

Para configurar y probar el inicio de sesión único de Azure AD con AWS Single-Account Access, siga los pasos a continuación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único de AWS Single-Account Access](#configure-aws-single-account-access-sso)** , para configurar los valores de Inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba de AWS Single-Account Access](#create-aws-single-account-access-test-user)** , para tener un homólogo de B. Simon en AWS Single-Account Access que esté vinculado a la representación del usuario en Azure AD.
    1. **[Cómo configurar el aprovisionamiento de roles en AWS Single-Account Access](#how-to-configure-role-provisioning-in-aws-single-account-access)**
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En Azure Portal, en la página de integración de la aplicación **AWS Single-Account Access**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono de edición o con forma de lápiz para abrir el cuadro de diálogo **Configuración básica de SAML** y modificar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, actualice los valores de **Identifier (Entity ID)** (Identificador [id. de entidad]) y **URL de respuesta** con el mismo valor predeterminado: `https://signin.aws.amazon.com/saml`. Seleccione **Guardar** para guardar los cambios de configuración.

1. Cuando se configure más de una instancia, proporcione un valor de identificador. A partir de la segunda instancia, use el formato siguiente, incluyendo el símbolo **#** para especificar un valor de SPN único.

    `https://signin.aws.amazon.com/saml#2`

1. La aplicación AWS espera las aserciones de SAML en un formato específico, que requiere que se agreguen asignaciones de atributos personalizados a la configuración de los atributos del token de SAML. La siguiente captura de muestra la lista de atributos predeterminados.

    ![imagen](common/default-attributes.png)

1. Además de lo anterior, la aplicación AWS espera que se devuelvan algunos atributos más, que se muestran a continuación, en la respuesta de SAML. Estos atributos también se rellenan previamente, pero puede revisarlos según sus requisitos.
    
    | Nombre  | Atributo de origen  | Espacio de nombres |
    | --------------- | --------------- | --------------- |
    | RoleSessionName | user.userprincipalname | `https://aws.amazon.com/SAML/Attributes` |
    | Role | user.assignedroles |  `https://aws.amazon.com/SAML/Attributes` |
    | SessionDuration | "proporcione un valor comprendido entre 900 segundos (15 minutos) y 43200 segundos (12 horas)" |  `https://aws.amazon.com/SAML/Attributes` |

    > [!NOTE]
    > AWS espera roles para los usuarios asignados a la aplicación. Configure estos roles en Azure AD para que se puedan asignar los roles correspondientes a los usuarios. Para aprender a configurar roles en Azure AD, consulte [este vínculo](../develop/howto-add-app-roles-in-azure-ad-apps.md#app-roles-ui).

1. En la página **Configuración del inicio de sesión único con SAML**, en el cuadro de diálogo **Certificado de firma de SAML** (paso 3), seleccione **Agregar un certificado**.

    ![Creación de un certificado de SAML](common/add-saml-certificate.png)

1. Genere un nuevo certificado de firma de SAML y, luego, seleccione **Nuevo certificado**. Escriba una dirección de correo electrónico para las notificaciones del certificado.
   
    ![Nuevo certificado de SAML](common/new-saml-certificate.png) 

1. En la sección **Certificado de firma de SAML**, busque **Federation Metadata XML** (Archivo XML de metadatos de federación) y seleccione **Descargar** para descargar el certificado y guardarlo en el equipo.

    ![Vínculo de descarga del certificado](./media/amazon-web-service-tutorial/certificate.png)

1. En la sección **Set up AWS Single-Account Access** (configurar AWS Single-Account Access), copie las direcciones URL adecuadas según sus necesidades.

    ![Copiar direcciones URL de configuración](common/copy-configuration-urls.png)

### <a name="create-an-azure-ad-test-user"></a>Creación de un usuario de prueba de Azure AD

En esta sección, va a crear un usuario de prueba llamado B.Simon en Azure Portal.

1. En Azure Portal, busque y seleccione **Azure Active Directory**.
1. En el menú de información general de Azure Active Directory, elija **Usuarios** > **Todos los usuarios**.
1. Seleccione **Nuevo usuario** en la parte superior de la pantalla.
1. En las propiedades del **usuario**, siga estos pasos:
   1. En el campo **Nombre**, escriba `B.Simon`.  
   1. En el campo **Nombre de usuario**, escriba username@companydomain.extension. Por ejemplo, `B.Simon@contoso.com`.
   1. Active la casilla **Show password** (Mostrar contraseña) y, después, anote el valor que se muestra en el cuadro **Contraseña**.
   1. Haga clic en **Crear**.

### <a name="assign-the-azure-ad-test-user"></a>Asignación del usuario de prueba de Azure AD

En esta sección, va a conceder a B. Simon acceso a AWS Single-Account Access a través del inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **AWS Single-Account Access**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.
1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.
1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que se asigne un rol a los usuarios, puede seleccionarlo en la lista desplegable **Seleccionar un rol**. Si no se ha configurado ningún rol para esta aplicación, verá seleccionado el rol "Acceso predeterminado".
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-aws-single-account-access-sso"></a>Configuración del inicio de sesión único de AWS Single-Account Access

1. En otra ventana del explorador, inicie sesión en su sitio de la compañía de AWS como administrador.

2. Seleccione **AWS Home** (Página principal de AWS).

    ![Captura de pantalla del sitio de la compañía de AWS, con el icono de la página principal de AWS resaltado][11]

3. Seleccione **Identity and Access Management** (Administración de identidades y accesos).

    ![Captura de pantalla de la página de servicios de AWS con la opción IAM resaltada][12]

4. Seleccione **Identity Providers** > **Create Provider** (Proveedores de identidades > Crear proveedor).

    ![Captura de pantalla de la página de IAM, con las opciones Identity Providers (Proveedores de identidades) y Create Provider (Crear proveedor) resaltadas][13]

5. En la página **Configure Provider** (Configurar proveedor), realice los pasos siguientes:

    ![Captura de pantalla de Configure Provider (Configurar proveedor)][14]

    a. En **Provider Type** (Tipo de proveedor), seleccione **SAML**.

    b. En **Provider Name** (Nombre de proveedor), escriba un nombre de proveedor (por ejemplo, *WAAD*).

    c. Para cargar el **archivo de metadatos** descargado de Azure Portal, seleccione **Choose file** (Elegir archivo).

    d. Seleccione **Next Step** (Siguiente paso).

6. En la página **Verify Provider Information** (Comprobar la información del proveedor), seleccione **Create** (Crear).

    ![Captura de pantalla de Verify Provider Information (Comprobar la información del proveedor), con la opción Create (Crear) resaltada][15]

7. Seleccione **Roles** > **Create role** (Crear rol).

    ![Captura de pantalla de la página Roles][16]

8. En la página **Crear rol**, realice los pasos siguientes:  

    ![Captura de pantalla de la página Create role (Crear rol)][19]

    a. En **Select type of trusted entity** (Seleccionar tipo de entidad de confianza), seleccione **SAML 2.0 federation** (Federación de SAML 2.0).

    b. En **Choose a SAML 2.0 Provider** (Elegir un proveedor de SAML 2.0), seleccione el **proveedor de SAML** que ha creado anteriormente (por ejemplo, *WAAD*).

    c. Seleccione **Allow programmatic and AWS Management Console access** (Permitir acceso mediante programación y a consola de AWS Management Console).
  
    d. Seleccione **Siguiente: Permisos**.

9. En el cuadro de diálogo **Attach Permissions Policies** (Adjuntar directivas de permisos), adjunte la directiva adecuada según su organización. Después, seleccione **Next: Review** (Siguiente: revisar).  

    ![Captura de pantalla del cuadro de diálogo Attach Permissions Policies (Adjuntar directivas de permisos)][33]

10. En el cuadro de diálogo **Review** (Revisar), realice los pasos siguientes:

    ![Captura de pantalla del cuadro de diálogo Review (Revisar)][34]

    a. En **Role name** (Nombre de rol), escriba el nombre de su rol.

    b. En **Role description** (Descripción del rol), escriba la descripción.

    c. Seleccione **Create Role** (Crear rol).

    d. Cree tantos roles como sea necesario y asígnelos al proveedor de identidades.

11. Use credenciales de cuenta de servicio de AWS para obtener los roles de cuenta de AWS en aprovisionamiento de usuarios de Azure AD. Para ello, abra la página principal de la consola de AWS.

12. Seleccione **Servicios**. En **Security, Identity & Compliance** (Seguridad, identidad y cumplimiento), seleccione **IAM**.

    ![Captura de pantalla de la página principal de la consola de AWS, con las opciones Services (Servicios) e IAM resaltadas](./media/amazon-web-service-tutorial/fetchingrole1.png)

13. En la sección IAM, seleccione **Policies** (Directivas).

    ![Captura de pantalla de la sección IAM, con la pestaña Policies (Directivas) resaltada](./media/amazon-web-service-tutorial/fetchingrole2.png)

14. Para crear una nueva directiva, seleccione **Create policy** (Crear directiva) para recuperar los roles de la cuenta de AWS de aprovisionamiento de usuarios de Azure AD.

    ![Captura de pantalla de la página Create rol (Crear rol), con la opción Create policy (Crear directiva) resaltada](./media/amazon-web-service-tutorial/fetchingrole3.png)

15. Cree su propia directiva para obtener todos los roles de cuentas de AWS.

    ![Captura de pantalla de la página Create policy (Crear directiva), con la opción JSON resaltada](./media/amazon-web-service-tutorial/policy1.png)

    a. En **Create policy** (Crear directiva), seleccione la pestaña **JSON**.

    b. En el documento de directiva, agregue el siguiente JSON:

    ```json
    {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Effect": "Allow",
                "Action": [
                "iam:ListRoles"
                ],
                "Resource": "*"
            }
        ]
    }
    ```

    c. Seleccione **Review policy** (Revisar directiva) para validar la directiva.

    ![Captura de pantalla de la página Create policy (Crear directiva)](./media/amazon-web-service-tutorial/policy5.png)

16. Defina la nueva directiva.

    ![Captura de pantalla de la página Create policy (Crear directiva), con los campos de nombre y descripción resaltados](./media/amazon-web-service-tutorial/policy2.png)

    a. En **Name** (Nombre), escriba **AzureAD_SSOUserRole_Policy**.

    b. En **Description** (Descripción), escriba **Esta directiva permitirá obtener los roles de cuentas de AWS**.

    c. Seleccione **Crear una directiva**.

17. Cree una nueva cuenta de usuario en el servicio IAM de AWS.

    a. En la consola de IAM de AWS, seleccione **Users** (Usuarios).

    ![Captura de pantalla de la consola IAM de AWS, con la opción Users (Usuarios) resaltada](./media/amazon-web-service-tutorial/policy3.png)

    b. Para crear un nuevo usuario, seleccione **Add user** (Agregar usuario).

    ![Captura de pantalla del botón Add user (Agregar usuario)](./media/amazon-web-service-tutorial/policy4.png)

    c. En la sección **Add user** (Agregar usuario):

    ![Captura de pantalla de la página Add user (Agregar usuario), con las opciones de nombre de usuario y tipo de acceso resaltadas](./media/amazon-web-service-tutorial/adduser1.png)

    * Escriba el nombre de usuario como **AzureADRoleManager**.

    * Para el tipo de acceso, seleccione **Programmatic access** (Acceso mediante programación). De este modo, el usuario puede llamar a las API y obtener los roles de la cuenta de AWS.

    * Seleccione **Next Permissions** (Siguientes permisos).

18. Cree una nueva directiva para este usuario.

    ![Captura de pantalla que muestra la página Add user (Agregar usuario) para crear una directiva para el usuario.](./media/amazon-web-service-tutorial/adduser2.png)

    a. Seleccione **Attach existing policies directly** (Asociar directivas existentes directamente).

    b. Busque la directiva recién creada en la sección de filtro **AzureAD_SSOUserRole_Policy**.

    c. Seleccione la directiva y, a continuación, seleccione **Next: Review** (Siguiente: revisar).

19. Revise la directiva del usuario asociado.

    ![Captura de pantalla de la página Add user (Agregar usuario), con la opción Create user (crear usuario) resaltada](./media/amazon-web-service-tutorial/adduser3.png)

    a. Revise el nombre de usuario, el tipo de acceso y la directiva asociada al usuario.

    b. Seleccione **Create User** (Crear usuario).

20. Descargue las credenciales de usuario de un usuario.

    ![Captura de pantalla que muestra la página Add user (Agregar usuario) con un botón Download c s v (Descargar CSV) para obtener las credenciales del usuario.](./media/amazon-web-service-tutorial/adduser4.png)

    a. Copie el **identificador de la clave de acceso** y la **clave de acceso secreta** del usuario.

    b. Escriba estas credenciales en la sección de aprovisionamiento de usuarios de Azure AD para obtener los roles de la consola de AWS.

    c. Seleccione **Cerrar**.

### <a name="how-to-configure-role-provisioning-in-aws-single-account-access"></a>Cómo configurar el aprovisionamiento de roles en AWS Single-Account Access

1. En el portal de administración de Azure AD, en la aplicación AWS, vaya a **Aprovisionamiento**.

    ![Captura de pantalla de la aplicación AWS, con la opción Aprovisionamiento resaltada](./media/amazon-web-service-tutorial/provisioning.png)

2. Escriba la clave de acceso y la clave secreta en los campos **Secreto de cliente** y **Token secreto** respectivamente.

    ![Captura de pantalla del cuadro de diálogo Credenciales de administrador](./media/amazon-web-service-tutorial/provisioning1.png)

    a. Escriba la clave de acceso de usuario de AWS en el campo **Secreto de cliente**.

    b. Escriba el secreto de usuario de AWS en el campo **Token secreto**.

    c. Seleccione **Test Connection** (Probar conexión).

    d. Seleccione **Guardar** para guardar la configuración.

3. En la sección **Configuración**, en **Estado de aprovisionamiento**, seleccione **Activado**. Después, seleccione **Guardar**.

    ![Captura de pantalla de la sección Configuración, con la opción Activado resaltada](./media/amazon-web-service-tutorial/provisioning2.png)

> [!NOTE]
> El servicio de aprovisionamiento solo importa roles de AWS en Azure AD. El servicio no aprovisiona los usuarios y grupos de Azure AD en AWS.

> [!NOTE]
> Después de guardar las credenciales de aprovisionamiento, debe esperar a que se ejecute el ciclo de sincronización inicial. Normalmente, la sincronización tarda aproximadamente 40 minutos en finalizar. Puede ver el estado en la parte inferior de la página **Aprovisionamiento**, en **Estado actual**.

### <a name="create-aws-single-account-access-test-user"></a>Creación de un usuario de prueba de AWS Single-Account Access

El objetivo de esta sección es crear un usuario llamado B. Simon en AWS Single-Account Access. AWS Single-Account Access no necesita que se cree un usuario en su sistema para el inicio de sesión único, por lo que no es necesario realizar ninguna acción aquí.

## <a name="test-sso"></a>Prueba de SSO

En esta sección, probará la configuración de inicio de sesión único de Azure AD con las siguientes opciones. 

#### <a name="sp-initiated"></a>Iniciado por SP:

* Haga clic en **Probar esta aplicación** en Azure Portal. Esta acción le redirigirá a la dirección URL de inicio de sesión de AWS Single-Account Access, donde puede iniciar el flujo de inicio de sesión.  

* Vaya directamente a la dirección URL de inicio de sesión de AWS Single-Account Access e inicie el flujo de inicio de sesión desde allí.

#### <a name="idp-initiated"></a>Iniciado por IDP:

* Haga clic en **Probar esta aplicación** en Azure Portal; debería iniciar sesión automáticamente en la instancia de AWS Single-Account Access para la que configuró el inicio de sesión único. 

También puede usar Aplicaciones de Microsoft para probar la aplicación en cualquier modo. Al hacer clic en el icono de AWS Single-Account Access en Mis aplicaciones, si se ha configurado en modo SP, se le redirigirá a la página de inicio de sesión de la aplicación para comenzar el flujo de inicio de sesión; y si se ha configurado en modo IDP, se debería iniciar sesión automáticamente en la instancia de AWS Single-Account Access para la que configuró el inicio de sesión único. Para más información acerca de Aplicaciones, consulte [Inicio de sesión e inicio de aplicaciones desde el portal Aplicaciones](../user-help/my-apps-portal-end-user-access.md).


## <a name="known-issues"></a>Problemas conocidos

* La integración del aprovisionamiento de AWS Single-Account Access solo se puede usar para conectarse a puntos de conexión de nube pública de AWS. No se puede usar dicha integración para acceder a entornos de AWS para el sector gubernamental.
 
* En la sección **Aprovisionamiento**, en la subsección **Asignaciones** aparece un mensaje del tipo "Cargando..." y no se muestran las asignaciones de atributos. El único flujo de trabajo de aprovisionamiento que se admite actualmente es la importación de roles desde AWS a Azure AD para su selección durante la asignación de usuarios o grupos. Las asignaciones de atributos para esto vienen predeterminadas y no se pueden configurar.

* La sección **Aprovisionamiento** solo admite escribir un conjunto de credenciales para un inquilino de AWS cada vez. Todos los roles importados se escriben en la propiedad `appRoles` del [objeto `servicePrincipal`](/graph/api/resources/serviceprincipal) de Azure AD para el inquilino de AWS.

  Se pueden agregar varios inquilinos de AWS (representados por `servicePrincipals`) a Azure AD desde la galería para el aprovisionamiento. Sin embargo, existe un problema conocido que impide escribir automáticamente todos los roles importados desde los diversos objetos `servicePrincipals` de AWS usados para el aprovisionamiento en el único objeto `servicePrincipal` usado para SSO.

  Una posible solución alternativa consiste en usar [Microsoft Graph API](/graph/api/resources/serviceprincipal) para extraer todos los objetos `appRoles` importados en cada objeto `servicePrincipal` de AWS en los que esté configurado el aprovisionamiento. Posteriormente, puede agregar estas cadenas de roles a objeto `servicePrincipal` de AWS donde se configura el inicio de sesión único.

* Los roles deben cumplir los siguientes requisitos para que se puedan importar desde AWS en Azure AD:

  * Los roles deben tener exactamente un proveedor SAML definido en AWS
  * La longitud combinada del ARN (nombre del recurso de Amazon) del rol y el ARN para el proveedor de SAML asociado debe ser de 240 caracteres como máximo.

## <a name="change-log"></a>Registro de cambios

* 01/12/2020: mayor límite de longitud de rol, de 119 a 239 caracteres. 

## <a name="next-steps"></a>Pasos siguientes

Una vez que haya configurado AWS Single-Account Access, podrá aplicar el control de sesión, que protege su organización en tiempo real frente a la filtración e infiltración de información confidencial. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-aad).


[11]: ./media/amazon-web-service-tutorial/ic795031.png
[12]: ./media/amazon-web-service-tutorial/ic795032.png
[13]: ./media/amazon-web-service-tutorial/ic795033.png
[14]: ./media/amazon-web-service-tutorial/ic795034.png
[15]: ./media/amazon-web-service-tutorial/ic795035.png
[16]: ./media/amazon-web-service-tutorial/ic795022.png
[17]: ./media/amazon-web-service-tutorial/ic795023.png
[18]: ./media/amazon-web-service-tutorial/ic795024.png
[19]: ./media/amazon-web-service-tutorial/ic795025.png
[32]: ./media/amazon-web-service-tutorial/ic7950251.png
[33]: ./media/amazon-web-service-tutorial/ic7950252.png
[35]: ./media/amazon-web-service-tutorial/tutorial_amazonwebservices_provisioning.png
[34]: ./media/amazon-web-service-tutorial/ic7950253.png
[36]: ./media/amazon-web-service-tutorial/tutorial_amazonwebservices_securitycredentials.png
[37]: ./media/amazon-web-service-tutorial/tutorial_amazonwebservices_securitycredentials_continue.png
[38]: ./media/amazon-web-service-tutorial/tutorial_amazonwebservices_createnewaccesskey.png
[39]: ./media/amazon-web-service-tutorial/tutorial_amazonwebservices_provisioning_automatic.png
[40]: ./media/amazon-web-service-tutorial/tutorial_amazonwebservices_provisioning_testconnection.png
[41]: ./media/amazon-web-service-tutorial/tutorial_amazonwebservices_provisioning_on.png