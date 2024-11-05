
# Tutorial AWS Step Functions

Este é um guia que fiz para explicar o que eu entendi sobre o uso da ferramenta **AWS Step Functions**. 
Basicamente, o Step Functions é um serviço da AWS para organizar e automatizar fluxos de trabalho complexos, permitindo que eu coordene 
vários serviços e tarefas dentro da AWS. Vou detalhar abaixo como criar uma **state machine** (máquina de estado) básica e alguns dos 
conceitos principais que aprendi.

## 1. Pré-requisitos

Antes de começar, você precisa:
- Ter uma conta na AWS.
- Permissões para criar e gerenciar funções Lambda, state machines no Step Functions e outros serviços que desejar integrar.
- Entendimento básico sobre funções Lambda pode ajudar.

## 2. O que são State Machines no Step Functions

A **state machine** é o coração do Step Functions. Basicamente, ela define um fluxo com uma série de estados, onde cada estado 
representa uma etapa da execução. Esses estados podem incluir tarefas, verificações de condição, espera, entre outros. 

Tipos de estados comuns que eu aprendi:
- **Task**: executa uma tarefa específica, como chamar uma função Lambda.
- **Choice**: adiciona uma lógica condicional.
- **Wait**: permite pausar a execução por um tempo.
- **Parallel**: executa várias etapas ao mesmo tempo.

## 3. Acessando o AWS Step Functions

1. No **Console da AWS**, vá para **Step Functions**.
2. Clique em **Create state machine**.

## 4. Criando uma State Machine

O Step Functions usa uma linguagem JSON chamada **Amazon States Language (ASL)** para definir o fluxo de cada estado. 
Abaixo está um exemplo básico que fiz para entender melhor.

### Exemplo de state machine simples

```json
{
  "StartAt": "HelloWorld",
  "States": {
    "HelloWorld": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:us-east-1:123456789012:function:HelloWorld",
      "End": true
    }
  }
}
```

### Explicação

- **StartAt**: define o estado inicial (neste caso, “HelloWorld”).
- **States**: lista de estados.
- **Type**: "Task" indica que será executada uma função Lambda.
- **Resource**: ARN da função Lambda que será chamada.
- **End**: termina a execução após a conclusão desse estado.

## 5. Adicionando Condições com Choice States

Achei interessante que o estado **Choice** permite adicionar condições para decidir o próximo passo. 
Veja um exemplo que criei para testar a condição de sucesso ou falha.

```json
{
  "StartAt": "CheckStatus",
  "States": {
    "CheckStatus": {
      "Type": "Choice",
      "Choices": [
        {
          "Variable": "$.status",
          "StringEquals": "SUCCEEDED",
          "Next": "SuccessState"
        },
        {
          "Variable": "$.status",
          "StringEquals": "FAILED",
          "Next": "FailedState"
        }
      ],
      "Default": "UnknownState"
    },
    "SuccessState": {
      "Type": "Pass",
      "Result": "The task succeeded!",
      "End": true
    },
    "FailedState": {
      "Type": "Pass",
      "Result": "The task failed.",
      "End": true
    },
    "UnknownState": {
      "Type": "Fail",
      "Error": "UnknownStatus",
      "Cause": "The status was not recognized."
    }
  }
}
```

### Explicação

- **Choices**: Verifica o valor de `status` e direciona o fluxo de acordo.
- **Default**: Se `status` não for SUCCEEDED ou FAILED, direciona para `UnknownState`.

## 6. Configurando e Executando a State Machine

1. Copie o JSON e cole na área de **Definition** ao criar a state machine no console da AWS.
2. Dê um nome e selecione o tipo da máquina: **Standard** ou **Express**.
3. Clique em **Create state machine**.
4. Para executar, clique em **Start execution**, insira o **input JSON** e observe o fluxo em tempo real.

## 7. Monitoramento e Logs

O Step Functions se integra com o **CloudWatch** para logs e métricas. Isso me ajuda a verificar a duração de cada estado, 
erros e monitorar o desempenho.

## 8. Exemplo Completo com Lambda e DynamoDB

Para testar o uso prático, criei um fluxo que verifica um pedido em DynamoDB e processa com Lambda:

```json
{
  "StartAt": "GetOrder",
  "States": {
    "GetOrder": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:us-east-1:123456789012:function:GetOrder",
      "Next": "CheckInventory"
    },
    "CheckInventory": {
      "Type": "Choice",
      "Choices": [
        {
          "Variable": "$.inventoryAvailable",
          "BooleanEquals": true,
          "Next": "ProcessOrder"
        }
      ],
      "Default": "OutOfStock"
    },
    "ProcessOrder": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:us-east-1:123456789012:function:ProcessOrder",
      "End": true
    },
    "OutOfStock": {
      "Type": "Fail",
      "Error": "OutOfStockError",
      "Cause": "Inventory not available"
    }
  }
}
```

### Conclusão

O AWS Step Functions facilita muito a criação de fluxos de trabalho complexos e permite integrar serviços da AWS sem precisar 
de servidores. Essa prática ajudou bastante a entender a ferramenta e agora me sinto mais preparada para criar e automatizar 
meus próprios workflows!
