# Order_Flow

O Sistema de Gerenciamento de Pedidos é uma aplicação desenvolvida em java para o processo de gestão de pedidos. A plataforma foi projetada com uma arquitetura baseada em microserviços, permitindo um controle modular sobre os principais aspectos do negócio, incluindo clientes, produtos, estoque, ordens de venda e pagamentos.

Essa documentação tem como objetivo apresentar o projeto, as requisições e modelo de dados utilizados. 

## Tecnologias Utilizadas

  - Event Storming: Miro;  
  - Linguagem: JAVA 17;  
  - Framework: Spring Boot, Spring JPA, Hibernate, Lombok; 
  - Repositório: GitHub; 
  - Banco de dados: H2 DataBase; 
  - Publicação: Docker; 
  - Testes: JUnit, Mockito;
  - Dependências: Maven; 

## Funcionalidades

  - Cadastro de Clientes
  - Cadastro de Endereços
  - Cadastro de Produtos
  - Gerenciamento de Estoque
  - Cadastro de Peditos
  - Processo de Pagamento

## Event Storming

Para visualizar a fase final e o resultado obtido durante o desenvolvimento do Event Storming, acesse o link abaixo:

https://miro.com/app/board/uXjVI0bo6mY=/

![image](https://github.com/CamilaLima21/order-flow/blob/main/MIRO.png?raw=true)

## Modelo de Dados

A modelagem de dados proposta organiza de forma clara a relação entre as principais entidades envolvidas na operação de um sistema distribuído para gerenciamento de pedidos. A arquitetura baseada em microsserviços garante integração, consistência, escalabilidade e robustez para as operações. As principais entidades incluem clientes, endereços, produtos, estoque, pedidos e pagamentos, respeitando regras essenciais de negócio como unicidade de CPF e SKU, controle de estoque e status de pedido com base em eventos do fluxo de compra.

1. **Endereços**: Modela os dados de entrega dos clientes, com campos como rua, número, bairro, cidade, estado e CEP. Um cliente pode ter vários endereços associados, sendo um deles utilizado no momento da compra. A entidade é fundamental para a logística de entrega dos pedidos.

2. **Clientes**: Representa os consumidores que realizam pedidos na plataforma. Cada cliente possui dados obrigatórios como nome completo, CPF (único no sistema), data de nascimento e telefone, estando vinculado a um ou mais endereços. O CPF é validado e não pode ser duplicado no sistema. Essa entidade centraliza as informações pessoais e cadastrais dos usuários para identificação e verificação em processos de negócio. 

3. **Produtos **: Contém os dados dos itens disponíveis para compra. Cada produto possui nome, SKU (único no sistema) e preço unitário. O SKU é validado para evitar duplicações e serve como identificador principal para controle de estoque e associação com os itens de pedidos. Essa entidade é essencial para a composição dos pedidos e cálculo do valor total da compra. 

4. **Estoque **: Gerencia a quantidade disponível de cada produto com base em seu SKU. Ao receber um pedido, esse serviço é acionado para verificar e debitar a quantidade solicitada de cada item. Caso não haja estoque suficiente para um dos produtos, o sistema reverte o processo de pagamento (se já autorizado) e marca o pedido com o status apropriado (FECHADO_SEM_ESTOQUE). Esse controle garante integridade e consistência do inventário. 

5. **Pedidos **: O microserviço **ms-orders** representa as solicitações de compra realizadas pelos clientes. Cada pedido está vinculado a:

- **Cliente** (identificado por um ID único);
- **Lista de itens**, contendo SKU e quantidades;
- **Dados de pagamento**, incluindo número do cartão;
- **Valor total**, calculado com base nos produtos solicitados;
- **Status**, refletindo o andamento do pedido.

### 🔄 Fluxo de Processamento de um Pedido

O processo de pedido segue um fluxo padrão, garantindo **rastreabilidade** e **consistência**:

a. O pedido inicia com status **`CREATED`**;
b. O estoque é verificado e os itens são debitados;
c. O pagamento é processado com base no valor total;
d. O status final depende da resposta dos serviços de estoque e pagamento:

   - ✅ **`CLOSED_SUCCESS`** → Pedido concluído com sucesso.
   - ❌ **`CLOSED_FAILED_NOT_STOCK`** → Falha devido à falta de estoque.
   - ❌ **`CLOSED_FAILED_NOT_PAID`** → Falha por pagamento recusado (estoque deve ser restabelecido).

6. **Pagamentos **: O microserviço **ms-payments** gerencia a autorização e validação do pagamento de um pedido, com base no número do cartão de crédito informado. O fluxo de pagamento segue as etapas abaixo:

1. O serviço recebe a solicitação de pagamento com o **valor total do pedido**.
2. A resposta da operadora de cartão é simulada utilizando **payments-wiremock**.
3. O status do pagamento (**APROVADO** ou **RECUSADO**) é registrado e comunicado ao serviço de pedidos.
4. Em caso de pagamento **recusado**, o serviço de estoque é notificado para restaurar as quantidades previamente debitadas.

### 🏗️ Arquitetura e Comunicação

Essa modelagem segue os princípios de **Clean Architecture**, garantindo uma separação clara de responsabilidades entre os microserviços. A comunicação ocorre de forma **assíncrona**, utilizando **eventos de domínio**, o que assegura:

- ✅ **Alta coesão interna**;
- ✅ **Baixo acoplamento entre serviços**;
- ✅ **Facilidade de manutenção e evolução**;
- ✅ **Tolerância a falhas em processos intermediários**;
- ✅ **Possibilidade de integração com serviços futuros**, como logística, monitoramento e fidelização.



A estrutura do sistema respeita as regras de negócio fornecidas e oferece uma base sólida para garantir uma experiência de compra confiável e segura para os usuários. 

Abaixo segue a diagrama da modelagem de dados relacional do sistema: 

![image](https://github.com/CamilaLima21/order-flow/blob/main/Relacionamento.png?raw=true))

## Como Executar

Para configurar a API, basta clonar o projeto: 

https://github.com/CamilaLima21/order-flow

## Recursos

### Address - Endereços
- Cadastrar Endereço: POST /addresses
- Consultar Endereço por Id: GET /addresses/{addressId}
- Consultar Todos Endereços: GET /addresses/{addressId}
- Consultar Endereço por CEP: GET /addresses/{zipCode}
- Editar um Endereço: PUT /addresses/{addressId}
- Deletar um Endereço: DELETE /addresses/{addressId}

### Clients - Clientes
- Cadastrar Cliente: POST /clients
- Consultar Cliente: GET /clients/{clientId}
- Consultar todos Clientes: GET /clients
- Editar um Cliente: PUT /clients/{clientId}
- Deletar um Cliente: DELETE /clients/{clientId}
- Consultar Cliente por cpf: /clients/{cpf}
- Consultar Cliente por nomme: /clients/{name}
- Consultar Cliente por email: /clients/{email}

### Products - Produtos
- Cadastrar um Produto: POST /products
- Consultar um Produto por Id: GET /products/{productId}
- Consultar um Produto por SKU: GET /products/{sku}
- Consultar todos Produto: GET /products
- Editar um Produto: PUT /products/{productId}
- Deletar um Produto: DELETE /products/{productId}

### Stocks - Estoques
- Cadastrar uma Estoque: POST /stocks
- Consultar uma Estoque por Id: GET /stocks/{stockId}
- Consultar uma Estoque por SKU: GET /stocks/{sku}
- Consultar todos os Estoque: GET /stocks
- Editar uma Estoque: PUT /stocks/{stockId}
- Deletar uma Estoque: DELETE /stocks/{stockId}
- Adicionar Estoque: POST /stocks/increase
- Remover Estoque: POST /stocks/decrease

### Orders - Pedidos
- Cadastrar um Pedido: POST /orders
- Pagar um Pedido: POST /orders/{orderId}/payment
- Consultar um Pedidos por Id: GET /orders/{orderId}
- Consultar todos os Pedidos: GET /orders
- Editar um Pedidos: PUT /orders/{orderId}
- Deletar um Pedidos: DELETE /orders/{orderId}

### OrdersItens - Itens do Pedido
- Cadastrar um Item: POST /order-items
- Consultar um Item por Id: GET /order-items/{itemId}
- Consultar todos os Itens: GET /order-items
- Editar um Item: PUT /order-items/{itemId}
- Deletar um Item: DELETE /order-items/{itemId}

- ### Payments - Pagamento
- Gerar Token de Autenticação: POST /payment/generateToken
- Gerar QR Code Pix: POST /payment/generateQR
- Verificar Pagamento PIX: GET /payment/{paymentId}
- Pagamento Cartão: POST /payment/card

## Pré-Requisitos
- Docker
- Java 17
- Maven 3.6+
- IDE como IntelliJ IDEA ou Eclipse

## Collection POSTMAN

A collection postman está disponível na raiz do projeto através do arquivo postman.json

## Licença
Este projeto é licenciado sob a Licença MIT.

## Contato
Camila Marques de Lima - cml.isa.17@gmail.com

Eduardo Bento Nakandakare - nakandakare9@gmail.com



