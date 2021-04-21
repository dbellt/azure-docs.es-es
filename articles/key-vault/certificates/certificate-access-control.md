---
title: Acerca del control de acceso de los certificados de Azure Key Vault
description: Información general del control de acceso de certificados de Azure Key Vault
services: key-vault
author: sebansal
tags: azure-resource-manager
ms.service: key-vault
ms.subservice: certificates
ms.topic: overview
ms.date: 10/12/2020
ms.author: sebansal
ms.openlocfilehash: 54874f30384d7f4827b13a597a469bfc67bc8fd2
ms.sourcegitcommit: 6686a3d8d8b7c8a582d6c40b60232a33798067be
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107752487"
---
# <a name="certificate-access-control"></a>Control de acceso al certificado

 El control de acceso para los certificados lo administra Key Vault y lo proporciona la instancia de Key Vault que contiene dichos certificados. La directiva de control de acceso para los certificados es distinta de la directiva de control de acceso para las claves y los secretos en la misma instancia de Key Vault. Los usuarios pueden crear uno o varios almacenes para almacenar los certificados, para mantener la segmentación adecuada del escenario y la administración de los certificados.  

 Los siguientes permisos se pueden utilizar, en una base por entidad, en la entrada de control de acceso de secretos en un almacén de claves y reflejan fielmente las operaciones permitidas en un objeto de secreto:  

- Permisos para operaciones de administración de certificados
  - **get**: obtener la versión actual del certificado o cualquier versión de un certificado
  - **list**: enumerar los certificados actuales o las versiones de un certificado  
  - **update**: crear un certificado
  - **create**: crear un certificado de Key Vault
  - **import**: importar material del certificado en un certificado de Key Vault
  - **delete**: eliminar un certificado, su directiva y todas sus versiones  
  - **recover**: recuperar un certificado eliminado
  - **backup**: hacer una copia de seguridad de un certificado de un almacén de claves.
  - **restore**: restaurar una copia de seguridad de un certificado a un almacén de claves
  - **managecontacts**: administrar los contactos del certificado de Key Vault  
  - **manageissuers**: administrar emisores y entidades de certificados de Key Vault
  - **getissuers**: obtener emisores y entidades de un certificado
  - **listissuers**: enumerar emisores y entidades de un certificado  
  - **setissuers**: crear o actualizar emisores y entidades de un certificado de Key Vault  
  - **deleteissuers**: eliminar emisores y entidades de un certificado de Key Vault  
 
- Permisos para las operaciones con privilegios
  - **purge**: purgar (eliminar permanentemente) un certificado eliminado

Para más información, consulte las [operaciones con certificados en la referencia de la API REST de Key Vault](/rest/api/keyvault). Para información sobre cómo establecer permisos, consulte [Almacenes: actualización de la directiva de acceso](/rest/api/keyvault/vaults/updateaccesspolicy).

## <a name="troubleshoot"></a>Solución de problemas
Es posible que vea un error porque falta la directiva de acceso. Por ejemplo ```Error type : Access denied or user is unauthorized to create certificate```. Para resolverlo, tendría que agregar certificados o crear permisos.

## <a name="next-steps"></a>Pasos siguientes

- [Acerca de Key Vault](../general/overview.md)
- [Información acerca de claves, secretos y certificados](../general/about-keys-secrets-certificates.md)
- [Autenticación, solicitudes y respuestas](../general/authentication-requests-and-responses.md)
- [Guía del desarrollador de Key Vault](../general/developers-guide.md)
