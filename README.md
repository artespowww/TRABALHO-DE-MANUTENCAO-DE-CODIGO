# Repositório da Disciplina de Manutenção de Software

Este é o repositório utilizado na disciplina de **Manutenção de Software** do curso de **Engenharia de Software da Unicesumar Ponta Grossa**.

## Mapa Mental da Disciplina

- [Mapa mental (XMind)](https://app.xmind.com/share/OBzglW7X)

## Visão Geral

O repositório reúne materiais de apoio e um projeto legado em Java para práticas de:

- manutenção preventiva
- manutenção corretiva
- manutenção evolutiva

O foco das atividades é melhorar o código de forma incremental, com mudanças pequenas, seguras e justificadas.

## Estrutura do Repositório

```text
unicesumar-manutencao/
├── exemplos/
│   └── clean-code/
├── ATIVIDADES/
│   ├── 1BIM/
│   │   ├── ATIVIDADE_1.md
│   │   └── ATIVIDADE_2.md
│   └── 2BIM/
└── library-maintenance-lab/
    ├── README.md
    └── src/
```

## Atividades

### 1º Bimestre

1. [Atividade 1 - Análise de Código e Manutenção Preventiva](ATIVIDADES/1BIM/ATIVIDADE_1.md)
2. [Atividade 2 - Manutenção Corretiva e Evolutiva](ATIVIDADES/1BIM/ATIVIDADE_2.md)

### 2º Bimestre

1. [Atividade 1 - Rastreamento e Classificação de Bugs](ATIVIDADES/2BIM/ATIVIDADE_1.md)
2. [Atividade 2 - Triagem, Relatório Profissional e Ciclo de Vida do Bug](ATIVIDADES/2BIM/ATIVIDADE_2.md)

## Lista de Exercícios de fixação para Prova 1

- [Lista de exercícios](LISTA_PROVA1.md)
- [Gabarito da lista](GABARITO_LISTA_PROVA1.md)

## Materiais Complementares

- [Projeto da biblioteca (contexto e execução)](library-maintenance-lab/README.md)
- [Exemplos de código limpo](exemplos/clean-code)

## Como Começar

1. Leia o contexto do projeto em [library-maintenance-lab/README.md](library-maintenance-lab/README.md).
2. Abra a [Atividade 1](ATIVIDADES/1BIM/ATIVIDADE_1.md).
3. Em seguida, desenvolva a [Atividade 2](ATIVIDADES/1BIM/ATIVIDADE_2.md).

## Licença

Este repositório está licenciado sob [LICENSE](LICENSE).

# ISSUE #1 – Tratamento Genérico de Exceções Oculta Erros Reais

## Título
Captura genérica de `Exception` impede identificação correta de falhas no sistema de empréstimos

---

## Descrição Detalhada

O sistema utiliza blocos `catch (Exception e)` em diversos fluxos críticos, especialmente nas operações de empréstimo (`handleBorrowBook()`), cadastro e devolução.

Essa abordagem captura qualquer tipo de erro de forma genérica, ocultando a causa real da falha e dificultando a manutenção do sistema.

Durante a execução, erros diferentes acabam produzindo a mesma mensagem para o usuário, impossibilitando identificar:

- erros de validação;
- falhas de lógica;
- dados inválidos;
- exceções inesperadas;
- bugs internos do sistema.

Além disso, a ausência de tratamento específico compromete o processo de debugging e dificulta a análise dos logs.

### Trecho identificado

```java
catch (Exception e) {
    System.out.println("Error borrow: " + e.getMessage());
    LegacyDatabase.addLog("handle-borrow-error-" + e.getMessage());
}
```

## Passos para Reproduzir 
Executar a aplicação LibrarySystem Acessar a opção:
```bash
    3 - Borrow book
Inserir valores inválidos durante o empréstimo:
```
```bash
    User ID: abc
    Book ID: xyz
```
Confirma Operação Comportamento Esperado O sistema deveria: identificar especificamente o tipo do erro; 
informar corretamente ao usuário o problema ocorrido; utilizar exceções específicas; registrar logs detalhados; evitar captura genérica de falhas inesperadas. 
## Exemplo esperado:

```bash
    Erro: o campo User ID deve conter apenas números.
```
ou
```bash
    Erro: livro indisponível para empréstimo.
```
## Comportamento Observado
 O sistema captura qualquer exceção utilizando catch (Exception e) e retorna mensagens genéricas. Saída observada:

```bash
    Error borrow: For input string: "abc"
```
Em alguns cenários, apenas:
```bash
   Error borrow
```
Isso dificulta: rastreamento da origem do problema;
 manutenção corretiva; 
 
identificação do bug real; 
análise técnica do sistema. 

## Ambiente de Execução
 Item Informação Linguagem / Java JDK / JDK 17 
 Sistema Operacional / Windows 11 
 IDE Eclipse / IntelliJ 
 Tipo da aplicação / CLI (Console) 
 Evidências da Ocorrência Entrada inválida no sistema
```bash
   Select option: 3

    User ID: abc
    Book ID: xyz
Saída da aplicação
```

```bash
   Error borrow: For input string: "abc"
Log registrado
```

```bash
   handle-borrow-error-For input string: "abc"
   ```


# ISSUE #2 – Quebra de Encapsulamento ao Retornar Estrutura Interna de Empréstimos

## Título
Método `getAllLoansDirect()` expõe diretamente a estrutura interna de dados do sistema

---

## Descrição Detalhada

O sistema possui um método chamado `getAllLoansDirect()` que retorna diretamente a lista interna de empréstimos armazenada no `LegacyDatabase`.

Esse comportamento viola o princípio de encapsulamento, pois permite que classes externas modifiquem diretamente os dados internos do sistema sem validação, controle ou regras de negócio.

### Trecho identificado

```java
public List<Map<String, Object>> getAllLoansDirect() {
    return LegacyDatabase.getLoans();
}
```

Como a lista original é retornada diretamente, qualquer alteração feita fora da classe afeta imediatamente os dados do sistema.

Isso pode causar:

- corrupção de dados;
- exclusão indevida de registros;
- inconsistência de empréstimos;
- alterações sem auditoria;
- comportamento inesperado da aplicação.

---

## Passos para Reproduzir

1. Executar a aplicação `LibrarySystem`

2. Obter acesso ao método:

```java
getAllLoansDirect()
```

3. Armazenar a lista retornada:

```java
List<Map<String, Object>> loans = system.getAllLoansDirect();
```

4. Modificar diretamente os dados:

```java
loans.clear();
```

ou

```java
loans.remove(0);
```

5. Executar novamente a listagem de empréstimos no sistema

---

## Comportamento Esperado

O sistema deveria:

- proteger a estrutura interna;
- impedir alterações externas;
- retornar apenas cópias dos dados;
- utilizar encapsulamento adequado;
- garantir integridade das informações.

### Exemplo esperado

```bash
Operação não permitida.
```

ou retorno imutável da coleção.

---

## Comportamento Observado

A lista original do banco é retornada diretamente.

Alterações feitas externamente afetam imediatamente os dados internos do sistema.

### Exemplo observado

```java
List<Map<String, Object>> loans = system.getAllLoansDirect();

loans.clear();
```

Após isso:

```bash
Loans list empty
```

Mesmo sem utilizar nenhuma função oficial de remoção.

Isso permite que qualquer parte do sistema altere os dados internos sem controle.

---

## Ambiente de Execução

| Item | Informação |
|---|---|
| Linguagem | Java |
| JDK | JDK 17 |
| Sistema Operacional | Windows 11 |
| IDE | Eclipse / IntelliJ |
| Tipo da aplicação | CLI (Console) |

---

## Evidências da Ocorrência

### Código utilizado

```java
List<Map<String, Object>> loans = system.getAllLoansDirect();

loans.clear();
```

### Resultado observado

```bash
Loans list empty
```

### Evidência técnica

```java
return LegacyDatabase.getLoans();
```

Sem:

- cópia defensiva;
- proteção imutável;
- encapsulamento.

---

# ISSUE #3 – Contagem Incorreta de Empréstimos Abertos por Livro

## Título
Método `countOpenLoansByBook()` utiliza `userId` em vez de `bookId`, gerando contagem incorreta de empréstimos

---

## Descrição Detalhada

O método `countOpenLoansByBook(int bookId)` possui um erro lógico na filtragem dos empréstimos abertos.

Durante a contagem, o sistema compara o campo:

```java
loan.get("userId")
```

quando o correto seria:

```java
loan.get("bookId")
```

Isso faz com que a busca utilize o identificador do usuário em vez do identificador do livro, retornando valores inconsistentes.

### Trecho identificado

```java
public static int countOpenLoansByBook(int bookId) {
    int c = 0;

    for (Map<String, Object> loan : loans) {

        if (((Integer) loan.get("userId")).intValue() == bookId) {

            if ("OPEN".equals(String.valueOf(loan.get("status")))) {
                c++;
            }
        }
    }

    return c;
}
```

---

## Passos para Reproduzir

1. Executar a aplicação `LibrarySystem`

2. Registrar um empréstimo para um livro:

```bash
Book ID: 1
User ID: 2
```

3. Criar múltiplos empréstimos com usuários diferentes

4. Executar o método:

```java
countOpenLoansByBook(1)
```

5. Verificar o valor retornado

---

## Comportamento Esperado

O sistema deveria:

- contar corretamente os empréstimos abertos de um livro;
- utilizar o campo `bookId` como filtro;
- retornar a quantidade correta de empréstimos ativos.

### Exemplo esperado

```bash
Livro ID 1 possui 2 empréstimos abertos.
```

---

## Comportamento Observado

O sistema utiliza o campo `userId` na comparação:

```java
loan.get("userId")
```

Isso faz com que a contagem dependa do ID do usuário em vez do ID do livro.

### Resultado observado

```bash
Livro ID 1 possui 0 empréstimos abertos.
```

Mesmo existindo empréstimos ativos registrados para o livro.

Em alguns cenários, o sistema retorna valores incorretos ou inconsistentes.

---

## Ambiente de Execução

| Item | Informação |
|---|---|
| Linguagem | Java |
| JDK | JDK 17 |
| Sistema Operacional | Windows 11 |
| IDE | Eclipse / IntelliJ |
| Tipo da aplicação | CLI (Console) |

---

## Evidências da Ocorrência

### Código com erro

```java
if (((Integer) loan.get("userId")).intValue() == bookId)
```

### Código correto esperado

```java
if (((Integer) loan.get("bookId")).intValue() == bookId)
```


# ISSUE #4 – Método unsafeUpdateBookField Permite Alterações Inseguras nos Dados

## Título
Método `unsafeUpdateBookField()` permite modificação arbitrária dos campos internos dos livros

---

## Descrição Detalhada

O sistema possui um método chamado `unsafeUpdateBookField()` responsável por alterar campos de livros diretamente na estrutura interna do banco em memória.

O problema ocorre porque o método permite alterar qualquer atributo sem validação, restrição ou controle de integridade.

### Trecho identificado

```java
public static void unsafeUpdateBookField(int id, String field, Object value) {

    Map<String, Object> b = books.get(id);

    if (b != null) {

        b.put(field, value);

        logs.add("book-updated-" + id + "-" + field);
    }
}
```

Como o campo é recebido dinamicamente através da variável:

```java
field
```

qualquer propriedade pode ser alterada, inclusive campos críticos do sistema.

Isso permite:
- inserção de dados inválidos;
- corrupção da estrutura dos livros;
- inconsistência de dados;
- alterações sem validação;
- falhas futuras na aplicação.

---

## Passos para Reproduzir

1. Executar a aplicação `LibrarySystem`

2. Criar ou carregar livros no sistema

3. Executar o método:

```java
LegacyDatabase.unsafeUpdateBookField(
    1,
    "availableCopies",
    -999
);
```

4. Verificar os dados do livro

ou

```java
LegacyDatabase.unsafeUpdateBookField(
    1,
    "id",
    null
);
```

5. Executar funcionalidades relacionadas aos livros

---

## Comportamento Esperado

O sistema deveria:

- validar os campos permitidos;
- impedir alterações críticas;
- bloquear valores inválidos;
- proteger atributos internos;
- utilizar setters controlados;
- garantir integridade dos dados.

### Exemplo esperado

```bash
Erro: valor inválido para availableCopies.
```

ou

```bash
Campo protegido: id não pode ser alterado.
```

---

## Comportamento Observado

O sistema altera qualquer campo diretamente sem validação.

### Exemplo observado

```java
LegacyDatabase.unsafeUpdateBookField(
    1,
    "availableCopies",
    -999
);
```

Resultado:

```bash
Livro atualizado com sucesso
```

Mesmo utilizando um valor inválido.

Outro exemplo:

```java
LegacyDatabase.unsafeUpdateBookField(
    1,
    "id",
    null
);
```

O sistema aceita a alteração, causando possível corrupção da estrutura interna.

---

## Ambiente de Execução

| Item | Informação |
|---|---|
| Linguagem | Java |
| JDK | JDK 17 |
| Sistema Operacional | Windows 11 |
| IDE | Eclipse / IntelliJ |
| Tipo da aplicação | CLI (Console) |

---

## Evidências da Ocorrência

### Código executado

```java
LegacyDatabase.unsafeUpdateBookField(
    1,
    "availableCopies",
    -999
);
```

### Resultado observado

```bash
book-updated-1-availableCopies
```

---

### Segundo cenário reproduzido

```java
LegacyDatabase.unsafeUpdateBookField(
    1,
    "id",
    null
);
```

### Resultado observado

```bash
book-updated-1-id
```

---


# ISSUE #5 – Empréstimo Duplicado ao Utilizar Canal SMS

## Título
Método `borrowBook()` cria empréstimos duplicados quando o canal selecionado é `"sms"`

---

## Descrição Detalhada

O método `borrowBook()` possui uma lógica adicional para integração com notificações SMS.

Durante o processo de empréstimo, o sistema executa novamente o método:

```java
LegacyDatabase.addLoanData()
```

quando o canal informado é `"sms"`.

Isso faz com que o sistema registre dois empréstimos abertos para a mesma operação.

### Trecho identificado

```java
if ("sms".equals(channel)) {

    LegacyDatabase.addLoanData(
        bookId,
        userId,
        borrowDate,
        dueDate,
        "",
        "OPEN",
        0.0,
        "loan-created-sync"
    );
}
```

O comentário presente no código já indica o problema:

```java
// BUG (state): duplicate open loan for SMS channel.
```

---

## Passos para Reproduzir

1. Executar a aplicação `LibrarySystem`

2. Selecionar a opção de empréstimo de livro

3. Informar dados válidos:

```bash
User ID: 1
Book ID: 1
```

4. Selecionar o canal:

```bash
sms
```

5. Confirmar a operação

6. Executar a listagem de empréstimos:

```java
listAllLoans()
```

---

## Comportamento Esperado

O sistema deveria:

- criar apenas um empréstimo;
- registrar somente uma entrada no banco;
- manter consistência dos dados;
- controlar corretamente a disponibilidade do livro.

### Exemplo esperado

```bash
Loan created with id 1
```

Apenas um registro deveria existir na lista de empréstimos.

---

## Comportamento Observado

O sistema cria dois empréstimos para a mesma operação quando o canal `"sms"` é utilizado.

### Resultado observado

```bash
Loan created with id 1
```

Porém, ao listar os empréstimos:

```bash
ID | USER | BOOK | STATUS

1 | 1 | 1 | OPEN
2 | 1 | 1 | OPEN
```

Dois registros são criados indevidamente.

---

## Ambiente de Execução

| Item | Informação |
|---|---|
| Linguagem | Java |
| JDK | JDK 17 |
| Sistema Operacional | Windows 11 |
| IDE | Eclipse / IntelliJ |
| Tipo da aplicação | CLI (Console) |

---


# TRIAGEM TÉCNICA

## Classificação do Tipo de Falha

ISSUE #1 – Tratamento Genérico de Exception	| Bug |	O erro já afeta diretamente o usuário durante o uso do sistema
ISSUE #2 – Quebra de Encapsulamento	| Defect |	Problema estrutural identificado internamente antes de falha visível ao usuário
ISSUE #3 – Contagem incorreta de empréstimos	| Bug |	O sistema apresenta comportamento incorreto ao usuário
ISSUE #4 – unsafeUpdateBookField	| Defect |	Falha interna de segurança e integridade identificada no código
ISSUE #5 – Empréstimo duplicado via SMS	| Bug |	O erro gera duplicidade real de empréstimos no sistema

## Verificação de Bugs Inválidos

Após análise técnica, nenhuma issue foi considerada inválida.

Todas as issues:

puderam ser reproduzidas;
representam falhas reais;
possuem impacto funcional;
afetam integridade, manutenção ou comportamento do sistema.


## Classificação de Severidade

ISSUE #1	| Normal |	O sistema continua funcionando, mas dificulta manutenção e diagnóstico
ISSUE #2	| Serious |	Permite alteração indevida da estrutura interna do sistema
ISSUE #3	| Serious |	Contagem incorreta afeta lógica principal de empréstimos
ISSUE #4	| Catastrophic |	Permite corrupção direta dos dados internos
ISSUE #5	| Catastrophic |	Cria duplicidade real de empréstimos e inconsistência geral


## Classificação de Prioridade

ISSUE #1	Média	Importante para manutenção futura
ISSUE #2	Alta	Compromete encapsulamento e segurança dos dados
ISSUE #3	Alta	Afeta regra principal do sistema
ISSUE #4	Alta	Pode corromper completamente os dados
ISSUE #5	Alta	Afeta diretamente operações críticas de empréstimo