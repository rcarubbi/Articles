# 🤖 Como eu desenvolvi uma plataforma web de criação de chatbots com interface gráfica baseada em blockly 🧩

Criar uma plataforma de chatbot que combina facilidade de uso com flexibilidade para fluxos complexos foi o objetivo principal do [Carubbi.ChatbotStudio](https://github.com/rcarubbi/Carubbi.ChatbotStudio). A plataforma foi projetada com uma interface visual baseada no Blockly para possibilitar que os usuários criem fluxos de conversa usando blocos visuais e também implementa uma série de funcionalidades avançadas para garantir personalização e interatividade. Aqui, compartilho como desenvolvi essa solução, destacando os principais recursos, processos e desafios.

---

### 1. **Objetivo e Escopo da Plataforma**

Desde o início, a plataforma foi pensada para facilitar o desenvolvimento de chatbots sem que os usuários precisassem programar diretamente. Os requisitos principais incluíam:

- **Interface Visual Intuitiva**: onde o usuário arrastasse e organizasse blocos que representam cada etapa do diálogo.
- **Testes em Tempo Real**: para que o usuário possa simular o funcionamento do bot enquanto configura o fluxo.
- **Ambientes de Desenvolvimento e Produção**: para garantir que o ambiente de desenvolvimento e ajustes não afete o bot em produção.

### 2. **Escolha das Ferramentas e Tecnologias**

Para o backend do bot, utilizei o **Microsoft Bot Framework**, um framework extensível que permite desenvolver bots escaláveis e personalizáveis. A interface gráfica foi desenvolvida com **Blockly**, que facilita a construção visual de lógicas e fluxos.

#### Tecnologias Utilizadas:

- **Backend**: Microsoft Bot Framework, C#.
- **Frontend**: Blockly, React e HTML5.
- **Simulador para Testes em Tempo Real**: que permite testar o fluxo enquanto ele é configurado.

### 3. **Implementação da Interface Gráfica com Blockly**

A implementação do Blockly foi um dos pontos mais importantes, uma vez que ele permite ao usuário estruturar o fluxo do chatbot usando blocos visuais que representam diferentes ações.

1. **Configuração do Blockly**: A interface do Blockly foi configurada no frontend e adaptada com blocos customizados para as ações comuns de um chatbot, como exibir uma mensagem, capturar a resposta do usuário e conduzir perguntas condicionais.
2. **Blocos Customizados**: Cada bloco criado representa um tipo de interação no fluxo do bot. Para o projeto, criei blocos específicos que permitem configurar cada parte do diálogo, como perguntas e respostas.
3. **Conversão dos Blocos para JSON**: O Blockly gera um JSON que contém a estrutura do fluxo montado pelo usuário. Esse JSON é então enviado ao backend, que interpreta e executa o fluxo conforme configurado.

### 4. **Integração do Backend com o Microsoft Bot Framework**

O backend da plataforma foi desenvolvido para processar o JSON gerado no frontend e traduzir as instruções visuais em um fluxo de conversação no Microsoft Bot Framework. Isso permite ao bot seguir a sequência configurada pelo usuário, garantindo flexibilidade e personalização em cada conversa.

- **Processamento de JSON no Bot Framework**: O JSON é interpretado pelo backend, que o converte em diálogos e fluxos no Microsoft Bot Framework. Cada instrução configurada no Blockly é traduzida em uma ação do bot, como fazer uma pergunta ou responder ao usuário.
- **Recuperação de Valores Informados pelo Usuário**: Para permitir personalização no fluxo, o bot armazena e resgata informações fornecidas pelo usuário em cada etapa. Essa funcionalidade permite que os dados coletados durante a conversa sejam utilizados posteriormente.

### 5. **Resgate de Valores e Expressões com Razor Engine**

A plataforma utiliza a **Razor Engine** para processar expressões dinâmicas e personalizar as respostas do bot conforme os valores fornecidos pelos usuários em etapas anteriores. Esse recurso permite que o bot resgate variáveis configuradas em passos anteriores, tornando o fluxo mais flexível e contextual.

#### Exemplo de Expressões Usadas no Projeto

No código, uma expressão comum usada para acessar valores anteriores é do tipo:

- **`@@Step2.Output.Answer`**: Esta expressão captura a resposta (`Answer`) dada pelo usuário na etapa `Step2`, que é uma etapa do tipo `Question`.

Essa funcionalidade é particularmente útil para personalizar o diálogo com base nas respostas que o usuário já forneceu, ajustando a experiência conversacional em tempo real.

### 6. **Intellisense para Sugerir Expressões de Valores Capturados**

Para facilitar o uso das expressões, a interface conta com um **Intellisense** que sugere automaticamente os passos e os valores capturados anteriormente enquanto o usuário digita a expressão `@@`. Essa funcionalidade ajuda o usuário a configurar o bot com mais precisão e reduz a possibilidade de erros de configuração.

### 7. **Gerenciamento de Ambientes de Desenvolvimento e Produção**

A plataforma também permite configurar e gerenciar ambientes de **desenvolvimento** e **produção** usando variáveis específicas, assegurando que o bot possa ser ajustado sem afetar a versão ativa.

1. **Configuração de Variáveis de Ambiente**: O projeto utiliza variáveis de ambiente para definir parâmetros importantes, como identificadores do bot e URLs de endpoints. Com isso, o mesmo código pode ser utilizado em diferentes ambientes, bastando alterar as variáveis de configuração.
2. **Configurações Específicas no Bot Framework**: As configurações do Microsoft Bot Framework permitem gerenciar fluxos de produção e desenvolvimento de forma separada, garantindo que os testes e ajustes de configuração ocorram com segurança no ambiente de desenvolvimento antes de qualquer atualização em produção.

### 8. **Testes e Validações em Tempo Real**

O simulador de testes em tempo real permite que o usuário interaja com o bot durante o processo de configuração, testando fluxos de conversa e ajustando-os conforme necessário. Essa funcionalidade foi essencial para garantir que o bot funcionasse conforme esperado, facilitando a identificação de ajustes necessários.

---

Esse projeto está disponível como open-source no [GitHub](https://github.com/rcarubbi/Carubbi.ChatbotStudio) e serve como um ponto de partida para quem deseja explorar o desenvolvimento de chatbots com interface visual e personalizada. Convido você a conferir o repositório e contribuir com sugestões, melhorias ou adaptações para outros projetos!

Veja também essa demo que eu coloquei no youtube: https://www.youtube.com/watch?v=Ri5ABGHxGq0
