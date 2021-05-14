---
title: archivo de inclusión
description: archivo de inclusión
services: container-service
author: mlearned
ms.service: container-service
ms.topic: include
ms.date: 04/06/2021
ms.author: mlearned
ms.custom: include file
ms.openlocfilehash: da22991b9a1c4b69d3a3d6eb6f76b0925a6ad3d4
ms.sourcegitcommit: 425420fe14cf5265d3e7ff31d596be62542837fb
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107800232"
---
| Resource                                                                                                           | Límite                                                                                                                                                                                                       |
| ------------------------------------------------------------------------------------------------------------------ | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Número máximo de clústeres por suscripción                                                                                  | 5000                                                                                                                                                                                                        |
| Número máximo de nodos por clúster con los conjuntos de disponibilidad de máquina virtual y el SKU básico de Load Balancer                       | 100                                                                                                                                                                                                         |
| Número máximo de nodos por clúster con Virtual Machine Scale Sets y el [SKU estándar de Load Balancer][standard-load-balancer] | 1000 (entre todos los [grupos de nodos][node-pool])                                            |
| Número máximo de grupos de nodos por clúster                                                                                     | 100                                                                                  |
| Número máximo de pods por nodo: [redes básicas][basic-networking] con Kubenet                                           | Máximo: 250 <br /> Valor predeterminado de la CLI de Azure: 110 <br /> Valor predeterminado de la plantilla de Azure Resource Manager: 110 <br /> Valor predeterminado de la implementación de Azure Portal: 30          |
| Número máximo de pods por nodo: [redes avanzadas][advanced-networking] con Azure Container Networking Interface        | Máximo: 250 <br /> Valor predeterminado: 30                                                      |
| Versión preliminar del complemento Open Service Mesh (OSM) AKS                                                                          | Versión del clúster de Kubernetes: 1.19+<sup>1</sup><br />Controladores OSM por clúster: 1<sup>1</sup><br />Pods por controlador OSM: 500<sup>1</sup><br />Cuentas de servicio de Kubernetes administradas por OSM: 50<sup>1</sup> |

<sup>1</sup> El complemento OSM para AKS se encuentra en versión preliminar y se someterá a mejoras adicionales antes de su disponibilidad con carácter general. Durante la fase de versión preliminar, se recomienda no superar los límites indicados.<br />

<!-- LINKS - Internal -->

[basic-networking]: ../articles/aks/concepts-network.md#kubenet-basic-networking
[advanced-networking]: ../articles/aks/concepts-network.md#azure-cni-advanced-networking
[standard-load-balancer]: ../articles/load-balancer/load-balancer-overview.md
[node-pool]: ../articles/aks/use-multiple-node-pools.md

<!-- LINKS - External -->

[azure-support]: https://ms.portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest
