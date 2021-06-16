---
title: Protección de Microsoft 365 contra ataques locales
description: Guía sobre cómo asegurar que un ataque local no afecte a Microsoft 365.
services: active-directory
author: BarbaraSelden
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.subservice: fundamentals
ms.topic: conceptual
ms.date: 12/22/2020
ms.author: baselden
ms.reviewer: ajburnle
ms.custom: it-pro, seodec18
ms.collection: M365-identity-device-management
ms.openlocfilehash: 5cee69fbd2cb829a7ec90179e94098e307e5a094
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111968529"
---
# <a name="protecting-microsoft-365-from-on-premises-attacks"></a>Protección de Microsoft 365 contra ataques locales

Muchos clientes conectan sus redes corporativas privadas a Microsoft 365 para beneficiar a sus usuarios, dispositivos y aplicaciones. Sin embargo, estas redes privadas pueden verse en peligro de muchas formas bien documentadas. Dado que Microsoft 365 actúa como una especie de sistema nervioso de muchas organizaciones, es fundamental protegerlo de una infraestructura local en peligro.

En este artículo se muestra cómo configurar los sistemas para proteger el entorno en la nube de Microsoft 365 frente a riesgos locales. Nos centramos principalmente en lo siguiente: 

- Las opciones de configuración de los inquilinos de Azure Active Directory (Azure AD).
- La manera en que los inquilinos de Azure AD se pueden conectar de manera segura a sistemas locales.
- Las concesiones necesarias para que los sistemas funcionen de formas que protejan los sistemas en la nube frente a los peligros locales.

Se recomienda encarecidamente implementar esta guía para proteger el entorno en la nube de Microsoft 365.
> [!NOTE]
> Este artículo se publicó inicialmente como una entrada de blog. Se movió a su ubicación actual por motivos de durabilidad y mantenimiento.
>
> Para crear una versión sin conexión de este artículo, use la funcionalidad de impresión en PDF del explorador. Aquí puede comprobar con frecuencia si hay actualizaciones.

## <a name="primary-threat-vectors-from-compromised-on-premises-environments"></a>Vectores de amenazas principales de entornos locales en peligro


El entorno en la nube de Microsoft 365 se beneficia de una amplia infraestructura de supervisión y seguridad. Con el aprendizaje automático y la inteligencia humana, Microsoft 365 busca en todo el tráfico mundial. Puede detectar rápidamente ataques y le permite reconfigurarlo casi en tiempo real. 

En las implementaciones híbridas que conectan la infraestructura local a Microsoft 365, muchas organizaciones delegan la confianza en componentes locales para tomar decisiones críticas de administración del estado de los objetos de directorio y autenticación.
Desafortunadamente, si el entorno local está en peligro, estas relaciones de confianza se convierten en la oportunidad para que un atacante ponga en peligro el entorno de Microsoft 365.

Los dos vectores de amenazas principales son las *relaciones de confianza de federación* y la *sincronización de cuentas*. Ambos vectores pueden conceder a un atacante acceso administrativo a la nube.

* Las **relaciones de confianza federadas**, como la autenticación SAML, se usan para autenticarse en Microsoft 365 a través de la infraestructura de identidad local. Si un certificado de firma de tokens de SAML está en peligro, la federación permitiría a cualquier usuario que tenga ese certificado suplantar a cualquier usuario en la nube. *Se recomienda deshabilitar las relaciones de confianza de federación para la autenticación en Microsoft 365 cuando sea posible*.

* La **sincronización de cuentas** se puede usar para modificar los usuarios con privilegios (incluidas sus credenciales) o los grupos que tienen privilegios administrativos en Microsoft 365. *Se recomienda asegurarse de que los objetos sincronizados no tengan privilegios más allá de un usuario en Microsoft 365,* directamente o a través de la inclusión en roles o grupos de confianza. Asegúrese de que estos objetos no tienen ninguna asignación directa o anidada en grupos o roles en la nube de confianza.

## <a name="protecting-microsoft-365-from-on-premises-compromise"></a>Protección de Microsoft 365 contra vulnerabilidades locales


Para abordar los vectores de amenazas descritos anteriormente, se recomienda cumplir los principios que se ilustran en el diagrama a continuación:

![Arquitectura de referencia para proteger Microsoft 365.](media/protect-m365/protect-m365-principles.png)

1. **Aísle por completo las cuentas de administrador de Microsoft 365.** Deben cumplir los siguientes requisitos:

    * Deben controlarse en Azure AD.

     * Deben autenticarse mediante la autenticación multifactor.

     *  Deben protegerse con el acceso condicional de Azure AD.

     *  Debe accederse a ellas solo mediante las estaciones de trabajo administradas de Azure.

    Estas cuentas de administrador son cuentas de uso restringido. *Ninguna cuenta local debe tener privilegios administrativos en Microsoft 365.* 

    Para obtener más información, consulte la [información general sobre los roles de administrador de Microsoft 365](/microsoft-365/admin/add-users/about-admin-roles). Consulte también [Roles para Microsoft 365 en Azure AD](../roles/m365-workload-docs.md).

1. **Administre dispositivos desde Microsoft 365**. Use la administración de dispositivos móviles (MDM) basados en la nube y unidos a Azure AD para eliminar dependencias en la infraestructura de administración de dispositivos local. Estas dependencias pueden comprometer los controles de seguridad y los dispositivos.

1. **Asegúrese de que ninguna cuenta local tenga privilegios elevados para Microsoft 365.**
    Algunas cuentas tienen acceso a aplicaciones locales que requieren autenticación NTLM, LDAP o Kerberos. Estas cuentas deben estar en la infraestructura de identidad local de la organización. Asegúrese de que estas cuentas, incluidas las cuentas de servicio, no estén incluidas en grupos ni roles en la nube con privilegios. Asegúrese también de que los cambios en estas cuentas no afecten a la integridad de su entorno de nube. El software local con privilegios no debe ser capaz de afectar a los roles ni a las cuentas con privilegios de Microsoft 365.

1. **Use la autenticación en la nube de Azure AD** para eliminar las dependencias de las credenciales locales. Use siempre una autenticación sólida, como Windows Hello, FIDO, Microsoft Authenticator o Azure AD Multifactor Authentication.

## <a name="specific-security-recommendations"></a>Recomendaciones de seguridad específicas


En las secciones siguientes se proporcionan instrucciones específicas sobre cómo implementar los principios descritos anteriormente.

### <a name="isolate-privileged-identities"></a>Aislamiento de identidades con privilegios


En Azure AD, los usuarios con roles con privilegios, como los administradores, son la raíz de confianza para crear y administrar el resto del entorno. Implemente las siguientes prácticas para minimizar los efectos de un riesgo.

* Use cuentas solo en la nube para los roles con privilegios de Azure AD y Microsoft 365.

* Implemente [dispositivos de acceso con privilegios](/security/compass/privileged-access-devices#device-roles-and-profiles) para el acceso con privilegios para administrar Microsoft 365 y Azure AD.

*  Implemente [Azure AD Privileged Identity Management](../privileged-identity-management/pim-configure.md) (PIM) para el acceso Just-in-Time (JIT) a todas las cuentas de usuarios que tienen roles con privilegios. Exija una autenticación sólida para activar los roles.

* Proporcione roles administrativos que permitan el [privilegio mínimo necesario para realizar las tareas requeridas](../roles/delegate-by-task.md).

* Para habilitar una experiencia de asignación de roles más enriquecida que incluya la delegación y varios roles al mismo tiempo, considere la posibilidad de usar grupos de seguridad de Azure AD o grupos de Microsoft 365. En su conjunto, estos grupos se denominan *grupos de nube*. Además, [habilite el control de acceso basado en roles](../roles/groups-assign-role.md). Puede usar [unidades administrativas](../roles/administrative-units.md) para restringir el ámbito de los roles a una parte de la organización.

* Implemente [cuentas de acceso de emergencia](../roles/security-emergency-access.md). *No* use almacenes de contraseñas locales para almacenar las credenciales.

Para obtener más información, consulte [Protección del acceso con privilegios](/security/compass/overview). Consulte también [Procedimientos de acceso seguro para administradores en Azure AD](../roles/security-planning.md).

### <a name="use-cloud-authentication"></a>Uso de la autenticación en la nube 

Las credenciales son un vector de ataque principal. Implemente los siguientes procedimientos para mejorar la seguridad de las credenciales:

* [Implementación de una autenticación sin contraseña](../authentication/howto-authentication-passwordless-deployment.md). reduzca el uso de contraseñas tanto como sea posible mediante la implementación de credenciales sin contraseña. Estas credenciales se administran y validan de forma nativa en la nube. Elija entre los métodos de autenticación siguientes:

   * [Windows Hello para empresas](/windows/security/identity-protection/hello-for-business/passwordless-strategy)

   * [La aplicación Microsoft Authenticator](../authentication/howto-authentication-passwordless-phone.md)

   * [Llaves de seguridad FIDO2](../authentication/howto-authentication-passwordless-security-key-windows.md)

* [Implementación de Multifactor Authentication](../authentication/howto-mfa-getstarted.md). Aprovisione [varias credenciales seguras mediante Azure AD Multifactor Authentication](../fundamentals/resilience-in-credentials.md). De este modo, el acceso a los recursos en la nube requerirá una credencial administrada en Azure AD, además de una contraseña local que se pueda manipular. Para obtener más información, consulte [Creación de una estrategia de administración de control de acceso resistente con Azure AD](./resilience-overview.md).

### <a name="limitations-and-tradeoffs"></a>Limitaciones y compromisos

* La administración de contraseñas de cuentas híbrida requiere componentes híbridos, como agentes de protección de contraseñas y agentes de escritura diferida de contraseñas Si la infraestructura local está en peligro, los atacantes pueden controlar los equipos en los que residen estos agentes. Esta vulnerabilidad no pondrá en peligro la infraestructura en la nube. Sin embargo, las cuentas en la nube no protegerán estos componentes frente al riesgo local.

*  Las cuentas locales sincronizadas desde Active Directory se marcan para que no expiren nunca en Azure AD. Esta configuración se suele mitigar mediante la configuración local de contraseñas de Active Directory. Sin embargo, si la instancia local de Active Directory está en peligro y la sincronización está deshabilitada, se debe establecer la opción [EnforceCloudPasswordPolicyForPasswordSyncedUsers](../hybrid/how-to-connect-password-hash-synchronization.md) para forzar los cambios de contraseña.

## <a name="provision-user-access-from-the-cloud"></a>Aprovisionamiento del acceso de usuarios desde la nube

El *aprovisionamiento* se refiere a la creación de cuentas de usuario y grupos en aplicaciones o proveedores de identidades.

![Diagrama de la arquitectura de aprovisionamiento.](media/protect-m365/protect-m365-provision.png)

Se recomiendan los siguientes métodos de aprovisionamiento:

* **Aprovisionamiento de aplicaciones de RR. HH. en la nube en Azure AD**: Este aprovisionamiento permite aislar un riesgo local sin interrumpir el ciclo de tipo alta, baja o traslado desde las aplicaciones de RR. HH. en la nube a Azure AD.

* **Aplicaciones en la nube**: Siempre que sea posible, implemente el [aprovisionamiento de aplicaciones de Azure AD](../app-provisioning/user-provisioning.md), en lugar de las soluciones de aprovisionamiento local. Este método protege algunas de las aplicaciones de software como servicio (SaaS) para que no se vean afectadas por perfiles de hackers malintencionados en vulneraciones locales. 

* **Identidades externas**: use la [colaboración B2B de Azure AD](../external-identities/what-is-b2b.md).
    Este método reduce la dependencia de las cuentas locales frente a la colaboración externa con asociados, clientes y proveedores. Evalúe detenidamente cualquier federación directa con otros proveedores de identidades. Se recomienda limitar las cuentas de invitado de B2B de las siguientes maneras:

   *  Limitar el acceso de invitado a los grupos de exploración y otras propiedades del directorio. Use la configuración de colaboración externa para restringir la capacidad de los invitados de leer los grupos de los que no son miembros. 

    *   Bloquear el acceso a Azure Portal. Puede hacer excepciones raras, pero necesarias.  Cree una directiva de acceso condicional que incluya a todos los usuarios invitados y externos. Luego [implemente una directiva para bloquear el acceso](../../role-based-access-control/conditional-access-azure-management.md). 

* **Bosques desconectados**: Use el [aprovisionamiento en la nube de Azure AD](../cloud-sync/what-is-cloud-sync.md). Este método le permite conectarse a los bosques desconectados, eliminando la necesidad de establecer una conexión o relaciones de confianza entre los bosques, lo que puede ampliar el efecto de una vulneración local. 
 
### <a name="limitations-and-tradeoffs"></a>Limitaciones y concesiones

Cuando se usan para aprovisionar cuentas híbridas, Azure AD en los sistemas de RR. HH. en la nube se basa en la sincronización local para completar el flujo de datos de Active Directory a Azure AD. Si se interrumpe la sincronización, los nuevos registros de empleados no estarán disponibles en Azure AD.

## <a name="use-cloud-groups-for-collaboration-and-access"></a>Uso de los grupos en la nube para la colaboración y el acceso

Los grupos en la nube permiten desacoplar la colaboración y el acceso de la infraestructura local.

* **Colaboración**: use Grupos de Microsoft 365 y Microsoft Teams para la colaboración moderna. Retire las listas de distribución locales y [actualice las listas de distribución a grupos de Microsoft 365 en Outlook](/office365/admin/manage/upgrade-distribution-lists).

* **Acceso**: use grupos de seguridad de Azure AD o Grupos de Microsoft 365 para autorizar el acceso a aplicaciones en Azure AD.
* **Licencias de Office 365**: Use las licencias basadas en grupos para aprovisionar en Office 365 usando grupos solo en la nube. Este método desacopla el control de la pertenencia a grupos de la infraestructura local.

Los propietarios de los grupos que se usan para el acceso deben considerarse identidades con privilegios para evitar la toma del control de la pertenencia a partir de un riesgo local.
Una toma de control incluiría la manipulación directa de la pertenencia a grupos local o la manipulación de atributos locales que pueden afectar a la pertenencia dinámica a grupos en Microsoft 365.

## <a name="manage-devices-from-the-cloud"></a>Administrar dispositivos de la nube


Use las funcionalidades de Azure AD para administrar dispositivos de forma segura.

-   **Use estaciones de trabajo de Windows 10**: [implemente dispositivos unidos a Azure AD](../devices/azureadjoin-plan.md) con directivas MDM. Habilite [Windows Autopilot](/mem/autopilot/windows-autopilot) para una experiencia de aprovisionamiento totalmente automatizada.

    -   Deje de usar máquinas que ejecuten Windows 8.1 y versiones anteriores.

    -   No implemente máquinas de sistemas operativos de servidor como estaciones de trabajo.

    -   Use [Microsoft Intune](https://www.microsoft.com/microsoft-365/enterprise-mobility-security/microsoft-intune) como origen de la autoridad de todas las cargas de trabajo de administración de dispositivos.

-   [**Implemente dispositivos de acceso con privilegios**](/security/compass/privileged-access-devices#device-roles-and-profiles): use el acceso con privilegios para administrar Microsoft 365 y Azure AD.

## <a name="workloads-applications-and-resources"></a>Cargas de trabajo, aplicaciones y recursos 

-   **Sistemas locales de inicio de sesión único (SSO)** 

    Deje de usar toda infraestructura local de administración de acceso web y federación. Configure las aplicaciones para usar Azure AD.  

-   **Aplicaciones SaaS y de línea de negocio (LOB) que admiten protocolos de autenticación modernos** 

    [Use Azure AD para SSO](../manage-apps/what-is-single-sign-on.md). Cuanto más aplicaciones configure para usar Azure AD para la autenticación, menor será el riesgo en caso de ataque local.


* **Aplicaciones heredadas** 

   * Puede habilitar la autenticación, la autorización y el acceso remoto a las aplicaciones heredadas que no admitan la autenticación moderna. Use [Application Proxy de Azure AD](../app-proxy/application-proxy.md). También puede habilitarlas a través de una solución de controlador de entrega de aplicaciones o de red mediante [integraciones de asociados de acceso híbrido seguro](../manage-apps/secure-hybrid-access.md).   

   * Elija un proveedor de VPN que admita la autenticación moderna. Integre la autenticación en Azure AD. En caso de que sea un riesgo local, puede usar Azure AD para deshabilitar o bloquear el acceso deshabilitando la VPN.

*  **Servidores de cargas de trabajo y aplicaciones**

   * Las aplicaciones o los recursos que necesitaban servidores se pueden migrar a la infraestructura como servicio (IaaS) de Azure. Use [Azure AD Domain Services](../../active-directory-domain-services/overview.md) (Azure AD DS) para desacoplar la confianza y la dependencia en instancias locales de Active Directory. Para lograr este desacoplamiento, asegúrese de que las redes virtuales que se usan para Azure AD DS no tengan conexión a las redes corporativas.

   * Siga las instrucciones para los [niveles de credenciales](/security/compass/privileged-access-access-model#ADATM_BM). Los servidores de aplicaciones se suelen considerar recursos de nivel 1.

## <a name="conditional-access-policies"></a>Directivas de acceso condicional

Use el acceso condicional de Azure AD para interpretar las señales y usarlas para tomar decisiones de autenticación. Para obtener más información, consulte [Planeamiento de la implementación del acceso condicional](../conditional-access/plan-conditional-access.md).

* Use el acceso condicional para [bloquear los protocolos de autenticación heredados](../conditional-access/howto-conditional-access-policy-block-legacy.md) siempre que sea posible. Además, deshabilite los protocolos de autenticación heredados en el nivel de aplicación mediante la configuración específica de una aplicación.

   Para obtener más información, consulte [Protocolos de autenticación heredados](../fundamentals/auth-sync-overview.md). O bien, consulte los detalles específicos de [Exchange Online](/exchange/clients-and-mobile-in-exchange-online/disable-basic-authentication-in-exchange-online#how-basic-authentication-works-in-exchange-online) y [SharePoint Online](/powershell/module/sharepoint-online/set-spotenant).

* Implemente las [configuraciones de acceso de dispositivos e identidades](/microsoft-365/security/office-365-security/identity-access-policies) recomendadas.

* Si usa una versión de Azure AD que no incluye el acceso condicional, asegúrese de usar los [valores predeterminados de seguridad de Azure AD](../fundamentals/concept-fundamentals-security-defaults.md).

   Para obtener más información sobre las licencias de características de Azure AD, consulte la [guía de precios de Azure AD](https://azure.microsoft.com/pricing/details/active-directory/).

## <a name="monitor"></a>Supervisión 

Después de haber configurado el entorno para proteger Microsoft 365 de un riesgo local, [supervise proactivamente](../reports-monitoring/overview-monitoring.md) el entorno.
### <a name="scenarios-to-monitor"></a>Escenarios que supervisar

Supervise los siguientes escenarios clave, además de los escenarios específicos de su organización. Por ejemplo, debe supervisar de forma proactiva el acceso a sus aplicaciones y recursos críticos para la empresa.

* **Actividad sospechosa** 

    Supervise todos los [eventos de riesgo de Azure AD](../identity-protection/overview-identity-protection.md#risk-detection-and-remediation) en busca de actividad sospechosa. [Azure AD Identity Protection](../identity-protection/overview-identity-protection.md) se integra de forma nativa con Azure Security Center.

    Defina las [ubicaciones con nombre](../conditional-access/location-condition.md) de red para evitar detecciones ruidosas en las señales basadas en la ubicación. 
*  **Alertas de análisis de comportamiento de usuarios y entidades (UEBA)** 

    Use UEBA para obtener conclusiones sobre la detección de anomalías.
    * Microsoft Cloud App Security (MCAS) proporciona [UEBA en la nube](/cloud-app-security/tutorial-ueba).

    * Puede [integrar UEBA local desde la protección contra amenazas avanzada (ATP) de Azure](/defender-for-identity/install-step2). MCAS lee las señales de Azure AD Identity Protection. 

* **Actividad de las cuentas de acceso de emergencia** 

    Supervise todo acceso que use [cuentas de acceso de emergencia](../roles/security-emergency-access.md). Cree alertas para iniciar investigaciones. Esta supervisión debe incluir lo siguiente: 

   * Inicios de sesión. 

   * Administración de credenciales. 

   * Todas las actualizaciones de pertenencias a grupos. 

   * Asignaciones de aplicaciones. 
* **Actividad de roles con privilegios**

    Configure y revise las [alertas de seguridad generadas por Azure AD Privileged Identity Management (PIM)](../privileged-identity-management/pim-how-to-configure-security-alerts.md?tabs=new#security-alerts).
    Supervise la asignación directa de roles con privilegios fuera de PIM mediante la generación de alertas cada vez que se asigne un usuario directamente.

* **Configuraciones de todo el inquilino en Azure AD**

    cualquier cambio en las configuraciones de todo el inquilino debe generar alertas en el sistema. Estos cambios incluyen, entre otros:

  *  Dominios personalizados actualizados.  

  * Cambios de Azure AD B2B en las listas de permitidos y listas de bloqueados.

  * Cambios de Azure AD B2B en los proveedores de identidades permitidos (proveedores de identidades de SAML mediante federación directa o inicios de sesión sociales).  

  * Cambios en la directiva de riesgo o de acceso condicional. 

* **Objetos de aplicación y de entidad de servicio**
    
   * Nuevas aplicaciones o entidades de servicio que pueden requerir directivas de acceso condicional 

   * Credenciales agregadas a las entidades de servicio.
   * Actividad de consentimiento de la aplicación 

* **Roles personalizados**
   * Actualizaciones en las definiciones de roles personalizados. 

   * Roles personalizados recién creados. 

### <a name="log-management"></a>Administración de registros

Defina una estrategia, un diseño y una implementación de almacenamiento y retención de registros para facilitar un conjunto de herramientas coherente. Por ejemplo, puede considerar sistemas de Administración de eventos e información de seguridad (SIEM), como Azure Sentinel, las consultas comunes, y los cuadernos de estrategias de investigación y análisis forenses.

* **Registros de Azure AD**: Ingiera los registros y las señales generados de forma coherente siguiendo los procedimientos recomendados de configuración, como la ingesta de diagnóstico, de retención de registros y de SIEM. 

    La estrategia de registro debe incluir los siguientes registros de Azure AD:
   * Actividad de inicio de sesión 

   * Registros de auditoría 

   * Eventos de riesgo 

    Azure AD proporciona la [integración de Azure Monitor](../reports-monitoring/concept-activity-logs-azure-monitor.md) para los registros de auditoría y el registro de actividad de inicio de sesión. Los eventos de riesgo se pueden ingerir mediante [Microsoft Graph API](/graph/api/resources/identityriskevent). Puede [transmitir registros de Azure AD a registros de Azure Monitor](../reports-monitoring/howto-integrate-activity-logs-with-log-analytics.md).

* **Registros de seguridad del sistema operativo de infraestructura híbrida**: Todos los registros del sistema operativo de infraestructura de identidad híbrida deben archivarse y supervisarse detenidamente como un sistema de nivel 0, dadas las consecuencias en el área expuesta. Incluya los siguientes elementos: 

   *  Azure AD Connect. [Azure AD Connect Health](../hybrid/whatis-azure-ad-connect.md) debe implementarse para supervisar la sincronización de identidades.

   *  Agentes de Application Proxy 


   * Agentes de escritura diferida de contraseñas 

   * Máquinas de puertas de enlace de protección de contraseñas  

   * Servidores de directivas de redes (NPS) que tienen la extensión RADIUS de Azure AD Multi-factor Authentication 

## <a name="next-steps"></a>Pasos siguientes
* [Creación de resistencia en la administración de identidad y acceso con Azure AD](resilience-overview.md)

* [Acceso externo seguro a los recursos](secure-external-access-resources.md) 
* [Integración de todas sus aplicaciones en Azure AD](five-steps-to-full-application-integration-with-azure-ad.md)