# 🐋 6 Melhores Práticas para Escrever um Dockerfile 📦

Desde que o Docker foi introduzido em 2013, ele se desenvolveu ao longo de mais de dez anos e agora se tornou o padrão da indústria para tecnologia de contêineres.

Ele é compatível com todos os principais sistemas operacionais e plataformas de nuvem, além de ser capaz de containerizar quase qualquer tipo de aplicação, permitindo que as aplicações sejam facilmente migradas entre diferentes máquinas, clusters ou até mesmo serviços de nuvem.

Todo contêiner Docker é construído a partir de um Dockerfile, portanto, é essencial seguir as melhores práticas ao escrever um Dockerfile.

Vamos explorar algumas dessas práticas a seguir. 

---

## 1. Adicione Arquivos Conforme Necessário

Ao escrever um Dockerfile, o aspecto mais crítico a considerar é o mecanismo de cache.

Toda vez que você constrói uma imagem Docker a partir de um Dockerfile, o Docker salva o cache gerado durante o processo de construção.

Se o cache estiver disponível ao reconstruir a imagem, isso pode acelerar significativamente o processo. Por exemplo, executar o comando `npm install` pode levar alguns minutos para baixar e instalar todas as dependências de um projeto Node.js.

Portanto, você deve aproveitar esse cache ao executar o comando `docker build`, permitindo que a próxima construção utilize rapidamente o cache em vez de esperar vários minutos cada vez, o que é tanto irritante quanto ineficiente.

Se você não se preocupa com o cache, seu Dockerfile pode ser assim:

```dockerfile
FROM node:20
COPY . .
RUN npm install
RUN npm build
```

Este Dockerfile utiliza o comando `COPY` do Docker para adicionar todos os arquivos do projeto (incluindo o código-fonte) à imagem, em seguida executa `npm install` para instalar as dependências e, por fim, executa `npm build` para compilar a aplicação a partir do código-fonte.

Embora isso seja viável, não é eficiente. Suponha que você execute o comando `docker build` e, depois, modifique alguma lógica de negócios nos arquivos do projeto, desejando reconstruir a imagem.

A primeira linha, `FROM node:20`, permanece inalterada, então o Docker usará o cache para esta parte. No entanto, o cache será quebrado na segunda linha, `COPY . .`, porque os arquivos foram alterados.

O Docker utiliza um mecanismo de cache em camadas, onde cada linha no Dockerfile geralmente representa uma camada.

Isso significa que, uma vez que o cache de uma camada seja quebrado, todas as camadas subsequentes não poderão usar o cache durante a construção.

Isso ocorre porque o Docker assume que cada camada subsequente depende de todas as camadas anteriores, o que é uma suposição razoável.

No nosso exemplo, o comando `npm install` será executado sempre que os arquivos do projeto forem alterados. No entanto, ele na verdade não depende do código-fonte do projeto; ele depende apenas dos arquivos `package.json` e `package-lock.json`.

O arquivo `package.json` define todas as dependências que o npm precisa instalar. Então, vamos melhorar o Dockerfile:

```dockerfile
FROM node:20
COPY package*.json .
RUN npm install
COPY . .
RUN npm build
```

Aqui, utilizo `package*.json` para copiar tanto o `package.json` quanto o `package-lock.json`.

Como você pode ver, copio o código-fonte completo da aplicação apenas **depois** de executar o `npm install` e **antes** de executar o `npm build`, já que o `npm build` depende do código-fonte.

Dessa forma, se o código-fonte for alterado, o comando `npm install` ainda poderá ser reutilizado a partir do cache, desde que o `package.json` permaneça inalterado.

Somente se alterarmos alguma dependência no `package.json`, será necessário executar novamente o `npm install`.

### Observação
O Dockerfile do exemplo foi projetado para ilustrar como o mecanismo de cache funciona.

Um Dockerfile real para uma aplicação Node.js seria diferente. Por exemplo, você deve definir o `WORKDIR` antes de adicionar os arquivos e executar comandos como `npm install` ou `npm build`.

---

## 2. Utilizando .dockerignore para Melhorar o Build

Quando você não quer adicionar um arquivo a um repositório Git, você o inclui no arquivo `.gitignore`.

De forma semelhante, quando você não quer incluir um arquivo no contexto de build do Docker, deve adicioná-lo ao arquivo `.dockerignore`.

### O Contexto de Build no Docker

Ao construir uma imagem Docker, você especifica o caminho do contexto de build. Por exemplo:

```bash
docker build -t image_tag .
```

Aqui, o ponto final (`.`) indica que o diretório de trabalho atual será utilizado como o contexto de build.

O contexto de build é então enviado (copiado) para o daemon do Docker, que constrói a imagem.

### Exemplo com Node.js

Suponha que usamos os comandos `npm install` e `npm start` localmente para executar a aplicação antes de construir a imagem Docker.

Como esses comandos são executados diretamente na máquina local, o npm cria um diretório `node_modules` no diretório do projeto para armazenar todas as dependências baixadas.

A estrutura do diretório do projeto pode se parecer com esta:

```plaintext
project/
├── node_modules/
├── package.json
├── package-lock.json
├── public/
├── src/
│   ├── index.js
│   └── ...
```

### Reduzindo o Contexto de Build com `.dockerignore`

Isso ocorre porque o diretório inteiro do projeto (incluindo `node_modules`) foi enviado como contexto de build.

Queremos evitar enviar o diretório `node_modules`, então criamos um arquivo `.dockerignore` e adicionamos `node_modules` a ele.

Agora, ao construir a imagem Docker novamente, o log de build mostra que o tamanho do contexto é muito menor do que antes:

```plaintext
 => [internal] load build context
 => => transferring context: 11.41kB
```

### A Estrutura do Nosso Projeto Agora Fica Assim:

```plaintext
node_modules/
public/
src/
Dockerfile
.dockerignore
package.json
package-lock.json
```

### Lembre-se do Local do `.dockerignore`

O arquivo `.dockerignore` deve sempre ser colocado no diretório raiz do contexto de build.

Você pode se perguntar por que excluímos o diretório `node_modules` do contexto de build.

### Por Que Excluir `node_modules`?

Isso ocorre porque `node_modules` é um diretório criado pelo npm; ele não contém o código-fonte da nossa aplicação.

Esse diretório é gerado localmente pelo npm na nossa máquina. O npm executado dentro do Docker deve criar seu próprio `node_modules` dentro da imagem Docker.

Adicionar o `node_modules` local à nossa imagem Docker não é uma boa prática.

Você deve fornecer apenas o código-fonte da aplicação ao Docker e, em seguida, executar os comandos de build dentro do Docker para construir a aplicação.

Dessa forma, o build do Docker não entrará em conflito com o build local.

---

## 3. Execute Todos os Comandos de Uma Só Vez

Esta prática é direta. Frequentemente, você se verá utilizando o `apt` ou outros gerenciadores de pacotes para instalar os pacotes necessários.

Antes de executar o `apt install`, é necessário executar primeiro o `apt update`.

Em vez de usar múltiplas instruções `RUN` em um Dockerfile, é melhor combiná-las em uma única instrução:

```dockerfile
RUN apt-get update && apt-get install -y \
  git \
  jq \
  kubectl
```

Observe como os nomes dos pacotes são divididos em várias linhas e organizados em ordem alfabética para melhorar a legibilidade.

Se você usar múltiplas instruções `RUN`, cada instrução cria uma nova camada, o que pode desacelerar o processo de build e ocupar mais espaço de armazenamento.

---

## 4. Definindo Variáveis de Ambiente e Versões

Usando a instrução `ENV`, você pode definir variáveis de ambiente durante o processo de build. Essas variáveis serão mantidas na imagem e estarão disponíveis quando o contêiner for executado.

Por exemplo, você pode modificar a variável `PATH` de forma elegante assim:

```dockerfile
ENV PATH=/opt/maven/bin:${PATH}
```

Ou, se você estiver executando uma aplicação Node.js e lendo `process.env.PORT` ao iniciar o servidor, pode definir a porta do servidor no Dockerfile assim:

```dockerfile
ENV PORT=8080
```

É geralmente recomendado configurar sua aplicação utilizando variáveis de ambiente sempre que possível.

Ao implantar a aplicação, alterar uma variável de ambiente é sempre mais fácil do que modificar um arquivo no código e, em seguida, reimplantar a aplicação.

Você também pode usar a instrução `ENV` para definir de forma intuitiva a versão de certas dependências:

```dockerfile
ENV KUBECTL_VERSION=1.27
RUN curl -fsSL https://pkgs.k8s.io/core:/stable:/v${KUBECTL_VERSION}/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
...
```

Quando se trata de versões e Docker, é fortemente recomendado não usar a tag "latest" para imagens.

Da mesma forma, você deve evitar o uso de números de versão muito específicos, como `1.27.4`, pois isso pode impedir que você receba atualizações importantes de patches que corrigem bugs ou melhoram a segurança.

Em vez disso, use os números de versão principais (`x`) ou secundários (`x.y`):

```dockerfile
FROM python:3.10
```

Você também poderia simplesmente escrever `python`, mas, nesse caso, o Docker sempre puxará a versão mais recente, o que pode quebrar sua aplicação se houver mudanças incompatíveis na nova versão.

---

## 5. Usando Multi-Stage Builds

Os builds de múltiplas etapas (Multi-Stage Builds) são um recurso poderoso e possivelmente subestimado do Docker. O conceito é dividir o processo de construção da imagem em várias etapas.

No final, apenas o conteúdo da última etapa será incluído na imagem final, enquanto as etapas anteriores serão descartadas.

Um caso de uso típico é utilizar ferramentas de build e código-fonte na primeira etapa para criar arquivos binários e, em seguida, copiar apenas esses arquivos binários para a próxima etapa.

A imagem final não incluirá o código-fonte nem as ferramentas de build, o que faz sentido, pois a imagem final deve apenas executar a aplicação, não construí-la.

Por exemplo, considere este Dockerfile oficial usado para construir e executar uma aplicação ASP.NET:

```dockerfile
FROM mcr.microsoft.com/dotnet/sdk:7.0 AS build-env
WORKDIR /App
​
# Build the app
COPY . ./
RUN dotnet restore
RUN dotnet publish -c Release -o out
​
# Build runtime image
FROM mcr.microsoft.com/dotnet/aspnet:7.0
WORKDIR /App
COPY --from=build-env /App/out .
ENTRYPOINT ["dotnet", "DotNet.Docker.dll"]
```

Observe o comando `COPY --from=build-env /App/out .`, que copia os arquivos binários da primeira etapa para a segunda etapa.

A primeira etapa é baseada em uma imagem que contém ferramentas de build (`mcr.microsoft.com/dotnet/sdk:7.0`), enquanto a segunda etapa é baseada em uma imagem de runtime menor (`mcr.microsoft.com/dotnet/aspnet:7.0`).

Além disso, repare como eles especificam a versão secundária da imagem.

---

## 6. Considere Usar Imagens Slim e Alpine

Ao trabalhar com Docker, considere o uso de imagens **Slim** ou **Alpine** sempre que possível. Essas versões são significativamente menores em tamanho e ajudam a reduzir o tempo de download e o espaço em disco ocupado pelas imagens.

As imagens **Alpine** são baseadas no Alpine Linux, conhecido por sua leveza, o que torna as imagens Alpine teoricamente mais rápidas de construir, puxar e executar.

Aqui está um exemplo do tamanho de diferentes imagens Python:

| REPOSITORY | TAG          | SIZE   |
|------------|--------------|--------|
| python     | 3.10-alpine  | 50.4MB |
| python     | 3.10-slim    | 128MB  |
| python     | 3.10         | 1GB    |

Tomando o Python como exemplo, o tamanho da imagem Alpine é 1/20 do tamanho da imagem completa baseada no Debian.

O Python também oferece imagens Slim, que são baseadas no Debian, mas com a maioria dos pacotes padrão removidos.

### Problema Comum com Imagens Menores

Um problema comum com essas imagens menores é que, se sua aplicação for um pouco mais complexa, você pode precisar instalar pacotes adicionais, o que inevitavelmente aumentará o tempo de build e o tamanho da imagem, contrariando o propósito original de escolhê-las.
