---
title: Cifrado de Azure Data Factory con claves administradas por el cliente
description: Mejora de la seguridad de Data Factory con Bring Your Own Key (BYOK)
author: chez-charlie
ms.service: data-factory
ms.topic: quickstart
ms.date: 05/08/2020
ms.author: chez
ms.reviewer: mariozi
ms.openlocfilehash: a18d06e3a0324889a4cb9936fb339fd9d8f9b816
ms.sourcegitcommit: 3f684a803cd0ccd6f0fb1b87744644a45ace750d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/02/2021
ms.locfileid: "106222710"
---
# <a name="encrypt-azure-data-factory-with-customer-managed-keys"></a>Cifrado de Azure Data Factory con claves administradas por el cliente

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]

Azure Data Factory cifra los datos en reposo, incluidas las definiciones de entidades y los datos almacenados en caché mientras las ejecuciones están en curso. De forma predeterminada, los datos se cifran con una clave administrada por Microsoft que se genera aleatoriamente y que se asigna de forma única a la factoría de datos. Para reforzar aún más las medidas de seguridad, ahora puede habilitar Bring Your Own Key (BYOK) con la característica de claves administradas por el cliente de Azure Data Factory. Cuando se especifica una clave administrada por el cliente, Data Factory usa __tanto__ la clave del sistema de la factoría como la CMK para cifrar los datos del cliente. Si falta alguna de ellas, se deniega el acceso a los datos y a la factoría.

Azure Key Vault es necesario para almacenar las claves administradas por el cliente. Puede crear sus propias claves y almacenarlas en un almacén de claves, o puede usar las API de Azure Key Vault para generarlas. Key Vault y Azure Data Factory deben estar en el mismo inquilino de Azure Active Directory (Azure AD) y en la misma región, pero pueden estar en distintas suscripciones. Para obtener más información sobre Azure Key Vault, consulte [¿Qué es Azure Key Vault?](../key-vault/general/overview.md)

## <a name="about-customer-managed-keys"></a>Acerca de las claves administradas por el cliente

En el siguiente diagrama se muestra cómo Data Factory usa Azure Active Directory y Azure Key Vault para realizar solicitudes mediante la clave administrada por el cliente:

  :::image type="content" source="media/enable-customer-managed-key/encryption-customer-managed-keys-diagram.png" alt-text="Diagrama que muestra cómo funcionan las claves administradas por el cliente en Azure Data Factory.":::

En la lista siguiente se explican los pasos numerados del diagrama:

1. Un administrador de Azure Key Vault concede permisos a las claves de cifrado para la identidad administrada que está asociada a la instancia de Data Factory.
1. Un administrador de Data Factory habilita la característica de claves administradas por el cliente en la factoría.
1. Data Factory usa la identidad administrada que está asociada a la factoría para autenticar el acceso a Azure Key Vault mediante Azure Active Directory.
1. Data Factory encapsula la clave de cifrado de la factoría con la clave de cliente de Azure Key Vault.
1. En operaciones de lectura y escritura, Data Factory envía solicitudes a Azure Key Vault para desencapsular la clave de cifrado de la cuenta con el fin de realizar operaciones de cifrado y descifrado.

Hay dos maneras de incorporar el cifrado de claves administradas por el cliente a las factorías de datos. Una es en el momento de creación de la fábrica en Azure Portal y el otro es tras la creación de la fábrica, en la interfaz de usuario de Data Factory.

## <a name="prerequisites---configure-azure-key-vault-and-generate-keys"></a>Requisitos previos: configuración de Azure Key Vault y generación de las claves

### <a name="enable-soft-delete-and-do-not-purge-on-azure-key-vault"></a>Habilitación de Eliminación temporal y No purgar en Azure Key Vault

Para usar las claves administradas por el cliente con Data Factory, es necesario establecer dos propiedades en Key Vault: __Eliminación temporal__ y __No purgar__. Estas propiedades se pueden habilitar mediante PowerShell o la CLI de Azure en un almacén de claves nuevo o existente. Para obtener información sobre cómo habilitar estas propiedades en un almacén de claves existente, consulte [Administración de la recuperación de Azure Key Vault con eliminación temporal y protección contra purga](../key-vault/general/key-vault-recovery.md).

Si va a crear una instancia de Azure Key Vault a través de Azure Portal, las propiedades __Eliminación temporal__ y __No purgan__ se pueden habilitar de la siguiente manera:

  :::image type="content" source="media/enable-customer-managed-key/01-enable-purge-protection.png" alt-text="Captura de pantalla que muestra cómo habilitar la eliminación temporal y la protección contra purga tras la creación de una instancia de Key Vault.":::

### <a name="grant-data-factory-access-to-azure-key-vault"></a>Concesión a Data Factory de acceso a Azure Key Vault

Asegúrese de que Azure Key Vault y Azure Data Factory estén en el mismo inquilino de Azure Active Directory (Azure AD) y en la _misma región_. Desde el control de acceso de Azure Key Vault, conceda los permisos siguientes a la factoría de datos: _Obtener_, _Desencapsular clave_ y _Encapsular clave_. Estos permisos son necesarios para habilitar las claves administradas por el cliente en Data Factory.

* Si desea agregar el cifrado de claves administradas por el cliente [después de crear la factoría en la interfaz de usuario de Data Factory](#post-factory-creation-in-data-factory-ui), asegúrese de que Managed Service Identity (MSI) tiene los tres permisos para Key Vault.
* Si desea agregar el cifrado de claves administradas por el cliente [en el momento de crear la factoría en Azure Portal](#during-factory-creation-in-azure-portal), asegúrese de que la identidad administrada asignada por el usuario (UA-MI) tiene los tres permisos para Key Vault.

  :::image type="content" source="media/enable-customer-managed-key/02-access-policy-factory-managed-identities.png" alt-text="Captura de pantalla que muestra cómo habilitar el acceso de Data Factory a Key Vault.":::

### <a name="generate-or-upload-customer-managed-key-to-azure-key-vault"></a>Generación o carga de la clave administrada por el cliente en Azure Key Vault

Puede crear sus propias claves y almacenarlas en un almacén de claves. O bien, usar las API de Azure Key Vault para generar las claves. Solo se admiten las claves RSA de 2048 bits con cifrado de Data Factory. Para más información, consulte el artículo [About keys, secrets, and certificates](../key-vault/general/about-keys-secrets-certificates.md) (Claves, secretos y certificados).

  :::image type="content" source="media/enable-customer-managed-key/03-create-key.png" alt-text="Captura de pantalla que muestra cómo generar una clave administrada por el cliente.":::

## <a name="enable-customer-managed-keys"></a>Habilitar claves administradas del cliente

### <a name="post-factory-creation-in-data-factory-ui"></a>Tras la creación de la factoría en la interfaz de usuario de Data Factory

En esta sección se describe el proceso para agregar el cifrado de claves administradas por el cliente a la interfaz de usuario de Data Factory _tras_ crear la factoría.

> [!NOTE]
> Una clave administrada por el cliente solo se puede configurar en una factoría de datos vacía. La factoría de datos no puede contener recursos, como servicios vinculados, canalizaciones y flujos de datos. Se recomienda habilitar la clave administrada por el cliente justo después de la creación de la factoría.

> [!IMPORTANT]
> Este enfoque no funciona con factorías habilitadas para redes virtuales administradas. Considere la [ruta alternativa](#during-factory-creation-in-azure-portal) si desea cifrar esas factorías.

1. Asegúrese de que la instancia de Managed Service Identity (MSI) de la factoría de datos tiene los permisos _Obtener_, _Desencapsular clave_ y _Encapsular clave_ en Key Vault.

1. Asegúrese de que la instancia de Data Factory esté vacía. La factoría de datos no puede contener recursos como servicios vinculados, canalizaciones y flujos de datos. Por ahora, la implementación de una clave administrada por el cliente en una factoría que no esté vacía producirá un error.

1. Para buscar el URI de la clave en Azure Portal, vaya a Azure Key Vault y seleccione la opción de configuración Claves. Seleccione la clave deseada y haga clic para ver sus versiones. Seleccione una de ellas para ver su configuración.

1. Copie el valor del campo Identificador de clave, que proporciona el URI. :::image type="content" source="media/enable-customer-managed-key/04-get-key-identifier.png" alt-text="Captura de pantalla de obtención del URI de Key Vault.":::

1. Inicie el portal de Azure Data Factory y, mediante la barra de navegación de la izquierda, salte al Portal de administración de Data Factory.

1. Haga clic en el icono de la __clave administrada por el cliente__. :::image type="content" source="media/enable-customer-managed-key/05-customer-managed-key-configuration.png" alt-text="Captura de pantalla que muestra cómo habilitar la clave administrada por el cliente en la interfaz de usuario de Data Factory.":::

1. Escriba el URI de la clave administrada por el cliente que copió anteriormente.

1. Haga clic en __Guardar__; el cifrado de claves administradas por el cliente se habilita para Data Factory.

### <a name="during-factory-creation-in-azure-portal"></a>Durante la creación de la factoría en Azure Portal

En esta sección se explican los pasos para agregar el cifrado de claves administradas por el cliente en Azure Portal _durante_ la implementación de la factoría.

Para cifrar la factoría, Data Factory debe recuperar primero la clave administrada por el cliente de Key Vault. Dado que la implementación de la factoría todavía está en curso, Managed Service Identity (MSI) aún no está disponible para autenticarse mediante Key Vault. Por lo tanto, para usar este enfoque, el cliente debe asignar una identidad administrada asignada por el usuario (UA-MI) a la factoría de datos. Asumiremos los roles definidos en la identidad UA-MI y nos autenticaremos mediante Key Vault.

Para obtener más información sobre la identidad administrada asignada por el usuario, consulte [Tipos de identidad administrados](../active-directory/managed-identities-azure-resources/overview.md#managed-identity-types) y [Asignación de un rol a una identidad administrada asignada por el usuario](../active-directory/managed-identities-azure-resources/how-to-manage-ua-identity-portal.md).

1. Asegúrese de que la identidad administrada asignada por el usuario (UA-MI) tiene los permisos _Obtener_, _Desencapsular clave_ y _Encapsular clave_ en Key Vault.

1. En la pestaña __Opciones avanzadas__, active la casilla _Enable encryption using a customer managed key_(Habilitar el cifrado mediante una clave administrada por el cliente)
  :::image type="content" source="media/enable-customer-managed-key/06-user-assigned-managed-identity.png" alt-text="Captura de pantalla que muestra la pestaña de opciones avanzadas para la experiencia de creación de una factoría de datos en Azure Portal.":::

1. Proporcione la URL de la clave administrada por el cliente almacenada en Key Vault.

1. Seleccione una identidad administrada asignada por el usuario adecuada para autenticarse mediante Key Vault.

1. Siga adelante con la implementación de la factoría.

## <a name="update-key-version"></a>Actualización de la versión de la clave

Al crear una versión de una clave, actualice la factoría de datos para que use esta nueva versión. Siga los mismos pasos que se describen en la sección sobre la [interfaz de usuario de Data Factory](#post-factory-creation-in-data-factory-ui), que incluye:

1. Busque el URI de la nueva versión de la clave en el portal de Azure Key Vault.

1. Vaya a la opción __Clave administrada por el cliente__.

1. Reemplace y pegue el URI de la nueva clave.

1. Haga clic en __Guardar__; Data Factory ahora se cifrará con la nueva versión de la clave.

## <a name="use-a-different-key"></a>Uso de una clave distinta

Para cambiar la clave usada para el cifrado de Data Factory, debe actualizar manualmente la configuración en Data Factory. Siga los mismos pasos que se describen en la sección sobre la [interfaz de usuario de Data Factory](#post-factory-creation-in-data-factory-ui), que incluye:

1. Busque el URI de la nueva clave en el portal de Azure Key Vault.

1. Vaya a la opción __Clave administrada por el cliente__.

1. Reemplace y pegue el URI de la nueva clave.

1. Haga clic en __Guardar__; Data Factory ahora se cifrará con la nueva clave.

## <a name="disable-customer-managed-keys"></a>Deshabilitación de las claves administradas por el cliente

De manera intencionada, una vez habilitada la característica de claves administradas por el cliente, no se puede quitar el paso de seguridad adicional. Para cifrar la factoría y los datos, siempre se esperará una clave proporcionada por el cliente.

## <a name="customer-managed-key-and-continuous-integration-and-continuous-deployment"></a>Clave administrada por el cliente para la integración continua y entrega continua

De forma predeterminada, la configuración de CMK no se incluye en la plantilla de Azure Resource Manager de fábrica. Para incluir la configuración del cifrado de claves administradas por el cliente en la plantilla de Resource Manager para la integración continua y entrega continua (CI/CD):

1. Asegurarse de que la fábrica está en modo Git
1. Vaya al portal de administración, a la sección Clave administrada por el cliente
1. Active la opción _Include in ARM template_ (Incluir en la plantilla de Resource Manager).

  :::image type="content" source="media/enable-customer-managed-key/07-include-in-template.png" alt-text="Captura de pantalla de la opción para incluir una clave administrada por el cliente en la plantilla de Resource Manager.":::

La siguiente configuración se agregará en la plantilla de Resource Manager. Estas propiedades se pueden parametrizar en canalizaciones de integración y entrega continuas mediante la edición de la [configuración de los parámetros de Azure Resource Manager](continuous-integration-deployment.md#use-custom-parameters-with-the-resource-manager-template).

  :::image type="content" source="media/enable-customer-managed-key/08-template-with-customer-managed-key.png" alt-text="Captura de pantalla de la opción para incluir la clave administrada por el cliente en la plantilla de Azure Resource Manager.":::

> [!NOTE]
> Al agregar la configuración de cifrado a las plantillas de Resource Manager, se agrega una configuración de fábrica que invalidará otras opciones de configuración de fábrica, como las configuraciones de Git, en otros entornos. Si tiene esta configuración habilitada en un entorno con privilegios elevados, como UAT o PROD, consulte [Parámetros globales en CI/CD](author-global-parameters.md#cicd).

## <a name="next-steps"></a>Pasos siguientes

Consulte los [tutoriales](tutorial-copy-data-dot-net.md) para obtener información acerca del uso de Data Factory en otros escenarios.
