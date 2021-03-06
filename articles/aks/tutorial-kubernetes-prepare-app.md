---
title: Tutorial do Kubernetes no Azure - Preparar um aplicativo
description: Neste tutorial do AKS (Serviço de Kubernetes do Azure), você aprenderá como preparar e criar um aplicativo de vários contêineres com o Docker Compose que poderá ser implantado no AKS.
services: container-service
ms.topic: tutorial
ms.date: 09/30/2020
ms.custom: mvc
ms.openlocfilehash: 15bf29c676c4ca41fc2d005f3500a89ed6b9c380
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91576329"
---
# <a name="tutorial-prepare-an-application-for-azure-kubernetes-service-aks"></a>Tutorial: preparar um aplicativo para o AKS (Serviço de Kubernetes do Azure)

Neste tutorial, parte um de sete, um aplicativo de vários contêineres é preparado para uso em Kubernetes. As ferramentas de desenvolvimento existentes, como o Docker Compose, são usadas para compilar e testar aplicativos localmente. Você aprenderá como:

> [!div class="checklist"]
> * Clonar a origem do aplicativo de exemplo do GitHub
> * Criar uma imagem de contêiner a partir da origem do aplicativo de exemplo
> * Testar o aplicativo de vários contêineres em um ambiente Docker local

Uma vez concluído, o seguinte aplicativo será executado em seu ambiente de desenvolvimento local:

![Imagem do cluster Kubernetes no Azure](./media/container-service-tutorial-kubernetes-prepare-app/azure-vote.png)

Em tutoriais adicionais, a imagem de contêiner será carregada para um Registro de Contêiner do Azure e, em seguida, implantada em um cluster AKS.

## <a name="before-you-begin"></a>Antes de começar

Este tutorial assume uma compreensão básica dos conceitos fundamentais do Docker como contêineres, imagens de contêiner e comandos do `docker`. Consulte [Introdução ao Docker][docker-get-started] para conhecer os conceitos básicos de contêiner.

Para concluir este tutorial, você precisa de um ambiente de desenvolvimento local do Docker que executa contêineres do Linux. O Docker fornece pacotes que o configuram facilmente em qualquer sistema [Mac][docker-for-mac], [Windows][docker-for-windows] ou [Linux][docker-for-linux].

O Azure Cloud Shell não inclui os componentes do Docker necessários para concluir as etapas destes tutoriais. Portanto, é recomendável usar um ambiente de desenvolvimento completo do Docker.

## <a name="get-application-code"></a>Obter o código de aplicativo

O aplicativo de exemplo usado neste tutorial é um aplicativo de votação básico. O aplicativo consiste de um componente Web de front-end e uma instância Redis de back-end. O componente Web é empacotado em uma imagem de contêiner personalizada. A instância Redis utiliza uma imagem não modificada do Hub Docker.

Use o [git][] para clonar o aplicativo de exemplo para seu ambiente de desenvolvimento:

```console
git clone https://github.com/Azure-Samples/azure-voting-app-redis.git
```

Altere para o diretório clonado.

```console
cd azure-voting-app-redis
```

Dentro do diretório está o código-fonte do aplicativo, um arquivo Docker Compose pré-criado e um arquivo de manifesto Kubernetes. Esses arquivos são usados em todo o conjunto do tutorial.

## <a name="create-container-images"></a>Criar imagens de contêiner

O [Docker Compose][docker-compose] pode ser usado para automatizar a compilação de imagens de contêiner e a implantação de aplicativos de vários contêineres.

Execute o arquivo `docker-compose.yaml` de exemplo para criar a imagem de contêiner, baixar a imagem Redis e iniciar o aplicativo:

```console
docker-compose up -d
```

Quando completado, use o comando [docker images][docker-images] para ver as imagens criadas. Três imagens foram baixadas ou criadas. A imagem *azure-vote-front* contém o aplicativo de front-end e usa a imagem *nginx-flask* como base. A imagem *redis* é usada para iniciar uma instância do Redis.

```
$ docker images

REPOSITORY                                     TAG                 IMAGE ID            CREATED             SIZE
mcr.microsoft.com/azuredocs/azure-vote-front   v1                  84b41c268ad9        9 seconds ago       944MB
mcr.microsoft.com/oss/bitnami/redis            6.0.8               3a54a920bb6c        2 days ago          103MB
tiangolo/uwsgi-nginx-flask                     python3.6           a16ce562e863        6 weeks ago         944MB
```

Execute o comando [docker ps][docker-ps] para ver os contêineres em execução:

```
$ docker ps

CONTAINER ID        IMAGE                                             COMMAND                  CREATED             STATUS              PORTS                           NAMES
d10e5244f237        mcr.microsoft.com/azuredocs/azure-vote-front:v1   "/entrypoint.sh /sta…"   3 minutes ago       Up 3 minutes        443/tcp, 0.0.0.0:8080->80/tcp   azure-vote-front
21574cb38c1f        mcr.microsoft.com/oss/bitnami/redis:6.0.8         "/opt/bitnami/script…"   3 minutes ago       Up 3 minutes        0.0.0.0:6379->6379/tcp          azure-vote-back
```

## <a name="test-application-locally"></a>Testar o aplicativo localmente

Para ver o aplicativo em execução, insira `http://localhost:8080` em um navegador da Web local. O aplicativo de exemplo é carregado, conforme mostra o exemplo a seguir:

![Imagem do cluster Kubernetes no Azure](./media/container-service-tutorial-kubernetes-prepare-app/azure-vote.png)

## <a name="clean-up-resources"></a>Limpar os recursos

Agora que a funcionalidade do aplicativo foi validada, os contêineres em execução podem ser interrompidos e removidos. Não exclua as imagens do contêiner. No próximo tutorial, a imagem *azure-vote-front* será carregada em uma instância do Registro de Contêiner do Azure.

Pare e remova as instâncias de contêiner e os recursos com o comando [docker-compose down][docker-compose-down]:

```console
docker-compose down
```

Quando o aplicativo local tiver sido removido, você terá uma imagem do Docker que contém o aplicativo Azure Vote, *azure-vote-front*, para usar no próximo tutorial.

## <a name="next-steps"></a>Próximas etapas

Neste tutorial, um aplicativo foi testado e imagens de contêiner foram criadas para o aplicativo. Você aprendeu a:

> [!div class="checklist"]
> * Clonar a origem do aplicativo de exemplo do GitHub
> * Criar uma imagem de contêiner a partir da origem do aplicativo de exemplo
> * Testar o aplicativo de vários contêineres em um ambiente Docker local

Avance para o próximo tutorial para saber como armazenar imagens de contêiner em um Registro de Contêiner do Azure.

> [!div class="nextstepaction"]
> [Enviar imagens por push ao Registro de Contêiner do Azure][aks-tutorial-prepare-acr]

<!-- LINKS - external -->
[docker-compose]: https://docs.docker.com/compose/
[docker-for-linux]: https://docs.docker.com/engine/installation/#supported-platforms
[docker-for-mac]: https://docs.docker.com/docker-for-mac/
[docker-for-windows]: https://docs.docker.com/docker-for-windows/
[docker-get-started]: https://docs.docker.com/get-started/
[docker-images]: https://docs.docker.com/engine/reference/commandline/images/
[docker-ps]: https://docs.docker.com/engine/reference/commandline/ps/
[docker-compose-down]: https://docs.docker.com/compose/reference/down
[git]: https://git-scm.com/downloads

<!-- LINKS - internal -->
[aks-tutorial-prepare-acr]: ./tutorial-kubernetes-prepare-acr.md
