---
title: Creación de un certificado de cliente con Microsoft Azure Confidential Ledger
description: Creación de un certificado de cliente con Microsoft Azure Confidential Ledger
services: confidential-ledger
author: msmbaldwin
ms.service: confidential-ledger
ms.topic: overview
ms.date: 04/15/2021
ms.author: mbaldwin
ms.openlocfilehash: 6d384ea07c49a8b7936e3cdf634b9775847018f0
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110386649"
---
# <a name="creating-a-client-certificate"></a>Creación de un certificado de cliente

Las API de Confidential Ledger requieren autenticación basada en certificados de cliente. Solo los certificados agregados a una lista de permitidos durante la creación de Ledger o la actualización de Ledger se pueden usar para llamar a las API funcionales de Confidential Ledger.

Necesitará un certificado en formato PEM. Puede crear más de un certificado y agregarlos o eliminarlos mediante la API de actualización de Ledger.

## <a name="openssl"></a>OpenSSL

Se recomienda usar OpenSSL para generar certificados. Si tiene GIT instalado, puede ejecutar OpenSSL en el shell de GIT. De lo contrario, puede instalar OpenSSL para el sistema operativo.

- **Windows**: Instale [chocolatey para Windows](https://chocolatey.org/install), abra una ventana de terminal de PowerShell en modo de administrador y ejecute `choco install openssl`. Como alternativa, puede instalar OpenSSL para Windows directamente desde [aquí](http://gnuwin32.sourceforge.net/packages/openssl.htm).
- **Linux**: Ejecute `sudo apt-get install openssl`.

Luego puede generar un certificado ejecutando `openssl` en una ventana de terminal de Bash o PowerShell:

```bash
openssl req -new -newkey rsa:1024 -days 365 -nodes -x509 -keyout user_privk.pem -out user_cert.pem -subj=/CN="User Client Certificate"
```

## <a name="next-steps"></a>Pasos siguientes

- [Información general de Microsoft Azure Confidential Ledger](overview.md)