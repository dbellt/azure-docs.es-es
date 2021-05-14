---
title: Guía del desarrollador de Azure Key Vault
description: Los desarrolladores pueden usar Azure Key Vault para administrar las claves criptográficas en el entorno de Microsoft Azure.
services: key-vault
author: msmbaldwin
ms.service: key-vault
ms.subservice: general
ms.topic: how-to
ms.date: 10/05/2020
ms.author: mbaldwin
ms.openlocfilehash: 08ac1ae09741b63648aec2b51b6a774a46b9af7c
ms.sourcegitcommit: 260a2541e5e0e7327a445e1ee1be3ad20122b37e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/21/2021
ms.locfileid: "107818447"
---
# <a name="azure-key-vault-developers-guide"></a>Guía del desarrollador de Azure Key Vault

Key Vault le permite acceder de forma segura a información confidencial desde sus aplicaciones:

- Las claves y los secretos y los certificados están protegidos sin tener que escribir el código manualmente, y puede usarlos fácilmente en sus aplicaciones.
- Permite a los clientes poseer y administrar sus propios secretos, claves y certificados, para poder así concentrarse en proporcionar las características de software principales. De este modo, las aplicaciones no serán responsables de las claves, secretos y certificados del inquilino de sus clientes.
- La aplicación puede usar claves para la firma y el cifrado, pero también mantiene la opción de administrar de forma externa las claves de la aplicación. Para obtener más información sobre las claves, consulte [Acerca de las claves.](../keys/about-keys.md)
- Puede administrar credenciales, como contraseñas, claves de acceso y tokens de SAS almacenándolas en Key Vault como secretos; consulte [Acerca de los secretos](../secrets/about-secrets.md) para más información.
- Administrar certificados. Para obtener más información, consulte [Acerca de los certificados](../certificates/about-certificates.md).

Para obtener más información sobre Azure Key Vault, consulte [¿Qué es Key Vault?](overview.md).

## <a name="public-previews"></a>Versiones preliminares públicas

Periódicamente se publica una versión preliminar pública de una nueva característica de Key Vault. Pruebe las características de la versión preliminar públicas y díganos lo que piensa a través de nuestra dirección de correo electrónico para enviar comentarios, azurekeyvault@microsoft.com.

## <a name="creating-and-managing-key-vaults"></a>Creación y administración de almacenes de claves

La administración de Key Vault es similar a otros servicios de Azure y se realiza a través del servicio Azure Resource Manager. Azure Resource Manager es el servicio de implementación y administración para Azure. Proporciona una capa de administración que le permite crear, actualizar y eliminar recursos de la cuenta de Azure. Para obtener más información, consulte [Azure Resource Manager](../../azure-resource-manager/management/overview.md).

El acceso a la capa de administración se controla mediante el [Control de acceso basado en roles de Azure](../../role-based-access-control/overview.md). En Key Vault, la capa de administración, también conocida como administración o plano de control, le permite crear y administrar almacenes de claves y sus atributos, incluidas las directivas de acceso; no podrá administrar claves, secretos y certificados, puesto que se administran en el plano de datos. Puede usar el rol predefinido `Key Vault Contributor` para conceder acceso de administración a Key Vault.     

**API y SDK para la administración del almacén de claves:**

| Azure CLI | PowerShell | API DE REST | Resource Manager | .NET | Python | Java | JavaScript |  
|--|--|--|--|--|--|--|--|
|[Referencia](/cli/azure/keyvault)<br>[Guía de inicio rápido](quick-create-cli.md)|[Referencia](/powershell/module/az.keyvault)<br>[Guía de inicio rápido](quick-create-powershell.md)|[Referencia](/rest/api/keyvault/)|[Referencia](/azure/templates/microsoft.keyvault/vaults)<br>[Guía de inicio rápido](./vault-create-template.md)|[Referencia](/dotnet/api/microsoft.azure.management.keyvault)|[Referencia](/python/api/azure-mgmt-keyvault/azure.mgmt.keyvault)|[Referencia](/java/api/com.microsoft.azure.management.keyvault)|[Referencia](/javascript/api/@azure/arm-keyvault)|

Consulte las [Bibliotecas de cliente](client-libraries.md) para ver los paquetes de instalación y el código fuente.

Para obtener más información acerca del plano de administración de Key Vault, consulte [Características de seguridad de Azure Key Vault](security-features.md).

## <a name="authenticate-to-key-vault-in-code"></a>Autenticación en Key Vault mediante el código

Como Key Vault usa la autenticación de Azure AD, requiere que la entidad de seguridad de Azure AD le conceda acceso. Una entidad de seguridad de Azure AD puede ser un usuario, una entidad de servicio de aplicación, una [identidad administrada para recursos de Azure](../../active-directory/managed-identities-azure-resources/overview.md) o un grupo de cualquier tipo de entidad de seguridad.

### <a name="authentication-best-practices"></a>Procedimientos recomendados de autenticación

Se recomienda usar la identidad administrada para las aplicaciones implementadas en Azure. Si usa servicios de Azure, los cuales no admiten la identidad administrada o si las aplicaciones se implementan de forma local, la [entidad de servicio con un certificado](../../active-directory/develop/howto-create-service-principal-portal.md) es una alternativa posible. En ese escenario, el certificado debe almacenarse en Key Vault y cambiarse a menudo. La entidad de servicio que cuenta con el secreto se puede usar en entornos de desarrollo y pruebas; si se usa de forma local o en Cloud Shell, es recomendable usar la entidad de seguridad de usuario.

Entidades de seguridad recomendadas por entorno:
- **Entorno de producción**:
  - identidad administrada o entidad de servicio con un certificado.
- **Entornos de desarrollo y pruebas**:
  - identidad administrada, entidad de servicio con un certificado o entidad de servicio con un secreto.
- **Desarrollo local**:
  - entidad de seguridad de usuario o entidad de servicio con un secreto.

Los escenarios de autenticación anteriores son compatibles con la **biblioteca cliente de Azure Identity** y se integran con los SDK de Key Vault. La biblioteca de identidades de Azure se puede usar en diferentes entornos y plataformas sin tener que cambiar el código. La identidad de Azure también recuperará automáticamente el token de autenticación del usuario que inició sesión en Azure con la CLI de Azure, Visual Studio, Visual Studio Code y otros. 

Para obtener más información acerca de la biblioteca cliente de Azure Identity, consulte:

**Bibliotecas cliente de Azure Identity**

| .NET | Python | Java | JavaScript |
|--|--|--|--|
|[SDK .NET de Azure Identity](/dotnet/api/overview/azure/identity-readme)|[SDK Python de Azure Identity](/python/api/overview/azure/identity-readme)|[SDK Java de Azure Identity](/java/api/overview/azure/identity-readme)|[SDK JavaScript de Azure Identity](/javascript/api/overview/azure/identity-readme)|     

>[!Note]
> [Biblioteca de autenticación de aplicaciones](/dotnet/api/overview/azure/service-to-service-authentication), que se recomendó para el SDK de .NET de Key Vault, versión 3, que actualmente está en desuso. Siga las [instrucciones de migración de AppAuthentication a Azure.Identity](/dotnet/api/overview/azure/app-auth-migration) para migrar al SDK de .NET de Key Vault, versión 4.

Para ver tutoriales sobre la autenticación en Key Vault en las aplicaciones, consulte:
- [Autenticación en Key Vault en una aplicación hospedada en la VM de .NET](./tutorial-net-virtual-machine.md)
- [Autenticación en Key Vault en una aplicación hospedada en la VM de Python](./tutorial-python-virtual-machine.md)
- [Autenticación en Key Vault con App Service](./tutorial-net-create-vault-azure-web-app.md)

## <a name="manage-keys-certificates-and-secrets"></a>Administración de claves, certificados y secretos

El acceso a las claves, los secretos y los certificados se controla mediante el plano de datos. El control de acceso del plano de datos se puede realizar mediante las directivas de acceso de almacén local o Azure RBAC.

**API y SDK de claves**

| Azure CLI | PowerShell | API DE REST | Resource Manager | .NET | Python | Java | JavaScript |  
|--|--|--|--|--|--|--|--|
|[Referencia](/cli/azure/keyvault/key)<br>[Guía de inicio rápido](../keys/quick-create-cli.md)|[Referencia](/powershell/module/az.keyvault/)<br>[Guía de inicio rápido](../keys/quick-create-powershell.md)|[Referencia](/rest/api/keyvault/#key-operations)|[Referencia](/azure/templates/microsoft.keyvault/vaults/keys)<br>[Guía de inicio rápido](../keys/quick-create-template.md)|[Referencia](/dotnet/api/azure.security.keyvault.keys)<br>[Guía de inicio rápido](../keys/quick-create-net.md)|[Referencia](/python/api/azure-mgmt-keyvault/azure.mgmt.keyvault)<br>[Guía de inicio rápido](../keys/quick-create-python.md)|[Referencia](https://azuresdkdocs.blob.core.windows.net/$web/java/azure-security-keyvault-keys/4.2.0/index.html)<br>[Guía de inicio rápido](../keys/quick-create-java.md)|[Referencia](/javascript/api/@azure/keyvault-keys/)<br>[Guía de inicio rápido](../keys/quick-create-node.md)|

**SDK y API de certificados**

| Azure CLI | PowerShell | API DE REST | Resource Manager | .NET | Python | Java | JavaScript |  
|--|--|--|--|--|--|--|--|
|[Referencia](/cli/azure/keyvault/certificate)<br>[Guía de inicio rápido](../certificates/quick-create-cli.md)|[Referencia](/powershell/module/az.keyvault)<br>[Guía de inicio rápido](../certificates/quick-create-powershell.md)|[Referencia](/rest/api/keyvault/#certificate-operations)|N/D|[Referencia](/dotnet/api/azure.security.keyvault.certificates)<br>[Guía de inicio rápido](../certificates/quick-create-net.md)|[Referencia](/python/api/overview/azure/keyvault-certificates-readme)<br>[Guía de inicio rápido](../certificates/quick-create-python.md)|[Referencia](https://azuresdkdocs.blob.core.windows.net/$web/java/azure-security-keyvault-certificates/4.1.0/index.html)<br>[Guía de inicio rápido](../certificates/quick-create-java.md)|[Referencia](/javascript/api/@azure/keyvault-certificates/)<br>[Guía de inicio rápido](../certificates/quick-create-node.md)|

**API y SDK de secretos**

| Azure CLI | PowerShell | API DE REST | Resource Manager | .NET | Python | Java | JavaScript |  
|--|--|--|--|--|--|--|--|
|[Referencia](/cli/azure/keyvault/secret)<br>[Guía de inicio rápido](../secrets/quick-create-cli.md)|[Referencia](/powershell/module/az.keyvault/)<br>[Guía de inicio rápido](../secrets/quick-create-powershell.md)|[Referencia](/rest/api/keyvault/#secret-operations)|[Referencia](/azure/templates/microsoft.keyvault/vaults/secrets)<br>[Guía de inicio rápido](../secrets/quick-create-template.md)|[Referencia](/dotnet/api/azure.security.keyvault.secrets)<br>[Guía de inicio rápido](../secrets/quick-create-net.md)|[Referencia](/python/api/overview/azure/keyvault-secrets-readme)<br>[Guía de inicio rápido](../secrets/quick-create-python.md)|[Referencia](https://azuresdkdocs.blob.core.windows.net/$web/java/azure-security-keyvault-secrets/4.2.0/index.html)<br>[Guía de inicio rápido](../secrets/quick-create-java.md)|[Referencia](/javascript/api/@azure/keyvault-secrets/)<br>[Guía de inicio rápido](../secrets/quick-create-node.md)|

Consulte las [Bibliotecas de cliente](client-libraries.md) para ver los paquetes de instalación y el código fuente.

Para obtener más información sobre el plano de datos de Key Vault, consulte [Características de seguridad de Azure Key Vault](security-features.md).

### <a name="code-examples"></a>Ejemplos de código

Para obtener ejemplos completos de cómo usar Key Vault con las aplicaciones, vea:

- [Ejemplos de código de Azure Key Vault](https://azure.microsoft.com/resources/samples/?service=key-vault). Ejemplos de código para Azure Key Vault. 

## <a name="how-tos"></a>Procedimientos

Los artículos y escenarios siguientes proporcionan instrucciones específicas de tarea sobre cómo trabajar con Azure Key Vault:

- [Acceso a Key Vault detrás de un firewall](access-behind-firewall.md). Para acceder a un almacén de claves, es preciso que la aplicación cliente de Key Vault pueda acceder a varios puntos de conexión para diversas funcionalidades.
- Cómo implementar certificados en VM desde Key Vault ([Windows](../../virtual-machines/extensions/key-vault-windows.md), [Linux](../../virtual-machines/extensions/key-vault-linux.md)): una aplicación de nube que se ejecute en una VM de Azure necesita un certificado. Sepa cómo añadirlo a la máquina virtual hoy mismo.
- [Implementar un certificado de Azure Web App mediante Key Vault](../../app-service/configure-ssl-certificate.md#import-a-certificate-from-key-vault).
- Asignación de una directiva de acceso ([CLI](assign-access-policy-cli.md) | [PowerShell](assign-access-policy-powershell.md) | [Azure Portal](assign-access-policy-portal.md)). 
- [Uso de la eliminación temporal de Key Vault con la CLI](./key-vault-recovery.md) le guiará por el uso y ciclo de vida de un almacén de claves y de varios objetos de almacén de claves con la eliminación temporal habilitada.
- [Paso de valores seguros (como contraseñas) durante la implementación](../../azure-resource-manager/templates/key-vault-parameter.md). Si necesita pasar un valor seguro (como una contraseña) como un parámetro durante la implementación, puede almacenar ese valor como un secreto en Azure Key Vault y hacer referencia al valor en otras plantillas de Resource Manager.

## <a name="integrated-with-key-vault"></a>Integración con Key Vault

En estos artículos se describen otros escenarios y servicios que usan Key Vault o se integran en este servicio.

- El [cifrado en reposo](../../security/fundamentals/encryption-atrest.md) permite la codificación (cifrado) de datos cuando se conserva. Las claves de cifrado de datos se cifran a menudo con la clave de cifrado de claves en Azure Key Vault para limitar aún más el acceso.
- [Azure Information Protection](/azure/information-protection/plan-implement-tenant-key) le permite administrar su propia clave de inquilino. Por ejemplo, en lugar de que Microsoft administre su clave de inquilino (la opción predeterminada), puede administrar su propia clave de inquilino para cumplir con las normas específicas que se aplican a su organización. La administración de su propia clave de inquilino también se conoce como "aportar su propia clave", o BYOK.
- El [servicio Azure Private Link](private-link-service.md) le permite acceder a los servicios de Azure (por ejemplo, Azure Key Vault, Azure Storage y Azure Cosmos DB) y a los servicios de asociados o clientes hospedados de Azure mediante un punto de conexión privado de la red virtual.
- La integración de Key Vault con [Event Grid](../../event-grid/event-schema-key-vault.md) permite a los usuarios recibir una notificación cuando cambia el estado de un secreto almacenado en el almacén de claves. Puede distribuir la nueva versión de los secretos en las aplicaciones, o cambiar los secretos que estén a punto de expirar para evitar interrupciones.
- Puede proteger los secretos de [Azure DevOps](/azure/devops/pipelines/release/azure-key-vault) de accesos no deseados en Key Vault.
- [Use el secreto almacenado en Key Vault en DataBricks para conectarse a Azure Storage](./integrate-databricks-blob-storage.md)
- Configure y ejecute el proveedor de Azure Key Vault correspondiente al [controlador Secrets Store CSI](./key-vault-integrate-kubernetes.md) en Kubernetes

## <a name="key-vault-overviews-and-concepts"></a>Conceptos y datos globales de Key Vault

- [Comportamiento de eliminación temporal de Key Vault](soft-delete-overview.md) describe una característica que permite la recuperación de objetos eliminados tanto si dicha eliminación ha sido accidental como intencionada.
- [Limitación del cliente de Key Vault](overview-throttling.md) proporciona orientación sobre los conceptos básicos de la limitación y ofrece un enfoque para la aplicación.
- [Espacios de seguridad de Key Vault](overview-security-worlds.md) describe las relaciones entre regiones y zonas de seguridad.

## <a name="social"></a>Redes sociales

- [Blog de Key Vault](/archive/blogs/kv/)
- [Foro sobre Key Vault](https://aka.ms/kvforum)