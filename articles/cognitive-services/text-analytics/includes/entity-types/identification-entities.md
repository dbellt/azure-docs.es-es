---
title: Entidades de identificación
titleSuffix: Azure Cognitive Services
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: text-analytics
ms.topic: include
ms.date: 04/16/2021
ms.author: aahi
ms.openlocfilehash: 6c13134ded2ef276520835880f9dda331232709a
ms.sourcegitcommit: 8bca2d622fdce67b07746a2fb5a40c0c644100c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/09/2021
ms.locfileid: "111761497"
---
### <a name="financial-account-identification"></a>Identificación de cuenta financiera

Esta categoría de entidad incluye información financiera y formas oficiales de identificación.

#### <a name="category-aba-routing-number"></a>Categoría: número de ruta de ABA

Este categoría contiene la entidad siguiente:

:::row:::
    :::column span="":::
        **Entidad**

        Número de ruta de ABA

    :::column-end:::
    :::column span="2":::
        **Detalles**

        Números de enrutamiento de tránsito de American Banker Association (ABA). También se devuelve con `domain=phi`.

        Para obtener esta categoría de entidad, agregue `ABARoutingNumber` al parámetro `pii-categories`. `ABARoutingNumber` también se devolverá en la respuesta de la API si se detecta.
      
    :::column-end:::
    :::column span="2":::
      **Idiomas de documento admitidos**

      `en`
      
   :::column-end:::
:::row-end:::


#### <a name="category-swift-code"></a>Categoría: código SWIFT

Este categoría contiene la entidad siguiente:

:::row:::
    :::column span="":::
        **Entidad**

        Código SWIFT

    :::column-end:::
    :::column span="2":::
        **Detalles**

        Códigos SWIFT de información de instrucciones de pago. También se devuelve con `domain=phi`.

        Para obtener esta categoría de entidad, agregue `SWIFTCode` al parámetro `pii-categories`. `SWIFTCode` se devolverá en la respuesta de la API si se detecta.
      
    :::column-end:::
    :::column span="2":::
      **Idiomas de documento admitidos**

      `en`, `es`, `fr`, `de`, `it`, `pt-pt`, `pt-br`, `ja`
      
   :::column-end:::
:::row-end:::

#### <a name="category-credit-card"></a>Categoría: tarjeta de crédito

Este categoría contiene la entidad siguiente:

:::row:::
    :::column span="":::
        **Entidad**

        Tarjeta de crédito

    :::column-end:::
    :::column span="2":::
        **Detalles**

        Números de tarjeta de crédito. También se devuelve con `domain=phi`.

        Para obtener esta categoría de entidad, agregue `CreditCardNumber` al parámetro `pii-categories`. `CreditCardNumber` se devolverá en la respuesta de la API si se detecta.

    :::column-end:::
    :::column span="2":::
      **Idiomas de documento admitidos**

      `en`, `es`, `fr`, `de`, `it`, `pt-pt`, `pt-br`, `ja`, `zh-hans`, `ja`, `ko`
      
   :::column-end:::
:::row-end:::

#### <a name="category-international-banking-account-number-iban"></a>Categoría: número de cuenta bancaria internacional (IBAN) 

Este categoría contiene la entidad siguiente:

:::row:::
    :::column span="":::
        **Entidad**

        Tarjeta de crédito

    :::column-end:::
    :::column span="2":::
        **Detalles**

        Códigos IBAN de información de instrucciones de pago. También se devuelve con `domain=phi`.

        Para obtener esta categoría de entidad, agregue `InternationlBankingAccountNumber` al parámetro `pii-categories`. `InternationlBankingAccountNumber` se devolverá en la respuesta de la API si se detecta.
      
    :::column-end:::
    :::column span="2":::
      **Idiomas de documento admitidos**

      `en`, `es`, `fr`, `de`, `it`, `pt-pt`, `pt-br`
      
   :::column-end:::
:::row-end:::

### <a name="government-and-countryregion-specific-identification"></a>Identificación específica del país o región y del gobierno

> [!NOTE]
> Las entidades financieras y específicas del país siguientes no se devuelven con el parámetro `domain=phi`:
> * Números de pasaporte
> * Id. de impuesto

Las entidades siguientes se agrupan y enumeran por país:

#### <a name="argentina"></a>Argentina

:::row:::
    :::column span="":::
        **Entidad**

        Número de identidad nacional de Argentina (DNI) 

    :::column-end:::
    :::column span="2":::
        **Detalles** También se devuelve con `domain=phi`.
        
        Para obtener esta categoría de entidad, agregue `ARNationalIdentityNumber` al parámetro `pii-categories`. `ARNationalIdentityNumber` se devolverá en la respuesta de la API si se detecta.
      
    :::column-end:::
    :::column span="":::
      **Idiomas de documento admitidos**

      `en`, `es`
      
   :::column-end:::
:::row-end:::


#### <a name="austria"></a>Austria

:::row:::
    :::column span="":::
        **Entidad**

        Tarjeta de identidad de Austria

    :::column-end:::
    :::column span="2":::
        **Detalles**

        Para obtener esta categoría de entidad, agregue `ATIdentityCard` al parámetro `pii-categories`. `ATIdentityCard` se devolverá en la respuesta de la API si se detecta.
      
    :::column-end:::
    :::column span="":::
      **Idiomas de documento admitidos**

      `de`
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Número de identificación fiscal de Austria

    :::column-end:::
    :::column span="2":::

        Para obtener esta categoría de entidad, agregue `ATTaxIdentificationNumber` al parámetro `pii-categories`. `ATTaxIdentificationNumber` se devolverá en la respuesta de la API si se detecta.
      
    :::column-end:::
    :::column span="":::

      `de`
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Número de identificación fiscal (NIF) de Austria

    :::column-end:::
    :::column span="2":::

        Para obtener esta categoría de entidad, agregue `ATValueAddedTaxNumber` al parámetro `pii-categories`. `ATValueAddedTaxNumber` se devolverá en la respuesta de la API si se detecta.
      
    :::column-end:::
    :::column span="":::

      `de`
      
   :::column-end:::
:::row-end:::



#### <a name="australia"></a>Australia

:::row:::
    :::column span="":::
        **Entidad**

        Número de cuenta bancaria de Australia

    :::column-end:::
    :::column span="2":::
        **Detalles**

        Para obtener esta categoría de entidad, agregue `AUDriversLicenseNumber` al parámetro `pii-categories`. `AUDriversLicenseNumber` se devolverá en la respuesta de la API si se detecta.
      
        También se devuelve con `domain=phi`.
    :::column-end:::
    :::column span="":::
      **Idiomas de documento admitidos**

      `en`
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Número de empresa de Australia

    :::column-end:::
    :::column span="2":::

        Para obtener esta categoría de entidad, agregue `AUBusinessNumber` al parámetro `pii-categories`. `AUBusinessNumber` se devolverá en la respuesta de la API si se detecta.
      
    :::column-end:::
    :::column span="":::

      `en`
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Número de sociedad de Australia

    :::column-end:::
    :::column span="2":::

        Para obtener esta categoría de entidad, agregue `AUCompanyNumber` al parámetro `pii-categories`. `AUCompanyNumber` se devolverá en la respuesta de la API si se detecta.
      
    :::column-end:::
    :::column span="":::

      `en`
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Permiso de conducir de Australia  

    :::column-end:::
    :::column span="2":::

        Para obtener esta categoría de entidad, agregue `AUDriversLicense` al parámetro `pii-categories`. `AUDriversLicense` se devolverá en la respuesta de la API si se detecta.
      
        También se devuelve con `domain=phi`.
    :::column-end:::
    :::column span="":::

      `en`
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Número de cuenta del seguro médico de Australia

    :::column-end:::
    :::column span="2":::

        Para obtener esta categoría de entidad, agregue `AUMedicalAccountNumber` al parámetro `pii-categories`. `AUMedicalAccountNumber` se devolverá en la respuesta de la API si se detecta.
      
        También se devuelve con `domain=phi`.
    :::column-end:::
    :::column span="":::

      `en`
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Número de pasaporte de Australia

    :::column-end:::
    :::column span="2":::

        Para obtener esta categoría de entidad, agregue `ATPassportNumber` al parámetro `pii-categories`. `ATPassportNumber` se devolverá en la respuesta de la API si se detecta.
      
    :::column-end:::
    :::column span="":::

      `en`
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Número de expediente fiscal de Australia

    :::column-end:::
    :::column span="2":::

        Para obtener esta categoría de entidad, agregue `ATTaxIdentificationNumber` al parámetro `pii-categories`. `ATTaxIdentificationNumber` se devolverá en la respuesta de la API si se detecta.
      
    :::column-end:::
    :::column span="":::

      `en`
      
   :::column-end:::
:::row-end:::


#### <a name="belgium"></a>Bélgica

:::row:::
    :::column span="":::
        **Entidad**

        Número nacional de Bélgica

    :::column-end:::
    :::column span="2":::
        **Detalles**

        Para obtener esta categoría de entidad, agregue `BENationalNumber` al parámetro `pii-categories`. `BENationalNumber` se devolverá en la respuesta de la API si se detecta.
      
        También se devuelve con `domain=phi`.
    :::column-end:::
    :::column span="":::
      **Idiomas de documento admitidos**

      `fr`, `de`
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Número de identificación fiscal (NIF) de Bélgica

    :::column-end:::
    :::column span="2":::

        Para obtener esta categoría de entidad, agregue `BEValueAddedTaxNumber` al parámetro `pii-categories`. `BEValueAddedTaxNumber` se devolverá en la respuesta de la API si se detecta.
      
    :::column-end:::
    :::column span="":::

      `fr`, `de`
      
   :::column-end:::
:::row-end:::


#### <a name="brazil"></a>Brasil 

:::row:::
    :::column span="":::
        **Entidad**

        Número de entidad jurídica de Brasil (CNPJ)

        

    :::column-end:::
    :::column span="2":::
        **Detalles**

        Para obtener esta categoría de entidad, agregue `BRLegalEntityNumber` al parámetro `pii-categories`. `BRLegalEntityNumber` se devolverá en la respuesta de la API si se detecta.
      
        También se devuelve con `domain=phi`.
    :::column-end:::
    :::column span="":::
      **Idiomas de documento admitidos**

      `pt-pt`, `pt-br`
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Número de CPF de Brasil

    :::column-end:::
    :::column span="2":::

        Para obtener esta categoría de entidad, agregue `BRCPFNumber` al parámetro `pii-categories`. `BRCPFNumber` se devolverá en la respuesta de la API si se detecta.
      
        También se devuelve con `domain=phi`.
    :::column-end:::
    :::column span="":::

      `pt-pt`, `pt-br`
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Tarjeta de identificación nacional de Brasil (RG)

    :::column-end:::
    :::column span="2":::

        Para obtener esta categoría de entidad, agregue `BRNationalIDRG` al parámetro `pii-categories`. `BRNationalIDRG` se devolverá en la respuesta de la API si se detecta.
      
        También se devuelve con `domain=phi`.
    :::column-end:::
    :::column span="":::

      `pt-pt`, `pt-br`
      
   :::column-end:::
:::row-end:::

#### <a name="canada"></a>Canadá

:::row:::
    :::column span="":::
        **Entidad**

        Número de cuenta bancaria de Canadá

    :::column-end:::
    :::column span="2":::
        **Detalles**

        Para obtener esta categoría de entidad, agregue `CABankAccountNumber` al parámetro `pii-categories`. `CABankAccountNumber` se devolverá en la respuesta de la API si se detecta.
    
        También se devuelve con `domain=phi`.
    :::column-end:::
    :::column span="":::
      **Idiomas de documento admitidos**

      `en`, `fr`
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Número de permiso de conducir de Canadá

    :::column-end:::

    :::column span="2":::

        Para obtener esta categoría de entidad, agregue `CADriversLicenseNumber` al parámetro `pii-categories`. `CADriversLicenseNumber` se devolverá en la respuesta de la API si se detecta.
      
        También se devuelve con `domain=phi`.
    :::column-end:::

    :::column span="":::

      `en`, `fr`
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Número del servicio de salud de Canadá

        
    :::column-end:::

    :::column span="2":::

        Para obtener esta categoría de entidad, agregue `CAHealthServiceNumber` al parámetro `pii-categories`. `CAHealthServiceNumber` se devolverá en la respuesta de la API si se detecta.
      
        También se devuelve con `domain=phi`.
    :::column-end:::

    :::column span="":::

      `en`, `fr`
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Número de pasaporte de Canadá

    :::column-end:::
    :::column span="2":::

        Para obtener esta categoría de entidad, agregue `CAPassportNumber` al parámetro `pii-categories`. `CAPassportNumber` se devolverá en la respuesta de la API si se detecta.
      
    :::column-end:::
    :::column span="":::

      `en`, `fr`
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Número de identificación sanitaria personal de Canadá (PHIN)

    :::column-end:::
    :::column span="2":::

        Para obtener esta categoría de entidad, agregue `CAPersonalHealthIdentification` al parámetro `pii-categories`. `CAPersonalHealthIdentification` se devolverá en la respuesta de la API si se detecta.

        También se devuelve con `domain=phi`.
      
    :::column-end:::
    :::column span="":::

      `en`, `fr`
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Número de la seguridad social de Canadá

    :::column-end:::
    :::column span="2":::

        Para obtener esta categoría de entidad, agregue `CASocialInsuranceNumber` al parámetro `pii-categories`. `CASocialInsuranceNumber` se devolverá en la respuesta de la API si se detecta.
      
        También se devuelve con `domain=phi`.
    :::column-end:::
    :::column span="":::

      `en`, `fr`
      
   :::column-end:::
:::row-end:::

#### <a name="chile"></a>Chile 

:::row:::
    :::column span="":::
        **Entidad**

        Número del carné de identidad de Chile

    :::column-end:::
    :::column span="2":::
        **Detalles**

        Para obtener esta categoría de entidad, agregue `CLIdentityCardNumber` al parámetro `pii-categories`. `CLIdentityCardNumber` se devolverá en la respuesta de la API si se detecta.
      
        También se devuelve con `domain=phi`.
    :::column-end:::
    :::column span="":::
      **Idiomas de documento admitidos**

      `es`
      
   :::column-end:::
:::row-end:::

#### <a name="china"></a>China

:::row:::
    :::column span="":::
        **Entidad**

        Número del carné de identidad de residente de China (PRC)

    :::column-end:::
    :::column span="2":::
        **Detalles**

        Para obtener esta categoría de entidad, agregue `CNResidentIdentityCardNumber` al parámetro `pii-categories`. `CNResidentIdentityCardNumber` se devolverá en la respuesta de la API si se detecta.
      
        También se devuelve con `domain=phi`.
    :::column-end:::
    :::column span="":::
      **Idiomas de documento admitidos**

      `zh-hans`
      
   :::column-end:::
:::row-end:::


#### <a name="european-union-eu"></a>Unión Europea (UE)

:::row:::
    :::column span="":::
        **Entidad**

        Número de tarjeta de débito de la UE

    :::column-end:::
    :::column span="2":::
        **Detalles**

        Para obtener esta categoría de entidad, agregue `EUDebitCardNumber` al parámetro `pii-categories`. `EUDebitCardNumber` se devolverá en la respuesta de la API si se detecta.
      
        También se devuelve con `domain=phi`.
    :::column-end:::
    :::column span="":::
      **Idiomas de documento admitidos**

      `en`, `es`, `fr`, `de`, `it`, `pt-pt` 
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Número de permiso de conducir de la UE

    :::column-end:::
    :::column span="2":::

        Para obtener esta categoría de entidad, agregue `EUDriversLicenseNumber` al parámetro `pii-categories`. `EUDriversLicenseNumber` se devolverá en la respuesta de la API si se detecta.
      
        También se devuelve con `domain=phi`.
    :::column-end:::
    :::column span="":::

      `en`, `es`, `fr`, `de`, `it`, `pt-pt` 
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Coordenadas GPU de la UE

    :::column-end:::
    :::column span="2":::

        Para obtener esta categoría de entidad, agregue `EUGPSCoordinates` al parámetro `pii-categories`. `EUGPSCoordinates` se devolverá en la respuesta de la API si se detecta.
      
    :::column-end:::
    :::column span="":::

      `en`, `es`, `fr`, `de`, `it`, `pt-pt` 
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Número de identificación nacional de la UE

    :::column-end:::
    :::column span="2":::

        Para obtener esta categoría de entidad, agregue `EUNationalIdentificationNumber` al parámetro `pii-categories`. `EUNationalIdentificationNumber` se devolverá en la respuesta de la API si se detecta.
      
        También se devuelve con `domain=phi`.
    :::column-end:::
    :::column span="":::

      `en`, `es`, `fr`, `de`, `it`, `pt-pt` 
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Número de pasaporte de la UE

    :::column-end:::
    :::column span="2":::

        Para obtener esta categoría de entidad, agregue `EUPassportNumber` al parámetro `pii-categories`. `EUPassportNumber` se devolverá en la respuesta de la API si se detecta.
      
    :::column-end:::
    :::column span="":::

      `en`, `es`, `fr`, `de`, `it`, `pt-pt` 
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Número del seguro social (SSN) o identificador equivalente de la UE

    :::column-end:::
    :::column span="2":::

        Para obtener esta categoría de entidad, agregue `EUSocialSecurityNumber` al parámetro `pii-categories`. `EUSocialSecurityNumber` se devolverá en la respuesta de la API si se detecta.
      
        También se devuelve con `domain=phi`.
    :::column-end:::
    :::column span="":::

      `en`, `es`, `fr`, `de`, `it`, `pt-pt` 
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Número de identificación fiscal de la UE (TIN)

    :::column-end:::
    :::column span="2":::

        Para obtener esta categoría de entidad, agregue `EUTaxIdentificationNumber` al parámetro `pii-categories`. `EUTaxIdentificationNumber` se devolverá en la respuesta de la API si se detecta.
      
    :::column-end:::
    :::column span="":::

      `en`, `es`, `fr`, `de`, `it`, `pt-pt` 
      
   :::column-end:::
:::row-end:::

#### <a name="france"></a>Francia

:::row:::
    :::column span="":::
        **Entidad**

        Número de permiso de conducir de Francia

    :::column-end:::
    :::column span="2":::
        **Detalles**

        Para obtener esta categoría de entidad, agregue `FRDriversLicenseNumber` al parámetro `pii-categories`. `FRDriversLicenseNumber` se devolverá en la respuesta de la API si se detecta.
      
        También se devuelve con `domain=phi`.
    :::column-end:::
    :::column span="":::
      **Idiomas de documento admitidos**

      `fr` 
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Número de seguro médico de Francia

    :::column-end:::
    :::column span="2":::

        Para obtener esta categoría de entidad, agregue `FRHealthInsuranceNumber` al parámetro `pii-categories`. `FRHealthInsuranceNumber` se devolverá en la respuesta de la API si se detecta.
      
    :::column-end:::
    :::column span="":::

      `fr` 
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Carné de identidad nacional de Francia (CNI)

    :::column-end:::
    :::column span="2":::

        Para obtener esta categoría de entidad, agregue `FRNationalID` al parámetro `pii-categories`. `FRNationalID` se devolverá en la respuesta de la API si se detecta.
      
        También se devuelve con `domain=phi`.
    :::column-end:::
    :::column span="":::

      `fr` 
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Número de pasaporte de Francia

    :::column-end:::
    :::column span="2":::

        Para obtener esta categoría de entidad, agregue `FRPassportNumber` al parámetro `pii-categories`. `FRPassportNumber` se devolverá en la respuesta de la API si se detecta.
      
    :::column-end:::
    :::column span="":::

      `fr` 
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Número del seguro social de Francia (INSEE)

    :::column-end:::
    :::column span="2":::

        Para obtener esta categoría de entidad, agregue `FRSocialSecurityNumber` al parámetro `pii-categories`. `FRSocialSecurityNumber` se devolverá en la respuesta de la API si se detecta.
      
        También se devuelve con `domain=phi`.
    :::column-end:::
    :::column span="":::

      `fr` 
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Número de identificación fiscal de Francia (número SPI)

    :::column-end:::
    :::column span="2":::

        Para obtener esta categoría de entidad, agregue `FRTaxIdentificationNumber` al parámetro `pii-categories`. `FRTaxIdentificationNumber` se devolverá en la respuesta de la API si se detecta.
      
    :::column-end:::
    :::column span="":::

      `fr` 
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Número de identificación fiscal (NIF) de Francia

    :::column-end:::
    :::column span="2":::

        Para obtener esta categoría de entidad, agregue `FRValueAddedTaxNumber` al parámetro `pii-categories`. `FRValueAddedTaxNumber` se devolverá en la respuesta de la API si se detecta.
      
    :::column-end:::
    :::column span="":::

      `fr` 
      
   :::column-end:::
:::row-end:::

#### <a name="germany"></a>Alemania

:::row:::
    :::column span="":::
        **Entidad**

        Número de permiso de conducir de Alemania

    :::column-end:::
    :::column span="2":::
        **Detalles**

        Para obtener esta categoría de entidad, agregue `DEDriversLicenseNumber` al parámetro `pii-categories`. `DEDriversLicenseNumber` se devolverá en la respuesta de la API si se detecta.
      
        También se devuelve con `domain=phi`.
    :::column-end:::
    :::column span="":::
      **Idiomas de documento admitidos**

      `de` 
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Número del carné de identidad de Alemania

    :::column-end:::
    :::column span="2":::

        Para obtener esta categoría de entidad, agregue `DEIdentityCardNumber` al parámetro `pii-categories`. `DEIdentityCardNumber` se devolverá en la respuesta de la API si se detecta.
      
        También se devuelve con `domain=phi`.
    :::column-end:::
    :::column span="":::

      `de`
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Número de pasaporte de Alemania

    :::column-end:::
    :::column span="2":::

        Para obtener esta categoría de entidad, agregue `DEPassportNumber` al parámetro `pii-categories`. `DEPassportNumber` se devolverá en la respuesta de la API si se detecta.
      
    :::column-end:::
    :::column span="":::

      `de` 
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Número de identificación fiscal de Alemania

    :::column-end:::
    :::column span="2":::

        Para obtener esta categoría de entidad, agregue `DETaxIdentificationNumber` al parámetro `pii-categories`. `DETaxIdentificationNumber` se devolverá en la respuesta de la API si se detecta.
      
    :::column-end:::
    :::column span="":::

      `de`
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Número de identificación fiscal de Alemania

    :::column-end:::
    :::column span="2":::
        **Detalles**

        Para obtener esta categoría de entidad, agregue `DEValueAddedNumber` al parámetro `pii-categories`. `DEValueAddedNumber` se devolverá en la respuesta de la API si se detecta.
      
    :::column-end:::
    :::column span="":::

      `de`
      
   :::column-end:::
:::row-end:::

#### <a name="hong-kong"></a>RAE de Hong Kong

:::row:::
    :::column span="":::
        **Entidad**

        Número del carné de identidad de Hong Kong (HKID)

    :::column-end:::
    :::column span="2":::
        **Detalles**

        Para obtener esta categoría de entidad, agregue `HKIdentityCardNumber` al parámetro `pii-categories`. `HKIdentityCardNumber` se devolverá en la respuesta de la API si se detecta.
      
        También se devuelve con `domain=phi`.
    :::column-end:::
    :::column span="":::
      **Idiomas de documento admitidos**

      `en`
      
   :::column-end:::
:::row-end:::

#### <a name="hungary"></a>Hungría

:::row:::
    :::column span="":::
        **Entidad**

        Número de identificación personal de Hungría

    :::column-end:::
    :::column span="2":::
        **Detalles**

        Para obtener esta categoría de entidad, agregue `HUPersonalIdentificationNumber` al parámetro `pii-categories`. `HUPersonalIdentificationNumber` se devolverá en la respuesta de la API si se detecta.
      
    :::column-end:::
    :::column span="":::
      **Idiomas de documento admitidos**

      `en`
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Número de identificación fiscal de Hungría

    :::column-end:::
    :::column span="2":::

        Para obtener esta categoría de entidad, agregue `HUTaxIdentificationNumber` al parámetro `pii-categories`. `HUTaxIdentificationNumber` se devolverá en la respuesta de la API si se detecta.
      
    :::column-end:::
    :::column span="":::

      `en`
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Número de identificación fiscal de Hungría

    :::column-end:::
    :::column span="2":::

        Para obtener esta categoría de entidad, agregue `HUValueAddedNumber` al parámetro `pii-categories`. `HUValueAddedNumber` se devolverá en la respuesta de la API si se detecta.
      
    :::column-end:::
    :::column span="":::

      `en`
      
   :::column-end:::
:::row-end:::

#### <a name="india"></a>India

:::row:::
    :::column span="":::
        **Entidad**

        Número de cuenta permanente de India (PAN)

    :::column-end:::
    :::column span="2":::
        **Detalles**

        Para obtener esta categoría de entidad, agregue `INPermanentAccount` al parámetro `pii-categories`. `INPermanentAccount` se devolverá en la respuesta de la API si se detecta.
      
        También se devuelve con `domain=phi`.
    :::column-end:::
    :::column span="":::
      **Idiomas de documento admitidos**

      `en`
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Número de identificación única de India (Aadhaar)

    :::column-end:::
    :::column span="2":::
        **Detalles**

        Para obtener esta categoría de entidad, agregue `INUniqueIdentificationNumber` al parámetro `pii-categories`. `INUniqueIdentificationNumber` se devolverá en la respuesta de la API si se detecta.
      
        También se devuelve con `domain=phi`.
    :::column-end:::
    :::column span="":::

      `en`
      
   :::column-end:::
:::row-end:::


#### <a name="indonesia"></a>Indonesia

:::row:::
    :::column span="":::
        **Entidad**

        Número del carné de identidad de Indonesia (KTP)

    :::column-end:::
    :::column span="2":::

        **Detalles**

        Para obtener esta categoría de entidad, agregue `IDIdentityCardNumber` al parámetro `pii-categories`. `IDIdentityCardNumber` se devolverá en la respuesta de la API si se detecta.
      
        También se devuelve con `domain=phi`.
    :::column-end:::
    :::column span="":::
      **Idiomas de documento admitidos**

      `en`
      
   :::column-end:::
:::row-end:::

#### <a name="ireland"></a>Irlanda

:::row:::
    :::column span="":::
        **Entidad**

        Número de servicio público personal de Irlanda (PPS)

    :::column-end:::
    :::column span="2":::
        **Detalles**

        Para obtener esta categoría de entidad, agregue `IEPersonalPublicServiceNumber` al parámetro `pii-categories`. `IEPersonalPublicServiceNumber` se devolverá en la respuesta de la API si se detecta.
      
        También se devuelve con `domain=phi`.
    :::column-end:::
    :::column span="":::
      **Idiomas de documento admitidos**

      `en`
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::
 
        Número de servicio público personal de Irlanda (PPS) versión 2

    :::column-end:::
    :::column span="2":::

        Para obtener esta categoría de entidad, agregue `IEPersonalPublicServiceNumberV2` al parámetro `pii-categories`. `IEPersonalPublicServiceNumberV2` se devolverá en la respuesta de la API si se detecta.
      
    :::column-end:::
    :::column span="":::
      **Idiomas de documento admitidos**

      `en`
      
   :::column-end:::
:::row-end:::

#### <a name="israel"></a>Israel

:::row:::
    :::column span="":::
        **Entidad**

        Identificación nacional

    :::column-end:::
    :::column span="2":::
        **Detalles**

        Para obtener esta categoría de entidad, agregue `ILNationalID` al parámetro `pii-categories`. `ILNationalID` se devolverá en la respuesta de la API si se detecta.
      
    :::column-end:::
    :::column span="":::
      **Idiomas de documento admitidos**

      `en`
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Número de cuenta bancaria de Israel

    :::column-end:::
    :::column span="2":::

        Para obtener esta categoría de entidad, agregue `ILBankAccountNumber` al parámetro `pii-categories`. `ILBankAccountNumber` se devolverá en la respuesta de la API si se detecta.
    
        También se devuelve con `domain=phi`.
    :::column-end:::
    :::column span="":::

      `en`
      
   :::column-end:::
:::row-end:::

#### <a name="italy"></a>Italia

:::row:::
    :::column span="":::
        **Entidad**

        Id. del permiso de conducir de Italia

    :::column-end:::
    :::column span="2":::
        **Detalles**

        Para obtener esta categoría de entidad, agregue `ITDriversLicenseNumber` al parámetro `pii-categories`. `ITDriversLicenseNumber` se devolverá en la respuesta de la API si se detecta.
      
        También se devuelve con `domain=phi`.
    :::column-end:::
    :::column span="":::
      **Idiomas de documento admitidos**

      `it`
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Código fiscal de Italia

    :::column-end:::
    :::column span="2":::

        Para obtener esta categoría de entidad, agregue `ITFiscalCode` al parámetro `pii-categories`. `ITFiscalCode` se devolverá en la respuesta de la API si se detecta.
      
    :::column-end:::
    :::column span="":::

      `it`
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Número de identificación fiscal de Italia

    :::column-end:::
    :::column span="2":::

        Para obtener esta categoría de entidad, agregue `ITValueAddedTaxNumber` al parámetro `pii-categories`. `ITValueAddedTaxNumber` se devolverá en la respuesta de la API si se detecta.
      
    :::column-end:::
    :::column span="":::

      `it`
      
   :::column-end:::
:::row-end:::


#### <a name="japan"></a>Japón

:::row:::
    :::column span="":::
        **Entidad**

        Número de cuenta bancaria de Japón

    :::column-end:::
    :::column span="2":::
        **Detalles**

        Para obtener esta categoría de entidad, agregue `JPBankAccountNumber` al parámetro `pii-categories`. `JPBankAccountNumber` se devolverá en la respuesta de la API si se detecta.
      
        También se devuelve con `domain=phi`.
    :::column-end:::
    :::column span="":::
      **Idiomas de documento admitidos**

      `ja`
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Número de permiso de conducir de Japón

    :::column-end:::
    :::column span="2":::

        Para obtener esta categoría de entidad, agregue `JPDriversLicenseNumber` al parámetro `pii-categories`. `JPDriversLicenseNumber` se devolverá en la respuesta de la API si se detecta.
      
        También se devuelve con `domain=phi`.
    :::column-end:::
    :::column span="":::

      `ja`
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Mi número personal de Japón

    :::column-end:::
    :::column span="2":::

        Para obtener esta categoría de entidad, agregue `JPMyNumberPersonal` al parámetro `pii-categories`. `JPMyNumberPersonal` se devolverá en la respuesta de la API si se detecta.
      
    :::column-end:::
    :::column span="":::

      `ja`
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Mi número corporativo de Japón

    :::column-end:::
    :::column span="2":::

        Para obtener esta categoría de entidad, agregue `JPMyNumberCorporate` al parámetro `pii-categories`. `JPMyNumberCorporate` se devolverá en la respuesta de la API si se detecta.
      
    :::column-end:::
    :::column span="":::

      `ja`
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Número de registro de residente de Japón

    :::column-end:::
    :::column span="2":::

        Para obtener esta categoría de entidad, agregue `ITValueAddedTaxNumber` al parámetro `pii-categories`. `ITValueAddedTaxNumber` se devolverá en la respuesta de la API si se detecta.
      
        También se devuelve con `domain=phi`.
    :::column-end:::
    :::column span="":::

     `ja`
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Número de la tarjeta de residencia de Japón

    :::column-end:::
    :::column span="2":::

        Para obtener esta categoría de entidad, agregue `JPResidenceCardNumber` al parámetro `pii-categories`. `JPResidenceCardNumber` se devolverá en la respuesta de la API si se detecta.
      
        También se devuelve con `domain=phi`.
    :::column-end:::
    :::column span="":::

      `ja`
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Número del seguro social (SIN) de Japón

    :::column-end:::
    :::column span="2":::

        Para obtener esta categoría de entidad, agregue `JPSocialInsuranceNumber` al parámetro `pii-categories`. `JPSocialInsuranceNumber` se devolverá en la respuesta de la API si se detecta.
      
        También se devuelve con `domain=phi`.
    :::column-end:::
    :::column span="":::

      `ja`
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Número de pasaporte de Japón

    :::column-end:::
    :::column span="2":::

        Para obtener esta categoría de entidad, agregue `JPPassportNumber` al parámetro `pii-categories`. `JPPassportNumber` se devolverá en la respuesta de la API si se detecta.
      
    :::column-end:::
    :::column span="":::

      `ja`
      
   :::column-end:::
:::row-end:::

#### <a name="luxembourg"></a>Luxemburgo

:::row:::
    :::column span="":::
        **Entidad**

        Número de identificación nacional de Luxemburgo (personas físicas)

    :::column-end:::
    :::column span="2":::
        **Detalles**

        Para obtener esta categoría de entidad, agregue `LUNationalIdentificationNumberNatural` al parámetro `pii-categories`. `LUNationalIdentificationNumberNatural` se devolverá en la respuesta de la API si se detecta.
      
    :::column-end:::
    :::column span="":::
      **Idiomas de documento admitidos**

      `fr`, `de`
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Número de identificación nacional de Luxemburgo (personas jurídicas)

    :::column-end:::
    :::column span="2":::

        Para obtener esta categoría de entidad, agregue `LUNationalIdentificationNumberNonNatural` al parámetro `pii-categories`. `LUNationalIdentificationNumberNonNatural` se devolverá en la respuesta de la API si se detecta.
      
    :::column-end:::
    :::column span="":::

      `fr`, `de`
      
   :::column-end:::
:::row-end:::

#### <a name="malta"></a>Malta

:::row:::
    :::column span="":::
        **Entidad**

        Número del carné de identidad de Malta

    :::column-end:::
    :::column span="2":::
        **Detalles**

        Para obtener esta categoría de entidad, agregue `MTIdentityCardNumber` al parámetro `pii-categories`. `MTIdentityCardNumber` se devolverá en la respuesta de la API si se detecta.
      
    :::column-end:::
    :::column span="":::
      **Idiomas de documento admitidos**

      `en`
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Número de identificación fiscal de Malta

    :::column-end:::
    :::column span="2":::

        Para obtener esta categoría de entidad, agregue `MTTaxIDNumber` al parámetro `pii-categories`. `MTTaxIDNumber` se devolverá en la respuesta de la API si se detecta.
      
    :::column-end:::
    :::column span="":::

      `en`
      
   :::column-end:::
:::row-end:::


#### <a name="new-zealand"></a>Nueva Zelanda

:::row:::
    :::column span="":::
        **Entidad**

        Número de cuenta bancaria de Nueva Zelanda

    :::column-end:::
    :::column span="2":::
        **Detalles**

        Para obtener esta categoría de entidad, agregue `NZBankAccountNumber` al parámetro `pii-categories`. `NZBankAccountNumber` se devolverá en la respuesta de la API si se detecta.
      
    :::column-end:::
    :::column span="":::
      **Idiomas de documento admitidos**

      `en`
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Número del permiso de conducir de Nueva Zelanda

    :::column-end:::
    :::column span="2":::

        Para obtener esta categoría de entidad, agregue `NZDriversLicenseNumber` al parámetro `pii-categories`. `NZDriversLicenseNumber` se devolverá en la respuesta de la API si se detecta.
      
    :::column-end:::
    :::column span="":::

      `en`
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Número de Hacienda Pública de Nueva Zelanda

    :::column-end:::
    :::column span="2":::

        Para obtener esta categoría de entidad, agregue `NZInlandRevenueNumber` al parámetro `pii-categories`. `NZInlandRevenueNumber` se devolverá en la respuesta de la API si se detecta.
      
    :::column-end:::
    :::column span="":::

      `en`
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Número del Ministerio de Sanidad de Nueva Zelanda

    :::column-end:::
    :::column span="2":::

        Para obtener esta categoría de entidad, agregue `NZMinistryOfHealthNumber` al parámetro `pii-categories`. `NZMinistryOfHealthNumber` se devolverá en la respuesta de la API si se detecta.
      
        También se devuelve con `domain=phi`.
    :::column-end:::
    :::column span="":::

      `en`
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

       Número de bienestar social de Nueva Zelanda

    :::column-end:::
    :::column span="2":::

        Para obtener esta categoría de entidad, agregue `NZSocialWelfareNumber` al parámetro `pii-categories`. `NZSocialWelfareNumber` se devolverá en la respuesta de la API si se detecta.
      
    :::column-end:::
    :::column span="":::

      `en`
      
   :::column-end:::
:::row-end:::


#### <a name="philippines"></a>Filipinas

:::row:::
    :::column span="":::
        **Entidad**

        Número de identificación multiuso unificado de Filipinas

    :::column-end:::
    :::column span="2":::
        **Detalles**

        Para obtener esta categoría de entidad, agregue `PHUnifiedMultiPurposeIDNumber` al parámetro `pii-categories`. `PHUnifiedMultiPurposeIDNumber` se devolverá en la respuesta de la API si se detecta.
      
        También se devuelve con `domain=phi`.
    :::column-end:::
    :::column span="":::
      **Idiomas de documento admitidos**

      `en`
      
   :::column-end:::
:::row-end:::

#### <a name="portugal"></a>Portugal 

:::row:::
    :::column span="":::
        **Entidad**

        Número de la tarjeta de ciudadano de Portugal

    :::column-end:::
    :::column span="2":::
        **Detalles**

        Para obtener esta categoría de entidad, agregue `PTCitizenCardNumber` al parámetro `pii-categories`. `PTCitizenCardNumber` se devolverá en la respuesta de la API si se detecta.
          
        También se devuelve con `domain=phi`.
    :::column-end:::
    :::column span="":::
      **Idiomas de documento admitidos**

      `pt-pt`
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

       Número de identificación fiscal de Portugal

    :::column-end:::
    :::column span="2":::

        Para obtener esta categoría de entidad, agregue `PTTaxIdentificationNumber` al parámetro `pii-categories`. `PTTaxIdentificationNumber` se devolverá en la respuesta de la API si se detecta.
      
        También se devuelve con `domain=phi`.
    :::column-end:::
    :::column span="":::

      `pt-pt`
      
   :::column-end:::
:::row-end:::

#### <a name="singapore"></a>Singapur

:::row:::
    :::column span="":::
        **Entidad**

        Número de la tarjeta de identificación del registro nacional de Singapur (NRIC)

    :::column-end:::
    :::column span="2":::
        **Detalles**

        Para obtener esta categoría de entidad, agregue `PTTaxIdentificationNumber` al parámetro `pii-categories`. `PTTaxIdentificationNumber` se devolverá en la respuesta de la API si se detecta.
      
    :::column-end:::
    :::column span="":::
      **Idiomas de documento admitidos**

      `en`, `zh-hans`
      
   :::column-end:::
:::row-end:::


#### <a name="south-africa"></a>Sudáfrica

:::row:::
    :::column span="":::
        **Entidad**

        Número de identificación de Sudáfrica

    :::column-end:::
    :::column span="2":::
        **Detalles**

        Para obtener esta categoría de entidad, agregue `ZAIdentificationNumber` al parámetro `pii-categories`. `ZAIdentificationNumber` se devolverá en la respuesta de la API si se detecta.
      
        También se devuelve con `domain=phi`.
    :::column-end:::
    :::column span="":::
      **Idiomas de documento admitidos**

      `en`
      
   :::column-end:::
:::row-end:::


#### <a name="south-korea"></a>Corea del Sur

:::row:::
    :::column span="":::
        **Entidad**

        Número de registro de residente de Corea del Sur

    :::column-end:::
    :::column span="2":::
        **Detalles**

        Para obtener esta categoría de entidad, agregue `KRResidentRegistrationNumber` al parámetro `pii-categories`. `KRResidentRegistrationNumber` se devolverá en la respuesta de la API si se detecta.
      
        También se devuelve con `domain=phi`.
    :::column-end:::
    :::column span="":::
      **Idiomas de documento admitidos**

      `ko`
      
   :::column-end:::
:::row-end:::

#### <a name="spain"></a>España

:::row:::
    :::column span="":::
        **Entidad**

        DNI de España

    :::column-end:::
    :::column span="2":::
        **Detalles**

        Para obtener esta categoría de entidad, agregue `ESDNI` al parámetro `pii-categories`. `ESDNI` se devolverá en la respuesta de la API si se detecta.
      
    :::column-end:::
    :::column span="":::
      **Idiomas de documento admitidos**

      `es`
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Número de seguridad social de España (SSN)

    :::column-end:::
    :::column span="2":::

        Para obtener esta categoría de entidad, agregue `ESSocialSecurityNumber` al parámetro `pii-categories`. `ESSocialSecurityNumber` se devolverá en la respuesta de la API si se detecta.
      
        También se devuelve con `domain=phi`.
    :::column-end:::
    :::column span="":::

      `es`
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Número de identificación fiscal de España

    :::column-end:::
    :::column span="2":::

        Para obtener esta categoría de entidad, agregue `ESTaxIdentificationNumber` al parámetro `pii-categories`. `ESTaxIdentificationNumber` se devolverá en la respuesta de la API si se detecta.
      
    :::column-end:::
    :::column span="":::

      `es`
      
   :::column-end:::
:::row-end:::
 
#### <a name="switzerland"></a>Suiza

:::row:::
    :::column span="":::
        **Entidad**

        Número del seguro social de Suiza (AHV)

    :::column-end:::
    :::column span="2":::
        **Detalles**

        Para obtener esta categoría de entidad, agregue `CHSocialSecurityNumber` al parámetro `pii-categories`. `CHSocialSecurityNumber` se devolverá en la respuesta de la API si se detecta.
      
    :::column-end:::
    :::column span="":::
      **Idiomas de documento admitidos**

      `fr`, `de`, `it`
      
   :::column-end:::
:::row-end:::


#### <a name="taiwan"></a>Taiwán 

:::row:::
    :::column span="":::
        **Entidad**

        Identificación nacional de Ta

    :::column-end:::
    :::column span="2":::
        **Detalles**

        Para obtener esta categoría de entidad, agregue `TWNationalID` al parámetro `pii-categories`. `TWNationalID` se devolverá en la respuesta de la API si se detecta.
      
        También se devuelve con `domain=phi`.
    :::column-end:::
    :::column span="":::
      **Idiomas de documento admitidos**

      `en`
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

       Certificado de residente de Taiwán (ARC/TARC)

    :::column-end:::
    :::column span="2":::

        Para obtener esta categoría de entidad, agregue `TWResidentCertificate` al parámetro `pii-categories`. `TWResidentCertificate` se devolverá en la respuesta de la API si se detecta.
      
        También se devuelve con `domain=phi`.
    :::column-end:::
    :::column span="":::

      `en`
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Número de pasaporte de Taiwán

    :::column-end:::
    :::column span="2":::

        Para obtener esta categoría de entidad, agregue `TWPassportNumber` al parámetro `pii-categories`. `TWPassportNumber` se devolverá en la respuesta de la API si se detecta.
      
    :::column-end:::
    :::column span="":::

      `en`
      
   :::column-end:::
:::row-end:::

#### <a name="united-kingdom"></a>Reino Unido

:::row:::
    :::column span="":::
        **Entidad**

        Reino Unido Número de permiso de conducir

    :::column-end:::
    :::column span="2":::
        **Detalles**

        Para obtener esta categoría de entidad, agregue `UKDriversLicenseNumber` al parámetro `pii-categories`. `UKDriversLicenseNumber` se devolverá en la respuesta de la API si se detecta.
      
        También se devuelve con `domain=phi`.
    :::column-end:::
    :::column span="":::
      **Idiomas de documento admitidos**

      `en`
      
    :::column-end:::
    
:::row-end:::
:::row:::
    :::column span="":::

       Reino Unido Número de lista electoral

    :::column-end:::
    :::column span="2":::

        Para obtener esta categoría de entidad, agregue `UKNationalInsuranceNumber` al parámetro `pii-categories`. `UKNationalInsuranceNumber` se devolverá en la respuesta de la API si se detecta.
      
        También se devuelve con `domain=phi`.
    :::column-end:::
    :::column span="":::

      `en`
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

       Reino Unido Número del servicio de salud nacional (NHS)

    :::column-end:::
    :::column span="2":::

        Para obtener esta categoría de entidad, agregue `UKNationalHealthNumber` al parámetro `pii-categories`. `UKNationalHealthNumber` se devolverá en la respuesta de la API si se detecta.
      
        También se devuelve con `domain=phi`.
    :::column-end:::
    :::column span="":::

      `en`
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

       Reino Unido Número del seguro social (NINO)

    :::column-end:::
    :::column span="2":::

        Para obtener esta categoría de entidad, agregue `UKNationalInsuranceNumber` al parámetro `pii-categories`. `UKNationalInsuranceNumber` se devolverá en la respuesta de la API si se detecta.
      
        También se devuelve con `domain=phi`.
    :::column-end:::
    :::column span="":::

      `en`
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

       Reino Unido o Estados Unidos Número de pasaporte

    :::column-end:::
    :::column span="2":::

        Para obtener esta categoría de entidad, agregue `USUKPassportNumber` al parámetro `pii-categories`. `USUKPassportNumber` se devolverá en la respuesta de la API si se detecta.
      
    :::column-end:::
    :::column span="":::

      `en`
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

       Reino Unido Número único de referencia del contribuyente

    :::column-end:::
    :::column span="2":::

        Para obtener esta categoría de entidad, agregue `UKUniqueTaxpayerNumber` al parámetro `pii-categories`. `UKUniqueTaxpayerNumber` se devolverá en la respuesta de la API si se detecta.
      
    :::column-end:::
    :::column span="":::

      `en`
      
   :::column-end:::
:::row-end:::


#### <a name="united-states"></a>Estados Unidos

:::row:::
    :::column span="":::
        **Entidad**

        EE. UU. Número de seguridad social

    :::column-end:::
    :::column span="2":::
        **Detalles**

        Para obtener esta categoría de entidad, agregue `USSocialSecurityNumber` al parámetro `pii-categories`. `USSocialSecurityNumber` se devolverá en la respuesta de la API si se detecta.
      
        También se devuelve con `domain=phi`.
    :::column-end:::
    :::column span="":::
      **Idiomas de documento admitidos**

      `en`
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

       EE. UU. Número de permiso de conducir

    :::column-end:::
    :::column span="2":::

        Para obtener esta categoría de entidad, agregue `USDriversLicenseNumber` al parámetro `pii-categories`. `USDriversLicenseNumber` se devolverá en la respuesta de la API si se detecta.
      
        También se devuelve con `domain=phi`.
    :::column-end:::
    :::column span="":::

      `en`
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

       Estados Unidos o Reino Unido Número de pasaporte

    :::column-end:::
    :::column span="2":::

        Para obtener esta categoría de entidad, agregue `USUKPassportNumber` al parámetro `pii-categories`. `USUKPassportNumber` se devolverá en la respuesta de la API si se detecta.
      
    :::column-end:::
    :::column span="":::

      `en`
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

       EE. UU. Número de identificación fiscal (NIF) individual

    :::column-end:::
    :::column span="2":::

        Para obtener esta categoría de entidad, agregue `USIndividualTaxpayerIdentification` al parámetro `pii-categories`. `USIndividualTaxpayerIdentification` se devolverá en la respuesta de la API si se detecta.
      
    :::column-end:::
    :::column span="":::

      `en`
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

       EE. UU. Número de la agencia antidroga de Estados Unidos (DEA)

    :::column-end:::
    :::column span="2":::

        Para obtener esta categoría de entidad, agregue `DrugEnforcementAgencyNumber` al parámetro `pii-categories`. `DrugEnforcementAgencyNumber` se devolverá en la respuesta de la API si se detecta.
      
        También se devuelve con `domain=phi`.
    :::column-end:::
    :::column span="":::

      `en`
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

       EE. UU. Número de cuenta bancaria

    :::column-end:::
    :::column span="2":::

        Para obtener esta categoría de entidad, agregue `USBankAccountNumber` al parámetro `pii-categories`. `USBankAccountNumber` se devolverá en la respuesta de la API si se detecta.
        
        También se devuelve con `domain=phi`.
    :::column-end:::
    :::column span="":::

      `en`
      
   :::column-end:::
:::row-end:::
