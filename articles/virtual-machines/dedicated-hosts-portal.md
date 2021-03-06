---
title: Implantar hosts dedicados do Azure usando o portal do Azure
description: Implante VMs e conjuntos de dimensionamento para hosts dedicados usando o portal do Azure.
author: cynthn
ms.service: virtual-machines
ms.topic: how-to
ms.workload: infrastructure
ms.date: 09/04/2020
ms.author: cynthn
ms.openlocfilehash: a6bef4944207e26f2de93daa89fa1418c5c44c4f
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91373066"
---
# <a name="deploy-vms-and-scale-sets-to-dedicated-hosts-using-the-portal"></a>Implantar VMs e conjuntos de dimensionamento para hosts dedicados usando o portal 

Este artigo orienta como criar um [host dedicado](dedicated-hosts.md) do Azure para hospedar suas máquinas virtuais (VMs). 


> [!IMPORTANT]
> Este artigo também aborda o posicionamento automático de VMs e instâncias do conjunto de dimensionamento. O posicionamento automático está atualmente em visualização pública.
> Para participar da versão prévia, conclua a pesquisa de integração de visualização em [https://aka.ms/vmss-adh-preview](https://aka.ms/vmss-adh-preview) .
> Para acessar o recurso de visualização no portal do Azure, você deve usar esta URL: [https://aka.ms/vmssadh](https://aka.ms/vmssadh) .
> Essa versão prévia é fornecida sem um contrato de nível de serviço e não é recomendada para cargas de trabalho de produção. Alguns recursos podem não ter suporte ou podem ter restrição de recursos. Para obter mais informações, consulte [Termos de Uso Complementares de Versões Prévias do Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="limitations"></a>Limitações

- Os tamanhos e tipos de hardware disponíveis para hosts dedicados variam de acordo com a região. Consulte a [página de preços](https://aka.ms/ADHPricing) do host para saber mais.

## <a name="create-a-host-group"></a>Criar um grupo de hosts

Um **grupo de hosts** é um recurso que representa uma coleção de hosts dedicados. Você cria um grupo de hosts em uma região e uma zona de disponibilidade e adiciona hosts a ele. Ao planejar a alta disponibilidade, há opções adicionais. Você pode usar uma ou ambas as opções a seguir com hosts dedicados: 
- Alcance de várias zonas de disponibilidade. Nesse caso, é necessário ter um grupo de hosts em cada uma das zonas que você quer usar.
- Alcance de vários domínios de falha que são mapeados para racks físicos. 
 
Em ambos os casos, você precisa fornecer a contagem de domínios de falha ao seu grupo de hosts. Se você não quiser abranger domínios de falha no seu grupo, use uma contagem de domínio de falha de 1. 

Você também pode optar por usar tanto zonas de disponibilidade quanto domínios de falha. 

Neste exemplo, criaremos um grupo de hosts usando 1 zona de disponibilidade e 2 domínios de falha. 


1. Abra o [portal](https://portal.azure.com)do Azure. Se você quiser experimentar a visualização para o **posicionamento automático**, use esta URL: [https://aka.ms/vmssadh](https://aka.ms/vmssadh) .
1. Selecione **criar um recurso** no canto superior esquerdo.
1. Procure por **grupo de hosts** e, em seguida, selecione **grupos de hosts** nos resultados.
1. Na página **grupos de hosts** , selecione **criar**.
1. Selecione a assinatura que você deseja usar e, em seguida, selecione **criar nova** para criar um novo grupo de recursos.
1. Digite *myDedicatedHostsRG* como o **nome** e, em seguida, selecione **OK**.
1. Para **nome do grupo de hosts**, digite *myhost*Group.
1. Em **Localização**, selecione **Leste dos EUA**.
1. Para **zona de disponibilidade**, selecione **1**.
1. Para **contagem de domínios de falha**, selecione **2**.
1. Se você usou a URL de **posicionamento automático** , selecione esta opção para atribuir automaticamente as VMs e as instâncias do conjunto de dimensionamento a um host disponível nesse grupo.
1. Selecione **revisar + criar** e aguarde a validação.
1. Depois de ver a mensagem **validação aprovada** , selecione **criar** para criar o grupo de hosts.

Só deve levar alguns minutos para criar o grupo de hosts.


## <a name="create-a-dedicated-host"></a>Criar um host dedicado

Agora, crie um host dedicado no grupo de hosts. Além de um nome para o host, você deve fornecer o SKU para o host. O SKU do host captura a série de VMs com suporte, bem como a geração de hardware para o host dedicado.

Para mais informações sobre preços e SKUs do host, consulte [Preços do Host Dedicado do Azure](https://aka.ms/ADHPricing).

Ao definir uma contagem de domínios de falha para seu grupo de hosts, você será solicitado a especificar o domínio de falha para o host.  

1. Selecione **criar um recurso** no canto superior esquerdo.
1. Pesquise **host dedicado** e, em seguida, selecione **hosts dedicados** nos resultados.
1. Na página **hosts dedicados** , selecione **criar**.
1. Selecione a assinatura que você deseja usar.
1. Selecione *myDedicatedHostsRG* como o **grupo de recursos**.
1. Em **detalhes da instância**, digite *myhost* para o **nome** e selecione *leste dos EUA* para o local.
1. Em **perfil de hardware**, selecione *família de Es3 padrão-tipo 1* para a **família de tamanho**, selecione *myhost* Group para o **grupo de hosts** e, em seguida, selecione *1* para o **domínio de falha**. Deixe os padrões para o restante dos campos.
1. Quando terminar, selecione **revisar + criar** e aguarde a validação.
1. Depois de ver a mensagem **validação aprovada** , selecione **criar** para criar o host.

## <a name="create-a-vm"></a>Criar uma máquina virtual

1. Escolha **Criar um recurso** no canto superior esquerdo do portal do Azure.
1. Na caixa de pesquisa acima da lista de recursos do Azure Marketplace, procure e selecione a imagem que deseja usar e, em seguida, escolha **criar**.
1. Na guia **noções básicas** , em **detalhes do projeto**, verifique se a assinatura correta está selecionada e, em seguida, selecione *MyDedicatedHostsRG* como o **grupo de recursos**. 
1. Em **Detalhes da instância**, digite *myVM* para o **Nome da máquina virtual** e escolha *Leste dos EUA* para **Localização**.
1. Em **Opções de disponibilidade** selecionar **zona de disponibilidade**, selecione *1* na lista suspensa.
1. Para o tamanho, selecione **alterar tamanho**. Na lista de tamanhos disponíveis, escolha um da série Esv3, como **Standard E2 v3**. Talvez seja necessário limpar o filtro para ver todos os tamanhos disponíveis.
1. Conclua o restante dos campos na guia **noções básicas** , conforme necessário.
1. Na parte superior da página, selecione a guia **avançado** e, na seção **host** , selecione *myhost* Group para o **grupo de hosts** e *myhost* para o **host**. 
    ![Selecionar grupo de hosts e host](./media/dedicated-hosts-portal/advanced.png)
1. Deixe os padrões restantes e, em seguida, selecione o botão **Examinar + criar** na parte inferior da página.
1. Quando você vir a mensagem a validação foi aprovada, selecione **criar**.

Levará alguns minutos para que sua VM seja implantada.

## <a name="create-a-scale-set-preview"></a>Criar um conjunto de dimensionamento (visualização)

> [!IMPORTANT]
> Os conjuntos de dimensionamento de máquinas virtuais em hosts dedicados estão atualmente em visualização pública.
>
> Para participar da versão prévia, conclua a pesquisa de integração de visualização em [https://aka.ms/vmss-adh-preview](https://aka.ms/vmss-adh-preview) .
>
> Para acessar o recurso de visualização no portal do Azure, você deve usar esta URL: [https://aka.ms/vmssadh](https://aka.ms/vmssadh) .
>
> Essa versão prévia é fornecida sem um contrato de nível de serviço e não é recomendada para cargas de trabalho de produção. Alguns recursos podem não ter suporte ou podem ter restrição de recursos. 
>
> Para obter mais informações, consulte [Termos de Uso Complementares de Versões Prévias do Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Ao implantar um conjunto de dimensionamento, você especifica o grupo de hosts.

1. Procure por *conjunto de dimensionamento* e selecione **conjuntos de dimensionamento de máquinas virtuais** na lista.
1. Selecione **Adicionar** para criar um novo conjunto de dimensionamento.
1. Preencha os campos na guia **noções básicas** como faria normalmente, mas certifique-se de selecionar um tamanho de VM que seja da série escolhida para o host dedicado, como o **Standard E2 v3**.
1. Na guia **avançado** , para **algoritmo de difusão** , selecione **distribuição máxima**.
1. Em **grupo de hosts**, selecione o grupo de hosts na lista suspensa. Se você criou recentemente o grupo, pode levar um minuto para ser adicionado à lista.

## <a name="add-an-existing-vm"></a>Adicionar uma VM existente 

Você pode adicionar uma VM existente a um host dedicado, mas a VM deve primeiro ser Stop\Deallocated. Antes de mover uma VM para um host dedicado, verifique se a configuração da VM tem suporte:

- O tamanho da VM deve estar na mesma família de tamanho que o host dedicado. Por exemplo, se o host dedicado for DSv3, o tamanho da VM poderá ser Standard_D4s_v3, mas não poderá ser um Standard_A4_v2. 
- A VM precisa estar localizada na mesma região que o host dedicado.
- A VM não pode fazer parte de um grupo de posicionamento de proximidade. Remova a VM do grupo de posicionamento de proximidade antes de movê-la para um host dedicado. Para obter mais informações, consulte [mover uma VM para fora de um grupo de posicionamento de proximidade](./windows/proximity-placement-groups.md#move-an-existing-vm-out-of-a-proximity-placement-group)
- A VM não pode estar em um conjunto de disponibilidade.
- Se a VM estiver em uma zona de disponibilidade, ela deverá ser a mesma zona de disponibilidade que o grupo de hosts. As configurações de zona de disponibilidade para a VM e o grupo de hosts devem corresponder.

Mova a VM para um host dedicado usando o [portal](https://portal.azure.com).

1. Abra a página da VM.
1. Selecione **parar** para STOP\DEALLOCATE a VM.
1. Selecione **configuração** no menu à esquerda.
1. Selecione um grupo de hosts e um host nos menus suspensos.
1. Quando terminar, selecione **salvar** na parte superior da página.
1. Depois que a VM tiver sido adicionada ao host, selecione **visão geral** no menu à esquerda.
1. Na parte superior da página, selecione **Iniciar** para reiniciar a VM.

## <a name="next-steps"></a>Próximas etapas

- Para mais informações, consulte a visão geral de [hosts dedicados](dedicated-hosts.md).

- Há um exemplo de modelo, [aqui](https://github.com/Azure/azure-quickstart-templates/blob/master/201-vm-dedicated-hosts/README.md), que usa zonas e domínios de falha para obter resiliência máxima em uma região.

- Você também pode implantar um host dedicado usando o [CLI do Azure](./linux/dedicated-hosts-cli.md) ou o [PowerShell](./windows/dedicated-hosts-powershell.md).
