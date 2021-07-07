---
title: Human Reference Genomes
titleSuffix: Azure Open Datasets
description: Aprenda a usar el conjunto de datos Human Reference Genomes en Azure Open Datasets.
ms.service: open-datasets
ms.topic: sample
author: peterclu
ms.author: peterlu
ms.date: 04/16/2021
ms.openlocfilehash: 2e8afede19a948af1863b6b24f277ab934861f22
ms.sourcegitcommit: c05e595b9f2dbe78e657fed2eb75c8fe511610e7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/11/2021
ms.locfileid: "112039074"
---
# <a name="human-reference-genomes"></a>Human Reference Genomes

Este conjunto de datos incluye dos referencias de genoma humano ensambladas por [Genome Reference Consortium](https://www.ncbi.nlm.nih.gov/grc): Hg19 y Hg38.

Para obtener más información sobre los datos de Hg19 (GRCh37), lea el [informe sobre GRCh37 de NCBI](https://www.ncbi.nlm.nih.gov/assembly/GCF_000001405.13/).

Para obtener más información sobre los datos de Hg38, lea el [informe sobre GRCh38 de NCBI](http://www.ncbi.nlm.nih.gov/assembly/GCF_000001405.26/).

Encontrará más información sobre los datos en el sitio de [RefSeq de NCBI](https://www.ncbi.nlm.nih.gov/refseq/).

[!INCLUDE [Open Dataset usage notice](../../includes/open-datasets-usage-note.md)]

## <a name="data-source"></a>Origen de datos

El contenido de este conjunto de datos procede de dos ubicaciones FTP:

ftp://ftp.ncbi.nih.gov/genomes/refseq/vertebrate_mammalian/Homo_sapiens/all_assembly_versions/GCF_000001405.25_GRCh37.p13/

ftp://ftp.ncbi.nih.gov/genomes/refseq/vertebrate_mammalian/Homo_sapiens/latest_assembly_versions/GCF_000001405.39_GRCh38.p13/

Los nombres de los blobs tienen un prefijo que comienza con el segmento "vertebrate_mammalian" del URI.

## <a name="data-volumes-and-update-frequency"></a>Volúmenes de datos y frecuencia de actualización

Este conjunto de datos contiene unos 10 GB de datos y se actualiza a diario.

## <a name="storage-location"></a>Ubicación de almacenamiento

Este conjunto de datos se almacena en las regiones Oeste de EE. UU. 2 y Centro-oeste de EE. UU. de Azure. Por afinidad, se recomienda asignar recursos de proceso de estas dos regiones.

## <a name="data-access"></a>Acceso a datos

Oeste de EE. UU. 2: https://datasetreferencegenomes.blob.core.windows.net/dataset

Centro-oeste de EE. UU.: https://datasetreferencegenomes-secondary.blob.core.windows.net/dataset

[Token de SAS](/azure/storage/common/storage-sas-overview): sv=2019-02-02&se=2050-01-01T08%3A00%3A00Z&si=prod&sr=c&sig=JtQoPFqiC24GiEB7v9zHLi4RrA2Kd1r%2F3iFt2l9%2FlV8%3D

## <a name="use-terms"></a>Términos de uso

Los datos están disponibles sin restricciones. Para obtener más información y detalles de cita, vea el [sitio de la base de datos de secuencias de referencia de NCBI](https://www.ncbi.nlm.nih.gov/refseq/).

## <a name="contact"></a>Contacto

Si tiene alguna pregunta o comentario sobre este conjunto de datos, póngase en contacto con [Genome Reference Consortium](https://www.ncbi.nlm.nih.gov/grc/contact-us).

## <a name="data-access"></a>Acceso a datos

### <a name="azure-notebooks"></a>Azure Notebooks

# <a name="azure-storage"></a>[azure-storage](#tab/azure-storage)

<!-- nbstart https://opendatasets-api.azure.com/discoveryapi/OpenDataset/DownloadNotebook?serviceType=AzureNotebooks&package=azure-storage&registryId=genomics-reference-genomes -->

> [!TIP]
> **[Descargue en su lugar el cuaderno](https://opendatasets-api.azure.com/discoveryapi/OpenDataset/DownloadNotebook?serviceType=AzureNotebooks&package=azure-storage&registryId=genomics-reference-genomes)** .

## <a name="getting-the-reference-genomes-from-azure-open-datasets"></a>Obtención de los genomas de referencia de Azure Open Datasets

Varios datos de genómica públicos se han cargado como un conjunto de datos de Azure Open Datasets [aquí](https://azure.microsoft.com/services/open-datasets/catalog/). Creamos un servicio de blob vinculado a este conjunto de datos abierto. Puede encontrar ejemplos de procedimientos de llamada de datos de Azure Open Datasets para el conjunto de datos `Reference Genomes` a continuación:

Los usuarios pueden llamar a la ruta de acceso siguiente con este cuaderno y descargarla: https://datasetreferencegenomes.blob.core.windows.net/dataset/vertebrate_mammalian/Homo_sapiens/latest_assembly_versions/GCF_000001405.39_GRCh38.p13/GCF_000001405.39_GRCh38.p13_assembly_structure/genomic_regions_definitions.txt.

**Nota importante:** Los usuarios deben iniciar sesión en su cuenta de Azure a través la CLI de Azure para ver los datos con el SDK de Azure ML. Por otro lado, no necesitan realizar ninguna acción para descargar los datos.

[Instalación de la CLI de Azure](/cli/azure/install-azure-cli).

### <a name="calling-the-data-from--reference-genome-datasets"></a>Llamada a los datos desde "Conjuntos de datos de genomas de referencia"

```python
import azureml.core
print("Azure ML SDK Version: ", azureml.core.VERSION)
```

```python
from azureml.core import  Dataset
reference_dataset = Dataset.File.from_files('https://datasetreferencegenomes.blob.core.windows.net/dataset')
mount = reference_dataset.mount()
```

```python
import os

REF_DIR = '/vertebrate_mammalian/Homo_sapiens/latest_assembly_versions/GCF_000001405.39_GRCh38.p13/GCF_000001405.39_GRCh38.p13_assembly_structure'
path = mount.mount_point + REF_DIR

with mount:
    print(os.listdir(path))
```

```python
import pandas as pd

# create mount context
mount.start()

# specify path to genomic_regions_definitions.txt file
REF_DIR = 'vertebrate_mammalian/Homo_sapiens/latest_assembly_versions/GCF_000001405.39_GRCh38.p13/GCF_000001405.39_GRCh38.p13_assembly_structure'
metadata_filename = '{}/{}/{}'.format(mount.mount_point, REF_DIR, 'genomic_regions_definitions.txt')

# read genomic_regions_definitions.txt file
metadata = pd.read_table(metadata_filename)
metadata
```

### <a name="download-the-specific-file"></a>Descarga del archivo específico

```python
import os
import uuid
import sys
from azure.storage.blob import BlockBlobService, PublicAccess

blob_service_client = BlockBlobService(account_name='datasetreferencegenomes',sas_token='sv=2019-02-02&se=2050-01-01T08%3A00%3A00Z&si=prod&sr=c&sig=JtQoPFqiC24GiEB7v9zHLi4RrA2Kd1r%2F3iFt2l9%2FlV8%3D')     
blob_service_client.get_blob_to_path('dataset/vertebrate_mammalian/Homo_sapiens/latest_assembly_versions/GCF_000001405.39_GRCh38.p13/GCF_000001405.39_GRCh38.p13_assembly_structure', 'genomic_regions_definitions.txt', './genomic_regions_definitions.txt')
```

<!-- nbend -->

---

## <a name="next-steps"></a>Pasos siguientes

Consulte el resto de los conjuntos de datos en el [catálogo de Open Datasets](dataset-catalog.md).