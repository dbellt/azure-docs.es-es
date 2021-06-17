---
title: 'Creación y configuración de clústeres de Enterprise Security Package: Azure'
description: Aprenda a crear clústeres de Enterprise Security Package en Azure HDInsight.
services: hdinsight
ms.service: hdinsight
ms.topic: how-to
ms.date: 12/10/2019
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 8ea27e16dc08ccaf475ccc33f08c575ff45f8124
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110692464"
---
# <a name="create-and-configure-enterprise-security-package-clusters-in-azure-hdinsight"></a>Creación y configuración de clústeres de Enterprise Security Package en Azure HDInsight

Enterprise Security Package (ESP) para Azure HDInsight le proporciona acceso a la autenticación basada en Active Directory, compatibilidad con varios usuarios y control de acceso basado en rol para los clústeres de Apache Hadoop en Azure. Los clústeres de ESP para HDInsight permiten a las organizaciones normalmente adheridas a estrictas directivas de seguridad corporativa procesar información confidencial de manera segura.

En esta guía se muestra cómo crear un clúster de Azure HDInsight habilitado para ESP. También muestra cómo crear una máquina virtual de IaaS Windows en la que se habilitan tanto Active Directory y Sistema de nombres de dominio (DNS). Use esta guía para configurar los recursos necesarios para permitir a los usuarios locales iniciar sesión en un clúster de HDInsight habilitado para ESP.

El servidor que cree actuará como sustitución de su entorno local *real*. Lo usará para los pasos de instalación y configuración. Más tarde repetirá los pasos en su propio entorno.

Esta guía también le ayudará a crear un entorno de identidad híbrida mediante la sincronización de hash de contraseñas con Azure Active Directory (Azure AD). La guía complementa [Uso de Enterprise Security Package en HDInsight](apache-domain-joined-architecture.md).

Antes de que use este proceso en su propio entorno:

* Configure Active Directory y DNS.
* Habilite Azure AD.
* Sincronice cuentas de usuario locales en Azure AD.

:::image type="content" source="./media/apache-domain-joined-create-configure-enterprise-security-cluster/hdinsight-image-0002.png" alt-text="Diagrama de arquitectura de Azure AD" border="false":::

## <a name="create-an-on-premises-environment"></a>Creación de un entorno local

En esta sección, usará una plantilla de implementación rápida de Azure para crear nuevas máquinas virtuales, configurar instancias DNS y agregar un nuevo bosque de Active Directory.

1. Vaya a la plantilla de implementación rápida para [crear una máquina virtual de Azure con un nuevo bosque de Active Directory](https://azure.microsoft.com/resources/templates/active-directory-new-domain/).

1. Seleccione **Implementar en Azure**.
1. Inicie sesión en la suscripción de Azure.
1. En la página **Crear una máquina virtual de Azure con un nuevo bosque de AD**, proporcione la siguiente información:

    |Propiedad | Valor |
    |---|---|
    |Subscription|Seleccione la suscripción en la que desea implementar los recursos.|
    |Resource group|Seleccione **Crear nuevo** y escriba el nombre `OnPremADVRG`.|
    |Location|Seleccione una ubicación.|
    |Nombre de usuario administrador|`HDIFabrikamAdmin`|
    |Contraseña de administrador|Escriba una contraseña.|
    |Nombre de dominio|`HDIFabrikam.com`|
    |Prefijo de DNS|`hdifabrikam`|

    Deje los valores predeterminados restantes.

    :::image type="content" source="./media/apache-domain-joined-create-configure-enterprise-security-cluster/create-azure-vm-ad-forest.png" alt-text="Plantilla correspondiente a Creación de una máquina virtual de Azure con un nuevo bosque de Azure AD" border="true":::

1. Revise los **Términos y condiciones** y seleccione **Acepto los términos y condiciones indicados anteriormente**.
1. Seleccione **Comprar**, supervise la implementación y espere a que se complete. La implementación tarda unos 30 minutos en completarse.

## <a name="configure-users-and-groups-for-cluster-access"></a>Configuración de usuarios y grupos para el acceso al clúster

En esta sección, va a crear usuarios que tendrán acceso al clúster de HDInsight al final de esta guía.

1. Conéctese al controlador de dominio mediante Escritorio remoto.
    1. En Azure Portal, vaya a **Grupos de recursos** > **OnPremADVRG** > **adVM** > **Conectar**.
    1. En la lista desplegable **Dirección IP**, seleccione la dirección IP pública.
    1. Seleccione **Descargar archivo RDP** y, a continuación, abra el archivo.
    1. Use `HDIFabrikam\HDIFabrikamAdmin` como nombre de usuario.
    1. Escriba la contraseña que eligió para la cuenta de administrador.
    1. Seleccione **Aceptar**.

1. En el panel del **Administrador del servidor** del controlador de dominio, vaya a **Herramientas** > **Usuarios y equipos de Active Directory**.

    :::image type="content" source="./media/apache-domain-joined-create-configure-enterprise-security-cluster/server-manager-active-directory-screen.png" alt-text="En el panel Administrador del servidor, abra Administración de Active Directory" border="true":::

1. Cree dos nuevos usuarios: **HDIAdmin** y **HDIUser**. Estos dos usuarios iniciarán sesión en los clústeres de HDInsight.

    1. En la página **Usuarios y equipos de Active Directory**, haga clic con el botón derecho en `HDIFabrikam.com` y, a continuación, vaya a **Nuevo** > **Usuario**.

        :::image type="content" source="./media/apache-domain-joined-create-configure-enterprise-security-cluster/create-active-directory-user.png" alt-text="Creación de un usuario de Active Directory nuevo" border="true":::

    1. En la página **Nuevo objeto: usuario**, escriba `HDIUser` para **Nombre** y **Nombre de inicio de sesión de usuario**. Los demás campos se rellenarán automáticamente. Luego, seleccione **Siguiente**.

        :::image type="content" source="./media/apache-domain-joined-create-configure-enterprise-security-cluster/hdinsight-image-0020.png" alt-text="Creación del primer objeto de usuario administrador" border="true":::

    1. En la ventana emergente que aparece, escriba una contraseña para la nueva cuenta. Seleccione **La contraseña nunca expira** y **Aceptar** en el mensaje emergente.
    1. Seleccione **Siguiente** y, a continuación, **Finalizar** para crear la nueva cuenta.
    1. Repita los pasos anteriores para crear el usuario `HDIAdmin`.

        :::image type="content" source="./media/apache-domain-joined-create-configure-enterprise-security-cluster/hdinsight-image-0024.png" alt-text="Creación de un segundo objeto de usuario administrador" border="true":::

1. Cree un grupo de seguridad global.

    1. En **Usuarios y equipos de Active Directory**, haga clic con el botón derecho en `HDIFabrikam.com` y, a continuación, vaya a **Nuevo** > **Grupo**.

    1. Escriba `HDIUserGroup` en el cuadro de texto **Nombre de grupo**.

    1. Seleccione **Aceptar**.

    :::image type="content" source="./media/apache-domain-joined-create-configure-enterprise-security-cluster/create-active-directory-group.png" alt-text="Creación de un grupo de Active Directory nuevo" border="true":::

    :::image type="content" source="./media/apache-domain-joined-create-configure-enterprise-security-cluster/hdinsight-image-0028.png" alt-text="Creación de un nuevo objeto" border="true":::

1. Agregue miembros a **HDIUserGroup**.

    1. Haga clic con el botón derecho en **HDIUser** y seleccione **Agregar a un grupo...** .
    1. En el cuadro de texto **Escriba los nombres de objeto que desea seleccionar**, escriba `HDIUserGroup`. A continuación, seleccione **Aceptar** y **Aceptar** de nuevo en el elemento emergente.
    1. Repita los pasos anteriores para la cuenta **HDIAdmin**.

        :::image type="content" source="./media/apache-domain-joined-create-configure-enterprise-security-cluster/active-directory-add-users-to-group.png" alt-text="Incorporación del miembro HDIUser al grupo HDIUserGroup" border="true":::

Ahora ha creado su entorno de Active Directory. Ha agregado dos usuarios y un grupo de usuarios que pueden obtener acceso al clúster de HDInsight.

Los usuarios se sincronizarán con Azure AD.

### <a name="create-an-azure-ad-directory"></a>Crear un directorio de Azure AD

1. Inicie sesión en Azure Portal.
1. Seleccione **Crear un recurso** y escriba `directory`. Seleccione **Azure Active Directory** > **Crear**.
1. En **Nombre de la organización**, escriba `HDIFabrikam`.
1. En **Nombre de dominio inicial**, escriba `HDIFabrikamoutlook`.
1. Seleccione **Crear**.

   :::image type="content" source="./media/apache-domain-joined-create-configure-enterprise-security-cluster/create-new-directory.png" alt-text="Crear un directorio de Azure AD" border="true":::

### <a name="create-a-custom-domain"></a>Creación de un dominio personalizado

1. En el nuevo directorio de **Azure Active Directory**, en **Administrar**, seleccione **Nombres de dominio personalizado**.
1. Seleccione **+ Agregar dominio personalizado**.
1. En **Nombre de dominio personalizado**, escriba `HDIFabrikam.com` y, a continuación, seleccione **Agregar dominio**.
1. A continuación, complete [Adición de la información de DNS en el registrador de dominios](../../active-directory/fundamentals/add-custom-domain.md#add-your-dns-information-to-the-domain-registrar).

   :::image type="content" source="./media/apache-domain-joined-create-configure-enterprise-security-cluster/create-custom-domain.png" alt-text="Creación de un dominio personalizado" border="true":::

### <a name="create-a-group"></a>Creación de un grupo

1. En el nuevo directorio de **Azure Active Directory**, en **Administrar**, seleccione **Grupos**.
1. Seleccione **+ Nuevo grupo**.
1. En el cuadro de texto **Nombre del grupo**, escriba `AAD DC Administrators`.
1. Seleccione **Crear**.

## <a name="configure-your-azure-ad-tenant"></a>Configuración del inquilino de Azure AD

Ahora va a configurar el inquilino de Azure AD de forma que pueda sincronizar usuarios y grupos desde la instancia de Active Directory en el entorno local a la nube.

Cree un administrador de inquilinos de Active Directory.

1. Inicie sesión en Azure Portal y seleccione el inquilino de Azure AD, **HDIFabrikam**.

1. Vaya a **Administrar** > **Usuarios** > **Nuevo usuario**.

1. Escriba la siguiente información del nuevo usuario:

   **Identidad**

   |Propiedad |Descripción |
   |---|---|
   |Nombre de usuario|Escriba `fabrikamazureadmin` en el cuadro de texto. En la lista desplegable Nombre de dominio, seleccione `hdifabrikam.com`.|
   |Nombre| Escriba `fabrikamazureadmin`.|

   **Contraseña**
   1. Seleccione **Permitirme crear la contraseña**.
   1. Escriba una contraseña segura de su elección.

   **Grupos y roles**
   1. Seleccione **0 grupos seleccionados**.
   1. Seleccione **Administradores del controlador de dominio de AAD** y, a continuación, **Seleccionar**.

      :::image type="content" source="./media/apache-domain-joined-create-configure-enterprise-security-cluster/azure-ad-add-group-member.png" alt-text="Cuadro de diálogo Grupos de Azure AD" border="true":::

   1. Seleccione **Usuario**.
   1. Seleccione **Administrador global** y, a continuación, **Seleccionar**.

      :::image type="content" source="./media/apache-domain-joined-create-configure-enterprise-security-cluster/azure-ad-add-role-member.png" alt-text="Cuadro de diálogo Rol de Azure AD" border="true":::

1. Seleccione **Crear**.

1. A continuación, haga que el nuevo usuario inicie sesión en Azure Portal, donde se le pedirá que cambie la contraseña. Deberá hacerlo antes de configurar Microsoft Azure Active Directory Connect.

## <a name="sync-on-premises-users-to-azure-ad"></a>Sincronización de usuarios locales en Azure AD

### <a name="configure-microsoft-azure-active-directory-connect"></a>Configuración de Microsoft Azure Active Directory Connect

1. Desde el controlador de dominio, descargue [Microsoft Azure Active Directory Connect](https://www.microsoft.com/download/details.aspx?id=47594).

1. Abra el archivo ejecutable que descargó y acepte los términos de licencia. Seleccione **Continuar**.

1. Seleccione **Usar configuración rápida**.

1. En la página **Conectar a Azure AD**, escriba el nombre de usuario y la contraseña del administrador global de Azure AD. Use el nombre de usuario `fabrikamazureadmin@hdifabrikam.com` que creó al configurar su inquilino de Active Directory. Luego, seleccione **Siguiente**.

   :::image type="content" source="./media/apache-domain-joined-create-configure-enterprise-security-cluster/hdinsight-image-0058.png" alt-text="Conectarse a Azure AD" border="true":::

1. En la página **Conectarse a Active Directory Domain Services**, escriba el nombre de usuario y la contraseña de una cuenta de administrador de empresa. Use el nombre de usuario `HDIFabrikam\HDIFabrikamAdmin` y la contraseña que creó anteriormente. Luego, seleccione **Siguiente**.

   :::image type="content" source="./media/apache-domain-joined-create-configure-enterprise-security-cluster/hdinsight-image-0060.png" alt-text="Página Conectar a A D D S." border="true":::

1. En la página **Configuración de inicio de sesión de Azure AD**, seleccione **Siguiente**.

   :::image type="content" source="./media/apache-domain-joined-create-configure-enterprise-security-cluster/hdinsight-image-0062.png" alt-text="Página Configuración de inicio de sesión de Azure AD" border="true":::

1. En la página **Listo para configurar**, seleccione **Instalar**.

   :::image type="content" source="./media/apache-domain-joined-create-configure-enterprise-security-cluster/hdinsight-image-0064.png" alt-text="Página Listo para configurar" border="true":::

1. En la página **Configuración completa**, seleccione **Salir**.
   :::image type="content" source="./media/apache-domain-joined-create-configure-enterprise-security-cluster/hdinsight-image-0078.png" alt-text="Página Configuración completada" border="true":::

1. Una vez completada la sincronización, confirme que los usuarios que creó en el directorio IaaS se sincronizan con Azure AD.
   1. Inicie sesión en Azure Portal.
   1. Seleccione **Azure Active Directory** > **HDIFabrikam** > **Usuarios**.

### <a name="create-a-user-assigned-managed-identity"></a>Crear una identidad administrada asignada por el usuario

Cree una identidad administrada asignada por el usuario que puede usar para configurar Azure AD Domain Services (Azure AD DS). Para obtener más información, consulte [Creación, enumeración, eliminación o asignación de un rol a una identidad administrada que haya asignado el usuario mediante Azure Portal](../../active-directory/managed-identities-azure-resources/how-to-manage-ua-identity-portal.md).

1. Inicie sesión en Azure Portal.
1. Seleccione **Crear un recurso** y escriba `managed identity`. Seleccione **Identidad administrada asignada por el usuario** > **Crear**.
1. En el **Nombre del recurso**, escriba `HDIFabrikamManagedIdentity`.
1. Seleccione su suscripción.
1. En **Grupo de recursos**, seleccione **Crear nuevo** y escriba `HDIFabrikam-CentralUS`.
1. En **Ubicación**, seleccione **Centro de EE. UU.**
1. Seleccione **Crear**.

:::image type="content" source="./media/apache-domain-joined-create-configure-enterprise-security-cluster/hdinsight-image-0082.png" alt-text="Creación de una identidad administrada asignada por el usuario" border="true":::

### <a name="enable-azure-ad-ds"></a>Habilitación de Azure AD DS

Siga estos pasos para habilitar Azure AD DS. Para más información, consulte [Habilitación de Azure AD mediante Azure Portal](../../active-directory-domain-services/tutorial-create-instance.md).

1. Cree una red virtual para hospedar a Azure AD DS. Ejecute el siguiente código de PowerShell.

    ```powershell
    # Sign in to your Azure subscription
    $sub = Get-AzSubscription -ErrorAction SilentlyContinue
    if(-not($sub))
    {
        Connect-AzAccount
    }

    # If you have multiple subscriptions, set the one to use
    # Select-AzSubscription -SubscriptionId "<SUBSCRIPTIONID>"
    
    $virtualNetwork = New-AzVirtualNetwork -ResourceGroupName 'HDIFabrikam-CentralUS' -Location 'Central US' -Name 'HDIFabrikam-AADDSVNET' -AddressPrefix 10.1.0.0/16
    $subnetConfig = Add-AzVirtualNetworkSubnetConfig -Name 'AADDS-subnet' -AddressPrefix 10.1.0.0/24 -VirtualNetwork $virtualNetwork
    $virtualNetwork | Set-AzVirtualNetwork
    ```

1. Inicie sesión en Azure Portal.
1. Seleccione **Crear recurso**, escriba `Domain services` y seleccione **Azure AD Domain Services** > **Crear**.
1. En la página **Básico**:
   1. En **Nombre de directorio**, seleccione el directorio de Azure AD que creó: **HDIFabrikam**.
   1. En **Nombre de dominio DNS**, escriba *HDIFabrikam.com*.
   1. Seleccione su suscripción.
   1. Especifique el grupo de recursos **HDIFabrikam-CentralUS**. En **Ubicación**, seleccione **Centro de EE. UU.**

      :::image type="content" source="./media/apache-domain-joined-create-configure-enterprise-security-cluster/hdinsight-image-0084.png" alt-text="Datos básicos de Azure AD DS" border="true":::

1. En la página **Red**, seleccione la red (**HDIFabrikam-VNET**) y la subred (**AADDS-subnet**) que creó con el script de PowerShell. O bien, elija **Crear nueva** para crear una red virtual ahora.

   :::image type="content" source="./media/apache-domain-joined-create-configure-enterprise-security-cluster/hdinsight-image-0086.png" alt-text="Paso Crear red virtual" border="true":::

1. En la página **Grupo de administradores**, aparecerá una notificación que le indica que un grupo denominado **Administradores de DC de AAD** ya se ha creado para administrar este grupo. Puede modificar la pertenencia de este grupo si lo desea, pero en este caso no es necesario cambiarla. Seleccione **Aceptar**.

   :::image type="content" source="./media/apache-domain-joined-create-configure-enterprise-security-cluster/hdinsight-image-0088.png" alt-text="Visualización del grupo de administradores de Azure AD" border="true":::

1. En la página **Sincronización**, habilite la sincronización completa seleccionando **Todos** > **Aceptar**.

   :::image type="content" source="./media/apache-domain-joined-create-configure-enterprise-security-cluster/hdinsight-image-0090.png" alt-text="Habilitación de la sincronización de Azure AD DS" border="true":::

1. En la página **Resumen**, compruebe los detalles de Azure AD DS y seleccione **Aceptar**.

   :::image type="content" source="./media/apache-domain-joined-create-configure-enterprise-security-cluster/hdinsight-image-0092.png" alt-text="Habilitación de Azure AD Domain Services" border="true":::

Después de habilitar Azure AD DS, un servidor DNS local se ejecuta en las máquinas virtuales de Azure AD.

### <a name="configure-your-azure-ad-ds-virtual-network"></a>Configuración de la red virtual de Azure AD DS

Siga estos pasos para configurar la red virtual de Azure AD DS (**HDIFabrikam-AADDSVNET**) para que use los servidores DNS personalizados.

1. Busque las direcciones IP de los servidores DNS personalizados.
   1. Seleccione el recurso de Azure AD DS `HDIFabrikam.com`.
   1. En **Administrar**, seleccione **Propiedades**.
   1. Busque las direcciones IP en **Dirección IP en la red virtual**.

   :::image type="content" source="./media/apache-domain-joined-create-configure-enterprise-security-cluster/hdinsight-image-0096.png" alt-text="Búsqueda de direcciones IP de DNS personalizadas para Azure AD DS" border="true":::

1. Configure **HDIFabrikam-AADDSVNET** para usar las direcciones IP personalizadas 10.0.0.4 y 10.0.0.5.

   1. En **Configuración**, seleccione **Servidores DNS**.
   1. seleccione **Personalizada**.
   1. En el cuadro de texto, escriba la primera dirección IP (*10.0.0.4*).
   1. Seleccione **Guardar**.
   1. Repita los pasos para agregar la otra dirección IP (*10.0.0.5*).

En nuestro escenario, configuramos Azure AD DS para usar las direcciones IP 10.0.0.4 y 10.0.0.5, configurando la misma dirección IP en la red virtual de Azure AD DS:

:::image type="content" source="./media/apache-domain-joined-create-configure-enterprise-security-cluster/hdinsight-image-0098.png" alt-text="Página de los servidores DNS personalizados" border="true":::

## <a name="securing-ldap-traffic"></a>Protección del tráfico de LDAP

El protocolo ligero de acceso a directorios (LDAP) se usa para leer de Azure Active Directory y escribir en él. Puede hacer que el tráfico de LDAP sea confidencial y seguro mediante la tecnología de Capa de sockets seguros (SSL) o Seguridad de la capa de transporte (TLS). Puede habilitar LDAP a través de SSL (LDAPS) mediante la instalación de un certificado con el formato correcto.

Para obtener más información sobre LDAP seguro, consulte [Configuración de LDAPS para un dominio administrado con Azure AD DS](../../active-directory-domain-services/tutorial-configure-ldaps.md).

En esta sección, cree un certificado autofirmado, descargue el certificado y configure LDAPS para el dominio administrado de Azure AD DS **HDIFabrikam**.

El script siguiente crea un certificado para **HDIFabrikam**. El certificado se guarda en la ruta de acceso *LocalMachine*.

```powershell
$lifetime = Get-Date
New-SelfSignedCertificate -Subject hdifabrikam.com `
-NotAfter $lifetime.AddDays(365) -KeyUsage DigitalSignature, KeyEncipherment `
-Type SSLServerAuthentication -DnsName *.hdifabrikam.com, hdifabrikam.com
```

> [!NOTE]  
> Cualquier utilidad o aplicación que crea una solicitud Public Key Cryptography Standards (PKCS) \#10 válida puede usarse para formar la solicitud de certificado TLS/SSL.

Compruebe que el certificado está instalado en el almacén **Personal** del equipo:

1. Inicie Microsoft Management Console (MMC).
1. Agregue el complemento **Certificados** que administra certificados en el equipo local.
1. Expanda **Certificados (equipo local)**  > **Personal** > **Certificados**. Debe existir un nuevo certificado en el almacén **Personal**. Este certificado se emite para el nombre de host completo.

    :::image type="content" source="./media/apache-domain-joined-create-configure-enterprise-security-cluster/hdinsight-image-0102.png" alt-text="Comprobación de la creación de certificados locales" border="true":::

1. En el panel de la derecha, haga clic con el botón derecho en el certificado que creó. Apunte a **Todas las tareas** y, a continuación, seleccione **Exportar**.

1. En la página **Exportar la clave privada**, seleccione **Exportar la clave privada**. El equipo donde se importará la clave necesita la clave privada para leer los mensajes cifrados.

   :::image type="content" source="./media/apache-domain-joined-create-configure-enterprise-security-cluster/hdinsight-image-0103.png" alt-text="Página Exportar la clave privada del Asistente para exportar certificados" border="true":::

1. En la página **Formato de archivo de exportación**, deje los valores predeterminados y, después, seleccione **Siguiente**.
1. En la página **Contraseña**, escriba una contraseña para la clave privada. En **Cifrado**, seleccione **TripleDES-SHA1**. Luego, seleccione **Siguiente**.
1. En la página **Archivo que se va a exportar**, escriba la ruta y el nombre del archivo de certificado exportado y seleccione **Siguiente**. El nombre de archivo debe tener la extensión .pfx. Este archivo se configura en Azure Portal para establecer una conexión segura.
1. Habilite LDAPS para un dominio administrado de Azure AD DS.
   1. En Azure Portal, seleccione el dominio `HDIFabrikam.com`.
   1. En **Administrar**, seleccione **LDAP seguro**.
   1. En la página **LDAP seguro**, en **LDAP seguro**, seleccione **Habilitar**.
   1. Busque el archivo de certificado .pfx que exportó en el equipo.
   1. Escriba la contraseña del certificado.

   :::image type="content" source="./media/apache-domain-joined-create-configure-enterprise-security-cluster/hdinsight-image-0113.png" alt-text="Habilitación de LDAP seguro" border="true":::

1. Ahora que ha habilitado el protocolo LDAPS, asegúrese de que sea accesible habilitando el puerto 636.
   1. En el grupo de recursos **HDIFabrikam-CentralUS**, seleccione el grupo de seguridad de red **AADDS-HDIFabrikam.com-NSG**.
   1. En **Configuración**, seleccione **Reglas de seguridad de entrada** > **Agregar**.
   1. En la página **Agregar regla de seguridad de entrada**, especifique las siguientes propiedades y seleccione **Agregar**:

      | Propiedad | Valor |
      |---|---|
      | Source | Any |
      | Source port ranges | * |
      | Destination | Any |
      | Destination port range | 636 |
      | Protocolo | Any |
      | Acción | Allow |
      | Priority | \<Desired number> |
      | Nombre | Port_LDAP_636 |

      :::image type="content" source="./media/apache-domain-joined-create-configure-enterprise-security-cluster/add-inbound-security-rule.png" alt-text="Cuadro de diálogo Agregar regla de seguridad de entrada" border="true":::

**HDIFabrikamManagedIdentity** es la identidad administrada asignada por el usuario. El rol de colaborador de Domain Services para HDInsight se habilita para la identidad administrada que permitirá esta identidad para leer, crear, modificar y eliminar las operaciones de Domain Services.

:::image type="content" source="./media/apache-domain-joined-create-configure-enterprise-security-cluster/hdinsight-image-0117.png" alt-text="Creación de una identidad administrada asignada por el usuario" border="true":::

## <a name="create-an-esp-enabled-hdinsight-cluster"></a>Creación de un clúster de HDInsight habilitado para ESP

Este paso requiere los siguientes requisitos previos:

1. Cree un nuevo grupo de recursos *HDIFabrikam-WestUS* en la ubicación **Oeste de EE. UU.**
1. Cree una red virtual que hospedará el clúster de HDInsight habilitado para ESP.

    ```powershell
    $virtualNetwork = New-AzVirtualNetwork -ResourceGroupName 'HDIFabrikam-WestUS' -Location 'West US' -Name 'HDIFabrikam-HDIVNet' -AddressPrefix 10.1.0.0/16
    $subnetConfig = Add-AzVirtualNetworkSubnetConfig -Name 'SparkSubnet' -AddressPrefix 10.1.0.0/24 -VirtualNetwork $virtualNetwork
    $virtualNetwork | Set-AzVirtualNetwork
    ```

1. Cree una relación del mismo nivel entre la red virtual que hospeda Azure AD DS (`HDIFabrikam-AADDSVNET`) y la red virtual que hospedará el clúster de HDInsight habilitado para ESP (`HDIFabrikam-HDIVNet`). Use el siguiente código de PowerShell para emparejar las dos redes virtuales.

    ```powershell
    Add-AzVirtualNetworkPeering -Name 'HDIVNet-AADDSVNet' -RemoteVirtualNetworkId (Get-AzVirtualNetwork -ResourceGroupName 'HDIFabrikam-CentralUS').Id -VirtualNetwork (Get-AzVirtualNetwork -ResourceGroupName 'HDIFabrikam-WestUS')

    Add-AzVirtualNetworkPeering -Name 'AADDSVNet-HDIVNet' -RemoteVirtualNetworkId (Get-AzVirtualNetwork -ResourceGroupName 'HDIFabrikam-WestUS').Id -VirtualNetwork (Get-AzVirtualNetwork -ResourceGroupName 'HDIFabrikam-CentralUS')
    ```

1. Cree una nueva cuenta de Azure Data Lake Storage Gen2 llamada **Hdigen2store**. Configure la cuenta con la identidad administrada por el usuario **HDIFabrikamManagedIdentity**. Para obtener más información, consulte [Uso de Data Lake Storage Gen2 con clústeres de Azure HDInsight](../hdinsight-hadoop-use-data-lake-storage-gen2.md).

1. Configure DNS personalizado en la red virtual **HDIFabrikam-AADDSVNET**.
    1. Vaya a Azure Portal > **Grupos de recursos** > **OnPremADVRG** > **HDIFabrikam-AADDSVNET** > **Servidores DNS**.
    1. Seleccione **Personalizado** y escriba *10.0.0.4* y *10.0.0.5*.
    1. Seleccione **Guardar**.

        :::image type="content" source="./media/apache-domain-joined-create-configure-enterprise-security-cluster/hdinsight-image-0123.png" alt-text="Almacenamiento de la configuración DNS personalizada para una red virtual" border="true":::

1. Cree un nuevo clúster de Spark en HDInsight habilitado para ESP.
    1. Seleccione **Personalizado (tamaño, configuración, aplicaciones)** .
    1. Escriba detalles para **Aspectos básicos** (sección 1). Asegúrese de que el **Tipo de clúster** es **Spark 2.3 (HDI 3.6)** . Asegúrese de que el **Grupo de recursos** es **HDIFabrikam-CentralUS**.

    1. En **Seguridad y redes** (sección 2), rellene los siguientes detalles:
        * En **Enterprise Security Package**, seleccione **Habilitado**.
        * Seleccione **Usuario administrador de clúster** y seleccione la cuenta **HDIAdmin** que creó como usuario administrador local. Haga clic en **Seleccionar**.
        * Seleccione **Grupo de acceso de clúster** > **HDIUserGroup**. Cualquier usuario que agregue a este grupo en el futuro podrá acceder a los clústeres de HDInsight.

            :::image type="content" source="./media/apache-domain-joined-create-configure-enterprise-security-cluster/hdinsight-image-0129.jpg" alt-text="Selección del grupo de acceso de clúster HDIUserGroup" border="true":::

    1. Complete los demás pasos de la configuración del clúster y compruebe los detalles en **Resumen del clúster**. Seleccione **Crear**.

1. Inicie sesión en la interfaz de usuario de Ambari para el clúster recién creado en `https://CLUSTERNAME.azurehdinsight.net`. Use el nombre de usuario de administrador `hdiadmin@hdifabrikam.com` y su contraseña.

    :::image type="content" source="./media/apache-domain-joined-create-configure-enterprise-security-cluster/hdinsight-image-0135.jpg" alt-text="Ventana de inicio de sesión de la interfaz de usuario de Apache Ambari" border="true":::

1. En el panel del clúster, seleccione **Roles**.
1. En la página **Roles**, en **Asignar roles a estos**, junto al rol **Administrador de clústeres**, escriba el grupo *hdiusergroup*.

    :::image type="content" source="./media/apache-domain-joined-create-configure-enterprise-security-cluster/hdinsight-image-0137.jpg" alt-text="Asignación del rol Administrador de clústeres a hdiusergroup" border="true":::

1. Abra su cliente de Secure Shell (SSH) e inicie sesión en el clúster. Use el **hdiuser** que creó en la instancia de Active Directory local.

    :::image type="content" source="./media/apache-domain-joined-create-configure-enterprise-security-cluster/hdinsight-image-0139.jpg" alt-text="Inicio de sesión en el clúster con el cliente SSH" border="true":::

Si puede iniciar sesión con esta cuenta, ha configurado el clúster de ESP correctamente para la sincronización con la instancia de Active Directory local.

## <a name="next-steps"></a>Pasos siguientes

Lea [Introducción a la seguridad de Apache Hadoop con ESP](hdinsight-security-overview.md).
