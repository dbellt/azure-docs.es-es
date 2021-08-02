---
title: 'Incorporación de un certificado de entidad de certificación personalizado: Azure API Management | Microsoft Docs'
description: Aprenda a agregar un certificado de entidad de certificación personalizado a Azure API Management. También puede ver instrucciones para eliminar un certificado.
services: api-management
documentationcenter: ''
author: mikebudzynski
ms.service: api-management
ms.topic: how-to
ms.date: 06/01/2021
ms.author: apimpm
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: e1cb09f24f12d8c4480833995a95e1e08b5e7bbe
ms.sourcegitcommit: a434cfeee5f4ed01d6df897d01e569e213ad1e6f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/09/2021
ms.locfileid: "111812235"
---
# <a name="how-to-add-a-custom-ca-certificate-in-azure-api-management"></a>Incorporación de un certificado de entidad de certificación personalizado a Azure API Management

Azure API Management permite instalar los certificados de entidad de certificación en la máquina dentro de los almacenes de certificados intermedio y raíz de confianza. Esta funcionalidad debe usarse si los servicios requieren un certificado de entidad de certificación personalizado.

En este artículo se muestra cómo administrar certificados de entidad de certificación de una instancia del servicio Azure API Management en Azure Portal. Por ejemplo, si usa certificados de cliente autofirmados, puede cargar certificados raíz de confianza personalizados en API Management. 

Los certificados de entidad de certificación cargados en API Management solo los puede usar para la validación de certificados la puerta de enlace de API Management administrada. Si usa la [puerta de enlace autohospedada](self-hosted-gateway-overview.md), obtenga información sobre cómo [crear una entidad de certificación personalizada para la puerta de enlace autohospedada](#create-custom-ca-for-self-hosted-gateway) más adelante en este artículo.

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

[!INCLUDE [premium-dev-standard-basic.md](../../includes/api-management-availability-premium-dev-standard-basic.md)]

## <a name="upload-a-ca-certificate"></a><a name="step1"> </a>Carga de un certificado de entidad de certificación

:::image type="content" source="media/api-management-howto-ca-certificates/00.png" alt-text="Certificados de entidad de certificación en Azure Portal":::

Siga los pasos que se describen a continuación para cargar un nuevo certificado de entidad de certificación. Si todavía no ha creado una instancia del servicio API Management, consulte el tutorial [Creación de una instancia del servicio API Management](get-started-create-service-instance.md).

1. Vaya a la instancia del servicio Azure API Management en Azure Portal.

1. En el menú, en **Seguridad**, seleccione **Certificados > Certificados de entidad de certificación > + Agregar**.

1. Busque el archivo .cer del certificado y decida cuál es el almacén de certificados. Solo se necesita la clave pública, por lo que la contraseña es opcional.

    :::image type="content" source="media/api-management-howto-ca-certificates/02.png" alt-text="Adición del certificado de entidad de certificación en Azure Portal"::: 

1. Seleccione **Guardar**. Esta operación puede tardar algunos minutos.

> [!NOTE]
> También puede cargar un certificado de entidad de certificación mediante el comando `New-AzApiManagementSystemCertificate` de PowerShell.

## <a name="delete-a-ca-certificate"></a><a name="step1a"> </a>Eliminación de un certificado de entidad de certificación

Seleccione el certificado y seleccione **Eliminar** en el menú contextual ( **...** ).

## <a name="create-custom-ca-for-self-hosted-gateway"></a>Creación de una entidad de certificación personalizada para una puerta de enlace autohospedada 

Si usa una [puerta de enlace auto hospedada](self-hosted-gateway-overview.md), no se admite la validación de certificados de cliente y servidor mediante certificados raíz de CA cargados en el servicio de API Management. Para establecer confianza, configure un certificado de cliente específico para que sea de confianza para la puerta de enlace como una entidad de certificación personalizada.

Use las API REST de la [entidad de certificación de puerta de enlace](/rest/api/apimanagement/2021-01-01-preview/gateway-certificate-authority) para crear y administrar CA personalizadas para una puerta de enlace autohospedada. Para crear una entidad de certificación personalizada:

1. [Agregue un archivo .pfx de certificado](api-management-howto-mutual-certificates.md) a la instancia de API Management.
1. Use la API REST de [Entidad de certificación de puerta de enlace: creación o actualización](/rest/api/apimanagement/2021-01-01-preview/gateway-certificate-authority/create-or-update) para asociar el certificado a la puerta de enlace autoadministrada.

[Upload a CA certificate]: #step1
[Delete a CA certificate]: #step1a
