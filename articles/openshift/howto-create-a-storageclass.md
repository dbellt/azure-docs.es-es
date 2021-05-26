---
title: Creación de una clase StorageClass de Azure Files en Red Hat OpenShift en Azure 4
description: Aprenda a crear de una instancia de StorageClass de Azure Files en Red Hat OpenShift en Azure
ms.service: azure-redhat-openshift
ms.topic: article
ms.date: 10/16/2020
author: grantomation
ms.author: b-grodel
keywords: aro, openshift, az aro, red hat, cli, azure file
ms.custom: mvc, devx-track-azurecli
ms.openlocfilehash: b7d40ecd22a66e1f4327f9147061ce4befd21e67
ms.sourcegitcommit: 42ac9d148cc3e9a1c0d771bc5eea632d8c70b92a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2021
ms.locfileid: "109848004"
---
# <a name="create-an-azure-files-storageclass-on-azure-red-hat-openshift-4"></a>Creación de una clase StorageClass de Azure Files en Red Hat OpenShift en Azure 4

En este artículo, creará instancia de StorageClass para Red Hat OpenShift 4 en Azure que aprovisiona de forma dinámica el almacenamiento ReadWriteMany (RWX) mediante Azure Files. Aprenderá a:

> [!div class="checklist"]
> * Configurar los requisitos previos e instalar las herramientas necesarias
> * Creación de una instancia de StorageClass de Red Hat OpenShift 4 en Azure con el aprovisionador de Azure Files

Si decide instalar y usar la CLI de forma local, en este tutorial es preciso que ejecute la CLI de Azure de la versión 2.6.0, u otra posterior. Ejecute `az --version` para encontrar la versión. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure](/cli/azure/install-azure-cli).

## <a name="before-you-begin"></a>Antes de empezar

Implemente un clúster de Red Hat OpenShift 4 en Azure en la suscripción; vea [Creación de un clúster de Red Hat OpenShift 4 en Azure](tutorial-create-cluster.md)


### <a name="set-up-azure-storage-account"></a>Configuración de una cuenta de almacenamiento de Azure

En este paso creará un grupo de recursos fuera del grupo de recursos del clúster de Red Hat OpenShift en Azure (ARO). Este grupo de recursos contendrá los recursos compartidos de Azure Files creados por el aprovisionador dinámico de Red Hat OpenShift en Azure.

```bash
AZURE_FILES_RESOURCE_GROUP=aro_azure_files
LOCATION=eastus

az group create -l $LOCATION -n $AZURE_FILES_RESOURCE_GROUP

AZURE_STORAGE_ACCOUNT_NAME=aroazurefilessa

az storage account create \
    --name $AZURE_STORAGE_ACCOUNT_NAME \
    --resource-group $AZURE_FILES_RESOURCE_GROUP \
    --kind StorageV2 \
    --sku Standard_LRS
```

## <a name="set-permissions"></a>Establecimiento de permisos
### <a name="set-resource-group-permissions"></a>Establecimiento de permisos de grupo de recursos

La entidad de servicio ARO requiere el permiso "listKeys" en el nuevo grupo de recursos de la cuenta de almacenamiento de Azure. Para lograrlo, asigne el rol "Colaborador".

```bash
ARO_RESOURCE_GROUP=aro-rg
CLUSTER=cluster
ARO_SERVICE_PRINCIPAL_ID=$(az aro show -g $ARO_RESOURCE_GROUP -n $CLUSTER --query servicePrincipalProfile.clientId -o tsv)

az role assignment create --role Contributor --assignee $ARO_SERVICE_PRINCIPAL_ID -g $AZURE_FILES_RESOURCE_GROUP
```

### <a name="set-aro-cluster-permissions"></a>Establecimiento de permisos de clúster de ARO

La cuenta de servicio del enlazador de volumen persistente de OpenShift necesitará la funcionalidad de lectura de secretos. Para conseguirlo, cree y asigne un rol de clúster de OpenShift.
```bash
ARO_API_SERVER=$(az aro list --query "[?contains(name,'$CLUSTER')].[apiserverProfile.url]" -o tsv)

oc login -u kubeadmin -p $(az aro list-credentials -g $ARO_RESOURCE_GROUP -n $CLUSTER --query=kubeadminPassword -o tsv) $ARO_API_SERVER

oc create clusterrole azure-secret-reader \
    --verb=create,get \
    --resource=secrets

oc adm policy add-cluster-role-to-user azure-secret-reader system:serviceaccount:kube-system:persistent-volume-binder
```

## <a name="create-storageclass-with-azure-files-provisioner"></a>Creación de una instancia de StorageClass con el aprovisionador de Azure Files

En este paso se creará una instancia de StorageClass con un aprovisionador de Azure Files. En el manifiesto de StorageClass, los detalles de la cuenta de almacenamiento son necesarios para que el clúster de ARO sepa que debe examinar una cuenta de almacenamiento que está fuera del grupo de recursos actual.

Durante el aprovisionamiento del almacenamiento, se crea un secreto denominado secretName para las credenciales de montaje. En un contexto de varios inquilinos, es muy recomendable establecer el valor de secretNamespace explícitamente; de lo contrario, otros usuarios podrían leer las credenciales de la cuenta de almacenamiento.

```bash
cat << EOF >> azure-storageclass-azure-file.yaml
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: azure-file
provisioner: kubernetes.io/azure-file
parameters:
  location: $LOCATION
  secretNamespace: kube-system
  skuName: Standard_LRS
  storageAccount: $AZURE_STORAGE_ACCOUNT_NAME
  resourceGroup: $AZURE_FILES_RESOURCE_GROUP
reclaimPolicy: Delete
volumeBindingMode: Immediate
EOF

oc create -f azure-storageclass-azure-file.yaml
```

## <a name="change-the-default-storageclass-optional"></a>Cambio del valor predeterminado de StorageClass (opcional)

El valor predeterminado de StorageClass en ARO se denomina managed-premium y usa el aprovisionador azure-disk. Para cambiarlo, emita comandos patch en los manifiestos de StorageClass.

```bash
oc patch storageclass managed-premium -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"false"}}}'

oc patch storageclass azure-file -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
```

## <a name="verify-azure-file-storageclass-optional"></a>Comprobación de StorageClass en Azure Files (opcional)

Cree una aplicación y asígnele almacenamiento.

```bash
oc new-project azfiletest
oc new-app -template httpd-example

#Wait for the pod to become Ready
curl $(oc get route httpd-example -n azfiletest -o jsonpath={.spec.host})

oc set volume dc/httpd-example --add --name=v1 -t pvc --claim-size=1G -m /data

#Wait for the new deployment to rollout
export POD=$(oc get pods --field-selector=status.phase==Running -o jsonpath={.items[].metadata.name})
oc exec $POD -- bash -c "mkdir ./data"
oc exec $POD -- bash -c "echo 'azure file storage' >> /data/test.txt"

oc exec $POD -- bash -c "cat /data/test.txt"
azure file storage
```
El archivo test.txt también será visible a través del Explorador de Storage en Azure Portal.

## <a name="next-steps"></a>Pasos siguientes

En este artículo, ha creado almacenamiento persistente dinámico mediante Microsoft Azure Files y Red Hat OpenShift 4 en Azure. Ha aprendido a:

> [!div class="checklist"]
> * Creación de una cuenta de almacenamiento
> * Configuración de StorageClass en un clúster de Red Hat OpenShift 4 en Azure mediante el aprovisionador de Azure Files

Vaya al artículo siguiente para aprender sobre los recursos compatibles con Red Hat OpenShift en Azure 4.

* [Directiva de soporte técnico de Red Hat OpenShift en Azure](support-policies-v4.md)
