---
title: Configuración de un cliente NFS para Azure NetApp Files | Microsoft Docs
description: Describe cómo configurar los clientes NFS para usarlos con Azure NetApp Files.
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
ms.date: 05/10/2021
ms.author: b-juche
ms.openlocfilehash: 695dd379e0b9f02f5ec6a08f2a037d071259d2b7
ms.sourcegitcommit: eda26a142f1d3b5a9253176e16b5cbaefe3e31b3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2021
ms.locfileid: "109734522"
---
# <a name="configure-an-nfs-client-for-azure-netapp-files"></a>Configuración de un cliente NFS para Azure NetApp Files

La configuración de cliente NFS que se describe en este artículo forma parte de la configuración cuando [configura el cifrado Kerberos de NFSv 4.1](configure-kerberos-encryption.md) o [crea un volumen de dos protocolos](create-volumes-dual-protocol.md). Hay una amplia variedad de distribuciones de Linux disponibles para usarse con Azure NetApp Files. En este artículo se describen las configuraciones de dos de los entornos de uso más frecuentes: RHEL 8 y Ubuntu 18.04. 

## <a name="requirements-and-considerations"></a>Requisitos y consideraciones  

Independientemente del tipo de Linux que use, se requieren las siguientes configuraciones:

* Configuración de un cliente NTP para evitar problemas con el sesgo horario.
* Configuración de las entradas DNS del cliente Linux para la resolución de nombres.  
    Esta configuración debe incluir el registro "A" (directo) y el registro PTR (inverso). 
* Para la unión a un dominio, cree una cuenta de equipo para el cliente Linux en la instancia de Active Directory de destino (que se crea durante el comando de unión a un dominio Kerberos). 
    > [!NOTE] 
    > La variable `$SERVICEACCOUNT` utilizada en los siguientes comandos debe ser una cuenta de usuario con permisos o delegación para crear una cuenta de equipo en la unidad organizativa de destino.

## <a name="rhel-8-configuration"></a>Configuración de RHEL 8 

En esta sección se describen las configuraciones de RHEL necesarias para el cifrado Kerberos NFSv4.1 y el protocolo dual.  

En los ejemplos de esta sección se usan el nombre de dominio y la dirección IP siguientes:  

* Nombre de dominio: `contoso.com`
* Dirección IP privada: `10.6.1.4`

### <a name="rhel-8-configuration-if-you-are-using-nfsv41-kerberos-encryption"></a><a name="rhel8_nfsv41_kerberos"></a>Configuración de RHEL 8 si se usa el cifrado Kerberos NFSv4.1

1. Configure `/etc/resolv.conf` con el servidor DNS adecuado.  

    Por ejemplo:  

    `[root@reddoc cbs]# cat /etc/resolv.conf`   
    `search contoso.com`   
    `nameserver 10.6.1.4(private IP)`   

2. Agregue el registro de cliente NFS al servidor DNS de la zona de búsqueda DNS inversa y directa.

3. Para comprobar DNS, use los siguientes comandos del cliente NFS:   

    `# nslookup [hostname/FQDN of NFS client(s)]`   
    `# nslookup [IP address of NFS client(s)]`

4. Instale los paquetes:   

    `yum update`   
    `sudo yum -y install realmd sssd adcli samba-common krb5-workstation chrony nfs-utils`

5.  Configure el cliente NTP.  

    RHEL 8 usa chrony de manera predeterminada. Siga las instrucciones de configuración de [Uso del conjunto `Chrony` para configurar NTP](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/configuring_basic_system_settings/using-chrony-to-configure-ntp).

6.  Una el dominio de Active Directory:  

    `sudo realm join $DOMAIN.NAME -U $SERVICEACCOUNT --computer-ou="OU=$YOUROU"`

    Por ejemplo: 

    `sudo realm join CONTOSO.COM -U ad_admin --computer-ou="CN=Computers"`
    
    Asegúrese de que `default_realm` esté establecido en el dominio kerberos proporcionado en `/etc/krb5.conf`.  Si no es así, agréguelo en la sección `[libdefaults]` del archivo, tal como se muestra en el ejemplo siguiente:
    
    ```
    [libdefaults]
        default_realm = CONTOSO.COM
        default_tkt_enctypes = aes256-cts-hmac-sha1-96
        default_tgs_enctypes = aes256-cts-hmac-sha1-96
        permitted_enctypes = aes256-cts-hmac-sha1-96
    [realms]
        CONTOSO.COM = {
            kdc = dc01.contoso.com
            admin_server = dc01.contoso.com
            master_kdc = dc01.contoso.com
            default_domain = contoso.com
        }
    [domain_realm]
        .contoso.com = CONTOSO.COM
        contoso.com = CONTOSO.COM
    [logging]
        kdc = SYSLOG:INFO
        admin_server = FILE=/var/kadm5.log
    ```

7. Reinicie todos los servicios NFS:  
 
    `systemctl start nfs-*`   
    `systemctl restart rpc-gssd.service`

    El reinicio evita la condición de error `“mount.nfs: an incorrect mount option was specified”` durante el montaje de Kerberos.

8. Ejecute el comando `kinit` con la cuenta de usuario para obtener vales: 
 
    `sudo kinit $SERVICEACCOUNT@DOMAIN`  

    Por ejemplo:   

    `sudo kinit ad_admin@CONTOSO.COM`


### <a name="rhel-8-configuration-if-you-are-using-dual-protocol"></a>Configuración de RHEL 8 si se usa el protocolo dual

Los pasos siguientes son opcionales. Solo se tienen que realizar si se emplea asignación de usuario en el cliente NFS: 

1. Realice todos los pasos de la sección [Configuración de RHEL 8 si se usa el cifrado Kerberos NFSv4.1](#rhel8_nfsv41_kerberos).   

2. Agregue un registro DNS estático al archivo /etc/hosts para usar el nombre de dominio completo (FQDN) de la instancia de AD en lugar de usar la dirección IP del archivo de configuración SSSD:  

    `cat /etc/hosts`   
    `10.6.1.4 winad2016.contoso.com`

3. Agregue una sección adicional para que los dominios resuelvan identificadores del servidor LDAP de AD:    

    `[root@reddoc cbs]# cat /etc/sssd/sssd.conf`   
    `[sssd]`   
    `domains = contoso.com, contoso-ldap (new entry added for LDAP as id_provider)`   
    `config_file_version = 2`   
    `services = nss, pam, ssh, sudo (ensure nss is present in this list)`   
 
    `[domain/contoso-ldap] (Copy the following lines. Modify as per your domain name.)`   
    `auth_provider = krb5`   
    `chpass_provider = krb5`   
    `id_provider = ldap`   
    `ldap_search_base = dc=contoso,dc=com(your domain)`   
    `ldap_schema = rfc2307bis`   
    `ldap_sasl_mech = GSSAPI`   
    `ldap_user_object_class = user`   
    `ldap_group_object_class = group`   
    `ldap_user_home_directory = unixHomeDirectory`   
    `ldap_user_principal = userPrincipalName`   
    `ldap_account_expire_policy = ad`   
    `ldap_force_upper_case_realm = true`   
    `ldap_user_search_base = cn=Users,dc=contoso,dc=com (based on your domain)`   
    `ldap_group_search_base = cn=Users,dc=contoso,dc=com (based on your domain)`   
    `ldap_sasl_authid = REDDOC$ (ensure $ at the end you can get this from “klist -kte” command)`   
    `krb5_server = winad2016.contoso.com (same as AD address which is added in /etc/hosts)`   
    `krb5_realm = CONTOSO.COM (domain name in caps)`   
    `krb5_kpasswd = winad2016.contoso.com (same as AD address which is added in /etc/hosts)`   
    `use_fully_qualified_names = false`   
 
    `[domain/contoso.com]  (Do not edit or remove any of the following information. This information is automatically generated during the realm join process.)`   
    `ad_domain = contoso.com`   
    `krb5_realm = CONTOSO.COM`   
    `realmd_tags = manages-system joined-with-adcli`   
    `cache_credentials = True`   
    `id_provider = ad`   
    `krb5_store_password_if_offline = True`   
    `default_shell = /bin/bash`   
    `ldap_id_mapping = True`   
    `use_fully_qualified_names = True`   
    `fallback_homedir = /home/%u@%d`   
    `access_provider = ad`   

4. Asegúrese de que `/etc/nsswitch.conf` tenga la entrada `sss`:   

    `cat /etc/nsswitch.conf`   
    `passwd: sss files systemd`   
    `group: sss files systemd`   
    `netgroup: sss files`   

5. Reinicie el servicio `sssd` y borre la caché:   

    `service sssd stop`   
    `rm -f /var/lib/sss/db/*`   
    `service sssd start`   
 
6. Realice una prueba para asegurarse de que el cliente está integrado en el servidor LDAP:   

    `[root@red81 cbs]# id ldapuser1`   
    `uid=1234(ldapuser1) gid=1111(ldapgroup1) groups=1111(ldapgroup1)`   

## <a name="ubuntu-configuration"></a>Configuración de Ubuntu   

En esta sección se describen las configuraciones de Ubuntu necesarias para el cifrado Kerberos NFSv4.1 y el protocolo dual. 

En los ejemplos de esta sección se usan el nombre de dominio y la dirección IP siguientes:  

* Nombre de dominio: `contoso.com`
* Dirección IP privada: `10.6.1.4`

1. Configure `/etc/resolv.conf` con el servidor DNS adecuado:

    `root@ubuntu-rak:/home/cbs# cat /etc/resolv.conf`   
    `search contoso.com`   
    `nameserver <private IP address of DNS server>`   

2. Agregue el registro de cliente NFS al servidor DNS de la zona de búsqueda DNS inversa y directa.
 
    Para comprobar DNS, use los siguientes comandos del cliente NFS:

    `# nslookup [hostname/FQDN of NFS client(s)]`   
    `# nslookup [IP address of NFS client(s)]`   

3. Instale los paquetes:
 
    `apt-get update`   
    `apt-get install -y realmd packagekit sssd adcli samba-common chrony krb5-user nfs-common`
    
    Cuando se le pida, escriba `$DOMAIN.NAME` (con mayúsculas, por ejemplo, `CONTOSO.COM`) como dominio Kerberos predeterminado.

4. Reinicie el servicio `rpc-gssd.service`: 

    `sudo systemctl start rpc-gssd.service`

5. Ubuntu 18.04 usa chrony de manera predeterminada. Siga las instrucciones de configuración de [Ubuntu Bionic: Uso de chrony para configurar NTP](https://ubuntu.com/blog/ubuntu-bionic-using-chrony-to-configure-ntp).

6. Una el dominio de Active Directory:   
 
    `sudo realm join $DOMAIN.NAME -U $SERVICEACCOUNT --computer-ou="OU=$YOUROU"`
 
    Por ejemplo:    
    `sudo realm join CONTOSO.COM -U ad_admin --computer-ou="CN=Computers"`

7. Use `kinit` con el usuario para obtener vales: 
 
    `sudo kinit $SERVICEACCOUNT`   
 
    Por ejemplo:    
    `sudo kinit ad_admin`  

### <a name="ubuntu-configuration-if-you-are-using-dual-protocol"></a>Configuración de Ubuntu si se usa el protocolo dual  

Los pasos siguientes son opcionales.  Solo se tienen que realizar si se quiere emplear asignación de usuario en el cliente NFS:  

1. Ejecute el siguiente comando para actualizar los paquetes instalados:   
    `sudo apt update && sudo apt install libnss-ldap libpam-ldap ldap-utils nscd`

    En el siguiente ejemplo se usan valores de ejemplo. Cuando el comando le pida una entrada, debe proporcionar una basada en el entorno. 

    `base dc=contoso,dc=com uri ldap://10.20.0.4:389/ ldap_version 3 rootbinddn cn=admin,cn=Users,dc=contoso,dc=com pam_password ad`   

2. Ejecute el siguiente comando para reiniciar y habilitar el servicio:

    `sudo systemctl restart nscd && sudo systemctl enable nscd`   

En el ejemplo siguiente se realiza una consulta al servidor LDAP de AD desde el cliente LDAP de Ubuntu para un usuario de LDAP `‘hari1’`: 

`root@cbs-k8s-varun4-04:/home/cbs# getent passwd hari1`   
`hari1:*:1237:1237:hari1:/home/hari1:/bin/bash`   


## <a name="next-steps"></a>Pasos siguientes  

* [Creación de un volumen de NFS para Azure NetApp Files](azure-netapp-files-create-volumes.md)
* [Creación de un volumen de dos protocolos para Azure NetApp Files](create-volumes-dual-protocol.md)

