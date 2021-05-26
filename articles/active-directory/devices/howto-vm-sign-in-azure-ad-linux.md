---
title: Inicio de sesión en una máquina virtual con Linux en Azure mediante Azure Active Directory (versión preliminar)
description: Inicio de sesión de Azure AD en una máquina virtual de Azure que ejecuta Linux
services: active-directory
ms.service: active-directory
ms.subservice: devices
ms.topic: how-to
ms.date: 05/20/2021
ms.author: joflore
author: MicrosoftGuyJFlo
manager: daveba
ms.reviewer: sandeo
ms.custom: references_regions, devx-track-azurecli
ms.collection: M365-identity-device-management
ms.openlocfilehash: 34a43212e8883e1ae727d18c53d5c28f873d9e94
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110458110"
---
# <a name="preview-login-to-a-linux-virtual-machine-in-azure-with-azure-active-directory-using-ssh-certificate-based-authentication"></a>Versión preliminar: Inicio de sesión en una máquina virtual con Linux en Azure con Azure Active Directory mediante la autenticación basada en certificados SSH

Para mejorar la seguridad de las máquinas virtuales (VM) con Linux en Azure, puede integrarla con la autenticación de Azure Active Directory (Azure AD). Ahora puede usar Azure AD como plataforma de autenticación principal y una entidad de certificación para conectarse mediante SSH a una VM Linux con la autenticación basada en certificados SSH y AD. Esta funcionalidad permite a las organizaciones controlar y aplicar de forma centralizada el control de acceso basado en rol (RBAC) de Azure y las directivas de acceso condicional que administran el acceso a las máquinas virtuales. En este artículo se muestra cómo crear y configurar una VM Linux e iniciar sesión con Azure AD mediante la autenticación basada en certificados SSH.

> [!IMPORTANT]
> Esta funcionalidad se encuentra actualmente en versión preliminar. [La versión anterior que usaba el flujo de código de dispositivo quedará en desuso el 15 de agosto de 2021](../../virtual-machines/linux/login-using-aad.md). Para migrar de la versión anterior a esta versión, consulte la sección [Migración desde la versión preliminar anterior](#migration-from-previous-preview).
> Esta versión preliminar se ofrece sin contrato de nivel de servicio y no es aconsejable usarla para cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas. Use esta característica en una máquina virtual de prueba que espere descartar después de realizar pruebas. Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Usar Azure AD con la autenticación basada en certificados SSH para iniciar sesión en las VM Linux en Azure implica varias ventajas, entre las que se incluyen:

- Puede usar las credenciales de Azure AD para iniciar sesión en las VM Linux de Azure.
- Puede obtener la autenticación basada en claves SSH sin necesidad de distribuir claves SSH a los usuarios ni aprovisionar claves públicas SSH en las VM Linux de Azure que implemente. Esta experiencia es mucho más sencilla que tener que preocuparse por la distribución de claves públicas SSH obsoletas que podrían provocar accesos no autorizados.
- Puede reducir la dependencia de cuentas de administrador locales, el robo de credenciales y las credenciales débiles.
- Las directivas de complejidad y vigencia de las contraseñas configuradas para Azure AD también ayudan a proteger las VM Linux.
- Con el control de acceso basado en roles de Azure, puede especificar quién puede iniciar sesión en una máquina virtual como usuario habitual o con privilegios de administrador. Cuando los usuarios se unen o dejan el equipo, puede actualizar la directiva de RBAC de Azure de la máquina virtual para conceder acceso según corresponda. Cuando los empleados dejan la organización y su cuenta de usuario se deshabilita o quita de Azure AD, dejan de tener acceso a los recursos.
- Con el acceso condicional, puede configurar directivas para requerir la autenticación multifactor o requerir que el dispositivo cliente que usa para la conexión SSH sea un dispositivo administrado (por ejemplo: un dispositivo compatible o unido a Azure AD híbrido) antes de poder conectarse mediante SSH a las VM Linux. 
- Puede usar las directivas de implementación y auditoría de Azure para requerir el inicio de sesión de Azure AD para las MV Linux y para marcar el uso de cuentas locales no aprobadas en las máquinas virtuales.
- El inicio de sesión en las MV Linux con Azure Active Directory también funciona con clientes que usan los Servicios de federación.

## <a name="supported-linux-distributions-and-azure-regions"></a>Distribuciones de Linux y regiones de Azure compatibles

La versión preliminar de esta característica admite actualmente estas distribuciones de Linux cuando se implementa en una región compatible:

| Distribución | Versión |
| --- | --- |
| CentOS | CentOS 7, CentOS 8.3 |
| Debian | Debian 9/Debian 10 |
| openSUSE | openSUSE Leap 42.3 |
| RedHat Enterprise Linux | RHEL 7.4 a RHEL 7.10, RHEL 8.3 |
| SUSE Linux Enterprise Server | SLES 12 |
| Ubuntu Server | Ubuntu Server 16.04 a Ubuntu Server 20.04 |

La versión preliminar de esta característica actualmente admite estas regiones de Azure:

- Azure Global

> [!Note]
> La versión preliminar de esta característica se admitirá en Azure Government y Azure China en junio de 2021.
 
No se admite el uso de esta extensión en clústeres de Azure Kubernetes Service (AKS). Para obtener más información, consulte [Directivas de soporte técnico para AKS](../../aks/support-policies.md).

Si decide instalar y usar la CLI de forma local, para este tutorial es preciso que ejecute la CLI de Azure de la versión 2.22.1 u otra posterior. Ejecute `az --version` para encontrar la versión. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure](/cli/azure/install-azure-cli).

## <a name="requirements-for-login-with-azure-ad-using-ssh-certificate-based-authentication"></a>Requisitos para iniciar sesión con Azure AD mediante la autenticación basada en certificados SSH

Para habilitar el inicio de sesión de Azure AD mediante la autenticación basada en certificados SSH para las MV Linux en Azure, debe asegurarse de que se cumplan los siguientes requisitos de red, máquina virtual y cliente (cliente ssh).

### <a name="network"></a>Red

La configuración de red de la máquina virtual debe permitir el acceso de salida a los siguientes puntos de conexión a través del puerto TCP 443:

Para Azure Global

- `https://packages.microsoft.com`: para la instalación y las actualizaciones de paquetes.
- `http://169.254.169.254`: punto de conexión Instance Metadata Service de Azure.
- `https://login.microsoftonline.com`: para flujos de autenticación basados en PAM (módulos de autenticación conectables).
- `https://pas.windows.net`: para flujos de Azure RBAC.

Para Azure Government

- `https://packages.microsoft.com`: para la instalación y las actualizaciones de paquetes.
- `http://169.254.169.254`: punto de conexión Instance Metadata Service de Azure.
- `https://login.microsoftonline.us`: para flujos de autenticación basados en PAM (módulos de autenticación conectables).
- `https://pasff.usgovcloudapi.net`: para flujos de Azure RBAC.

Para Azure China

- `https://packages.microsoft.com`: para la instalación y las actualizaciones de paquetes.
- `http://169.254.169.254`: punto de conexión Instance Metadata Service de Azure.
- `https://login.chinacloudapi.cn`: para flujos de autenticación basados en PAM (módulos de autenticación conectables).
- `https://pas.chinacloudapi.cn`: para flujos de Azure RBAC.

### <a name="virtual-machine"></a>Máquina virtual

Asegúrese de que la máquina virtual está configurada con la siguiente funcionalidad:

- Identidad administrada asignada por el sistema. Esta opción se selecciona automáticamente cuando se usa Azure Portal para crear la máquina virtual y seleccionar la opción de inicio de sesión de Azure AD. También puede habilitar la identidad administrada asignada por el sistema en una máquina virtual nueva o existente mediante la CLI de Azure.
- aadsshlogin y aadsshlogin-selinux (según corresponda). Estos paquetes se instalan con la extensión de máquina virtual AADSSHLoginForLinux. La extensión se instala cuando se usa Azure Portal para crear una máquina virtual y habilitar el inicio de sesión de Azure AD (pestaña Administración) o a través de la CLI de Azure.

### <a name="client"></a>Remoto

Asegúrese de que el cliente cumple los requisitos siguientes:

- El cliente SSH debe ser compatible con los certificados basados en OpenSSH para la autenticación. Puede usar la CLI de Az (2.21.1 o posterior) o Azure Cloud Shell para cumplir este requisito. 
- Extensión SSH para la CLI de Az. Puede instalarla con az. No es necesario instalar esta extensión cuando se usa Azure Cloud Shell ya que viene preinstalada.
- Si usa cualquier otro cliente SSH que no sea la CLI de Az o Azure Cloud Shell compatible con OpenSSH, deberá usar la CLI de Az con la extensión SSH para recuperar el certificado SSH efímero de un archivo de configuración y, a continuación, usar el archivo de configuración con el cliente SSH.

## <a name="enabling-azure-ad-login-in-for-linux-vm-in-azure"></a>Habilitación del inicio de sesión de Azure AD para VM Linux en Azure

Para usar el inicio de sesión de Azure AD para una VM Linux en Azure, primero debe habilitar una opción de inicio de sesión de Azure AD para la VM Linux, configurar las asignaciones de roles de Azure para los usuarios que están autorizados para iniciar sesión en la máquina virtual y, a continuación, usar el cliente SSH compatible con OpensSSH, como la CLI de Az o Azure Cloud Shell para conectarse mediante SSH a la VM Linux. Hay varias maneras de habilitar el inicio de sesión de Azure AD para la MV Linux, por ejemplo, puede usar:

- La experiencia de Azure Portal al crear una VM Linux
- La experiencia de Azure Cloud Shell al crear una VM Windows o para una VM Linux existente

### <a name="using-azure-portal-create-vm-experience-to-enable-azure-ad-login"></a>Uso de la experiencia de creación de VM en Azure Portal para habilitar el inicio de sesión de Azure AD

Puede habilitar el inicio de sesión de Azure AD para cualquiera de las distribuciones de Linux admitidas mencionadas anteriormente mediante Azure Portal.

Por ejemplo, para crear una máquina virtual Ubuntu Server 18.04 LTS en Azure con el inicio de sesión de Azure AD:

1. Inicie sesión en el Azure Portal con una cuenta que tenga acceso para crear VM y seleccione **+ Crear un recurso**.
1. Haga clic en **Crear** en **Ubuntu Server 18.04 LTS** en la vista **Popular**.
1. En la pestaña **Administración** 
   1. Active la casilla para habilitar **Iniciar sesión con Azure Active Directory (versión preliminar)** .
   1. Asegúrese de que la opción **Identidad administrada asignada por el sistema** se haya marcado.
1. Pase por el resto de la experiencia de creación de una máquina virtual. Durante esta versión preliminar, tendrá que crear una cuenta de administrador con nombre de usuario y contraseña o clave pública SSH.
 
### <a name="using-the-azure-cloud-shell-experience-to-enable-azure-ad-login"></a>Uso de la experiencia de Azure Cloud Shell para habilitar el inicio de sesión de Azure AD

Azure Cloud Shell es un shell interactivo gratuito que puede usar para ejecutar los pasos de este artículo. Cloud Shell incluye herramientas comunes de Azure preinstaladas y configuradas para que las use con su cuenta. Solo hay que seleccionar el botón Copiar para copiar el código, pegarlo en Cloud Shell y presionar Entrar para que se ejecute. Existen varias maneras de abrir Cloud Shell:

- Seleccione Pruébelo en la esquina superior derecha de un bloque de código.
- Abra Cloud Shell en el explorador.
- Seleccione el botón Cloud Shell en el menú de la esquina superior derecha de Azure Portal.

Si decide instalar y usar la CLI localmente, para este artículo es necesario que ejecute la CLI de Azure versión 2.22.1 o posterior. Ejecute `az --version` para encontrar la versión. Si necesita instalar o actualizar, consulte el artículo Instalación de la CLI de Azure.

1. Cree un grupo de recursos con [az group create](/cli/azure/group#az_group_create).
1. Cree una VM con [az vm create](/cli/azure/vm#az_vm_create&preserve-view=true) con una distribución compatible en una región compatible.
1. Instale la extensión de VM de inicio de Azure AD con [az vm extension set](/cli/azure/vm/extension?view=azure-cli-latest#az_vm_extension_set&preserve-view=true).

En el ejemplo siguiente se implementa una máquina virtual denominada *myVM* que usa *Ubuntu 18.04 LTS* en un grupo de recursos denominado *AzureADLinuxVMPreview* en la región *southcentralus*. Luego instala la *extensión de VM de inicio de sesión de Azure AD* para habilitar el inicio de sesión de Azure AD para VM Linux. Las extensiones de máquina virtual son aplicaciones pequeñas que realizan tareas de automatización y configuración posterior a la implementación en máquinas virtuales de Azure.

El ejemplo se puede personalizar para admitir los requisitos de prueba según sea necesario.

```azurecli-interactive
az group create --name AzureADLinuxVMPreview --location southcentralus

az vm create \
    --resource-group AzureADLinuxVMPreview \
    --name myVM \   
    --image UbuntuLTS \
    --assign-identity \
    --admin-username azureuser \
    --generate-ssh-keys

az vm extension set \
    --publisher Microsoft.Azure.ActiveDirectory \
    --name AADSSHLoginForLinux \
    --resource-group AzureADLinuxVMPreview \
    --vm-name myVM
```

La creación de la máquina virtual y los recursos auxiliares tarda unos minutos en realizarse.

La extensión AADSSHLoginForLinux se puede instalar en una VM Linux existente (distribución compatible) con un agente de máquina virtual en ejecución para habilitar la autenticación de Azure AD. Si implementa esta extensión en una máquina virtual creada anteriormente, asegúrese de que la máquina tenga al menos 1 GB de memoria asignada o la extensión no se instalará.

El estado provisioningState con valor Succeeded aparece una vez que la extensión se instala correctamente en la máquina virtual. La máquina virtual necesita un [agente de máquina virtual](../../virtual-machines/extensions/agent-linux.md) en ejecución para instalar la extensión.

## <a name="configure-role-assignments-for-the-vm"></a>Configuración de asignaciones de roles para la máquina virtual

Ahora que ha creado la VM, debe configurar la directiva RBAC de Azure para determinar quién puede iniciar sesión en la ella. Para autorizar el inicio de sesión de una VM se usan dos roles de Azure:

- **Inicio de sesión de administrador de Virtual Machine**: Los usuarios que tienen asignado este rol pueden iniciar sesión en una máquina virtual de Azure con privilegios de administrador.
- **Inicio de sesión de usuario de Virtual Machine**: los usuarios que tienen asignado este rol pueden iniciar sesión en una máquina virtual de Azure con privilegios de usuario habitual.
 
Para permitir que un usuario inicie sesión en la máquina virtual a través de SSH, debe asignarle el rol Inicio de sesión de administrador de máquina virtual o Inicio de sesión de usuario de máquina virtual. Un usuario de Azure que tiene asignados los roles Propietario o Colaborador para una máquina virtual no tiene automáticamente privilegios de inicio de sesión de Azure AD para la máquina virtual a través de SSH. Esta separación sirve para proporcionar una separación auditada entre el conjunto de personas que controlan máquinas virtuales frente al conjunto de personas que pueden tener acceso a las máquinas virtuales. 

Hay varias maneras de configurar las asignaciones de roles para la VM, por ejemplo, puede usar:

- La experiencia del portal de Azure AD
- La experiencia de Azure Cloud Shell

> [!Note]
> Los roles Inicio de sesión de administrador de máquina virtual e Inicio de sesión de usuario de máquina virtual usan dataActions y, por tanto, no se les puede asignar al ámbito del grupo de administración. Actualmente, estos roles solo se pueden asignar en el ámbito de suscripción, grupo de recursos o recurso. Se recomienda asignar los roles en el nivel de suscripción o recurso y no en el nivel de máquina virtual individual para evitar el riesgo de agotar el [límite de asignaciones de roles de Azure](../../role-based-access-control/troubleshooting.md#azure-role-assignments-limit) por suscripción.

### <a name="using-azure-ad-portal-experience"></a>Mediante la experiencia de Azure AD Portal

Para configurar las asignaciones de roles para las VM Linux habilitadas para Azure AD:

1. Navegue hasta la máquina virtual que se va a configurar.
1. Seleccione **Control de acceso (IAM)** en las opciones de menú.
1. Seleccione **Agregar**, **Agregar asignación de rol** para abrir el panel Agregar asignación de rol.
1. En la lista desplegable **Rol**, seleccione el rol **Inicio de sesión de administrador de máquina virtual** o **Inicio de sesión de usuario de máquina virtual**.
1. En la lista **Seleccionar**, seleccione un usuario, grupo, entidad de servicio o identidad administrada. Si no ve la entidad de seguridad en la lista, puede escribir en el cuadro **Seleccionar** para buscar en el directorio nombres para mostrar, direcciones de correo electrónico e identificadores de objeto.
1. Seleccione **Guardar** para asignar el rol.

Transcurridos unos instantes, se asigna el rol a la entidad de seguridad en el ámbito seleccionado.
 
### <a name="using-the-azure-cloud-shell-experience"></a>Mediante la experiencia de Azure Cloud Shell

En el ejemplo siguiente se usa [az role assignment create](/cli/azure/role/assignment#az_role_assignment_create) para asignar el rol Inicio de sesión de administrador de Virtual Machine a la VM para el usuario de Azure actual. El nombre de usuario de la cuenta de Azure actual se obtiene con [az account show](/cli/azure/account#az_account_show) y el ámbito se establece en la VM que se creó en un paso anterior con [az vm show](/cli/azure/vm#az_vm_show). El ámbito también se podría asignar en un nivel de grupo de recursos o de suscripción, y se aplican los permisos de herencia de Azure RBAC normales.

```azurecli-interactive
username=$(az account show --query user.name --output tsv)
vm=$(az vm show --resource-group AzureADLinuxVMPreview --name myVM --query id -o tsv)

az role assignment create \
    --role "Virtual Machine Administrator Login" \
    --assignee $username \
    --scope $vm
```

> [!NOTE]
> Si su dominio de Azure AD y el dominio del nombre de usuario de inicio de sesión no coinciden, debe especificar el id. de objeto de su cuenta de usuario mediante `--assignee-object-id`, y no solo el nombre de usuario para `--assignee`. Puede obtener el identificador de objeto para su cuenta de usuario mediante [az ad user list](/cli/azure/ad/user#az_ad_user_list).

Para obtener más información sobre cómo usar Azure RBAC para administrar el acceso a los recursos de la suscripción de Azure, consulte el artículo [Pasos para asignar un rol de Azure](../../role-based-access-control/role-assignments-steps.md).

## <a name="install-ssh-extension-for-az-cli"></a>Instalación de la extensión SSH para la CLI de Az

Si usa Azure Cloud Shell, no se necesita ninguna otra configuración, ya que la versión mínima necesaria de la CLI de Az y la extensión SSH para la CLI de Az ya están incluidas en el entorno de Cloud Shell.

Ejecute el siguiente comando para agregar la extensión SSH para la CLI de Az.

```azurecli
az extension add --name ssh
```

La versión mínima necesaria para la extensión es 0.1.4. Compruebe la versión de la extensión SSH instalada con el siguiente comando.

```azurecli
az extension show --name ssh
```

## <a name="using-conditional-access"></a>Uso del acceso condicional

Puede aplicar directivas de acceso condicional, como requerir MFA para el usuario, requerir un dispositivo compatible o unido a Azure AD híbrido para el dispositivo que ejecuta el cliente SSH, comprobar si hay un riesgo bajo de inicio de sesión y de usuario antes de autorizar el acceso a las MV Linux en Azure que están habilitadas con el inicio de sesión de Azure AD. 

Para aplicar la directiva de acceso condicional, debe seleccionar la aplicación "Azure Linux VM Sign-In" (Inicio de sesión de VM Linux de Azure) en la opción de asignación de aplicaciones o acciones en la nube y, a continuación, usar el riesgo de inicio de sesión o de usuario como condición y controles de acceso como Conceder acceso después de satisfacer la solicitud de autenticación multifactor o requerir un dispositivo compatible o unido a Azure AD híbrido.

> [!NOTE]
> La aplicación de directivas de acceso condicional que requieren cumplimiento del dispositivo o la unión a Azure AD híbrido en el dispositivo cliente que ejecuta el cliente SSH solo funciona con la CLI de Az que se ejecuta en Windows y macOS. No se admite cuando se usa la CLI de Az en Linux o Azure Cloud Shell.

## <a name="login-using-azure-ad-user-account-to-ssh-into-the-linux-vm"></a>Inicio de sesión mediante la cuenta de usuario de Azure AD para SSH en la VM Linux

### <a name="using-az-cli"></a>Uso de la CLI de Az

En primer lugar, ejecute az login y, después, az ssh vm.

```azurecli
az login 
```

Este comando iniciará una ventana del explorador y un usuario puede iniciar sesión con su cuenta de Azure AD. 

En el ejemplo siguiente se resuelve automáticamente la dirección IP adecuada para la máquina virtual.

```azurecli
az ssh vm -n myVM -g AzureADLinuxVMPreview
```

Si se le solicita, escriba las credenciales de inicio de sesión de Azure AD en la página de inicio de sesión, realice una MFA o satisfaga las comprobaciones del dispositivo. Solo se le solicitará si la sesión de la CLI de Az no cumple los criterios de acceso condicional necesarios. Cierre la ventana del explorador, vuelva al símbolo del sistema SSH y se conectará automáticamente a la máquina virtual.

Ya inició sesión en la máquina virtual Linux de Azure con los permisos de rol asignados, como Usuario de VM o Administrador de VM. Si la cuenta de usuario tiene asignado el rol Inicio de sesión de administrador de máquina virtual, puede usar sudo para ejecutar los comandos que requieren privilegios raíz.

### <a name="using-az-cloud-shell"></a>Uso de Az Cloud Shell

Puede usar Az Cloud Shell para conectarse a las máquinas virtuales sin necesidad de instalar nada localmente en la máquina cliente. Para iniciar Cloud Shell, haga clic en el icono de shell en la esquina superior derecha de Azure Portal.
 
Az Cloud Shell se conectará automáticamente a una sesión en el contexto del usuario que ha iniciado sesión. Durante el inicio de sesión de Azure AD para Linux Preview, **debe ejecutar az login de nuevo y pasar por un flujo de inicio de sesión interactivo**.

```azurecli
az login
```

Después puede usar los comandos az ssh vm normales para conectarse mediante el nombre y el grupo de recursos o la dirección IP de la máquina virtual.

```azurecli
az ssh vm -n myVM -g AzureADLinuxVMPreview
```

> [!NOTE]
> Cuando se usa Az Cloud Shell, no se admite la aplicación de directivas de acceso condicional que requieran el cumplimiento del dispositivo o la unión a Azure AD híbrido.

### <a name="login-using-azure-ad-service-principal-to-ssh-into-the-linux-vm"></a>Inicio de sesión mediante entidad de servicio de Azure AD para SSH en la VM Linux

La CLI de Azure admite la autenticación con una entidad de servicio en lugar de con una cuenta de usuario. Dado que las entidades de servicio son cuentas no vinculadas a ningún usuario determinado, los clientes pueden usarlas para conectarse mediante SSH a una máquina virtual para admitir cualquier escenario de automatización que puedan tener. La entidad de servicio debe tener asignados derechos de administrador o usuario de máquina virtual. Asigne permisos en el nivel de suscripción o grupo de recursos. 

En el ejemplo siguiente se asignarán derechos de administrador de máquina virtual a la entidad de servicio en el nivel de grupo de recursos. Reemplace los campos de id. de objeto de entidad de servicio, de id. de suscripción y de nombre del grupo de recursos.

```azurecli
az role assignment create \
    --role "Virtual Machine Administrator Login" \
    --assignee-object-id <service-principal-objectid> \
    --assignee-principal-type ServicePrincipal \
    --scope “/subscriptions/<subscription-id>/resourceGroups/<resourcegroup-name>"
```

Use el ejemplo siguiente para autenticarse paren la CLI de Azure mediante la entidad de servicio. Para más información sobre el inicio de sesión con una entidad de servicio, consulte el artículo [Inicio de sesión en la CLI de Azure con una entidad de servicio](/cli/azure/authenticate-azure-cli#sign-in-with-a-service-principal).

```azurecli
az login --service-principal -u <sp-app-id> -p <password-or-cert> --tenant <tenant-id>
```

Una vez completada la autenticación con una entidad de servicio, use los comandos SSH de la CLI de Az normales para conectarse a la máquina virtual.

```azurecli
az ssh vm -n myVM -g AzureADLinuxVMPreview
```

### <a name="exporting-ssh-configuration-for-use-with-ssh-clients-that-support-openssh"></a>Exportación de la configuración de SSH para su uso con clientes SSH compatibles con OpenSSH

El inicio de sesión en las VM Linux de Azure con Azure AD admite la exportación del certificado y la configuración de OpenSSH, lo que le permite usar cualquier cliente SSH que admita certificados basados en OpenSSH para iniciar sesión en Azure AD. En el ejemplo siguiente se exporta la configuración de todas las direcciones IP asignadas a la máquina virtual.

```azurecli
az ssh config --file ~/.ssh/config -n myVM -g AzureADLinuxVMPreview
```

Como alternativa, puede exportar la configuración especificando solo la dirección IP. Reemplace la dirección IP del ejemplo por la dirección IP pública o privada de la máquina virtual. Escriba `az ssh config -h` para obtener ayuda sobre este comando.

```azurecli
az ssh config --file ~/.ssh/config --ip 10.11.123.456
```

A continuación, puede conectarse a la máquina virtual mediante el uso normal de OpenSSH. La conexión se puede realizar a través de cualquier cliente SSH que use OpenSSH.

## <a name="sudo-and-azure-ad-login"></a>Sudo e inicio de sesión de Azure AD

Una vez que los usuarios con el rol de administrador de máquinas virtuales asignado correctamente se conecten mediante SSH a una VM Linux, podrán ejecutar sudo sin ningún otro requisito de interacción o autenticación. Los usuarios que tengan asignado el rol de usuario de máquina virtual no podrán ejecutar sudo.

## <a name="virtual-machine-scale-set-support"></a>Compatibilidad con los conjuntos de escalado de máquinas virtuales

Se admiten los conjuntos de escalado de máquinas virtuales, pero los pasos son ligeramente diferentes para habilitar y conectarse a las máquinas virtuales del conjunto de escalado de máquinas virtuales.

En primer lugar, cree un conjunto de escalado de máquinas virtuales o elija uno que ya exista. Habilite una identidad administrada asignada por el sistema para el conjunto de escalado de máquinas virtuales.

```azurecli
az vmss identity assign --vmss-name myVMSS --resource-group AzureADLinuxVMPreview
```

Instale la extensión de Azure AD en el conjunto de escalado de máquinas virtuales.

```azurecli
az vmss extension set --publisher Microsoft.Azure.ActiveDirectory --name Azure ADSSHLoginForLinux --resource-group AzureADLinuxVMPreview --vmss-name myVMSS
```

Normalmente, el conjunto de escalado de máquinas virtuales no tiene direcciones IP públicas, por lo que debe tener conectividad con ellas desde otra máquina que pueda acceder a su instancia de Azure Virtual Network. En este ejemplo se muestra cómo usar la dirección IP privada de una máquina virtual del conjunto de escalado de máquinas virtuales para conectarse. 

```azurecli
az ssh vm --ip 10.11.123.456
```

> [!NOTE]
> No puede determinar automáticamente las direcciones IP de las máquinas virtuales del conjunto de escalado de máquinas virtuales mediante los conmutadores `--resource-group` y `--name`.

## <a name="migration-from-previous-preview"></a>Migración desde una versión preliminar anterior

Para los clientes que usan la versión anterior del inicio de sesión de Azure AD para Linux que se basaba en el flujo de código del dispositivo, complete los pasos siguientes.

1. Desinstale la extensión AADLoginForLinux en la máquina virtual.
   1. Con la CLI de Azure: `az vm extension delete -g MyResourceGroup --vm-name MyVm -n AADLoginForLinux`
1. Habilite una identidad administrada asignada por el sistema en la máquina virtual.
   1. Con la CLI de Azure: `az vm identity assign -g myResourceGroup -n myVm`
1. Instale la extensión AADLoginForLinux en la máquina virtual.
   1. Con la CLI de Azure:
      ```azurecli
      az vm extension set \
                --publisher Microsoft.Azure.ActiveDirectory \
                --name AADSSHLoginForLinux \
                --resource-group myResourceGroup \
                --vm-name myVM
      ```
## <a name="using-azure-policy-to-ensure-standards-and-assess-compliance"></a>Uso de Azure Policy para garantizar estándares y evaluar el cumplimiento

Use Azure Policy para asegurarse de que el inicio de sesión de Azure AD está habilitado para las máquinas virtuales Linux nuevas y existentes, y evaluar el cumplimiento de su entorno a escala en el panel de cumplimiento de directivas de Azure. Con esta funcionalidad, puede usar muchos niveles de aplicación: puede marcar las VM Linux nuevas y existentes dentro de su entorno que no tengan habilitado el inicio de sesión Azure AD. También puede usar Azure Policy para implementar la extensión de Azure AD en nuevas VM Linux que no tengan habilitado el inicio de sesión de Azure AD, así como corregir las VM Linux existentes con el mismo estándar. Además de estas funcionalidades, también puede usar la directiva para detectar y marcar las VM Linux que tienen cuentas locales no aprobadas creadas en sus máquinas. Para más información, revise [Azure Policy](https://www.aka.ms/AzurePolicy).

## <a name="troubleshoot-sign-in-issues"></a>Solución de problemas con el inicio de sesión

Algunos errores comunes que se producen al intentar establecer SSH con las credenciales de Azure AD incluyen no tener asignado ningún rol de Azure y solicitudes repetidas para iniciar sesión. Use las secciones siguientes para corregir estos problemas.

### <a name="could-not-retrieve-token-from-local-cache"></a>No se pudo recuperar el token de la caché local

Debe ejecutar az login de nuevo y pasar por un flujo de inicio de sesión interactivo. Revise la sección [Uso de Az Cloud Shell](#using-az-cloud-shell).

### <a name="access-denied-azure-role-not-assigned"></a>Acceso denegado: no hay ningún rol de Azure asignado

Si observa el siguiente error en el símbolo del sistema de SSH, compruebe que ha configurado directivas de Azure RBAC para la máquina virtual que concede al usuario los roles de Inicio de sesión de administrador de máquina virtual o Inicio de sesión de usuario de máquina virtual. Si tiene problemas con las asignaciones de roles de Azure, consulte el artículo [Solución de problemas de Azure RBAC](../../role-based-access-control/troubleshooting.md#azure-role-assignments-limit).

### <a name="extension-install-errors"></a>Errores en la instalación de la extensión

La instalación de la extensión de máquina virtual AADSSHLoginForLinux en equipos existentes produce un error con uno de los siguientes códigos de error conocidos:

#### <a name="non-zero-exit-code-22"></a>Código de salida que no es cero: 22

El estado de la extensión de máquina virtual AADSSHLoginForLinux se muestra como En transición en el portal.

Causa 1: este error se debe a que se requiere una identidad administrada asignada por el sistema.

Solución 1: realice estas acciones:

1. Desinstale la extensión con el error.
1. Para habilitar una identidad administrada asignada por el sistema en la máquina virtual de Azure.
1. Vuelva a ejecutar el comando de instalación de la extensión.

#### <a name="non-zero-exit-code-23"></a>Código de salida que no es cero: 23

El estado de la extensión de máquina virtual AADSSHLoginForLinux se muestra como En transición en el portal.

Causa 1: este error se debe a que la extensión de máquina virtual AADLoginForLinux anterior todavía está instalada.

Solución 1: realice estas acciones:

1. Desinstale la extensión de máquina virtual AADLoginForLinux anterior de la máquina virtual. El estado de la nueva extensión de máquina virtual AADSSHLoginForLinux cambiará a Aprovisionamiento realizado correctamente en el portal.

#### <a name="az-ssh-vm-fails-with-keyerror-access_token"></a>Az ssh vm produce un error con KeyError: "access_token".

Causa 1: se usa una versión obsoleta del cliente de la CLI de Azure.

Solución 1: actualice el cliente de la CLI de Azure a la versión 2.21.0 o posterior.

#### <a name="ssh-connection-closed"></a>Se cerró la conexión SSH

Una vez que el usuario ha iniciado sesión correctamente con az login, la conexión a la máquina virtual mediante `az ssh vm -ip <addres>` o `az ssh vm --name <vm_name> -g <resource_group>` produce un error con *Connection closed by <ip_address> port 22* (Conexión cerrada por <dirección_ip> puerto 22).

Causa 1: el usuario no está asignado a ninguno de los roles de Azure RBAC de inicio de sesión de administrador o usuario de máquina virtual dentro del ámbito de esta máquina virtual.

Solución 1: agregue el usuario a alguno de los roles de Azure RBAC de inicio de sesión de administrador o usuario de máquina virtual dentro del ámbito de esta máquina virtual.

Causa 2: el usuario tiene un rol de Azure RBAC necesario, pero la identidad administrada asignada por el sistema se ha deshabilitado en la máquina virtual.

Solución 2: realice estas acciones:

1. Habilite una identidad administrada asignada por el sistema en la máquina virtual.
1. Deje pasar varios minutos antes de intentar conectarse mediante `az ssh vm --ip <ip_address>`.

### <a name="virtual-machine-scale-set-connection-issues"></a>Problemas de conexión del conjunto de escalado de máquinas virtuales

Las conexiones de máquina virtual del conjunto de escalado de máquinas virtuales pueden producir un error si las instancias del conjunto de escalado de máquinas virtuales ejecutan un modelo anterior. La actualización de instancias del conjunto de escalado de máquinas virtuales al modelo más reciente puede resolver problemas, especialmente si no se ha realizado una actualización desde que se instaló la extensión de inicio de sesión de Azure AD. La actualización de una instancia aplica una configuración estándar del conjunto de escalado de máquinas virtuales a la instancia individual.

## <a name="preview-feedback"></a>Comentarios sobre la versión preliminar

Comparta sus comentarios sobre esta Característica en vista previa (GB) o notifique cualquier problema mediante el [foro de comentarios de Azure AD](https://feedback.azure.com/forums/169401-azure-active-directory?category_id=166032).

## <a name="next-steps"></a>Pasos siguientes
