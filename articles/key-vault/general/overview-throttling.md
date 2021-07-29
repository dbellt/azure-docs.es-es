---
title: Guía de las limitaciones de Azure Key Vault
description: La limitación de Key Vault limita el número de llamadas simultáneas para evitar el uso excesivo de recursos.
services: key-vault
author: msmbaldwin
ms.service: key-vault
ms.subservice: general
ms.topic: conceptual
ms.date: 12/02/2019
ms.author: mbaldwin
ms.openlocfilehash: f9e3f2095e6fd7a744769c11209ed115767c3aed
ms.sourcegitcommit: bc29cf4472118c8e33e20b420d3adb17226bee3f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/08/2021
ms.locfileid: "113492602"
---
# <a name="azure-key-vault-throttling-guidance"></a>Guía de las limitaciones de Azure Key Vault

La limitación es un proceso que se inicia y que limita el número de llamadas simultáneas al servicio de Azure para evitar el uso excesivo de recursos. Azure Key Vault se ha diseñado para controlar un volumen muy alto de solicitudes. Si tiene lugar un número excesivo de solicitudes, la limitación de las solicitudes del cliente le ayuda a mantener un rendimiento óptimo y la confiabilidad del servicio Azure Key Vault.

Las limitaciones varían según el escenario. Por ejemplo, si está realizando un gran volumen de escrituras, la posibilidad de limitación es mayor que si solo realiza lecturas.

## <a name="how-does-key-vault-handle-its-limits"></a>¿Cómo maneja Key Vaults sus límites?

Los límites de servicio en Key Vault sirven para evitar el uso incorrecto de los recursos y garantizar la calidad de servicio para todos los clientes de Key Vault. Cuando se supera un umbral de servicio, Key Vault limita las solicitudes sucesivas de ese cliente durante un período de tiempo, devuelve un código de estado HTTP 429 (Demasiadas solicitudes) y la solicitud produce un error. Las solicitudes con error que devuelven un código 429 no cuentan para la limitación cuyo seguimiento realiza Key Vault. 

Key Vault se diseñó originalmente para almacenar y recuperar los secretos en el momento de la implementación.  El mundo ha evolucionado y Key Vault se utiliza en tiempo de ejecución para almacenar y recuperar secretos y, a menudo, las aplicaciones y los servicios buscan el uso de Key Vault como una base de datos.  Los límites actuales no admiten altas tasas de rendimiento.

Key Vault se creó originalmente con los límites especificados en [Límites del servicio Azure Key Vault](service-limits.md).  Para maximizar las tasas de rendimiento de Key Vault, estas son algunas directrices y procedimientos recomendados para maximizar el rendimiento:
1. Asegúrese de que ha implementado la limitación.  El cliente debe respetar las directivas de retroceso exponencial para los errores 429 y asegurarse de que realiza los reintentos según las instrucciones que se indican a continuación.
1. Divida el tráfico de Key Vault entre varios almacenes y regiones diferentes.   Use un almacén independiente para cada dominio de seguridad o de disponibilidad.   Si tiene cinco aplicaciones, cada una en dos regiones, se recomiendan 10 almacenes, cada uno con los secretos exclusivos de la aplicación y la región.  Un límite global para la suscripción para todos los tipos de transacciones es cinco veces el límite del almacén de claves individual. Por ejemplo, en las otras transacciones HSM por suscripción, el límite es de 5000 transacciones en 10 segundos por suscripción. Considere la posibilidad de almacenar en caché el secreto dentro de su servicio o aplicación para reducir también directamente las RPS al almacén de claves y controlar el tráfico basado en ráfagas.  También puede dividir el tráfico entre diferentes regiones para minimizar la latencia y usar una suscripción o almacén diferentes.  No envíe más del límite de suscripción al servicio Key Vault en una sola región de Azure.
1. Almacene en memoria caché los secretos que recupere de Azure Key Vault en memoria y vuelva a usar la memoria siempre que sea posible.  Vuelva a leer desde Azure Key Vault solo cuando la copia en memoria caché deje de funcionar (por ejemplo, porque se ha realizado una rotación en origen). 
1. Key Vault está diseñado para sus propios secretos de servicios.   Si va a almacenar los secretos de sus clientes (especialmente para escenarios de almacenamiento de claves de alto rendimiento), considere la posibilidad de colocar las claves en una base de datos o una cuenta de almacenamiento con cifrado y almacenar solo la clave maestra en Azure Key Vault.
1. Las operaciones de cifrado, encapsulado y verificación de clave pública se pueden realizar sin acceso a Key Vault, lo que no solo reduce el riesgo de limitación, sino que también mejora la confiabilidad (siempre que se almacene correctamente en memoria caché el material de clave pública).
1. Si usa Key Vault para almacenar las credenciales de un servicio, compruebe si ese servicio admite la Autenticación de Azure AD para autenticarse directamente. Esto reduce la carga en Key Vault, mejora la confiabilidad y simplifica el código, ya que Key Vault ahora puede usar el token de Azure AD.  Muchos servicios se han pasado al uso de la Autenticación de Azure AD.  Consulte la lista actual en [Servicios que admiten identidades administradas para recursos de Azure](../../active-directory/managed-identities-azure-resources/services-support-managed-identities.md#azure-services-that-support-managed-identities-for-azure-resources).
1. Considere la posibilidad de escalonar la carga o la implementación durante un período de tiempo más largo para permanecer dentro de los límites actuales de RPS.
1. Si la aplicación consta de varios nodos que necesitan leer los mismos secretos, considere la posibilidad de usar un patrón de distribución ramificada, donde una entidad lee el secreto de Key Vault y se distribuye a todos los nodos.   Almacene en caché los secretos recuperados solo en memoria.


## <a name="how-to-throttle-your-app-in-response-to-service-limits"></a>Limitación de la aplicación en respuesta a los límites de servicio

Los siguientes son los **procedimientos recomendados** que debe implementar cuando el servicio está limitado:
- Reduzca el número de operaciones por solicitud.
- Reduzca la frecuencia de las solicitudes.
- Evite los reintentos inmediatos. 
    - Todas las solicitudes se acumulan en los límites de utilización.

Al implementar el control de errores de la aplicación, utilice el código de error HTTP 429 para detectar la necesidad de limitación del cliente. Si la solicitud vuelve a enviar un código de error HTTP 429, todavía se está produciendo un límite de servicio Azure. Continúe usando el método de limitación del cliente recomendado y vuelva a intentar la solicitud hasta que lo consiga.

A continuación, se muestra el código que implementa el retroceso exponencial. 
```
SecretClientOptions options = new SecretClientOptions()
    {
        Retry =
        {
            Delay= TimeSpan.FromSeconds(2),
            MaxDelay = TimeSpan.FromSeconds(16),
            MaxRetries = 5,
            Mode = RetryMode.Exponential
         }
    };
    var client = new SecretClient(new Uri("https://keyVaultName.vault.azure.net"), new DefaultAzureCredential(),options);
                                 
    //Retrieve Secret
    secret = client.GetSecret(secretName);
```


El uso de este código en una aplicación cliente de C# es sencillo. 

### <a name="recommended-client-side-throttling-method"></a>Método de limitación del cliente recomendado

En el código de error HTTP 429, comience la limitación del cliente mediante un enfoque de retroceso exponencial:

1. Espere 1 segundo, reintente la solicitud
2. Si todavía está limitado, espere 2 segundos, reintente la solicitud
3. Si todavía está limitado, espere 4 segundos, reintente la solicitud
4. Si todavía está limitado, espere 8 segundos, reintente la solicitud
5. Si todavía está limitado, espere 16 segundos, reintente la solicitud

En este momento, se deben no está recibiendo los códigos de respuesta HTTP 429.

## <a name="see-also"></a>Consulte también

Para obtener una orientación más profunda de la limitación en Microsoft Cloud, consulte [Patrón de limitación](/azure/architecture/patterns/throttling).
