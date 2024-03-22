# Etapa 1

Nesta etapa, será criado o desenho do projeto, onde veremos ponto a ponto onde iremos atuar em cada etapa.

## Desenho Geral do Projeto
![img.png](img.png)

## Estratégia
Como o desenho do projeto é extenso, será dividido em várias etapas. Sugiro começar desenvolvendo as bordas ou fronteiras do projeto e gradualmente avançando para o centro do projeto. Abaixo listarei as responsabilidades de cada aplicação.

### Coração da Ideia do Projeto
1. ### Inclusão de Pagamento Pix
    - Irá receber uma requisição do tipo POST com o JSON.
    - [Link do Contrato](contratos/inclusao_pagamento_pix/contrato.json)
    - [Link do Exemplo](contratos/inclusao_pagamento_pix/exemplo.json)
    - Tipos de chaves de pagamento possíveis:
        - email
        - chave_aleatoria
        - cpf
        - telefone
    - Validações do contrato:
        - Ao incluir o pagamento, deverá validar se todos os atributos estão preenchidos.
        - A data do pagamento deverá ser superior ou igual à data atual.
        - O valor do pagamento deverá ser superior a ZERO.
    - Procedimento:
        - Ao efetivar o pagamento, deverá receber o payload do contrato e consultar a agência e a conta origem na API [nome_api_consulta].
        - Ao enviar agência e conta para essa API, o retorno dela deverá ser [Link do Exemplo](contratos/consulta_conta_origem/exemplo.json).
        - Se houver retorno válido dessa API, enriquecer os dados do domínio da aplicação para salvar. Caso não exista, a inclusão do pagamento deverá ser negada.
        - Consultar chave pix destino.
        - Ao consultar chave pix, o retorno deve ser [Link do Contrato](contratos/consulta_chave_pix/contrato.json).
        - Se houver retorno válido dessa API, enriquecer os dados do domínio da aplicação para salvar. Caso não exista, a inclusão do pagamento deverá ser negada.
        - Depois, será gravado na tabela de pagamentos_pix [Link da Estrutura da Tabela](banco_dados/pagamentos_pix.json).
2. ### Autorizar Pagamento Pix
   - Irá receber uma requisição PUT.
   - Validações do contrato:
      - Validação do id pagamento existente
   - Procedimento:
      - Validação se a situação do pagamento é aguarando_confirmação 
      - Envia Sqs para fila de efetivação
      - Grava na tabela pagamento_pix com situação aguardando_efetivacao
3. ### Efetivar Pagamento Pix
   - Consumira Msgs Sqs [Link do Contrato](contratos/efetivacao_sqs_pix/contrato.json).
   - Validações do contrato:
      - Validação do id pagamento existente
      - Valida se data do pagamento é maior que a data atual
   - Procedimento:
     - Produtor Msg Kafka
        - Envia Fraude de pix passando agencia e conta do pagador chave pix recebedor [Link do Contrato](contratos/fraude_validacao/contrato.json).
        - Envia Bloqueia Saldo [Link do Contrato](contratos/bloqueio_saldo/contrato.json).
        - Efetiva Pagamento Pix [Link do Contrato](contratos/efetivacao_sqs_pix/contrato.json).
     - Consumer Kafka
       - Consome Retorno filtrando filtrando pelo tipo pagamento pix
       - Consome Retorno de Bloqueio de Saldo
       - Consome Retorno de da efetivação do pagaemtno
       - Todos os consumidores deverão retornar um contrato 