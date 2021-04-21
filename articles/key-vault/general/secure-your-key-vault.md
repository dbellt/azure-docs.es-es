---
title: Protección del acceso a un almacén de claves
description: El modelo de acceso para Azure Key Vault, incluidos los puntos de conexión de recursos y autenticación de Active Directory.
services: key-vault
author: ShaneBala-keyvault
manager: ravijan
ms.service: key-vault
ms.subservice: general
ms.topic: conceptual
ms.date: 10/07/2020
ms.author: sudbalas
ms.openlocfilehash: b1ec89db7bc12ffbd21c6e302e065449eba3ac20
ms.sourcegitcommit: dddd1596fa368f68861856849fbbbb9ea55cb4c7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/13/2021
ms.locfileid: "107366494"
---
# <a name="secure-access-to-a-key-vault"></a>Protección del acceso a un almacén de claves

Azure Key Vault es un servicio en la nube que protege las claves de cifrado y los secretos, como certificados, cadenas de conexión y contraseñas. Dado que estos datos son confidenciales y críticos para la empresa, debe proteger el acceso a los almacenes de claves, de modo que solo se admita el acceso de las aplicaciones y los usuarios autorizados. En este artículo se proporciona información general del modelo de acceso de Key Vault. Se explican la autenticación y la autorización, y se describe cómo proteger el acceso a los almacenes de claves.

Para obtener más información sobre Key Vault, consulte el artículo [Acerca de Azure Key Vault](overview.md); para obtener más información sobre lo que se puede almacenar en un almacén de claves, consulte el artículo [Acerca de las claves, secretos y certificados](about-keys-secrets-certificates.md).

## <a name="access-model-overview"></a>Introducción al modelo acceso

El acceso a un almacén de claves se controla mediante dos interfaces: el **plano de administración** y el **plano de datos**. El plano de administración es donde puede administrar el propio almacén de claves. Las operaciones en este plano incluyen crear y eliminar los almacenes de claves, recuperar las propiedades de un almacén de claves y actualizar las directivas de acceso. El plano de datos es donde se trabaja con los datos almacenados en un almacén de claves. Puede agregar, eliminar y modificar claves, secretos y certificados.

Ambos planos usan [Azure Active Directory (Azure AD)](../../active-directory/fundamentals/active-directory-whatis.md) para la autenticación. Para realizar la autorización, el plano de administración usa el [control de acceso basado en roles de Azure (RBAC de Azure)](../../role-based-access-control/overview.md) y el plano de datos utiliza una [directiva de acceso de Key Vault](./assign-access-policy-portal.md) y [RBAC de Azure para las operaciones del plano de datos de Key Vault](./rbac-guide.md).

Para obtener acceso a un almacén de claves en cualquier plano, todos los llamadores (usuarios o aplicaciones) deben tener una autorización y autenticación correctas. La autenticación establece la identidad del llamador. La autorización determina las operaciones que puede ejecutar el llamador. La autenticación con Key Vault funciona junto con [Azure Active Directory (Azure AD)](../../active-directory/fundamentals/active-directory-whatis.md), que es responsable de autenticar la identidad de cualquier **entidad de seguridad**.

Una entidad de seguridad es un objeto que representa un usuario, un grupo o una entidad de servicio que solicita acceso a recursos de Azure. Azure asigna un **identificador de objeto** único a cada entidad de seguridad.

* Una entidad de seguridad de **usuario** identifica un individuo que tiene un perfil en Azure Active Directory.

* Una entidad de seguridad de **grupo** identifica un conjunto de usuarios creados en Azure Active Directory. Los roles o permisos asignados al grupo se conceden a todos los usuarios dentro del grupo.

* Una **entidad de servicio** es un tipo de entidad de seguridad que identifica una aplicación o un servicio, es decir, un fragmento de código en lugar de un usuario o grupo. El identificador de objeto de una entidad de servicio se conoce como su **Id. de cliente** y actúa como su nombre de usuario. El **secreto de cliente** de la entidad de servicio o el **certificado** actúa como contraseña. Muchos servicios de Azure admiten la asignación de [identidades administradas](../../active-directory/managed-identities-azure-resources/overview.md) con la administración automatizada del **id. de cliente** y el **certificado**. La identidad administrada es la opción más segura y recomendada para realizar la autenticación en Azure.

Para obtener más información sobre la autenticación en Key Vault, consulte [Autenticación en Azure Key Vault](authentication.md).

## <a name="key-vault-authentication-options"></a>Opciones de autenticación de Key Vault

Cuando se crea un almacén de claves en una suscripción de Azure, se asocia automáticamente al inquilino de Azure AD de dicha suscripción. En ambos planos, todos los llamadores deben registrarse en este inquilino y autenticarse para acceder al almacén de claves. En ambos casos, las aplicaciones pueden acceder a Key Vault de tres maneras:

- **Solo la aplicación**: La aplicación representa una entidad de servicio o una identidad administrada. Esta identidad es el escenario más común para aplicaciones que necesitan acceder periódicamente a certificados, claves o secretos del almacén de claves. Para que este escenario funcione, el elemento `objectId` de la aplicación debe especificarse en la directiva de acceso y el elemento `applicationId` _no_ debe especificarse o debe ser `null`.
- **Solo el usuario**: el usuario accede al almacén de claves desde cualquier aplicación registrada en el inquilino. Los ejemplos de este tipo de acceso incluyen Azure PowerShell y Azure Portal. Para que este escenario funcione, el elemento `objectId` del usuario debe especificarse en la directiva de acceso y el elemento `applicationId` _no_ debe especificarse o debe ser `null`.
- **Aplicación y usuario** (a veces denominado _identidad compuesta_): 4el usuario tiene que acceder al almacén de claves desde una aplicación específica _y_ la aplicación debe usar el flujo de autenticación en nombre de (OBO) para suplantar al usuario. Para que este escenario funcione, se deben especificar ambos objetos `applicationId` y `objectId` en la directiva de acceso. El elemento `applicationId` identifica la aplicación necesaria y el elemento `objectId` identifica al usuario. Actualmente, esta opción no está disponible para Azure RBAC en el plano de datos.

Para todos los tipos de acceso, la aplicación se autentica con Azure AD. La aplicación utiliza cualquiera [método de autenticación compatible](../../active-directory/develop/authentication-vs-authorization.md) según el tipo de aplicación. La aplicación adquiere un token para un recurso del plano para conceder acceso. El recurso es un punto de conexión en el plano de administración o de datos, según el entorno de Azure. La aplicación usa el token y envía la solicitud de una API de REST a Key Vault. Para más información, revise [todo el flujo de autenticación](../../active-directory/develop/v2-oauth2-auth-code-flow.md).

El modelo de un único mecanismo de autenticación para ambos planos tiene varias ventajas:

- Las organizaciones pueden controlar el acceso de forma centralizada a todos los almacenes de claves de su organización.
- Si un usuario abandona la organización, al instante pierde el acceso a todos los almacenes de claves de la organización.
- Las organizaciones pueden personalizar la autenticación mediante las opciones de Azure AD, como la habilitación de Multi-Factor Authentication para aumentar la seguridad.

## <a name="resource-endpoints"></a>Puntos de conexión de recursos

Las aplicaciones acceden a los planos a través de puntos de conexión. Los controles de acceso para los dos planos funcionan de forma independiente. Para conceder acceso a una aplicación para usar las claves de un almacén de claves, debe conceder acceso al plano de datos mediante una directiva de acceso de Key Vault o Azure RBAC. Para conceder a un usuario acceso de lectura a las propiedades y etiquetas de Key Vault, pero sin que este pueda acceder a los datos (claves, secretos o certificados), debe conceder acceso al plano de administración con Azure RBAC.

En la siguiente tabla se muestran los puntos de conexión para los planos de administración y datos.

| Plano de&nbsp;acceso | Puntos de conexión de acceso | Operaciones | Mecanismo de&nbsp;control de acceso |
| --- | --- | --- | --- |
| Plano de administración | **Global:**<br> management.azure.com:443<br><br> **Azure China 21Vianet:**<br> management.chinacloudapi.cn:443<br><br> **Azure US Gov:**<br> management.usgovcloudapi.net:443<br><br> **Azure Alemania:**<br> management.microsoftazure.de:443 | Crear, leer, actualizar y eliminar almacenes de claves<br><br>Establecer directivas de acceso de Key Vault<br><br>Establecer etiquetas de Key Vault | Azure RBAC |
| Plano de datos | **Global:**<br> &lt;vault-name&gt;.vault.azure.net:443<br><br> **Azure China 21Vianet:**<br> &lt;vault-name&gt;.vault.azure.cn:443<br><br> **Azure US Gov:**<br> &lt;vault-name&gt;.vault.usgovcloudapi.net:443<br><br> **Azure Alemania:**<br> &lt;vault-name&gt;.vault.microsoftazure.de:443 | Claves: encrypt, decrypt, wrapKey, unwrapKey, sign, verify, get, list, create, update, import, delete, recover, backup, restore y purge<br><br> Certificados: managecontacts, getissuers, listissuers, setissuers, deleteissuers, manageissuers, get, list, create, import, update, delete, recover, backup, restore y purge<br><br>  Secretos: get, list, set, delete, recover, backup, restore y purge | Directiva de acceso de Key Vault o Azure RBAC |

## <a name="management-plane-and-azure-rbac"></a>Plano de administración y Azure RBAC

En el plano de administración, utilice el [control de acceso basado en roles de Azure (Azure RBAC)](../../role-based-access-control/overview.md) para autorizar las operaciones que el autor de la llamada puede ejecutar. En el modelo de Azure RBAC, cada suscripción de Azure tiene una instancia de Azure AD. Puede conceder acceso a usuarios, grupos y aplicaciones desde este directorio. El acceso se concede para administrar recursos de la suscripción de Azure que usan el modelo de implementación de Azure Resource Manager.

Puede crear un almacén de claves en un grupo de recursos y administrar el acceso mediante Azure AD. Puede conceder a usuarios o grupos la capacidad de administrar los almacenes de claves en un grupo de recursos. Puede conceder acceso a un nivel de ámbito específico mediante la asignación de roles de Azure apropiados. Para conceder acceso a un usuario para administrar los almacenes de claves, debe asignar el rol predefinido [Colaborador de almacén de claves](../../role-based-access-control/built-in-roles.md#key-vault-contributor) a este usuario en un ámbito específico. Los siguientes niveles de ámbitos se pueden asignar a un rol de Azure:

- **Suscripción**: Un rol de Azure asignado al nivel de suscripción se aplica a todos los recursos y grupos de recursos de esa suscripción.
- **Grupo de recursos**: Un rol de Azure asignado al nivel de grupo de recursos se aplica a todos los recursos de dicho grupo de recursos.
- **Recurso específico**: un rol de Azure asignado a un recurso concreto se aplica a dicho recurso. En este caso, el recurso es un almacén de claves específico.

Existen varios roles predefinidos. Si un rol predefinido no se ajusta a sus necesidades, puede definir uno propio. Para más información, consulte [Roles integrados en Azure](../../role-based-access-control/built-in-roles.md). 

Debe tener los permisos `Microsoft.Authorization/roleAssignments/write` y `Microsoft.Authorization/roleAssignments/delete`, como [Administrador de acceso de usuario](../../role-based-access-control/built-in-roles.md#user-access-administrator) o [Propietario](../../role-based-access-control/built-in-roles.md#owner).

> [!IMPORTANT]
> Si un usuario tiene permisos `Contributor` en un plano de administración de Key Vault, se puede conceder a sí mismo acceso al plano de datos estableciendo una directiva de acceso de Key Vault. Debe controlar estrechamente quién tiene el rol de acceso `Contributor` a los almacenes de claves. Asegúrese de que solo las personas autorizadas pueden acceder a los almacenes de claves, las claves, los secretos y los certificados y administrarlos.
>

<a id="data-plane-access-control"></a>
## <a name="data-plane-and-access-policies"></a>Directivas de acceso y plano de datos

Puede conceder acceso al plano de datos mediante el establecimiento de directivas de acceso de Key Vault para un almacén de claves. Para establecer estas directivas de acceso, un usuario, grupo o aplicación deben tener permisos `Key Vault Contributor` para el plano de administración de dicho almacén de claves.

Puede conceder acceso a un usuario, grupo o aplicación para ejecutar operaciones concretas en las claves o los secretos de un almacén de claves. Key Vault admite hasta 1024 entradas de directivas de acceso para un almacén de claves. Para conceder acceso al plano de datos a varios usuarios, cree un grupo de seguridad de Azure AD y agregue usuarios a dicho grupo.

Puede ver la lista completa de operaciones con almacenes y secretos aquí: [Referencia de operación de Key Vault](/rest/api/keyvault/#vault-operations)

<a id="key-vault-access-policies"></a> Las directivas de acceso de Key Vault conceden permisos a las claves, los secretos y los certificados por separado.  Los permisos para acceder a las claves, los secretos y los certificados se encuentran en el nivel del almacén. 

Para obtener más información sobre el uso de las directivas de acceso del almacén de claves, consulte [Asignación de una directiva de acceso de Key Vault](assign-access-policy-portal.md).

> [!IMPORTANT]
> Las directivas de acceso de Key Vault se aplican en el nivel de almacén. Cuando se concede a un usuario permiso para crear y eliminar claves, este puede realizar dichas operaciones en todas las claves de dicho almacén de claves.
Las directivas de acceso de Key Vault no admiten permisos de nivel de objeto pormenorizados, como una clave, un secreto o un certificado específicos. 
>

## <a name="data-plane-and-azure-rbac"></a>Plano de datos y RBAC de Azure

El control de acceso basado en roles de Azure es un modelo de permisos alternativo para controlar el acceso al plano de datos de Azure Key Vault, que se puede habilitar en almacenes de claves individuales. El modelo de permisos de Azure RBAC es exclusivo y una vez establecido, las directivas de acceso al almacén se convierten en inactivas. Azure Key Vault define un conjunto de roles integrados de Azure que engloban los conjuntos comunes de permisos que se usan para acceder a las claves, los secretos o los certificados.

Cuando un rol de Azure se asigna a una entidad de seguridad de Azure AD, Azure concede a esa entidad de seguridad acceso a los recursos. El acceso se puede limitar al nivel de la suscripción, el grupo de recursos, el almacén de claves o una clave, secreto o certificado individual. Una entidad de seguridad de Azure AD puede ser un usuario, un grupo, una entidad de servicio de aplicación o una [identidad de servicio administrada para recursos de Azure](../../active-directory/managed-identities-azure-resources/overview.md).

Las principales ventajas de usar el permiso de Azure RBAC en las directivas de acceso al almacén son la administración centralizada del control de acceso y su integración con [Privileged Identity Management (PIM)](../../active-directory/privileged-identity-management/pim-configure.md). Privileged Identity Management proporciona activación de rol basada en tiempo y en aprobación para mitigar los riesgos de tener unos permisos de acceso excesivos, innecesarios o mal utilizados en los recursos que le interesan.

Para obtener más información sobre el plano de datos de Key Vault con RBAC de Azure, consulte las [Claves, certificados y secretos de Key Vault con un control de acceso basado en roles de Azure](rbac-guide.md).

## <a name="firewalls-and-virtual-networks"></a>Firewalls y redes virtuales

Para una capa adicional de seguridad, puede configurar firewall y reglas de red virtual. Puede configurar los firewalls y redes virtuales de Key Vault para denegar el acceso al tráfico de todas las redes (incluido el tráfico de Internet) de forma predeterminada. Puede conceder acceso al tráfico desde [redes virtuales específicas de Azure](../../virtual-network/virtual-networks-overview.md) y rangos de direcciones IP públicas de Internet, lo que le permite generar un límite de red seguro para las aplicaciones.

Estos son algunos ejemplos de cómo puede usar los puntos de conexión de servicio:

* Si utiliza Key Vault para almacenar claves de cifrado, secretos de aplicación o certificados y quiere bloquear el acceso al almacén de claves de Internet pública.
* Si quiere bloquear el acceso al almacén de claves para que solo la aplicación o una lista breve de los hosts designados puedan conectarse al almacén de claves.
* Tiene una aplicación que se ejecuta en su red virtual de Azure y esta red virtual está bloqueada para todo el tráfico entrante y saliente. Su aplicación aún necesita conectarse a Key Vault para capturar secretos o certificados, o usar claves criptográficas.

Para obtener más información sobre el firewall de Key Vault y las redes virtuales, consulte [Configuración de firewalls y redes virtuales de Azure Key Vault](network-security.md).

> [!NOTE]
> Los firewall de Key Vault y las reglas de red virtual solo se aplican al plan de datos de Key Vault. Las operaciones del plano de control de Key Vault (como crear, eliminar, modificar operaciones, configurar directivas de acceso, y configurar firewall y reglas de red virtual) no se ven afectadas por los firewall ni las reglas de red virtual.

## <a name="private-endpoint-connection"></a>Conexión de punto de conexión privado

En caso de que sea necesario bloquear completamente la exposición de Key Vault al público, se puede usar un [punto de conexión privado de Azure](../../private-link/private-endpoint-overview.md). Un punto de conexión privado de Azure es una interfaz de red que le conecta de forma privada y segura a un servicio con la tecnología de Azure Private Link. El punto de conexión privado usa una dirección IP privada de la red virtual para incorporar el servicio de manera eficaz a su red virtual. Todo el tráfico dirigido al servicio se puede enrutar mediante el punto de conexión privado, por lo que no se necesita ninguna puerta de enlace, dispositivos NAT, conexiones de ExpressRoute o VPN ni direcciones IP públicas. El tráfico entre la red virtual y el servicio atraviesa la red troncal de Microsoft, eliminando la exposición a la red pública de Internet. Puede conectarse a una instancia de un recurso de Azure, lo que le otorga el nivel más alto de granularidad en el control de acceso.

Escenarios comunes para usar Private Link para servicios de Azure:

- **Acceso privado a servicios en la plataforma Azure**: conecte la red virtual a los servicios de Azure sin una dirección IP pública en el origen o el destino. Los proveedores de servicios pueden representar los servicios en su propia red virtual y los consumidores pueden acceder a ellos en su red virtual local. La plataforma Private Link administrará la conectividad entre el consumidor y los servicios a través de la red troncal de Azure. 
 
- **Redes locales y emparejadas**: acceda a los servicios que se ejecutan en Azure desde el entorno local a través del emparejamiento privado de ExpressRoute, los túneles VPN y las redes virtuales emparejadas mediante puntos de conexión privados. No es necesario configurar el emparejamiento público ni atravesar Internet para llegar hasta el servicio. Private Link proporciona una manera segura de migrar cargas de trabajo a Azure.
 
- **Protección contra la pérdida de datos**: un punto de conexión privado se asigna a una instancia de un recurso de PaaS en lugar de al servicio entero. Los consumidores solo pueden conectarse al recurso específico. Se bloquea el acceso a cualquier otro recurso del servicio. Este mecanismo proporciona protección contra los riesgos de pérdida de datos. 
 
- **Alcance global**: conéctese de forma privada a los servicios que se ejecutan en otras regiones. La red virtual del consumidor podría estar en la región A y puede conectarse a los servicios que hay detrás de Private Link en la región B.  
 
- **Ampliación de los propios servicios**: habilite la misma experiencia y funcionalidad para representar su servicio de forma privada para los consumidores de Azure. Al colocar el servicio detrás de una instancia de Azure Load Balancer estándar, puede habilitarlo para Private Link. Después, el consumidor puede conectarse directamente al servicio mediante un punto de conexión privado en su propia red virtual. Puede administrar las solicitudes de conexión mediante un flujo de llamadas de aprobación. Azure Private Link funciona también para los consumidores y servicios que pertenecen a distintos inquilinos de Azure Active Directory. 

Para obtener más información acerca de los puntos de conexión privados, consulte [Key Vault con Azure Private Link](./private-link-service.md).

## <a name="example"></a>Ejemplo

En este ejemplo, se desarrolla una aplicación que usa un certificado para TLS/SSL, Azure Storage para almacenar los datos y una clave RSA de 2048 bits para cifrar datos en Azure Storage. La aplicación se ejecuta en una máquina virtual (VM) de Azure o en un conjunto de escalado de máquinas virtuales. Podemos usar un almacén de claves para almacenar los secretos de la aplicación. Podemos almacenar el certificado de arranque que usa la aplicación para autenticarse con Azure AD.

Se necesita acceso a los siguientes secretos y claves almacenados:
- **Certificado TLS/SSL**: Se usa para TLS/SSL.
- **Clave de almacenamiento**: se usa para acceder a la cuenta de almacenamiento.
- **Clave RSA de 2048 bits**: Se usa para encapsular y desencapsular la clave de cifrado de datos mediante Azure Storage.
- **Identidad administrada de aplicación**: se usa para la autenticación con Azure AD. Después de conceder acceso a Key Vault, la aplicación puede capturar el certificado y la clave de almacenamiento.

Es necesario definir los siguientes roles para especificar quién puede administrar, implementar y auditar la aplicación:
- **Equipo de seguridad**: el personal de TI de la oficina del CSO (director de seguridad) o colaboradores equivalentes. El equipo de seguridad es responsable de la protección adecuada de los secretos. Los secretos pueden incluir los certificados TLS/SSL, las claves RSA usadas para el cifrado, las cadenas de conexión y las claves de la cuenta de almacenamiento.
- **Desarrolladores y operadores**: personal que desarrolla la aplicación y la implementa en Azure. Los miembros de este equipo no forman parte del personal de seguridad. No deben tener acceso a información confidencial, como los certificados TLS/SSL y las claves RSA. Solo la aplicación que implementan debe tener acceso a información confidencial.
- **Auditores**: este rol es para los colaboradores que no sean miembros del personal de TI general o de desarrollo. Deben revisar el uso y mantenimiento de los certificados, las claves y los secretos para garantizar el cumplimiento de los estándares de seguridad.

Hay otro rol que está fuera del ámbito de la aplicación: el administrador de la suscripción o del grupo de recursos. El administrador de la suscripción configura los permisos de acceso iniciales del equipo de seguridad. Conceden acceso al equipo de seguridad mediante el uso de un grupo de recursos que tiene los recursos requeridos por la aplicación.

Es necesario autorizar las siguientes operaciones para los roles:

**Equipo de seguridad**
- Crear instancias de Key Vault.
- Activar el registro de Key Vault.
- Agregar claves y secretos.
- Crear copias de seguridad de las claves para la recuperación ante desastres.
- Establecer directivas de acceso de Key Vault y asignar roles para conceder permisos a usuarios y aplicaciones para operaciones concretas.
- Rotar periódicamente las claves y los secretos.

**Desarrolladores y operadores**
- Obtener referencias del equipo de seguridad para los certificados de arranque y TLS/SSL (huellas digitales), la clave de almacenamiento (identificador URI de secreto) y la clave RSA (identificador URI de clave) para encapsular y desencapsular.
- Desarrollar e implementar una aplicación para acceder a los certificados y los secretos mediante programación.

**Auditores**
- Revisar los registros de Key Vault para confirmar el uso adecuado de las claves y los secretos, así como el cumplimiento de los estándares de seguridad de datos.

En la tabla siguiente se resumen los permisos de acceso para los roles y la aplicación.

| Role | Permisos del plano de administración | Permisos de plano de datos: directivas de acceso de almacén | Permisos de plano de datos: RBAC de Azure  |
| --- | --- | --- | --- |
| Equipo de seguridad | [Colaborador de almacén de claves](../../role-based-access-control/built-in-roles.md#key-vault-contributor) | Certificados: todas las operaciones <br> Claves: todas las operaciones <br> Secretos: todas las operaciones | [Administrador de Key Vault](../../role-based-access-control/built-in-roles.md#key-vault-administrator) |
| Desarrolladores y&nbsp;operadores | Permiso de implementación en Key Vault<br><br> **Nota**: Este permiso permite que las máquinas virtuales implementadas recuperen los secretos de un almacén de claves. | None | None |
| Auditores | None | Certificados: lista <br> Claves: enumeración<br>Secretos: enumeración<br><br> **Nota**: Este permiso permite a los auditores inspeccionar los atributos (etiquetas, fechas de activación y fechas de expiración) para las claves y los secretos que no se emiten en los registros. | [Lector de Key Vault](../../role-based-access-control/built-in-roles.md#key-vault-reader) |
| Cuenta de Azure Storage | None | Claves: get, list, wrapKey, unwrapKey <br> | [Usuario de cifrado de servicio criptográfico de Key Vault](../../role-based-access-control/built-in-roles.md#key-vault-crypto-service-encryption-user) |
| Application | None | Secretos: get y list <br> Certificados: get y list | [Lector de Key Vault Reader](../../role-based-access-control/built-in-roles.md#key-vault-reader) y [usuario de secretos de Key Vault](../../role-based-access-control/built-in-roles.md#key-vault-secrets-user) |

Los tres roles de equipo necesitan tener acceso a otros recursos junto con los permisos de Key Vault. Para implementar las máquinas virtuales (o la característica Web Apps de Azure App Service), los desarrolladores y operadores necesitan implementar el acceso. Los auditores necesitan acceso de lectura a la cuenta de almacenamiento donde se almacenan los registros de Key Vault.

En el ejemplo se describe un escenario sencillo. Los escenarios reales pueden ser más complejos. Puede ajustar los permisos para el almacén de claves según sus necesidades. Se supone que el equipo de seguridad proporciona las referencias de clave y secreto (identificadores URI y huellas digitales), que el personal de DevOps utiliza en sus aplicaciones. Los desarrolladores y operadores no requieren ningún acceso al plano de datos. Nos centramos en cómo proteger el almacén de claves.

> [!NOTE]
> En este ejemplo se muestra cómo se bloquea el acceso a Key Vault en producción. Los desarrolladores deben tener su propia suscripción o grupo de recursos con permisos completos para administrar sus almacenes, máquinas virtuales y la cuenta de almacenamiento donde desarrollan la aplicación.

## <a name="resources"></a>Recursos

- [Acerca de Azure Key Vault](overview.md)
- [Azure Active Directory](../../active-directory/fundamentals/active-directory-whatis.md)
- [Privileged Identity Management](../../active-directory/privileged-identity-management/pim-configure.md)
- [Azure RBAC](../../role-based-access-control/overview.md)
- [Private Link](../../private-link/private-link-overview.md)

## <a name="next-steps"></a>Pasos siguientes

[Autenticación en Azure Key Vault](authentication.md)

[Asignación de una directiva de acceso de Key Vault](assign-access-policy-portal.md)

[Asignación de un rol de Azure para acceder a claves, secretos y certificados](rbac-guide.md)

[Configuración de firewalls y redes virtuales de Key Vault](network-security.md)

[Establecimiento de una conexión de vínculo privado con Key Vault](private-link-service.md)
