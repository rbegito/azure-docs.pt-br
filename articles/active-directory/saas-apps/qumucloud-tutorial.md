---
title: 'Tutorial: Integração do Azure Active Directory com o Qumu Cloud | Microsoft Docs'
description: Saiba como configurar o logon único entre o Azure Active Directory e o Qumu Cloud.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 04/03/2019
ms.author: jeedes
ms.openlocfilehash: 3547de9ce307686e8796b1b3ad0f4fdfff18c9c2
ms.sourcegitcommit: 59f506857abb1ed3328fda34d37800b55159c91d
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 10/24/2020
ms.locfileid: "92511855"
---
# <a name="tutorial-azure-active-directory-integration-with-qumu-cloud"></a>Tutorial: Integração do Azure Active Directory com o Qumu Cloud

Neste tutorial, você aprenderá a integrar o Qumu Cloud ao Microsoft Azure AD (Azure Active Directory).
A integração do Qumu Cloud ao Microsoft Azure AD oferece os seguintes benefícios:

* No Microsoft Azure AD, é possível controlar quem tem acesso ao Qumu Cloud.
* Você pode permitir que os usuários sejam conectados automaticamente ao Qumu Cloud (logon único) com suas contas do Azure AD.
* Você pode gerenciar suas contas em um único local central – o portal do Azure.

Para conhecer mais detalhadamente a integração de aplicativos de SaaS ao AD do Azure, consulte [O que é o acesso a aplicativos e logon único com o Active Directory do Azure](../manage-apps/what-is-single-sign-on.md).
Se você não tiver uma assinatura do Azure, [crie uma conta gratuita](https://azure.microsoft.com/free/) antes de começar.

## <a name="prerequisites"></a>Prerequisites

Para configurar a integração do Microsoft Azure AD com o Qumu Cloud, você precisa dos seguintes itens:

* Uma assinatura do Azure AD. Se não tiver um ambiente do Azure AD, poderá obter uma [conta gratuita](https://azure.microsoft.com/free/)
* Assinatura habilitada para logon único do Qumu Cloud

## <a name="scenario-description"></a>Descrição do cenário

Neste tutorial, você configurará e testará o logon único do Azure AD em um ambiente de teste.

* O Qumu Cloud dá suporte ao SSO iniciado por **SP** e **IDP**

* O Qumu Cloud dá suporte ao provisionamento de usuário **just-in-time**

## <a name="adding-qumu-cloud-from-the-gallery"></a>Adição do Qumu Cloud da galeria

Para configurar a integração do Qumu Cloud ao Microsoft Azure AD, você precisará adicionar o Qumu Cloud da galeria à sua lista de aplicativos SaaS gerenciados.

**Para adicionar o Qumu Cloud da galeria, execute as seguintes etapas:**

1. No **[Portal do Azure](https://portal.azure.com)** , no painel navegação à esquerda, clique no ícone **Azure Active Directory** .

    ![O botão Azure Active Directory](common/select-azuread.png)

2. Navegue até **Aplicativos Empresariais** e, em seguida, selecione a opção **Todos os Aplicativos** .

    ![A folha Aplicativos empresariais](common/enterprise-applications.png)

3. Clique no botão **Novo aplicativo** na parte superior da caixa de diálogo para adicionar o novo aplicativo.

    ![O botão Novo aplicativo](common/add-new-app.png)

4. Na caixa de pesquisa, digite **Qumu Cloud** , selecione **Qumu Cloud** no painel de resultados e, depois, clique no botão **Adicionar** para adicionar o aplicativo.

    ![Qumu Cloud na lista de resultados](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configurar e testar logon único do Azure AD

Nesta seção, você configurará e testará o logon único do Azure AD com o Qumu Cloud, com base em um usuário de teste chamado **Brenda Fernandes** .
Para que o logon único funcione, é necessário estabelecer uma relação de vínculo entre um usuário do Azure AD e o usuário relacionado do Qumu Cloud.

Para configurar e testar o logon único do Microsoft Azure AD com o Qumu Cloud, você precisa concluir os seguintes blocos de construção:

1. **[Configurar o logon único do Azure AD](#configure-azure-ad-single-sign-on)** – para habilitar seus usuários a usar esse recurso.
2. **[Configurar o logon único do Qumu Cloud](#configure-qumu-cloud-single-sign-on)** – para definir as configurações de logon único no lado do aplicativo.
3. **[Criar um usuário de teste do Azure AD](#create-an-azure-ad-test-user)** – para testar o logon único do Azure AD com Brenda Fernandes.
4. **[Atribuir o usuário de teste do Azure AD](#assign-the-azure-ad-test-user)** – para permitir que Brenda Fernandes use o logon único do Azure AD.
5. **[Criar um usuário de teste do Qumu Cloud](#create-qumu-cloud-test-user)** – para ter um equivalente de Brenda Fernandes no Qumu Cloud que esteja vinculado à representação de usuário do Azure AD.
6. **[Teste o logon único](#test-single-sign-on)** – para verificar se a configuração funciona.

### <a name="configure-azure-ad-single-sign-on"></a>Configurar o logon único do Azure AD

Nesta seção, você habilitará o logon único do Azure AD no portal do Azure.

Para configurar o logon único do Azure AD com o Qumu Cloud, execute as seguintes etapas:

1. No [portal do Azure](https://portal.azure.com/), na página de integração de aplicativos do **Qumu Cloud** , selecione **Logon único** .

    ![Link Configurar logon único](common/select-sso.png)

2. Na caixa de diálogo **Selecionar um método de logon único** , selecione o modo **SAML/WS-Fed** para habilitar o logon único.

    ![Modo de seleção de logon único](common/select-saml-option.png)

3. Na página **Definir logon único com SAML** , clique no ícone **Editar** para abrir a caixa de diálogo **Configuração básica do SAML** .

    ![Editar a Configuração Básica de SAML](common/edit-urls.png)

4. Na seção **Configuração Básica do SAML** , caso deseje configurar o aplicativo no modo iniciado por **IDP** execute as seguintes etapas:

    ![Captura de tela que mostra a Configuração Básica de SAML, em que você pode inserir o Identificador e a URL de Resposta e selecionar Salvar.](common/idp-intiated.png)

    a. No **identificador** caixa de texto, digite uma URL usando o seguinte padrão: `https://<subdomain>.qumucloud.com/saml/SSO`

    b. No **URL de resposta** caixa de texto, digite uma URL usando o seguinte padrão: `https://<subdomain>.qumucloud.com/saml/SSO`

5. Clique em **Definir URLs adicionais** e execute o passo seguinte se quiser configurar a aplicação no modo **SP** iniciado:

    ![Captura de tela que mostra Definir URLs adicionais, em que você pode inserir uma URL de Logon.](common/metadata-upload-additional-signon.png)

    Na caixa de texto **URL de logon** , digite um URL usando o seguinte padrão: `https://<subdomain>.qumucloud.com`

    > [!NOTE]
    > Esses valores não são reais. Atualize esses valores com o Identificador, a URL de Resposta e a URL de Logon reais. Contate a [equipe de suporte ao cliente do Qumu Cloud](mailto:support@qumu.com) para obter esses valores. Você também pode consultar os padrões exibidos na seção **Configuração Básica de SAML** no portal do Azure.

6. O aplicativo Qumu Cloud espera as declarações SAML em um formato específico, o que exige a adição de mapeamentos de atributo personalizado à configuração de atributos do token SAML. A captura de tela a seguir mostra a lista de atributos padrão. Clique no ícone **Editar** para abrir a caixa de diálogo **Atributos de Usuário** .

    ![Captura de tela que mostra Atributos de usuário com o ícone Editar selecionado.](common/edit-attribute.png)

7. Além do indicado acima, o aplicativo Qumu Cloud espera que mais alguns atributos sejam passados novamente na resposta SAML. Na seção **Declarações de Usuário** da caixa de diálogo **Atributos de Usuário** , execute as seguintes etapas para adicionar o atributo de token SAML, conforme mostrado na tabela abaixo:

    | Nome | Atributo de Origem|
    | ---------------| --------------- |    
    | urn: oid:2.5.4.42 | user.givenname |
    | urn: oid:2.5.4.4 | user.surname |
    | urn:oid:0.9.2342.19200300.100.1.3 | user.mail |
    | urn:oid:0.9.2342.19200300.100.1.1 | user.userprincipalname |

    a. Clique em **Adicionar nova reivindicação** para abrir a caixa de diálogo **Gerenciar declarações de usuários** .

    ![Captura de tela que mostra Declarações de usuário com a opção de Adicionar declaração.](common/new-save-attribute.png)

    ![Captura de tela que mostra a caixa de diálogo Gerenciar declarações do usuário, na qual você pode inserir os valores descritos.](common/new-attribute-details.png)

    b. Na caixa de texto **Nome** , digite o nome do atributo mostrado para essa linha.

    c. Deixe o **Namespace** em branco.

    d. Escolha Origem como **Atributo** .

    e. Na lista **Atributo de origem** , digite o valor do atributo mostrado para essa linha.

    f. Clique em **Save** (Salvar).

8. Na página **Configurar Logon Único com SAML** , na seção **Certificado de Autenticação SAML** , clique em **Baixar** para baixar o **XML de Metadados de Federação** usando as opções fornecidas de acordo com seus requisitos e salve-o no computador.

    ![O link de download do Certificado](common/metadataxml.png)

9. Na seção **Configurar o Qumu Cloud** , copie as URLs apropriadas de acordo com suas necessidades.

    ![Copiar URLs de configuração](common/copy-configuration-urls.png)

    a. URL de logon

    b. Identificador do Azure AD

    c. URL de logoff

### <a name="configure-qumu-cloud-single-sign-on"></a>Configurar o logon único do Qumu Cloud

Para configurar o logon único no lado do **Qumu Cloud** , é necessário enviar o **XML de Metadados de Federação** baixado e as URLs apropriadas copiadas do portal do Azure para a [equipe de suporte do Qumu Cloud](mailto:support@qumu.com). Eles definem essa configuração para ter a conexão de SSO de SAML definida corretamente em ambos os lados.

### <a name="create-an-azure-ad-test-user"></a>Criar um usuário de teste do Azure AD 

O objetivo desta seção é criar um usuário de teste no Portal do Azure chamado Brenda Fernandes.

1. No Portal do Azure, no painel esquerdo, selecione **Azure Active Directory** , selecione **Usuários** e, em seguida, **Todos os usuários** .

    ![Os links “Usuários e grupos” e “Todos os usuários”](common/users.png)

2. Selecione **Novo usuário** na parte superior da tela.

    ![Botão Novo usuário](common/new-user.png)

3. Nas Propriedades do usuário, execute as etapas a seguir.

    ![A caixa de diálogo Usuário](common/user-properties.png)

    a. No campo **Nome** , insira **BrendaFernandes** .
  
    b. No campo **Nome de usuário** , digite `brittasimon@yourcompanydomain.extension`. Por exemplo, BrittaSimon@contoso.com

    c. Marque a caixa de seleção **Mostrar senha** e, em seguida, anote o valor exibido na caixa Senha.

    d. Clique em **Criar** .

### <a name="assign-the-azure-ad-test-user"></a>Atribuir o usuário de teste do Azure AD

Nesta seção, você permitirá que Brenda Fernandes use o logon único do Azure, concedendo a ela acesso ao Qumu Cloud.

1. No portal do Azure, selecione **Aplicativos Empresariais** , **Todos os aplicativos** e, em seguida, **Qumu Cloud** .

    ![Folha de aplicativos empresariais](common/enterprise-applications.png)

2. Na lista de aplicativos, escolha **Qumu Cloud** .

    ![O link do Qumu Cloud na lista de aplicativos](common/all-applications.png)

3. No menu à esquerda, selecione **Usuários e grupos** .

    ![O link “Usuários e grupos”](common/users-groups-blade.png)

4. Escolha o botão **Adicionar usuário** e, em seguida, escolha **Usuários e grupos** na caixa de diálogo **Adicionar Atribuição** .

    ![O painel Adicionar Atribuição](common/add-assign-user.png)

5. Na caixa de diálogo **Usuários e grupos** , escolha **Brenda Fernandes** na lista Usuários e clique no botão **Selecionar** na parte inferior da tela.

6. Se você estiver esperando um valor de função na declaração SAML, na caixa de diálogo **Selecionar função** , escolha a função de usuário apropriada na lista e clique no botão **Selecionar** na parte inferior da tela.

7. Na caixa de diálogo **Adicionar atribuição** , clique no botão **Atribuir** .

### <a name="create-qumu-cloud-test-user"></a>Criar um usuário de teste do Qumu Cloud

Nesta seção, um usuário chamado Brenda Fernandes será criado no Qumu Cloud. O Qumu Cloud dá suporte ao provisionamento de usuário just-in-time, que está habilitado por padrão. Não há itens de ação para você nesta seção. Se um usuário ainda não existir no Qumu Cloud, um novo será criado após a autenticação.

>[!Note]
>Se você precisar criar um usuário manualmente, contate a [equipe de suporte ao Cliente do Qumu Cloud](mailto:support@qumu.com).

### <a name="test-single-sign-on"></a>Testar logon único 

Nesta seção, você testará sua configuração de logon único do Azure AD usando o Painel de Acesso.

Ao clicar no bloco do Qumu Cloud no Painel de Acesso, você deverá ser conectado automaticamente ao Qumu Cloud, para o qual você configurou o SSO. Para saber mais sobre o Painel de Acesso, veja [Introdução ao Painel de Acesso](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Recursos adicionais

- [Lista de tutoriais sobre como integrar aplicativos SaaS com o Active Directory do Azure](./tutorial-list.md)

- [O que é o acesso a aplicativos e logon único com o Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

- [O que é o acesso condicional no Azure Active Directory?](../conditional-access/overview.md)