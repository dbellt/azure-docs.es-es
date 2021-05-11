---
title: Control de cuentas de servicio locales | Azure Active Directory
description: Use esta guía para crear y ejecutar un proceso de ciclo de vida para las cuentas de servicio.
services: active-directory
author: BarbaraSelden
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.subservice: fundamentals
ms.topic: conceptual
ms.date: 2/15/2021
ms.author: baselden
ms.reviewer: ajburnle
ms.custom: it-pro, seodec18
ms.collection: M365-identity-device-management
ms.openlocfilehash: 50ba13a49d8e08b70ebf8a8bb12dfe92d8a35bb7
ms.sourcegitcommit: a5dd9799fa93c175b4644c9fe1509e9f97506cc6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108206591"
---
# <a name="govern-on-premises-service-accounts"></a>Control de las cuentas de servicio locales

Active Directory ofrece cuatro tipos de cuentas de servicio locales:

* [Cuentas de servicio administradas de grupo (gMSA)](service-accounts-group-managed.md)  
* [Cuentas de servicio administradas independientes (sMSA)](service-accounts-standalone-managed.md)  
* [Cuentas de equipo](service-accounts-computer.md)  
* [Cuentas de usuario que funcionan como cuentas de servicio](service-accounts-user-on-premises.md)


Es fundamental controlar de cerca las cuentas de servicio de forma que pueda: 

* Protegerlas en función de los requisitos y propósito de sus casos de uso.  
* Administrar el ciclo de vida de las cuentas y sus credenciales.  
* Evaluar las cuentas de servicio según el riesgo al que se expondrán y los permisos que tengan.  
* Asegúrese de que Active Directory y Azure Active Directory no tienen ninguna cuenta de servicio obsoleta con permisos amplios.

## <a name="principles-for-creating-a-new-service-account"></a>Principios para crear una nueva cuenta de servicio

Al crear una cuenta de servicio, debe tener en cuenta las consideraciones enumeradas en la tabla siguiente:

| Principio| Consideración | 
| - |- | 
| Asignación de cuentas de servicio| Vincule la cuenta de servicio a un único servicio, aplicación o script. |
| Propiedad| Asegúrese de que existe un propietario que solicita la cuenta y asume su responsabilidad. |
| Ámbito| Defina el ámbito claramente y anticipe la duración del uso de la cuenta de servicio. |
| Propósito| Cree cuentas de servicio con un fin único específico. |
| Permisos | Aplique el principio del *permiso mínimo*. Para ello:<li>Nunca asigne permisos a grupos integrados, como los administradores.<li>Quite los permisos de máquina local, cuando sea conveniente.<li>Personalice el acceso y use la delegación de Active Directory para el acceso al directorio.<li>Use permisos de acceso pormenorizados.<li>Establezca expiraciones de cuenta y restricciones basadas en la ubicación en las cuentas de servicio basadas en el usuario. |
| Supervisión y uso para auditorías| Supervise los datos de inicio de sesión y asegúrese de que coinciden con el uso previsto. Establezca alertas para usos anómalos. |
| | |

### <a name="set-restrictions-for-user-accounts"></a>Establecimiento de restricciones para las cuentas de usuario

En el caso de las cuentas de usuario que se usan como cuentas de servicio, aplique la siguiente configuración:

* [**Expiración de la cuenta**](/powershell/module/activedirectory/set-adaccountexpiration?view=winserver2012-ps&preserve-view=true): configure la cuenta de servicio para que expire automáticamente a una hora fijada después del período de revisión, a menos que se determine que debe continuar.

*  **LogonWorkstations**: restrinja los permisos de donde la cuenta de servicio puede iniciar sesión. Si se ejecuta localmente en una máquina y solo tiene acceso a los recursos de esa máquina, restrinja el inicio de sesión en cualquier otra parte.

* [**No puede cambiar la contraseña**](/powershell/module/addsadministration/set-aduser): impida que la cuenta de servicio cambie su propia contraseña; para ello, establezca el parámetro en false.
 
## <a name="build-a-lifecycle-management-process"></a>Creación de un proceso de administración del ciclo de vida

Para mantener la seguridad de las cuentas de servicio, debe administrarlas desde el momento en que identifica que son necesarias y hasta que se retiran. 

Utilice el siguiente proceso para la administración del ciclo de vida de las cuentas de servicio:

1. Recopile información de uso de la cuenta.
1. Mueva la cuenta de servicio y la aplicación a la base de datos de administración de configuración (CMDB).
1. Realice una evaluación de riesgos o una revisión formal.
1. Cree la cuenta de servicio y aplique restricciones.
1. Programe y realice revisiones periódicas. Ajuste los permisos y los ámbitos según sea necesario.
1. Desaprovisione la cuenta cuando sea conveniente.

### <a name="collect-usage-information-for-the-service-account"></a>Recopilación de información de uso de la cuenta de servicio

Recopile información empresarial de interés para cada cuenta de servicio. En la tabla siguiente se muestra la cantidad mínima de información que se va a recopilar, sin embargo, debe recopilar todo lo necesario para crear el caso empresarial de cada cuenta existente.

| data| Descripción |
| - | - |
| Propietario| Usuario o grupo responsable de la cuenta de servicio |
| Propósito| Propósito de la cuenta de servicio |
| Permisos (ámbitos)| Conjunto previsto de permisos |
| Vínculos de CMDB| Cuenta de servicio entre vínculos con el script, o la aplicación, y los propietarios de destino |
| Riesgo| Puntuación de impacto empresarial y riesgo en función de la evaluación de riesgos de seguridad |
| Período de duración| Duración máxima anticipada para permitir la programación de la expiración o la recertificación de la cuenta |
| | |

Lo ideal es que cree la solicitud de autoservicio de la cuenta y solicite la información pertinente. El propietario puede ser el propietario de una empresa o aplicación, un miembro de TI o el propietario de una infraestructura. El uso de una herramienta, como Microsoft Forms con esta solicitud y la información asociada, facilitarán la migración a la herramienta de inventario de CMDB si la cuenta está aprobada.

### <a name="onboard-service-account-to-cmdb"></a>Incorporación de la cuenta de servicio a CMDB

Almacene la información recopilada en una aplicación de tipo CMDB. Además de la información empresarial, incluya todas las dependencias de otras infraestructuras, aplicaciones y procesos.  Este repositorio central le facilitará llevar a cabo lo siguiente:

* Evalúe el riesgo.  
* Configurar la cuenta de servicio con las restricciones necesarias.  
* Comprender las dependencias funcionales y de seguridad pertinentes.  
* Realice revisiones periódicas de seguridad y necesidades continuadas.  
* Ponerse en contacto con los propietarios para revisar, retirar y cambiar la cuenta de servicio.

Considere una cuenta de servicio que se usa para ejecutar un sitio web y que tiene permisos para conectarse a una o varias bases de datos SQL de Recursos Humanos (RR. HH.). La información almacenada en la CMDB para la cuenta de servicio, incluidas las descripciones de ejemplo, se muestra en la tabla siguiente:

|data | Descripción de ejemplo|
| - | - |
| Propietario, Suplente| John Bloom, Anna Mayers |
| Propósito| Ejecute la página web de RR. HH. y conéctese a sus bases de datos. Puede suplantar a los usuarios finales al acceder a las bases de datos. |
| Permisos, ámbitos| HR-WEBServer: iniciar sesión localmente; ejecutar página web<br>HR-SQL1: iniciar sesión localmente; permisos de lectura en todas las bases de datos de RR. HH.<br>HR-SQL2: iniciar sesión localmente; permisos de lectura solo en la base de datos Salary |
| Cost Center| 883944 |
| Riesgo evaluado| Medio; impacto empresarial: medio; información privada; medio |
| Restricciones de cuenta| Inicie sesión en: solo los servidores mencionados anteriormente; no se puede cambiar la contraseña; directiva de contraseñas MBI; |
| Período de duración| Sin restricciones |
| Ciclo de revisión| Semestral (por propietario, por equipo de seguridad, por privacidad) |
| | |

### <a name="perform-a-risk-assessment-or-formal-review-of-service-account-usage"></a>Realización de una evaluación de riesgos o una revisión formal del uso de la cuenta de servicio

Supongamos que la cuenta está en peligro debido a un origen no autorizado. Evalúe los riesgos que podría suponer la cuenta para su aplicación o servicio asociados y para su infraestructura. Considere el riesgo directo e indirecto. 

* ¿A qué podría obtener acceso directo un usuario no autorizado?  
* ¿A qué otra información o sistema puede acceder la cuenta de servicio?  
* ¿Se puede usar la cuenta para conceder permisos adicionales?  
* ¿Cómo sabrá cuándo cambian los permisos?

Después de haber realizado y documentado la evaluación de riesgos, es posible que encuentre que los riesgos afectan a:

* Las restricciones de cuenta  
* La duración de la cuenta  
* Los requisitos de revisión de cuentas (cadencia y revisores)

### <a name="create-a-service-account-and-apply-account-restrictions"></a>Creación de una cuenta de servicio y aplicación de restricciones de cuenta

Cree una cuenta de servicio solo después de haber completado la evaluación de riesgos y haber documentado la información pertinente en su CMDB. Alinee las restricciones de la cuenta con la evaluación de riesgos. Tenga en cuenta las siguientes restricciones cuando sean pertinentes para la evaluación:

* Para todas las cuentas de usuario usadas como cuentas de servicio, defina una fecha de finalización realista y precisa. Establezca la fecha mediante la marca **La cuenta expira**. Para más información, consulte [Set-ADAccountExpiration](/powershell/module/activedirectory/set-adaccountexpiration). 

* Inicie sesión en [LogonWorkstation](/powershell/module/activedirectory/set-aduser).

* Requisitos de [directiva de contraseña](../../active-directory-domain-services/password-policy.md).

* Creación de cuentas en una [ubicación de unidad organizativa](/windows-server/identity/ad-ds/plan/delegating-administration-of-account-ous-and-resource-ous) que garantiza la administración solo para los usuarios permitidos.

* Configuración y recopilación de eventos de auditoría [que detecten cambios](/windows/security/threat-protection/auditing/audit-directory-service-changes) en la cuenta de servicio y en el [uso de la cuenta de servicio](https://www.manageengine.com/products/active-directory-audit/how-to/audit-kerberos-authentication-events.html).

Cuando esté listo para poner la cuenta de servicio en producción, conceda acceso a ella de forma más segura. 

### <a name="schedule-regular-reviews-of-service-accounts"></a>Programación de revisiones periódicas de cuentas de servicio

Configure revisiones periódicas de las cuentas de servicio clasificadas como de riesgo medio y alto. Las revisiones deben incluir: 

* Atestación del propietario de la necesidad continuada de la cuenta y justificación de los privilegios y los ámbitos.

* Revisión por privacidad y equipos de seguridad, incluida la evaluación de las conexiones ascendentes y descendentes.

* Datos de las auditorías, que garantizan su uso exclusivo para los fines previstos.

### <a name="deprovision-service-accounts"></a>Desaprovisionamiento de cuentas de servicio

En el proceso de desaprovisionamiento, quite primero los permisos y la supervisión y, después, quite la cuenta si es necesario.

Desaprovisione las cuentas de servicio en los casos siguientes:

* El script o la aplicación para los que se creó la cuenta de servicio se han retirado.

* La función del script o la aplicación para la que se usa la cuenta de servicio (por ejemplo, para acceder a un recurso específico) se ha retirado.

* La cuenta de servicio se ha reemplazado por una cuenta de servicio diferente.

Después de quitar todos los permisos, quite la cuenta de la manera siguiente:

1. Después de desaprovisionar la aplicación o el script asociados, supervise los inicios de sesión y el acceso a los recursos de las cuentas de servicio asociadas para asegurarse de que no se usan en otro proceso. Si está seguro de que ya no es necesario, vaya al paso siguiente.

1. Deshabilite la cuenta de servicio para evitar el inicio de sesión y asegúrese de que ya no sea necesaria. Cree una directiva empresarial para cuando las cuentas deban permanecer deshabilitadas.

1. Elimine la cuenta de servicio cuando se cumpla la directiva para mantenerla deshabilitada. 

   * **En las cuentas MSA**: [desinstale la cuenta](/powershell/module/activedirectory/uninstall-adserviceaccount?view=winserver2012-ps&preserve-view=true) mediante PowerShell o elimínela manualmente desde el contenedor de la cuenta de servicio administrada.

   * **En las cuentas de equipo o usuario**: elimine manualmente la cuenta desde Active Directory.

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre la protección de las cuentas de servicio, consulte los artículos siguientes:

* [Introducción a las cuentas de servicio locales](service-accounts-on-premises.md)  
* [Protección de cuentas de servicio administradas de grupo](service-accounts-group-managed.md)  
* [Protección de cuentas de servicio administradas independientes](service-accounts-standalone-managed.md)  
* [Protección de cuentas de equipo](service-accounts-computer.md)  
* [Protección de cuentas de usuario](service-accounts-user-on-premises.md)
