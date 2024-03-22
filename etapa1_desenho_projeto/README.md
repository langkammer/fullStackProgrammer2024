# Etapa 1

Nesta etapa, será criado o desenho do projeto, onde veremos ponto a ponto onde iremos atuar em cada etapa.

## Desenho Geral do Projeto
![img.png](img.png)

## Estratégia
Como o desenho do projeto é extenso, será dividido em várias etapas. Sugiro começar desenvolvendo as bordas ou fronteiras do projeto e gradualmente avançar para o centro do projeto. Abaixo, listarei as responsabilidades de cada aplicação.

### Lista de Aplicações a serem criadas
- [Consulta Conta](#consulta-conta)
- [Consulta Chave Pix](#consulta-chave-pix)
- [Api de Fraudes](#aplicação-de-leitura-topico-kafka-fraude)
- [Api de Verifica Saldo e Bloqueia](#autorizar-pagamento-pix)
- [Api de Inclusão Pagamento](#inclusão-de-pagamento-pix)
- [Api de Autorização Pagamento](#autorizar-pagamento-pix)
- [Efetivação de Pagamento](#efetivar-pagamento-pix)

### Coração da Ideia do Projeto
Desenvolver uma aplicação PIX pagamentos completa, desde a inclusão até a autorização e efetivação do pagamento, passando pelas etapas de consulta de dados de pagamento e chaves de acesso e aplicações de consumo de mensageria.
1. ### Inclusão de Pagamento Pix
    - Receberá uma requisição do tipo POST com o JSON.
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
        - Validação se a situação do pagamento é aguardando_confirmação
        - Envia Sqs para fila de efetivação
        - Grava na tabela pagamento_pix com situação aguardando_efetivacao
3. ### Efetivar Pagamento Pix
    - Consumirá mensagens Sqs [Link do Contrato](contratos/efetivacao_sqs_pix/contrato.json).
    - Validações do contrato:
        - Validação do id pagamento existente
        - Valida se data do pagamento é maior que a data atual
    - Procedimento:
        - Produtor de mensagens Kafka
        - Para toda mensagem enviada, o cabeçalho ou header das mensagens Kafka será enviado os seguintes argumentos
            - chave_pix = vem da inclusão do pagamento
            - id_pagamento_pix = uuid gerado na inclusão do pagamento
            - tipo_pagamento = "pix"
        - Envia primeiro para aplicação de fraude na fila mencionada e tendo retorno se é ou não fraude ele bloqueia saldo e com id do bloqueio ele efetiva transação na aplicação de efetivação
            - Envia Fraude de pix passando agência e conta do pagador, chave pix recebedor [Link do Contrato](contratos/fraude_validacao/contrato.json).
            - Envia Bloqueia Saldo [Link do Contrato](contratos/bloqueio_saldo/contrato.json).
            - Efetiva Pagamento Pix [Link do Contrato](contratos/efetivacao_sqs_pix/contrato.json).
        - Consumidor Kafka
            - Filtrando mensagens no cabeçalho ou no header, tipo_pagamento = "pix"
                - Consome Retorno de Fraudes [Link do Contrato](contratos/fraude_validacao/retorno_contrato.json).
                - Consome Retorno de Bloqueio de Saldo [Link do Contrato](contratos/bloqueio_saldo/retorno_contrato.json).
                - Consome Retorno da efetivação do pagamento [Link do Contrato](contratos/efetivacao_pagamento_conta/retorno_contrato.json).
        - Casos:
            - Em caso de fraude, grava situação do pix como suspeita_de_fraude
            - Em caso de bloqueio de saldo por falta de saldo, ele gravará cliente_sem_saldo
            - Em caso de erro na efetivação por falta, gravará erro_na_efetivacao

4. ### Consulta Conta
    - Esta API recebe via GET agência e conta e devolve esse contrato [Link do Contrato](contratos/consulta_conta_origem/contrato.json).
    - Faça uma tabela chamada dados_conta [Link da Estrutura do Banco de Dados](banco_dados/dados_conta.json).
    - E pode fazer uma pequena massa de registros como preferir.
    - Esta API deverá consultar por agência e conta nessa tabela e deverá retornar os dados informados no contrato.


6. ### Aplicação de leitura topico kafka fraude
    - Essa Api é simples ela faz leitura do topico fraude-pix recebendo esses dados [Link do Contrato](contratos/fraude_validacao/contrato.json).
    - Faça um tabela chamada fraude [Link da extrutura do banco de dados](banco_dados/dados_chave_pix.json).
    - E pode fazer uma pequena massa de registros como preferir
    - Essa api deverá consultar por chave e tipo de chave nessa tabela
    - e deverá retornar os dados informados no contrato