---
title: Criar uma imagem de máquina virtual e usar uma identidade gerenciada atribuída pelo usuário para acessar arquivos no armazenamento do Azure (versão prévia)
description: Criar imagem de máquina virtual usando o construtor de imagem do Azure, que pode acessar arquivos armazenados no armazenamento do Azure usando a identidade gerenciada atribuída pelo usuário.
author: cynthn
ms.author: cynthn
ms.date: 05/02/2019
ms.topic: how-to
ms.service: virtual-machines
ms.subservice: imaging
ms.openlocfilehash: f5734d4b1871dd285fc83a72631f7d645e0b72ff
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91307255"
---
# <a name="create-an-image-and-use-a-user-assigned-managed-identity-to-access-files-in-azure-storage"></a>Criar uma imagem e usar uma identidade gerenciada atribuída pelo usuário para acessar arquivos no armazenamento do Azure 

O construtor de imagem do Azure dá suporte ao uso de scripts ou à cópia de arquivos de vários locais, como o GitHub e o armazenamento do Azure, etc. Para usá-los, eles devem ter sido externamente acessíveis ao construtor de imagens do Azure, mas você pode proteger os blobs de armazenamento do Azure usando tokens SAS.

Este artigo mostra como criar uma imagem personalizada usando o construtor de imagem de VM do Azure, em que o serviço usará uma [identidade gerenciada atribuída pelo usuário](../../active-directory/managed-identities-azure-resources/overview.md) para acessar arquivos no armazenamento do Azure para a personalização da imagem, sem a necessidade de tornar os arquivos acessíveis publicamente ou configurar tokens SAS.

No exemplo a seguir, você criará dois grupos de recursos, um será usado para a imagem personalizada e o outro hospedará uma conta de armazenamento do Azure, que contém um arquivo de script. Isso simula um cenário de vida real, no qual você pode ter artefatos de compilação ou arquivos de imagem em diferentes contas de armazenamento, fora do construtor de imagem. Você criará uma identidade atribuída pelo usuário e concederá permissões de leitura no arquivo de script, mas não definirá qualquer acesso público a esse arquivo. Em seguida, você usará o personalizador de Shell para baixar e executar o script da conta de armazenamento.


> [!IMPORTANT]
> O Construtor de Imagens do Azure está atualmente em versão prévia pública.
> Essa versão prévia é fornecida sem um contrato de nível de serviço e não é recomendada para cargas de trabalho de produção. Alguns recursos podem não ter suporte ou podem ter restrição de recursos. Para obter mais informações, consulte [Termos de Uso Complementares de Versões Prévias do Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="register-the-features"></a>Registrar os recursos
Para usar o Construtor de Imagens do Azure durante a versão prévia, você precisa registrar o novo recurso.

```azurecli-interactive
az feature register --namespace Microsoft.VirtualMachineImages --name VirtualMachineTemplatePreview
```

Verifique o status do registro do recurso.

```azurecli-interactive
az feature show --namespace Microsoft.VirtualMachineImages --name VirtualMachineTemplatePreview | grep state
```

Verifique seu registro.


```azurecli-interactive
az provider show -n Microsoft.VirtualMachineImages | grep registrationState
az provider show -n Microsoft.KeyVault | grep registrationState
az provider show -n Microsoft.Compute | grep registrationState
az provider show -n Microsoft.Storage | grep registrationState
```

Caso o status não seja mostrado como registrado, execute o seguinte:

```azurecli-interactive
az provider register -n Microsoft.VirtualMachineImages
az provider register -n Microsoft.Compute
az provider register -n Microsoft.KeyVault
az provider register -n Microsoft.Storage
```


## <a name="create-a-resource-group"></a>Criar um grupo de recursos

Usaremos algumas informações repetidamente, portanto, criaremos algumas variáveis para armazená-las.


```console
# Image resource group name 
imageResourceGroup=aibmdimsi
# storage resource group
strResourceGroup=aibmdimsistor
# Location 
location=WestUS2
# name of the image to be created
imageName=aibCustLinuxImgMsi01
# image distribution metadata reference name
runOutputName=u1804ManImgMsiro
```

Criar uma variável para a ID da assinatura. Obtenha isso usando `az account show | grep id`.

```console
subscriptionID=<Your subscription ID>
```

Crie os grupos de recursos para a imagem e o armazenamento de script.

```console
# create resource group for image template
az group create -n $imageResourceGroup -l $location
# create resource group for the script storage
az group create -n $strResourceGroup -l $location
```

Crie uma identidade atribuída pelo usuário e defina permissões no grupo de recursos.

O Image Builder usará a [identidade do usuário](../../active-directory/managed-identities-azure-resources/qs-configure-cli-windows-vm.md#user-assigned-managed-identity) fornecida para injetar a imagem no grupo de recursos. Neste exemplo, você criará uma definição de função do Azure que tem as ações granulares para executar a distribuição da imagem. A definição de função será então atribuída à identidade do usuário.

```console
# create user assigned identity for image builder to access the storage account where the script is located
idenityName=aibBuiUserId$(date +'%s')
az identity create -g $imageResourceGroup -n $idenityName

# get identity id
imgBuilderCliId=$(az identity show -g $imageResourceGroup -n $idenityName | grep "clientId" | cut -c16- | tr -d '",')

# get the user identity URI, needed for the template
imgBuilderId=/subscriptions/$subscriptionID/resourcegroups/$imageResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/$idenityName

# download preconfigured role definition example
curl https://raw.githubusercontent.com/danielsollondon/azvmimagebuilder/master/solutions/12_Creating_AIB_Security_Roles/aibRoleImageCreation.json -o aibRoleImageCreation.json

# update the definition
sed -i -e "s/<subscriptionID>/$subscriptionID/g" aibRoleImageCreation.json
sed -i -e "s/<rgName>/$imageResourceGroup/g" aibRoleImageCreation.json

# create role definitions
az role definition create --role-definition ./aibRoleImageCreation.json

# grant role definition to the user assigned identity
az role assignment create \
    --assignee $imgBuilderCliId \
    --role "Azure Image Builder Service Image Creation Role" \
    --scope /subscriptions/$subscriptionID/resourceGroups/$imageResourceGroup
```

Crie o armazenamento e copie o script de exemplo para ele do GitHub.

```azurecli-interactive
# script storage account
scriptStorageAcc=aibstorscript$(date +'%s')

# script container
scriptStorageAccContainer=scriptscont$(date +'%s')

# script url
scriptUrl=https://$scriptStorageAcc.blob.core.windows.net/$scriptStorageAccContainer/customizeScript.sh

# create storage account and blob in resource group
az storage account create -n $scriptStorageAcc -g $strResourceGroup -l $location --sku Standard_LRS

az storage container create -n $scriptStorageAccContainer --fail-on-exist --account-name $scriptStorageAcc

# copy in an example script from the GitHub repo 
az storage blob copy start \
    --destination-blob customizeScript.sh \
    --destination-container $scriptStorageAccContainer \
    --account-name $scriptStorageAcc \
    --source-uri https://raw.githubusercontent.com/danielsollondon/azvmimagebuilder/master/quickquickstarts/customizeScript.sh
```

Conceda permissão do construtor de imagem para criar recursos no grupo de recursos de imagem. O `--assignee` valor é a ID de identidade do usuário.

```azurecli-interactive
az role assignment create \
    --assignee $imgBuilderCliId \
    --role "Storage Blob Data Reader" \
    --scope /subscriptions/$subscriptionID/resourceGroups/$strResourceGroup/providers/Microsoft.Storage/storageAccounts/$scriptStorageAcc/blobServices/default/containers/$scriptStorageAccContainer 
```




## <a name="modify-the-example"></a>Modificar o exemplo

Baixe o arquivo example. JSON e configure-o com as variáveis que você criou.

```console
curl https://raw.githubusercontent.com/danielsollondon/azvmimagebuilder/master/quickquickstarts/7_Creating_Custom_Image_using_MSI_to_Access_Storage/helloImageTemplateMsi.json -o helloImageTemplateMsi.json
sed -i -e "s/<subscriptionID>/$subscriptionID/g" helloImageTemplateMsi.json
sed -i -e "s/<rgName>/$imageResourceGroup/g" helloImageTemplateMsi.json
sed -i -e "s/<region>/$location/g" helloImageTemplateMsi.json
sed -i -e "s/<imageName>/$imageName/g" helloImageTemplateMsi.json
sed -i -e "s%<scriptUrl>%$scriptUrl%g" helloImageTemplateMsi.json
sed -i -e "s%<imgBuilderId>%$imgBuilderId%g" helloImageTemplateMsi.json
sed -i -e "s%<runOutputName>%$runOutputName%g" helloImageTemplateMsi.json
```

## <a name="create-the-image"></a>Criar a imagem

Envie a configuração de imagem para o serviço Construtor de Imagens do Azure.

```azurecli-interactive
az resource create \
    --resource-group $imageResourceGroup \
    --properties @helloImageTemplateMsi.json \
    --is-full-object \
    --resource-type Microsoft.VirtualMachineImages/imageTemplates \
    -n helloImageTemplateMsi01
```

Inicie o build da imagem.

```azurecli-interactive
az resource invoke-action \
     --resource-group $imageResourceGroup \
     --resource-type  Microsoft.VirtualMachineImages/imageTemplates \
     -n helloImageTemplateMsi01 \
     --action Run 
```

Aguarde a conclusão do build. Isso pode levar cerca de 15 minutos.

## <a name="create-a-vm"></a>Criar uma máquina virtual

Crie uma VM a partir da imagem. 

```azurecli
az vm create \
  --resource-group $imageResourceGroup \
  --name aibImgVm00 \
  --admin-username aibuser \
  --image $imageName \
  --location $location \
  --generate-ssh-keys
```

Depois que a VM tiver sido criada, inicie uma sessão SSH com a VM.

```console
ssh aibuser@<publicIp>
```

Você deve ver que a imagem foi personalizada com uma Mensagem do Dia assim que a conexão SSH é estabelecida.

```output

*******************************************************
**            This VM was built from the:            **
**      !! AZURE VM IMAGE BUILDER Custom Image !!    **
**         You have just been Customized :-)         **
*******************************************************
```

## <a name="clean-up"></a>Limpeza

Quando tiver terminado, você poderá excluir os recursos se eles não forem mais necessários.

```azurecli-interactive

az role definition delete --name "$imageRoleDefName"
```azurecli-interactive
az role assignment delete \
    --assignee $imgBuilderCliId \
    --role "$imageRoleDefName" \
    --scope /subscriptions/$subscriptionID/resourceGroups/$imageResourceGroup
az identity delete --ids $imgBuilderId
az resource delete \
    --resource-group $imageResourceGroup \
    --resource-type Microsoft.VirtualMachineImages/imageTemplates \
    -n helloImageTemplateMsi01
az group delete -n $imageResourceGroup
az group delete -n $strResourceGroup
```

## <a name="next-steps"></a>Próximas etapas

Se você tiver problemas para trabalhar com o construtor de imagem do Azure, consulte [solução de problemas](image-builder-troubleshoot.md?toc=%2fazure%2fvirtual-machines%context%2ftoc.json).
