---
title: Migración de la autenticación de las aplicaciones a Azure Active Directory
description: En estas notas del producto se proporcionan detalles sobre la planificación y ventajas de migrar la autenticación de la aplicación a Azure AD.
services: active-directory
author: iantheninja
manager: CelesteDG
ms.service: active-directory
ms.subservice: app-mgmt
ms.topic: how-to
ms.workload: identity
ms.date: 02/05/2021
ms.author: iangithinji
ms.reviewer: baselden
ms.collection: M365-identity-device-management
ms.openlocfilehash: b7e4a92f84cdbc0061509fa710de148e9e9f2b52
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108125164"
---
# <a name="migrate-application-authentication-to-azure-active-directory"></a>Migración de la autenticación de las aplicaciones a Azure Active Directory

## <a name="about-this-paper"></a>Acerca de estas notas

En estas notas del producto se proporcionan detalles sobre la planificación y ventajas de migrar la autenticación de la aplicación a Azure AD. Están diseñadas para los administradores de Azure y los profesionales de identidades.

Al dividir el proceso en cuatro fases, cada una con criterios detallados de planeación y salida detallados, están diseñadas para ayudarle a planear la estrategia de migración y a comprender cómo la autenticación de Azure AD ayuda a los objetivos de su organización.

## <a name="introduction"></a>Introducción

En la actualidad, su organización requiere un gran número de aplicaciones para que los usuarios puedan trabajar. Probablemente siga agregando, desarrollando o retirando aplicaciones todos los días. Los usuarios tienen acceso a estas aplicaciones desde una amplia gama de dispositivos y ubicaciones tanto corporativos como personales. Abren las aplicaciones de muchas maneras, como:

- a través de la página principal o portal de la empresa

- mediante marcadores en sus exploradores web

- a través de la dirección URL de un proveedor para las aplicaciones de software como servicio (SaaS)

- vínculos enviados directamente a los dispositivos de escritorio o móviles del usuario a través de una solución de administración de dispositivos/aplicaciones móviles (MDM/MAM)

Es probable que sus aplicaciones usen los siguientes tipos de autenticación:

- Soluciones de federación locales (como los Servicios de federación de Active Directory (AD FS) y Ping)

- Active Directory (como la autenticación de Kerberos y la autenticación integrada de Windows)

- Otras soluciones de administración de identidades y acceso (IAM) basadas en la nube (como Okta u Oracle)

- Infraestructuras web locales (como IIS y Apache)

- Infraestructuras hospedadas en la nube (como Azure y AWS)

**Para asegurarse de que los usuarios puedan acceder de forma fácil y segura a las aplicaciones, el objetivo es tener un único conjunto de controles y directivas de acceso en los entornos locales y en la nube.**

[Azure Active Directory (Azure AD)](../fundamentals/active-directory-whatis.md) ofrece una plataforma de identidad universal que proporciona a los usuarios, asociados y clientes una identidad única para acceder a las aplicaciones que quieran y colaborar desde cualquier plataforma y dispositivo.

![Diagrama de conectividad de Azure Active Directory](media/migrating-application-authentication-to-azure-active-directory-1.jpg)

Azure AD tiene un [conjunto completo de funcionalidades de administración de identidades](../fundamentals/active-directory-whatis.md#which-features-work-in-azure-ad). La estandarización de la autorización y la autenticación de sus aplicaciones en Azure AD le permite obtener las ventajas que ofrecen estas funcionalidades.

Puede encontrar más recursos de migración en [https://aka.ms/migrateapps](./migration-resources.md).

## <a name="benefits-of-migrating-app-authentication-to-azure-ad"></a>Ventajas de migrar la autenticación de las aplicaciones a Azure AD

Cambiar la autenticación de la aplicación a Azure AD le ayudará a administrar los riesgos y costos, aumentar la productividad y cumplir con los requisitos de cumplimiento y gobernanza.

### <a name="manage-risk"></a>Administración de los riesgos.

Para proteger las aplicaciones, es necesaria una vista completa de todos los factores de riesgo. La migración de las aplicaciones a Azure AD consolida sus soluciones de seguridad. Por ejemplo, puede:

- Mejorar el acceso seguro de los usuarios a las aplicaciones y los datos corporativos asociados mediante la tecnología de las[directivas de acceso condicional](../conditional-access/overview.md), la [autenticación multifactor](../authentication/concept-mfa-howitworks.md) y la [protección de identidades](../identity-protection/overview-identity-protection.md) basadas en el riesgo en tiempo real.

- Proteger el acceso del usuario con privilegios a su entorno mediante el acceso de administrador [Just-in-Time](../../azure-resource-manager/managed-applications/request-just-in-time-access.md).

- Usar el [diseño multiinquilino, distribuido geográficamente y de alta disponibilidad de Azure AD](https://cloudblogs.microsoft.com/enterprisemobility/2014/09/02/azure-ad-under-the-hood-of-our-geo-redundant-highly-available-distributed-cloud-directory/) para sus necesidades empresariales más críticas.

- Proteger sus aplicaciones heredadas con una de nuestras [integraciones de asociados de acceso híbrido seguro](https://aka.ms/secure-hybrid-access) que quizá ya haya implementado.

### <a name="manage-cost"></a>Administración de costos

Su organización puede tener varias soluciones de Administración de acceso a identidades (IAM). La migración a una infraestructura de Azure AD es una oportunidad para reducir las dependencias de las licencias de IAM (locales o en la nube) y los costos de infraestructura. En los casos en los que ya haya pagado Azure AD mediante licencias de Microsoft 365, no hay razón para pagar el costo adicional de otra solución IAM.

**Con Azure AD, puede reducir los costos de infraestructura de la siguiente manera:**

- Ofrezca acceso remoto seguro a aplicaciones locales mediante [Azure AD Application Proxy](./application-proxy.md).

- Disasocie las aplicaciones del enfoque de credenciales local en el inquilino mediante la [configuración de Azure AD como proveedor de identidades universal de confianza](../hybrid/plan-connect-user-signin.md#choosing-the-user-sign-in-method-for-your-organization).

### <a name="increase-productivity"></a>Aumento de la productividad

Las ventajas económicas y de seguridad animan a las organizaciones a adoptar Azure AD, pero la adopción y cumplimiento completos son más probables si los usuarios también se benefician. Con Azure AD puede hacer lo siguiente:

- Mejore la experiencia de [inicio de sesión único (SSO)](./what-is-single-sign-on.md) del usuario final con un acceso sin problemas y seguro a cualquier aplicación, desde cualquier dispositivo y cualquier ubicación.

- Use las funcionalidades de autoservicio de IAM, como el [autoservicio de restablecimiento de contraseña](../authentication/concept-sspr-howitworks.md) y el [autoservicio de administración de grupos](../enterprise-users/groups-self-service-management.md).

- Reduzca la sobrecarga administrativa mediante la administración de una única identidad para cada usuario en todos los entornos locales y en la nube:

  - [Automatice el aprovisionamiento](../app-provisioning/user-provisioning.md) de las cuentas de usuario (en la [galería de Azure AD](https://azuremarketplace.microsoft.com/marketplace/apps/category/azure-active-directory-apps)) basadas en identidades de Azure AD.
  - Acceda a todas las aplicaciones desde el panel Mis aplicaciones en [Azure Portal](https://portal.azure.com/).

- Permita a los desarrolladores proteger el acceso a sus aplicaciones y mejorar la experiencia del usuario final mediante el uso de la [plataforma de identidad de Microsoft](../develop/v2-overview.md) con la biblioteca de autenticación de Microsoft (MSAL).

- Permita a sus asociados el acceso a los recursos en la nube mediante la [colaboración B2B de Azure AD](../external-identities/what-is-b2b.md). Los recursos en la nube eliminan la sobrecarga que supone configurar la federación de punto a punto con sus asociados.

### <a name="address-compliance-and-governance"></a>Control de cumplimiento y gobernanza

Garantice el cumplimiento de los requisitos normativos al aplicar las directivas de acceso corporativo y supervisar el acceso de los usuarios a las aplicaciones y datos asociados mediante API y herramientas de auditoría integradas. Con Azure AD, puede supervisar los inicios de sesión en la aplicación mediante informes que usan [herramientas de supervisión de eventos e incidentes de seguridad (SIEM)](../reports-monitoring/plan-monitoring-and-reporting.md). Puede acceder a los informes desde el portal o las API y auditar mediante programación quién tiene acceso a las aplicaciones, así como quitar el acceso a los usuarios inactivos a través de revisiones de acceso.

## <a name="plan-your-migration-phases-and-project-strategy"></a>Planificación de las fases de migración y la estrategia del proyecto

A menudo, los errores en los proyectos tecnológicos se deben a expectativas incompatibles, a que las partes interesadas adecuadas no están implicadas o a una falta de comunicación. Garantice una realización correcta mediante la planeación del proyecto.

### <a name="the-phases-of-migration"></a>Fases de la migración

Antes de abordar la información sobre herramientas, debe comprender el proceso de migración. Gracias a varios talleres directos para los clientes, se recomiendan las cuatro fases siguientes:

![Un diagrama de las fases de la migración](media/migrating-application-authentication-to-azure-active-directory-2.jpg)

### <a name="assemble-the-project-team"></a>Conformación del equipo del proyecto

La migración de aplicaciones es un trabajo en equipo y debe asegurarse de que todos los puestos importantes están cubiertos. El apoyo de los directivos de la empresa es importante. Asegúrese de involucrar al conjunto adecuado de patrocinadores ejecutivos, responsables de la toma de decisiones empresariales y expertos en la materia (SME).

Durante el proyecto de migración, una persona puede cumplir varios roles o varias personas pueden cumplir cada rol, en función del tamaño y la estructura de su organización. También puede depender de otros equipos que jueguen un papel clave en el panorama de seguridad.

En la tabla siguiente se incluyen los roles importantes y sus contribuciones:

| Role          | Contribuciones                                              |
| ------------- | ---------------------------------------------------------- |
| **Administrador del proyecto** | Responsable del proyecto que se encarga de dirigir el proyecto, lo que incluye:<br /> - obtener el apoyo de los ejecutivos<br /> - hacer participar a las partes interesadas<br /> - administrar programaciones, documentación y comunicaciones |
| **Arquitecto de identidades/administrador de aplicación de Azure AD** | Se encarga de lo siguiente:<br /> - diseñar la solución en conjunto con las partes interesadas<br /> - documentar el diseño de la solución y los procedimientos operativos para entregarlos al equipo de operaciones<br /> - administrar los entornos de preproducción y producción |
| **Equipo de operaciones de AD local** | Organización que administra los distintos orígenes de identidades locales, como los bosques de AD, los directorios de LDAP, los sistemas de recursos humanos, etc.<br /> - realizar todas las tareas de corrección necesarias antes de la sincronización<br /> - proporcionar las cuentas de servicio necesarias para la sincronización<br /> - proporcionar el acceso para configurar la federación a Azure AD |
| **Administrador de soporte técnico de TI** | Un representante de la organización de soporte técnico de TI que puede proporcionar información sobre la compatibilidad de este cambio desde una perspectiva del departamento de soporte técnico. |
| **Propietario de seguridad**  | Representante del equipo de seguridad que puede asegurarse de que el plan cumplirá los requisitos de seguridad de la organización. |
| **Propietarios técnicos de aplicaciones** | Incluye a los propietarios técnicos de las aplicaciones y servicios que se integrarán con Azure AD. Proporcionan los atributos de identidad de las aplicaciones que deben incluirse en el proceso de sincronización. Normalmente tienen una relación con los representantes de CSV. |
| **Propietario empresarial de la aplicación** | Colegas de los representantes que pueden proporcionar información sobre la experiencia del usuario y la utilidad de este cambio desde la perspectiva del usuario y que son propietarios del aspecto empresarial general de la aplicación, que puede incluir la administración del acceso. |
| **Grupo piloto de usuarios** | Usuarios que realizan pruebas como parte de su trabajo diario, la experiencia piloto y que proporcionan comentarios para guiar el resto de las implementaciones. |

### <a name="plan-communications"></a>Planeamiento de las comunicaciones

La clave del éxito es la comunicación y la interacción empresariales efectivas. Es importante proporcionar a las partes interesadas y a los usuarios finales una vía para obtener información y mantenerse informados de las actualizaciones programadas. Explique a todos el valor de la migración, cuáles son las escalas de tiempo previstas y cómo prepararse para cualquier interrupción temporal de las operaciones. Use varias vías, como sesiones informativas, correos electrónicos, reuniones uno a uno, banners y asambleas públicas.

Según la estrategia de comunicación que haya elegido para la aplicación, puede que quiera recordar a los usuarios del tiempo de inactividad pendiente. También debe comprobar que no hay cambios recientes o repercusiones empresariales que requieran posponer la implementación.

En la tabla siguiente, encontrará la comunicación mínima sugerida para mantener informadas a las partes interesadas:

**Fases del plan y estrategia del proyecto**:

| Comunicación      | Público                                          |
| ------------------ | ------------------------------------------------- |
| Difusión e importancia técnica/empresaral del proyecto | Todos, excepto los usuarios finales |
| Convocatoria para aplicaciones piloto | - Propietarios empresariales de la aplicación<br />- Propietarios técnicos de la aplicación<br />- Equipo de arquitectos e identidades |

**Fase 1: detección y ámbito**:

| Comunicación      | Público                                          |
| ------------------ | ------------------------------------------------- |
| - Convocatoria de información de la aplicación<br />- Resultado del ejercicio de ámbito | - Propietarios técnicos de la aplicación<br />- Propietarios empresariales de la aplicación |

**Fase 2: clasificación de aplicaciones y planificación del piloto**:

| Comunicación      | Público                                          |
| ------------------ | ------------------------------------------------- |
| - Resultado de las clasificaciones e implicaciones para la programación de la migración<br />- Programación de la migración preliminar | - Propietarios técnicos de la aplicación<br /> - Propietarios empresariales de la aplicación |

**Fase 3: planificación de la migración y pruebas**:

| Comunicación      | Público                                          |
| ------------------ | ------------------------------------------------- |
| - Resultado de las pruebas de migración de aplicaciones. | - Propietarios técnicos de la aplicación<br />- Propietarios empresariales de la aplicación |
| - Notificación de que la migración se aproxima y explicación de las experiencias de usuario final resultantes.<br />- Tiempo de inactividad que se aproxima y comunicación completa, incluido qué deben hacer ahora, los comentarios y cómo obtener ayuda | - Usuarios finales (y todos los demás) |

**Fase 4: administración y obtención de conclusiones**:

| Comunicación      | Público                                          |
| ------------------ | ------------------------------------------------- |
| Análisis disponibles y cómo acceder | - Propietarios técnicos de la aplicación<br />- Propietarios empresariales de la aplicación |

### <a name="migration-states-communication-dashboard"></a>Panel de comunicación sobre estados de migración

Es fundamental comunicar el estado general del proyecto de migración, ya que muestra el progreso, y ayuda a los propietarios de las aplicaciones que están próximas a migrarse a prepararse para el traslado. Puede crear un panel sencillo con Power BI u otras herramientas de informes para ofrecer visibilidad del estado de las aplicaciones durante la migración.

Puede plantearse usar los siguientes estados de migración:

| Estados de migración       | Plan de acción                                   |
| ---------------------- | --------------------------------------------- |
| **Solicitud inicial** | Busque la aplicación y póngase en contacto con el propietario para obtener más información. |
| **Evaluación completada** | El propietario de la aplicación evalúa los requisitos de la aplicación y devuelve el cuestionario de la aplicación.</td>
| **Configuración en curso** | Desarrolle los cambios necesarios para administrar la autenticación en Azure AD. |
| **Configuración de prueba correcta** | Evalúe los cambios y autentique la aplicación en el inquilino de Azure AD de prueba en el entorno de prueba. |
| **Configuración de producción correcta** | Cambie las configuraciones para que funcionen con el inquilino de AD de producción y evalúe la autenticación de la aplicación en el entorno de prueba. |
| **Completado/aprobado** | Implemente los cambios de la aplicación en el entorno de producción y ejecútelos en el inquilino de Azure AD de producción. |

Esto garantizará que los propietarios de la aplicación sepan cuál es la programación correspondiente a la migración y la prueba de la aplicación cuando las aplicaciones van a migrarse, y cuáles son los resultados de otras aplicaciones que ya se han migrado. También puede plantearse la posibilidad de proporcionar vínculos a la base de datos de seguimiento de errores para que los propietarios puedan archivar y ver los problemas de las aplicaciones que se van a migrar.

### <a name="best-practices"></a>Procedimientos recomendados

Los siguientes son los casos de éxito de nuestros clientes y asociados, así como los procedimientos recomendados sugeridos:

- [Cinco sugerencias para mejorar el proceso de migración a Azure Active Directory](https://techcommunity.microsoft.com/t5/Azure-Active-Directory-Identity/Five-tips-to-improve-the-migration-process-to-Azure-Active/ba-p/445364) de Patriot Consulting, un miembro de nuestra red de asociados que se centra en ayudar a los clientes a implementar soluciones en la nube de Microsoft de manera segura.

- [Desarrollo de una estrategia de administración de riesgos para migrar su aplicación de Azure AD](https://techcommunity.microsoft.com/t5/Azure-Active-Directory-Identity/Develop-a-risk-management-strategy-for-your-Azure-AD-application/ba-p/566488)de Edgile, un asociado que se centra en las soluciones de IAM y administración de riesgos.

## <a name="phase-1-discover-and-scope-apps"></a>Fase 1: Detección y ámbito de aplicaciones

**La detección y el análisis de aplicaciones es un ejercicio fundamental para lograr un buen comienzo.** Es posible que no lo sepa todo, por lo que debe estar preparado para adaptarse a las aplicaciones desconocidas.

### <a name="find-your-apps"></a>Búsqueda de aplicaciones

La primera decisión en la migración de una aplicación es qué aplicaciones se van a migrar, si alguna debe quedarse y qué aplicaciones se van a dejar de usar. Siempre existe la posibilidad de dejar de usar las aplicaciones que no usará en su organización. Hay varias maneras de buscar aplicaciones en su organización. **Durante la detección de aplicaciones, asegúrese de incluir las aplicaciones en desarrollo y planificadas. Use Azure AD para la autenticación en todas las aplicaciones futuras.**

**Uso de servicios de federación de Active Directory (AD FS) para recopilar un inventario de aplicaciones correctas:**

- **Use Azure AD Connect Health.** Si tiene una licencia de Azure AD Premium, se recomienda que implemente [Azure AD Connect Health](../hybrid/how-to-connect-health-adfs.md) para analizar el uso de la aplicación en el entorno local. Puede usar el [informe de la aplicación de ADFS](./migrate-adfs-application-activity.md) (versión preliminar) para detectar las aplicaciones de ADFS que se pueden migrar y evaluar la preparación de la aplicación que se va a migrar. Después de completar la migración, implemente [Cloud Discovery](/cloud-app-security/set-up-cloud-discovery), que le permite supervisar continuamente la shadow IT de su organización una vez que esté en la nube.

- **Análisis de registros de AD FS**. Si no tiene licencias de Azure AD Premium, se recomienda que use las herramientas de migración de la aplicación de ADFS a Azure AD basadas en [PowerShell](https://github.com/AzureAD/Deployment-Plans/tree/master/ADFS%20to%20AzureAD%20App%20Migration). Consulte la [guía de soluciones](./migrate-adfs-apps-to-azure.md):

[Migración de aplicaciones de servicios de federación de Active Directory (AD FS) a Azure AD.](./migrate-adfs-apps-to-azure.md)

### <a name="using-other-identity-providers-idps"></a>Uso de otros proveedores de identidades (IdP)

En el caso de otros proveedores de identidades (como Okta o Ping), puede usar sus herramientas para exportar el inventario de aplicaciones. Puede considerar la posibilidad de consultar los principios de servicio registrados en Active Directory que se corresponden con las aplicaciones web de su organización.

### <a name="using-cloud-discovery-tools"></a>Uso de herramientas de Cloud Discovery

En el entorno en la nube, necesita visibilidad enriquecida, control sobre el viaje de los datos y análisis sofisticados para encontrar y combatir las ciberamenazas en todos los servicios en la nube. Puede recopilar el inventario de aplicaciones en la nube mediante las siguientes herramientas:

- **Agente de seguridad de acceso a la nube (CASB**): un [CASB](/cloud-app-security/) suele funcionar junto con el firewall para proporcionar visibilidad sobre el uso de la aplicación en la nube de los empleados y ayuda a proteger los datos corporativos de las amenazas de ciberseguridad. El informe de CASB puede ayudarle a determinar las aplicaciones más usadas en la organización y los primeros objetivos para migrar a Azure AD.

- **Cloud Discovery**: mediante la configuración de [Cloud Discovery](/cloud-app-security/set-up-cloud-discovery), obtiene visibilidad del uso de aplicaciones en la nube y puede detectar aplicaciones de shadow IT o no autorizadas.

- **API**: para las aplicaciones conectadas a la infraestructura en la nube, puede usar las API y las herramientas de esos sistemas para empezar a realizar un inventario de las aplicaciones hospedadas. En el entorno de Azure:

  - Use el cmdlet [Get-AzureWebsite](/powershell/module/servicemanagement/azure.service/get-azurewebsite) para obtener información sobre los sitios web de Azure.

  - Use el cmdlet [Get-AzureRMWebApp](/powershell/module/azurerm.websites/get-azurermwebapp) para obtener información sobre Azure Web Apps.
D
  - Puede encontrar todas las aplicaciones que se ejecutan en Microsoft IIS desde la línea de comandos de Windows mediante [AppCmd.exe](/iis/get-started/getting-started-with-iis/getting-started-with-appcmdexe#working-with-sites-applications-virtual-directories-and-application-pools).

  - Use [aplicaciones](/previous-versions/azure/ad/graph/api/entity-and-complex-type-reference#application-entity) y [entidades de servicio](/previous-versions/azure/ad/graph/api/entity-and-complex-type-reference#serviceprincipal-entity) para obtener información sobre una aplicación y una instancia de la aplicación en un directorio de Azure AD.

### <a name="using-manual-processes"></a>Uso de procesos manuales

Una vez que haya llevado a cabo los enfoques automatizados descritos anteriormente, tendrá un buen manejo de sus aplicaciones. No obstante, se recomienda que haga lo siguiente para asegurarse de que tiene una buena cobertura en todas las áreas de acceso de usuario:

- Póngase en contacto con los distintos propietarios empresariales de su organización para encontrar las aplicaciones que se están usando en su organización.

- Ejecute una herramienta de inspección de HTTP en el servidor proxy o analice los registros del proxy para ver a dónde se enruta normalmente el tráfico.

- Revise los registros web de los sitios de portal de empresa populares para ver los vínculos a los que más acceden los usuarios.

- Póngase en contacto con los ejecutivos u otros miembros empresariales importantes para asegurarse de que ha cubierto las aplicaciones críticas para la empresa.

### <a name="type-of-apps-to-migrate"></a>Tipos de aplicaciones que se van a migrar

Una vez que encuentre las aplicaciones, identificará estos tipos de aplicaciones en su organización:

- Aplicaciones que ya usan protocolos de autenticación modernos

- Aplicaciones que usan protocolos de autenticación heredados que ha decidido modernizar

- Aplicaciones que usan protocolos de autenticación heredados que ha decidido no modernizar

- Nuevas aplicaciones de línea de negocio (LoB)

### <a name="apps-that-use-modern-authentication-already"></a>Aplicaciones que ya usan la autenticación moderna

Las aplicaciones ya modernizadas tienen más probabilidades de trasladarse a Azure AD. Estas aplicaciones ya usan protocolos de autenticación modernos (como SAML u OpenID Connect) y se pueden volver a configurar para autenticarse con Azure AD.

Además de las opciones de la [galería de aplicaciones de Azure AD](https://azuremarketplace.microsoft.com/marketplace/apps/category/azure-active-directory-apps), estas pueden ser aplicaciones que ya existen en la organización o cualquier aplicación de terceros de un proveedor que no forma parte de la galería de Azure AD ([aplicaciones que no son de la galería](./add-application-portal.md)).

Aplicaciones heredadas que elige modernizar

En el caso de las aplicaciones heredadas que quiera modernizar, el traslado a Azure AD de la autenticación y la autorización principales desbloquea toda la eficacia y variedad de datos que ofrecen [Microsoft Graph](https://developer.microsoft.com/graph/gallery/?filterBy=Samples,SDKs) e [Intelligent Security Graph](https://www.microsoft.com/security/operations/intelligence?rtc=1).

Se recomienda **actualizar el código de la pila de autenticación** para estas aplicaciones del protocolo heredado (como la autenticación integrada de Windows, la delegación restringida de Kerberos y la autenticación basada en encabezados HTTP) a un protocolo moderno (como SAML u OpenID Connect).

### <a name="legacy-apps-that-you-choose-not-to-modernize"></a>Aplicaciones heredadas que elige no modernizar

En el caso de ciertas aplicaciones que usan protocolos de autenticación heredados, a veces modernizar la autenticación no es la decisión adecuada por motivos empresariales. Este caso se puede aplicar a los siguientes tipos de aplicaciones:

- Las aplicaciones que se mantienen en el entorno local por motivos de cumplimiento o control.

- Las aplicaciones conectadas a un proveedor de federación o identidades local que no quiere cambiar.

- Las aplicaciones desarrolladas con estándares de autenticación locales que no tiene la intención de trasladar.

Azure AD puede aportar grandes ventajas a estas aplicaciones heredadas, ya que puede habilitar las características modernas de seguridad y gobernanza de Azure AD como la [autenticación multifactor](../authentication/concept-mfa-howitworks.md), el [acceso condicional](../conditional-access/overview.md), [Identity Protection](../identity-protection/index.yml), el [acceso de aplicaciones delegado](./access-panel-manage-self-service-access.md) y las [revisiones de acceso](../governance/manage-user-access-with-access-reviews.md#create-and-perform-an-access-review) en estas aplicaciones sin modificarlas en lo absoluto.

Comience por **extender estas aplicaciones a la nube** con Azure AD [Application Proxy](./application-proxy-configure-single-sign-on-password-vaulting.md) con un método sencillo de autenticación (por ejemplo, el almacenamiento de contraseñas) para que los usuarios migren rápidamente, o a través de las [integraciones de asociados](https://azure.microsoft.com/services/active-directory/sso/secure-hybrid-access/) con los controladores de entrega de aplicaciones que ya ha implementado.

### <a name="new-line-of-business-lob-apps"></a>Nuevas aplicaciones de línea de negocio (LoB)

Normalmente se desarrollan aplicaciones LoB para el uso interno en la organización. Si tiene aplicaciones nuevas en la canalización, se recomienda usar la [plataforma de identidad de Microsoft](../develop/v2-overview.md) para implementar OpenID Connect.

### <a name="apps-to-deprecate"></a>Aplicaciones que se dejarán de usar

Las aplicaciones sin propietarios claros y sin un mantenimiento y supervisión claros presentan un riesgo de seguridad para su organización. Considere la posibilidad de dejar de usar las aplicaciones cuando:

- su **funcionalidad sea muy redundante** con otros sistemas • no tienen ningún **propietario empresarial**

- no hay **ningún uso** claro

Se recomienda **no dejar de usar aplicaciones críticas para la empresa con un gran impacto**. En esos casos, trabaje con los propietarios de la empresa para determinar la estrategia correcta.

### <a name="exit-criteria"></a>Exit criteria (Criterios de salida)

Esta fase se realiza correctamente con lo siguiente:

- Una buena comprensión de los sistemas en el ámbito de la migración (que puede retirar una vez que se haya trasladado a Azure AD)

- Una lista de aplicaciones que incluya lo siguiente:

  - A qué sistemas se conectan dichas aplicaciones.
  - Desde dónde y en qué dispositivos pueden acceder los usuarios a ellas.
  - Si se van a migrar, dejar en desuso o a conectarse con [Azure AD Connect](../hybrid/whatis-azure-ad-connect.md).

> [!NOTE]
> Puede descargar la [hoja de cálculo de detección de aplicaciones](https://download.microsoft.com/download/2/8/3/283F995C-5169-43A0-B81D-B0ED539FB3DD/Application%20Discovery%20worksheet.xlsx) para registrar las aplicaciones que quiere migrar a la autenticación de Azure AD, y las que quiere dejar de administrar con [Azure AD Connect](../hybrid/whatis-azure-ad-connect.md).

## <a name="phase-2-classify-apps-and-plan-pilot"></a>Fase 2: clasificación de aplicaciones y planificación del piloto

Clasificar la migración de las aplicaciones es un ejercicio importante. No todas las aplicaciones deben migrarse y cambiarse al mismo tiempo. Una vez que haya recopilado información sobre cada una de las aplicaciones, podrá racionalizar cuáles aplicaciones se deben migrar primero y cuáles pueden demorar más.

### <a name="classify-in-scope-apps"></a>Clasificación de aplicaciones en el ámbito

Una manera de pensar en esto consiste en usar los ejes de importancia para la empresa, uso y duración, cada uno de los cuales depende de varios factores.

### <a name="business-criticality"></a>Crucial para la empresa

La importancia para la empresa adoptará dimensiones diferentes para cada negocio, pero las dos medidas que debe tener en cuenta son las **características y la funcionalidad** y los **perfiles de usuario**. Asigne a las aplicaciones con funcionalidades únicas un valor en puntos superior al de las funcionalidades redundantes u obsoletas.

![Un diagrama de los espectros de características y funcionalidad, así como de perfiles de usuario](media/migrating-application-authentication-to-azure-active-directory-3.jpg)

### <a name="usage"></a>Uso

Las aplicaciones con **números de uso elevados** deben recibir un valor superior al de las aplicaciones con poco uso. Asigne un valor mayor a las aplicaciones con usuarios externos, ejecutivos o del equipo de seguridad. Completes estas evaluaciones para cada aplicación de la cartera de migración.

![Un diagrama de los espectros Volumen de usuarios y Amplitud de usuarios](media/migrating-application-authentication-to-azure-active-directory-4.jpg)

Una vez que haya determinado los valores de importancia para la empresa y uso, puede determinar la **duración de la aplicación** y crear una matriz de prioridad. La siguiente es una matriz de este tipo:

![Diagrama de triángulo que muestra las relaciones entre el uso, la duración esperada y la importancia para la empresa](media/migrating-application-authentication-to-azure-active-directory5.jpg)

### <a name="prioritize-apps-for-migration"></a>Clasificación de las aplicaciones para la migración

Puede optar por comenzar la migración de aplicaciones con las aplicaciones de prioridad más baja o con las aplicaciones de prioridad más alta en función de las necesidades de su organización.

En un escenario en el que quizá no tenga experiencia en el uso de Azure AD y los servicios de identidad, considere la posibilidad de mover primero las **aplicaciones de prioridad más baja** a Azure AD. Esto reducirá el impacto en sus operaciones y podrá tomar impulso. Una vez que haya trasladado correctamente estas aplicaciones y haya ganado la confianza de la parte interesada, podrá continuar con la migración de las otras aplicaciones.

Si no hay ninguna prioridad clara, considere la posibilidad de mover primero las aplicaciones que se encuentran en la [galería de Azure AD](https://azuremarketplace.microsoft.com/marketplace/apps/category/azure-active-directory-apps) y admitir varios proveedores de identidades (ADFS u Okta) porque son más fáciles de integrar. Es probable que estas aplicaciones sean las **aplicaciones de prioridad más alta** de la organización. Para ayudarle a integrar las aplicaciones SaaS con Azure AD, hemos desarrollado un conjunto de [tutoriales](../saas-apps/tutorial-list.md) que le guían a lo largo del proceso de configuración.

Cuando tenga una fecha límite para migrar las aplicaciones, el cubo con las aplicaciones de prioridad más alta tomará la carga de trabajo principal. Más adelante puede seleccionar las aplicaciones de menor prioridad, ya que no cambiarán el costo aunque haya cambiado la fecha límite. Incluso si debe renovar la licencia, será para una cantidad pequeña.

Además de esta clasificación, y en función de la urgencia de la migración, también puede considerar la posibilidad de crear una **programación de migración** en la que los propietarios de la aplicación deben participar para migrar sus aplicaciones. Al final de este proceso, debe tener una lista de todas las aplicaciones en cubos clasificadas por orden de prioridad para la migración.

### <a name="document-your-apps"></a>Documentación de las aplicaciones

En primer lugar, empiece por recopilar los detalles importantes de sus aplicaciones. La [hoja de cálculo de detección de aplicaciones](https://download.microsoft.com/download/2/8/3/283F995C-5169-43A0-B81D-B0ED539FB3DD/Application%20Discovery%20worksheet.xlsx) le ayudará a tomar las decisiones de migración rápidamente y a enviar una recomendación al grupo de negocios al instante.

La siguiente es información importante para tomar la decisión de migración:

- **Nombre de la aplicación**: ¿cómo se conoce esta aplicación en la empresa?

- **Tipo de aplicación**: ¿se trata de una aplicación SaaS de terceros? ¿Es una aplicación web de línea de negocio personalizada? ¿Es una API?

- **Importancia para la empresa**: ¿su importancia es alta? ¿Es baja? ¿O está en el medio?

- **Volumen de acceso de usuario**: ¿todos los usuarios acceden a esta aplicación, o solo unas pocas personas?

- **Duración planeada**: ¿cuánto durará esta aplicación? ¿Menos de 6 meses? ¿Más de 2 años?

- **Proveedor de identidades actual**: ¿cuál es el IdP principal para esta aplicación? ¿O depende del almacenamiento local?

- **Método de autenticación**: ¿la aplicación se autentica con estándares abiertos?

- **Si planea actualizar el código de la aplicación**: ¿la aplicación se encuentra en desarrollo activo o planificado?

- **Si planea mantener la aplicación en el entorno local**: ¿quiere conservar la aplicación en el centro de datos a largo plazo?

- **Si la aplicación depende de otras aplicaciones o API**: ¿la aplicación llama actualmente a otras aplicaciones o API?

- **Si la aplicación se encuentra en la galería de Azure AD**: ¿la aplicación ya está integrada en la [galería de Azure AD](https://azuremarketplace.microsoft.com/marketplace/apps/category/azure-active-directory-apps)?

Los siguientes son otros datos que le ayudarán más adelante, pero sobre los que no necesita tomar una decisión de migración inmediata:

- **Dirección URL de la aplicación**: ¿a dónde se dirigen los usuarios para acceder a la aplicación?

- **Descripción de la aplicación**: ¿cuál es una breve descripción de lo que hace la aplicación?

- **Propietario de la aplicación**: ¿qué persona de la empresa es la POC principal de la aplicación?

- **Comentarios o notas generales**: cualquier otra información general sobre la aplicación o la propiedad de la empresa.

Una vez que haya clasificado la aplicación y haya documentado los detalles, asegúrese de lograr que el propietario de la empresa participe en su estrategia de migración planeada.

### <a name="plan-a-pilot"></a>Planeamiento de un piloto

Las aplicaciones que seleccione para el piloto deben representar los requisitos importantes de identidad y seguridad de la organización, y debe tener una participación clara de los propietarios de la aplicación. Los pilotos suelen ejecutarse en un entorno de prueba independiente. Consulte los [procedimientos recomendados para un piloto](../fundamentals/active-directory-deployment-plans.md#best-practices-for-a-pilot) en la página de planes de implementación.

**No se olvide de los asociados externos.** Asegúrese de que participan en las programaciones y pruebas de migración. Por último, asegúrese de que tienen una manera de acceder al departamento de soporte técnico si surgen problemas importantes.

### <a name="plan-for-limitations"></a>Preparación para las limitaciones

Aunque algunas aplicaciones son fáciles de migrar, otras pueden tardar más tiempo debido a varios servidores o instancias. Por ejemplo, la migración de SharePoint puede tardar más tiempo debido a las páginas de inicio de sesión personalizadas.

Muchos proveedores de aplicaciones SaaS cobran por cambiar la conexión de SSO. Consulte con ellos y prepárese para este escenario.

Azure AD también tiene [límites de servicio y restricciones](../enterprise-users/directory-service-limits-restrictions.md) que debe tener en cuenta.

### <a name="app-owner-sign-off"></a>Aprobación del propietario de la aplicación

Las aplicaciones críticas para la empresa y usadas universalmente pueden necesitar un grupo de usuarios piloto para probar la aplicación en la fase piloto. Una vez que haya probado una aplicación en el entorno de preproducción o piloto, asegúrese de que los propietarios empresariales de la aplicación aprueben el rendimiento antes de la migración de la aplicación y todos los usuarios para el uso en producción de Azure AD para la autenticación.

### <a name="plan-the-security-posture"></a>Preparación de la postura de seguridad

Antes de iniciar el proceso de migración, tómese un momento para examinar a fondo la postura de seguridad que quiere desarrollar para el sistema de identidades corporativas. Esto se basa en los siguientes conjuntos de información valiosa: **Identidades, dispositivos y ubicaciones que acceden a los datos**.

### <a name="identities-and-data"></a>Identidades y datos

La mayoría de las organizaciones tienen requisitos específicos relacionados con la protección de datos e identidades que varían según el segmento del sector y las funciones del puesto en las organizaciones. Consulte el artículo de [configuraciones de identidad y acceso a dispositivos](/microsoft-365/enterprise/microsoft-365-policies-configurations) para conocer nuestras recomendaciones, incluido un conjunto de [directivas de acceso condicional](../conditional-access/overview.md) y funcionalidades relacionadas.

Puede usar esta información para proteger el acceso a todos los servicios integrados con Azure AD. Estas recomendaciones están alineadas con la puntuación de seguridad de Microsoft y con la [puntuación de identidad de Azure AD](../fundamentals/identity-secure-score.md). La puntuación le ayuda a:

- Medir de forma objetiva su nivel de seguridad de la identidad

- Planear la realización de mejoras en la seguridad de la identidad

- Ver si las mejoras han logrado sus objetivos

Esto también le ayudará a implementar los [cinco pasos para proteger la infraestructura de identidades](../../security/fundamentals/steps-secure-identity.md). Use la guía como punto inicial para su organización y ajuste las directivas para cumplir los requisitos específicos de su organización.

### <a name="who-is-accessing-your-data"></a>¿Quién accederá a los datos?

Azure AD admite dos categorías principales de usuarios para las aplicaciones y recursos:

- **Internos:** empleados, contratistas y proveedores que tengan cuentas en el proveedor de identidades. Esto podría requerir más dinamizaciones con diferentes reglas para los administradores o ejecutivos, en contraste con otros empleados.

- **Externos:** proveedores, distribuidores u otros asociados comerciales que interactúan con su organización en el curso normal de las actividades empresariales con la [colaboración B2B de Azure AD](../external-identities/what-is-b2b.md).

Puede definir grupos para estos usuarios y cubrirlos de diversas maneras. Puede optar por que un administrador deba agregar miembros manualmente a un grupo o puede habilitar el autoservicio de pertenencia al grupo. Se pueden establecer reglas que agreguen miembros automáticamente a los grupos según los criterios especificados con [grupos dinámicos](../enterprise-users/groups-dynamic-membership.md).

Los usuarios externos también pueden hacer referencia a los clientes. El producto independiente denominado [Azure AD B2C](../../active-directory-b2c/overview.md) admite la autenticación del cliente. No obstante, eso no se explica en estas notas.

### <a name="devicelocation-used-to-access-data"></a>Dispositivo/ubicación usados para acceder a los datos

También son importantes tanto el dispositivo como la ubicación que utiliza un usuario para acceder a una aplicación. Los dispositivos conectados físicamente a la red corporativa son más seguros. Puede que las conexiones establecidas fuera de la red a través de VPN deban evaluarse.

![Diagrama que muestra la relación entre la ubicación de usuario y el acceso a los datos](media/migrating-application-authentication-to-azure-active-directory-6.jpg)

Teniendo en cuenta estos aspectos del recurso, el usuario y el dispositivo, puede optar por usar las funcionalidades de [acceso condicional de Azure AD](../conditional-access/overview.md). El acceso condicional va más allá de los permisos de usuario: se basa en una combinación de factores, como la identidad de un usuario o un grupo, la red a la que está conectado el usuario, el dispositivo y la aplicación que está usando y el tipo de datos al que intenta acceder. El acceso que se concede al usuario se adapta a este conjunto más amplio de condiciones.

### <a name="exit-criteria"></a>Exit criteria (Criterios de salida)

Esta fase se realiza correctamente cuando se cumple lo siguiente:

- Conoce sus aplicaciones
  - Ha documentado por completo las aplicaciones que quiere migrar
  - Ha clasificado por orden de prioridad las aplicaciones en función de la importancia para la empresa, el volumen de uso y la duración

- Ha seleccionado aplicaciones que representen los requisitos de un piloto

- Ha logrado la participación del propietario de la empresa en la clasificación por orden de prioridad y la estrategia

- Comprende las necesidades de la postura seguridad y cómo implementarlas

## <a name="phase-3-plan-migration-and-testing"></a>Fase 3: planificación de la migración y pruebas

Una vez que haya logrado la participación de la empresa, el siguiente paso es empezar a migrar estas aplicaciones a la autenticación de Azure AD.

### <a name="migration-tools-and-guidance"></a>Guías y herramientas de migración

Use las herramientas y las guías siguientes para seguir los pasos exactos necesarios para migrar las aplicaciones a Azure AD:

- **Guía de migración general**: use las notas del producto, las herramientas, las plantillas de correo electrónico y el cuestionario de aplicaciones en el [kit de herramientas de migración de aplicaciones a Azure AD](./migration-resources.md) para detectar, clasificar y migrar sus aplicaciones.

- **Aplicaciones SaaS**: vea consulte la lista de [cientos de tutoriales de aplicaciones SaaS](../saas-apps/tutorial-list.md) y el [plan de implementación completo de SSO de Azure AD](https://aka.ms/ssodeploymentplan) para realizar paso a paso el proceso integral.

- **Aplicaciones que se ejecutan en el entorno local**: aprenda todo [sobre Azure AD Application Proxy](./application-proxy.md) y use el [plan de implementación completo de Azure AD Application Proxy](https://aka.ms/AppProxyDPDownload) para empezar a trabajar rápidamente.

- **Aplicaciones que está desarrollando**: lea nuestra guía paso a paso de [integración](../develop/quickstart-register-app.md) y [registro](../develop/quickstart-register-app.md).

Después de la migración, puede optar por enviar la comunicación que informa a los usuarios de la implementación correcta y recordarles los pasos nuevos que deben seguir.

### <a name="plan-testing"></a>Planeamiento de pruebas

Durante el proceso de migración, es posible que la aplicación ya tenga un entorno de prueba que se usa durante las implementaciones normales. Puede seguir usando este entorno para las pruebas de migración. Si actualmente no hay disponible un entorno de prueba, quizá pueda configurar uno con Azure App Service o Azure Virtual Machines, en función de la arquitectura de la aplicación. Puede optar por configurar un inquilino de Azure AD de prueba independiente para usarlo cuando desarrolle las configuraciones de las aplicaciones. Este inquilino se iniciará en un estado limpio y no estará configurado para sincronizarse con ningún sistema.

Para probar todas las aplicaciones, inicie sesión con un usuario de prueba y asegúrese de que todas las funcionalidades son las mismas que antes de la migración. Si durante las pruebas determina que los usuarios deberán actualizar la configuración de [MFA](../authentication/howto-mfa-userstates.md) o [SSPR](../authentication/tutorial-enable-sspr.md), o si agrega esta funcionalidad durante la migración, asegúrese de agregarla al plan de comunicaciones para el usuario final. Consulte las plantillas de comunicación con el usuario final de [MFA](https://aka.ms/mfatemplates) y [SSPR](https://aka.ms/ssprtemplates).

Una vez que haya migrado las aplicaciones, vaya a [Azure Portal](https://aad.portal.azure.com/) para probar si la migración se realizó correctamente. Siga las instrucciones que se describen a continuación:

- Seleccione **Aplicaciones empresariales &gt; Todas las aplicaciones** y busque la aplicación en la lista.

- Seleccione **Administrar &gt; Usuarios y grupos** para asignar al menos un usuario o grupo a la aplicación.

- Seleccione **Administrar &gt; Acceso condicional**. Revise la lista de directivas y asegúrese de que no está bloqueando el acceso a la aplicación con una [directiva de acceso condicional](../conditional-access/overview.md).

En función de cómo configure la aplicación, compruebe que el inicio de sesión único funciona correctamente.

| Tipo de autenticación      | Prueba                                             |
| ------------------------ | --------------------------------------------------- |
| **OAuth / OpenID Connect** | Seleccione **Aplicaciones empresariales &gt; Permisos** y asegúrese de haber dado su consentimiento a que la aplicación se use en la organización desde la configuración del usuario correspondiente a la aplicación. |
| **SSO basado en SAML** | Use el botón [Probar la configuración de SAML](./debug-saml-sso-issues.md) que está en **Inicio de sesión único.** |
| **SSO basado en contraseñas** | Descarga e instalación de la [extensión de inicio de sesión seguro de Mis aplicaciones](../user-help/my-apps-portal-end-user-access.md#download-and-install-the-my-apps-secure-sign-in-extension). Esta extensión lo ayudará a iniciar cualquiera de las aplicaciones en la nube de su organización que requiera que se use un proceso de inicio de sesión único. |

|  **[Application Proxy](./application-proxy.md)** | Asegúrese de que el conector esté en ejecución y de que esté asignado a la aplicación. Visite la [Solución de problemas y mensajes de error de Proxy de aplicación](./application-proxy-troubleshoot.md) para obtener más ayuda. |

### <a name="troubleshoot"></a>Solución de problemas

Si tiene problemas, consulte nuestra [Guía de solución de problemas de aplicaciones](../app-provisioning/isv-automatic-provisioning-multi-tenant-apps.md) para obtener ayuda. También puede consultar nuestros artículos de solución de problemas; vea [Problemas al iniciar sesión en aplicaciones configuradas de inicio de sesión único basadas en SAML](/troubleshoot/azure/active-directory/troubleshoot-sign-in-saml-based-apps).

### <a name="plan-rollback"></a>Planeamiento de la reversión

Si se produce un error en la migración, la mejor estrategia es la reversión y la prueba. Estos son los pasos que puede seguir para mitigar los problemas de migración:

- **Realice capturas de pantalla** de la configuración existente de la aplicación. Puede consultarlas si necesita volver a configurar la aplicación.

- También puede **proporcionar vínculos a la autenticación heredada** si surgen problemas con la autenticación en la nube.

- Antes de completar la migración, **no cambie la configuración existente** con el proveedor de identidades anterior.

- Comience por migrar **las aplicaciones que admiten varios IdP**. Si algo va mal, siempre puede cambiar a la configuración del IdP preferido.

- Asegúrese de que la experiencia de la aplicación tenga un botón **Comentarios** o punteros al **departamento de soporte técnico** en caso de problemas.

### <a name="exit-criteria"></a>Exit criteria (Criterios de salida)

Esta fase se realiza correctamente cuando ha hecho lo siguiente:

- Determinó cómo se migrará cada aplicación

- Revisó las herramientas de migración

- Planeó las pruebas, incluidos los grupos y entornos de prueba

- Planeó la reversión

## <a name="phase-4-plan-management-and-insights"></a>Fase 4: administración del plan y conclusiones

Una vez que se hayan migrado las aplicaciones, debe asegurarse de que:

- Los usuarios pueden acceder y administrar de forma segura.

- Puede obtener las conclusiones adecuadas sobre el uso y el estado de la aplicación.

Se recomienda realizar las siguientes acciones según sea necesario para la organización.

### <a name="manage-your-users-app-access"></a>Administración del acceso a la aplicación de los usuarios

Una vez que haya migrado las aplicaciones, puede enriquecer la experiencia del usuario de muchas maneras.

**Haga que las aplicaciones se puedan buscar**

**Dirija al usuario** a la experiencia del portal [Mis aplicaciones](../user-help/my-apps-portal-end-user-access.md#download-and-install-the-my-apps-secure-sign-in-extension). Aquí, podrá acceder a todas las aplicaciones basadas en la nube, a las aplicaciones que pone a su disposición mediante [Azure AD Connect](../hybrid/whatis-azure-ad-connect.md) y a las aplicaciones que usan [Application Proxy](./application-proxy.md), siempre que tenga permisos de acceso a esas aplicaciones.


Puede guiar a los usuarios en la búsqueda de sus aplicaciones:

- Use la característica de [inicio de sesión único existente](./view-applications-portal.md) para **vincular a los usuarios a cualquier aplicación**.


- Habilite el [acceso de autoservicio a las aplicaciones](./manage-self-service-access.md) de una aplicación y **permita que los usuarios agreguen aplicaciones que usted mantiene**.

- [Oculte las aplicaciones de los usuarios finales](./hide-application-from-user-portal.md) (aplicaciones predeterminadas de Microsoft u otras aplicaciones) para **hacer que las aplicaciones que necesitan sean más fáciles de encontrar**.

### <a name="make-apps-accessible"></a>Hacer que las aplicaciones sean accesibles

**Permita que los usuarios accedan a las aplicaciones desde sus dispositivos móviles**. Los usuarios pueden acceder al portal Mis aplicaciones con el explorador administrado de Intune en sus dispositivos [iOS](./hide-application-from-user-portal.md) versión 7.0 o posterior o [Android](./hide-application-from-user-portal.md).

Los usuarios pueden descargar un **explorador administrado de Intune**:

- **Para dispositivos Android**, desde [Google Play Store](https://play.google.com/store/apps/details?id=com.microsoft.intune).

- **Para dispositivos de Apple**, desde [Apple App Store](https://itunes.apple.com/us/app/microsoft-intune-managed-browser/id943264951?mt=8) o pueden descargar la [aplicación móvil Mis aplicaciones para iOS](https://apps.apple.com/us/app/my-apps-azure-active-directory/id824048653).

**Permita que los usuarios abran las aplicaciones desde una extensión del navegador.**

Los usuarios pueden [descargar la extensión de inicio de sesión seguro de Mis aplicaciones](https://www.microsoft.com/p/my-apps-secure-sign-in-extension/9pc9sckkzk84?rtc=1&activetab=pivot%3Aoverviewtab) en [Chrome](https://chrome.google.com/webstore/detail/my-apps-secure-sign-in-ex/ggjhpefgjjfobnfoldnjipclpcfbgbhl), [Firefox](https://addons.mozilla.org/firefox/addon/access-panel-extension/) o [Microsoft Edge](https://www.microsoft.com/p/my-apps-secure-sign-in-extension/9pc9sckkzk84?rtc=1&activetab=pivot%3Aoverviewtab) y pueden iniciar las aplicaciones directamente desde la barra del explorador para:

- **Buscar sus aplicaciones y hacer que se muestren sus aplicaciones usadas más recientemente**.

- **Convertir automáticamente las direcciones URL internas** que haya configurado en [Application Proxy](./application-proxy.md) en las direcciones URL externas adecuadas. Los usuarios ahora pueden trabajar con los vínculos con los que están familiarizados, independientemente de dónde se encuentren.

**Permita que los usuarios abran las aplicaciones desde Office.com.**

Los usuarios pueden ir a [Office.com](https://www.office.com/) para **buscar las aplicaciones y hacer que se muestren las aplicaciones usadas más recientemente** justo desde el sitio en el que realizan su trabajo.

### <a name="secure-app-access"></a>Protección del acceso a las aplicaciones

Azure AD proporciona una ubicación de acceso centralizada para administrar las aplicaciones migradas. Vaya a [Azure Portal](https://portal.azure.com/) y habilite las siguientes funcionalidades:

- **Proteger el acceso de los usuarios a las aplicaciones.** Habilite las [directivas de acceso condicional](../conditional-access/overview.md) o [Identity Protection](../identity-protection/overview-identity-protection.md) para proteger el acceso de los usuarios a las aplicaciones en función del estado del dispositivo, la ubicación, etc.

- **Aprovisionamiento automático.** Configure el [aprovisionamiento automático de usuarios](../app-provisioning/user-provisioning.md) con una variedad de aplicaciones SaaS de terceros a las que los usuarios necesitan acceder. Además de crear identidades de usuario, incluye el mantenimiento y la eliminación de identidades de usuario a medida que el estado o los roles cambian.

- **Delegar la administración** **de acceso de usuarios**. Según sea necesario, habilite el acceso de autoservicio a las aplicaciones y *asigne un aprobador de la empresa para aprobar el acceso a esas aplicaciones*. Use la [administración de grupos de autoservicio](../enterprise-users/groups-self-service-management.md) para los grupos asignados a las colecciones de aplicaciones.

- **Delegar el acceso de administrador.** Use el **rol de directorio** para asignar un rol de administrador (por ejemplo, administrador de aplicaciones, administrador de aplicaciones en la nube o desarrollador de aplicaciones) al usuario.

### <a name="audit-and-gain-insights-of-your-apps"></a>Auditoría y obtención de conclusiones sobre las aplicaciones

También puede usar [Azure Portal](https://portal.azure.com/) para auditar todas las aplicaciones desde una ubicación centralizada.

- **Audite su aplicación** mediante la **auditoría de aplicaciones empresariales u obtenga acceso a la misma información de la [API de informes de Azure AD](../reports-monitoring/concept-reporting-api.md) para integrarla en sus herramientas favoritas.

- **Vea los permisos de una aplicación** mediante los **permisos de aplicaciones empresariales** para las aplicaciones que usan OAuth u OpenID Connect.

- **Obtenga conclusiones del inicio de sesión** con los **inicios de sesión de aplicaciones empresariales**. Acceda a la misma información de la [API de informes de Azure AD](../reports-monitoring/concept-reporting-api.md).

- **Visualice el uso de la aplicación** desde el [paquete de contenido de Azure AD Power BI](../reports-monitoring/howto-use-azure-monitor-workbooks.md).

### <a name="exit-criteria"></a>Exit criteria (Criterios de salida)

Esta fase se realiza correctamente cuando se cumple lo siguiente:

- Proporciona acceso seguro a las aplicaciones para los usuarios.

- Logra auditar y obtener conclusiones sobre las aplicaciones migradas.

### <a name="do-even-more-with-deployment-plans"></a>Hace aún más con los planes de implementación.

Los planes de implementación explican el valor empresarial, la planificación, los pasos de implementación y la administración de las soluciones de Azure AD, incluidos los escenarios de migración de aplicaciones. Reúnen todo lo que necesita para empezar la implementación y aprovechar las funcionalidades de Azure AD. Las guías de implementación incluyen contenido como procedimientos recomendados de Microsoft, comunicaciones para el usuario final, guías de planeación, pasos de implementación, casos de prueba, etc.

Hay muchos [planes de implementación](../fundamentals/active-directory-deployment-plans.md) disponibles y siempre estamos creando más.

### <a name="contact-support"></a>Ponerse en contacto con soporte técnico

Visite los siguientes vínculos de soporte técnico para crear una incidencia de soporte técnico o realizar el seguimiento de la misma, así como supervisar el mantenimiento.

- **Soporte técnico de Azure:** puede llamar al [Soporte técnico de Microsoft](https://azure.microsoft.com/support) y crear una incidencia para cualquier problema de implementación de Azure

Identity en función del Contrato Enterprise con Microsoft.

- **FastTrack**: si ha adquirido licencias de Enterprise Mobility and Security (EMS) o Azure AD Premium, puede recibir asistencia de implementación del [programa FastTrack.](/enterprise-mobility-security/solutions/enterprise-mobility-fasttrack-program)

- **Comuníquese con el equipo de ingeniería de producto:** si está trabajando en una implementación de clientes principales con millones de usuarios, tiene derecho a recibir soporte técnico del equipo de cuentas de Microsoft o el arquitecto de soluciones en la nube. En función de la complejidad de la implementación del proyecto, puede trabajar directamente con el [equipo de ingenieros de productos de identidad de Azure.](https://aad.portal.azure.com/#blade/Microsoft_Azure_Marketplace/MarketplaceOffersBlade/selectedMenuItemId/solutionProviders)

- **Blog de identidad de Azure AD:** suscríbase al [blog de identidad de Azure AD](https://techcommunity.microsoft.com/t5/Azure-Active-Directory-Identity/bg-p/Identity) para mantenerse al día con todos los anuncios de producto más recientes, análisis detallados e información de hoja de ruta que proporciona directamente el equipo de ingenieros de Identity.