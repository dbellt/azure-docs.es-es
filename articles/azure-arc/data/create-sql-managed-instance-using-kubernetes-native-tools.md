---
title: Creación de una instancia de SQL Managed Instance mediante las herramientas de Kubernetes
description: Creación de una instancia de SQL Managed Instance mediante las herramientas de Kubernetes
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
author: dnethi
ms.author: dinethi
ms.reviewer: mikeray
ms.date: 06/02/2021
ms.topic: how-to
ms.openlocfilehash: 24abb1ece1d307276be736b384c3e5e3c7d40f2a
ms.sourcegitcommit: c385af80989f6555ef3dadc17117a78764f83963
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/04/2021
ms.locfileid: "111407496"
---
# <a name="create-azure-sql-managed-instance-using-kubernetes-tools"></a>Creación de una instancia de Azure SQL Managed Instance mediante las herramientas de Kubernetes

[!INCLUDE [azure-arc-data-preview](../../../includes/azure-arc-data-preview.md)]

## <a name="prerequisites"></a>Requisitos previos

Ya debería haber creado un [controlador de datos de Azure Arc](./create-data-controller.md).

Para crear una instancia de SQL Managed Instance mediante las herramientas de Kubernetes, debe tener instaladas dichas herramientas.  En los ejemplos de este artículo se usará `kubectl`, pero se podrían emplear enfoques similares con otras herramientas de Kubernetes como el panel de Kubernetes, `oc` o `helm` si está familiarizado con esas herramientas y los formatos YAML y JSON de Kubernetes.

[Instalación de la herramienta kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/)

## <a name="overview"></a>Introducción

Para crear una instancia de SQL Managed Instance, debe crear un secreto de Kubernetes para almacenar el inicio de sesión y la contraseña del administrador del sistema de forma segura, y un recurso personalizado de SQL Managed Instance basado en la definición de recursos personalizada de sqlmanagedinstance.

## <a name="create-a-yaml-file"></a>Creación de un archivo YAML

Puede usar el archivo [YAML de plantilla](https://raw.githubusercontent.com/microsoft/azure_arc/main/arc_data_services/deploy/yaml/sqlmi.yaml) como punto de partida para crear su propio archivo YAML de SQL Managed Instance personalizado.  Descargue este archivo en el equipo local y ábralo en un editor de texto.  Resulta útil usar un editor de texto como [VS Code](https://code.visualstudio.com/download) que admita el linting y el resaltado de la sintaxis en archivos YAML.

Este es un archivo YAML de ejemplo:

```yaml
apiVersion: v1
data:
  password: <your base64 encoded password>
  username: <your base64 encoded username>
kind: Secret
metadata:
  name: sql1-login-secret
type: Opaque
---
apiVersion: sql.arcdata.microsoft.com/v1alpha1
kind: sqlmanagedinstance
metadata:
  name: sql1
  annotations:
    exampleannotation1: exampleannotationvalue1
    exampleannotation2: exampleannotationvalue2
  labels:
    examplelabel1: examplelabelvalue1
    examplelabel2: examplelabelvalue2
spec:
  scheduling:
    default:
      resources:
        limits:
          cpu: "2"
          memory: 4Gi
        requests:
          cpu: "1"
          memory: 2Gi
  services:
    primary:
      type: LoadBalancer
  storage:
    backups:
      volumes:
      - className: default # Use default configured storage class or modify storage class based on your Kubernetes environment
        size: 5Gi
    data:
      volumes:
      - className: default # Use default configured storage class or modify storage class based on your Kubernetes environment
        size: 5Gi
    datalogs:
      volumes:
      - className: default # Use default configured storage class or modify storage class based on your Kubernetes environment
        size: 5Gi
    logs:
      volumes:
      - className: default # Use default configured storage class or modify storage class based on your Kubernetes environment
        size: 5Gi
```

### <a name="customizing-the-login-and-password"></a>Personalización del inicio de sesión y la contraseña

Un secreto de Kubernetes se almacena como una cadena codificada en Base64, una para el nombre de usuario y otra para la contraseña.  Deberá codificar en Base64 el inicio de sesión y la contraseña del administrador del sistema y colocarlos en la ubicación del marcador de posición en `data.password` y `data.username`.  No incluya los símbolos `<` y `>` que se proporcionan en la plantilla.

> [!NOTE]
> Para lograr una seguridad óptima, no se permite usar el valor "sa" para el inicio de sesión.
> Siga la [directiva de complejidad de la contraseña](/sql/relational-databases/security/password-policy#password-complexity).

Puede usar una herramienta en línea para codificar en Base64 el nombre de usuario y la contraseña que desee, o bien puede usar las herramientas integradas de la CLI, en función de la plataforma.

PowerShell

```console
[Convert]::ToBase64String([System.Text.Encoding]::UTF8.GetBytes('<your string to encode here>'))

#Example
#[Convert]::ToBase64String([System.Text.Encoding]::UTF8.GetBytes('example'))
```

Linux/macOS

```console
echo -n '<your string to encode here>' | base64

#Example
# echo -n 'example' | base64
```

### <a name="customizing-the-name"></a>Personalización del nombre

La plantilla tiene un valor de "sql1" para el atributo name.  Puede cambiarlo, pero deben ser caracteres que sigan los estándares de nomenclatura de DNS.  También debe cambiar el nombre del secreto para que coincida.  Por ejemplo, si cambia el nombre de la instancia administrada de SQL por "sql2", debe cambiar el nombre del secreto "sql1-login-secret" por "sql2-login-secret".

### <a name="customizing-the-resource-requirements"></a>Personalización de los requisitos de recursos

Puede cambiar los requisitos de los recursos (solicitudes y límites de RAM y núcleo), según sea necesario.  

> [!NOTE]
> Puede obtener más información sobre la [gobernanza de recursos de Kubernetes](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/#resource-units-in-kubernetes).

Requisitos para las solicitudes y los límites de recursos:
- El valor de límite de núcleos es **necesario** para la facturación.
- El resto de límites y solicitudes de recursos son opcionales.
- La solicitud y el límite de núcleos deben ser un valor entero positivo, si se especifican.
- Se requiere un mínimo de 2 núcleos para la solicitud de núcleos, si se especifica.
- El formato del valor de memoria sigue la notación de Kubernetes.  
- Se requiere un mínimo de 2 Gi para la solicitud de memoria, si se especifica.
- Como norma general, debe tener 4 GB de RAM por cada núcleo para los casos de uso de producción.

### <a name="customizing-service-type"></a>Personalización del tipo de servicio

Si lo desea, el tipo de servicio se puede cambiar a NodePort.  Se asignará un número de puerto aleatorio.

### <a name="customizing-storage"></a>Personalización del almacenamiento

Puede personalizar las clases de almacenamiento para que el almacenamiento coincida con su entorno.  Si no está seguro de qué clases de almacenamiento están disponibles, puede ejecutar el comando `kubectl get storageclass` para verlas.  La plantilla tiene el valor predeterminado "default".  Esto significa que existe una clase de almacenamiento con _name_ establecido en "default", no que exista una clase de almacenamiento que _sea_ la predeterminada.  También tiene la opción de cambiar el tamaño del almacenamiento.  Puede obtener más información sobre la [configuración de almacenamiento](./storage-configuration.md).

## <a name="creating-the-sql-managed-instance"></a>Creación de la instancia de SQL Managed Instance

Ahora que ha personalizado el archivo YAML de SQL Managed Instance, puede crear la instancia de SQL Managed Instance mediante la ejecución del siguiente comando:

```console
kubectl create -n <your target namespace> -f <path to your yaml file>

#Example
#kubectl create -n arc -f C:\arc-data-services\sqlmi.yaml
```

## <a name="monitoring-the-creation-status"></a>Supervisión del estado de creación

La creación de la instancia de SQL Managed Instance tardará unos minutos en completarse. Puede supervisar el progreso en otra ventana de terminal con los siguientes comandos:

> [!NOTE]
>  En los siguientes comandos de ejemplo se da por hecho que ha creado una instancia de SQL Managed Instance denominada "sql1" y un espacio de nombres de Kubernetes con el nombre "arc".  Si ha usado otro un nombre para el espacio de nombres o la instancia de SQL Managed Instance, puede reemplazar "arc" y "sqlmi" por sus nombres.

```console
kubectl get sqlmi/sql1 --namespace arc
```

```console
kubectl get pods --namespace arc
```

También puede comprobar el estado de creación de un pod determinado ejecutando un comando como el siguiente.  Esto es especialmente útil para solucionar problemas.

```console
kubectl describe pod/<pod name> --namespace arc

#Example:
#kubectl describe pod/sql1-0 --namespace arc
```

## <a name="troubleshooting-creation-problems"></a>Solución de problemas de creación

Si tiene problemas con la creación, consulte la [guía de solución de problemas](troubleshoot-guide.md).

## <a name="next-steps"></a>Pasos siguientes

[Conexión a SQL Managed Instance habilitada para Azure Arc](connect-managed-instance.md)
