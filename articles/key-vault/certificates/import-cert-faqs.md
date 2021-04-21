---
title: 'Preguntas más frecuentes: importación de certificados de Azure Key Vault'
description: Obtenga respuestas a preguntas frecuentes sobre la importación de certificados de Azure Key Vault.
services: key-vault
author: msmbaldwin
tags: azure-resource-manager
ms.service: key-vault
ms.subservice: certificates
ms.topic: how-to
ms.date: 07/20/2020
ms.author: sebansal
ms.openlocfilehash: 97dfc2db837f728b8cb4ece9a064f99006c9996b
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107767824"
---
# <a name="importing-azure-key-vault-certificates-faq"></a>Preguntas frecuentes sobre la importación de certificados de Azure Key Vault

En este artículo se responde a las preguntas frecuentes sobre la importación de certificados de Azure Key Vault.

## <a name="frequently-asked-questions"></a>Preguntas más frecuentes

### <a name="how-can-i-import-a-certificate-in-azure-key-vault"></a>¿Cómo importo un certificado en Azure Key Vault?

En la operación de importación de certificados, Azure Key Vault acepta dos formatos de archivo de certificado: PEM y PFX. Aunque hay archivos PEM que contienen solo la parte pública, Key Vault requiere y acepta únicamente un archivo PEM o PFX con una clave privada. Para más información, consulte [Importación de un certificado en Key Vault](./tutorial-import-certificate.md#import-a-certificate-to-key-vault).

### <a name="after-i-import-a-password-protected-certificate-to-key-vault-and-then-download-it-why-cant-i-see-the-password-thats-associated-with-it"></a>Después de importar un certificado protegido por contraseña a Key Vault y luego descargarlo, ¿por qué se ve la contraseña que tiene asociada?
     
Una vez que se ha importado y protegido un certificado en Key Vault, su contraseña asociada no se guarda. La contraseña solo es necesaria una vez durante la operación de importación. Aunque se trata de una decisión intencionada, siempre puede obtener el certificado como secreto y convertirlo de Base64 a PFX y agregar la contraseña mediante [Azure PowerShell](https://social.technet.microsoft.com/wiki/contents/articles/37431.exporting-azure-app-service-certificates.aspx).

### <a name="how-can-i-resolve-a-bad-parameter-error-what-are-the-supported-certificate-formats-for-importing-to-key-vault"></a>¿Cómo se puede resolver un error de "parámetro incorrecto"? ¿Cuáles son los formatos de certificado admitidos para la importación en Key Vault?

Al importar un certificado, debe asegurarse de que la clave esté incluida en el archivo. Si tiene una clave privada almacenada por separado en un formato diferente, debe combinar la clave con el certificado. Algunas entidades de certificación (CA) proporcionan certificados en otros formatos. Por lo tanto, antes de importar el certificado, asegúrese de que se encuentra en formato de archivo PEM o PFX y de que la clave usa el cifrado de Rivest, Shamir y Adleman (RSA) o de criptografía de curva elíptica (ECC). 

Para más información, consulte los [requisitos de certificado](./certificate-scenarios.md#formats-of-import-we-support) y los [requisitos de clave de certificado](../keys/about-keys.md).

###  <a name="can-i-import-a-certificate-by-using-an-arm-template"></a>¿Se puede importar un certificado mediante una plantilla de ARM?

No, no es posible realizar operaciones de certificado mediante una plantilla de Azure Resource Manager (ARM). La solución alternativa recomendada sería usar los métodos de importación de certificados en Azure API, la CLI de Azure o PowerShell. Si ya tiene un certificado, puede importarlo como secreto.

### <a name="when-i-import-a-certificate-via-the-azure-portal-i-get-a-something-went-wrong-error-how-can-i-investigate-further"></a>Al importar un certificado a través de Azure Portal, aparece el error "Hubo un problema". ¿Cómo puedo investigar más?
     
Para ver un error más descriptivo, importe el archivo de certificado mediante la [CLI de Azure](/cli/azure/keyvault/certificate#az_keyvault_certificate_import) o [PowerShell](/powershell/module/azurerm.keyvault/import-azurekeyvaultcertificate).

### <a name="how-can-i-resolve-error-type-access-denied-or-user-is-unauthorized-to-import-certificate"></a>¿Cómo se resuelve el "Tipo de error: Acceso denegado o usuario no autorizado para importar el certificado"?
    
La operación de importación requiere la concesión de permisos de usuario para importar el certificado en las directivas de acceso. Para ello, vaya al almacén de claves, seleccione **Directivas de acceso** > **Agregar directiva de acceso** > **Select Certificate Permissions** (Seleccionar permisos de certificado)  > **Entidad de seguridad**, busque el usuario y, a continuación, agregue su dirección de correo electrónico. 

Para más información sobre las directivas de acceso relacionadas con los certificados, consulte [Acerca de los certificados de Azure Key Vault](./about-certificates.md#certificate-access-control).


### <a name="how-can-i-resolve-error-type-conflict-when-creating-a-certificate"></a>¿Cómo se resuelve el "Tipo de error: Conflicto al crear un certificado"?
    
El nombre de cada certificado debe ser único. Un certificado con el mismo nombre podría estar en un estado de eliminación temporal. Además, según la [composición de un certificado](./about-certificates.md#composition-of-a-certificate), cuando se crea un certificado, se crea un secreto direccionable con el mismo nombre, por lo que, si hay otra clave o secreto en el almacén de claves con el mismo nombre que el que intenta especificar para el certificado, no se podrá crear este y deberá quitar esa clave o secreto o usar un nombre diferente para él. 

Para más información, consulte [Operación de obtención de certificado eliminado](/rest/api/keyvault/getdeletedcertificate/getdeletedcertificate).

### <a name="why-am-i-getting-error-type-char-length-is-too-long"></a>¿Por qué se muestra el mensaje "Tipo de error: Longitud de caracteres demasiado larga"?
Este error puede deberse a dos razones:    
* El nombre del firmante del certificado está limitado a 200 caracteres.
* La contraseña del certificado está limitada a 200 caracteres.


### <a name="error-the-specified-pem-x509-certificate-content-is-in-an-unexpected-format-please-check-if-certificate-is-in-valid-pem-format"></a>Error "The specified PEM X.509 certificate content is in an unexpected format. Please check if certificate is in valid PEM format" (El contenido del certificado PEM X.509 especificado tiene un formato inesperado. Compruebe si el certificado tiene un formato PEM válido).
Compruebe que el contenido del archivo PEM use separadores de líneas de estilo UNIX `(\n)`.

### <a name="can-i-import-an-expired-certificate-to-azure-key-vault"></a>¿Se puede importar un certificado expirado en Azure Key Vault?
    
No, los certificados PFX expirados no se pueden importar a Key Vault.

### <a name="how-can-i-convert-my-certificate-to-the-proper-format"></a>¿Cómo se puede convertir el certificado al formato adecuado?

Puede pedir a la entidad de certificación que proporcione el certificado en el formato necesario. También hay herramientas de terceros que pueden ayudarle a convertir el certificado al formato adecuado.

### <a name="can-i-import-certificates-from-non-partner-cas"></a>¿Puedo importar certificados de entidades de certificación no asociadas?
Sí, puede importar certificados desde cualquier entidad de certificación, pero el almacén de claves no podrá renovarlos automáticamente. Puede establecer avisos para recibir notificaciones sobre la expiración del certificado.

### <a name="if-i-import-a-certificate-from-a-partner-ca-will-the-autorenewal-feature-still-work"></a>Si se importa un certificado de una entidad de certificación asociada, ¿seguirá funcionando la característica de renovación automática?
Sí. Una vez cargado el certificado, asegúrese de especificar la rotación automática en la directiva de emisión del certificado. La configuración permanecerá en vigor hasta que se publique el siguiente ciclo o versión de certificado.

### <a name="why-cant-i-see-the-app-service-certificate-that-i-imported-to-key-vault"></a>¿Por qué no se puede ver el certificado de App Service que se importó a Key Vault? 
Si ha importado el certificado correctamente, debería poder confirmarlo en el panel **Secretos**.


## <a name="next-steps"></a>Pasos siguientes

- [Certificados de Azure Key Vault](./about-certificates.md)
