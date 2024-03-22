# Etapa 1

Nessa etapa será criado o desenho do projeto onde veremos ponto a ponto onde nos iremos atuar em cada etapa.



## Desenho Geral do Projeto
![img.png](img.png)

## Estrategia
Como o desenho do projeto é gigante será dividido em x etapas, para que tais etapas seja feito sugiro que comece desenvolvendo as beiradas 
ou as fronteiras do projeto e indo depois chegando mais dentro do coração do projeto abaixo listarei o que cada aplicação
é responsavel

Coração da ideia do projeto
1) Incluir Pagamento Pix 
    1. Irá receber uma requisição com o json 
       2. [Link Do Contrato](etapa1_desenho_projeto/contratos/inclusao_pagamento_pix/contrato.json)
       3. [Link do exemplo](etapa1_desenho_projeto/contratos/inclusao_pagamento_pix/exemplo.json)  
       4. Tipos pagamentos chaves possiveis: 
          5. email
          6. chave_aleatoria
          7. cpf 
          8. telefone
       10. Validações do contrato 
           11. Ao Incluir Pagamento deverá validar se todos os atributos estão preenchidos
           12. Data do pagamento deverá ser superior ou igual a data de atual
           13. Valor do Pagamento deverá ser superior a ZERO
       11. Procedimento :
           12. Ao efetivar o Pagamento deverá receber o payload do contrato e consultar a agencia e conta origem na api item [nome_api_consulta]
               13. ao enviar agencia e conta para essa api o retorno dela deverá ser [Link do exemplo](etapa1_desenho_projeto/contratos/consulta_conta_origem/exemplo.json)
               14. Se houver retorno valido dessa api enriquecer os dados do dominio da aplicação para salvar caso não existe deverá ser negado a inclusão do pagamento
           15. Consultar chave pix destino
               16. ao consultar chave pix o retorno deve ser [Link do Contrato](etapa1_desenho_projeto/contratos/consulta_chave_pix/contrato.json)
               17. Se houver retorno valido dessa api enriquecer os dados do dominio da aplicação para salvar caso não existe deverá ser negado a inclusão do pagamento
           16. Depois será gravado na tabela do pagamentos_pix [Link da extrutura da tabela](etapa1_desenho_projeto/banco_dados/pagamentos_pix.json)