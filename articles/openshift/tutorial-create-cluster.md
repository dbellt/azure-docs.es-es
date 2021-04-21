---
title: 'Tutorial: Creación de un clúster de la versión 4 de Red Hat OpenShift en Azure'
description: Aprenda a crear un clúster de Microsoft Azure Red Hat OpenShift con la CLI de Azure
author: sakthi-vetrivel
ms.author: suvetriv
ms.topic: tutorial
ms.service: azure-redhat-openshift
ms.date: 10/26/2020
ms.openlocfilehash: dda4fc6a80bbe07977f8d2a5ffcbea895a4e1fe6
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107771846"
---
# <a name="tutorial-create-an-azure-red-hat-openshift-4-cluster"></a>Tutorial: Creación de un clúster de la versión 4 de Red Hat OpenShift en Azure

En este tutorial, el primero de un conjunto de tres, preparará el entorno para crear clústeres de Red Hat OpenShift en Azure que ejecute OpenShift 4 y creará un clúster. Aprenderá a:
> [!div class="checklist"]
> * Configurar los requisitos previos. 
> * Crear la red virtual y las subredes necesarias.
> * Implementación de un clúster

## <a name="before-you-begin"></a>Antes de empezar

Si decide instalar y usar la CLI localmente, en este tutorial es preciso que ejecute la versión 2.6.0 u otra posterior. Ejecute `az --version` para encontrar la versión. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure](/cli/azure/install-azure-cli).

Red Hat OpenShift en Azure requiere 40 núcleos como mínimo para crear y ejecutar un clúster de OpenShift. La cuota predeterminada de recursos de Azure para una suscripción nueva de Azure no cumple este requisito. Para solicitar un aumento del límite de recursos, consulte [Cuota estándar: Aumento de los límites por serie de máquinas virtuales](../azure-portal/supportability/per-vm-quota-requests.md).

El secreto de extracción de ARO no cambia el costo de la licencia de Red Hat OpenShift para ARO.

### <a name="verify-your-permissions"></a>Comprobación de los permisos

Durante este tutorial, creará un grupo de recursos que contendrá la red virtual del clúster. Debe tener permisos de administrador de acceso de usuario y de colaborador, o permisos de propietario, ya sea directamente en la red virtual o en el grupo de recursos o la suscripción que lo contienen.

También necesitará permisos suficientes de Azure Active Directory para que las herramientas creen una aplicación y una entidad de servicio en su nombre para el clúster.

### <a name="register-the-resource-providers"></a>Registro de los proveedores de recursos

1. Si tiene varias suscripciones de Azure, especifique el identificador de la relevante:

    ```azurecli-interactive
    az account set --subscription <SUBSCRIPTION ID>
    ```

1. Registre el proveedor de recursos `Microsoft.RedHatOpenShift`:

    ```azurecli-interactive
    az provider register -n Microsoft.RedHatOpenShift --wait
    ```
    
1. Registre el proveedor de recursos `Microsoft.Compute`:

    ```azurecli-interactive
    az provider register -n Microsoft.Compute --wait
    ```
    
1. Registre el proveedor de recursos `Microsoft.Storage`:

    ```azurecli-interactive
    az provider register -n Microsoft.Storage --wait
    ```
    
1. Registre el proveedor de recursos `Microsoft.Authorization`:

    ```azurecli-interactive
    az provider register -n Microsoft.Authorization --wait
    ```

### <a name="get-a-red-hat-pull-secret-optional"></a>Obtención de un secreto de extracción de Red Hat (opcional)

Los secretos de extracción de Red Hat permiten al clúster acceder a los registros de contenedor de Red Hat junto con contenido adicional. Este paso es opcional pero recomendable.

1. [Vaya al portal del administrador de clústeres de Red Hat OpenShift](https://cloud.redhat.com/openshift/install/azure/aro-provisioned) e inicie sesión.

   Tendrá que iniciar sesión en su cuenta de Red Hat, o bien crear una cuenta de Red Hat con su correo electrónico empresarial y aceptar los términos y condiciones.

1. Haga clic en **Download pull secret** (Descargar secreto de extracción) y descargue el secreto de extracción que se usará con el clúster de ARO.

    Mantenga el archivo de `pull-secret.txt` guardado en algún lugar seguro. El archivo se usará en cada creación de un clúster si necesita crear un clúster que incluya ejemplos u operadores para Red Hat o asociados certificados.

    Al ejecutar el comando `az aro create`, puede hacer referencia al secreto de incorporación de cambios mediante el parámetro `--pull-secret @pull-secret.txt`. Ejecute `az aro create` desde el directorio donde haya almacenado el archivo `pull-secret.txt`. De lo contrario, reemplace `@pull-secret.txt` por `@/path/to/my/pull-secret.txt`.

    Si va a copiar el secreto de incorporación de cambios o hacer referencia a él en otros scripts, debe tener el formato de una cadena JSON válida.

### <a name="prepare-a-custom-domain-for-your-cluster-optional"></a>Preparación de un dominio personalizado para el clúster (opcional)

Al ejecutar el comando `az aro create`, puede especificar un dominio personalizado para el clúster mediante el parámetro `--domain foo.example.com`.

Si proporciona un dominio personalizado para el clúster, tenga en cuenta los siguientes puntos:

* Después de crear el clúster, debe crear dos registros de DNS A en el servidor DNS para el `--domain` especificado:
    * **API**: apunta a la dirección IP del servidor de API.
    * **\*.apps**: apunta a la dirección IP de entrada.
    * Para recuperar estos valores, ejecute el comando siguiente después de la creación del clúster: `az aro show -n -g --query '{api:apiserverProfile.ip, ingress:ingressProfiles[0].ip}'`.

* La consola de OpenShift estará disponible en una dirección URL como `https://console-openshift-console.apps.example.com`, en lugar del dominio integrado `https://console-openshift-console.apps.<random>.<location>.aroapp.io`.

* De forma predeterminada, OpenShift usa certificados autofirmados para todas las rutas creadas en los dominios personalizados `*.apps.example.com`.  Si decide usar DNS personalizado después de conectarse al clúster, tendrá que seguir la documentación de OpenShift para [configurar una entidad de certificación personalizada para el controlador de entrada](https://docs.openshift.com/container-platform/4.6/security/certificates/replacing-default-ingress-certificate.html) y una [entidad de certificación personalizada para el servidor de API](https://docs.openshift.com/container-platform/4.6/security/certificates/api-server.html).

### <a name="create-a-virtual-network-containing-two-empty-subnets"></a>Creación de una red virtual que contenga dos subredes vacías

A continuación, creará una red virtual que contenga dos subredes vacías. Si ya tiene una red virtual adaptada a sus necesidades, puede omitir este paso.

1. **Establezca las siguientes variables en el entorno de shell en el que se ejecutarán los comandos `az`.**

   ```console
   LOCATION=eastus                 # the location of your cluster
   RESOURCEGROUP=aro-rg            # the name of the resource group where you want to create your cluster
   CLUSTER=cluster                 # the name of your cluster
   ```

2. **Cree un grupo de recursos.** .

   Un grupo de recursos de Azure es un grupo lógico en el que se implementan y administran recursos de Azure. Cuando se crea un grupo de recursos, se le pide que especifique una ubicación. Esta ubicación es donde se almacenan los metadatos del grupo de recursos, y es también el lugar en el que los recursos se ejecutan en Azure si no se especifica otra región al crearlos. Cree un grupo de recursos con el comando [az group create](/cli/azure/group#az_group_create).
    
   > [!NOTE] 
   > Red Hat OpenShift en Azure no está disponible en todas las regiones en las que se puede crear un grupo de recursos de Azure. Consulte [Regiones disponibles](https://azure.microsoft.com/en-gb/global-infrastructure/services/?products=openshift) para obtener información sobre dónde se admite Red Hat OpenShift en Azure.

   ```azurecli-interactive
   az group create \
     --name $RESOURCEGROUP \
     --location $LOCATION
   ```

   En la siguiente salida de ejemplo se muestra que los recursos se crearon correctamente:

   ```json
   {
     "id": "/subscriptions/<guid>/resourceGroups/aro-rg",
     "location": "eastus",
     "name": "aro-rg",
     "properties": {
       "provisioningState": "Succeeded"
     },
     "type": "Microsoft.Resources/resourceGroups"
   }
   ```

2. **Cree una red virtual.**

   Los clústeres de Red Hat OpenShift en Azure que ejecutan OpenShift 4 requieren una red virtual con dos subredes vacías, una para los nodos maestros y otra para los nodos de trabajo. Puede crear una red virtual para ello o usar una ya existente.

   Cree una red virtual en el mismo grupo de recursos que creó anteriormente:

   ```azurecli-interactive
   az network vnet create \
      --resource-group $RESOURCEGROUP \
      --name aro-vnet \
      --address-prefixes 10.0.0.0/22
   ```

   En la siguiente salida de ejemplo se muestra la red virtual creada correctamente:

   ```json
   {
     "newVNet": {
       "addressSpace": {
         "addressPrefixes": [
           "10.0.0.0/22"
         ]
       },
       "dhcpOptions": {
         "dnsServers": []
       },
       "id": "/subscriptions/<guid>/resourceGroups/aro-rg/providers/Microsoft.Network/virtualNetworks/aro-vnet",
       "location": "eastus",
       "name": "aro-vnet",
       "provisioningState": "Succeeded",
       "resourceGroup": "aro-rg",
       "type": "Microsoft.Network/virtualNetworks"
     }
   }
   ```

3. **Agregue una subred vacía para los nodos maestros.**

   ```azurecli-interactive
   az network vnet subnet create \
     --resource-group $RESOURCEGROUP \
     --vnet-name aro-vnet \
     --name master-subnet \
     --address-prefixes 10.0.0.0/23 \
     --service-endpoints Microsoft.ContainerRegistry
   ```

4. **Agregue una subred vacía para los nodos de trabajo.**

   ```azurecli-interactive
   az network vnet subnet create \
     --resource-group $RESOURCEGROUP \
     --vnet-name aro-vnet \
     --name worker-subnet \
     --address-prefixes 10.0.2.0/23 \
     --service-endpoints Microsoft.ContainerRegistry
   ```

5. **[Deshabilite las directivas de punto de conexión privado](../private-link/disable-private-link-service-network-policy.md) en la subred maestra.** Esto es necesario para que el servicio pueda conectarse al clúster y administrarlo.

   ```azurecli-interactive
   az network vnet subnet update \
     --name master-subnet \
     --resource-group $RESOURCEGROUP \
     --vnet-name aro-vnet \
     --disable-private-link-service-network-policies true
   ```

## <a name="create-the-cluster"></a>Creación del clúster

Ejecute el siguiente comando para crear un clúster. Si decide usar cualquiera de las siguientes opciones, modifique el comando según corresponda:
* También puede [pasar el secreto de incorporación de cambios de Red Hat](#get-a-red-hat-pull-secret-optional), que permite al clúster acceder a los registros de contenedor de Red Hat junto con contenido adicional. Agregue el argumento `--pull-secret @pull-secret.txt` al comando.
* Si lo desea, puede [usar un dominio personalizado](#prepare-a-custom-domain-for-your-cluster-optional). Agregue el argumento `--domain foo.example.com` al comando, reemplazando `foo.example.com` por su propio dominio personalizado.

> [!NOTE]
> Si va a agregar argumentos opcionales al comando, asegúrese de cerrar el argumento de la línea anterior del comando con una barra diagonal inversa al final.

```azurecli-interactive
az aro create \
  --resource-group $RESOURCEGROUP \
  --name $CLUSTER \
  --vnet aro-vnet \
  --master-subnet master-subnet \
  --worker-subnet worker-subnet
```

Después de ejecutar el comando `az aro create`, se tarda aproximadamente 35 minutos en crear un clúster.

## <a name="next-steps"></a>Pasos siguientes

En esta parte del tutorial, ha aprendido a:
> [!div class="checklist"]
> * Configurar los requisitos previos y crear la red virtual y las subredes necesarias
> * Implementación de un clúster

Avance hasta el siguiente tutorial:
> [!div class="nextstepaction"]
> [Conexión a un clúster de Red Hat OpenShift en Azure](tutorial-connect-cluster.md)
