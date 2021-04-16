---
title: Control de cuentas de servicio locales | Azure Active Directory
description: Guía para crear y ejecutar un proceso de ciclo de vida de cuenta para cuentas de servicio
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
ms.openlocfilehash: 10ea524620f810e0bf1dddc230716031bbc10e69
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/30/2021
ms.locfileid: "105642386"
---
# <a name="governing-on-premises-service-accounts"></a>Control de cuentas de servicio locales

Hay cuatro tipos de cuentas de servicio locales en Windows Active Directory:

* [Cuentas de servicio administradas de grupo](service-accounts-group-managed.md) (gMSA)

* [Cuentas de servicio administradas independientes](service-accounts-standalone-managed.md) (sMSA)

* [Cuentas de equipo](service-accounts-computer.md)

* [Cuentas de usuario que funcionan como cuentas de servicio](service-accounts-user-on-premises.md)


Es fundamental que las cuentas de servicio se rijan estrictamente por: 

* Proteja las cuentas de servicio en función de los requisitos y propósitos de sus casos de uso.

* Administre el ciclo de vida de las cuentas de servicio y sus credenciales.

* Evalúe las cuentas de servicio en función del riesgo al que se expondrán y de los permisos que tengan. 

* Asegúrese de que Active Directory y Azure Active Directory no tienen ninguna cuenta de servicio obsoleta con permisos amplios.

## <a name="principles-for-creating-a-new-service-account"></a>Principios para crear una nueva cuenta de servicio

Utilice los siguientes criterios para crear una nueva cuenta de servicio.

| Principios| Consideraciones | 
| - |- | 
| Asignación de cuentas de servicio| Vincule la cuenta de servicio a un único servicio, aplicación o script. |
| Propiedad| Asegúrese de que existe un propietario que solicita la cuenta y asume su responsabilidad. |
| Ámbito| Defina el ámbito claramente y anticipe la duración del uso de la cuenta de servicio. |
| Propósito| Cree cuentas de servicio para un único propósito específico. |
| Privilegio| Aplique el principio de privilegios mínimos siguiendo estas instrucciones: <br>Nunca los asigne a grupos integrados como administradores.<br> Quite los privilegios de la máquina local cuando corresponda.<br>Personalice el acceso y use la delegación de Active Directory delegación para el acceso al directorio.<br>Use permisos de acceso pormenorizados.<br>Establezca expiraciones de cuenta y restricciones basadas en la ubicación en las cuentas de servicio basadas en el usuario. |
| Supervisión y uso para auditorías| Supervise los datos de inicio de sesión y asegúrese de que coinciden con el uso previsto. Establezca alertas para usos anómalos. |

### <a name="enforce-least-privilege-for-user-accounts-and-limit-account-overuse"></a>Aplicación de privilegios mínimos para las cuentas de usuario y limitación del uso excesivo de las cuentas

Use la siguiente configuración con las cuentas de usuario empleadas como cuentas de servicio:

* [**Expiración de la cuenta**](/powershell/module/activedirectory/set-adaccountexpiration?view=winserver2012-ps&preserve-view=true): configure la cuenta de servicio para que expire automáticamente a una hora establecida después del período de revisión, a menos que se determine que debe continuar.

*  **LogonWorkstations**: restrinja los permisos de ubicación donde la cuenta de servicio puede iniciar sesión. Si se ejecuta localmente en una máquina y solo tiene acceso a los recursos de esa máquina, restrinja el inicio de sesión en cualquier otra parte.

* [**No se puede cambiar la contraseña**](/powershell/module/addsadministration/set-aduser): impida que la cuenta de servicio cambie su propia contraseña; para ello, establezca el parámetro en false.

 
## <a name="build-a-lifecycle-management-process"></a>Creación de un proceso de administración del ciclo de vida

Para mantener la seguridad de las cuentas de servicio, debe administrarlas desde el momento en que identifica la necesidad y hasta que se retiran. 

Utilice el siguiente proceso para la administración del ciclo de vida de las cuentas de servicio:

1. Recopile información de uso de la cuenta.
1. Incorpore la cuenta de servicio y la aplicación a la base de datos de administración de configuración (CMDB).
1. Realice una evaluación de riesgos o una revisión formal.
1. Cree la cuenta de servicio y aplique restricciones.
1. Programe y realice revisiones periódicas. Ajuste los permisos y los ámbitos según sea necesario.
1. Desaprovisione la cuenta cuando corresponda.

### <a name="collect-usage-information-for-the-service-account"></a>Recopilación de información de uso de la cuenta de servicio

Recopile la información empresarial pertinente de cada cuenta de servicio. En la tabla siguiente se muestra la información mínima que se debe recopilar, aunque recuerde que debe recopilar todo lo necesario para el caso empresarial de la existencia de las cuentas.

| data| Detalles |
| - | - |
| Propietario| Usuario o grupo responsable de la cuenta de servicio |
| Propósito| Propósito de la cuenta de servicio |
| Permisos (ámbitos)| Conjunto previsto de permisos |
| Vínculos de la base de datos de administración de configuración (CMDB)| Cuenta de servicio entre vínculos con el script o la aplicación y los propietarios de destino |
| Riesgo| Puntuación de impacto empresarial y riesgo en función de la evaluación de riesgos de seguridad |
| Período de duración| Vigencia máxima anticipada para habilitar la programación de la expiración o la recertificación de la cuenta |


 

Lo ideal es que realice la solicitud de autoservicio de la cuenta y requiera la información pertinente. Propietario, que puede ser el propietario de una empresa o aplicación, un miembro de TI o el propietario de una infraestructura. El uso de una herramienta como Microsoft Forms para esta solicitud y la información asociada facilitarán la migración a la herramienta de inventario de CMDB si la cuenta está aprobada.

### <a name="onboard-service-account-to-cmdb"></a>Incorporación de la cuenta de servicio a CMDB

Almacene la información recopilada en una aplicación de tipo CMDB. Además de la información empresarial, incluya todas las dependencias en otras infraestructuras, aplicaciones y procesos.  Este repositorio central le facilitará lo siguiente:

* Evalúe el riesgo.

* Configure la cuenta de servicio con las restricciones necesarias.

* Comprenda las dependencias funcionales y de seguridad pertinentes.

* Realice revisiones periódicas de seguridad y necesidades continuadas.

* Póngase en contacto con los propietarios para revisar, retirar y cambiar la cuenta de servicio.

Considere una cuenta de servicio que se use para ejecutar un sitio web y que tenga privilegios para conectarse a una o varias bases de datos SQL. La información almacenada en su CMDB para esta cuenta de servicio podría ser:

|data | Detalles|
| - | - |
| Propietario, Suplente| John Bloom, Anna Mayers |
| Propósito| Ejecute la página web de recursos humanos y conéctese a sus bases de datos. Puede suplantar al usuario final al obtener acceso a las bases de datos. |
| Permisos, ámbitos| HR-WEBServer: iniciar sesión localmente, ejecutar página web<br>HR-SQL1: iniciar sesión localmente, lectura en toda la base de datos HR*<br>HR-SQL2: iniciar sesión localmente, lectura en la base de datos SALARY* |
| Cost Center| 883944 |
| Riesgo evaluado| Medio; impacto empresarial: medio; información privada; medio |
| Restricciones de cuenta| Inicie sesión en: solo los servidores mencionados anteriormente; no se puede cambiar la contraseña; directiva de contraseñas MBI; |
| Período de duración| Sin restricciones |
| Ciclo de revisión| Semestral (por propietario, por equipo de seguridad, por privacidad) |

### <a name="perform-risk-assessment-or-formal-review-of-service-account-usage"></a>Realización de una evaluación de riesgos o una revisión formal del uso de la cuenta de servicio

Teniendo en cuenta los permisos y el propósito, evalúe el riesgo que la cuenta puede suponer para su aplicación o servicio asociado y para su infraestructura si se pone en peligro. Considere el riesgo directo e indirecto. 

* ¿A qué podría obtener acceso directo un adversario?

* ¿A qué otra información o sistema puede acceder la cuenta de servicio?

* ¿Se puede usar la cuenta para conceder permisos adicionales?

* ¿Cómo sabrá cuándo cambian los permisos?

La evaluación de riesgos, una vez realizada y documentada, puede afectar a los elementos siguientes:

* Restricciones de cuenta

* Vigencia de la cuenta

* Requisitos de revisión de cuentas (cadencia y revisores)

### <a name="create-a-service-account-and-apply-account-restrictions"></a>Creación de una cuenta de servicio y aplicación de restricciones de cuenta

Cree una cuenta de servicio solo después de documentar la información pertinente en su CMDB y de realizar una evaluación de riesgos. Las restricciones de cuenta deben estar alineadas con la evaluación de riesgos. Tenga en cuenta las siguientes restricciones cuando sea pertinente para su evaluación:

* [Expiración de la cuenta](/powershell/module/activedirectory/set-adaccountexpiration?view=winserver2012-ps&preserve-view=true)

   * Para todas las cuentas de usuario usadas como cuentas de servicio, defina una fecha de finalización realista y precisa para su uso. Establezca esta opción mediante la marca "La cuenta expira". Para obtener más información, consulte [ Set-ADAccountExpiration](/powershell/module/addsadministration/set-adaccountexpiration). 

* Iniciar sesión en ([LogonWorkstation](/powershell/module/addsadministration/set-aduser))

* Requisitos de [directiva de contraseñas](../../active-directory-domain-services/password-policy.md)

* Creación en una [Ubicación de unidad organizativa](/windows-server/identity/ad-ds/plan/delegating-administration-of-account-ous-and-resource-ous) que garantice la administración solo para los usuarios con privilegios

* Configuración y recopilación de auditoría [que detecte los cambios](/windows/security/threat-protection/auditing/audit-directory-service-changes) en la cuenta de servicio (y en el [uso de la cuenta de servicio](https://www.manageengine.com/products/active-directory-audit/how-to/audit-kerberos-authentication-events.html)).

Cuando esté a punto para pasar a producción, conceda acceso a la cuenta de servicio de forma segura. 

### <a name="schedule-regular-reviews-of-service-accounts"></a>Programación de revisiones periódicas de cuentas de servicio

Configure revisiones periódicas de las cuentas de servicio clasificadas como de riesgo medio y alto. Las revisiones deben incluir: 

* Atestación del propietario en la necesidad continuada de la cuenta y justificación de los privilegios y los ámbitos.

* Revise los equipos de privacidad y seguridad, incluida la evaluación de las conexiones ascendentes y descendentes.

* Los datos de las auditorías garantizan su uso exclusivo para los fines previstos.

### <a name="deprovision-service-accounts"></a>Desaprovisionamiento de cuentas de servicio

En el proceso de desaprovisionamiento, quite primero los permisos y el monitor y, después, quite la cuenta si es necesario.

Desaprovisione las cuentas de servicio en los casos siguientes:

* El script o la aplicación para los que se creó la cuenta de servicio se ha retirado.

* La función del script o la aplicación para el que se usa la cuenta de servicio (por ejemplo, para acceder a un recurso específico) se ha retirado.

* La cuenta de servicio se ha reemplazado por una cuenta de servicio diferente.

Después de quitar todos los permisos, use este proceso para quitar la cuenta.

1. Después de desaprovisionar la aplicación o el script asociado, supervise los inicios de sesión y el acceso a los recursos de las cuentas de servicio asociadas para asegurarse de que no se usan en otro proceso. Si está seguro de que ya no es necesario, vaya al paso siguiente.

2. Deshabilite la cuenta de servicio para el inicio de sesión y asegúrese de que ya no se necesita. Crear una directiva empresarial para el tiempo en que las cuentas deben permanecer deshabilitadas.

3. Elimine la cuenta de servicio cuando se cumpla la directiva de mantenimiento de cuenta deshabilitada. 

   * Para las cuentas MSA, puede [desinstalarla](/powershell/module/activedirectory/uninstall-adserviceaccount?view=winserver2012-ps&preserve-view=true) mediante PowerShell o eliminarla manualmente desde el contenedor de la cuenta de servicio administrada.

   * En el caso de las cuentas de equipo o de usuario, puede eliminar manualmente la cuenta desde Active Directory.

## <a name="next-steps"></a>Pasos siguientes
Consulte los siguientes artículos sobre la protección de cuentas de servicio.

* [Introducción a las cuentas de servicio locales](service-accounts-on-premises.md)

* [Protección de cuentas de servicio administradas de grupo](service-accounts-group-managed.md)

* [Protección de cuentas de servicio administradas independientes](service-accounts-standalone-managed.md)

* [Protección de cuentas de equipo](service-accounts-computer.md)

* [Protección de cuentas de usuario](service-accounts-user-on-premises.md)

* [Control de cuentas de servicio locales](service-accounts-govern-on-premises.md)