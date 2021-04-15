---
title: 'ML Studio (clásico): Migración a Azure Machine Learning - Execute R Script (Ejecutar script R)'
description: Recompile los módulos Execute R script (Ejecutar script R) de Studio (clásico) para que se ejecuten en Azure Machine Learning.
services: machine-learning
ms.service: machine-learning
ms.subservice: studio-classic
ms.topic: how-to
author: xiaoharper
ms.author: zhanxia
ms.date: 03/08/2021
ms.openlocfilehash: 18d9013b1a5afcf57b54cffc01a15948382210bb
ms.sourcegitcommit: b4fbb7a6a0aa93656e8dd29979786069eca567dc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/13/2021
ms.locfileid: "107311046"
---
# <a name="migrate-execute-r-script-modules-in-studio-classic"></a>Migración de los módulos Execute R Script (Ejecutar script R) en Studio (clásico)

En este artículo, aprenderá a recompilar un módulo **Execute R Script** (Ejecutar script R) de Studio (clásico) en Azure Machine Learning.

Para obtener más información sobre cómo migrar desde Studio (clásico), consulte el [artículo de información general sobre la migración](migrate-overview.md).

## <a name="execute-r-script"></a>Ejecución script de R

El diseñador de Azure Machine Learning ahora se ejecuta en Linux. Studio (clásico) se ejecuta en Windows. Debido al cambio de plataforma, debe ajustar el módulo **Execute script R** (Ejecutar script R) durante la migración; de lo contrario, se producirá un error en la canalización.

Para migrar un módulo **Execute script R** (Ejecutar script R) desde Studio (clásico), debe reemplazar las interfaces `maml.mapInputPort` y `maml.mapOutputPort` por funciones estándar.

En la tabla siguiente se resumen los cambios en el módulo de script R:

|Característica|Studio (clásico)|Diseñador de Azure Machine Learning|
|---|---|---|
|Interfaz de script|`maml.mapInputPort` y `maml.mapOutputPort`|Interfaz de función|
|Plataforma|Windows|Linux|
|Accesible desde Internet |No|Sí|
|Memoria|14 GB|Depende de la SKU de proceso|

### <a name="how-to-update-the-r-script-interface"></a>Cómo actualizar la interfaz del módulo de script R

Este es el contenido de un módulo **Ejecutar script R** (Ejecutar script R) de ejemplo en Studio (clásico):
```r
# Map 1-based optional input ports to variables 
dataset1 <- maml.mapInputPort(1) # class: data.frame 
dataset2 <- maml.mapInputPort(2) # class: data.frame 

# Contents of optional Zip port are in ./src/ 
# source("src/yourfile.R"); 
# load("src/yourData.rdata"); 

# Sample operation 
data.set = rbind(dataset1, dataset2); 

 
# You'll see this output in the R Device port. 
# It'll have your stdout, stderr and PNG graphics device(s). 

plot(data.set); 

# Select data.frame to be sent to the output Dataset port 
maml.mapOutputPort("data.set"); 
```

Este es el contenido actualizado en el diseñador. Observe que `maml.mapInputPort` y `maml.mapOutputPort` se han reemplazado por la interfaz de función estándar `azureml_main`. 
```r
azureml_main <- function(dataframe1, dataframe2){ 
    # Use the parameters dataframe1 and dataframe2 directly 
    dataset1 <- dataframe1 
    dataset2 <- dataframe2 

    # Contents of optional Zip port are in ./src/ 
    # source("src/yourfile.R"); 
    # load("src/yourData.rdata"); 

    # Sample operation 
    data.set = rbind(dataset1, dataset2); 


    # You'll see this output in the R Device port. 
    # It'll have your stdout, stderr and PNG graphics device(s). 
    plot(data.set); 

  # Return datasets as a Named List 

  return(list(dataset1=data.set)) 
} 
```
Para obtener más información, consulte la [referencia del módulo Execute R Script (Ejecutar script R)](../algorithm-module-reference/execute-r-script.md) en el diseñador.

### <a name="install-r-packages-from-the-internet"></a>Instalación de paquetes de R desde Internet

El diseñador de Azure Machine Learning permite instalar paquetes directamente desde CRAN.

Esta es una mejora con respecto a Studio (clásico). Como Studio (clásico) se ejecuta en un entorno de espacio aislado sin acceso a Internet, había que cargar scripts en un conjunto zip para instalar más paquetes. 

Use el código siguiente para instalar paquetes de CRAN en el módulo **Execute R Script** (Ejecutar script R) del diseñador:
```r
  if(!require(zoo)) { 
      install.packages("zoo",repos = "http://cran.us.r-project.org") 
  } 
  library(zoo) 
```

## <a name="next-steps"></a>Pasos siguientes

En este artículo, ha aprendido a migrar los módulos Execute R Script (Ejecutar script R) a Azure Machine Learning.

Consulte los demás artículos de la serie de migración de Studio (clásico):

1. [Información general sobre la migración](migrate-overview.md).
1. [Migración de un conjunto de datos de Studio a Azure Machine Learning](migrate-register-dataset.md).
1. [Recompilación de una canalización de entrenamiento de Studio (clásico)](migrate-rebuild-experiment.md).
1. [Recompilación de un servicio web de Studio (clásico)](migrate-rebuild-web-service.md).
1. [Integración de un servicio web de Azure Machine Learning con aplicaciones cliente](migrate-rebuild-integrate-with-client-app.md).
1. **Migración de módulos de Execute R Script (Ejecutar script R)** .