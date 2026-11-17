# Módulo 07 — Spring Boot

## O que você vai aprender

- O que é Spring e Spring Boot
- Inversão de Controle e Injeção de Dependências
- Criando uma REST API completa
- Validação de dados
- Tratamento global de erros
- Configuração com application.properties

---

## 7.1 O que é Spring Boot?

**Spring** = o framework Java mais popular do mundo para aplicações enterprise.  
**Spring Boot** = Spring pré-configurado. Você sobe uma API em minutos, não em horas.

| Característica | Spring Boot |
|---------------|------------|
| Servidor embutido | Tomcat/Jetty sem configuração extra |
| Auto-configuração | Detecta dependências e configura automaticamente |
| Starters | Um dependency resolve 10 dependências |
| Production-ready | Métricas, health check, logs já prontos |

**Comparação PHP → Spring Boot:**

| PHP | Spring Boot |
|-----|------------|
| `index.php` / arquivo de rota recebe request HTTP | `@RestController` recebe request HTTP |
| Classe com lógica de negócio | `@Service` com métodos de negócio |
| PDO / Eloquent / Doctrine | `JpaRepository` / `@Query` |
| Validação manual / `filter_var()` | `@Valid` + Bean Validation |
| `config.php` / `.env` | `application.properties` / `application.yml` |
| `composer.json` | `pom.xml` (Maven) |

---

## 7.2 Criando o Projeto

**Via Spring Initializr (recomendado):**

1. Acesse: [start.spring.io](https://start.spring.io)
2. Configure:
   - **Project:** Maven
   - **Language:** Java
   - **Spring Boot:** 3.x (mais recente estável)
   - **Group:** `br.com.caoa` (ou `br.com.seudominio`)
   - **Artifact:** `produtos-api`
   - **Java:** 17 ou 21
3. **Dependencies:** Spring Web, Spring Data JPA, H2 Database, Validation
4. Clique em **Generate** → extrai o zip → abre no IntelliJ

**Estrutura gerada:**
```
produtos-api/
├── src/
│   ├── main/
│   │   ├── java/br/com/caoa/produtosapi/
│   │   │   └── ProdutosApiApplication.java  ← ponto de entrada
│   │   └── resources/
│   │       └── application.properties       ← configurações
│   └── test/
│       └── java/br/com/caoa/produtosapi/
│           └── ProdutosApiApplicationTests.java
└── pom.xml                                  ← dependências Maven
```

---

## 7.3 Inversão de Controle e Injeção de Dependências

**O Spring gerencia a criação dos objetos.** Você não usa `new`.

```java
// SEM Spring (você gerencia tudo)
ProdutoRepository repo = new ProdutoRepositoryImpl();
ProdutoService service = new ProdutoService(repo);  // você instancia
ProdutoController controller = new ProdutoController(service);

// COM Spring (framework gerencia)
// Spring vê @Component/@Service/@Repository e cria automaticamente
// Você recebe o objeto pronto via injeção
@Service
public class ProdutoService {
    // Spring injeta o repository automaticamente
    private final ProdutoRepository repository;

    // Injeção via construtor — melhor prática
    public ProdutoService(ProdutoRepository repository) {
        this.repository = repository;
    }
}
```

**Anotações principais:**

| Anotação | Uso |
|----------|-----|
| `@Component` | Componente genérico gerenciado pelo Spring |
| `@Service` | Lógica de negócio |
| `@Repository` | Acesso a dados |
| `@RestController` | Controller HTTP (retorna JSON) |
| `@Controller` | Controller HTTP (retorna View/HTML) |
| `@Configuration` | Configuração de beans |

---

## 7.4 REST API Completa — CRUD de Produto

### Entidade

```java
// src/main/java/.../model/Produto.java
import jakarta.persistence.*;
import jakarta.validation.constraints.*;

@Entity
@Table(name = "produto")
public class Produto {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @NotBlank(message = "Nome é obrigatório")
    @Size(min = 2, max = 100, message = "Nome deve ter 2-100 caracteres")
    @Column(nullable = false)
    private String nome;

    @NotNull(message = "Preço é obrigatório")
    @Positive(message = "Preço deve ser positivo")
    @Column(nullable = false)
    private Double preco;

    @NotNull(message = "Estoque é obrigatório")
    @PositiveOrZero(message = "Estoque não pode ser negativo")
    @Column(nullable = false)
    private Integer estoque;

    @Column(length = 500)
    private String descricao;

    // Construtores
    public Produto() {}

    public Produto(String nome, Double preco, Integer estoque) {
        this.nome = nome;
        this.preco = preco;
        this.estoque = estoque;
    }

    // Getters e Setters
    public Long getId() { return id; }
    public String getNome() { return nome; }
    public void setNome(String nome) { this.nome = nome; }
    public Double getPreco() { return preco; }
    public void setPreco(Double preco) { this.preco = preco; }
    public Integer getEstoque() { return estoque; }
    public void setEstoque(Integer estoque) { this.estoque = estoque; }
    public String getDescricao() { return descricao; }
    public void setDescricao(String descricao) { this.descricao = descricao; }
}
```

### Repository

```java
// src/main/java/.../repository/ProdutoRepository.java
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Query;
import java.util.List;

// Spring Data JPA gera o SQL automaticamente!
public interface ProdutoRepository extends JpaRepository<Produto, Long> {

    // Spring gera: SELECT * FROM produto WHERE nome LIKE %:nome%
    List<Produto> findByNomeContainingIgnoreCase(String nome);

    // Spring gera: SELECT * FROM produto WHERE estoque < :quantidade
    List<Produto> findByEstoqueLessThan(int quantidade);

    // Query manual se precisar
    @Query("SELECT p FROM Produto p WHERE p.preco BETWEEN :min AND :max ORDER BY p.preco")
    List<Produto> findByFaixaPreco(Double min, Double max);
}
```

### Service

```java
// src/main/java/.../service/ProdutoService.java
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;
import java.util.List;

@Service
@Transactional  // Spring gerencia a transação
public class ProdutoService {

    private final ProdutoRepository repository;

    public ProdutoService(ProdutoRepository repository) {
        this.repository = repository;
    }

    public List<Produto> listarTodos() {
        return repository.findAll();
    }

    public Produto buscarPorId(Long id) {
        return repository.findById(id)
            .orElseThrow(() -> new ProdutoNaoEncontradoException(id));
    }

    public Produto criar(Produto produto) {
        return repository.save(produto);
    }

    public Produto atualizar(Long id, Produto dadosNovos) {
        Produto produto = buscarPorId(id);
        produto.setNome(dadosNovos.getNome());
        produto.setPreco(dadosNovos.getPreco());
        produto.setEstoque(dadosNovos.getEstoque());
        produto.setDescricao(dadosNovos.getDescricao());
        return repository.save(produto);
    }

    public void deletar(Long id) {
        buscarPorId(id); // lança exceção se não existir
        repository.deleteById(id);
    }

    public List<Produto> alertarEstoqueBaixo(int minimo) {
        return repository.findByEstoqueLessThan(minimo);
    }
}
```

### Controller

```java
// src/main/java/.../controller/ProdutoController.java
import org.springframework.http.*;
import org.springframework.web.bind.annotation.*;
import jakarta.validation.Valid;
import java.util.List;

@RestController
@RequestMapping("/api/v1/produtos")
public class ProdutoController {

    private final ProdutoService service;

    public ProdutoController(ProdutoService service) {
        this.service = service;
    }

    // GET /api/v1/produtos
    @GetMapping
    public ResponseEntity<List<Produto>> listar() {
        return ResponseEntity.ok(service.listarTodos());
    }

    // GET /api/v1/produtos/{id}
    @GetMapping("/{id}")
    public ResponseEntity<Produto> buscar(@PathVariable Long id) {
        return ResponseEntity.ok(service.buscarPorId(id));
    }

    // POST /api/v1/produtos
    @PostMapping
    public ResponseEntity<Produto> criar(@Valid @RequestBody Produto produto) {
        Produto criado = service.criar(produto);
        return ResponseEntity.status(HttpStatus.CREATED).body(criado);
    }

    // PUT /api/v1/produtos/{id}
    @PutMapping("/{id}")
    public ResponseEntity<Produto> atualizar(
            @PathVariable Long id,
            @Valid @RequestBody Produto produto) {
        return ResponseEntity.ok(service.atualizar(id, produto));
    }

    // DELETE /api/v1/produtos/{id}
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deletar(@PathVariable Long id) {
        service.deletar(id);
        return ResponseEntity.noContent().build();
    }

    // GET /api/v1/produtos/estoque-baixo?minimo=10
    @GetMapping("/estoque-baixo")
    public ResponseEntity<List<Produto>> estoqueBaixo(
            @RequestParam(defaultValue = "10") int minimo) {
        return ResponseEntity.ok(service.alertarEstoqueBaixo(minimo));
    }
}
```

### Tratamento Global de Erros

```java
// src/main/java/.../exception/GlobalExceptionHandler.java
import org.springframework.http.*;
import org.springframework.web.bind.annotation.*;
import org.springframework.validation.FieldError;
import org.springframework.web.bind.MethodArgumentNotValidException;
import java.util.Map;
import java.util.stream.Collectors;

@RestControllerAdvice
public class GlobalExceptionHandler {

    // Produto não encontrado → 404
    @ExceptionHandler(ProdutoNaoEncontradoException.class)
    public ResponseEntity<Map<String, String>> handleNaoEncontrado(
            ProdutoNaoEncontradoException ex) {
        return ResponseEntity.status(HttpStatus.NOT_FOUND)
            .body(Map.of("erro", ex.getMessage()));
    }

    // Dados inválidos (@Valid falhou) → 400
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<Map<String, Object>> handleValidacao(
            MethodArgumentNotValidException ex) {
        Map<String, String> erros = ex.getBindingResult()
            .getFieldErrors()
            .stream()
            .collect(Collectors.toMap(
                FieldError::getField,
                FieldError::getDefaultMessage
            ));
        return ResponseEntity.badRequest()
            .body(Map.of("erros", erros));
    }

    // Qualquer outro erro → 500
    @ExceptionHandler(Exception.class)
    public ResponseEntity<Map<String, String>> handleGenerico(Exception ex) {
        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR)
            .body(Map.of("erro", "Erro interno. Tente novamente."));
    }
}
```

---

## 7.5 Configuração — application.properties

```properties
# src/main/resources/application.properties

# Servidor
server.port=8080

# Banco de dados (H2 em memória para desenvolvimento)
spring.datasource.url=jdbc:h2:mem:produtosdb
spring.datasource.username=sa
spring.datasource.password=
spring.h2.console.enabled=true

# JPA
spring.jpa.hibernate.ddl-auto=create-drop
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true

# Para Oracle (produção)
# spring.datasource.url=jdbc:oracle:thin:@servidor:1521/ORCL
# spring.datasource.username=usuario
# spring.datasource.password=senha
# spring.datasource.driver-class-name=oracle.jdbc.OracleDriver

# Logs
logging.level.br.com.caoa=DEBUG
logging.level.org.hibernate.SQL=DEBUG
```

---

## 7.6 Testando com curl

```bash
# Criar produto
curl -X POST http://localhost:8080/api/v1/produtos \
  -H "Content-Type: application/json" \
  -d '{"nome":"Parafuso M6","preco":0.50,"estoque":1000}'

# Listar todos
curl http://localhost:8080/api/v1/produtos

# Buscar por ID
curl http://localhost:8080/api/v1/produtos/1

# Atualizar
curl -X PUT http://localhost:8080/api/v1/produtos/1 \
  -H "Content-Type: application/json" \
  -d '{"nome":"Parafuso M6 Zincado","preco":0.65,"estoque":950}'

# Deletar
curl -X DELETE http://localhost:8080/api/v1/produtos/1
```

---

## ✅ Exercícios

1. Adicione um endpoint de busca por nome: `GET /produtos?nome=parafuso`
2. Adicione paginação com `Pageable`: `GET /produtos?page=0&size=10&sort=preco`
3. Crie um endpoint que retorna estatísticas: total de produtos, valor total do estoque, produto mais caro

---

## ➡️ Próximo módulo: [08 - Banco de Dados com JPA](../08-banco-dados/README.md)
