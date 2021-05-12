---
title: Causas comunes del reciclado de roles del servicio en la nube (clásico) | Microsoft Docs
description: Un rol del servicio en la nube que se recicla repentinamente puede causar un considerable tiempo de inactividad. A continuación encontrará algunos de los problemas comunes que provocan el reciclaje de los roles, lo que le ayudará a reducir el tiempo de inactividad.
ms.topic: troubleshooting
ms.service: cloud-services
ms.date: 10/14/2020
ms.author: tagore
author: tanmaygore
ms.reviewer: mimckitt
ms.custom: ''
ms.openlocfilehash: daf8bdb0e6729d5e95fdf0e6186f6511c39690f8
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108735900"
---
# <a name="common-issues-that-cause-azure-cloud-service-classic-roles-to-recycle"></a>Problemas comunes que hacen que se reciclen los roles del servicio en la nube de Azure (clásico)

> [!IMPORTANT]
> [Azure Cloud Services (soporte extendido)](../cloud-services-extended-support/overview.md) es un nuevo modelo de implementación basado en Azure Resource Manager para el producto Azure Cloud Services. Con este cambio, se ha modificado el nombre del modelo de implementación basado en Azure Cloud Services para Azure Service Manager a Cloud Services (clásico), y todas las implementaciones nuevas deben usar [Cloud Services (soporte extendido)](../cloud-services-extended-support/overview.md).

En este artículo se describen algunas de las causas comunes de problemas relacionados con la implementación y se proporcionan sugerencias para la resolución de dichos problemas. Aparece una indicación de que existe un problema con una aplicación cuando la instancia de rol no se inicia o cuando alterna entre los estados inicializando, ocupado y deteniendo.

[!INCLUDE [support-disclaimer](../../includes/support-disclaimer.md)]

## <a name="missing-runtime-dependencies"></a>Dependencias en tiempo de ejecución que faltan
Si un rol de la aplicación se basa en algún ensamblado que no forma parte de .NET Framework o de la biblioteca administrada de Azure, debe incluir explícitamente ese ensamblado en el paquete de aplicación. Tenga presente que otros marcos de trabajo de Microsoft no están disponibles en Azure de forma predeterminada. Si el rol se basa en uno de esos marcos de trabajo, debe agregar esos ensamblados al paquete de la aplicación.

Antes de compilar y empaquetar la aplicación, compruebe lo siguiente:

* Si usa Visual Studio, asegúrese de que la propiedad **Copy Local** se establece en **True** para cada ensamblado del proyecto que no forme parte de Azure SDK o de .NET Framework.
* Asegúrese de que el archivo web.config no hace referencia a ningún ensamblado sin usar en el elemento de compilación.
* El elemento **Build Action** de cada archivo .cshtml se establece en **Content**. Esto garantiza que los archivos aparecerán correctamente en el paquete y permite que otros archivos referenciados aparezcan en el paquete.

## <a name="assembly-targets-wrong-platform"></a>El ensamblado tiene como destino la plataforma equivocada
Azure es un entorno de 64 bits. Por lo tanto, los ensamblados de .NET compilados para un destino de 32 bits no funcionarán en Azure.

## <a name="role-throws-unhandled-exceptions-while-initializing-or-stopping"></a>El rol genera excepciones no controladas cuando se inicializa o se detiene
Todas las excepciones iniciadas por los métodos de la clase [RoleEntryPoint], que incluye los métodos [OnStart], [OnStop] y [Ejecutar], son excepciones no controladas. Si se produce una excepción no controlada en uno de estos métodos, el rol se reciclará. Si el rol se recicla de forma repetida, es posible que inicie una excepción no controlada cada vez que intente iniciarse.

## <a name="role-returns-from-run-method"></a>El rol se devuelve del método Run
El método [Ejecutar] está pensado para que se ejecute indefinidamente. Si el código invalida el método [Ejecutar] , podría quedar inactivo indefinidamente. Si se devuelve el método [Ejecutar] , el rol se recicla.

## <a name="incorrect-diagnosticsconnectionstring-setting"></a>Configuración incorrecta de DiagnosticsConnectionString
Si la aplicación usa Diagnósticos de Azure, el archivo de configuración de servicio debe especificar el valor de configuración de `DiagnosticsConnectionString` . Este valor debe especificar una conexión HTTPS a la cuenta de almacenamiento de Azure.

Para asegurarse de que el valor de `DiagnosticsConnectionString` sea correcto antes de implementar el paquete de aplicación en Azure, compruebe lo siguiente:  

* El valor de `DiagnosticsConnectionString` apunta a una cuenta de almacenamiento válida de Azure.  
  De forma predeterminada, este valor apunta a la cuenta de almacenamiento emulado, por lo que debe cambiar explícitamente esta configuración antes de implementar el paquete de aplicación. Si no cambia este valor, se produce una excepción cuando la instancia de rol intenta iniciar al monitor de diagnóstico. Esto puede hacer que la instancia de rol se recicle indefinidamente.
* La cadena de conexión se especifica en el siguiente [formato](../storage/common/storage-configure-connection-string.md). (El protocolo debe especificarse como HTTPS). Reemplace *MyAccountName* por el nombre de su cuenta de almacenamiento y *MyAccountKey* por la clave de acceso de su cuenta:    

```console
DefaultEndpointsProtocol=https;AccountName=MyAccountName;AccountKey=MyAccountKey
```

  Si está desarrollando una aplicación con Azure Tools para Microsoft Visual Studio, puede usar las páginas de propiedades para establecer este valor.

## <a name="exported-certificate-does-not-include-private-key"></a>El certificado exportado no incluye una clave privada
Para ejecutar un rol web con TLS, debe asegurarse de que el certificado de administración exportado incluya la clave privada. Si usa el *administrador de certificados de Windows* para exportar el certificado, asegúrese de seleccionar **Sí** en la opción **Exportar la clave privada**. El certificado se debe exportar al formato PFX, que es el único formato que se admite actualmente.

## <a name="next-steps"></a>Pasos siguientes
Vea más [artículos de solución de problemas](../index.yml?product=cloud-services&tag=top-support-issue) para servicios en la nube.

En la [series de blogs de Kevin Williamson](/archive/blogs/kwill/windows-azure-paas-compute-diagnostics-data)puede ver más escenarios de reciclaje de roles.

[RoleEntryPoint]: /previous-versions/azure/reference/ee758619(v=azure.100)
[OnStart]: /previous-versions/azure/reference/ee772851(v=azure.100)
[OnStop]: /previous-versions/azure/reference/ee772844(v=azure.100)
[Ejecutar]: /previous-versions/azure/reference/ee772746(v=azure.100)