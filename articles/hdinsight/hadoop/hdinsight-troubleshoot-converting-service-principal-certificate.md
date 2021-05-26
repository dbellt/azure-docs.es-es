---
title: 'Conversión del contenido del certificado a base-64: Azure HDInsight'
description: Conversión del contenido del certificado de entidad de servicio al formato de cadena con codificación Base 64 en Azure HDInsight
ms.service: hdinsight
ms.topic: troubleshooting
ms.date: 07/31/2019
ms.custom: devx-track-csharp
ms.openlocfilehash: 1ccd47b0a2f10c67dc808fb13aafca55fa5a5e67
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110088757"
---
# <a name="converting-service-principal-certificate-contents-to-base-64-encoded-string-format-in-hdinsight"></a>Conversión del contenido del certificado de entidad de servicio al formato de cadena con codificación Base 64 en HDInsight

En este artículo se describen los pasos de solución de problemas y las posibles soluciones para los problemas que se producen al usar clústeres de Azure HDInsight.

## <a name="issue"></a>Problema

Recibe un mensaje de error en el que se indica que la entrada no es una cadena con codificación Base 64 válida, ya que contiene un carácter que no corresponde a dicha codificación, más de dos caracteres de relleno o un carácter que no es un espacio en blanco entre los caracteres de relleno.

## <a name="cause"></a>Causa

Al usar PowerShell o la implementación de plantillas de Azure para crear clústeres con Data Lake como almacenamiento principal o adicional, el contenido del certificado de entidad de servicio proporcionado para acceder a la cuenta de almacenamiento de Data Lake se encontrará en el formato Base 64. La conversión incorrecta del contenido del certificado PFX a una cadena con codificación Base 64 puede producir este error.

## <a name="resolution"></a>Solución

Cuando tenga el certificado de entidad de servicio en formato PFX (haga clic [aquí](https://github.com/Azure/azure-quickstart-templates/tree/master/quickstarts/microsoft.hdinsight/hdinsight-datalake-store-azure-storage) para ver los pasos de creación de la entidad de servicio de ejemplo), use el siguiente fragmento de código de C# o comando de PowerShell para convertir el contenido del certificado al formato Base 64.

```powershell
$servicePrincipalCertificateBase64 = [System.Convert]::ToBase64String([System.IO.File]::ReadAllBytes(path-to-servicePrincipalCertificatePfxFile))
```

```csharp
using System;
using System.IO;

namespace ConsoleApplication
{
    class Program
    {
        static void Main(string[] args)
        {
            var certContents = File.ReadAllBytes(@"<path to pfx file>");
            string certificateData = Convert.ToBase64String(certContents);
            System.Diagnostics.Debug.WriteLine(certificateData);
        }
    }
}
```

## <a name="next-steps"></a>Pasos siguientes

[!INCLUDE [troubleshooting next steps](../../../includes/hdinsight-troubleshooting-next-steps.md)]