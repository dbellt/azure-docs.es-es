---
title: Autenticación en Azure Key Vault
description: Aprenda a autenticarse en Azure Key Vault
author: msmbaldwin
ms.author: mbaldwin
ms.date: 03/31/2021
ms.service: key-vault
ms.subservice: general
ms.topic: conceptual
ms.openlocfilehash: 1fa4f4b4b9b56b5acc3a3d3d9e75177889a4a8af
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110479242"
---
# <a name="authentication-in-azure-key-vault"></a>Autenticación de Azure Key Vault

La autenticación con Key Vault funciona junto con [Azure Active Directory (Azure AD)](../../active-directory/fundamentals/active-directory-whatis.md), que es responsable de autenticar la identidad de cualquier **entidad de seguridad**.

Una entidad de seguridad es un objeto que representa un usuario, un grupo o una entidad de servicio que solicita acceso a recursos de Azure. Azure asigna un **identificador de objeto** único a cada entidad de seguridad.

* Una entidad de seguridad de **usuario** identifica un individuo que tiene un perfil en Azure Active Directory.

* Una entidad de seguridad de **grupo** identifica un conjunto de usuarios creados en Azure Active Directory. Los roles o permisos asignados al grupo se conceden a todos los usuarios dentro del grupo.

* Una **entidad de servicio** es un tipo de entidad de seguridad que identifica una aplicación o un servicio, es decir, un fragmento de código en lugar de un usuario o grupo. El identificador de objeto de una entidad de servicio se conoce como su **Id. de cliente** y actúa como su nombre de usuario. El **secreto de cliente** de la entidad de servicio actúa como su contraseña.

En el caso de las aplicaciones, hay dos maneras de obtener una entidad de servicio:

* Recomendado: habilite una **identidad administrada** asignada por el sistema para la aplicación.

    Con la identidad administrada, Azure administra internamente la entidad de servicio de la aplicación y autentica la aplicación automáticamente con otros servicios de Azure. La identidad administrada está disponible para las aplicaciones implementadas en diversos servicios.

    Para obtener más información, consulte [Introducción a las identidades administradas](../../active-directory/managed-identities-azure-resources/overview.md). Consulte también [Servicios que admiten identidades administradas para recursos de Azure](../../active-directory/managed-identities-azure-resources/services-support-managed-identities.md), que contiene vínculos a artículos que describen cómo habilitar identidades administradas para servicios específicos (como App Service, Azure Functions, Virtual Machines, etc.).

* Si no puede usar la identidad administrada, en su lugar **registre** la aplicación con el inquilino de Azure D, como se describe en [Inicio rápido: Registro de una aplicación en la plataforma de identidad de Microsoft](../../active-directory/develop/quickstart-register-app.md). El registro también crea un segundo objeto de aplicación que identifica la aplicación en todos los inquilinos.

## <a name="configure-the-key-vault-firewall"></a>Configuración del firewall de Key Vault

De manera predeterminada, Key Vault permite el acceso a los recursos a través de direcciones IP públicas. Para mayor seguridad, también puede restringir el acceso a intervalos de direcciones IP, puntos de conexión de servicio, redes virtuales o puntos de conexión privados específicos.

Para más información, consulte [Acceso a Azure Key Vault desde detrás de un firewall](./access-behind-firewall.md).

## <a name="the-key-vault-request-operation-flow-with-authentication"></a>Flujo de operaciones de solicitud de Key Vault con autenticación

La autenticación de Key Vault se produce como parte de cada operación de solicitud en Key Vault. Una vez recuperado el token, se puede reutilizar en las llamadas posteriores. Ejemplo del flujo de autenticación:

1. Un token solicita autenticarse con Azure AD, por ejemplo:
    * Un recurso de Azure, pongamos, una máquina virtual o una aplicación de App Service con una identidad administrada se pone en contacto con el punto de conexión de REST para obtener un token de acceso.
    * Un usuario que inicia sesión en Azure Portal mediante un nombre de usuario y una contraseña.

1. Si la autenticación con Azure AD se realiza correctamente, se le concede un token de OAuth a la entidad de seguridad.

1. Una llamada a la API REST de Key Vault mediante el punto de conexión (URI) de Key Vault.

1. El firewall de Key Vault comprueba los siguientes criterios. Si se cumple algún criterio, se permite la llamada. En caso contrario, se bloquea la llamada y se devuelve una respuesta prohibida.

    * El firewall está deshabilitado y el punto de conexión público de Key Vault es accesible desde la red pública de Internet.
    * El autor de la llamada es un [servicio de confianza de Key Vault](./overview-vnet-service-endpoints.md#trusted-services), lo que le permite omitir el firewall.
    * El autor de la llamada aparece listado en el firewall por su dirección IP, red virtual o punto de conexión de servicio.
    * El autor de la llamada puede conectar con Key Vault mediante una conexión de vínculo privado configurada.    

1. Si el firewall permite la llamada, Key Vault llama a Azure AD para validar el token de acceso de la entidad de seguridad.

1. Key Vault comprueba si la entidad de seguridad tiene el permiso necesario para la operación solicitada. En caso contrario, Key Vault devuelve una respuesta prohibida.

1. Key Vault realiza la operación solicitada y devuelve el resultado.

En el diagrama siguiente se ilustra el proceso de una aplicación que llama a una API "Get secret" de Key Vault:

![Flujo de autenticación de Azure Key Vault](../media/authentication/authentication-flow.png)

> [!NOTE]
> Los clientes del SDK de Key Vault para secretos, certificados y claves realizan una llamada adicional a Key Vault sin token de acceso, lo que da como resultado una respuesta 401 para recuperar la información del inquilino. Para más información, consulte [Autenticación, solicitudes y respuestas](authentication-requests-and-responses.md).

## <a name="authentication-to-key-vault-in-application-code"></a>Autenticación en Key Vault en el código de la aplicación

El SDK de Key Vault usa la biblioteca cliente de Azure Identity, que permite la autenticación perfecta en Key Vault entre entornos con el mismo código.

**Bibliotecas cliente de Azure Identity**

| .NET | Python | Java | JavaScript |
|--|--|--|--|
|[SDK .NET de Azure Identity](/dotnet/api/overview/azure/identity-readme)|[SDK Python de Azure Identity](/python/api/overview/azure/identity-readme)|[SDK Java de Azure Identity](/java/api/overview/azure/identity-readme)|[SDK JavaScript de Azure Identity](/javascript/api/overview/azure/identity-readme)|   

Para más información sobre los procedimientos recomendados y ver ejemplos de desarrolladores, consulte [Autenticación en Key Vault en el código](developers-guide.md#authenticate-to-key-vault-in-code).

## <a name="next-steps"></a>Pasos siguientes

- [Guía del desarrollador de Key Vault](developers-guide.md)
- [Asignación de una directiva de acceso de Key Vault mediante Azure Portal](assign-access-policy-portal.md)
- [Asignación del rol de Azure RBAC a Key Vault](rbac-guide.md)
- [Solución de problemas de las directivas de acceso de Azure Key Vault](troubleshooting-access-issues.md)
- [Códigos de error de la API REST de Azure Key Vault](rest-error-codes.md)

- [¿Qué es el control de acceso basado en rol de Azure (RBAC)?](../../role-based-access-control/overview.md)
