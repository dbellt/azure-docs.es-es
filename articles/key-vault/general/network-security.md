---
title: Configuración de firewalls y redes virtuales de Azure Key Vault
description: Información sobre la configuración de redes del almacén de claves
services: key-vault
author: msmbaldwin
ms.service: key-vault
ms.subservice: general
ms.topic: tutorial
ms.date: 10/01/2020
ms.author: mbaldwin
ms.custom: devx-track-azurecli
ms.openlocfilehash: 770e0469f9c9cf8c20f0abf826a42c0584108cd9
ms.sourcegitcommit: 42ac9d148cc3e9a1c0d771bc5eea632d8c70b92a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2021
ms.locfileid: "109845949"
---
# <a name="configure-azure-key-vault-firewalls-and-virtual-networks"></a>Configuración de firewalls y redes virtuales de Azure Key Vault

En este documento se explican las distintas configuraciones del firewall de Key Vault con detalle. Para seguir las instrucciones paso a paso sobre cómo configurar estas opciones, siga la guía que encontrará [aquí](how-to-azure-key-vault-network-security.md). 

Para más información, consulte [Puntos de conexión de servicio de red virtual para Azure Key Vault](overview-vnet-service-endpoints.md).

## <a name="firewall-settings"></a>Configuración del firewall

En esta sección se tratan las distintas formas en que se puede configurar el firewall de Azure Key Vault.

### <a name="key-vault-firewall-disabled-default"></a>Firewall de Key Vault Firewall deshabilitado (opción predeterminada)

De forma predeterminada, cuando se crea un nuevo almacén de claves, el firewall de Azure Key Vault está deshabilitado. Todas las aplicaciones y los servicios de Azure pueden acceder al almacén de claves y enviarle solicitudes. Tenga en cuenta que esta configuración no significa que cualquier usuario podrá realizar operaciones en el almacén de claves. El almacén de claves todavía se restringe a los secretos, las claves y los certificados almacenados en él al requerir permisos de directiva de acceso y autenticación de Azure Active Directory. Para comprender con más detalle la autenticación del almacén de claves, consulte [aquí](./authentication-fundamentals.md) el documento sobre los aspectos básicos de la autenticación del almacén de claves. Para más información, consulte [Acceso a Azure Key Vault desde detrás de un firewall](./access-behind-firewall.md).

### <a name="key-vault-firewall-enabled-trusted-services-only"></a>Firewall de Key Vault habilitado (solo servicios de confianza)

Al habilitar el firewall de Key Vault, se le ofrecerá la opción de permitir que los servicios de confianza de Microsoft omitan este firewall. La lista de servicios de confianza no cubre todos los servicios de Azure. Por ejemplo, Azure DevOps no se encuentra en la lista de servicios de confianza. **Esto no implica que los servicios que no aparecen en la lista de servicios de confianza no sean de confianza o no sean seguros.** La lista de servicios de confianza abarca los servicios en los que Microsoft controla todo el código que se ejecuta en el servicio. Dado que los usuarios pueden escribir código personalizado en servicios de Azure como Azure DevOps, Microsoft no ofrece la opción de crear una aprobación global para el servicio. Además, que un servicio aparezca en la lista de servicios de confianza no significa que se permita en todos los escenarios. 

Para determinar si un servicio que está intentando usar está en la lista de servicios de confianza, consulte [aquí](./overview-vnet-service-endpoints.md#trusted-services) el siguiente documento.
Para obtener una guía paso a paso, siga las instrucciones que se proporcionan aquí para [Azure Portal, la CLI de Azure y PowerShell](how-to-azure-key-vault-network-security.md)

### <a name="key-vault-firewall-enabled-ipv4-addresses-and-ranges---static-ips"></a>Firewall de Key Vault habilitado (intervalos y direcciones IPv4: direcciones IP estáticas)

Si desea autorizar el acceso de un servicio determinado al almacén de claves a través del firewall de Key Vault, puede agregar su dirección IP a la lista de permitidos del firewall del almacén de claves. Esta configuración es la mejor para los servicios que usan direcciones IP estáticas o intervalos conocidos. En este caso, hay un límite de 1000 intervalos de CIDR.

Para permitir una dirección IP o un intervalo de direcciones de un recurso de Azure, como una aplicación web o una aplicación lógica, realice los pasos siguientes.

1. Iniciar sesión en Azure Portal
1. Seleccione el recurso (instancia específica del servicio).
1. Haga clic en la hoja "Propiedades" en "Configuración".
1. Busque el campo "Dirección IP".
1. Copie este valor o intervalo y escríbalo en la lista de permitidos del firewall del almacén de claves.

Para permitir un servicio completo de Azure a través del firewall de Key Vault, use la lista de direcciones IP de centros de datos documentadas públicamente para Azure que se encuentra [aquí](https://www.microsoft.com/download/details.aspx?id=41653). Busque las direcciones IP asociadas con el servicio que desea en la región que quiera y agregue esas direcciones IP al firewall del almacén de claves mediante los pasos anteriores.

### <a name="key-vault-firewall-enabled-virtual-networks---dynamic-ips"></a>Firewall de Key Vault habilitado (redes virtuales: direcciones IP dinámicas)

Si está intentando permitir un recurso de Azure como una máquina virtual a través de Key Vault, es posible que no pueda usar direcciones IP estáticas y que no desee permitir que todas las direcciones IP de Azure Virtual Machines tengan acceso al almacén de claves.

En este caso, debe crear el recurso en una red virtual y, a continuación, permitir que el tráfico de la red virtual y la subred concretas acceda al almacén de claves. Para ello, siga estos pasos.

1. Iniciar sesión en Azure Portal
1. Seleccione el almacén de claves que desee configurar.
1. Seleccione la hoja "Redes".
1. Seleccione "+Agregar red virtual existente".
1. Seleccione la red virtual y la subred que desee permitir a través del firewall del almacén de claves.

### <a name="key-vault-firewall-enabled-private-link"></a>Firewall de Key Vault habilitado (Private Link)

Para saber cómo configurar una conexión de vínculo privado en el almacén de claves, consulte este [documento](./private-link-service.md).

> [!IMPORTANT]
> Una vez que las reglas del firewall están en vigor, los usuarios solo pueden realizar operaciones del [plano de datos](security-features.md#privileged-access) de Key Vault cuando las solicitudes se originan desde redes virtuales o rangos de direcciones IPv4 permitidos. Esto también se aplica al acceso de Key Vault desde Azure Portal. Aunque los usuarios pueden ir a un almacén de claves desde Azure Portal, es posible que no pueda enumerar las claves, los secretos o los certificados si su equipo cliente no está en la lista de dispositivos permitidos. Esto también afecta al selector de Key Vault de otros servicios de Azure. Los usuarios podrían ver la lista de almacenes de claves, pero no enumerar las claves si las reglas del firewall limitan su equipo cliente.

> [!NOTE]
> Tenga en cuenta las siguientes limitaciones de configuración:
> * Se permite un máximo de 127 reglas de red virtual y 127 reglas de IPv4. 
> * Las reglas de red IP solo se permiten para direcciones IP públicas. No se permiten intervalos de direcciones IP reservados para redes privadas (tal y como se define en RFC 1918) en las reglas IP. Las redes privadas incluyen direcciones que comienzan por **10.** , **172.16-31** y **192.168.** . 
> * Solo se admiten direcciones IPV4 en este momento.

## <a name="references"></a>Referencias
* Referencia de plantilla de ARM: [Referencia de la plantilla de ARM de Azure Key Vault](/azure/templates/Microsoft.KeyVault/vaults)
* Comandos de la CLI de Azure: [az keyvault network-rule](/cli/azure/keyvault/network-rule)
* Cmdlets de Azure PowerShell: [Get-AzKeyVault](/powershell/module/az.keyvault/get-azkeyvault), [Add-AzKeyVaultNetworkRule](/powershell/module/az.KeyVault/Add-azKeyVaultNetworkRule), [Remove-AzKeyVaultNetworkRule](/powershell/module/az.KeyVault/Remove-azKeyVaultNetworkRule), [Update-AzKeyVaultNetworkRuleSet](/powershell/module/az.KeyVault/Update-azKeyVaultNetworkRuleSet)

## <a name="next-steps"></a>Pasos siguientes

* [Puntos de conexión de servicio de red virtual para Key Vault](overview-vnet-service-endpoints.md)
* [Introducción a la seguridad de Azure Key Vault](security-features.md)