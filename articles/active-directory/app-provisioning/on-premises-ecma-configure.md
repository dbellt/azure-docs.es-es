---
title: Configuración del host del conector ECMA de Azure AD
description: En este artículo se describe cómo configurar el host del conector ECMA de Azure AD.
services: active-directory
author: billmath
manager: mtillman
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 06/04/2021
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: cad38ff8c0dc18d363773783e95e544d67f55193
ms.sourcegitcommit: ff1aa951f5d81381811246ac2380bcddc7e0c2b0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/07/2021
ms.locfileid: "111570455"
---
# <a name="configure-the-azure-ad-ecma-connector-host-and-the-provisioning-agent"></a>Configure el host del conector ECMA de Azure AD y el agente de aprovisionamiento.

>[!IMPORTANT]
> La versión preliminar de aprovisionamiento local se encuentra actualmente en una versión preliminar solo por invitación. Puede solicitar acceso a la funcionalidad [aquí](https://aka.ms/onpremprovisioningpublicpreviewaccess). Abriremos la versión preliminar para más clientes y conectores durante los próximos meses a medida que nos preparamos para la disponibilidad general.

En este artículo se proporciona una guía sobre cómo configurar el host del conector ECMA de Azure AD y el agente de aprovisionamiento una vez que los ha instalado correctamente.

La instalación y configuración del host del conector ECMA de Azure AD es un proceso.  Use el flujo siguiente como guía para el proceso.

 ![Flujo de instalación](./media/on-premises-ecma-configure/flow-1.png)  

Para más información sobre la instalación y configuración, consulte:
   - [Requisitos previos del host del conector ECMA de Azure AD](on-premises-ecma-prerequisites.md)
   - [Instalación del host del conector ECMA de Azure AD](on-premises-ecma-install.md)
    - [Configuración del conector SQL genérico del host del conector ECMA de Azure AD](on-premises-sql-connector-configure.md)
## <a name="configure-the-azure-ad-ecma-connector-host"></a>Configuración del host del conector ECMA de Azure AD
La configuración del host del conector ECMA de Azure AD se produce en dos partes.
    
   - **Configuración de los valores**: Configure el puerto y el certificado del host del conector ECMA de Azure AD que se usará.  Esto solo se hace la primera vez que se inicia el host del conector ECMA.
   - **Crear un conector**: Cree un conector (por ejemplo, SQL o LDAP) para permitir que el host del conector ECMA de Azure AD exporte o importe datos a un origen de datos.

### <a name="configure-the-settings"></a>Configuración de los valores
La primera vez que inicie el host del conector ECMA de Azure AD verá un número de puerto que ya se habrá rellenado con el valor predeterminado 8585.  

 ![Configuración de los valores](.\media\on-premises-ecma-configure\configure-1.png)

Para la versión preliminar, tendrá que generar un nuevo certificado autofirmado.

 >[!NOTE]
 >Esta versión preliminar usa un certificado sujeto a limitación temporal. El certificado generado automáticamente estará autofirmado, parte de la raíz de confianza y la SAN coincide con el nombre de host.


### <a name="create-a-connector"></a>Creación de un conector
Ahora tiene que crear un conector para su uso por parte del host del conector ECMA de Azure AD.  Este conector permitirá que el host del conector ECMA exporte (e importe, si lo quiere) datos al origen de datos para el conector que cree.  

Los pasos de configuración para cada uno de los conectores individuales son más largos y se incluyen en sus propios documentos.

Siga uno de los vínculos siguientes para crear y configurar un conector.

- [Conector SQL genérico](on-premises-sql-connector-configure.md): un conector que funcionará con bases de datos SQL, como Microsoft SQL o MySQL.


## <a name="establish-connectivity-between-azure-ad-and-the-azure-ad-ecma-connector-host"></a>Establecimiento de conectividad entre Azure AD y el host del conector ECMA de Azure AD
En las secciones siguientes encontrará una guía para establecer la conectividad con el host del conector ECMA de Azure AD local y Azure AD.

#### <a name="ensure-ecma2host-service-is-running"></a>Comprobación de que el servicio ECMA2Host se está ejecutando
1.  En el servidor en el que se ejecuta el host del conector ECMA de Azure AD, haga clic en Inicio.
2. Escriba run y, luego, services.msc en el cuadro.
3. En los servicios, asegúrese de que **Microsoft ECMA2Host** esté presente y en ejecución.  Si no es así, haga clic en **Iniciar**.
 ![El servicio está en ejecución](.\media\on-premises-ecma-configure\configure-2.png)

#### <a name="add-enterprise-application"></a>Incorporación de una aplicación empresarial
1.  Inicie sesión en Azure Portal como administrador de la aplicación.
2. En el portal, vaya a Azure Active Directory, **Aplicaciones empresariales**.
3. Haga clic en **Nueva aplicación**.
 ![Agregar nueva aplicación](.\media\on-premises-ecma-configure\configure-4.png)
4. Busque la aplicación "Aprovisionamiento local" en la galería y haga clic en **Crear**.

### <a name="configure-the-application-and-test"></a>Configuración de la aplicación y prueba
  1. Una vez que se haya creado, haga clic en la **página Aprovisionamiento**.
  2. Haga clic en **Introducción**.
 ![introducción](.\media\on-premises-ecma-configure\configure-6.png)
  3. En la **página Aprovisionamiento**, cambie el modo a **Automático**
   ![Cambio de modo](.\media\on-premises-ecma-configure\configure-7.png)
  4. En la sección Conectividad local, seleccione el agente que acaba de implementar y haga clic en Asignar agentes.
     ![Asignación de un agente](.\media\on-premises-ecma-configure\configure-8.png)</br>

  >[!NOTE]
  >Después de agregar el  agente, debe esperar de 10 a 20 minutos para que se complete el registro.  La prueba de conectividad no funcionará hasta que se complete el registro.
  >
  >Como alternativa, puede reiniciar el agente de aprovisionamiento en el servidor para forzar que el registro del agente se complete. Vaya al servidor, busque los servicios en la barra de búsqueda de Windows, busque el servicio del agente de aprovisionamiento de Azure AD Connect, haga clic con el botón derecho en el servicio y reinicie.
  

  5.  Después de 10 minutos, en la sección **Credenciales de administrador**, escriba la siguiente dirección URL y reemplace la parte "connectorName" por el nombre del conector en el host ECMA.

      |Propiedad|Valor|
      |-----|-----|
      |URL de inquilino|https://localhost:8585/ecma2host_connectorName/scim|

  6. Escriba el valor del token secreto que definió al crear el conector.
  7. Haga clic en Probar conexión y espere un minuto.
     ![Comprobación de la conexión](.\media\on-premises-ecma-configure\configure-5.png)

     >[!NOTE]
     >Asegúrese de esperar de 10 a 20 minutos después de asignar el agente para probar la conexión.  Se producirá un error en la conexión si no se ha completado el registro.
  8. Una vez que la prueba de conexión se complete correctamente, haga clic en **Guardar**.</br>
   ![Prueba correcta](.\media\on-premises-ecma-configure\configure-9.png)

## <a name="configure-who-is-in-scope-for-provisioning"></a>Configuración de quién está en el ámbito de aprovisionamiento
Ahora que tiene el host del conector ECMA de Azure AD en comunicación con Azure AD, puede pasar a configurar quién está dentro del ámbito del aprovisionamiento.  En las secciones siguientes encontrará información sobre definir el ámbito de los usuarios.

### <a name="assign-users-to-your-application"></a>Asignación de usuarios a la aplicación
Azure AD permite establecer el ámbito de quién se debe aprovisionar en función de la asignación a una aplicación o mediante el filtrado por un atributo determinado. Decida quién debe estar en el ámbito del aprovisionamiento y defina las reglas de limitación del ámbito según sea necesario.  Para más información, consulte [Administración de la asignación de usuarios para una aplicación en Azure Active Directory](../../active-directory/manage-apps/assign-user-or-group-access-portal.md).

### <a name="configure-your-attribute-mappings"></a>Configuración de las asignaciones de atributos
Deberá asignar los atributos de usuario en Azure AD a los atributos de la aplicación de destino. El servicio de aprovisionamiento de Azure AD se basa en el estándar SCIM para el aprovisionamiento y, como consecuencia, los atributos expuestos tienen el espacio de nombres de SCIM. En el ejemplo siguiente se muestra cómo puede asignar los atributos mail y objectId de Azure AD a los atributos Email e InternalGUID de una aplicación. 

>[!NOTE]
>La asignación predeterminada contiene userPrincipalName para un nombre de atributo PLACEHOLDER. Tendrá que cambiar el atributo PLACEHOLDER a uno que se encuentre en la aplicación. Para más información, consulte [Emparejamiento de usuarios en los sistemas de origen y destino](customize-application-attributes.md#matching-users-in-the-source-and-target--systems).

|Nombre del atributo en Azure AD|Nombre de atributo en SCIM|Nombre de atributo en la aplicación de destino|
|-----|-----|-----|
|mail|urn:ietf:params:scim:schemas:extension:ECMA2Host:2.0:User:Email|Email|
|objectId|urn:ietf:params:scim:schemas:extension:ECMA2Host:2.0:User:InternalGUID|InternalGUID|

#### <a name="configure-attribute-mapping"></a>Configuración de la asignación de atributos
 1. En el portal de Azure AD, en **Aplicaciones empresariales**, haga clic en la **página Aprovisionamiento**.
 2. Haga clic en **Introducción**.
 3. Expanda **Asignaciones y** haga clic en **Provision Azure Active Directory Users** (Aprovisionar usuarios de Azure Active Directory)
   ![Aprovisionamiento de un usuario](.\media\on-premises-ecma-configure\configure-10.png)
 4. Haga clic en **Agregar nueva asignación**
   ![Adición de una asignación](.\media\on-premises-ecma-configure\configure-11.png)
 5. Especifique los atributos de origen y destino, y haga clic en **Aceptar**.</br>
   ![Edición de atributos](.\media\on-premises-ecma-configure\configure-12.png)


Para más información sobre la asignación de atributos de usuario de aplicaciones a Azure AD, consulte [Tutorial: Personalización de las asignaciones de atributos de aprovisionamiento de usuarios para aplicaciones SaaS en Azure Active Directory](customize-application-attributes.md).

### <a name="test-your-configuration-by-provisioning-users-on-demand"></a>Prueba de la configuración mediante el aprovisionamiento de usuarios a petición
Para probar la configuración, puede usar el aprovisionamiento de usuarios a petición.  Para más información sobre el aprovisionamiento de usuarios a petición, consulte [Aprovisionamiento a petición](provision-on-demand.md).

 1. Vaya a la hoja de inicio de sesión único y, luego, vuelva a la hoja de aprovisionamiento. En la nueva hoja de información general de aprovisionamiento, haga clic en A petición.
 2. Pruebe el aprovisionamiento de algunos usuarios a petición como se describe [aquí](provision-on-demand.md).
   ![Prueba del aprovisionamiento](.\media\on-premises-ecma-configure\configure-13.png)

### <a name="start-provisioning-users"></a>Inicio del aprovisionamiento de usuarios
 1. Una vez que el aprovisionamiento a petición se complete correctamente, vuelva a la página de configuración del aprovisionamiento. Asegúrese de que el ámbito esté establecido solo en los usuarios y grupos asignados, **active el aprovisionamiento** y haga clic en **Guardar**.
   ![Iniciar aprovisionamiento](.\media\on-premises-ecma-configure\configure-14.png)
  2.  Espere varios minutos para que se inicie el aprovisionamiento (puede tardar hasta 40 minutos). Para más información sobre el rendimiento del servicio de aprovisionamiento, consulte aquí. Una vez completado el trabajo de aprovisionamiento, como se describe en la sección siguiente, puede desactivar el estado de aprovisionamiento y hacer clic en Guardar. Esto impedirá que el servicio de aprovisionamiento se ejecute en el futuro.

### <a name="verify-users-have-been-successfully-provisioned"></a>Comprobación de que los usuarios se han aprovisionado correctamente
Después de esperar, compruebe el origen de datos para ver si se están aprovisionando nuevos usuarios.
 ![Comprobación de que los usuarios se están aprovisionando](.\media\on-premises-ecma-configure\configure-15.png)

## <a name="monitor-your-deployment"></a>Supervisión de la implementación

1. Use los registros de aprovisionamiento para determinar qué usuarios se han aprovisionado correctamente y los que no.
2. Cree alertas, paneles y consultas personalizados mediante la integración de Azure Monitor.
3. Si la configuración de aprovisionamiento parece estar en mal estado, la aplicación pasará a estar en cuarentena. Más información sobre los estados de cuarentena aquí.

## <a name="next-steps"></a>Pasos siguientes

- [Requisitos previos del host del conector ECMA de Azure AD](on-premises-ecma-prerequisites.md)
- [Instalación del host del conector ECMA de Azure AD](on-premises-ecma-install.md)
- [Conector SQL genérico](on-premises-sql-connector-configure.md)