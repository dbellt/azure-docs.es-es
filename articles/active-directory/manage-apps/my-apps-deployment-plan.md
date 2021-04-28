---
title: Planeamiento de la configuración de Aplicaciones de Azure Active Directory
description: Guía de planeamiento para usar eficazmente mis aplicaciones en su organización.
services: active-directory
author: barbaraselden
manager: CelesteDG
ms.service: active-directory
ms.subservice: app-mgmt
ms.topic: conceptual
ms.workload: identity
ms.date: 02/29/2020
ms.author: baselden
ms.openlocfilehash: 9f05e14de9d6ebe76f722d1e4b21bc2085a74888
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108125143"
---
# <a name="plan-azure-active-directory-my-apps-configuration"></a>Planeamiento de la configuración de Aplicaciones de Azure Active Directory

> [!NOTE]
> Este artículo va dirigido a aquellos profesionales de TI que necesitan planear la configuración del portal Aplicaciones de su organización. 
>
> **Para ver la documentación para el usuario final, consulte [Inicio de sesión e inicio de aplicaciones desde el portal Aplicaciones](../user-help/my-apps-portal-end-user-access.md)** .

Aplicación, de Azure Active Directory (Azure AD), es un portal web para iniciar y administrar aplicaciones. La página Aplicaciones proporciona a los usuarios un lugar desde el que pueden iniciar el trabajo y buscar todas las aplicaciones a las que tienen acceso. Los usuarios accede a Aplicaciones en [https://myapps.microsoft.com](https://myapps.microsoft.com/).

> [!VIDEO https://www.youtube.com/embed/atj6Ivn5m0k]

## <a name="why-configure-my-apps"></a>Razones para configurar Aplicaciones

El portal Aplicaciones está disponible para los usuarios de forma predeterminada y no se puede desactivar. Es importante configurarlo para que funcione de la mejor forma posible y para que sea útil. 

En él aparecen todas las aplicaciones empresariales de Azure Active Directory que cumplen estas dos condiciones:

* La propiedad visibility de la aplicación se ha establecido en true. 

* La aplicación se ha asignado a algún usuario o grupo. Aparece para los usuarios asignados.

La configuración del portal garantiza que las personas adecuadas puedan encontrar fácilmente las aplicaciones adecuadas.

 
### <a name="how-is-the-my-apps-portal-used"></a>¿Cómo se usa el portal Aplicaciones?

Los usuarios acceden al portal Aplicaciones para:

* Descubrir y acceder a todas las aplicaciones conectadas a Azure AD de la organización a las que tienen acceso.

   * Para proporcionar a los usuarios la mejor experiencia posible, conviene asegurarse de que las aplicaciones están configuradas para el inicio de sesión único (SSO).

* Solicitar acceso a las nuevas aplicaciones configuradas para el autoservicio.

* Crear colecciones personales de aplicaciones.

* Administrar el acceso a las aplicaciones para otros usuarios cuando se asigne el rol de propietario del grupo o control delegado en el grupo que se usa para conceder acceso a las aplicaciones.

Los administradores pueden configurar:

* [Experiencias de consentimiento](../manage-apps/configure-user-consent.md), incluidos los términos del servicio.

* [Solicitudes de acceso y detección de aplicaciones en forma de autoservicio](../manage-apps/access-panel-manage-self-service-access.md).

* [Colecciones de aplicaciones](../manage-apps/access-panel-collections.md).

* Asignación de iconos a aplicaciones

* Nombres de aplicaciones descriptivos para el usuario

* Personalización de marca de empresa que se muestra en Aplicaciones

 

## <a name="plan-consent-configuration"></a>Planeamiento de la configuración del consentimiento

### <a name="user-consent-for-applications"></a>Consentimiento del usuario para las aplicaciones 

Para que un usuario pueda iniciar sesión en una aplicación y la aplicación pueda acceder a los datos de su organización, el usuario o administrador debe conceder los permisos de la aplicación. Puede configurar si se permite el consentimiento del usuario y en qué condiciones. **Microsoft recomienda permitir solo el consentimiento del usuario para las aplicaciones de los editores comprobados.**

Para obtener más información, consulte [Configuración del consentimiento de los usuarios finales en las aplicaciones](../manage-apps/configure-user-consent.md).

### <a name="group-owner-consent-for-apps-accessing-data"></a>Consentimiento del propietario de un grupo para las aplicaciones que acceden a los datos

Los propietarios de grupos y equipos pueden dar permiso a aplicaciones tales como las aplicaciones que publican otros proveedores, para obtener acceso a los datos de la organización asociados al grupo. Consulte [Consentimiento específico de recursos en Microsoft Teams](/microsoftteams/resource-specific-consent) para obtener más información. 

Puede configurar si quiere permitir o deshabilitar esta característica.

Para más información, consulte el artículo sobre la [configuración de los permisos para el consentimiento en los grupos](../manage-apps/configure-user-consent-groups.md).

### <a name="plan-communications"></a>Planeamiento de las comunicaciones

La comunicación es fundamental para el éxito de cualquier servicio nuevo. Informe de forma proactiva a los usuarios sobre cómo y cuándo cambiará su experiencia y cómo obtener soporte técnico si es necesario.

Aunque Aplicaciones no suele dar problemas a los usuarios, es importante estar preparado. Cree guías y una lista de todos los recursos para el personal de soporte técnico antes de su lanzamiento.

#### <a name="communications-templates"></a>Plantillas de comunicaciones

Microsoft proporciona [plantillas personalizables para correos electrónicos y otras comunicaciones](https://aka.ms/APTemplates) de Aplicaciones. Puede adaptar estos recursos para su uso en otros canales de comunicación, como resulte adecuado para su cultura corporativa.

 

## <a name="plan-your-sso-configuration"></a>Planeación de la configuración del inicio de sesión único

Es aconsejable que todas las aplicaciones del portal tengan el inicio de sesión único habilitado para que los usuarios puedan usarlas sin problemas sin tener que escribir sus credenciales.

Azure AD admite varias opciones de inicio de sesión único. 

* Para más información al respecto, consulte [Opciones de inicio de sesión único en Azure AD](sso-options.md).

* Para obtener más información sobre cómo usar Azure AD como proveedor de identidades de una aplicación, consulte la [serie de inicios rápidos sobre la administración de aplicaciones](../manage-apps/view-applications-portal.md).

### <a name="use-federated-sso-if-possible"></a>Uso de SSO federado siempre que sea posible

Para que el usuario tenga la mejor experiencia posible en la página Aplicaciones, empiece por la integración de las aplicaciones en la nube disponibles para el inicio de sesión único federado (OpenID Connect o SAML). El inicio de sesión único federado permite a los usuarios tener una experiencia coherente de un solo clic en las superficies de inicio de las aplicaciones y tiende a ser más sólido en el control de la configuración.

Para más información sobre cómo configurar las aplicaciones como software como servicio (SaaS) para poder usar el inicio de sesión único, vea el [plan de implementación de SSO de SaaS]. /Desktop/plan-sso-deployment.md).

### <a name="considerations-for-special-sso-circumstances"></a>Consideraciones sobre el inicio de sesión único en circunstancias especiales

> [!TIP]
> Para mejorar la experiencia del usuario, utilice el inicio de sesión único federado con Azure AD (OpenID Connect o SAML) cuando las aplicaciones lo admitan, en lugar del inicio de sesión único basado en contraseña y ADFS.

Para iniciar sesión en aplicaciones que usan contraseña para el inicio de sesión único o en aplicaciones a las que accede Azure AD Application Proxy, los usuarios deben instalar y usar la extensión de inicio de sesión seguro de Aplicaciones. A los usuarios se les solicita que instalen la extensión la primera vez que inician la aplicación de Application Proxy o con inicio de sesión único mediante contraseña. 

![Captura de pantalla de](./media/my-apps-deployment-plan/ap-dp-install-myapps.png)

Para más información sobre la extensión, consulte el artículo sobre la [instalación de la extensión del explorador Aplicaciones](../user-help/my-apps-portal-end-user-access.md).

Si debe integrar estas aplicaciones, debería definir un mecanismo para implementar la extensión a escala con [exploradores compatibles](../user-help/my-apps-portal-end-user-access.md). Las opciones son:

* [Descarga y configuración de Chrome, Firefox, Microsoft Edge o IE controladas por el usuario](../user-help/my-apps-portal-end-user-access.md)

* [Configuration Manager para Internet Explorer](/mem/configmgr/core/clients/deploy/deploy-clients-to-windows-computers)

Esta extensión permite a los usuarios iniciar cualquier aplicación desde su barra de búsqueda, encontrar acceso a las aplicaciones usadas recientemente y tener un vínculo a la página Aplicaciones.

![Captura de pantalla de la búsqueda de la extensión Aplicaciones](./media/my-apps-deployment-plan/my-app-extsension.png)

#### <a name="plan-for-mobile-access"></a>Planeación del acceso móvil

En el caso de las aplicaciones que usan el inicio de sesión único mediante contraseña o a las que se accede mediante [Microsoft Azure AD Application Proxy](../manage-apps/application-proxy.md), debe usar Microsoft Edge para dispositivos móviles. Para otras aplicaciones, se puede usar cualquier explorador para dispositivos móviles. 

### <a name="linked-sso"></a>Inicio de sesión único vinculado

Las aplicaciones se pueden agregar mediante la opción de inicio de sesión único vinculado. Puede configurar un icono de la aplicación que se vincule a la dirección URL de la aplicación web existente. El inicio de sesión único vinculado le permite empezar a dirigir a los usuarios al portal Aplicaciones sin migrar todas las aplicaciones al inicio de sesión único de Azure AD. Puede moverse gradualmente a aplicaciones configuradas con el inicio de sesión único de Azure AD sin interrumpir la experiencia de los usuarios.

## <a name="plan-the-user-experience"></a>Planeamiento de la experiencia del usuario

De forma predeterminada, tanto las aplicaciones a las que el usuario tiene acceso como todas las aplicaciones configuradas para la detección autoservicio aparecen en el panel Aplicaciones del usuario. En muchas organizaciones, esta lista puede ser muy grande, lo que puede hacer que sea algo muy pesado si no está organizada

### <a name="plan-my-apps-collections"></a>Planeamiento de las colecciones de Aplicaciones

Todas las aplicaciones de Azure AD a las que algún usuario tenga acceso aparecerán en Aplicaciones en la colección Todas las aplicaciones. Use las colecciones para agrupar aplicaciones relacionadas y presentarlas en una pestaña independiente, lo que facilita su búsqueda. Por ejemplo, puede usar colecciones para crear agrupaciones lógicas de aplicaciones para roles de trabajo específicos, tareas, proyectos, etc. 

Los usuarios finales también pueden personalizar su experiencia. Para ello deben:

* Crear sus propias colecciones de aplicaciones.

* [Ocultar y reordenar las colecciones de aplicaciones](access-panel-collections.md).

![Captura de pantalla de la configuración mediante autoservicio](./media/my-apps-deployment-plan/collections.png)

Hay una opción para ocultar aplicaciones en el portal Aplicaciones y, al mismo tiempo, permitir el acceso desde otras ubicaciones, como el portal de Microsoft 365. Más información: [Ocultación de una aplicación de la experiencia del usuario en Azure Active Directory](hide-application-from-user-portal.md)

> [!IMPORTANT]
> Desde Aplicaciones solo se puede acceder a las 950 aplicaciones a las que los usuarios tienen acceso, entre las que se encuentran las aplicaciones que ha ocultado el usuario o el administrador. 

### <a name="plan-self-service-group-management-membership"></a>Planeamiento de la pertenencia a la administración grupos de autoservicio

Puede permitir que los usuarios creen y administren sus propios grupos de seguridad o grupos de Microsoft 365 en Azure AD. El propietario del grupo puede aprobar o rechazar solicitudes de pertenencia y delegar el control de la pertenencia a grupos. Las características de administración de grupos de autoservicio no están disponibles para grupos de seguridad habilitados para correo electrónico o listas de distribución.

Para planear la pertenencia a grupos de autoservicio, determine si va a permitir que todos los usuarios de la organización creen y administren los grupos, o solo un subconjunto de usuarios. Si va a permitir a un subconjunto de usuarios, deberá configurar un grupo al que se agreguen esas personas. 

Consulte [Configuración de la administración de grupos de autoservicio en Azure Active Directory](../enterprise-users/groups-self-service-management.md) para más información sobre cómo habilitar estos escenarios.

### <a name="plan-self-service-application-access"></a>Planeamiento del acceso de la aplicación de autoservicio

Puede permitir que los usuarios detecten aplicaciones y soliciten acceso a ellas mediante el panel Aplicaciones. Para ello, lo primero que debe hacer es: 

* Habilitar la administración de grupos de autoservicio.

* Habilitar la aplicación para el inicio de sesión único.

* Crear un grupo de acceso a la aplicación.

![Captura de pantalla de la configuración de autoservicio de Aplicaciones](./media/my-apps-deployment-plan/my-apps-self-service.png)

Cuando los usuarios solicitan acceso, dicho acceso es al grupo subyacente y los propietarios del grupo pueden delegar el permiso para administrar la pertenencia al grupo y, por tanto, el acceso a la aplicación. Hay flujos de trabajo de aprobación disponibles para la aprobación explícita para acceder a las aplicaciones. Los usuarios que sean aprobadores recibirán notificaciones en el portal Aplicaciones cuando haya solicitudes pendientes de acceso a la aplicación.

## <a name="plan-reporting-and-auditing"></a>Planeación de informes y auditoría

Azure AD proporciona [informes que ofrecen datos técnicos y información empresarial../reports-monitoring/overview-reports.md). Colabore con los propietarios de las aplicaciones técnicas y empresariales para que asuman la propiedad de los informes y los consuman de forma periódica. En la tabla siguiente se proporcionan algunos ejemplos de escenarios de informes típicos.

| Ejemplo| Administración de los riesgos.| Aumento de la productividad| Gobernanza y cumplimiento |
| - | - | - | -|
| Tipos de informes| Permisos y uso de la aplicación| Actividad de aprovisionamiento de cuentas| Revise quién accede a las aplicaciones |
| Acciones posibles| Auditar el acceso; revocar los permisos| Corregir los errores de aprovisionamiento| Revocar el acceso |


Azure AD mantiene la mayoría de los datos de auditoría durante 30 días. Los datos están disponibles a través del portal de administración de Azure o de la API para que pueda descargarlos en los sistemas de análisis.

#### <a name="auditing"></a>Auditoría

Los registros de auditoría para el acceso a las aplicaciones estarán disponibles durante 30 días. Si la organización requiere que los registros se conserven más tiempo, expórtelos deberán a una herramienta de Administración de eventos e información de seguridad (SIEM), como Splunk o ArcSight.

En el caso de las copias de seguridad de las auditorías, informes y recuperación ante desastres, documente la frecuencia de descarga necesaria, cuál es el sistema de destino y quién es el responsable de administrar cada copia de seguridad. Es posible que no necesite copias de seguridad independientes de auditoría e informes. La copia de seguridad de recuperación ante desastres debe ser una entidad independiente.

## <a name="validate-your-deployment"></a>Validación de la implementación

Asegúrese de que la implementación de Aplicaciones se ha probado exhaustivamente y de que hay un plan de reversión en vigor.

Realice las siguientes pruebas tanto con dispositivos que sean de propiedad la empresa como con dispositivos personales. Estos casos de prueba también deben reflejar los casos de uso empresarial. A continuación se muestran algunos casos basados en escenarios técnicos habituales. Agregue otros que sean específicos a sus necesidades.

#### <a name="application-sso-access-test-case-examples"></a>Ejemplos de casos de prueba de acceso con el inicio de sesión único de aplicación:


| Oportunidad de negocio| Resultado esperado |
| - | - |
| El usuario inicia sesión en el portal Aplicaciones.| El usuario puede iniciar sesión y ver sus aplicaciones. |
| El usuario inicia una aplicación de inicio de sesión único federado.| El usuario inicia sesión automáticamente en la aplicación. |
| El usuario inicia una aplicación de inicio de sesión único con contraseña por primera vez.| El usuario debe instalar la extensión Aplicaciones. |
| El usuario inicia una aplicación de inicio de sesión único con contraseña las veces posteriores.| El usuario inicia sesión automáticamente en la aplicación. |
| El usuario inicia una aplicación desde el portal de Microsoft 365.| El usuario inicia sesión automáticamente en la aplicación. |
| El usuario inicia una aplicación desde Managed Browser.| El usuario inicia sesión automáticamente en la aplicación. |


#### <a name="application-self-service-capabilities-test-case-examples"></a>Casos de prueba de ejemplo de las funcionalidades de autoservicio de aplicaciones


| Oportunidad de negocio| Resultado esperado |
| - | - |
| El usuario puede administrar la pertenencia a la aplicación.| El usuario puede agregar o quitar miembros con acceso a la aplicación. |
| El usuario puede editar la aplicación.| El usuario puede editar la descripción y las credenciales de la aplicación en el caso de aplicaciones de inicio de sesión único con contraseña. |


### <a name="rollback-steps"></a>Pasos de reversión

Es importante planear qué hacer si la implementación no funciona según lo previsto. Si se produce un error en la configuración del inicio de sesión único durante la implementación, debe saber cómo [solucionar los problemas de inicio de sesión único](../hybrid/tshoot-connect-sso.md) y reducir el impacto para los usuarios. En circunstancias extremas, puede que necesite [revertir el inicio de sesión único](plan-sso-deployment.md).

## <a name="manage-your-implementation"></a>Administrar la implementación

Use el rol con menos privilegios para realizar una tarea necesaria en Azure Active Directory. [Revise los distintos roles disponibles](../roles/permissions-reference.md) y elija el correcto para resolver las necesidades para cada rol de esta aplicación. Es posible que algunos roles se tengan que aplicar temporalmente y quitar una vez completada la implementación.

| Personas| Roles| Rol de Azure AD |
| - | - | - |
| Administrador del departamento de soporte técnico| Soporte técnico de nivel 1| None |
| Administración de identidades| Configurar y depurar cuando haya problemas que afectan a Azure AD| Administrador global |
| Administrador de aplicaciones| Atestación de usuarios en la aplicación, configuración en usuarios con permisos| None |
| Administradores de infraestructura| Propietario de la sustitución de certificados| Administrador global |
| Propietario de la empresa o parte interesada| Atestación de usuarios en la aplicación, configuración en usuarios con permisos| None |


Puede usar [Privileged Identity Management](../privileged-identity-management/pim-configure.md) para administrar los roles con el fin de permitir la auditoría, control y revisión de acceso adicionales para los usuarios con permisos de directorio.

## <a name="next-steps"></a>Pasos siguientes

[Planeamiento de una implementación de Azure AD Multi-Factor Authentication](../authentication/howto-mfa-getstarted.md)

[Planeación de la implementación de un proxy de aplicación](application-proxy-deployment-plan.md)