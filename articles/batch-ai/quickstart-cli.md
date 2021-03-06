---
title: 'Guía de inicio rápido de Azure: Aprendizaje de CNTK con Batch AI mediante la CLI de Azure | Microsoft Docs'
description: Aprenda rápidamente a ejecutar un trabajo de aprendizaje de CNTK con Batch AI mediante la CLI de Azure.
services: batch-ai
documentationcenter: na
author: AlexanderYukhanov
manager: Vaman.Bedekar
editor: tysonn
ms.assetid: ''
ms.custom: ''
ms.service: batch-ai
ms.workload: ''
ms.tgt_pltfrm: na
ms.devlang: CLI
ms.topic: quickstart
ms.date: 10/06/2017
ms.author: Alexander.Yukhanov
ms.openlocfilehash: 82e3885021a2f2309dfed456d472e7027b8d6cf2
ms.sourcegitcommit: 8c3267c34fc46c681ea476fee87f5fb0bf858f9e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/09/2018
---
# <a name="run-a-cntk-training-job-using-the-azure-cli"></a>Ejecución de un trabajo de aprendizaje de CNTK mediante la CLI de Azure

Esta guía de inicio rápido detalla el uso de la interfaz de la línea de comandos (CLI) de Azure para ejecutar un trabajo de aprendizaje de Microsoft Cognitive Toolkit (CNTK) mediante el servicio de Batch IA. La CLI de Azure se usa para crear y administrar recursos de Azure desde la línea de comandos o en scripts.

En este ejemplo se utiliza la base de datos MNIST de imágenes manuscritas para entrenar una red neuronal convolucional (CNN) en un clúster de GPU de un solo nodo administrado por Batch AI. 

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

Para esta guía de inicio rápido es preciso que ejecute la versión más reciente de la CLI de Azure. Si necesita instalarla o actualizarla, consulte [Instalación de la CLI de Azure 2.0]( /cli/azure/install-azure-cli).

Los proveedores de recursos de Batch IA también necesitan estar registrados una vez para su suscripción mediante Azure Cloud Shell o la CLI de Azure. Un registro de proveedor puede tardar hasta 15 minutos.

```azurecli
az provider register -n Microsoft.BatchAI
az provider register -n Microsoft.Batch
```


## <a name="create-a-resource-group"></a>Crear un grupo de recursos

Los clústeres y los trabajos de Batch AI son recursos de Azure y se deben colocar en un grupo de recursos de Azure.

Cree un grupo de recursos con el comando [az group create](/cli/azure/group#az_group_create).

En el ejemplo siguiente, se crea un grupo de recursos denominado *myResourceGroup* en la ubicación *eastus*. A continuación, utiliza el comando [az configure](/cli/azure/reference-index#az_configure) para establecer este grupo de recursos y la ubicación como valor predeterminado.

```azurecli
az group create --name myResourceGroup --location eastus

az configure --defaults group=myResourceGroup

az configure --defaults location=eastus
```

>[!NOTE]
>El establecimiento de los valores predeterminados para el comando `az` es un paso opcional. Puede elegir no establecer los valores predeterminados. Si desea establecer los valores predeterminados, debe quitar la configuración predeterminada después de finalizar el tutorial. Quitar la configuración predeterminada mediante los siguientes comandos:
>
>```azurecli
>az configure --defaults group=''
>
>az configure --defaults location=''
>```
>

## <a name="create-a-storage-account"></a>Crear una cuenta de almacenamiento

En este tutorial rápido se usa una cuenta de almacenamiento de Azure para datos de host y scripts para el trabajo de aprendizaje. Cree una cuenta de almacenamiento con el comando [az storage account create](/cli/azure/storage/account#az_storage_account_create).

```azurecli
az storage account create --name mystorageaccount --sku Standard_LRS
```

>[!NOTE]
>Cada cuenta de almacenamiento debe tener un nombre único. En el comando `az` anterior y otros comandos similares de este tutorial, reemplace el valor de la configuración `mystorageaccount` por el nombre de la cuenta de almacenamiento.

## <a name="prepare-azure-file-share"></a>Preparación de un recurso compartido de Azure Files

Con fines meramente ilustrativos, esta guía de inicio rápido usa un recurso compartido de archivos de Azure para hospedar los datos de aprendizaje y los scripts para el trabajo de aprendizaje.

1. Cree un recurso compartido de archivos denominado *batchaiquickstart* mediante el comando [az storage share create](/cli/azure/storage/share#az_storage_share_create).

  ```azurecli
  az storage share create --account-name mystorageaccount --name batchaiquickstart
  ```
2. Cree un directorio en el recurso compartido denominado *mnistcntksample* mediante el comando [az storage directory create](/cli/azure/storage/directory#az_storage_directory_create).

  ```azurecli
  az storage directory create --share-name batchaiquickstart  --name mnistcntksample
  ```

3. Descargue el [paquete de ejemplo](https://batchaisamples.blob.core.windows.net/samples/BatchAIQuickStart.zip?st=2017-09-29T18%3A29%3A00Z&se=2099-12-31T08%3A00%3A00Z&sp=rl&sv=2016-05-31&sr=b&sig=hrAZfbZC%2BQ%2FKccFQZ7OC4b%2FXSzCF5Myi4Cj%2BW3sVZDo%3D) y descomprímalo. Cargue el contenido en el directorio mediante el comando [az storage file upload](/cli/azure/storage/file#az_storage_file_upload):

  ```azurecli
  az storage file upload --share-name batchaiquickstart --source Train-28x28_cntk_text.txt --path mnistcntksample

  az storage file upload --share-name batchaiquickstart --source Test-28x28_cntk_text.txt --path mnistcntksample

  az storage file upload --share-name batchaiquickstart --source ConvNet_MNIST.py --path mnistcntksample
  ```


## <a name="create-gpu-cluster"></a>Creación del clúster de GPU
Use el comando [az batchai cluster create](/cli/azure/batchai/cluster#az_batchai_cluster_create) para crear un clúster de Batch AI que consta de un único nodo de máquina virtual de GPU. En este ejemplo, la máquina virtual ejecuta en la imagen de Ubuntu LTS predeterminada. Especifique `image UbuntuDSVM` en su lugar para ejecutar Microsoft Deep Learning Virtual Machine, que es compatible con plataformas de aprendizaje adicional. El tamaño de NC6 tiene una GPU NVIDIA K80. Monte el recurso compartido de archivos en una carpeta denominada *azurefileshare*. La ruta de acceso completa de esta carpeta en el nodo de proceso de GPU es $AZ_BATCHAI_MOUNT_ROOT/azurefileshare.


```azurecli
az batchai cluster create --name mycluster --vm-size STANDARD_NC6 --image UbuntuLTS --min 1 --max 1 --storage-account-name mystorageaccount --afs-name batchaiquickstart --afs-mount-path azurefileshare --user-name <admin_username> --password <admin_password>
```


Después de creado el clúster, la salida es similar a la siguiente:

```azurecli
{
  "allocationState": "resizing",
  "allocationStateTransitionTime": "2017-10-05T02:09:03.194000+00:00",
  "creationTime": "2017-10-05T02:09:01.998000+00:00",
  "currentNodeCount": 0,
  "errors": null,
  "id": "/subscriptions/10d0b7c6-9243-4713-xxxx-xxxxxxxxxxxx/resourceGroups/myresourcegroup/providers/Microsoft.BatchAI/clusters/mycluster",
  "location": "eastus",
  "name": "mycluster",
  "nodeSetup": {
    "mountVolumes": {
      "azureBlobFileSystems": null,
      "azureFileShares": [
        {
          "accountName": "batchaisamples",
          "azureFileUrl": "https://batchaisamples.file.core.windows.net/batchaiquickstart",
          "credentialsInfo": {
            "accountKey": null,
            "accountKeySecretUrl": null
          },
          "directoryMode": "0777",
          "fileMode": "0777",
          "relativeMountPath": "azurefileshare"
        }
      ],
      "fileServers": null,
      "unmanagedFileSystems": null
    },
    "setupTask": null
  },
  "nodeStateCounts": {
    "idleNodeCount": 0,
    "leavingNodeCount": 0,
    "preparingNodeCount": 0,
    "runningNodeCount": 0,
    "unusableNodeCount": 0
  },
  "provisioningState": "succeeded",
  "provisioningStateTransitionTime": "2017-10-05T02:09:02.857000+00:00",
  "resourceGroup": "myresourcegroup",
  "scaleSettings": {
    "autoScale": null,
    "manual": {
      "nodeDeallocationOption": "requeue",
      "targetNodeCount": 1
    }
  },
  "subnet": {
    "id": null
  },
  "tags": null,
  "type": "Microsoft.BatchAI/Clusters",
  "userAccountSettings": {
    "adminUserName": "demoUser",
    "adminUserPassword": null,
    "adminUserSshPublicKey": null
  },
  "virtualMachineConfiguration": {
    "imageReference": {
      "offer": "UbuntuServer",
      "publisher": "Canonical",
      "sku": "16.04-LTS",
      "version": "latest"
    }
  },
  "vmPriority": "dedicated",
  "vmSize": "STANDARD_NC6"
```
## <a name="get-cluster-status"></a>Obtención del estado del clúster

Para obtener información general sobre el estado del clúster, ejecute el comando [az batchai cluster list](/cli/azure/batchai/cluster#az_batchai_cluster_list):

```azurecli
az batchai cluster list -o table
```

La salida es similar a la siguiente:

```azurecli
Name        Resource Group    VM Size        State     Idle    Running    Preparing    Unusable    Leaving
---------   ----------------  -------------  -------   ------  ---------  -----------  ----------  --------
mycluster   myresourcegroup   STANDARD_NC6   steady    1       0          0            0            0
```

Para más información, ejecute el comando [az batchai cluster show](/cli/azure/batchai/cluster#az_batchai_cluster_show). Devuelve todas las propiedades del clúster que se muestran después de su creación.

El clúster está listo cuando los nodos se han asignado y ha finalizado la preparación (consulte el atributo `nodeStateCounts`). Si se produjo un error, el atributo `errors` contiene la descripción del error.

## <a name="create-training-job"></a>Creación de un trabajo de aprendizaje

Cuando el clúster esté listo, configure y envíe el trabajo de aprendizaje.

1. Cree un archivo de plantilla JSON para la creación de trabajo denominado job.json:

  ```JSON
  {
    "properties": {
        "stdOutErrPathPrefix": "$AZ_BATCHAI_MOUNT_ROOT/azurefileshare",
       "inputDirectories": [{
            "id": "SAMPLE",
            "path": "$AZ_BATCHAI_MOUNT_ROOT/azurefileshare/mnistcntksample"
        }],
        "outputDirectories": [{
            "id": "MODEL",
            "pathPrefix": "$AZ_BATCHAI_MOUNT_ROOT/azurefileshare",
            "pathSuffix": "model",
            "type": "custom"
        }],
        "containerSettings": {
            "imageSourceRegistry": {
                "image": "microsoft/cntk:2.1-gpu-python3.5-cuda8.0-cudnn6.0"
            }
        },
        "nodeCount": 1,
        "cntkSettings": {
            "pythonScriptFilePath": "$AZ_BATCHAI_INPUT_SAMPLE/ConvNet_MNIST.py",
            "commandLineArgs": "$AZ_BATCHAI_INPUT_SAMPLE $AZ_BATCHAI_OUTPUT_MODEL"
        }
    }
  }
  ```
2. Cree un trabajo denominado *myjob* para que se ejecute en el clúster con el comando [az batchai job create](/cli/azure/batchai/job#az_batchai_job_create):

  ```azurecli
  az batchai job create --name myjob --cluster-name mycluster --config job.json
  ```

La salida es similar a la siguiente:

```azurecli
{
  "caffeSettings": null,
  "chainerSettings": null,
  "cluster": {
    "id": "/subscriptions/10d0b7c6-9243-4713-xxxx-xxxxxxxxxxxx/resourceGroups/myresourcegroup/providers/Microsoft.BatchAI/clusters/mycluster",
    "resourceGroup": "myresourcegroup"
  },
  "cntkSettings": {
    "commandLineArgs": "$AZ_BATCHAI_INPUT_SAMPLE $AZ_BATCHAI_OUTPUT_MODEL",
    "configFilePath": null,
    "languageType": "Python",
    "processCount": 1,
    "pythonInterpreterPath": null,
    "pythonScriptFilePath": "$AZ_BATCHAI_INPUT_SAMPLE/ConvNet_MNIST.py"
  },
  "constraints": {
    "maxTaskRetryCount": null,
    "maxWallClockTime": "7 days, 0:00:00"
  },
  "containerSettings": {
    "imageSourceRegistry": {
      "credentials": null,
      "image": "microsoft/cntk:2.1-gpu-python3.5-cuda8.0-cudnn6.0",
      "serverUrl": null
    }
  },
  "creationTime": "2017-10-05T06:41:42.163000+00:00",
  "customToolkitSettings": null,
  "environmentVariables": null,
  "executionInfo": {
    "endTime": null,
    "errors": null,
    "exitCode": null,
    "lastRetryTime": null,
    "retryCount": null,
    "startTime": "2017-10-05T06:41:44.392000+00:00"
  },
  "executionState": "running",
  "executionStateTransitionTime": "2017-10-05T06:41:44.953000+00:00",
  "experimentName": null,
  "id": "/subscriptions/10d0b7c6-9243-4713-xxxx-xxxxxxxxxxxx/resourceGroups/demo/providers/Microsoft.BatchAI/jobs/myjob",
  "inputDirectories": [
    {
      "id": "SAMPLE",
      "path": "$AZ_BATCHAI_MOUNT_ROOT/azurefileshare/mnistcntksample"
    }
  ],
  "jobPreparation": null,
  "location": null,
  "name": "cntk_job",
  "nodeCount": 1,
  "outputDirectories": [
    {
      "createNew": true,
      "id": "MODEL",
      "pathPrefix": "$AZ_BATCHAI_MOUNT_ROOT/azurefileshare",
      "pathSuffix": "model",
      "type": "Custom"
    }
  ],
  "priority": 0,
  "provisioningState": "succeeded",
  "provisioningStateTransitionTime": "2017-10-05T06:41:44.238000+00:00",
  "resourceGroup": "demo",
  "stdOutErrPathPrefix": "$AZ_BATCHAI_MOUNT_ROOT/azurefileshare",
  "tags": null,
  "tensorFlowSettings": null,
  "toolType": "CNTK",
  "type": "Microsoft.BatchAI/Jobs"
}
```

## <a name="monitor-job"></a>Supervisar el trabajo

Use el comando [az batchai job list](/cli/azure/batchai/job#az_batchai_job_list) para obtener información general sobre el estado del trabajo:

```azurecli
az batchai job list -o table
```

La salida es similar a la siguiente:

```azurecli
Name        Resource Group    Cluster    Cluster RG      Nodes  State    Exit code
----------  ----------------  ---------  --------------- -----  -------  -----------
myjob       myresourcegroup   mycluster  myresourcegroup 1      running

```

Para más información, ejecute el comando [az batchai job show](/cli/azure/batchai/job#az_batchai_job_show).

`executionState` contiene el estado de ejecución actual del trabajo:

* `queued`: el trabajo está esperando que los nodos del clúster estén disponibles
* `running`: el trabajo se está ejecutando
*   `succeeded` (o `failed`): el trabajo finaliza y `executionInfo` contiene los detalles del resultado


## <a name="list-stdout-and-stderr-output"></a>Enumeración de la salida stdout y stderr
Use el comando [az batchai job list files](/cli/azure/batchai/job#az_batchai_job_list_files) para enumerar los vínculos a los archivos de registro stdout y stderr:

```azurecli
az batchai job list-files --name myjob --output-directory-id stdouterr
```

La salida es similar a la siguiente:

```azurecli
[
  {
    "contentLength": 733,
    "downloadUrl": "https://batchaisamples.file.core.windows.net/batchaiquickstart/10d0b7c6-9243-4713-91a9-2730375d3a1b/demo/jobs/cntk_job/stderr.txt?sv=2016-05-31&sr=f&sig=Rh%2BuTg9C1yQxm7NfA9YWiKb%2B5FRKqWmEXiGNRDeFMd8%3D&se=2017-10-05T07%3A44%3A38Z&sp=rl",
    "lastModified": "2017-10-05T06:44:38+00:00",
    "name": "stderr.txt"
  },
  {
    "contentLength": 300,
    "downloadUrl": "https://batchaisamples.file.core.windows.net/batchaiquickstart/10d0b7c6-9243-4713-91a9-2730375d3a1b/demo/jobs/cntk_job/stdout.txt?sv=2016-05-31&sr=f&sig=jMhJfQOGry9jr4Hh3YyUFpW5Uaxnp38bhVWNrTTWMtk%3D&se=2017-10-05T07%3A44%3A38Z&sp=rl",
    "lastModified": "2017-10-05T06:44:29+00:00",
    "name": "stdout.txt"
  }
]
```


## <a name="observe-output"></a>Observación de la salida

Puede transmitir o seguir los archivos de salida de un trabajo mientras se está ejecutando. En el ejemplo siguiente se usa el comando [az batchai job stream-file](/cli/azure/batchai/job#az_batchai_job_stream_file) para transmitir el registro de stderr.txt:

```azurecli
az batchai job stream-file --job-name myjob --output-directory-id stdouterr --name stderr.txt
```

La salida es similar a la siguiente. Interrumpa la salida presionando [Ctrl]-[C].

```azurecli
…
Finished Epoch[2 of 40]: [Training] loss = 0.104846 * 60000, metric = 3.00% * 60000 3.849s (15588.5 samples/s);
Finished Epoch[3 of 40]: [Training] loss = 0.077043 * 60000, metric = 2.23% * 60000 3.902s (15376.7 samples/s);
Finished Epoch[4 of 40]: [Training] loss = 0.063050 * 60000, metric = 1.82% * 60000 3.811s (15743.9 samples/s);
…

```

## <a name="delete-resources"></a>Eliminar recursos

Use el comando [az batchai job delete](/cli/azure/batchai/job#az_batchai_job_delete) para eliminar el trabajo:

```azurecli
az batchai job delete --name myjob
```
Use el comando [az batchai cluster delete](/cli/azure/batchai/cluster#az_batchai_cluster_delete) para eliminar el clúster:

```azurecli
az batchai cluster delete --name mycluster
```

Use el comando `az group delete` para eliminar el grupo de recursos que ha creado para esta guía de inicio rápido:

```azurecli
az group delete --name myResourceGroup
```

## <a name="restore-azure-cli-20-default-settings"></a>Restauración de la configuración predeterminada de la CLI de Azure 2.0

Quite el valor predeterminado configurado previamente para el grupo de recursos y la ubicación:

```azurecli
az configure --defaults group=''

az configure --defaults location=''
```

## <a name="next-steps"></a>Pasos siguientes

En esta guía de inicio rápido, ha aprendido cómo ejecutar un trabajo de aprendizaje de CNTK en un clúster de Batch AI, mediante la CLI de Azure. Para más información sobre el uso de Batch AI con kits de herramientas diferentes, consulte las [recetas de aprendizaje](https://github.com/Azure/BatchAI).

Para más información sobre el uso de CLI de Azure 2.0 para administrar Batch AI, consulte la [documentación de github](https://github.com/Azure/BatchAI/blob/master/documentation/using-azure-cli-20.md).
