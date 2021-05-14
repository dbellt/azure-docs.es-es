---
title: Disponibilidad de regiones y residencia de datos
titleSuffix: Azure AD B2C
description: Disponibilidad de regiones, residencia de datos e información sobre los inquilinos de versión preliminar de Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: reference
ms.date: 04/27/2021
ms.author: mimart
ms.subservice: B2C
ms.custom: references_regions
ms.openlocfilehash: 2d4ecf925eadd00c40685fbee413df6fca0368d9
ms.sourcegitcommit: 2e123f00b9bbfebe1a3f6e42196f328b50233fc5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/27/2021
ms.locfileid: "108070864"
---
# <a name="azure-active-directory-b2c-region-availability--data-residency"></a>Azure Active Directory B2C: Disponibilidad de región y residencia de datos

Los datos de identidad de Azure AD B2C se almacenan en una ubicación geográfica en función del país o de la región proporcionados al crear el inquilino.

La disponibilidad por región y la residencia de datos son dos conceptos diferentes que se aplican a Azure AD B2C. En este artículo se explican las diferencias entre estos dos conceptos y se compara cómo se aplican a Azure frente a Azure AD B2C.

Azure AD B2C suele estar **disponible en todo el mundo** con la opción de **residencia de datos** en **Estados Unidos, Europa o Asia Pacifico**. Azure AD B2C está en **versión preliminar pública** en Australia.

[Disponibilidad en regiones ](#region-availability) se refiere a que un servicio esté disponible para su uso.

[Residencia de datos](#data-residency) hace referencia a dónde se almacenan los datos de usuario.

## <a name="region-availability"></a>Disponibilidad en regiones

Azure AD B2C está disponible en todo el mundo a través de la nube pública de Azure. Puede ver ejemplos de esta característica en la página [Productos disponibles por región](https://azure.microsoft.com/regions/services/) de Azure y en la [calculadora de precios de Active Directory B2C](https://azure.microsoft.com/pricing/details/active-directory-b2c/).

## <a name="data-residency"></a>Residencia de datos

Azure AD B2C almacena los datos de usuarios en la región de Estados Unidos, Europa o Asia Pacifico.

La residencia de datos viene determinada por el país o la región que seleccione cuando [cree un inquilino de Azure AD B2C](tutorial-create-tenant.md):

![Captura de pantalla de un formulario Crear inquilino, con la elección de país o región.](./media/data-residency/data-residency-b2c-tenant.png)

Los datos residen en **Estados Unidos** para los países o regiones siguientes:

> Estados Unidos (EE. UU.), Canadá (CA), Costa Rica (CR), República Dominicana (DO), El Salvador (SV), Guatemala (GT), México (MX), Panamá (PA), Puerto Rico (PR) y Trinidad y Tobago (TT)

Los datos residen en **Europa** para los países o regiones siguientes:

> Argelia (DZ), Austria (AT), Azerbaiyán (AZ), Bahrein (BH), Bielorrusia (BY), Bélgica (ser), Bulgaria (BG), Croacia (as), Chipre (CY), República Checa (CZ), Dinamarca (DK), Egipto (EG), Estonia (EE), Finlandia (FT), Francia (FR), Alemania (DE), Grecia (GR), Hungría (HU), Islandia (IS), Irlanda (IE), Israel (IL), Italia (TI), Jordania (JO), Kazajstán (KZ), Kenia (KE), Kuwait (KW), Letonia (LV), Líbano (LB), Liechtenstein (LI), Lituania (LT), Luxemburgo (LU), Macedonia del Norte (ML), Malta (MT), Montenegro (ME), Marruecos (MA), Países Bajos (NL), Nigeria (NG), Noruega (NO), Omán (OM), Pakistán (PK), Polonia (PL), Portugal (PT), Qatar (QA), Rumania (RO), Rusia (RU), Arabia Saudí (SA), Serbia (RS), Eslovaquia (SK), Eslovenia (ST), Sudáfrica (ZA), España (ES), Suecia (SE), Suiza (CH), Túnez (TN), Turquía (TR), Ucrania (UA), Emiratos Árabes Unidos (AE) y Reino Unido (GB)

Los datos residen en **Asia Pacifico** para los países o regiones siguientes:

> Afganistán (AF), Hong Kong ZAE (HK), India (IN), Indonesia (ID), Japón (JP), Corea (KR), Malasia (MY), Filipinas (PH), Singapur (SG), Sri Lanka (LK), Taiwán (TW) y Tailandia (TH)

Los datos residen en **Australia** (versión preliminar) para los países o regiones siguientes:

> Australia y Nueva Zelanda

Los países o regiones siguientes están a punto de agregarse a la lista. De momento, todavía puede usar Azure AD B2C escogiendo cualquiera de los países o regiones anteriores.

> Argentina, Brasil, Chile, Colombia, Ecuador, Iraq, Paraguay, Perú, Uruguay y Venezuela

## <a name="remote-profile-solution"></a>Solución de perfil remoto

Con las [directivas personalizadas](custom-policy-overview.md) de Azure AD B2C, puede realizar la integración con los [servicios de la API RESTful](api-connectors-overview.md), lo cual le permite almacenar y leer perfiles de usuario desde una base de datos remota (como, por ejemplo, una base de datos de marketing, un sistema CRM o cual aplicación de línea de negocio).  
- Durante los flujos de edición de perfiles y de registro, Azure AD B2C llama a una API REST personalizada para conservar el perfil de usuario en el origen de datos remoto. Las credenciales del usuario se almacenan en el directorio de Azure AD B2C. 
- Tras el inicio de sesión, después de la validación de credenciales con una cuenta local o de red social, Azure AD B2C invoca a la API REST, la cual envía el identificador único del usuario como clave principal de usuario (dirección de correo electrónico u objectId de usuario). La API REST lee los datos de la base de datos remota y devuelve el perfil de usuario.  

Una vez que se ha completado el registro, la edición de perfiles o el inicio de sesión, Azure AD B2C incluye el perfil de usuario en el token de acceso que se devuelve a la aplicación. Para más información, consulte la [solución de ejemplo de perfil remoto de Azure AD B2C](https://github.com/azure-ad-b2c/samples/tree/master/policies/remote-profile) en GitHub.

## <a name="next-steps"></a>Pasos siguientes

- [Crear un inquilino de Azure AD B2C](tutorial-create-tenant.md).
