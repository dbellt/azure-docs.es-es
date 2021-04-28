---
title: Introducción a la seguridad de Azure Key Vault
description: Información general sobre las características de seguridad y los procedimientos recomendados en Azure Key Vault.
services: key-vault
author: msmbaldwin
tags: azure-resource-manager
ms.service: key-vault
ms.subservice: general
ms.topic: conceptual
ms.date: 04/15/2021
ms.author: mbaldwin
ms.openlocfilehash: fe88933049ad39de57f879789e8c1b86ed7a54f5
ms.sourcegitcommit: 6686a3d8d8b7c8a582d6c40b60232a33798067be
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107753279"
---
# <a name="azure-key-vault-security"></a>Seguridad de Azure Key Vault

Azure Key Vault protege las claves de cifrado y los secretos, como certificados, cadenas de conexión y contraseñas, en la nube. No obstante, cuando almacena datos confidenciales y críticos para la empresa, debe tomar medidas para maximizar la seguridad de los almacenes y de los datos almacenados en estos.

En este artículo se proporciona información general sobre las características de seguridad y los procedimientos recomendados en Azure Key Vault.

> [!NOTE]
> Para obtener una lista completa de las recomendaciones de seguridad de Azure Key Vault, consulte la [línea de base de seguridad de Azure Key Vault](security-baseline.md).

## <a name="network-security"></a>Seguridad de red

Puede reducir la exposición de los almacenes si especifica las direcciones IP que tienen acceso a ellos. Los puntos de conexión de servicio de red virtual para Azure Key Vault permiten restringir el acceso a una red virtual especificada. También permiten restringir el acceso a una lista de intervalos de direcciones IPv4 (protocolo de Internet, versión 4). A todos los usuarios que se conecten a su almacén de claves desde fuera de esos orígenes se les negará el acceso.  Para conocer los detalles completos, consulte [Puntos de conexión de servicio de red virtual en Azure Key Vault](overview-vnet-service-endpoints.md).

Una vez que las reglas del firewall están en vigor, los usuarios solo pueden leer datos de Key Vault cuando las solicitudes se originan desde redes virtuales o rangos de direcciones IPv4 permitidos. Esto también se aplica al acceso de Key Vault desde Azure Portal. Aunque los usuarios pueden ir a un almacén de claves desde Azure Portal, es posible que no pueda enumerar las claves, los secretos o los certificados si su equipo cliente no está en la lista de dispositivos permitidos. Para conocer los pasos de implementación, consulte [Configuración de firewalls y redes virtuales de Azure Key Vault](network-security.md).

El servicio Azure Private Link permite acceder a Azure Key Vault y a los servicios de asociados o clientes hospedados de Azure mediante un punto de conexión privado de la red virtual. Un punto de conexión privado de Azure es una interfaz de red que le conecta de forma privada y segura a un servicio con la tecnología de Azure Private Link. El punto de conexión privado usa una dirección IP privada de la red virtual para incorporar el servicio de manera eficaz a su red virtual. Todo el tráfico dirigido al servicio se puede enrutar mediante el punto de conexión privado, por lo que no se necesita ninguna puerta de enlace, dispositivos NAT, conexiones de ExpressRoute o VPN ni direcciones IP públicas. El tráfico entre la red virtual y el servicio atraviesa la red troncal de Microsoft, eliminando la exposición a la red pública de Internet. Puede conectarse a una instancia de un recurso de Azure, lo que le otorga el nivel más alto de granularidad en el control de acceso.  Para conocer los pasos de implementación, consulte [Integración de Key Vault con Azure Private Link](private-link-service.md).

## <a name="tls-and-https"></a>TLS y HTTPS

- El front-end de Key Vault (plano de datos) es un servidor de varios inquilinos. Esto significa que los almacenes de claves de distintos clientes pueden compartir la misma dirección IP pública. Con el fin de lograr el aislamiento, cada solicitud HTTP se autentica y autoriza con independencia de otras solicitudes.
- Puede identificar versiones anteriores de TLS para notificar las vulnerabilidades, pero debido a que la dirección IP pública está compartida, no es posible que el equipo de servicio de Key Vault deshabilite las versiones anteriores de TLS para almacenes de claves individuales en el nivel de transporte.
- El protocolo HTTPS permite al cliente participar en la negociación de TLS. Los **clientes pueden aplicar la última versión de TLS** y, cada vez que un cliente lo hace, toda la conexión usará la protección de nivel correspondiente. El hecho de que Key Vault siga siendo compatible con versiones anteriores de TLS no afectará a la seguridad de las conexiones con las versiones más recientes de TLS.
- A pesar de las vulnerabilidades conocidas del protocolo TLS, no hay ningún ataque conocido que permita a un agente malintencionado extraer cualquier información de su almacén de claves cuando el atacante inicie una conexión con una versión de TLS que tenga vulnerabilidades. El atacante todavía tendría que autenticarse y autorizarse a sí mismo, y siempre y cuando los clientes legítimos se conecten siempre con versiones de TLS recientes, no existe la posibilidad de que las credenciales se hayan filtrado de las vulnerabilidades de versiones de TLS anteriores.

## <a name="identity-management"></a>Administración de identidades

Cuando se crea un almacén de claves en una suscripción de Azure, se asocia automáticamente al inquilino de Azure AD de dicha suscripción. Cualquier persona que intente administrar o recuperar el contenido desde un almacén debe autenticarse con Azure AD. En ambos casos, las aplicaciones pueden acceder a Key Vault de tres maneras:

- **Solo la aplicación**: La aplicación representa una entidad de servicio o una identidad administrada. Esta identidad es el escenario más común para aplicaciones que necesitan acceder periódicamente a certificados, claves o secretos del almacén de claves. Para que este escenario funcione, el elemento `objectId` de la aplicación debe especificarse en la directiva de acceso y el elemento `applicationId` _no_ debe especificarse o debe ser `null`.
- **Solo el usuario**: el usuario accede al almacén de claves desde cualquier aplicación registrada en el inquilino. Los ejemplos de este tipo de acceso incluyen Azure PowerShell y Azure Portal. Para que este escenario funcione, el elemento `objectId` del usuario debe especificarse en la directiva de acceso y el elemento `applicationId` _no_ debe especificarse o debe ser `null`.
- **Aplicación y usuario** (a veces denominado _identidad compuesta_): 4el usuario tiene que acceder al almacén de claves desde una aplicación específica _y_ la aplicación debe usar el flujo de autenticación en nombre de (OBO) para suplantar al usuario. Para que este escenario funcione, se deben especificar ambos objetos `applicationId` y `objectId` en la directiva de acceso. El elemento `applicationId` identifica la aplicación necesaria y el elemento `objectId` identifica al usuario. Actualmente, esta opción no está disponible para Azure RBAC en el plano de datos.

Para todos los tipos de acceso, la aplicación se autentica con Azure AD. La aplicación utiliza cualquiera [método de autenticación compatible](../../active-directory/develop/authentication-vs-authorization.md) según el tipo de aplicación. La aplicación adquiere un token para un recurso del plano para conceder acceso. El recurso es un punto de conexión en el plano de administración o de datos, según el entorno de Azure. La aplicación usa el token y envía la solicitud de una API de REST a Key Vault. Para más información, revise [todo el flujo de autenticación](../../active-directory/develop/v2-oauth2-auth-code-flow.md).

Para conocer los detalles completos, consulte [Aspectos básicos de la autenticación de Key Vault](authentication-fundamentals.md).

## <a name="key-vault-authentication-options"></a>Opciones de autenticación de Key Vault

Cuando se crea un almacén de claves en una suscripción de Azure, se asocia automáticamente al inquilino de Azure AD de dicha suscripción. En ambos planos, todos los llamadores deben registrarse en este inquilino y autenticarse para acceder al almacén de claves. En ambos casos, las aplicaciones pueden acceder a Key Vault de tres maneras:

- **Solo la aplicación**: La aplicación representa una entidad de servicio o una identidad administrada. Esta identidad es el escenario más común para aplicaciones que necesitan acceder periódicamente a certificados, claves o secretos del almacén de claves. Para que este escenario funcione, el elemento `objectId` de la aplicación debe especificarse en la directiva de acceso y el elemento `applicationId` _no_ debe especificarse o debe ser `null`.
- **Solo el usuario**: el usuario accede al almacén de claves desde cualquier aplicación registrada en el inquilino. Los ejemplos de este tipo de acceso incluyen Azure PowerShell y Azure Portal. Para que este escenario funcione, el elemento `objectId` del usuario debe especificarse en la directiva de acceso y el elemento `applicationId` _no_ debe especificarse o debe ser `null`.
- **Aplicación y usuario** (a veces denominado _identidad compuesta_): 4el usuario tiene que acceder al almacén de claves desde una aplicación específica _y_ la aplicación debe usar el flujo de autenticación en nombre de (OBO) para suplantar al usuario. Para que este escenario funcione, se deben especificar ambos objetos `applicationId` y `objectId` en la directiva de acceso. El elemento `applicationId` identifica la aplicación necesaria y el elemento `objectId` identifica al usuario. Esta opción no está disponible actualmente para Azure RBAC en el plano de datos (versión preliminar)

Para todos los tipos de acceso, la aplicación se autentica con Azure AD. La aplicación utiliza cualquiera [método de autenticación compatible](../../active-directory/develop/authentication-vs-authorization.md) según el tipo de aplicación. La aplicación adquiere un token para un recurso del plano para conceder acceso. El recurso es un punto de conexión en el plano de administración o de datos, según el entorno de Azure. La aplicación usa el token y envía la solicitud de una API de REST a Key Vault. Para más información, revise [todo el flujo de autenticación](../../active-directory/develop/v2-oauth2-auth-code-flow.md).

El modelo de un único mecanismo de autenticación para ambos planos tiene varias ventajas:

- Las organizaciones pueden controlar el acceso de forma centralizada a todos los almacenes de claves de su organización.
- Si un usuario abandona la organización, al instante pierde el acceso a todos los almacenes de claves de la organización.
- Las organizaciones pueden personalizar la autenticación mediante las opciones de Azure AD, como la habilitación de Multi-Factor Authentication para aumentar la seguridad.

## <a name="access-model-overview"></a>Introducción al modelo acceso

El acceso a un almacén de claves se controla mediante dos interfaces: el **plano de administración** y el **plano de datos**. El plano de administración es donde puede administrar el propio almacén de claves. Las operaciones en este plano incluyen crear y eliminar los almacenes de claves, recuperar las propiedades de un almacén de claves y actualizar las directivas de acceso. El plano de datos es donde se trabaja con los datos almacenados en un almacén de claves. Puede agregar, eliminar y modificar claves, secretos y certificados.

Ambos planos usan [Azure Active Directory (Azure AD)](../../active-directory/fundamentals/active-directory-whatis.md) para la autenticación. Para realizar la autorización, el plano de administración usa el [control de acceso basado en roles de Azure (RBAC de Azure)](../../role-based-access-control/overview.md) y el plano de datos utiliza una [directiva de acceso de Key Vault](./assign-access-policy-portal.md) y [RBAC de Azure para las operaciones del plano de datos de Key Vault](./rbac-guide.md).

Para obtener acceso a un almacén de claves en cualquier plano, todos los llamadores (usuarios o aplicaciones) deben tener una autorización y autenticación correctas. La autenticación establece la identidad del llamador. La autorización determina las operaciones que puede ejecutar el llamador. La autenticación con Key Vault funciona junto con [Azure Active Directory (Azure AD)](../../active-directory/fundamentals/active-directory-whatis.md), que es responsable de autenticar la identidad de cualquier **entidad de seguridad**.

Una entidad de seguridad es un objeto que representa un usuario, un grupo o una entidad de servicio que solicita acceso a recursos de Azure. Azure asigna un **identificador de objeto** único a cada entidad de seguridad.

- Una entidad de seguridad de **usuario** identifica un individuo que tiene un perfil en Azure Active Directory.
- Una entidad de seguridad de **grupo** identifica un conjunto de usuarios creados en Azure Active Directory. Los roles o permisos asignados al grupo se conceden a todos los usuarios dentro del grupo.
- Una **entidad de servicio** es un tipo de entidad de seguridad que identifica una aplicación o un servicio, es decir, un fragmento de código en lugar de un usuario o grupo. El identificador de objeto de una entidad de servicio se conoce como su **Id. de cliente** y actúa como su nombre de usuario. El **secreto de cliente** de la entidad de servicio o el **certificado** actúa como contraseña. Muchos servicios de Azure admiten la asignación de [identidades administradas](../../active-directory/managed-identities-azure-resources/overview.md) con la administración automatizada del **id. de cliente** y el **certificado**. La identidad administrada es la opción más segura y recomendada para realizar la autenticación en Azure.

Para obtener más información sobre la autenticación en Key Vault, consulte [Autenticación en Azure Key Vault](authentication.md).

## <a name="privileged-access"></a>Acceso con privilegios

La autorización determina las operaciones que puede realizar quien envía la llamada. La autorización en Key Vault usa una combinación de [control de acceso basado en roles de Azure ](../../role-based-access-control/overview.md) (Azure RBAC) y directivas de acceso de Azure Key Vault.

El acceso a los almacenes se realiza a través de dos interfaces o planos. Estos son el plano de administración y el plano de datos.

- El *plano de administración* es donde se administra Key Vault y es la interfaz utilizada para crear y eliminar almacenes. También puede leer las propiedades del almacén de claves y administrar las directivas de acceso.
- El *plano de datos* es donde se trabaja con los datos almacenados en un almacén de claves. Puede agregar, eliminar y modificar claves, secretos y certificados.

Las aplicaciones acceden a los planos a través de puntos de conexión. Los controles de acceso para los dos planos funcionan de forma independiente. Para conceder acceso a una aplicación para usar las claves de un almacén de claves, debe conceder acceso al plano de datos mediante una directiva de acceso de Key Vault o Azure RBAC. Para conceder a un usuario acceso de lectura a las propiedades y etiquetas de Key Vault, pero sin que este pueda acceder a los datos (claves, secretos o certificados), debe conceder acceso al plano de administración con Azure RBAC.

En la siguiente tabla se muestran los puntos de conexión para los planos de administración y datos.

| Plano de&nbsp;acceso | Puntos de conexión de acceso | Operaciones | Mecanismo de&nbsp;control de acceso |
| --- | --- | --- | --- |
| Plano de administración | **Global:**<br> management.azure.com:443<br><br> **Azure China 21Vianet:**<br> management.chinacloudapi.cn:443<br><br> **Azure US Gov:**<br> management.usgovcloudapi.net:443<br><br> **Azure Alemania:**<br> management.microsoftazure.de:443 | Crear, leer, actualizar y eliminar almacenes de claves<br><br>Establecer directivas de acceso de Key Vault<br><br>Establecer etiquetas de Key Vault | Azure RBAC |
| Plano de datos | **Global:**<br> &lt;vault-name&gt;.vault.azure.net:443<br><br> **Azure China 21Vianet:**<br> &lt;vault-name&gt;.vault.azure.cn:443<br><br> **Azure US Gov:**<br> &lt;vault-name&gt;.vault.usgovcloudapi.net:443<br><br> **Azure Alemania:**<br> &lt;vault-name&gt;.vault.microsoftazure.de:443 | Claves: encrypt, decrypt, wrapKey, unwrapKey, sign, verify, get, list, create, update, import, delete, recover, backup, restore y purge<br><br> Certificados: managecontacts, getissuers, listissuers, setissuers, deleteissuers, manageissuers, get, list, create, import, update, delete, recover, backup, restore y purge<br><br>  Secretos: get, list, set, delete, recover, backup, restore y purge | Directiva de acceso de Key Vault o Azure RBAC |

### <a name="managing-administrative-access-to-key-vault"></a>Administración del acceso administrativo a Key Vault

Cuando crea un almacén de claves en un grupo de recursos, administra el acceso mediante Azure AD. Puede conceder a usuarios o grupos la capacidad de administrar los almacenes de claves en un grupo de recursos. Puede conceder acceso a un nivel de ámbito específico si asigna los roles de Azure apropiados. Para conceder acceso a un usuario para administrar almacén de claves, debe asignar un rol `key vault Contributor` predefinido al usuario en un ámbito específico. Los siguientes niveles de ámbitos se pueden asignar a un rol de Azure:

- **Suscripción**: Un rol de Azure asignado al nivel de suscripción se aplica a todos los recursos y grupos de recursos de esa suscripción.
- **Grupo de recursos**: Un rol de Azure asignado al nivel de grupo de recursos se aplica a todos los recursos de dicho grupo de recursos.
- **Recurso específico**: un rol de Azure asignado a un recurso concreto se aplica a dicho recurso. En este caso, el recurso es un almacén de claves específico.

Existen varios roles predefinidos. Si un rol predefinido no se ajusta a sus necesidades, puede definir uno propio. Para más información, consulte [Azure RBAC: roles integrados](../../role-based-access-control/built-in-roles.md).

> [!IMPORTANT]
> Si un usuario tiene permisos `Contributor` en un plano de administración de Key Vault, se puede conceder a sí mismo acceso al plano de datos estableciendo una directiva de acceso de Key Vault. Debe controlar estrechamente quién tiene el rol de acceso `Contributor` a los almacenes de claves. Asegúrese de que solo las personas autorizadas pueden acceder a los almacenes de claves, las claves, los secretos y los certificados y administrarlos.

### <a name="controlling-access-to-key-vault-data"></a>Control del acceso a datos de Key Vault

Las directivas de acceso de Key Vault conceden permisos independientes a las claves, los secretos o los certificados. Puede conceder acceso a un usuario solo a las claves y no a los secretos. Los permisos para acceder a las claves, los secretos y los certificados se administran en el nivel del almacén.

> [!IMPORTANT]
> Las directivas de acceso de Key Vault no admiten permisos de nivel de objeto pormenorizados, como una clave, un secreto o un certificado específicos. Cuando se concede a un usuario permiso para crear y eliminar claves, este puede realizar dichas operaciones en todas las claves de dicho almacén de claves.

Puede establecer directivas de acceso de un almacén de claves mediante [Azure Portal](assign-access-policy-portal.md), la [CLI de Azure](assign-access-policy-cli.md), [Azure PowerShell](assign-access-policy-powershell.md) o las [API de REST de administración de Key Vault](/rest/api/keyvault/).

## <a name="logging-and-monitoring"></a>Registro y supervisión

Los registros de Key Vault guardan información acerca de las actividades realizadas en el almacén. Para obtener información completa, vea [Registro de Key Vault](logging.md).

Puede integrar Key Vault con Event Grid para recibir notificaciones cuando el estado de una clave, un certificado o un secreto almacenados en Key Vault haya cambiado. Para más información, consulte [Supervisión de Key Vault con Azure Event Grid](event-grid-overview.md).

También es importante supervisar el estado del almacén de claves para comprobar que el servicio funciona según lo previsto. Para saber cómo hacerlo, consulte [Supervisión y alertas de Azure Key Vault](alert.md).

## <a name="backup-and-recovery"></a>Copia de seguridad y recuperación

La protección frente a la eliminación temporal y la purga de Azure Key Vault permite recuperar almacenes y objetos de almacén eliminados. Para conocer los detalles completos, consulte [Introducción a la eliminación temporal en Azure Key Vault](soft-delete-overview.md).

También, debe hacer copias de seguridad del almacén periódicamente, cuando actualice, elimine o cree objetos dentro de él.  

## <a name="next-steps"></a>Pasos siguientes

- [Línea de base de seguridad de Azure Key Vault](security-baseline.md)
- [Procedimientos recomendados de Azure Key Vault](security-baseline.md)
- [Puntos de conexión de servicio de red virtual para Azure Key Vault](overview-vnet-service-endpoints.md)
- [RBAC de Azure: roles integrados](../../role-based-access-control/built-in-roles.md)