---
title: Funcionamiento de la sincronización en Azure AD Domain Services | Microsoft Docs
description: Obtenga información sobre cómo funciona el proceso de sincronización para objetos y credenciales de un inquilino de Azure AD o de un entorno local de Active Directory Domain Services en un dominio administrado de Azure Active Directory Domain Services.
services: active-directory-ds
author: justinha
manager: daveba
ms.assetid: 57cbf436-fc1d-4bab-b991-7d25b6e987ef
ms.service: active-directory
ms.subservice: domain-services
ms.workload: identity
ms.topic: conceptual
ms.date: 03/26/2021
ms.author: justinha
ms.openlocfilehash: 6b61bfbc1419b59d21e8db583b3a8aad5145494e
ms.sourcegitcommit: 7f59e3b79a12395d37d569c250285a15df7a1077
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/02/2021
ms.locfileid: "110785920"
---
# <a name="how-objects-and-credentials-are-synchronized-in-an-azure-active-directory-domain-services-managed-domain"></a>Procedimiento para sincronizar objetos y credenciales en un dominio administrado de Azure Active Directory Domain Services

Los objetos y las credenciales de un dominio administrado de Azure Active Directory Domain Services (Azure AD DS) pueden crearse localmente en el dominio o sincronizarse desde un inquilino de Azure Active Directory (Azure AD). La primera vez que implemente Azure AD DS, se configurará una sincronización unidireccional automática y comenzará a replicar los objetos de Azure AD. Esta sincronización unidireccional continúa ejecutándose en segundo plano para mantener actualizado el dominio administrado de Azure AD DS con los cambios de Azure AD. No se produce ninguna sincronización desde Azure AD DS hacia Azure AD de vuelta.

En un entorno híbrido, los objetos y las credenciales de un dominio de AD DS local se pueden sincronizar con Azure AD mediante Azure AD Connect. Una vez que los objetos se han sincronizado correctamente con Azure AD, la sincronización en segundo plano automática hace que esos objetos y credenciales estén disponibles para las aplicaciones que usan el dominio administrado.

Si las instancias de AD DS y Azure AD locales están configuradas para la autenticación federada mediante ADFS, entonces no hay ningún hash de contraseña (actual o válido) disponible en Azure DS. Las cuentas de usuario de Azure AD creadas antes de que se implementara la autenticación federada pueden tener un hash de contraseña antiguo, que es probable que no coincida con un hash de la contraseña local. De ahí que Azure AD DS no pueda validar las credenciales de los usuarios.

En el diagrama siguiente se muestra cómo funciona la sincronización entre Azure AD DS, Azure AD y un entorno de AD DS local opcional:

![Información general sobre sincronización en un dominio administrado de Azure AD Domain Services](./media/active-directory-domain-services-design-guide/sync-topology.png)

## <a name="synchronization-from-azure-ad-to-azure-ad-ds"></a>Sincronización de Azure AD a Azure AD DS

Las cuentas de usuario, las pertenencias a grupos y los hash de credenciales se sincronizan de manera unidireccional de Azure AD a Azure AD DS. Este proceso de sincronización es automático; No tiene que configurar, supervisar ni administrar este proceso de sincronización. La sincronización inicial puede tardar entre unas horas hasta un par de días, según el número de objetos del directorio de Azure AD. Una vez completada la sincronización inicial, los cambios que se realicen en Azure AD, como los cambios de contraseña o de atributo, se sincronizarán automáticamente en Azure AD DS.

Cuando se crea un usuario en Azure AD, no se sincroniza con Azure AD DS hasta que cambia su contraseña en Azure AD. Este proceso de cambio de contraseña hace que los valores hash de contraseña para la autenticación Kerberos y NTLM se generen y almacenen en Azure AD. Los hashes de contraseña son necesarios para autenticar correctamente a un usuario en Azure AD DS.

El proceso de sincronización es, por diseño, unidireccional. No hay ninguna sincronización inversa de los cambios de Azure AD DS de nuevo a Azure AD. Un dominio administrado es, en gran medida, de solo lectura, excepto para las unidades organizativas personalizadas que se creen. No se podrán realizar cambios en los atributos de usuario, las contraseñas de usuario o las pertenencias a grupos en un dominio administrado.

## <a name="attribute-synchronization-and-mapping-to-azure-ad-ds"></a>Sincronización de atributos y asignación a Azure AD DS

En la tabla siguiente se enumeran algunos atributos comunes y se describe cómo se sincronizan con Azure AD DS.

| Atributo en Azure AD DS | Source | Notas |
|:--- |:--- |:--- |
| UPN | Atributo *UPN* del usuario en el inquilino de Azure AD | El atributo UPN del inquilino de Azure AD se sincroniza tal cual con Azure AD DS. La forma más fiable de iniciar sesión consiste en usar el atributo UPN. |
| SAMAccountName | Atributo *mailNickname* del usuario en el inquilino de Azure AD o en uno generado automáticamente | El atributo *SAMAccountName* se obtiene del atributo *mailNickname* en el inquilino de Azure AD. Si existen varias cuentas de usuario que tienen el mismo atributo *mailNickname*, *SAMAccountName* se genera automáticamente. Si el prefijo de *UPN* o el atributo *mailNickname* del usuario tienen más de 20 caracteres, *SAMAccountName* se genera automáticamente para cumplir el límite de 20 caracteres de los atributos *SAMAccountName*. |
| Contraseñas | Contraseña del usuario del inquilino de Azure AD | Los hash de contraseñas heredados necesarios para la autenticación NTLM o Kerberos se sincronizan desde el inquilino de Azure AD. Si el inquilino de Azure AD está configurado para la sincronización híbrida mediante Azure AD Connect, estos hash de contraseña se obtienen del entorno de AD DS local. |
| SID de los grupos y usuarios primarios | Generado automáticamente | El SID primario de las cuentas de usuarios o grupos se genera automáticamente en Azure AD DS. Este atributo no coincide con el SID principal de los grupos o usuarios del objeto de un entorno de AD DS local. Esto se debe a que el dominio administrado tiene un espacio de nombres de SID diferente al del dominio de AD DS local. |
| Historial de SID de usuarios y grupos | SID de usuarios y grupos primarios locales | El atributo *SidHistory* de usuarios y grupos en Azure AD DS se establece para que coincida con el SID principal de grupos o usuarios correspondiente de un entorno de AD DS. Gracias a esta característica, le resultará más sencillo migrar mediante lift-and-shift aplicaciones locales a Azure AD DS, ya que no tendrá que volver a incluir los recursos en listas ACL. |

> [!TIP]
> **Inicie sesión en el dominio administrado con el formato de UPN**: el atributo *SAMAccountName*, como `AADDSCONTOSO\driley`, puede generarse automáticamente en algunas cuentas de usuario de un dominio administrado. Los atributos *SAMAccountName* de los usuarios generados automáticamente pueden ser distintos de sus prefijos de UPN, por lo que no es siempre una manera fiable de iniciar sesión.
>
> Por ejemplo, si varios usuarios tienen el mismo atributo *mailNickname* o prefijos UPN excesivamente largos, el atributo *SAMAccountName* para estos usuarios puede generarse automáticamente. Use el formato UPN, como `driley@aaddscontoso.com`, para iniciar sesión de forma fiable en un dominio administrado.

### <a name="attribute-mapping-for-user-accounts"></a>Asignación de atributos para cuentas de usuario

En la tabla siguiente se muestra cómo determinados atributos de objetos de usuario en Azure AD se sincronizan con los atributos correspondientes de Azure AD DS.

| Atributo de usuario en Azure AD | Atributo de usuario en Azure AD DS |
|:--- |:--- |
| accountEnabled |userAccountControl (establece o borra el bit ACCOUNT_DISABLED) |
| city |l |
| companyName |companyName |
| country |co |
| department |department |
| DisplayName |DisplayName |
| employeeId |employeeId |
| facsimileTelephoneNumber |facsimileTelephoneNumber |
| givenName |givenName |
| jobTitle |title |
| mail |mail |
| mailNickname |msDS-AzureADMailNickname |
| mailNickname |SAMAccountName (a veces, puede generarse automáticamente) |
| manager |manager |
| mobile |mobile |
| objectid |msDS-AzureADObjectId |
| onPremiseSecurityIdentifier |sidHistory |
| passwordPolicies |userAccountControl (establece o borra el bit DONT_EXPIRE_PASSWORD) |
| physicalDeliveryOfficeName |physicalDeliveryOfficeName |
| postalCode |postalCode |
| preferredLanguage |preferredLanguage |
| proxyAddresses | proxyAddresses |
| state |st |
| streetAddress |streetAddress |
| surname |sn |
| telephoneNumber |telephoneNumber |
| userPrincipalName |userPrincipalName |

### <a name="attribute-mapping-for-groups"></a>Asignación de atributos de grupos

En la tabla siguiente se muestra cómo determinados atributos de objetos de grupo en Azure AD se sincronizan con los atributos correspondientes de Azure AD DS.

| Atributo de grupo en Azure AD | Atributo de grupo en Azure AD DS |
|:--- |:--- |
| DisplayName |DisplayName |
| DisplayName |SAMAccountName (a veces, puede generarse automáticamente) |
| mail |mail |
| mailNickname |msDS-AzureADMailNickname |
| objectid |msDS-AzureADObjectId |
| onPremiseSecurityIdentifier |sidHistory |
| proxyAddresses | proxyAddresses |
| securityEnabled |groupType |

## <a name="synchronization-from-on-premises-ad-ds-to-azure-ad-and-azure-ad-ds"></a>Sincronización de un entorno AD DS local a Azure AD y Azure AD DS

Azure AD Connect se utiliza para sincronizar cuentas de usuario, pertenencias a grupos y hash de credenciales de un entorno de AD DS local a Azure AD. Los atributos de cuentas de usuario, como el UPN y el identificador de seguridad local (SID), se sincronizan. Para iniciar sesión con Azure AD DS, los hash de contraseña heredados necesarios para la autenticación NTLM y Kerberos también se sincronizan con Azure AD.

> [!IMPORTANT]
> Azure AD Connect solo debe instalarse y configurarse para la sincronización con entornos de AD DS locales. No se admite la instalación de Azure AD Connect en un dominio administrado para volver a sincronizar los objetos con Azure AD.

Si configura la reescritura, los cambios de Azure AD se vuelven a sincronizar con el entorno de AD DS local. Por ejemplo, si un usuario cambia la contraseña mediante la administración de autoservicio de contraseña de Azure AD, la contraseña se volverá a actualizar en el entorno de AD DS local.

> [!NOTE]
> Utilice siempre la versión más reciente de Azure AD Connect para asegurarse de contar con correcciones para todos los errores conocidos.

### <a name="synchronization-from-a-multi-forest-on-premises-environment"></a>Sincronización en un entorno local de varios bosques

Muchas organizaciones tienen un entorno de AD DS local bastante complejo que incluye varios bosques. Azure AD Connect admite la sincronización de usuarios, grupos y hash de credenciales de entornos de varios bosques con Azure AD.

Azure AD tiene un espacio de nombres mucho más sencillo y plano. Si quiere que los usuarios puedan acceder de manera confiable a las aplicaciones protegidas mediante Azure AD, resuelva todos los conflictos de UPN que haya en las cuentas de usuario de los todos los bosques. Los dominios administrados usan una estructura plana de unidad organizativa, similar a Azure AD. Las cuentas de usuario y los grupos se almacenan en el contenedor *Usuarios de AADDC*, aunque se sincronicen desde dominios o bosques locales distintos, incluso si se ha configurado una estructura jerárquica de unidad organizativa local. El dominio administrado simplifica las estructuras jerárquicas de unidades organizativas.

Como se ha detallado anteriormente, no hay ninguna sincronización de Azure AD DS de nuevo a Azure AD. Puede [crear una unidad organizativa (OU) personalizada](create-ou.md) en Azure AD DS y, después, usuarios, grupos o cuentas de servicio en esas unidades organizativas personalizadas. Ninguno de los objetos creados en las unidades organizativas personalizadas se vuelven a sincronizar con Azure AD. Estos objetos solo están disponibles en el dominio administrado y no se pueden ver mediante cmdlets de PowerShell de Azure AD, Microsoft Graph API ni la interfaz de usuario de administración de Azure AD.

## <a name="what-isnt-synchronized-to-azure-ad-ds"></a>Objetos o atributos que no están sincronizados con Azure AD DS

Los siguientes objetos o atributos no se sincronizan desde un entorno AD DS local a Azure AD o Azure AD DS:

* **Atributos excluidos:** gracias a Azure AD Connect, puede elegir qué atributos no se sincronizarán con Azure AD a partir de un entorno de AD DS local. Estos atributos excluidos no están disponibles en Azure AD DS.
* **Directivas de grupo:** las directivas de grupo configuradas en un entorno de AD DS local no se sincronizan con Azure AD DS.
* **Carpeta Sysvol:** el contenido de la carpeta *Sysvol* de un entorno de AD DS local no se sincroniza con Azure AD DS.
* **Objetos de equipo:** Los objetos de equipo de equipos unidos a un entorno de AD DS local no se sincronizan con Azure AD DS. Estos equipos no tienen una relación de confianza con el dominio administrado y solo pertenecen al entorno de AD DS local. En Azure AD DS, solo se muestran objetos de equipo que se hayan unido expresamente al dominio administrado.
* **Atributos SidHistory de usuarios y grupos:** los SID de los grupos y usuarios principales de un entorno de AD DS local se sincronizan con Azure AD DS. Pero los atributos *SidHistory* existentes de usuarios y grupos no se sincronizan desde el entorno de AD DS local a Azure AD DS.
* **Estructuras de unidades organizativas:** las unidades organizativas definidas en un entorno de AD DS local no se sincronizan con Azure AD DS. Hay dos unidades organizativas integradas en Azure AD DS; una para los usuarios y otra para los equipos. El dominio administrado tiene una estructura plana de unidad organizativa. Puede optar por [crear una unidad organizativa personalizada en el dominio administrado](create-ou.md).

## <a name="password-hash-synchronization-and-security-considerations"></a>Consideraciones de seguridad y sincronización de hash de contraseña

Al habilitar Azure AD DS, se necesitan hash de contraseña heredados para la autenticación NTLM y Kerberos. Azure AD no almacena contraseñas no cifradas, por lo que estos hash no se pueden generar automáticamente para las cuentas de usuario existentes. Una vez generados y almacenados, los hash de contraseña en los formatos compatibles NTLM y Kerberos siempre se almacenarán de forma cifrada en Azure AD.

Las claves de cifrado serán únicas para cada inquilino de Azure AD. Estos hash se cifrarán de forma que solo Azure AD DS tenga acceso a las claves de descifrado. Ningún otro servicio o componente de Azure AD tendrá acceso a ellas.

Los hash de contraseña heredados se sincronizan desde Azure AD en los controladores de dominio de un dominio administrado. Los discos de estos controladores de dominio administrados en Azure AD DS se cifrarán en reposo. Estos hash de contraseña se almacenarán y protegerán en dichos controladores de dominio de modo similar a cómo se almacenan y protegen las contraseñas en un entorno de AD DS local.

En el caso de los entornos de Azure AD solo de nube, los [usuarios deben restablecer o cambiar su contraseña](tutorial-create-instance.md#enable-user-accounts-for-azure-ad-ds) para que los hash de contraseña se puedan generar y almacenar en Azure AD. En el caso de cualquier cuenta de usuario en la nube creada en Azure AD tras habilitar Azure AD Domain Services, los hash de contraseña se generarán y almacenarán en los formatos compatibles NTLM y Kerberos. Todas las cuentas de usuario en la nube deben cambiar la contraseña antes de sincronizarse con Azure AD DS.

En el caso de las cuentas de usuario híbridas sincronizadas desde el entorno de AD DS local mediante Azure AD Connect, deberá [configurar Azure AD Connect para que sincronice los hash de contraseña en los formatos compatibles NTLM y Kerberos](tutorial-configure-password-hash-sync.md).

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información sobre los detalles de la sincronización de contraseña, consulte [Funcionamiento de la sincronización de hash de contraseña con Azure AD Connect](../active-directory/hybrid/how-to-connect-password-hash-synchronization.md?context=/azure/active-directory-domain-services/context/azure-ad-ds-context).

Para empezar a usar Azure AD DS, [cree un dominio administrado](tutorial-create-instance.md).
