# Módulo 06 — Arquiteturas de Software

## O que você vai aprender

- Por que arquitetura importa
- Arquitetura em Camadas (Layered)
- MVC (Model-View-Controller)
- Hexagonal / Ports and Adapters
- Qual usar em cada situação
- Padrões de projeto essenciais

---

## 6.1 Por que Arquitetura Importa?

Código sem arquitetura = casa sem planta.  
No começo funciona. Com o tempo, vira um caos impossível de manter.

**Problemas de código desorganizado:**
- Mudar uma coisa quebra outra
- Impossível testar unitariamente
- Ninguém consegue entender o código dos outros
- Duplicação de lógica por todo lado

**O que boa arquitetura resolve:**
- **Separação de responsabilidades** — cada coisa no seu lugar
- **Testabilidade** — você consegue testar cada parte isolada
- **Manutenibilidade** — fácil de mudar, fácil de entender
- **Escalabilidade** — adicionar features sem medo

---

## 6.2 Arquitetura em Camadas (Layered Architecture)

A mais comum e mais fácil de entender. Ideal para sistemas CRUD e APIs.

```
┌─────────────────────────────────────┐
│          Controller / API           │  ← Recebe requisições HTTP
│         (Camada de Apresentação)    │
├─────────────────────────────────────┤
│            Service / UseCase        │  ← Regras de negócio
│         (Camada de Negócio)         │
├─────────────────────────────────────┤
│           Repository / DAO          │  ← Acesso a dados
│         (Camada de Dados)           │
├─────────────────────────────────────┤
│        Banco de Dados / API ext.    │  ← Infraestrutura
└─────────────────────────────────────┘
```

**Regra de ouro:** a dependência flui de cima para baixo.  
Controller → Service → Repository. Nunca o contrário.

### Exemplo prático — CRUD de Produto

```java
// ===== MODEL / ENTIDADE =====
// Representa os dados (geralmente mapeado ao banco)
public class Produto {
    private Long id;
    private String nome;
    private double preco;
    private int estoque;
    
    // construtores, getters, setters
}
```

```java
// ===== REPOSITORY (Camada de Dados) =====
// Só acessa banco. Não tem regra de negócio.
public interface ProdutoRepository {
    List<Produto> findAll();
    Optional<Produto> findById(Long id);
    Produto save(Produto produto);
    void deleteById(Long id);
    List<Produto> findByEstoqueLessThan(int quantidade);
}
```

```java
// ===== SERVICE (Camada de Negócio) =====
// Orquestra as regras. Chama o repository.
public class ProdutoService {
    private final ProdutoRepository repository;

    // Injeção via construtor (melhor prática)
    public ProdutoService(ProdutoRepository repository) {
        this.repository = repository;
    }

    public List<Produto> listarTodos() {
        return repository.findAll();
    }

    public Produto buscarPorId(Long id) {
        return repository.findById(id)
            .orElseThrow(() -> new RuntimeException("Produto não encontrado: " + id));
    }

    public Produto criar(Produto produto) {
        // REGRA DE NEGÓCIO: nome não pode estar em branco
        if (produto.getNome() == null || produto.getNome().isBlank()) {
            throw new IllegalArgumentException("Nome do produto é obrigatório");
        }
        // REGRA DE NEGÓCIO: preço deve ser positivo
        if (produto.getPreco() <= 0) {
            throw new IllegalArgumentException("Preço deve ser positivo");
        }
        return repository.save(produto);
    }

    public void baixarEstoque(Long produtoId, int quantidade) {
        Produto produto = buscarPorId(produtoId);
        // REGRA DE NEGÓCIO: não pode ficar negativo
        if (produto.getEstoque() < quantidade) {
            throw new IllegalStateException("Estoque insuficiente");
        }
        produto.setEstoque(produto.getEstoque() - quantidade);
        repository.save(produto);
    }

    public List<Produto> alertarEstoqueBaixo(int minimo) {
        return repository.findByEstoqueLessThan(minimo);
    }
}
```

```java
// ===== CONTROLLER (Camada de Apresentação) =====
// Recebe HTTP, chama service, retorna resposta. Sem lógica de negócio.
@RestController
@RequestMapping("/produtos")
public class ProdutoController {
    private final ProdutoService service;

    public ProdutoController(ProdutoService service) {
        this.service = service;
    }

    @GetMapping
    public List<Produto> listar() {
        return service.listarTodos();
    }

    @GetMapping("/{id}")
    public Produto buscar(@PathVariable Long id) {
        return service.buscarPorId(id);
    }

    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    public Produto criar(@RequestBody Produto produto) {
        return service.criar(produto);
    }
}
```

---

## 6.3 MVC — Model View Controller

Muito usado em aplicações web com frontend integrado (Thymeleaf, JSP).  
Similar à camada arquitetura, mas o Controller retorna uma View (página HTML).

```
REQUEST → Controller → Service → Repository → DB
                ↓
              Model
                ↓
              View (HTML) → RESPONSE
```

| Camada | Responsabilidade |
|--------|-----------------|
| **Model** | Dados e regras de negócio |
| **View** | Apresentação (HTML, JSON) |
| **Controller** | Orquestra Model e View |

---

## 6.4 Arquitetura Hexagonal (Ports and Adapters)

**Para sistemas mais complexos.** Isola completamente a lógica de negócio de frameworks e banco de dados.

```
       ┌──────────────────────────────────┐
       │                                  │
 HTTP  │  ┌────────────────────────────┐  │  Banco
  ──▶ Port│    DOMÍNIO (Negócio puro)  │Port▶ Oracle
 REST  │  │   Sem Spring, sem JPA      │  │   MySQL
       │  │   Sem framework nenhum     │  │
 CLI  ──▶ └────────────────────────────┘  │  Email
       │                                  │  Kafka
       └──────────────────────────────────┘
              Adapters (implementações)
```

**Vantagem:** seu domínio não depende de NADA externo. Trocar banco de dados = trocar um adapter. Testar sem banco = usar adapter em memória.

```
📁 src/
├── domain/              ← NUNCA tem import de Spring, JPA, etc.
│   ├── model/
│   │   └── Pedido.java
│   ├── port/            ← Interfaces (contratos)
│   │   ├── PedidoRepository.java  (output port)
│   │   └── PedidoService.java     (input port)
│   └── service/
│       └── PedidoServiceImpl.java ← Só usa as interfaces
│
├── infrastructure/      ← Implementações concretas
│   ├── persistence/
│   │   └── PedidoRepositoryJpa.java  ← Usa Spring Data JPA
│   └── web/
│       └── PedidoController.java     ← Usa Spring MVC
│
└── config/              ← Wiring (Spring Boot)
    └── BeanConfig.java
```

---

## 6.5 Qual Arquitetura Usar?

| Situação | Arquitetura Recomendada |
|----------|------------------------|
| CRUD simples, APIs básicas, MVP | **Layered (Camadas)** |
| Web app com templates HTML | **MVC** |
| Sistema complexo, múltiplas integrações | **Hexagonal** |
| Microserviços com eventos | **Event-Driven + Hexagonal** |

> **Para quem vem do PHP:** comece com Layered. É o mais próximo do que você já faz em PHP (Controller = arquivo de rota/action, Service = classe de regras de negócio, Repository = classe de consultas SQL).

---

## 6.6 Padrões de Projeto Essenciais

### Singleton — uma única instância

```java
public class Configuracao {
    private static Configuracao instancia;
    private final String urlBanco;

    private Configuracao() {
        this.urlBanco = System.getenv("DATABASE_URL");
    }

    public static synchronized Configuracao getInstance() {
        if (instancia == null) {
            instancia = new Configuracao();
        }
        return instancia;
    }

    public String getUrlBanco() { return urlBanco; }
}
// Uso: Configuracao.getInstance().getUrlBanco()
// No Spring: use @Bean + @Scope("singleton") — o framework gerencia
```

### Builder — construção fluente de objetos complexos

```java
public class Email {
    private final String para;
    private final String assunto;
    private final String corpo;
    private final List<String> cc;

    private Email(Builder builder) {
        this.para    = builder.para;
        this.assunto = builder.assunto;
        this.corpo   = builder.corpo;
        this.cc      = builder.cc;
    }

    public static class Builder {
        private String para;
        private String assunto;
        private String corpo;
        private List<String> cc = new ArrayList<>();

        public Builder para(String para) {
            this.para = para;
            return this;
        }
        public Builder assunto(String assunto) {
            this.assunto = assunto;
            return this;
        }
        public Builder corpo(String corpo) {
            this.corpo = corpo;
            return this;
        }
        public Builder cc(String email) {
            this.cc.add(email);
            return this;
        }
        public Email build() {
            Objects.requireNonNull(para, "Destinatário obrigatório");
            return new Email(this);
        }
    }
}

// Uso fluente e legível
Email email = new Email.Builder()
    .para("time@caoa.com.br")
    .assunto("Deploy realizado")
    .corpo("Sistema atualizado com sucesso!")
    .cc("gestor@caoa.com.br")
    .build();
// No Spring: Lombok @Builder gera isso automaticamente
```

### Strategy — algoritmos intercambiáveis

```java
// Interface da estratégia
public interface CalculadoraFrete {
    double calcular(double peso, double distancia);
}

// Implementações
public class FreteCorreios implements CalculadoraFrete {
    @Override
    public double calcular(double peso, double distancia) {
        return peso * 1.5 + distancia * 0.01;
    }
}

public class FreteFedex implements CalculadoraFrete {
    @Override
    public double calcular(double peso, double distancia) {
        return peso * 2.0 + distancia * 0.02 + 15.0;  // taxa fixa
    }
}

// Contexto que usa a estratégia
public class Pedido {
    private CalculadoraFrete calculadora;

    public void setCalculadora(CalculadoraFrete calculadora) {
        this.calculadora = calculadora;
    }

    public double calcularFrete(double peso, double distancia) {
        return calculadora.calcular(peso, distancia);
    }
}

// Uso — troca fácil sem if/else
Pedido pedido = new Pedido();
pedido.setCalculadora(new FreteCorreios());
System.out.println("Correios: R$ " + pedido.calcularFrete(5.0, 300.0));

pedido.setCalculadora(new FreteFedex());
System.out.println("FedEx: R$ " + pedido.calcularFrete(5.0, 300.0));

// Com Lambda (ainda mais simples!)
pedido.setCalculadora((peso, dist) -> peso * 1.2 + dist * 0.015);
```

---

## ✅ Pontos-chave para lembrar

1. **Layered** = mais simples, começa aqui
2. **Controller não tem if de negócio** — isso é trabalho do Service
3. **Repository não sabe de regra de negócio** — só faz CRUD
4. **Dependa de interfaces, não de implementações** — facilita testes e mudanças

---

## ✅ Exercícios

### Exercício 1 — Refatorar para Layered
Pegue este código desorganizado e refatore para a arquitetura Layered (Controller → Service → Repository):

```java
// Código bagunçado (tudo misturado)
public class ProdutoController {
    public Produto buscarProduto(long id) {
        // Validação misturada com acesso a dados
        if (id <= 0) throw new RuntimeException("ID inválido");
        // Lógica de negócio misturada
        List<Produto> todos = bancoDeDados.buscarTodos();
        return todos.stream().filter(p -> p.getId() == id).findFirst()
            .orElseThrow(() -> new RuntimeException("Não encontrado"));
    }
}
```

### Exercício 2 — Injeção de Dependência Manual
Sem usar Spring, implemente manualmente a injeção de dependência:
- Crie interfaces `ProdutoRepository` e `NotificacaoService`
- Crie implementações `MemoriaProdutoRepository` e `EmailNotificacaoService`
- Injete-as em `ProdutoService` via construtor
- Demonstre como trocar a implementação sem mudar o Service

### Exercício 3 — Arquitetura Hexagonal Simplificada
Modele um sistema de pagamentos usando Hexagonal:
- **Domain:** `Pagamento`, `PagamentoService`
- **Port (input):** `ProcessarPagamentoUseCase`
- **Port (output):** `PagamentoRepository`, `NotificacaoGateway`
- **Adapter:** `PagamentoApiController`, `BancoDadosPagamentoRepository`

---

## ➡️ Próximo módulo: [07 - Spring Boot](../07-spring-boot/README.md)
