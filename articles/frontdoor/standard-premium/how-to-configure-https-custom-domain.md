---
title: Configuración de HTTPS para su dominio personalizado en una configuración de SKU de Azure Front Door Estándar/Prémium
description: En este artículo, aprenderá a incorporar un dominio personalizado a la SKU de Azure Front Door Estándar/Prémium.
services: frontdoor
author: duongau
ms.service: frontdoor
ms.topic: article
ms.workload: infrastructure-services
ms.date: 06/10/2021
ms.author: amsriva
ms.openlocfilehash: 0e8c597037ac769c293a2f04cb2e300658db93b4
ms.sourcegitcommit: 190658142b592db528c631a672fdde4692872fd8
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/11/2021
ms.locfileid: "112005238"
---
# <a name="configure-https-on-a-front-door-standardpremium-sku-preview-custom-domain-using-the-azure-portal"></a>Configuración de HTTPS en un dominio personalizado de la SKU de Front Door Estándar/Prémium (versión preliminar) mediante Azure Portal

> [!NOTE]
> Esta documentación trata sobre Azure Front Door Estándar/Prémium (versión preliminar). ¿Busca información sobre Azure Front Door? Consulte [aquí](../front-door-overview.md).

Azure Front Door Estándar/Prémium permite la entrega segura de TLS a las aplicaciones de manera predeterminada cuando se agrega un dominio personalizado. Mediante el protocolo HTTPS en el dominio personalizado, se garantiza que los datos confidenciales se entregan de manera segura a través del cifrado TLS/SSL cuando se envían por Internet. Cuando el explorador web se conecta a un sitio web a través de HTTPS, valida el certificado de seguridad del sitio web y comprueba que lo ha emitido una entidad de certificación legítima. Este proceso aporta seguridad y protege las aplicaciones web de posibles ataques.

Azure Front Door Estándar/Prémium admite tanto certificados administrados por el cliente como administrados por Azure. De manera predeterminada, Azure Front Door habilita automáticamente HTTPS para todos los dominios personalizados mediante certificados administrados por Azure. No se requieren otros pasos para obtener un certificado administrado por Azure. Un certificado se crea durante el proceso de validación de dominio. También puede usar su propio certificado mediante la integración de Azure Front Door Estándar/Prémium con su instancia de Key Vault.

> [!IMPORTANT]
> Azure Front Door Estándar/Prémium (versión preliminar) está disponible actualmente en versión preliminar pública.
> Esta versión preliminar se ofrece sin Acuerdo de Nivel de Servicio y no se recomienda para cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas.
> Para más información, consulte [**Condiciones de uso complementarias de las versiones preliminares de Microsoft Azure**](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="prerequisites"></a>Requisitos previos

* Antes de poder configurar HTTPS para el dominio personalizado, primero debe crear un perfil de Azure Front Door Estándar/Prémium. Para más información, consulte el artículo de [inicio rápido sobre cómo crear un perfil de Azure Front Door Estándar/Prémium](create-front-door-portal.md).

* Si todavía no tiene un dominio personalizado, primero debe comprar uno a un proveedor de dominios. Por ejemplo, vea [Compra de un nombre de dominio personalizado](../../app-service/manage-custom-dns-buy-domain.md).

* Si usa Azure para hospedar sus [dominios DNS](../../dns/dns-overview.md), debe delegar el sistema de nombres de dominio (DNS) del proveedor de dominios en una instancia de Azure DNS. Para más información, vea [Delegación de un dominio en DNS de Azure](../../dns/dns-delegate-domain-azure-dns.md). De lo contrario, si usa un proveedor de dominios para controlar el dominio DNS, debe validar manualmente el dominio. Para ello, debe escribir los registros TXT de DNS solicitados.

## <a name="azure-managed-certificates"></a>Certificados administrados por Azure

1. En la configuración del perfil de Azure Front Door Estándar/Prémium, seleccione **Dominios** y, luego, **+ Agregar** para agregar un dominio nuevo.

    :::image type="content" source="../media/how-to-configure-https-custom-domain/add-new-custom-domain.png" alt-text="Captura de pantalla de la página de aterrizaje de la configuración de dominio.":::

1. En la página **Agregar un dominio**, en *Administración de DNS*, seleccione la opción **DNS administrado por Azure**. 

    :::image type="content" source="../media/how-to-configure-https-custom-domain/add-domain-azure-managed.png" alt-text="Captura de pantalla de la página para agregar un dominio con la opción DNS administrado por Azure seleccionada.":::

1. Para validar y asociar el dominio personalizado a un punto de conexión, siga los pasos que se describen en la habitación de un [dominio personalizado](how-to-add-custom-domain.md).

1. Una vez que el dominio personalizado se asocia correctamente al punto de conexión, un certificado administrado por Azure se implementa en Front Door. Este proceso puede tardar de varios minutos a una hora en completarse.

## <a name="using-your-own-certificate"></a>Uso de su propio certificado

También puede elegir usar su propio certificado TLS.  Al crear el certificado TLS/SSL, tiene que crear una cadena de certificados completa con una entidad de certificación (CA) permitida que forme parte de la [lista de CA de confianza de Microsoft](https://ccadb-public.secure.force.com/microsoft/IncludedCACertificateReportForMSFT). Si usa una CA no permitida, la solicitud se rechazará.  El certificado debe tener una cadena de certificados completa con certificados de hoja e intermedios, y la CA raíz debe formar parte de la [lista de CA de confianza de Microsoft](https://ccadb-public.secure.force.com/microsoft/IncludedCACertificateReportForMSFT). Si se presenta un certificado sin una cadena completa, no se garantiza que las solicitudes que involucran ese certificado funcionen según lo previsto. Este certificado se debe importar a una instancia de Azure Key Vault a fin de poder usarlo con Azure Front Door Estándar/Prémium. Consulte cómo [importar un certificado](../../key-vault/certificates/tutorial-import-certificate.md) a Azure Key Vault.

#### <a name="prepare-your-azure-key-vault-account-and-certificate"></a>Preparación de la cuenta y el certificado de Azure Key Vault
 
1. Debe tener una cuenta de Azure Key Vault en ejecución en la misma suscripción que su instancia de Azure Front Door Estándar/Prémium para la que quiera habilitar HTTPS personalizado. Cree una cuenta de Azure Key Vault si no tiene una.

    > [!WARNING]
    > Azure Front Door actualmente solo admite cuentas de Key Vault en la misma suscripción que la configuración de Front Door. Si elige una instancia de Key Vault que se encuentre en una suscripción diferente que la de Azure Front Door Estándar/Prémium, obtendrá un error.

1. Si ya tiene un certificado, puede cargarlo directamente en su cuenta de Azure Key Vault. De lo contrario, cree un certificado directamente a través de Azure Key Vault a partir de una de las entidades de certificación de asociado con que se integra Azure Key Vault. Cargue el certificado como un objeto de **certificado** en lugar de como un **secreto**.

    > [!NOTE]
    > Para su propio certificado TLS/SSL, Front Door no admite certificados con algoritmos de criptografía de EC. El certificado debe tener una cadena de certificados completa con certificados de hoja e intermedios, y la CA raíz debe formar parte de la [lista de CA de confianza de Microsoft](https://ccadb-public.secure.force.com/microsoft/IncludedCACertificateReportForMSFT). 

#### <a name="register-azure-front-door"></a>Registro en Azure Front Door

Registre la entidad de servicio para Azure Front Door como una aplicación en Azure Active Directory mediante PowerShell.

> [!NOTE]
> Esta acción requiere permisos de administrador global y solo debe realizarse **una vez** por inquilino.

1. Si es necesario, instale [Azure PowerShell](/powershell/azure/install-az-ps) en PowerShell en la máquina local.

1. En PowerShell, ejecute el siguiente comando:

     `New-AzADServicePrincipal -ApplicationId "205478c0-bd83-4e1b-a9d6-db63a3e1e1c8"`              

#### <a name="grant-azure-front-door-access-to-your-key-vault"></a>Concesión a Azure Front Door de acceso al almacén de claves
 
Conceda a Azure Front Door permisos para acceder a los certificados ubicados en su cuenta de Azure Key Vault.

1. En su cuenta de Key Vault, en CONFIGURACIÓN, seleccione **Directivas de acceso**. Luego, seleccione **Agregar nueva** para crear una directiva.

1. En **Seleccionar la entidad de seguridad**, busque **205478c0-bd83-4e1b-a9d6-db63a3e1e1c8** y elija ** Microsoft.AzureFrontDoor-Cdn**. Haga clic en **Seleccionar**.

1. En **Permisos de secretos**, seleccione **Obtener** para permitir que Front Door recupere el certificado.

1. En **Permisos de certificado**, seleccione **Obtener** para permitir que Front Door recupere el certificado.

1. Seleccione **Aceptar**. 

#### <a name="select-the-certificate-for-azure-front-door-to-deploy"></a>Selección del certificado de Azure Front Door que se va a implementar
 
1. Vuelva a su instancia de Azure Front Door Estándar/Prémium en el portal.

1. Vaya a **Secretos** en *Configuración* y seleccione **Agregar certificado**.

    :::image type="content" source="../media/how-to-configure-https-custom-domain/add-certificate.png" alt-text="Captura de pantalla de la página de aterrizaje de secretos de Azure Front Door.":::

1. En la página **Agregar certificado**, active la casilla correspondiente al certificado que quiere agregar a Azure Front Door Estándar/Prémium. Deje la selección de la versión como "Más reciente" y seleccione **Agregar**. 

    :::image type="content" source="../media/how-to-configure-https-custom-domain/add-certificate-page.png" alt-text="Captura de pantalla de la página para agregar certificado.":::

1. Una vez que el certificado se aprovisiona correctamente, podrá usarlo al agregar un dominio personalizado nuevo.

    :::image type="content" source="../media/how-to-configure-https-custom-domain/successful-certificate-provisioned.png" alt-text="Captura de pantalla del certificado agregado correctamente a los secretos.":::

1. Vaya a **Dominios** en *Configuración* y seleccione **+ Agregar** para agregar un dominio personalizado nuevo. En la página **Agregar un dominio**, elija la opción "Bring Your Own Certificate (BYOC)" para *HTTPS*. En *Secreto*, seleccione el certificado que quiere usar en el menú desplegable. 

    > [!NOTE]
    > El certificado seleccionado debe tener un nombre común (CN) igual al del dominio personalizado que se va a agregar.

    :::image type="content" source="../media/how-to-configure-https-custom-domain/add-custom-domain-https.png" alt-text="Captura de pantalla de la página para agregar un dominio personalizado con HTTPS.":::

1. Para validar el certificado, siga los pasos que aparecen en la pantalla. A continuación, asocie el dominio personalizado recién creado a un punto de conexión, tal como se describe en la guía de [creación de un dominio personalizado](how-to-add-custom-domain.md).

#### <a name="change-from-azure-managed-to-bring-your-own-certificate-byoc"></a>Cambio de un certificado administrado por Azure o Bring Your Own Certificate (BYOC)

1. Para cambiar un certificado administrado por Azure existente a un certificado administrado por el usuario, seleccione el estado del certificado para abrir la página **Detalles del certificado**.

    :::image type="content" source="../media/how-to-configure-https-custom-domain/domain-certificate.png" alt-text="Captura de pantalla del estado del certificado en la página de aterrizaje de dominios." lightbox="../media/how-to-configure-https-custom-domain/domain-certificate-expanded.png":::

1. En la página **Detalles del certificado**, puede cambiar de la opción "Administrado por Azure" a la opción "Bring Your Own Certificate (BYOC)". Luego, siga los mismos pasos mencionados para elegir un certificado. Seleccione **Actualizar** para cambiar el certificado asociado con un dominio.

    :::image type="content" source="../media/how-to-configure-https-custom-domain/certificate-details-page.png" alt-text="Captura de pantalla de la página de detalles del certificado.":::

## <a name="next-steps"></a>Pasos siguientes

Obtenga información sobre el [almacenamiento en caché con Azure Front Door Estándar/Prémium](concept-caching.md).
