---
title: ClinVar Annotations
titleSuffix: Azure Open Datasets
description: Aprenda a usar el conjunto de datos ClinVar Annotations en Azure Open Datasets.
ms.service: open-datasets
ms.topic: sample
author: peterclu
ms.author: peterlu
ms.date: 04/16/2021
ms.openlocfilehash: 384eb565882cf5f62cec1c6d38769647a82ec712
ms.sourcegitcommit: c05e595b9f2dbe78e657fed2eb75c8fe511610e7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/11/2021
ms.locfileid: "112039116"
---
# <a name="clinvar-annotations"></a>ClinVar Annotations

[ClinVar](https://www.ncbi.nlm.nih.gov/clinvar/) es un archivo público de acceso libre de informes sobre las relaciones entre las variantes y los fenotipos humanos, con pruebas que avalan la información. Facilita el acceso a las relaciones afirmadas entre las variantes humanas y el estado de salud observado, así como el historial de cada interpretación, y también facilita la comunicación al respecto. Proporciona acceso a un conjunto más amplio de interpretaciones clínicas que se pueden incorporar a aplicaciones y flujos de trabajo de genómica.

Para obtener más información sobre los datos, consulte el [diccionario de datos](https://www.ncbi.nlm.nih.gov/projects/clinvar/ClinVarDataDictionary.pdf) y las [preguntas más frecuentes](https://www.ncbi.nlm.nih.gov/clinvar/docs/faq/).

[!INCLUDE [Open Dataset usage notice](../../includes/open-datasets-usage-note.md)]

## <a name="data-source"></a>Origen de datos

Este conjunto de datos es un reflejo de ftp://ftp.ncbi.nlm.nih.gov/pub/clinvar/xml/

## <a name="data-volumes-and-update-frequency"></a>Volúmenes de datos y frecuencia de actualización

Este conjunto de datos contiene unos 56 GB de datos y se actualiza a diario.

## <a name="storage-location"></a>Ubicación de almacenamiento

Este conjunto de datos se almacena en las regiones Oeste de EE. UU. 2 y Centro-oeste de EE. UU. de Azure. Por afinidad, se recomienda asignar recursos de proceso de estas dos regiones.

## <a name="data-access"></a>Acceso a datos

Oeste de EE. UU. 2: https://datasetclinvar.blob.core.windows.net/dataset

Centro-oeste de EE. UU.: https://datasetclinvar-secondary.blob.core.windows.net/dataset

[Token de SAS](../storage/common/storage-sas-overview.md): sv=2019-02-02&se=2050-01-01T08%3A00%3A00Z&si=prod&sr=c&sig=qFPPwPba1RmBvaffkzkLuzabYU5dZstSTgMwxuLNME8%3D

## <a name="use-terms"></a>Términos de uso
Los datos están disponibles sin restricciones. Para más información y detalles de cita, consulte [Uso de datos y acceso a estos en ClinVar](https://www.ncbi.nlm.nih.gov/clinvar/docs/maintenance_use/).

## <a name="contact"></a>Contacto

Si tiene alguna pregunta o comentario sobre este conjunto de datos, póngase en contacto con clinvar@ncbi.nlm.nih.gov.

## <a name="data-access"></a>Acceso a datos

### <a name="azure-notebooks"></a>Azure Notebooks

# <a name="azure-storage"></a>[azure-storage](#tab/azure-storage)

<!-- nbstart https://opendatasets-api.azure.com/discoveryapi/OpenDataset/DownloadNotebook?serviceType=AzureNotebooks&package=azure-storage&registryId=genomics-clinvar -->

> [!TIP]
> **[Descargue en su lugar el cuaderno](https://opendatasets-api.azure.com/discoveryapi/OpenDataset/DownloadNotebook?serviceType=AzureNotebooks&package=azure-storage&registryId=genomics-clinvar)** .

## <a name="getting-the-clinvar-data-from-azure-open-dataset"></a>Obtención de los datos ClinVar de Azure Open Datasets

Varios datos de genómica públicos se han cargado como un conjunto de datos de Azure Open Datasets [aquí](https://azure.microsoft.com/services/open-datasets/catalog/). Creamos un servicio de blob vinculado a este conjunto de datos abierto. Puede encontrar ejemplos del procedimiento de llamada a datos de Azure Open Datasets para conjunto de datos `ClinVar` a continuación:

Los usuarios pueden llamar a la ruta de acceso siguiente con este cuaderno y descargarla: https://datasetclinvar.blob.core.windows.net/dataset/ClinVarFullRelease_00-latest.xml.gz.md5.

> [!NOTE]
> Los usuarios deben iniciar sesión en su cuenta de Azure mediante la CLI de Azure para ver los datos con el SDK de Azure ML. Por otro lado, no necesitan realizar ninguna acción para descargar los datos.

Para obtener más información sobre cómo instalar la CLI de Azure, consulte [Instalación de la CLI de Azure](/cli/azure/install-azure-cli).

### <a name="calling-the-data-from--clinvar-data-set"></a>Llamada a los datos desde el "conjunto de datos de ClinVar"

```python
import azureml.core
print("Azure ML SDK Version: ", azureml.core.VERSION)
```

```python
from azureml.core import  Dataset
reference_dataset = Dataset.File.from_files('https://datasetclinvar.blob.core.windows.net/dataset')
mount = reference_dataset.mount()
```

```python
import os

REF_DIR = '/dataset'
path = mount.mount_point + REF_DIR

with mount:
    print(os.listdir(path))
```

```python
import pandas as pd

# create mount context
mount.start()

# specify path to README file
REF_DIR = '/dataset'
metadata_filename = '{}/{}/{}'.format(mount.mount_point, REF_DIR, '_README')

# read README file
metadata = pd.read_table(metadata_filename)
metadata
```

### <a name="download-the-specific-file"></a>Descarga del archivo específico

```python
import os
import uuid
import sys
from azure.storage.blob import BlockBlobService, PublicAccess

blob_service_client = BlockBlobService(account_name='datasetclinvar', sas_token='sv=2019-02-02&se=2050-01-01T08%3A00%3A00Z&si=prod&sr=c&sig=qFPPwPba1RmBvaffkzkLuzabYU5dZstSTgMwxuLNME8%3D')     
blob_service_client.get_blob_to_path('dataset', 'ClinVarFullRelease_00-latest.xml.gz.md5', './ClinVarFullRelease_00-latest.xml.gz.md5')
```

<!-- nbend -->

---

## <a name="next-steps"></a>Pasos siguientes

Consulte el resto de los conjuntos de datos en el [catálogo de Open Datasets](dataset-catalog.md).