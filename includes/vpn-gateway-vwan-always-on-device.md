---
title: archivo de inclusión
description: archivo de inclusión
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: include
ms.date: 05/26/2021
ms.author: cherylmc
ms.custom: include file
ms.openlocfilehash: 756df0b7bce34119230bdcad614e2d45ae0e582a
ms.sourcegitcommit: e1d5abd7b8ded7ff649a7e9a2c1a7b70fdc72440
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/27/2021
ms.locfileid: "110579751"
---
Para poder establecer correctamente un túnel de dispositivo, deben cumplirse los siguientes requisitos:

* El dispositivo debe ser un equipo unido a un dominio que ejecute la versión 1809 u otra posterior de Windows 10 Enterprise o Education.
* El túnel solamente puede configurarse en la solución VPN integrada en Windows y debe establecerse utilizando IKEv2 con la autenticación de certificados de equipo.
* Solo se puede configurar un túnel de dispositivo en cada dispositivo.

1. Instale certificados de cliente en el cliente de Windows 10, tal y como se muestra en este artículo sobre [el cliente VPN de punto a sitio](../articles/vpn-gateway/point-to-site-how-to-vpn-client-install-azure-cert.md). El certificado debe estar en el almacén del equipo local.
1. Cree un perfil de VPN y configurar el túnel de dispositivo en el contexto de la cuenta de sistema local mediante [estas instrucciones](/windows-server/remote/remote-access/vpn/vpn-device-tunnel-config#vpn-device-tunnel-configuration).

### <a name="configuration-example-for-device-tunnel"></a>Ejemplo de configuración de túnel de dispositivo

Una vez que haya configurado la puerta de enlace de red virtual e instalado el certificado de cliente en el almacén del equipo local del cliente de Windows 10, use los ejemplos siguientes para configurar un túnel de dispositivo de cliente.

1. Copie el texto siguiente y guárdelo como ***devicecert.ps1***.

   ```
   Param(
   [string]$xmlFilePath,
   [string]$ProfileName
   )

   $a = Test-Path $xmlFilePath
   echo $a

   $ProfileXML = Get-Content $xmlFilePath

   echo $XML

   $ProfileNameEscaped = $ProfileName -replace ' ', '%20'

   $Version = 201606090004

   $ProfileXML = $ProfileXML -replace '<', '&lt;'
   $ProfileXML = $ProfileXML -replace '>', '&gt;'
   $ProfileXML = $ProfileXML -replace '"', '&quot;'

   $nodeCSPURI = './Vendor/MSFT/VPNv2'
   $namespaceName = "root\cimv2\mdm\dmmap"
   $className = "MDM_VPNv2_01"

   $session = New-CimSession

   try
   {
   $newInstance = New-Object Microsoft.Management.Infrastructure.CimInstance $className, $namespaceName
   $property = [Microsoft.Management.Infrastructure.CimProperty]::Create("ParentID", "$nodeCSPURI", 'String', 'Key')
   $newInstance.CimInstanceProperties.Add($property)
   $property = [Microsoft.Management.Infrastructure.CimProperty]::Create("InstanceID", "$ProfileNameEscaped", 'String', 'Key')
   $newInstance.CimInstanceProperties.Add($property)
   $property = [Microsoft.Management.Infrastructure.CimProperty]::Create("ProfileXML", "$ProfileXML", 'String', 'Property')
   $newInstance.CimInstanceProperties.Add($property)

   $session.CreateInstance($namespaceName, $newInstance)
   $Message = "Created $ProfileName profile."
   Write-Host "$Message"
   }
   catch [Exception]
   {
   $Message = "Unable to create $ProfileName profile: $_"
   Write-Host "$Message"
   exit
   }
   $Message = "Complete."
   Write-Host "$Message"
   ```
1. Copie el siguiente texto y guárdelo como ***VPNProfile.xml** _ en la misma carpeta que _*devicecert.ps1**. Modifique el texto siguiente para adaptarlo a su entorno.

   * `<Servers>azuregateway-1234-56-78dc.cloudapp.net</Servers> <= Can be found in the VpnSettings.xml in the downloaded profile zip file`
   * `<Address>192.168.3.5</Address> <= IP of resource in the vnet or the vnet address space`
   * `<Address>192.168.3.4</Address> <= IP of resource in the vnet or the vnet address space`

   ```
   <VPNProfile>  
     <NativeProfile>  
   <Servers>azuregateway-1234-56-78dc.cloudapp.net</Servers>  
   <NativeProtocolType>IKEv2</NativeProtocolType>  
   <Authentication>  
     <MachineMethod>Certificate</MachineMethod>  
   </Authentication>  
   <RoutingPolicyType>SplitTunnel</RoutingPolicyType>  
    <!-- disable the addition of a class based route for the assigned IP address on the VPN interface -->
   <DisableClassBasedDefaultRoute>true</DisableClassBasedDefaultRoute>  
     </NativeProfile> 
     <!-- use host routes(/32) to prevent routing conflicts -->  
     <Route>  
   <Address>192.168.3.5</Address>  
   <PrefixSize>32</PrefixSize>  
     </Route>  
     <Route>  
   <Address>192.168.3.4</Address>  
   <PrefixSize>32</PrefixSize>  
     </Route>  
   <!-- need to specify always on = true --> 
     <AlwaysOn>true</AlwaysOn> 
   <!-- new node to specify that this is a device tunnel -->  
    <DeviceTunnel>true</DeviceTunnel>
   <!--new node to register client IP address in DNS to enable manage out -->
   <RegisterDNS>true</RegisterDNS>
   </VPNProfile>
   ```
1. Descargue **PsExec** de [Sysinternals](/sysinternals/downloads/psexec) y extraiga los archivos en **C:\PSTools**.
1. En un símbolo del sistema de CMD con privilegios de administrador, inicie PowerShell ejecutando:

   ```
   PsExec.exe Powershell for 32-bit Windows
   PsExec64.exe Powershell for 64-bit Windows
   ```

   ![Captura de pantalla que muestra una ventana del símbolo del sistema con un comando para iniciar la versión de 64 bits de PowerShell.](./media/vpn-gateway-vwan-always-on-device/powershell.png)
1. En PowerShell, vaya la carpeta donde se encuentra **devicecert.ps1** y **VPNProfile.xml** y ejecute el siguiente comando:

   ```powershell
   .\devicecert.ps1 .\VPNProfile.xml MachineCertTest
   ```
   
   ![Captura de pantalla que muestra una ventana de PowerShell que ejecutó MachineCertTest mediante el script devicesert.](./media/vpn-gateway-vwan-always-on-device/machinecerttest.png)
1. Ejecute **rasphone**.

   ![Captura de pantalla que muestra un cuadro de diálogo Ejecutar con el rasphone seleccionado.](./media/vpn-gateway-vwan-always-on-device/rasphone.png)
1. Busque la entrada **MachineCertTest** y haga clic en **Conectar**.

   ![Captura de pantalla que muestra un cuadro de diálogo Conexiones de red con MachineCertTest seleccionado y un botón Conectar.](./media/vpn-gateway-vwan-always-on-device/connect.png)
1. Si la conexión se realiza correctamente, reinicie el equipo. El túnel se conectará automáticamente.