---
title: Conjuntos de notificaciones de Azure Attestation
description: Los conjuntos de notificaciones de Azure Attestation.
services: attestation
author: msmbaldwin
ms.service: attestation
ms.topic: overview
ms.date: 08/31/2020
ms.author: mbaldwin
ms.openlocfilehash: e82e9fc93bf8c816fcbfd5869156745dea630313
ms.sourcegitcommit: db925ea0af071d2c81b7f0ae89464214f8167505
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2021
ms.locfileid: "107517562"
---
# <a name="claim-sets"></a>Conjuntos de notificaciones

Las notificaciones generadas en el proceso de atestación de enclaves mediante Microsoft Azure Attestation se pueden dividir en las siguientes categorías:

- **Notificaciones entrantes**: las notificaciones que genera Microsoft Azure Attestation después de analizar la evidencia de atestación; lo pueden usar los creadores de directivas para definir las reglas de autorización en una directiva personalizada.

- **Notificaciones salientes**: las notificaciones que genera Azure Attestation y que se incluyen en el token de atestación.

- **Notificaciones de propiedad**: notificaciones que Azure Attestation crea como salida. Contiene todas las notificaciones que representan las propiedades del token de atestación, como la codificación del informe, la duración de la validez del informe, etc.

## <a name="incoming-claims"></a>Notificaciones entrantes 

### <a name="sgx-attestation"></a>Atestación de SGX

Notificaciones que deben usar los creadores de directivas para definir las reglas de autorización en una directiva de atestación de SGX:

- **x-ms-sgx-is-debuggable**: valor booleano que indica si el enclave tiene o no habilitada la depuración.
  
  Los enclaves de SGX se pueden cargar con la depuración deshabilitada o habilitada. Cuando la marca se establece en true en el enclave, se habilitan las características de depuración para el código del enclave. Esto incluye la posibilidad de acceder a la memoria del enclave. Por lo tanto, se recomienda establecer la marca en true solo con fines de desarrollo. Si está habilitada en el entorno de producción, las garantías de seguridad de SGX no se conservarán.
  
  Los usuarios de Azure Attestation pueden usar la directiva de atestación para comprobar si la depuración está deshabilitada para el enclave de SGX. Después de agregar la regla de directiva, se producirá un error en la atestación cuando un usuario malintencionado active la compatibilidad con la depuración para obtener acceso al contenido del enclave.

- **x-ms-sgx-product-id**: un valor entero, que indica el identificador de producto del enclave de SGX.

  El autor del enclave asigna un identificador de producto a cada enclave. Dicho identificador permite al autor del enclave segmentar los enclaves firmados con el mismo valor MRSIGNER. Al agregar una regla de validación en la directiva de atestación, los clientes pueden comprobar si están usando los enclaves previstos. Si el identificador de producto del enclave no coincide con el valor publicado por el autor del enclave, se producirá un error en la atestación.

- **x-ms-sgx-mrsigner**: valor de cadena que identifica al autor del enclave de SGX.

  MRSIGNER es el hash de la clave pública del autor del enclave que se usa para firmar el archivo binario del enclave. Al validar MRSIGNER mediante una directiva de atestación, los clientes pueden comprobar si se ejecutan archivos binarios de confianza dentro de un enclave. Cuando la notificación de directiva no coincide con el valor MRSIGNER del autor del enclave, significa que el archivo binario del enclave no está firmado por un origen de confianza y se produce un error en la atestación.
  
  Cuando, por motivos de seguridad, el autor del enclave prefiere rotar MRSIGNER, la directiva de Azure Attestation debe actualizarse para admitir los valores de MRSIGNER nuevos y antiguos antes de que se actualicen los archivos binarios. De lo contrario, las comprobaciones de autorización producirán errores de atestación.
  
  La directiva de atestación debe actualizarse con el formato siguiente. 
 
  #### <a name="before-key-rotation"></a>Antes de la rotación de claves
 
   ```
    version= 1.0;
    authorizationrules 
    {
    [ type=="x-ms-sgx-is-debuggable", value==false]&&
    [ type=="x-ms-sgx-mrsigner", value=="mrsigner1"] => permit(); 
    };
  ```

   #### <a name="during-key-rotation"></a>Durante la rotación de claves

    ```
      version= 1.0;
      authorizationrules 
      {
      [ type=="x-ms-sgx-is-debuggable", value==false]&&
      [ type=="x-ms-sgx-mrsigner", value=="mrsigner1"] => permit(); 
      [ type=="x-ms-sgx-is-debuggable", value==false ]&& 
      [ type=="x-ms-sgx-mrsigner", value=="mrsigner2"] => permit(); 
      };
    ```

   #### <a name="after-key-rotation"></a>Después de la rotación de claves

    ```
      version= 1.0;
      authorizationrules 
      { 
      [ type=="x-ms-sgx-is-debuggable", value==false]&& 
      [ type=="x-ms-sgx-mrsigner", value=="mrsigner2"] => permit(); 
      };
    ```

- **x-ms-sgx-mrenclave**: valor de cadena que identifica el código y los datos cargados en la memoria del enclave. 

  MRENCLAVE es una de las medidas de enclave que se pueden usar para comprobar los archivos binarios del enclave. Es el hash del código que se ejecuta dentro del enclave. La medida cambia con cada cambio en el código binario del enclave. Al validar MRENCLAVE a través de una directiva de atestación, los clientes pueden comprobar si los archivos binarios previstos se ejecutan dentro de un enclave. Sin embargo, como se espera que MRENCLAVE cambie con frecuencia con cualquier modificación del código existente, por mínima que sea, se recomienda comprobar los archivos binarios del enclave mediante la validación de MRSIGNER en una directiva de atestación.

- **x-ms-sgx-svn**: un valor entero que indica el número de versión de seguridad del enclave de SGX.

  El autor del enclave asigna un número de versión de seguridad (SVN) a cada versión del enclave de SGX. Cuando se detecta un problema de seguridad en el código del enclave, el autor del enclave aumenta el valor de SVN después de corregir la vulnerabilidad. Para evitar la interacción con código de enclave poco seguro, los clientes pueden agregar una regla de validación en la directiva de atestación. Si el SVN del código del enclave no coincide con la versión recomendada por el autor del enclave, se producirá un error en la atestación.

Las siguientes notificaciones se consideran en desuso, pero se admiten por completo y seguirán estando incluidas en el futuro. Se recomienda usar los nombres de notificaciones que no estén en desuso.

Notificación en desuso | Notificación recomendada
--- | --- 
$is-debuggable | x-ms-sgx-is-debuggable
$product-id | x-ms-sgx-product-id
$sgx-mrsigner | x-ms-sgx-mrsigner
$sgx-mrenclave | x-ms-sgx-mrenclave
$svn | x-ms-sgx-svn

### <a name="tpm-attestation"></a>Atestación de TPM

Notificaciones que deben usar los creadores de directivas para definir las reglas de autorización en una directiva de atestación de TPM:

- **aikValidated**: valor booleano que indica si se ha validado o no el certificado de la clave de identidad de la atestación (AIK).
- **aikPubHash**:  cadena que contiene el hash SHA256 en base64 (clave pública de identidad de atestación en formato DER).
- **tpmVersion**:   valor entero que contiene la versión principal del Módulo de plataforma segura (TPM).
- **secureBootEnabled**: valor booleano que indica si está habilitado el arranque seguro.
- **iommuEnabled**:  valor booleano que indica si está habilitada la unidad de administración de memoria de entrada y salida (IOMMU).
- **bootDebuggingDisabled**: valor booleano que indica si está deshabilitada la depuración de arranque.
- **notSafeMode**:  valor booleano que indica si Windows no se ejecuta en modo seguro.
- **notWinPE**:  valor booleano que indica si Windows no se ejecuta en modo WinPE.
- **vbsEnabled**:  valor booleano que indica si VBS está habilitado.
- **vbsReportPresent**:  valor booleano que indica si está disponible el informe de enclaves de VBS.

### <a name="vbs-attestation"></a>Atestación de VBS

Además de las notificaciones de directivas de atestación de TPM, los creadores de directivas pueden usar las notificaciones siguientes para definir reglas de autorización en una directiva de atestación de VBS.

- **enclaveAuthorId**:  valor de cadena que contiene el valor codificado en Base64Url del identificador de autor del enclave; es decir, el identificador de autor del módulo principal del enclave.
- **enclaveImageId**: valor de cadena que contiene el valor codificado en Base64Url del identificador de imagen del enclave, es decir, el identificador de imagen del módulo principal del enclave.
- **enclaveOwnerId**: valor de cadena que contiene el valor codificado en Base64Url del identificador de propietario del enclave.
- **enclaveFamilyId**:  valor de cadena que contiene el valor codificado en Base64Url del identificador de familia del enclave; es decir, el identificador de familia del módulo principal del enclave.
- **enclaveSvn**:  valor entero que contiene el número de versión de seguridad del módulo principal del enclave.
- **enclavePlatformSvn**:  valor entero que contiene el número de versión de seguridad de la plataforma que hospeda el enclave.
- **enclaveFlags**:  la notificación enclaveFlags es un valor entero que contiene marcas que describen la directiva de tiempo de ejecución del enclave.

## <a name="outgoing-claims"></a>Notificaciones salientes 

### <a name="common-for-all-attestation-types"></a>Común para todos los tipos de atestación

Azure Attestation incluye las siguientes notificaciones en el token de atestación de todos los tipos de atestación. 

- **x-ms-ver**: versión de esquema de JWT (se espera que sea "1.0").
- **x-ms-attestation-type**: valor de cadena que representa el tipo de atestación. 
- **x-ms-policy-hash**: hash de la directiva de evaluación de Azure Attestation calculada como BASE64URL (SHA256 (UTF8 (BASE64URL (UTF8 (texto de directiva))))
- **x-ms-policy-signer**: objeto JSON con un miembro "jwk" que representa la clave que un cliente usó para firmar su directiva. Se puede aplicar cuando el cliente carga una directiva firmada

Los nombres de notificaciones siguientes se usan en la [especificación JWT de IETF](https://tools.ietf.org/html/rfc7519)

- **Notificación "jti" (ID. de JWT)** : identificador único para el JSON Web Token.
- **Notificación "iss" (emisor)** : entidad de seguridad que ha emitido el JSON Web Token. 
- **Notificación "iat" (emitida a las)** : hora a la que se ha emitido el JSON Web Token 
- **Notificación "exp" (hora de expiración)** : hora de expiración tras la que el JSON Web Token no debe aceptarse para su procesamiento.
- **Notificación "nbf" (no antes de)** : hora antes de la cual no debe aceptarse el JSON Web Token para su procesamiento. 

Los nombres de notificaciones siguientes se usan en la [especificación del borrador EAT de IETF](https://tools.ietf.org/html/draft-ietf-rats-eat-03#page-9)

- **"Notificación nonce" (nonce)** : una copia directa no transformada de un valor de nonce opcional proporcionado por un cliente 

Las siguientes notificaciones se consideran en desuso, pero se admiten por completo y seguirán estando incluidas en el futuro. Se recomienda usar los nombres de notificaciones que no estén en desuso.

Notificación en desuso | Notificación recomendada
--- | --- 
ver | x-ms-ver
tee | x-ms-attestation-type
policy_hash | x-ms-policy-hash
maa-policyHash | x-ms-policy-hash
policy_signer  | x-ms-policy-signer

### <a name="sgx-attestation"></a>Atestación de SGX 

El servicio genera las siguientes notificaciones y las incluyen en el token de atestación para la atestación de SGX.

- **x-ms-sgx-is-debuggable**: valor booleano que indica si el enclave tiene habilitada la depuración o no
- **x-ms-sgx-product-id**: valor de identificador del producto del enclave de SGX 
- **sgx-mrsigner**: valor hexadecimal codificado del campo "mrsigner" de la oferta.
- **x-ms-sgx-mrenclave**: valor hexadecimal codificado del campo "mrenclave" de la oferta.
- **x-ms-sgx-svn**: número de versión de seguridad codificado en la oferta. 
- **x-MS-SGX-EHD**: datos contenidos en el enclave con el formato BASE64URL (datos contenidos en el encave).
- **x-ms-sgx-collateral**: objeto JSON que describe el colateral que se usa para realizar la atestación. El valor de la notificación colateral x-ms-sgx es un objeto JSON anidado con los siguientes pares clave-valor:
    - **qeidcertshash**: valor SHA256 de los certificados de emisión de identidades de Quoting Enclave (QE)
    - **qeidcrlhash**: valor SHA256 de la lista CRL de certificados de emisión de identidades de QE.
    - **qeidhash**: valor SHA256 del colateral de identidad de QE.
    - **quotehash**: valor SHA256 del presupuesto evaluado.
    - **tcbinfocertshash**: valor SHA256 de los certificados de emisión de información de TCB.
    - **tcbinfocrlhash**: valor SHA256 de la lista CRL de certificados de emisión de información de TCB.
    - **tcbinfohash**: valor SHA256 de la documentación y material adjunto de la información de TCB

Las siguientes notificaciones se consideran en desuso, pero se admiten por completo y seguirán estando incluidas en el futuro. Se recomienda usar los nombres de notificaciones que no estén en desuso.

Notificación en desuso | Notificación recomendada
--- | --- 
$is-debuggable | x-ms-sgx-is-debuggable
$product-id | x-ms-sgx-product-id
$sgx-mrsigner | x-ms-sgx-mrsigner
$sgx-mrenclave | x-ms-sgx-mrenclave
$svn | x-ms-sgx-svn
$maa-ehd | x-ms-sgx-ehd
$aas-ehd | x-ms-sgx-ehd
$maa-attestationcollateral | x-ms-sgx-collateral

### <a name="tpm-and-vbs-attestation"></a>Atestación de TPM y VBS

- **cnf (confirmación)** : esta notificación se usa para identificar la clave de la prueba de posesión. La notificación de confirmación, tal como se define en RFC 7800, contiene la parte pública de la clave del enclave atestado que se representa como un objeto de clave web JSON (JWK) (RFC 7517).
- **rp_data (datos de usuario de confianza)**: datos de usuario de confianza, si los hay, especificados en la solicitud, que utiliza este usuario como nonce para garantizar que el informe está actualizado. rp_data solo se agrega si hay rp_data

## <a name="property-claims"></a>Notificaciones de propiedad

### <a name="tpm-and-vbs-attestation"></a>Atestación de TPM y VBS

- **report_validity_in_minutes**: notificación de entero que indica el periodo de validez del token.
  - **Valor predeterminado (tiempo)**: un día, expresado en minutos.
  - **Valor máximo (tiempo)**: un año, expresado en minutos.
- **omit_x5c**: notificación booleana que indica si Azure Attestation debe omitir el certificado usado para proporcionar la autenticidad de la prueba de servicio. Si es True, se agrega x5t al token de atestación. Si es False (valor predeterminado), se agregar x5c al token de atestación.

## <a name="next-steps"></a>Pasos siguientes
- [Creación y firma de una directiva de atestación](author-sign-policy.md)
- [Configuración de Azure Attestation con PowerShell](quickstart-powershell.md)
