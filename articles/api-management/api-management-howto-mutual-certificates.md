---
title: Protección del back-end de API Management mediante la autenticación de certificados de cliente
titleSuffix: Azure API Management
description: Obtenga información sobre cómo administrar certificados de cliente y proteger servicios back-end con la autenticación de certificados de cliente en Azure API Management.
services: api-management
documentationcenter: ''
author: mikebudzynski
ms.service: api-management
ms.topic: article
ms.date: 01/26/2021
ms.author: apimpm
ms.openlocfilehash: d5d261368260a1c9658ae0bef8bdf63a7ca6bafe
ms.sourcegitcommit: 6686a3d8d8b7c8a582d6c40b60232a33798067be
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107750633"
---
# <a name="secure-backend-services-using-client-certificate-authentication-in-azure-api-management"></a>Protección de servicios back-end con la autenticación de certificados de cliente en Azure API Management

API Management permite acceder de forma segura al servicio back-end de una API con certificados de cliente. En esta guía se muestra cómo administrar certificados de la instancia de servicio de Azure API Management con Azure Portal. También se explica cómo configurar una API para que use un certificado para acceder a un servicio back-end.

También puede administrar certificados de API Management mediante la [API REST de API Management](/rest/api/apimanagement/2020-06-01-preview/certificate).

## <a name="certificate-options"></a>Opciones de certificado

API Management proporciona dos opciones para administrar los certificados que se usan para proteger el acceso a los servicios back-end:

* Hacer referencia a un certificado administrado en [Azure Key Vault](../key-vault/general/overview.md) 
* Agregar un archivo de certificado directamente en API Management

Se recomienda el uso de certificados de Key Vault, ya que esto ayuda a mejorar la seguridad de API Management:

* Los certificados almacenados en almacenes de claves se pueden reutilizar entre servicios.
* Las [directivas de acceso](../key-vault/general/security-overview.md#privileged-access) granular se pueden aplicar a los certificados almacenados en los almacenes de claves.
* Los certificados actualizados en el almacén de claves se rotan automáticamente en API Management. Después de la actualización en el almacén de claves, un certificado en API Management se actualiza en un plazo de 4 horas. También puede actualizar manualmente el certificado mediante Azure Portal o a través de la API REST de administración.

## <a name="prerequisites"></a>Requisitos previos

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

* Si todavía no ha creado una instancia de servicio de API Management, consulte [Creación de una instancia de servicio de API Management][Create an API Management service instance].
* Debe tener el servicio back-end configurado para la autenticación de certificados de cliente. Para configurar la autenticación de certificados en Azure App Service, consulte [este artículo][to configure certificate authentication in Azure WebSites refer to this article]. 
* Debe acceder al certificado y a la contraseña para su administración en Azure Key Vault o para cargarlos al servicio de API Management. El certificado debe estar en formato **.pfx**. Se admiten los certificados autofirmados.

### <a name="prerequisites-for-key-vault-integration"></a>Requisitos previos para la integración de un almacén de claves

1. Para conocer los pasos para crear un almacén de claves, vea [Inicio rápido: Creación de un almacén de claves mediante Azure Portal](../key-vault/general/quick-create-portal.md).
1. Habilite una [identidad administrada](api-management-howto-use-managed-service-identity.md) asignada por el sistema o por el usuario en la instancia de API Management.
1. Asigne una [directiva de acceso del almacén de claves](../key-vault/general/assign-access-policy-portal.md) a la identidad administrada con permisos para obtener y enumerar los certificados del almacén. Para agregar la directiva:
    1. En el portal, vaya al almacén de claves.
    1. Seleccione **Configuración > Directivas de acceso > + Agregar directiva de acceso**.
    1. Seleccione **Permisos de certificado** y, a continuación, seleccione **Obtener** y **Enumerar**.
    1. En **Seleccionar la entidad de seguridad**, seleccione el nombre de recurso de la identidad administrada. Si usa una identidad asignada por el sistema, la entidad de seguridad es el nombre de la instancia de API Management.
1. Cree o importe un certificado en el almacén de claves. Consulte [Quickstart: Establecimiento y recuperación de un certificado de Azure Key Vault mediante Azure Portal](../key-vault/certificates/quick-create-portal.md).

[!INCLUDE [api-management-key-vault-network](../../includes/api-management-key-vault-network.md)]

## <a name="add-a-key-vault-certificate"></a>Adición de un certificado del almacén de claves

Consulte [Requisitos previos para la integración de un almacén de claves](#prerequisites-for-key-vault-integration).

> [!CAUTION]
> Al usar un certificado del almacén de claves en API Management, tenga cuidado de no eliminar el certificado, el almacén de claves ni la identidad administrada que se usa para acceder al almacén de claves.

Para agregar un certificado del almacén de claves a API Management:

1. Vaya a la instancia de API Management en [Azure Portal](https://portal.azure.com).
1. En **Seguridad**, seleccione **Certificados**.
1. Seleccione **Certificados** >  **+ Agregar**.
1. En **Id.** , introduzca el nombre que desee.
1. En **Certificado**, seleccione **Almacén de claves**.
1. Escriba el identificador de un certificado de almacén de claves o elija **Seleccionar** para seleccionar un certificado de un almacén de claves.
    > [!IMPORTANT]
    > Si especifica usted mismo el identificador de un certificado del almacén de claves, asegúrese de que no tenga la información de versión. De lo contrario, el certificado no se rotará automáticamente en API Management después de una actualización en el almacén de claves.
1. En **Client identity** (Identidad del cliente), seleccione una identidad asignada por el sistema o una identidad administrada existente asignada por el usuario. Obtenga información sobre cómo [agregar o modificar identidades administradas en el servicio de API Management](api-management-howto-use-managed-service-identity.md).
    > [!NOTE]
    > La identidad necesita permisos para obtener y enumerar los certificados del almacén de claves. Si aún no ha configurado el acceso al almacén de claves, API Management se lo pedirá para poder configurar automáticamente la identidad con los permisos necesarios.
1. Seleccione **Agregar**.

    :::image type="content" source="media/api-management-howto-mutual-certificates/apim-client-cert-kv.png" alt-text="Adición de un certificado del almacén de claves":::

## <a name="upload-a-certificate"></a>Carga de un certificado

Para cargar un certificado de cliente en API Management: 

1. Vaya a la instancia de API Management en [Azure Portal](https://portal.azure.com).
1. En **Seguridad**, seleccione **Certificados**.
1. Seleccione **Certificados** >  **+ Agregar**.
1. En **Id.** , introduzca el nombre que desee.
1. En **Certificado**, seleccione **Personalizado**.
1. Navegue para seleccionar el archivo .pfx del certificado y escriba su contraseña.
1. Seleccione **Agregar**.

    :::image type="content" source="media/api-management-howto-mutual-certificates/apim-client-cert-add.png" alt-text="Cargar un certificado de cliente":::

Una vez cargado el certificado, se muestra en la ventana **Certificados**. Si tiene muchos certificados, tome nota de la huella digital del certificado deseado con el fin de configurar una API para realizar la [autenticación de puerta de enlace](#configure-an-api-to-use-client-certificate-for-gateway-authentication) con un certificado de cliente.

> [!NOTE]
> Para desactivar la validación de la cadena de certificados cuando se utiliza, por ejemplo, un certificado autofirmado, siga los pasos descritos en la sección [Certificados autofirmados](#self-signed-certificates), que se incluye más adelante en este artículo.

## <a name="configure-an-api-to-use-client-certificate-for-gateway-authentication"></a>Configuración de una API para usar un certificado de cliente para la autenticación de puerta de enlace

1. Vaya a la instancia de API Management en [Azure Portal](https://portal.azure.com).
1. En **API**, seleccione **API**.
1. Seleccione una API en la lista. 
2. En la pestaña **Diseño**, seleccione el icono del editor en la sección **Back-end**.
3. En **credenciales de puerta de enlace**, seleccione **Certificado de cliente** y seleccione el certificado en la lista desplegable.
1. Seleccione **Guardar**.

    :::image type="content" source="media/api-management-howto-mutual-certificates/apim-client-cert-enable-select.png" alt-text="Uso de un certificado de cliente para la autenticación de puerta de enlace":::

> [!CAUTION]
> Este cambio se hace efectivo de forma inmediata y llama a las operaciones de la API que realizarán la autenticación en el servidor back-end con el certificado.

> [!TIP]
> Cuando se especifica un certificado para la autenticación de puerta de enlace del servicio back-end de una API, el certificado se integra en la directiva de dicha API y puede verse en el editor de directivas.

## <a name="self-signed-certificates"></a>Certificados autofirmados

Si utiliza certificados autofirmados, tendrá que deshabilitar la validación de la cadena de certificados para que API Management pueda comunicarse con el sistema back-end. De lo contrario, se producirá un error con el código 500. Para establecer esta configuración, puede utilizar los cmdlets de PowerShell [`New-AzApiManagementBackend`](/powershell/module/az.apimanagement/new-azapimanagementbackend) (con el nuevo back-end) o [`Set-AzApiManagementBackend`](/powershell/module/az.apimanagement/set-azapimanagementbackend) (con el back-end existente) y establecer el parámetro `-SkipCertificateChainValidation` en `True`.

```powershell
$context = New-AzApiManagementContext -resourcegroup 'ContosoResourceGroup' -servicename 'ContosoAPIMService'
New-AzApiManagementBackend -Context  $context -Url 'https://contoso.com/myapi' -Protocol http -SkipCertificateChainValidation $true
```

## <a name="delete-a-client-certificate"></a>Eliminar un certificado de cliente

Para eliminar un certificado, selecciónelo y, a continuación, seleccione **Eliminar** en el menú contextual ( **...** ).

:::image type="content" source="media/api-management-howto-mutual-certificates/apim-client-cert-delete-new.png" alt-text="Eliminar un certificado":::

> [!IMPORTANT]
> Si alguna directiva hace referencia al certificado, aparecerá una pantalla de advertencia. Para eliminar el certificado, primero quítelo de todas las directivas que se hayan configurado para usarlo.

## <a name="next-steps"></a>Pasos siguientes

* [Protección de API mediante la autenticación de certificados de cliente en API Management](api-management-howto-mutual-certificates-for-clients.md)
* Más información sobre las [Directivas de Azure API Management](api-management-howto-policies.md)


[How to add operations to an API]: ./mock-api-responses.md
[How to add and publish a product]: api-management-howto-add-products.md
[Monitoring and analytics]: ../api-management-monitoring.md
[Add APIs to a product]: api-management-howto-add-products.md#add-apis
[Publish a product]: api-management-howto-add-products.md#publish-product
[Get started with Azure API Management]: get-started-create-service-instance.md
[API Management policy reference]: ./api-management-policies.md
[Caching policies]: ./api-management-policies.md#caching-policies
[Create an API Management service instance]: get-started-create-service-instance.md

[Azure API Management REST API Certificate entity]: ./api-management-caching-policies.md
[WebApp-GraphAPI-DotNet]: https://github.com/AzureADSamples/WebApp-GraphAPI-DotNet
[to configure certificate authentication in Azure WebSites refer to this article]: ../app-service/app-service-web-configure-tls-mutual-auth.md

