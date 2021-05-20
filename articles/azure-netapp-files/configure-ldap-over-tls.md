---
title: Configuración de ADDS LDAP sobre TLS para Azure NetApp Files | Microsoft Docs
description: Describe cómo configurar ADDS LDAP sobre TLS para Azure NetApp Files, incluida la administración de certificados de entidad de certificación raíz.
services: azure-netapp-files
documentationcenter: ''
author: b-juche
manager: ''
editor: ''
ms.assetid: ''
ms.service: azure-netapp-files
ms.workload: storage
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 05/05/2021
ms.author: b-juche
ms.openlocfilehash: e9a1b9148948319ebf05e2bcc9f5f306b2a21ec0
ms.sourcegitcommit: 89c4843ec85d1baea248e81724781d55bed86417
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108796013"
---
# <a name="configure-adds-ldap-over-tls-for-azure-netapp-files"></a>Configuración de ADDS LDAP sobre TLS para Azure NetApp Files

Puede usar LDAP sobre TLS para proteger la comunicación entre un volumen Azure NetApp Files y el servidor LDAP de Active Directory.  Puede habilitar LDAP sobre TLS para volúmenes de protocolo dual, SMB y NFS de Azure NetApp Files.  

## <a name="considerations"></a>Consideraciones

* LDAP a través de TLS no se debe habilitar si usa Azure Active Directory Domain Services (AADDS). AADDS usa LDAPS (puerto 636) para proteger el tráfico LDAP en lugar de LDAP sobre TLS (puerto 389).  

## <a name="register-the-ldap-over-tls-feature"></a>Registro de la característica LDAP sobre TLS 

La característica LDAP sobre TLS actualmente está en versión preliminar. Si usa esta característica por primera vez, debe registrarla primero.

1.  Registre la característica:

    ```azurepowershell-interactive
    Register-AzProviderFeature -ProviderNamespace Microsoft.NetApp -FeatureName ANFLdapOverTls
    ```

2. Compruebe el estado del registro de la característica: 

    > [!NOTE]
    > **RegistrationState** puede estar en el estado `Registering` hasta 60 minutos antes de cambiar a `Registered`. Espere hasta que el estado sea `Registered` antes de continuar.

    ```azurepowershell-interactive
    Get-AzProviderFeature -ProviderNamespace Microsoft.NetApp -FeatureName ANFLdapOverTls
    ```
También puede usar los comandos de la [CLI de Azure](/cli/azure/feature?preserve-view=true&view=azure-cli-latest) `az feature register` y `az feature show` para registrar la característica y mostrar el estado del registro. 

## <a name="generate-and-export-root-ca-certificate"></a>Generación y exportación de certificados de entidad de certificación raíz 

Si no tiene un certificado de entidad de certificación raíz, debe generar uno y exportarlo para su uso con la autenticación LDAP sobre TLS. 

1. Siga las indicaciones de [Instalación de la entidad de certificación](/windows-server/networking/core-network-guide/cncg/server-certs/install-the-certification-authority) para instalar y configurar la entidad de certificación de ADDS. 

2. Siga las indicaciones de [Ver certificados con el complemento MMC](/dotnet/framework/wcf/feature-details/how-to-view-certificates-with-the-mmc-snap-in) para usar el complemento MMC y la herramienta Administrador de certificados.  
    Use el complemento Administrador de certificados para buscar el certificado raíz o emisor del dispositivo local. Debe ejecutar los comandos del complemento Administración de certificados desde una de las siguientes opciones:  
    * Un cliente basado en Windows que se haya unido al dominio y tenga instalado el certificado raíz. 
    * Otra máquina del dominio que contenga el certificado raíz.  

3. Exporte el certificado de entidad de certificación raíz.  
    Los certificados de entidad de certificación raíz se pueden exportar desde el directorio de entidades emisoras de certificados raíz de confianza o personales, tal y como se muestra en los ejemplos siguientes:   
    ![captura de pantalla que muestra los certificados personales](../media/azure-netapp-files/personal-certificates.png)   
    ![captura de pantalla que muestra las entidades emisoras de certificados raíz de confianza](../media/azure-netapp-files/trusted-root-certification-authorities.png)    

    Asegúrese de que el certificado se exporta en el formato X.509 codificado en Base-64 (.CER): 

    ![Asistente para exportación de certificados](../media/azure-netapp-files/certificate-export-wizard.png)

## <a name="enable-ldap-over-tls-and-upload-root-ca-certificate"></a>Habilitación de LDAP sobre TLS y carga del certificado de entidad de certificación raíz 

1. Vaya a la cuenta de NetApp que se usa para el volumen y haga clic en **Conexiones de Active Directory**. A continuación, haga clic en **Conectar** para crear una nueva conexión de AD o en **Editar** para editar una conexión de AD existente.  

2. En la ventana **Join Active Directory** (Conectarse a Active Directory) o **Edit Active Directory** (Editar Active Directory) que aparece, seleccione la casilla de verificación **LDAP over TLS** (LDAP sobre TLS) para habilitar LDAP sobre TLS en el volumen. A continuación haga clic en **Server root CA Certificate** (Certificado de entidad de certificación raíz del servidor) y cargue el [certificado de entidad de certificación raíz generado](#generate-and-export-root-ca-certificate) para usarlo con LDAP sobre TLS.  

    ![Captura de pantalla que muestra la opción LDAP sobre TLS](../media/azure-netapp-files/ldap-over-tls-option.png)

    Asegúrese de que el nombre de la entidad de certificación pueda resolverse mediante DNS. Este nombre es el campo "Emitido por" o "Emisor" del certificado:  

    ![Captura de pantalla que muestra la información del certificado](../media/azure-netapp-files/certificate-information.png)

Si ha cargado un certificado no válido y tiene configuraciones de AD, volúmenes SMB o volúmenes Kerberos existentes, se produce un error similar al siguiente:

`Error updating Active Directory settings The LDAP client configuration "ldapUserMappingConfig" for Vservers is an invalid configuration.`

Para resolver el error, cargue un certificado de entidad de certificación raíz válido en su cuenta de NetApp según lo requiera el servidor LDAP de Windows Active Directory para la autenticación LDAP.

## <a name="next-steps"></a>Pasos siguientes  

* [Creación de un volumen de NFS para Azure NetApp Files](azure-netapp-files-create-volumes.md)
* [Cree un volumen SMB para Azure NetApp Files](azure-netapp-files-create-volumes-smb.md) 
* [Creación de un volumen de dos protocolos para Azure NetApp Files](create-volumes-dual-protocol.md)

