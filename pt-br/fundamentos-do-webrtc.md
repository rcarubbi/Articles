# Fundamentos do WebRTC: Construindo uma Aplicação de Videochamada

O **WebRTC (Web Real-Time Communication)** é uma tecnologia que permite que navegadores e aplicações se comuniquem em tempo real, viabilizando videochamadas, troca de áudio e dados sem a necessidade de plugins. Essa tecnologia, que opera em conexões peer-to-peer, reúne várias APIs e protocolos para estabelecer uma comunicação robusta, segura e de baixa latência.

Neste artigo vamos explorar os conceitos fundamentais do WebRTC, utilizando exemplos reais do repositório [carubbi-video-link](https://github.com/rcarubbi/carubbi-video-link) e complementando com pontos de vista de outros artigos e da documentação oficial, para que o fluxo da explicação seja o mais claro e completo possível.

---

## Como o WebRTC Funciona?

Em essência, o WebRTC permite a comunicação direta entre navegadores (ou entre aplicações) sem precisar rotear o tráfego de mídia por um servidor central. Porém, para que dois pares possam estabelecer uma conexão, é preciso que haja uma troca inicial de informações (sinalização) que define como os dados serão enviados. Essa troca envolve:

- **Session Description Protocol (SDP):** Define os parâmetros da sessão, como codecs, formatos de mídia e tipos de transporte.
- **ICE (Interactive Connectivity Establishment):** Um framework que identifica a melhor rota para a conexão.
- **STUN/TURN:** Servidores que ajudam na descoberta do endereço IP público e, se necessário, atuam como intermediários quando conexões diretas não são possíveis.

> Essas etapas garantem que mesmo em redes com NATs e firewalls a comunicação possa ser estabelecida de forma eficiente.  

---

## Captura de Mídia com `getUserMedia`

O primeiro passo em uma aplicação WebRTC é capturar o áudio e o vídeo do usuário. A API `navigator.mediaDevices.getUserMedia` solicita acesso aos dispositivos de captura (como câmera e microfone) e retorna um objeto `MediaStream`.

No repositório **carubbi-video-link** essa etapa é realizada de forma robusta. Veja um exemplo simplificado:

```javascript
async function requestMediaPermissions() {
  try {
    const stream = await navigator.mediaDevices.getUserMedia({
      video: true,
      audio: true,
    });
    // Para não manter a câmera ativa desnecessariamente, as tracks são paradas
    stream.getTracks().forEach((track) => track.stop());
  } catch (err) {
    ui.showError("Access to user media denied");
  }
}
```
Este código é necessário para que o navegador solicite as permissões para acessar os dispositivos de audio e video de modo que possamos em seguida chamar o método `enumerateDevices` que popula os dropown lists permitindo que o usuário selecione qual dispositivo ele quer usar na chamada. Esta seleção está disponível na tela de configurações.

Além disso, ao iniciar uma chamada, o código captura a mídia com **constraints** que definem parâmetros como resolução, taxa de quadros e configurações de áudio (cancelamento de eco, supressão de ruído, etc.):

```javascript
_localStream = await navigator.mediaDevices.getUserMedia({
  video: { deviceId: _videoDeviceId, ...videoConstraints },
  audio: { deviceId: _audioDeviceId, ...audioConstraints },
});
ui.showLocalVideoStream(_localStream);
```

> Essa abordagem garante alta definição e qualidade, adaptando os dispositivos disponíveis conforme as necessidades do usuário.  

---

## Conectividade

### ICE Framework

Para falar de conectividade é importante entendermos um pouco sobre o **ICE (Interactive Connectivity Establishment)** que é um parte essencial da pilha WebRTC, responsável por descobrir e selecionar a melhor rota para a comunicação entre dois pares. 

ICE é um conjunto de métodos e protocolos que permite aos navegadores (ou outras aplicações) identificar possíveis caminhos de rede (candidates) para estabelecer uma conexão peer-to-peer. Esses candidates podem ser:

- **Host candidates:** Endereços locais da máquina.
- **Server Reflexive candidates (STUN):** Endereços públicos descobertos via servidores **STUN (Session Traversal Utilities for NAT)** explicados mais adiante.
- **Relayed candidates (TURN):** Endereços fornecidos por servidores **TURN (Traversal Using Relays around NAT)** que também serão explicados posteriormente.

Esses candidatos são trocados entre os pares por meio do **Signaling Server**. A troca de candidates é fundamental para que ambos os lados saibam quais endereços podem ser usados para a conexão.

O ICE tenta primeiro estabelecer uma conexão direta. Se isso falhar, ele tenta um servidor **STUN** para descobrir um caminho possível. Caso a conexão ainda não seja possível devido a firewalls ou NATs restritivos, o ICE utiliza um **servidor TURN**, que age como um retransmissor para garantir a comunicação.

---

#### O Processo de Troca de Candidates

Dentro do contexto do WebRTC, a troca de ICE candidates ocorre da seguinte forma:

1. **Coleta de Candidates:**  
   Quando um `RTCPeerConnection` é criado, ele começa a coletar possíveis candidates com base nos dispositivos e nas condições da rede. Essa coleta é realizada de forma assíncrona.

2. **Callback `onicecandidate`:**  
   Cada vez que um novo candidate é encontrado, o callback `onicecandidate` é disparado. Um exemplo extraído do arquivo [webrtc.js](https://github.com/rcarubbi/carubbi-video-link/blob/main/client/src/js/webrtc.js) mostra:

   ```javascript
   _peerConnection.onicecandidate = (event) => {
     if (event.candidate) {
       signalingClient.sendIceCandidate({
         candidate: event.candidate,
         remoteUserId: _remoteUserId,
       });
     }
   };
   ```

   Nesse trecho, cada candidate é enviado para o par remoto através do mecanismo de sinalização (aqui abstraído pelo `signalingClient`).

3. **Adição dos Candidates à Conexão:**  
   Do lado que recebe os candidates, eles são adicionados à instância de `RTCPeerConnection` usando o método `addIceCandidate`.

   ```javascript
   export async function addIceCandidate(candidate) {
    if (!_peerConnection || !_peerConnection.remoteDescription) {
        _pendingIceCandidates.push(candidate);
    } else {
        await _peerConnection.addIceCandidate(new RTCIceCandidate(candidate));
    }
   }
   ```

---

#### Por Que Armazenar Candidates Temporariamente?

Uma questão comum durante o estabelecimento da conexão é que os ICE candidates podem chegar **antes** que o par remoto esteja pronto para processá-los. Mais especificamente, o método `addIceCandidate()` só funciona corretamente se a descrição remota (obtida via `setRemoteDescription()`) já estiver configurada na conexão. Se os candidates forem enviados antes dessa configuração, o RTCPeerConnection ainda não está apto a integrá-los, podendo resultar em erros ou na perda de candidates importantes.

Para evitar esse problema, o exemplo do repositório utiliza um array (_pendingIceCandidates) para armazenar temporariamente os ICE candidates recebidos. Assim que a descrição remota é definida, uma função (como `flushPendingIceCandidates()`) é chamada para adicionar todos os candidates armazenados:

```javascript
async function flushPendingIceCandidates() {
  for (const candidate of _pendingIceCandidates) {
    await _peerConnection.addIceCandidate(new RTCIceCandidate(candidate));
  }
  _pendingIceCandidates = [];
}
```

Esse mecanismo garante que nenhum candidate seja descartado e que todos sejam processados assim que a conexão estiver pronta para receber novos candidates.

### STUN e TURN servers

O ICE depende de dois componentes essenciais para possibilitar a conexão peer-to-peer mesmo em redes com NAT e firewalls: **STUN** e **TURN**. Cada um deles possui funções distintas e impactos diferentes em termos de custo e infraestrutura.

---

#### STUN (Session Traversal Utilities for NAT)

O STUN é utilizado para que um cliente descubra seu endereço IP público e a porta utilizada pela conexão. Isso é feito através do envio de uma requisição a um servidor STUN, que retorna as informações necessárias. 

Como o STUN apenas informa o endereço sem retransmitir tráfego, ele não exige muita largura de banda então muitas empresas oferecem STUN servers de forma gratuita. Por exemplo, o Google disponibiliza servidores como `stun.l.google.com:19302`.

---

#### TURN (Traversal Using Relays around NAT)

O TURN é utilizado quando uma conexão direta não pode ser estabelecida (por exemplo, em redes com NATs restritivos ou firewalls). Nesse caso, o tráfego de mídia é retransmitido por um servidor TURN, atuando como relay.

Como o TURN precisa retransmitir streams de áudio e vídeo, ele utiliza uma quantidade significativa de largura de banda. Devido à alta demanda de recursos, o serviço TURN é geralmente cobrado.

Você ainda pode configurar um TURN server utilizando o **COTURN**, que é uma solução open-source. Contudo, isso requer infraestrutura própria e garantia de largura de banda suficiente para suportar o tráfego de relay.

---

##### Exemplo Prático: Utilizando o Serviço Metered

No repositório [carubbi-video-link](https://github.com/rcarubbi/carubbi-video-link/blob/main/client/src/js/webrtc.js) podemos observar a configuração de ICE que inclui tanto STUN quanto TURN servers. Por exemplo:

```javascript
const iceServers = [
  { urls: "stun:stun.relay.metered.ca:80" },
  {
    urls: "turn:global.relay.metered.ca:80",
    username: "**********",
    credential: "**********",
  },
  // Outras configurações TURN, incluindo opções via TCP e secure (turns)
];
```

Neste caso, estamos utilizando o serviço da [Metered](https://dashboard.metered.ca), que oferece:
- Um **tier gratuito** com até 0.5GB mensais de dados para o relé das streams.
- Caso o uso ultrapasse esse limite, a conta é cobrada conforme o consumo de dados.

Essa abordagem é bastante prática para aplicações de desenvolvimento ou de pequena escala, onde o volume de tráfego é limitado. Para aplicações maiores, pode ser necessário considerar a escalabilidade ou até mesmo subir seu próprio servidor TURN com COTURN.

---

### Configurando o RTCPeerConnection

Agora que já desvendamos os conceitos de ICE, STUN e TURN, vamos dar uma olhada mais a fundo nas classes que a API do webRTC disponibiliza.

O núcleo da comunicação WebRTC é o objeto `RTCPeerConnection`, que gerencia a conexão direta entre os pares. No exemplo do repositório, a função `createPeerConnection` configura o objeto com uma lista de servidores ICE (incluindo STUN e TURN):

```javascript
async function createPeerConnection() {
  _peerConnection = new RTCPeerConnection({ iceServers });
  
  _peerConnection.onicecandidate = (event) => {
    if (event.candidate) {
      signalingClient.sendIceCandidate({
        candidate: event.candidate,
        remoteUserId: _remoteUserId,
      });
    }
  };

  _peerConnection.ontrack = (event) => {
    ui.showRemoteVideoStream(event.streams[0]);
  };

  // Adiciona todas as tracks do stream local à conexão
  _localStream.getTracks().forEach((track) => {
    _peerConnection.addTrack(track, _localStream);
  });
}
```

> Essa função também registra callbacks para o recebimento de candidatos ICE e para exibir a mídia remota na interface do usuário.  

### Processo de Negociação (Offer/Answer)

O fluxo para iniciar uma chamada envolve os seguintes passos:

1. **Captura da mídia local:** Com `getUserMedia`, conforme mostrado anteriormente.
2. **Criação de uma oferta (Offer):** O Cliente A que inicia a chamada gera uma descrição da sessão.
3. **Envio da oferta via sinalização:** A oferta é transmitida para o Cliente B através de um servidor de sinalização (geralmente via WebSocket ou outro mecanismo).
4. **Criação da resposta (Answer):** O receptor (Cliente B), ao receber a oferta, atribui a mesma ao seu  `RTCPeerConnection` encapsulando a offer em um objeto `RTCSessionDescription`, após isto, configura sua mídia, cria uma resposta e a envia de volta ao Cliente A.
5. **Recebimento da resposta:** Ao receber a resposta, o cliente A atribui a resposta ao seu `RTCPeerConnection` também encapsulando em um `RTCSessionDescription`. 
6. **Troca de ICE Candidates:** Durante e após essa negociação, ambos os lados trocam candidatos para otimizar a rota da conexão como explicado no capítulo sobre ICE. 

Exemplo da função que inicia uma chamada:

```javascript
export async function startCall({ localUserId, remoteUserId }) {
  _remoteUserId = remoteUserId;
  
  // Captura a mídia local com os constraints definidos
  _localStream = await navigator.mediaDevices.getUserMedia({
    video: { deviceId: _videoDeviceId, ...videoConstraints },
    audio: { deviceId: _audioDeviceId, ...audioConstraints },
  });
  ui.showLocalVideoStream(_localStream);
  
  await createPeerConnection();
  
  // Cria a oferta e configura a descrição local
  const offer = await _peerConnection.createOffer();
  await _peerConnection.setLocalDescription(offer);
  
  // Envia a oferta para o usuário remoto
  signalingClient.sendOffer({
    to: _remoteUserId,
    from: localUserId,
    offer,
  });
}
```

> Esse fluxo segue o padrão "offer/answer" definido pelo protocolo SDP e é essencial para a sincronização e estabelecimento da conexão P2P.  

---

### Sinalização e Fluxo de Dados

Embora o WebRTC defina o modelo para a troca de mídia e dados, o mecanismo de sinalização – responsável pela troca inicial de informações (como SDP e ICE candidates) – não é especificado. Cabe ao desenvolvedor implementar a sinalização, usando ferramentas como WebSockets, SIP ou outras tecnologias.

> Essa flexibilidade permite que cada aplicação adapte o mecanismo de sinalização às suas necessidades específicas.  

---

## Gerenciamento da Sessão e Recursos

Além de estabelecer a conexão, a aplicação precisa gerenciar a troca de mídia durante toda a chamada. As funções para alternar o áudio ou vídeo (por exemplo, `toggleAudio` e `toggleVideo`) demonstram como controlar dinamicamente os recursos de mídia:

```javascript
export function toggleVideo() {
  _localStream.getVideoTracks()[0].enabled = !_localStream.getVideoTracks()[0].enabled;
}

export function toggleAudio() {
  _localStream.getAudioTracks()[0].enabled = !_localStream.getAudioTracks()[0].enabled;
}
```

Outras funções importantes incluem:
- **Aceitar ou rejeitar uma chamada:** Processamento da oferta recebida e envio da resposta.
- **Encerrar a chamada:** Parar os tracks da mídia, fechar a conexão e limpar os recursos.

> Esses controles são essenciais para uma experiência de comunicação fluida e responsiva.  

---

## Conclusão

O WebRTC revolucionou a forma como implementamos comunicações em tempo real, permitindo que videochamadas e troca de dados aconteçam diretamente entre navegadores sem a necessidade de plugins ou infraestrutura pesada. Ao explorar os conceitos de captura de mídia, criação de conexões P2P, negociação via SDP e o uso de ICE/STUN/TURN, você pode construir aplicações robustas e escaláveis.

O exemplo do repositório [carubbi-video-link](https://github.com/rcarubbi/carubbi-video-link) demonstra de maneira prática como implementar esses conceitos usando JavaScript, proporcionando uma base sólida para que você comece a desenvolver suas próprias soluções de comunicação.

Se você deseja se aprofundar ainda mais, recomendo a leitura da [documentação do MDN sobre WebRTC](https://developer.mozilla.org/pt-BR/docs/Web/API/WebRTC_API) e a exploração de tutoriais que abordam tanto a parte teórica quanto a prática da tecnologia.
