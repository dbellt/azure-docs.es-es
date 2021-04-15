---
title: 'Detección de idioma: pasos de configuración e implementación de Kubernetes'
titleSuffix: Azure Cognitive Services
description: 'Detección de idioma: pasos de configuración e implementación de Kubernetes'
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: text-analytics
ms.topic: include
ms.date: 04/01/2020
ms.author: aahi
ms.openlocfilehash: 1edca9cf8449ff386d0a9920e7d80d69692536fd
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/30/2021
ms.locfileid: "96017882"
---
### <a name="deploy-the-language-detection-container-to-an-aks-cluster"></a>Implementación del contenedor de detección de idioma en un clúster de AKS

1. Abra la CLI de Azure e inicie sesión en Azure.

    ```azurecli
    az login
    ```

1. Inicie sesión en el clúster de AKS. Reemplace `your-cluster-name` y `your-resource-group` por los valores adecuados.

    ```azurecli
    az aks get-credentials -n your-cluster-name -g -your-resource-group
    ```

    Después de la ejecución de este comando, se muestra un mensaje similar al siguiente:

    ```output
    Merged "your-cluster-name" as current context in /home/username/.kube/config
    ```

    > [!WARNING]
    > Si tiene varias suscripciones disponibles en su cuenta de Azure y el comando `az aks get-credentials` devuelve un error, puede deberse a un problema común de uso de la suscripción incorrecta. Establezca el contexto de la sesión de la CLI de Azure para usar la misma suscripción con la que creó los recursos y vuelva a intentarlo.
    > ```azurecli
    >  az account set -s subscription-id
    > ```

1. Abra el editor de texto que prefiera. En este ejemplo se utiliza Visual Studio Code.

    ```console
    code .
    ```

1. En el editor de texto, cree un nuevo archivo llamado *language.yaml* y pegue el siguiente código YAML en él. Asegúrese de reemplazar `billing/value` y `apikey/value` por su propia información.

    ```yaml
    apiVersion: apps/v1beta1
    kind: Deployment
    metadata:
      name: language
    spec:
      template:
        metadata:
          labels:
            app: language-app
        spec:
          containers:
          - name: language
            image: mcr.microsoft.com/azure-cognitive-services/language
            ports:
            - containerPort: 5000
            resources:
              requests:
                memory: 2Gi
                cpu: 1
              limits:
                memory: 4Gi
                cpu: 1
            env:
            - name: EULA
              value: "accept"
            - name: billing
              value: # {ENDPOINT_URI}
            - name: apikey
              value: # {API_KEY}
     
    --- 
    apiVersion: v1
    kind: Service
    metadata:
      name: language
    spec:
      type: LoadBalancer
      ports:
      - port: 5000
      selector:
        app: language-app
    ```

1. Guarde el archivo y cierre el editor de texto.
1. Ejecute el comando `apply` de Kubernetes con el archivo *language.yaml* como destino:

    ```console
    kubectl apply -f language.yaml
    ```

    Una vez que el comando ha aplicado correctamente la configuración de implementación, se muestra un mensaje similar al siguiente:

    ```output
    deployment.apps "language" created
    service "language" created
    ```
1. Compruebe que el pod está implementado:

    ```console
    kubectl get pods
    ```

    La salida del estado de ejecución del pod:

    ```output
    NAME                         READY     STATUS    RESTARTS   AGE
    language-5c9ccdf575-mf6k5   1/1       Running   0          1m
    ```

1. Compruebe que el servicio está disponible y obtenga la dirección IP.

    ```console
    kubectl get services
    ```

    La salida del estado de ejecución del servicio de *idioma* en el pod:

    ```output
    NAME         TYPE           CLUSTER-IP    EXTERNAL-IP      PORT(S)          AGE
    kubernetes   ClusterIP      10.0.0.1      <none>           443/TCP          2m
    language     LoadBalancer   10.0.100.64   168.61.156.180   5000:31234/TCP   2m
    ```
