---
title: 'Tutorial: Actualización de la frecuencia de rotación automática de certificados en Key Vault | Microsoft Docs'
description: Tutorial que muestra cómo actualizar la frecuencia de rotación automática de un certificado en Azure Key Vault mediante Azure Portal.
services: key-vault
author: msmbaldwin
tags: azure-resource-manager
ms.service: key-vault
ms.subservice: certificates
ms.topic: tutorial
ms.custom: mvc
ms.date: 04/16/2020
ms.author: sebansal
ms.openlocfilehash: ec88dfc7ded7ecb1d640eb4d73ef4c8045e549cf
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107791992"
---
# <a name="tutorial-configure-certificate-auto-rotation-in-key-vault"></a>Tutorial: Configuración de la rotación automática de certificados en Key Vault

El uso de Azure Key Vault permite realizar el aprovisionamiento, administración e implementación de certificados digitales de forma sencilla. Los certificados pueden ser certificados de Capa de sockets seguros (SSL) o de Seguridad de la capa de transporte (TLS) públicos y privados firmados por una entidad de certificación (CA) o autofirmados. Key Vault también puede solicitar y renovar certificados mediante asociaciones con entidades de certificación, lo que proporciona una solución sólida para la administración del ciclo de vida de los certificados. En este tutorial, actualizará los atributos período de validez, frecuencia de rotación automática y entidad de certificación de un certificado.

En este tutorial se muestra cómo realizar las siguientes acciones:

> [!div class="checklist"]
> * Administrar un certificado mediante el Azure Portal.
> * Agregar una cuenta de proveedor de entidad de certificación.
> * Actualizar el período de validez del certificado.
> * Actualizar la frecuencia de rotación automática del certificado.
> * Actualizar los atributos del certificado mediante Azure PowerShell.

Antes de empezar, lea los [conceptos básicos de Key Vault](../general/basic-concepts.md).

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="sign-in-to-azure"></a>Inicio de sesión en Azure

Inicie sesión en Azure Portal en https://portal.azure.com.

## <a name="create-a-vault"></a>Creación de un almacén

Cree una instancia de Azure Key Vault mediante [Azure Portal](../general/quick-create-portal.md), la [CLI de Azure](../general/quick-create-cli.md) o [Azure PowerShell](../general/quick-create-powershell.md). En el ejemplo, el nombre del almacén de claves es **Example-Vault**.

![Salida tras finalizar la creación de Key Vault](../media/certificates/tutorial-import-cert/vault-properties.png)

## <a name="create-a-certificate-in-key-vault"></a>Creación de un certificado en Key Vault

Cree un certificado o impórtelo en el almacén de claves (consulte los [pasos para crear un certificado en Key Vault](../secrets/quick-create-portal.md)). En este caso, trabajaremos en un certificado llamado **ExampleCertificate**.

## <a name="update-certificate-lifecycle-attributes"></a>Actualización de atributos del ciclo de vida del certificado

En Azure Key Vault, puede actualizar los atributos del ciclo de vida de un certificado tanto en el momento de la creación del certificado como después.

Un certificado creado en Key Vault puede ser:

- Un certificado autofirmado.
- Un certificado creado con una entidad de certificación asociada con Key Vault.
- Un certificado creado con una entidad de certificación que no esté asociada a Key Vault.

Las siguientes entidades de certificación son los proveedores asociados actualmente con Key Vault:

- DigiCert: Key Vault ofrece certificados SSL/TLS de OV.
- GlobalSign: Key Vault ofrece certificados SSL/TLS de OV.

Key Vault rota automáticamente los certificados mediante asociaciones establecidas con las entidades de certificación. Dado que Key Vault solicita y renueva automáticamente los certificados mediante esta asociación, la funcionalidad de rotación automática no es aplicable a los certificados creados con entidades de certificación que no están asociadas con Key Vault.

> [!NOTE]
> Un administrador de cuentas de un proveedor de entidades de certificación crea las credenciales que Key Vault utiliza para crear, renovar y usar certificados TLS/SSL.
![Entidad de certificación](../media/certificates/tutorial-rotate-cert/cert-authority-create.png)
>

### <a name="update-certificate-lifecycle-attributes-at-the-time-of-creation"></a>Actualización de los atributos del ciclo de vida del certificado en el momento de la creación

1. En las páginas de propiedades de Key Vault, seleccione **Certificados**.
1. Seleccione **Generar o importar**.
1. En la pantalla **Crear un certificado**, actualice los siguientes valores:

   - **Periodo de validez**: Escriba el valor (en meses). Una práctica de seguridad recomendada es la creación de certificados de corta duración. De forma predeterminada, el valor de validez de un certificado recién creado es de 12 meses.
   - **Tipo de acción de duración**: Seleccione la acción de renovación automática y alertas del certificado y, a continuación, actualice **Porcentaje de duración** o **Número de días antes de que expire**. De forma predeterminada, la renovación automática de un certificado se establece en el 80 por ciento de su duración. En el menú desplegable, seleccione una de las opciones siguientes.

      |  Automatically renew at a given time (Renovar automáticamente en un momento dado)| Email all contacts at a given time (Enviar por correo electrónico todos los contactos en un momento dado) |
      |-----------|------|
      |Al seleccionar esta opción, se *activa* la rotación automática. | Al seleccionar esta opción, *no* se realizará la rotación automática, solo se alertará a los contactos.|
      
      Dispone de información sobre cómo configurar el contacto de correo electrónico [en esta página](https://docs.microsoft.com/azure/key-vault/certificates/overview-renew-certificate#get-notified-about-certificate-expiration).

1. Seleccione **Crear**.

![Ciclo de vida del certificado](../media/certificates/tutorial-rotate-cert/create-cert-lifecycle.png)

### <a name="update-lifecycle-attributes-of-a-stored-certificate"></a>Actualizar los atributos del ciclo de vida de un certificado almacenado

1. Seleccione el almacén de claves.
1. En las páginas de propiedades de Key Vault, seleccione **Certificados**.
1. Seleccione el certificado que desea actualizar. En este caso, trabajaremos en un certificado llamado **ExampleCertificate**.
1. Seleccione **Directiva de emisión** en la barra de menús superior.

   ![Captura de pantalla que resalta el botón Directiva de emisión.](../media/certificates/tutorial-rotate-cert/cert-issuance-policy.png)

1. En la pantalla **Directiva de emisión**, actualice los valores siguientes:

   - **Periodo de validez**: Actualice el valor (en meses).
   - **Tipo de acción de duración**: Seleccione la acción de renovación automática y alertas del certificado y, a continuación, actualice el **Porcentaje de duración** o **Número de días antes de que expire**.

   ![Propiedades del certificado](../media/certificates/tutorial-rotate-cert/cert-policy-change.png)

1. Seleccione **Guardar**.

> [!IMPORTANT]
> Al cambiar el tipo de acción de duración de un certificado, se registrarán inmediatamente las modificaciones de los certificados existentes.


### <a name="update-certificate-attributes-by-using-powershell"></a>Actualización de los atributos de certificado mediante PowerShell

```azurepowershell


Set-AzureKeyVaultCertificatePolicy -VaultName $vaultName 
                                   -Name $certificateName 
                                   -RenewAtNumberOfDaysBeforeExpiry [276 or appropriate calculated value]
```

> [!TIP]
> Para modificar la directiva de renovación de una lista de certificados, escriba `File.csv` que contiene `VaultName,CertName` como en el ejemplo siguiente:
> <br/>
 `vault1,Cert1` <br/>
>  `vault2,Cert2`
>
>  ```azurepowershell
>  $file = Import-CSV C:\Users\myfolder\ReadCSVUsingPowershell\File.csv 
> foreach($line in $file)
> {
> Set-AzureKeyVaultCertificatePolicy -VaultName $vaultName -Name $certificateName -RenewAtNumberOfDaysBeforeExpiry [276 or appropriate calculated value]
> }
>  ```
> 
Para más información sobre los parámetros, consulte [az keyvault certificate](/cli/azure/keyvault/certificate#az_keyvault_certificate_set_attributes).

## <a name="clean-up-resources"></a>Limpieza de recursos

Otros tutoriales de Key Vault se basan en este tutorial. Si tiene previsto trabajar con estos tutoriales, debería mantener estos recursos existentes en su lugar.
Cuando ya no lo necesite, elimine el grupo de recursos; de este modo se eliminarán también el almacén de claves y los recursos relacionados.

Para eliminar el grupo de recursos mediante el portal:

1. Escriba el nombre del grupo de recursos en el cuadro de **búsqueda** de la parte superior del portal. Cuando vea el grupo de recursos que se utiliza en este inicio rápido en los resultados de búsqueda, selecciónelo.
1. Seleccione **Eliminar grupo de recursos**.
1. En el cuadro **ESCRIBA EL NOMBRE DEL GRUPO DE RECURSOS:** escriba el nombre del grupo de recursos y seleccione **Eliminar**.


## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha actualizado los atributos del ciclo de vida de un certificado. Para más información sobre Key Vault y cómo integrarlo con las aplicaciones, continúe con los artículos siguientes:

- Más información sobre la [administración de la creación de certificados en Azure Key Vault](./create-certificate-scenarios.md).
- Revise [Introducción a Key Vault](../general/overview.md).
