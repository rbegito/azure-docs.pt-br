---
title: Conceitos-identidade e acesso
description: Saiba mais sobre os conceitos de identidade e acesso da solução do Azure VMware
ms.topic: conceptual
ms.date: 11/11/2020
ms.openlocfilehash: e9c0d62968d94e2b018186f67072b6ae7078db02
ms.sourcegitcommit: 6ab718e1be2767db2605eeebe974ee9e2c07022b
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94536091"
---
# <a name="azure-vmware-solution-identity-concepts"></a>Conceitos de identidade da solução VMware do Azure

As nuvens privadas da solução Azure VMware são provisionadas com um servidor vCenter e com o Gerenciador de NSX-T. Use o vCenter para gerenciar cargas de trabalho de máquina virtual (VM). Você usa o Gerenciador de NSX-T para estender a nuvem privada.

Gerenciamento de acesso e identidade use privilégios de grupo CloudAdmin para o vCenter e direitos de administrador restritos para o Gerenciador de NSX-T. Ele garante que sua plataforma de nuvem privada seja automaticamente atualizada com os recursos e patches mais recentes.  Para obter mais informações, consulte o [artigo conceitos de atualizações de nuvem privada][concepts-upgrades].

## <a name="vcenter-access-and-identity"></a>acesso e identidade do vCenter

O grupo CloudAdmin fornece os privilégios no vCenter. Você gerencia o grupo localmente no vCenter. Outra opção é por meio da integração do logon único LDAP do vCenter com o Azure Active Directory. Você habilita essa integração depois de implantar sua nuvem privada. 

A tabela mostra os privilégios **CloudAdmin** e **CloudGlobalAdmin** .

|  Conjunto de privilégios           | CloudAdmin | CloudGlobalAdmin | Comentário |
| :---                     |    :---:   |       :---:      |   :--:  |
|  Alarmes                  | Um usuário CloudAdmin tem todos os privilégios de alarmes no Compute-ResourcePool e nas VMs.     |          --        |  -- |
|  Implantação automática             |  --  |        --        |  A Microsoft faz o gerenciamento de hosts.  |
|  Certificados            |  --  |        --       |  A Microsoft faz o gerenciamento de certificados.  |
|  Biblioteca de conteúdo         | Um usuário CloudAdmin tem privilégios para criar e usar arquivos em uma biblioteca de conteúdo.    |         Habilitado com SSO.         |  A Microsoft distribuirá arquivos na biblioteca de conteúdo para hosts ESXi.  |
|  Datacenter              |  --  |        --          |  A Microsoft faz todas as operações de data center.  |
|  Repositório de dados               | Datastore. AllocateSpace, datastore. procurar, Datastore.Config, datastore. Excluirfile, datastore. filemanagement, datastore. UpdateVirtualMachineMetadata     |    --    |   -- |
|  Gerenciador de Agentes ESX       |  --  |         --       |  A Microsoft faz todas as operações.  |
|  Pasta                  |  Um usuário CloudAdmin tem todos os privilégios de pasta.     |  --  |  --  |
|  Global                  |  Global. CancelTask, global. GlobalTag, global. Health, global. LogEvent, global. ManageCustomFields, global. promanagers, global. autocustomfield, Global.SystemTag         |                  |    |
|  Host                    |  Host. HBR. HbrManagement      |        --          |  A Microsoft faz todas as outras operações de host.  |
|  InventoryService        |  InventoryService. marcação      |        --          |  --  |
|  Rede                 |  Network.Assign    |                  |  A Microsoft faz todas as outras operações de rede.  |
|  Permissões             |  --  |        --       |  A Microsoft faz todas as operações de permissões.  |
|  Armazenamento controlado por perfil  |  --  |        --       |  A Microsoft faz todas as operações de perfil.  |
|  Recurso                |  Um usuário CloudAdmin tem todos os privilégios de recurso.        |      --       | --   |
|  Tarefa agendada          |  Um usuário CloudAdmin tem todos os privilégios de ScheduleTask.   |   --   | -- |
|  Sessões                |  Sessions. GlobalMessage, Sessions. ValidateSession      |   --   |  A Microsoft realiza todas as outras operações de sessão.  |
|  Exibições de armazenamento           |  StorageViews. exibição   |        --          |  A Microsoft faz todas as outras operações de exibição de armazenamento (configure service).  |
|  Tarefas                   |  --  |  --   |  A Microsoft gerencia extensões que gerenciam tarefas.  |
|  vApp                    |  Um usuário CloudAdmin tem todos os privilégios de vApp.  |  --  |  --  |
|  Máquina Virtual         |  Um usuário CloudAdmin tem todos os privilégios de VirtualMachine.  |  --  |  --  |
|  vService                |  Um usuário CloudAdmin tem todos os privilégios de vService.  |  --  |  --  |

## <a name="nsx-t-manager-access-and-identity"></a>Acesso e identidade do NSX-T Manager

Use a conta "administrador" para acessar o Gerenciador de NSX-T. Ele tem privilégios totais e permite criar e gerenciar roteadores T1, comutadores lógicos e todos os serviços. Os privilégios dão acesso ao roteador NSX-T T0. Uma alteração no roteador de T0 pode resultar em desempenho de rede degradado ou sem acesso à nuvem privada. Abra uma solicitação de suporte no portal do Azure para solicitar qualquer alteração ao seu roteador do NSX-T T0.
  
## <a name="next-steps"></a>Próximas etapas

A próxima etapa é aprender sobre os [conceitos de atualização de nuvem privada][concepts-upgrades].

<!-- LINKS - external -->

<!-- LINKS - internal -->
[concepts-upgrades]: ./concepts-upgrades.md