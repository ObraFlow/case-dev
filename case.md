# Desafio Técnico — ObraFlow

## Contexto

O **ObraFlow** é uma plataforma para empresas do setor de construção civil.

Diferentes empresas utilizam a plataforma e possuem seus próprios dados. Por isso, um dos conceitos importantes do sistema é o **multi-tenancy**: os dados de uma empresa nunca devem ser acessíveis por outra.

Neste desafio, você deverá desenvolver uma pequena aplicação para:

1. Criar tenants;
2. Cadastrar obras dentro de um tenant;
3. Consultar informações das obras através de perguntas em linguagem natural;
4. Garantir o isolamento entre tenants.

Você não precisa conhecer o ObraFlow para realizar o desafio.

---

# Tecnologias

Utilize obrigatoriamente:

* **Python 3.14**
* **FastAPI ou FastMCP**
* **Pydantic**
* **pytest**
* **Docker**
* **Docker Compose**

Para armazenamento dos dados, você pode utilizar a solução que considerar adequada.

Exemplos:

* SQLite
* PostgreSQL
* JSON
* outro armazenamento simples

Não é necessário utilizar serviços externos.

---

# 1. Tenants

Um tenant representa uma empresa utilizando o ObraFlow.

A aplicação deve permitir criar tenants.

Exemplo:

```http
POST /tenants
```

Request:

```json
{
  "name": "Construtora Nova",
  "slug": "construtora-nova"
}
```

Response:

```json
{
  "id": "construtora-nova",
  "name": "Construtora Nova"
}
```

O `slug` deve identificar o tenant na aplicação.

Exemplos:

```text
construtora-a
construtora-b
construtora-nova
```

O sistema deve impedir a criação de dois tenants com o mesmo identificador.

---

# 2. Obras

Cada obra pertence a um tenant.

Uma obra deve possuir, no mínimo:

```json
{
  "name": "Portobelo",
  "status": "Em andamento",
  "delivery_date": "2027-10-15",
  "units": 240,
  "units_sold": 187,
  "responsible_engineer": "João Silva"
}
```

Você pode adicionar outros campos se considerar necessário.

Crie pelo menos **2 tenants**, cada um com pelo menos **2 obras**, para demonstrar o isolamento dos dados.

## Cadastro

Exemplo:

```http
POST /tenants/construtora-a/projects
```

Request:

```json
{
  "name": "Portobelo",
  "status": "Em andamento",
  "delivery_date": "2027-10-15",
  "units": 240,
  "units_sold": 187,
  "responsible_engineer": "João Silva"
}
```

A obra deverá pertencer ao tenant informado.

---

# 3. Consultas

A aplicação deve permitir fazer perguntas sobre as obras de um tenant.

Endpoint sugerido:

```http
POST /query
```

Request:

```json
{
  "tenant": "construtora-a",
  "question": "Quantas unidades já foram vendidas?"
}
```

Response:

```json
{
  "answer": "Foram vendidas 187 das 240 unidades."
}
```

A implementação da interpretação da pergunta fica a seu critério.

Você pode utilizar regras, palavras-chave ou outra abordagem.

**Não é necessário utilizar uma LLM.**

---

## Exemplos de perguntas

A aplicação deve conseguir responder perguntas como:

```text
Qual o status da obra Portobelo?
```

```text
Quantas unidades a Portobelo possui?
```

```text
Quantas unidades já foram vendidas?
```

```text
Quantas unidades ainda estão disponíveis?
```

```text
Quem é o engenheiro responsável?
```

Você pode definir outras perguntas suportadas.

---

# 4. Multi-tenancy

Este é um dos requisitos mais importantes do desafio.

Os dados devem ser isolados entre tenants.

Por exemplo:

```text
construtora-a
├── Portobelo
└── Alphaville

construtora-b
├── Reserva do Parque
└── Jardim Central
```

Uma consulta feita para:

```text
construtora-a
```

não pode retornar informações de:

```text
construtora-b
```

O sistema deve garantir esse isolamento independentemente da forma escolhida para armazenar os dados.

---

# 5. Just-in-time

Ao criar um tenant, ele deve estar imediatamente pronto para utilização.

O fluxo esperado é:

```text
POST /tenants
      ↓
Tenant criado
      ↓
Pode cadastrar obras
      ↓
Pode realizar consultas
```

Não deve ser necessário alterar código ou executar configurações manuais para que um novo tenant funcione.

Para este desafio, o provisionamento pode ser simples.

Não é necessário criar bancos, containers ou outros recursos de infraestrutura separados para cada tenant.

Queremos apenas avaliar o conceito de **criação e preparação automática de um novo tenant**.

---

# 6. Validações e erros

A aplicação deve tratar situações como:

### Tenant inexistente

```json
{
  "tenant": "empresa-inexistente",
  "question": "Qual o status da obra?"
}
```

### Obra inexistente

```text
Qual o status da obra Hogwarts?
```

### Pergunta não suportada

```text
Qual será o clima amanhã?
```

### Dados inválidos

Por exemplo, uma requisição de criação de tenant sem `name` ou `slug`.

Os erros devem possuir respostas HTTP adequadas e mensagens compreensíveis.

---

# 7. Testes

Utilize **pytest** para testar a aplicação.

Não esperamos cobertura de 100%.

Os testes devem demonstrar pelo menos:

* criação de tenant;
* criação de obra;
* consultas;
* isolamento entre tenants;
* validação de dados;
* tratamento de recursos inexistentes.

Um dos testes mais importantes deve garantir que:

```text
Tenant A → acessa dados de A
Tenant A → NÃO acessa dados de B
```

---

# 8. Docker

A aplicação deve ser executada utilizando **Docker Compose**.

O projeto deve possuir:

```text
Dockerfile
docker-compose.yml
```

O objetivo é que uma pessoa consiga iniciar a aplicação com:

```bash
docker compose up --build
```

e tenha tudo que é necessário para executar o projeto.

Não deve ser necessário instalar Python ou outras dependências da aplicação diretamente na máquina.

Caso sejam utilizados serviços adicionais, como um banco de dados, eles também devem ser configurados pelo `docker-compose.yml`.

---

# 9. Front-end

O front-end é **opcional do ponto de vista do requisito mínimo**, mas será considerado um importante diferencial na avaliação.

A ideia é observar não apenas a capacidade de seguir requisitos, mas também **iniciativa e proatividade para pensar no produto como um todo**.

Uma implementação esperada poderia permitir:

### Criar tenant

```text
┌──────────────────────────────┐
│       Novo Tenant            │
├──────────────────────────────┤
│ Nome:                        │
│ [ Construtora Nova        ]  │
│                              │
│ Slug:                        │
│ [ construtora-nova        ]  │
│                              │
│       [ Criar tenant ]       │
└──────────────────────────────┘
```

### Cadastrar obra

Depois de selecionar um tenant:

```text
┌──────────────────────────────┐
│        Nova Obra              │
├──────────────────────────────┤
│ Nome:                        │
│ [ Portobelo               ]  │
│                              │
│ Status:                      │
│ [ Em andamento            ]  │
│                              │
│ Data de entrega:             │
│ [ 15/10/2027              ]  │
│                              │
│ Unidades:                    │
│ [ 240                    ]   │
│                              │
│ Unidades vendidas:           │
│ [ 187                    ]   │
│                              │
│ Engenheiro responsável:      │
│ [ João Silva             ]   │
│                              │
│       [ Cadastrar obra ]     │
└──────────────────────────────┘
```

### Fazer perguntas

```text
Tenant:
[ Construtora Nova ▼ ]

Pergunta:

┌─────────────────────────────────────────┐
│ Quantas unidades já foram vendidas?     │
└─────────────────────────────────────────┘

              [ Perguntar ]
```

Resposta:

```text
Foram vendidas 187 das 240 unidades.
```

O front-end não precisa ser visualmente sofisticado.

Uma interface simples, funcional e bem integrada com a API já é suficiente.

Você pode utilizar a tecnologia de front-end que preferir.

---

# 10. Proatividade

O requisito mínimo do desafio é o back-end descrito acima.

Porém, durante a avaliação, também observaremos a capacidade de **ir além do requisito mínimo**.

Alguns exemplos:

* desenvolvimento do front-end;
* melhoria da experiência de utilização;
* validações adicionais;
* documentação além do mínimo;
* logs;
* tratamento de casos extremos;
* melhorias de arquitetura;
* preocupação com qualidade de código;
* automação do ambiente de desenvolvimento.

Não é necessário implementar todos esses itens.

**Não queremos uma aplicação cheia de funcionalidades sem necessidade.**

Queremos entender se você consegue identificar oportunidades de melhoria e tomar iniciativa.

---

# 11. Organização

Não existe uma estrutura obrigatória.

Organize o projeto de maneira que o código seja fácil de entender e manter.

Por exemplo:

```text
.
├── app/
│   ├── main.py
│   ├── models/
│   ├── services/
│   └── ...
├── tests/
├── Dockerfile
├── docker-compose.yml
├── README.md
└── pyproject.toml
```

Essa é apenas uma sugestão.

---

# 12. Documentação

O projeto deve possuir um README explicando:

* como executar o projeto;
* como executar os testes;
* endpoints disponíveis;
* exemplos de utilização;
* principais decisões técnicas.

Inclua também uma seção:

```markdown
## Decisões técnicas
```

Explique brevemente:

* como os tenants são armazenados;
* como as obras são relacionadas aos tenants;
* como as perguntas são interpretadas;
* como o isolamento é garantido;
* o que você mudaria em uma versão de produção.

---

# O que estamos avaliando

Os principais critérios são:

| Critério                 | Peso |
| ------------------------ | ---: |
| Funcionamento            |  25% |
| Qualidade do código      |  20% |
| Raciocínio e arquitetura |  15% |
| Multi-tenancy            |  15% |
| Testes                   |  10% |
| Tratamento de erros      |  10% |
| Documentação             |   5% |

Além desses critérios, **iniciativa e proatividade serão consideradas durante a avaliação**.

O front-end não é necessário para que a solução mínima esteja completa, mas uma implementação funcional demonstra capacidade de enxergar o problema além do requisito básico.

---

# Uso de IA

Você pode utilizar ferramentas de IA durante o desenvolvimento.

Porém, durante a entrevista técnica, vamos conversar sobre o código entregue.

Você deverá conseguir explicar:

* como a solução funciona;
* por que tomou determinadas decisões;
* quais são as limitações;
* o que faria diferente em produção.

**O uso de IA é permitido. O código entregue deve ser compreendido por você.**

---

# Tempo

Esperamos que o desafio possa ser realizado em aproximadamente:

**3 a 5 horas.**

Não esperamos uma solução perfeita ou pronta para produção.

Uma solução **simples, funcional e bem estruturada** é preferível a uma solução complexa e incompleta.

---

# Entrega

Envie o link para um repositório Git contendo o projeto.

O projeto deve conseguir ser executado seguindo as instruções do README.

O primeiro comando para executar o projeto deve ser, idealmente:

```bash
docker compose up --build
```

Boa sorte! 🚀
