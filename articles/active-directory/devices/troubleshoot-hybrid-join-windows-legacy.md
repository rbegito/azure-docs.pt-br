---
title: Solucionar problemas de dispositivos ingressados Azure Active Directory híbridos herdados
description: Solução de problemas do Azure Active Directory híbrido ingressado em dispositivos de nível inferior.
services: active-directory
ms.service: active-directory
ms.subservice: devices
ms.topic: troubleshooting
ms.date: 11/21/2019
ms.author: joflore
author: MicrosoftGuyJFlo
manager: daveba
ms.reviewer: jairoc
ms.collection: M365-identity-device-management
ms.openlocfilehash: 2a4e8ec75d6610e19f241d2047518c3a43132a6e
ms.sourcegitcommit: 3bdeb546890a740384a8ef383cf915e84bd7e91e
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93079012"
---
# <a name="troubleshooting-hybrid-azure-active-directory-joined-down-level-devices"></a>Solução de problemas do Azure Active Directory híbrido ingressado em dispositivos de nível inferior 

Este aplicativo é aplicável apenas aos seguintes dispositivos: 

- Windows 7 
- Windows 8.1 
- Windows Server 2008 R2 
- Windows Server 2012 
- Windows Server 2012 R2 

Para Windows 10 ou Windows Server 2016, confira [Solução de problemas do Azure Active Directory híbrido ingressado em dispositivos do Windows 10 e do Windows Server 2016](troubleshoot-hybrid-join-windows-current.md).

Este artigo pressupõe que você tenha [dispositivos configurados e ingressados no Azure Active Directory híbrido](hybrid-azuread-join-plan.md) para dar suporte aos seguintes cenários:

- Acesso Condicional baseado no dispositivo

Este artigo fornece orientação para solução de possíveis problemas.  

**O que você deve saber:** 

- O ingresso de dispositivos Windows de versões anteriores no Azure AD híbrido funciona de forma um pouco diferente do que no Windows 10. Muitos clientes não percebem que precisam do AD FS (para domínios federados) ou do SSO contínuo configurado (para domínios gerenciados).
- Para clientes com domínios federados, se o SCP (Ponto de Conexão do Serviço) tiver sido configurado de modo a apontar para o nome de domínio gerenciado (por exemplo, contoso.onmicrosoft.com, em vez de contoso.com), o Ingresso no Azure AD Híbrido de dispositivos Windows de versões anteriores não funcionará.
- O mesmo dispositivo físico é exibido várias vezes no Azure AD quando vários usuários de domínio entram nos dispositivos de versões anteriores ingressados no Azure AD híbrido.  Por exemplo, se *jdoe* e *jharnett* entrarem no dispositivo, um registro separado (DeviceID) será criado para cada um desses usuários na guia de informações do **USUÁRIO** . 
- Você também pode obter várias entradas para um dispositivo na guia Informações do usuário devido a uma reinstalação do sistema operacional ou a um novo registro manual.
- O registro inicial / junção de dispositivos é configurado para realizar uma tentativa de login ou bloqueio / desbloqueio. Pode haver um atraso de cinco minutos disparado por uma tarefa do agendador de tarefas. 
- Certifique-se de que o [KB4284842](https://support.microsoft.com/help/4284842) está instalado, no caso do Windows 7 SP1 ou Windows Server 2008 R2 SP1. Essa atualização evita futuras falhas de autenticação devido à perda de acesso do cliente a chaves protegidas após a alteração da senha.

## <a name="step-1-retrieve-the-registration-status"></a>Etapa 1: Recuperar o status do registro 

**ara verificar o status do registro:**  

1. Logon com a conta de usuário que executou uma associação do Microsoft Azure Active Directory híbrido.
1. Abra o prompt de comando 
1. Digite `"%programFiles%\Microsoft Workplace Join\autoworkplace.exe" /i`

Esse comando exibe uma caixa de diálogo que fornece detalhes sobre o status do ingresso.

:::image type="content" source="./media/troubleshoot-hybrid-join-windows-legacy/01.png" alt-text="Captura de tela da caixa de diálogo Workplace Join para Windows. O texto que inclui um endereço de email informa que um determinado dispositivo ingressou em um local de trabalho." border="false":::

## <a name="step-2-evaluate-the-hybrid-azure-ad-join-status"></a>Etapa 2: Avaliar o status do ingresso do Azure AD híbrido 

Se o dispositivo não tiver ingressado no Azure AD híbrido, você poderá tentar fazer o ingresso híbrido do Azure AD clicando no botão "Ingressar". Se a tentativa de fazer o ingresso no Azure AD híbrido falhar, os detalhes sobre a falha serão mostrados.

**As tarefas mais comuns são:**

- Um AD FS configurado incorretamente ou problemas de rede ou no Azure AD

    :::image type="content" source="./media/troubleshoot-hybrid-join-windows-legacy/02.png" alt-text="Captura de tela da caixa de diálogo Workplace Join para Windows. O texto que inclui um endereço de email informa que um determinado dispositivo ingressou em um local de trabalho." border="false":::
    
   - O Autoworkplace.exe não pode autenticar silenciosamente com o Microsoft Azure Active Directory ou o AD FS. Isso pode ser causado pela ausência de AD FS ou sua configuração incorreta (para domínios federados), ou ausência do Logon único contínuo do Azure AD ou sua configuração incorreta (para domínios gerenciados) ou problemas de rede. 
   - Pode ser que a MFA (autenticação multifator) esteja habilitada/configurada para o usuário e o WIAORMULTIAUTHN não esteja configurado no servidor de AD FS. 
   - Outra possibilidade é que a página de descoberta de domínio doméstico (HRD) esteja aguardando a interação do usuário, o que evita que o **autoworkplace.exe** solicite silenciosamente um token.
   - É possível que o AD FS e as URLs do Azure AD estejam ausentes na zona de intranet do IE no cliente.
   - Problemas de conectividade de rede podem estar impedindo o **autoworkplace.exe** de alcançar o AD FS ou as URLs do Azure AD. 
   - **Autoworkplace.exe** exige que o cliente tenha uma linha de visão direta do cliente para o controlador de domínio do AD local da organização, o que significa que a junção híbrida do Azure AD é realizada com sucesso somente quando o cliente está conectado à intranet da organização.
   - Sua organização usa o Logon Único Contínuo do Microsoft Azure Active Directory `https://autologon.microsoftazuread-sso.com` ou `https://aadg.windows.net.nsatc.net` não está presente nas configurações de intranet do Internet Explorer do dispositivo, e **Permitir atualizações à barra de status por meio de script** não está habilitada para a zona da Intranet.
- Você não está conectado como um usuário de domínio

   :::image type="content" source="./media/troubleshoot-hybrid-join-windows-legacy/03.png" alt-text="Captura de tela da caixa de diálogo Workplace Join para Windows. O texto que inclui um endereço de email informa que um determinado dispositivo ingressou em um local de trabalho." border="false":::

   Há algumas razões diferentes pelas quais isso pode ocorrer:

   - O usuário conectado não é um usuário de domínio (por exemplo, um usuário local). O ingresso no Azure AD Híbrido em dispositivos de nível inferior tem suporte apenas para usuários do domínio.
   - O cliente não é capaz de se conectar a um controlador de domínio.    
- Você atingiu uma cota

    :::image type="content" source="./media/troubleshoot-hybrid-join-windows-legacy/04.png" alt-text="Captura de tela da caixa de diálogo Workplace Join para Windows. O texto que inclui um endereço de email informa que um determinado dispositivo ingressou em um local de trabalho." border="false":::

- O serviço não está respondendo 

    :::image type="content" source="./media/troubleshoot-hybrid-join-windows-legacy/05.png" alt-text="Captura de tela da caixa de diálogo Workplace Join para Windows. O texto que inclui um endereço de email informa que um determinado dispositivo ingressou em um local de trabalho." border="false":::

Também é possível encontrar informações de status no log de eventos em: **Log de Aplicativos e Serviços\Microsoft-Workplace Join**
  
**As causas mais comuns para a falha do ingresso do Azure AD híbrido são:** 

- O computador não está conectado à rede interna da organização nem a uma VPN com conexão ao controlador de domínio do AD local.
- Você está conectado ao computador com uma conta de computador local. 
- Problemas de configuração do serviço: 
   - O servidor do AD FS não foi configurado para compatibilidade com **WIAORMULTIAUTHN** . 
   - A floresta do seu computador não tem objeto de Ponto de Conexão de Serviço que aponta o seu nome de domínio verificado no Microsoft Azure Active Directory 
   - Ou, caso seu domínio seja gerenciado, o SSO contínuo não foi configurado ou não está funcionando.
   - Um usuário atingiu o limite de dispositivos. 

## <a name="next-steps"></a>Próximas etapas

Para perguntas, consulte as [Perguntas frequentes sobre o gerenciamento de dispositivos](faq.md)  
