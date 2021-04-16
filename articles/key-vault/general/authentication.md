---
title: Autenticación en Azure Key Vault
description: Aprenda a autenticarse en Azure Key Vault
author: ShaneBala-keyvault
ms.author: sudbalas
ms.date: 08/27/2020
ms.service: key-vault
ms.subservice: general
ms.topic: how-to
ms.openlocfilehash: 8a8fe4ed0c24d2ccda5fb844005a33a93e85a169
ms.sourcegitcommit: dddd1596fa368f68861856849fbbbb9ea55cb4c7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/13/2021
ms.locfileid: "107365525"
---
# <a name="authenticate-to-azure-key-vault"></a>Autenticación en Azure Key Vault

Azure Key Vault le permite almacenar secretos y controlar su distribución en un repositorio en la nube centralizado y seguro, lo que elimina la necesidad de almacenar credenciales en las aplicaciones. Las aplicaciones solo necesitan autenticarse con Key Vault en tiempo de ejecución para tener acceso a esos secretos.

## <a name="app-identity-and-security-principals"></a>Identidad de la aplicación y entidades de seguridad

La autenticación con Key Vault funciona junto con [Azure Active Directory (Azure AD)](../../active-directory/fundamentals/active-directory-whatis.md), que es responsable de autenticar la identidad de cualquier **entidad de seguridad**.

Una entidad de seguridad es un objeto que representa un usuario, un grupo o una entidad de servicio que solicita acceso a recursos de Azure. Azure asigna un **identificador de objeto** único a cada entidad de seguridad.

* Una entidad de seguridad de **usuario** identifica un individuo que tiene un perfil en Azure Active Directory.

* Una entidad de seguridad de **grupo** identifica un conjunto de usuarios creados en Azure Active Directory. Los roles o permisos asignados al grupo se conceden a todos los usuarios dentro del grupo.

* Un **entidad de servicio** es un tipo de entidad de seguridad que identifica una aplicación o un servicio, es decir, un fragmento de código en lugar de un usuario o grupo. El identificador de objeto de una entidad de servicio se conoce como su **Id. de cliente** y actúa como su nombre de usuario. El **secreto de cliente** de la entidad de servicio actúa como su contraseña.

En el caso de las aplicaciones, hay dos maneras de obtener una entidad de servicio:

* Recomendado: habilite una **identidad administrada** asignada por el sistema para la aplicación.

    Con la identidad administrada, Azure administra internamente la entidad de servicio de la aplicación y autentica la aplicación automáticamente con otros servicios de Azure. La identidad administrada está disponible para las aplicaciones implementadas en diversos servicios.

    Para obtener más información, consulte [Introducción a las identidades administradas](../../active-directory/managed-identities-azure-resources/overview.md). Consulte también [Servicios que admiten identidades administradas para recursos de Azure](../../active-directory/managed-identities-azure-resources/services-support-managed-identities.md), que contiene vínculos a artículos que describen cómo habilitar identidades administradas para servicios específicos (como App Service, Azure Functions, Virtual Machines, etc.).

* Si no puede usar la identidad administrada, en su lugar **registre** la aplicación con el inquilino de Azure D, como se describe en [Inicio rápido: Registro de una aplicación en la plataforma de identidad de Microsoft](../../active-directory/develop/quickstart-register-app.md). El registro también crea un segundo objeto de aplicación que identifica la aplicación en todos los inquilinos.

## <a name="authorize-a-security-principal-to-access-key-vault"></a>Autorización de una entidad de seguridad para tener acceso a Key Vault

Key Vault funciona con dos niveles de autorización independientes:

- Las **directivas de acceso** controlan si un usuario, grupo o entidad de servicio está autorizado para tener acceso a secretos, claves y certificados *dentro de* un recurso de Key Vault existente (a veces denominado "plano de datos"). Normalmente, las directivas de acceso se conceden a usuarios, grupos y aplicaciones.

    Para asignar directivas de acceso, consulte los siguientes artículos:

    - [Azure Portal](assign-access-policy-portal.md)
    - [CLI de Azure](assign-access-policy-cli.md)
    - [Azure PowerShell](assign-access-policy-portal.md)

- Los **permisos de rol** controlan si un usuario, grupo o entidad de servicio está autorizado para crear, eliminar y administrar un recurso de Key Vault (lo que a veces se denomina operaciones del "plano de administración"). Estos roles se conceden normalmente solo a los administradores.
 
    Para asignar y administrar roles, consulte los siguientes artículos:

    - [Azure Portal](../../role-based-access-control/role-assignments-portal.md)
    - [CLI de Azure](../../role-based-access-control/role-assignments-cli.md)
    - [Azure PowerShell](../../role-based-access-control/role-assignments-powershell.md)

    Para obtener información general sobre roles, consulte [¿Qué es el control de acceso basado en rol de Azure (RBAC)?](../../role-based-access-control/overview.md).


> [!IMPORTANT]
> Para mayor seguridad, siempre debe seguir la entidad de seguridad de privilegios mínimos y conceder solo las directivas de acceso y los roles más específicos que sean necesarios. 
    
## <a name="configure-the-key-vault-firewall"></a>Configuración del firewall de Key Vault

De manera predeterminada, Key Vault permite el acceso a los recursos a través de direcciones IP públicas. Para mayor seguridad, también puede restringir el acceso a intervalos de direcciones IP, puntos de conexión de servicio, redes virtuales o puntos de conexión privados específicos.

Para más información, consulte [Acceso a Azure Key Vault desde detrás de un firewall](./access-behind-firewall.md).


## <a name="the-key-vault-authentication-flow"></a>Flujo de autenticación de Key Vault

1. Una entidad de servicio solicita que se autentique con Azure AD, por ejemplo:
    * Un usuario que inicia sesión en Azure Portal mediante un nombre de usuario y una contraseña.
    * Una aplicación que invoca una API REST de Azure,que debe presentar un id. de cliente y un secreto o un certificado de cliente.
    * Un recurso de Azure, como una máquina virtual con una identidad administrada, que contacta con el punto de conexión REST de [Instance Metadata Service de Azure (IMDS)](../../virtual-machines/windows/instance-metadata-service.md) para obtener un token de acceso.

1. Si la autenticación con Azure AD se realiza correctamente, se le concede un token de OAuth a la entidad de servicio.

1. La entidad de servicio realiza una llamada a la API REST de Key Vault a través del punto de conexión (URI) de Key Vault.

1. El firewall de Key Vault comprueba los siguientes criterios. Si se cumple algún criterio, se permite la llamada. En caso contrario, se bloquea la llamada y se devuelve una respuesta prohibida.

    * El firewall está deshabilitado y el punto de conexión público de Key Vault es accesible desde la red pública de Internet.
    * El autor de la llamada es un [servicio de confianza de Key Vault](./overview-vnet-service-endpoints.md#trusted-services), lo que le permite omitir el firewall.
    * El autor de la llamada aparece listado en el firewall por su dirección IP, red virtual o punto de conexión de servicio.
    * El autor de la llamada puede conectar con Key Vault mediante una conexión de vínculo privado configurada.    

1. Si el firewall permite la llamada, Key Vault llama a Azure AD para validar el token de acceso de la entidad de servicio.

1. Key Vault comprueba si la entidad de servicio tiene la directiva de acceso necesaria para la operación solicitada. En caso contrario, Key Vault devuelve una respuesta prohibida.

1. Key Vault realiza la operación solicitada y devuelve el resultado.

En el diagrama siguiente se ilustra el proceso de una aplicación que llama a una API "Get secret" de Key Vault:

![Flujo de autenticación de Azure Key Vault](../media/authentication/authentication-flow.png)

> [!NOTE]
> Los clientes del SDK de Key Vault para secretos, certificados y claves realizan una llamada adicional a Key Vault sin token de acceso, lo que da como resultado una respuesta 401 para recuperar la información del inquilino. Para más información, consulte [Autenticación, solicitudes y respuestas](authentication-requests-and-responses.md).

## <a name="code-examples"></a>Ejemplos de código

En la tabla siguiente se incluyen vínculos a artículos diferentes que muestran cómo trabajar con Key Vault en el código de aplicación mediante las bibliotecas de Azure SDK para el lenguaje en cuestión. Otras interfaces, como la CLI de Azure y Azure Portal, se incluyen para mayor comodidad.

| Secretos de Key Vault | Claves de Key Vault | Certificados en Key Vault |
|  --- | --- | --- |
| [Python](../secrets/quick-create-python.md) | [Python](../keys/quick-create-python.md) | [Python](../certificates/quick-create-python.md) | 
| [.NET](../secrets/quick-create-net.md) | [.NET](../keys/quick-create-net.md) | [.NET](../certificates/quick-create-net.md) |
| [Java](../secrets/quick-create-java.md) | [Java](../keys/quick-create-java.md) | [Java](../certificates/quick-create-java.md) |
| [JavaScript](../secrets/quick-create-node.md) | [JavaScript](../keys/quick-create-node.md) | [JavaScript](../certificates/quick-create-node.md) | 
| [Azure Portal](../secrets/quick-create-portal.md) | [Azure Portal](../keys/quick-create-portal.md) | [Azure Portal](../certificates/quick-create-portal.md) |
| [CLI de Azure](../secrets/quick-create-cli.md) | [CLI de Azure](../keys/quick-create-cli.md) | [CLI de Azure](../certificates/quick-create-cli.md) |
| [Azure PowerShell](../secrets/quick-create-powershell.md) | [Azure PowerShell](../keys/quick-create-powershell.md) | [Azure PowerShell](../certificates/quick-create-powershell.md) |
| [Plantilla ARM](../secrets/quick-create-net.md) | -- | -- |

## <a name="next-steps"></a>Pasos siguientes

- [Solución de problemas de las directivas de acceso de Azure Key Vault](troubleshooting-access-issues.md)
- [Códigos de error de la API REST de Azure Key Vault](rest-error-codes.md)
- [Guía del desarrollador de Key Vault](developers-guide.md)
- [¿Qué es el control de acceso basado en rol de Azure (RBAC)?](../../role-based-access-control/overview.md)
