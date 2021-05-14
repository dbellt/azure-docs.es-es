---
title: Configuración de los registros y las métricas locales para la puerta de enlace autohospedada de Azure API Management | Microsoft Docs
description: Aprenda a configurar los registros y las métricas locales para la puerta de enlace autohospedada de Azure API Management en un clúster de Kubernetes.
services: api-management
documentationcenter: ''
author: miaojiang
manager: gwallace
editor: ''
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.topic: article
ms.date: 05/11/2021
ms.author: apimpm
ms.openlocfilehash: a8199f88527cfd1417997c12f9d682be1c60a810
ms.sourcegitcommit: 32ee8da1440a2d81c49ff25c5922f786e85109b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2021
ms.locfileid: "109784546"
---
# <a name="configure-local-metrics-and-logs-for-azure-api-management-self-hosted-gateway"></a>Configuración de los registros y las métricas locales para la puerta de enlace autohospedada de Azure API Management

En este artículo se proporcionan detalles para configurar los registros y las métricas locales para la [puerta de enlace autohospedada](./self-hosted-gateway-overview.md) implementada en un clúster de Kubernetes. Para configurar los registros y las métricas en la nube, consulte [este artículo](how-to-configure-cloud-metrics-logs.md).

## <a name="metrics"></a>Métricas

La puerta de enlace autohospedada admite [StatsD](https://github.com/statsd/statsd) que se ha convertido en un protocolo de unificación para la recopilación y agregación de métricas. En esta sección se describen los pasos para implementar StatsD en Kubernetes, configurar la puerta de enlace para emitir métricas mediante StatsD y usar [Prometheus](https://prometheus.io/) para supervisar las métricas.

### <a name="deploy-statsd-and-prometheus-to-the-cluster"></a>Implementación de StatsD y Prometheus en el clúster

A continuación se muestra un ejemplo de configuración de YAML para la implementación de StatsD y Prometheus en el clúster de Kubernetes en el que se implementa una puerta de enlace autohospedada. También crea un [servicio](https://kubernetes.io/docs/concepts/services-networking/service/) para cada uno de ellos. La puerta de enlace autohospedada publicará métricas en el servicio StatsD. Se obtendrá acceso al panel de Prometheus mediante su servicio.

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: sputnik-metrics-config
data:
  statsd.yaml: ""
  prometheus.yaml: |
    global:
      scrape_interval:     3s
      evaluation_interval: 3s
    scrape_configs:
      - job_name: 'prometheus'
        static_configs:
          - targets: ['localhost:9090']
      - job_name: 'test_metrics'
        static_configs:
          - targets: ['localhost:9102']
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: sputnik-metrics
spec:
  replicas: 1
  selector:
    matchLabels:
      app: sputnik-metrics
  template:
    metadata:
      labels:
        app: sputnik-metrics
    spec:
      containers:
      - name: sputnik-metrics-statsd
        image: prom/statsd-exporter
        ports:
        - name: tcp
          containerPort: 9102
        - name: udp
          containerPort: 8125
          protocol: UDP
        args:
          - --statsd.mapping-config=/tmp/statsd.yaml
          - --statsd.listen-udp=:8125
          - --web.listen-address=:9102
        volumeMounts:
          - mountPath: /tmp
            name: sputnik-metrics-config-files
      - name: sputnik-metrics-prometheus
        image: prom/prometheus
        ports:
        - name: tcp
          containerPort: 9090
        args:
          - --config.file=/tmp/prometheus.yaml
        volumeMounts:
          - mountPath: /tmp
            name: sputnik-metrics-config-files
      volumes:
        - name: sputnik-metrics-config-files
          configMap:
            name: sputnik-metrics-config
---
apiVersion: v1
kind: Service
metadata:
  name: sputnik-metrics-statsd
spec:
  type: NodePort
  ports:
  - name: udp
    port: 8125
    targetPort: 8125
    protocol: UDP
  selector:
    app: sputnik-metrics
---
apiVersion: v1
kind: Service
metadata:
  name: sputnik-metrics-prometheus
spec:
  type: LoadBalancer
  ports:
  - name: http
    port: 9090
    targetPort: 9090
  selector:
    app: sputnik-metrics
```

Guarde las configuraciones en un archivo denominado `metrics.yaml` y use el siguiente comando para implementar todo en el clúster:

```console
kubectl apply -f metrics.yaml
```

Una vez finalizada la implementación, ejecute el siguiente comando para comprobar que los pods se están ejecutando. Tenga en cuenta que el nombre del pod será diferente.

```console
kubectl get pods
NAME                                   READY   STATUS    RESTARTS   AGE
sputnik-metrics-f6d97548f-4xnb7        2/2     Running   0          1m
```

Ejecute el siguiente comando para comprobar que los servicios se están ejecutando: Tome nota de `CLUSTER-IP` y `PORT` del servicio StatsD, ya que los va a necesitar más adelante. Puede visitar el panel de Prometheus con `EXTERNAL-IP` y `PORT`.

```console
kubectl get services
NAME                         TYPE           CLUSTER-IP    EXTERNAL-IP     PORT(S)                      AGE
sputnik-metrics-prometheus   LoadBalancer   10.0.252.72   13.89.141.90    9090:32663/TCP               18h
sputnik-metrics-statsd       NodePort       10.0.41.179   <none>          8125:32733/UDP               18h
```

### <a name="configure-the-self-hosted-gateway-to-emit-metrics"></a>Configuración de la puerta de enlace autohospedada para emitir métricas

Ahora que StatsD y Prometheus se han implementado, podemos actualizar las configuraciones de la puerta de enlace autohospedada para empezar a emitir métricas mediante StatsD. La característica se puede habilitar o deshabilitar con la clave `telemetry.metrics.local` en ConfigMap de la implementación de puerta de enlace autohospedada con opciones adicionales. A continuación se muestra un desglose de las opciones disponibles:

| Campo  | Valor predeterminado | Descripción |
| ------------- | ------------- | ------------- |
| telemetry.metrics.local  | `none` | Habilita el registro mediante StatsD. El valor puede ser `none`, `statsd`. |
| telemetry.metrics.local.statsd.endpoint  | N/D | Especifica el punto de conexión de StatsD. |
| telemetry.metrics.local.statsd.sampling  | N/D | Especifica la frecuencia de muestreo de las métricas. El valor puede estar comprendido entre 0 y 1. Por ejemplo, `0.5`|
| telemetry.metrics.local.statsd.tag-format  | N/D | [Formato de etiquetado](https://github.com/prometheus/statsd_exporter#tagging-extensions) del exportador de StatsD. El valor puede ser `none`, `librato`, `dogStatsD`, `influxDB`. |

A continuación se muestra un ejemplo de configuración:

```yaml
    apiVersion: v1
    kind: ConfigMap
    metadata:
        name: contoso-gateway-environment
    data:
        config.service.endpoint: "<self-hosted-gateway-management-endpoint>"
        telemetry.metrics.local: "statsd"
        telemetry.metrics.local.statsd.endpoint: "10.0.41.179:8125"
        telemetry.metrics.local.statsd.sampling: "1"
        telemetry.metrics.local.statsd.tag-format: "dogStatsD"
```

Actualice el archivo YAML de la implementación de puerta de enlace autohospedada con las configuraciones anteriores y aplique los cambios con el siguiente comando:

```console
kubectl apply -f <file-name>.yaml
```

Para recoger los últimos cambios de configuración, reinicie la implementación de la puerta de enlace con el siguiente comando:

```console
kubectl rollout restart deployment/<deployment-name>
```

### <a name="view-the-metrics"></a>Visualización de las métricas

Ahora que hemos implementado y configurado todo, la puerta de enlace autohospedada debe notificar las métricas mediante StatsD. Prometheus tomará las métricas de StatsD. Vaya al panel de Prometheus mediante `EXTERNAL-IP` y `PORT` del servicio Prometheus.

Realice algunas llamadas de API mediante la puerta de enlace autohospedada; si todo está configurado correctamente, debería poder ver las métricas siguientes:

| Métrica  | Descripción |
| ------------- | ------------- |
| Requests  | Número de solicitudes de API en el período |
| DurationInMS | Número de milisegundos transcurridos desde el momento en que la puerta de enlace recibió la solicitud hasta que se envió toda la respuesta |
| BackendDurationInMS | Número de milisegundos empleados en la E/S global de back-end (conexión, envío y recepción de bytes)  |
| ClientDurationInMS | Número de milisegundos empleados en la E/S global del cliente (conexión, envío y recepción de bytes)  |

## <a name="logs"></a>Registros

La puerta de enlace autohospedada genera registros en `stdout` y `stderr` de forma predeterminada. Puede ver fácilmente los registros con el siguiente comando:

```console
kubectl logs <pod-name>
```

Si la puerta de enlace autohospedada está implementada en Azure Kubernetes Service, puede habilitar [Azure Monitor para contenedores](../azure-monitor/containers/container-insights-overview.md) con el fin de recopilar `stdout` y `stderr` de sus cargas de trabajo y ver los registros en Log Analytics.

La puerta de enlace autohospedada también admite varios protocolos, como `localsyslog`, `rfc5424` y `journal`. En la tabla siguiente se resumen todas las opciones admitidas.

| Campo  | Valor predeterminado | Descripción |
| ------------- | ------------- | ------------- |
| telemetry.logs.std  | `text` | Habilita el registro en flujos estándares. El valor puede ser `none`, `text`, `json`. |
| telemetry.logs.local  | `none` | Habilita el registro local. El valor puede ser `none`, `auto`, `localsyslog`, `rfc5424`, `journal`.  |
| telemetry.logs.local.localsyslog.endpoint  | N/D | Especifica el punto de conexión localsyslog.  |
| telemetry.logs.local.localsyslog.facility  | N/D | Especifica el [código de componente](https://en.wikipedia.org/wiki/Syslog#Facility) de localsyslog. Por ejemplo, `7`
| telemetry.logs.local.rfc5424.endpoint  | N/D | Especifica el punto de conexión rfc5424.  |
| telemetry.logs.local.rfc5424.facility  | N/D | Especifica el código de componente por [rfc5424](https://tools.ietf.org/html/rfc5424). Por ejemplo, `7`  |
| telemetry.logs.local.journal.endpoint  | N/D | Especifica el punto de conexión del diario.  |

A continuación, puede ver un ejemplo de configuración de registro local:

```yaml
    apiVersion: v1
    kind: ConfigMap
    metadata:
        name: contoso-gateway-environment
    data:
        config.service.endpoint: "<self-hosted-gateway-management-endpoint>"
        telemetry.logs.std: "text"
        telemetry.logs.local.localsyslog.endpoint: "/dev/log"
        telemetry.logs.local.localsyslog.facility: "7"
```

## <a name="next-steps"></a>Pasos siguientes

* Para obtener más información sobre la puerta de enlace autohospedada, consulte [Introducción a la puerta de enlace autohospedada de Azure API Management](self-hosted-gateway-overview.md)
* Más información acerca de la [configuración y persistencia de registros en la nube](how-to-configure-local-metrics-logs.md).
