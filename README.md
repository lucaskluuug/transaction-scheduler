# transaction-scheduler
Transaction scheduler simulator using timestamps for a DBMS

# Relatório - Simulador de Escalonador de Transações com Protocolo de Timestamps

## 1. Descrição do Trabalho Realizado

Este trabalho implementa um simulador de escalonador de transações que utiliza o protocolo de timestamps (marcadores de tempo) para controlar a execução concorrente de transações. A implementação foi desenvolvida em JavaScript e executada em um navegador web, fornecendo uma interface simples para entrada e visualização dos resultados.

O simulador processa uma História Inicial (HI) de operações de transações, aplicando as regras do protocolo de timestamps para determinar quais operações podem ser executadas, quais transações devem ser abortadas, e como reorganizar a execução quando ocorrem aborts.

---

## 2. Características Implementadas

### 2.1 Funcionalidades Principais

 **Protocolo de Timestamps Completo**
- Atribuição automática de timestamps quando a primeira operação de uma transação chega ao escalonador
- Validação de leituras: verifica se `TS(Ti) < WTS(X)` para abortar leituras tardias
- Validação de escritas: verifica se `TS(Ti) < RTS(X)` ou `TS(Ti) < WTS(X)` para abortar escritas tardias
- Atualização automática de RTS (Read Timestamp) e WTS (Write Timestamp) após operações válidas

 **Gerenciamento de Transações**
- Criação automática de transações quando a primeira operação chega
- Estados de transação: `ativa`, `abortada`, `comitada`
- Memória local para cada transação (permite leituras repetidas da mesma variável sem acessar o banco)
- Rastreamento de operações executadas por cada transação

 **Sistema de Abort e Reinício**
- Detecção automática de violações do protocolo de timestamps
- Remoção de todas as operações da transação abortada da História Final (HF)
- Reinício automático de transações abortadas com novo timestamp
- Contagem de reinícios por transação
- Reinserção das operações da transação abortada no final da fila de processamento

 **Banco de Dados Simulado**
- Variáveis criadas dinamicamente conforme necessário
- Valores iniciais padrão de 0 para variáveis não inicializadas
- Atualização do banco apenas no momento do commit
- Suporte a expressões matemáticas nas escritas (ex: `(A + B)`, `(X * 2)`, etc.)

 **Validação de Expressões**
- Verificação de timestamps das variáveis usadas em expressões matemáticas
- Cálculo dinâmico de expressões usando valores da memória local da transação
- Tratamento de erros em expressões inválidas

 **Exibição Detalhada**
- Construção passo a passo da História Final
- Log detalhado de cada operação processada
- Informações sobre valores lidos/calculados e origem dos dados
- Exibição de timestamps (TS, RTS, WTS) em cada etapa
- Mensagens claras sobre aborts e motivos
- Estado final do banco de dados
- Informações sobre transações comitadas e abortadas

 **Interface Web**
- Interface HTML simples e funcional
- Área de texto para entrada da História Inicial
- Área de visualização dos resultados
- Exemplo pré-carregado para facilitar testes


---

## 3. Formato de Entrada dos Dados

A entrada deve ser fornecida como uma **História Inicial (HI)** no formato texto, onde cada linha representa uma operação de uma transação.

### 3.1 Formato das Operações

Cada linha segue o padrão:
```
<identificador_transação> <tipo_operação> [variável] [valor/expressão]
```

Onde:
- **`<identificador_transação>`**: Identificador único da transação (ex: `t0`, `t1`, `t2`)
- **`<tipo_operação>`**: Tipo de operação:
  - `r` - Leitura (read)
  - `w` - Escrita (write)
  - `c` - Commit
- **`[variável]`**: Nome da variável (obrigatório para `r` e `w`)
- **`[valor/expressão]`**: 
  - Para `w`: valor numérico ou expressão matemática (ex: `100`, `(A + B)`, `(X * 2 + 10)`)
  - Não aplicável para `r` e `c`

### 3.2 Exemplos de Entrada

**Exemplo 1 - Operações Básicas:**
```
t0 w A 100
t0 w B 200
t0 c
t1 r A
t1 w C (A + 50)
t1 c
```

**Exemplo 2 - Múltiplas Transações Concorrentes:**
```
t1 r X
t2 r X
t1 w X (X + 10)
t2 w X (X + 20)
t1 c
t2 c
```

**Exemplo 3 - Expressões Complexas:**
```
t0 w VAL1 10
t0 w VAL2 20
t0 c
t1 r VAL1
t1 r VAL2
t1 w SOMA (VAL1 + VAL2 + 100)
t1 c
```

### 3.3 Regras de Formatação

- Uma operação por linha
- Campos separados por espaços em branco
- Linhas vazias são ignoradas
- Expressões matemáticas podem usar parênteses e operadores básicos (`+`, `-`, `*`, `/`)
- Nomes de variáveis devem começar com letra e podem conter letras, números e underscore
- Não há limite de caracteres por linha

---

## 4. Formato de Saída dos Dados

A saída é gerada em formato texto e contém várias seções detalhadas:

### 4.1 Estrutura da Saída

**1. Cabeçalho do Processamento**
```
=== INICIANDO ESCALONAMENTO ===
Total de operações na HI: <número>
```

**2. Log Passo a Passo**
Para cada operação processada, são exibidas informações detalhadas:

- **Nova Transação:**
  ```
  [NOVA TRANSAÇÃO] <id> iniciada [ (REINÍCIO após abort)]
    TS(<id>) = <timestamp>
  ```

- **Leitura:**
  ```
  [LEITURA] <id> r <variável>
    Valor lido: <valor> (banco de dados | memória local)
    RTS(<variável>) = <timestamp>
  ```
  Ou em caso de abort:
  ```
  [LEITURA] <id> r <variável>
    ABORT: TS(<id>)=<ts> < WTS(<variável>)=<wts>
    Motivo: Leitura tardia detectada!
  ```

- **Escrita:**
  ```
  [ESCRITA] <id> w <variável> <expressão>
    Valor calculado: <valor>
    WTS(<variável>) = <timestamp>
  ```
  Ou em caso de abort:
  ```
  [ESCRITA] <id> w <variável> <expressão>
    ABORT: TS(<id>)=<ts> < RTS(<variável>)=<rts>
    Motivo: Escrita tardia (após leitura)!
  ```

- **Commit:**
  ```
  [COMMIT] <id>
    Transação finalizada com sucesso
  ```

- **Reinício após Abort:**
  ```
  [REINÍCIO] Transação <id> será reiniciada
    Operações removidas da HF
    Todas as <número> operações adicionadas ao final da fila
  ```

**3. História Final (HF)**
```
=== HISTÓRIA FINAL (HF) ===
<id> r <variável>
<id> w <variável> <expressão>
<id> c
...
```

**4. Estado Final do Banco de Dados**
```
=== ESTADO FINAL DO BANCO DE DADOS ===
<variável> = <valor>
...
```

**5. Informações das Transações**
```
=== INFORMAÇÕES DAS TRANSAÇÕES ===
<id>: <estado> (TS=<timestamp>)
...
```

**6. Transações que Abortaram (se houver)**
```
=== TRANSAÇÕES QUE ABORTARAM ===
<id> - <número> reinício(s)
...
```

### 4.2 Exemplo de Saída Completa

```
=== INICIANDO ESCALONAMENTO ===
Total de operações na HI: 6

[NOVA TRANSAÇÃO] t1 iniciada
  TS(t1) = 0

[LEITURA] t1 r X
  Valor lido: 0 (banco de dados)
  RTS(X) = 0

[NOVA TRANSAÇÃO] t2 iniciada
  TS(t2) = 1

[LEITURA] t2 r X
  Valor lido: 0 (banco de dados)
  RTS(X) = 1

[ESCRITA] t1 w X (X + 10)
  ABORT: TS(t1)=0 < RTS(X)=1
  Motivo: Escrita tardia (após leitura)!

[REINÍCIO] Transação t1 será reiniciada
  Operações removidas da HF
  Todas as 3 operações adicionadas ao final da fila

[ESCRITA] t2 w X (X + 20)
  Valor calculado: 20
  WTS(X) = 1

[COMMIT] t2
  Transação finalizada com sucesso

[NOVA TRANSAÇÃO] t1 iniciada (REINÍCIO após abort)
  TS(t1) = 2

[LEITURA] t1 r X
  Valor lido: 20 (banco de dados)
  RTS(X) = 2

[ESCRITA] t1 w X (X + 10)
  Valor calculado: 30
  WTS(X) = 2

[COMMIT] t1
  Transação finalizada com sucesso


=== HISTÓRIA FINAL (HF) ===
t2 r X
t2 w X (X + 20)
t2 c
t1 r X
t1 w X (X + 10)
t1 c

=== ESTADO FINAL DO BANCO DE DADOS ===
X = 30

=== INFORMAÇÕES DAS TRANSAÇÕES ===
t2: comitada (TS=1)
t1: comitada (TS=2)

=== TRANSAÇÕES QUE ABORTARAM ===
t1 - 1 reinício(s)
```

---

## 5. Estrutura de Dados

### 5.1 Armazenamento de Transações

**Estrutura:** `transacoes` (objeto JavaScript)
```javascript
transacoes = {
  "<id_transacao>": {
    estado: "ativa" | "abortada" | "comitada",
    timestamp: <número>,
    memoria: {
      "<variavel>": <valor>
    },
    operacoes: [
      { op: "r" | "w", varName: "<nome>", valor: <número>, expressao?: "<expressão>" }
    ]
  }
}
```

**Campos:**
- `estado`: Estado atual da transação
- `timestamp`: Timestamp atribuído à transação
- `memoria`: Memória local da transação (valores lidos/calculados)
- `operacoes`: Lista de operações já executadas pela transação

### 5.2 Armazenamento das Histórias

**História Inicial (HI):**
- **Estrutura:** Array de objetos
```javascript
linhas = [
  { t: "<id>", op: "r" | "w" | "c", varName: "<nome>", val: "<valor/expressão>" }
]
```

**História Final (HF):**
- **Estrutura:** Array de objetos
```javascript
historiaFinal = [
  { t: "<id>", op: "r", varName: "<nome>", valor: <número> },
  { t: "<id>", op: "w", varName: "<nome>", valor: <número>, expressao: "<expressão>" },
  { t: "<id>", op: "c" }
]
```

**Fila de Operações Pendentes:**
- **Estrutura:** Array de objetos (mesmo formato da HI)
- Usado para gerenciar operações ainda não processadas e reinserir transações abortadas

### 5.3 Controle de Execução

**Timestamps de Variáveis:**
```javascript
writeTimestamps = {
  "<variavel>": <timestamp>  // WTS(X) - maior timestamp de escrita em X
}

readTimestamps = {
  "<variavel>": <timestamp>  // RTS(X) - maior timestamp de leitura em X
}
```

**Controle de Transações Abortadas:**
```javascript
transacoesAbortadas = Set("<id1>", "<id2>", ...)  // IDs de transações que abortaram

transacoesReiniciadas = {
  "<id>": <número_de_reinicios>
}
```

**Contador de Tempo:**
```javascript
tempo = <número>  // Contador global para atribuir timestamps sequenciais
```

### 5.4 Banco de Dados Simulado

**Estrutura:** `db` (objeto JavaScript)
```javascript
db = {
  "<variavel>": <valor_numerico>
}
```

- Variáveis são criadas dinamicamente quando acessadas pela primeira vez
- Valores iniciais padrão: `0`
- Atualizado apenas no momento do commit de uma transação
- Armazena apenas valores numéricos

### 5.5 Estrutura de Resultado

**Estrutura:** Array de strings
```javascript
resultado = [
  "=== INICIANDO ESCALONAMENTO ===\n",
  "Total de operações na HI: 6\n\n",
  "[NOVA TRANSAÇÃO] t1 iniciada\n",
  ...
]
```

- Acumula todas as mensagens de log durante o processamento
- Convertido para string única no final com `resultado.join('')`

---

## 6. Algoritmo de Processamento

### 6.1 Fluxo Principal

1. **Parse da HI**: Converte texto em array de objetos estruturados
2. **Inicialização**: Cria estruturas de dados vazias
3. **Loop Principal**: Processa cada operação da fila pendente:
   - Verifica se transação existe, cria se necessário
   - Valida estado da transação
   - Processa operação conforme tipo (`r`, `w`, `c`)
   - Aplica regras do protocolo de timestamps
   - Atualiza estruturas de controle
   - Em caso de abort: remove da HF, reinsere na fila
4. **Geração de Resultado**: Formata e exibe todas as seções

### 6.2 Regras do Protocolo Aplicadas

**Leitura (`r`):**
- Se `TS(Ti) < WTS(X)` → **ABORT**
- Caso contrário: executa, atualiza `RTS(X) = max(RTS(X), TS(Ti))`

**Escrita (`w`):**
- Se `TS(Ti) < RTS(X)` → **ABORT**
- Se `TS(Ti) < WTS(X)` → **ABORT**
- Se variáveis na expressão têm `WTS > TS(Ti)` → **ABORT**
- Caso contrário: executa, atualiza `WTS(X) = max(WTS(X), TS(Ti))`

**Commit (`c`):**
- Escreve todas as operações de escrita no banco de dados
- Marca transação como comitada

---

## 7. Observações Técnicas

### 7.1 Limitações e Proteções

- **Limite de Tentativas**: Implementado limite de 1000 tentativas para evitar loops infinitos
- **Memória Local**: Transações leem da memória local se a variável já foi lida/escrita anteriormente
- **Expressões Dinâmicas**: Uso de `new Function()` para avaliar expressões matemáticas dinamicamente
- **Reinício Automático**: Transações abortadas são automaticamente reinseridas no final da fila com novo timestamp

### 7.2 Tratamento de Erros

- Validação de valores ausentes em escritas
- Tratamento de erros em expressões matemáticas inválidas
- Verificação de estados inconsistentes de transações
- Mensagens de erro descritivas para facilitar debug

---

## 8. Conclusão

A implementação atende completamente aos requisitos especificados, fornecendo:

-  Protocolo de timestamps completo e funcional
-  Sistema robusto de abort e reinício de transações
-  Exibição detalhada passo a passo do escalonamento
-  Interface web simples e funcional
-  Suporte a expressões matemáticas complexas
-  Estruturas de dados bem organizadas e eficientes

O simulador está pronto para uso e pode processar histórias iniciais complexas com múltiplas transações concorrentes, detectando e resolvendo conflitos através do protocolo de timestamps.
