# Módulo 10 — Migrando de PHP para Java

## Por que migrar?

PHP é excelente para aplicações web rápidas, mas conforme o sistema cresce, Java oferece vantagens reais:

| Característica | PHP | Java |
|---|---|---|
| Tipagem | Dinâmica (fraca) | Estática (forte) |
| Concorrência | Limitada (por requisição) | Nativa (threads, CompletableFuture) |
| Ecossistema corporativo | Moderado | Robusto (Spring, JPA, Maven) |
| Performance em alto volume | Variável | Alta e previsível |
| Verificação em tempo de compilação | Não | Sim |
| Frameworks enterprise | Laravel, Symfony | Spring Boot, Quarkus |

---

## 10.1 Sintaxe: PHP vs Java lado a lado

### Variáveis e tipos

```php
// PHP — sem tipagem obrigatória
$nome = "João";
$idade = 30;
$ativo = true;
$preco = 19.99;
```

```java
// Java — tipos obrigatórios
String nome = "João";
int idade = 30;
boolean ativo = true;
double preco = 19.99;

// Java 10+ — inferência de tipo local
var nome = "João";   // compilador infere String
```

---

### Arrays e coleções

```php
// PHP — array é tudo: lista, mapa e objeto ao mesmo tempo
$frutas = ["maçã", "banana", "laranja"];
$pessoa = ["nome" => "João", "idade" => 30];

echo $frutas[0];          // maçã
echo $pessoa["nome"];     // João
```

```java
// Java — tipos distintos para cada estrutura
List<String> frutas = new ArrayList<>(List.of("maçã", "banana", "laranja"));
Map<String, Object> pessoa = new HashMap<>();
pessoa.put("nome", "João");
pessoa.put("idade", 30);

System.out.println(frutas.get(0));        // maçã
System.out.println(pessoa.get("nome"));   // João
```

---

### Funções / Métodos

```php
// PHP
function calcularDesconto(float $preco, float $percentual): float {
    return $preco - ($preco * $percentual / 100);
}

$final = calcularDesconto(100.0, 10.0); // 90.0
```

```java
// Java — funções vivem dentro de classes
public class CalculoDesconto {
    public static double calcularDesconto(double preco, double percentual) {
        return preco - (preco * percentual / 100);
    }

    public static void main(String[] args) {
        double final_ = calcularDesconto(100.0, 10.0); // 90.0
    }
}
```

---

### Classes e objetos

```php
// PHP
class Produto {
    public string $nome;
    public float $preco;

    public function __construct(string $nome, float $preco) {
        $this->nome = $nome;
        $this->preco = $preco;
    }

    public function getDescricao(): string {
        return "{$this->nome} - R$ {$this->preco}";
    }
}

$p = new Produto("Caneta", 2.50);
echo $p->getDescricao();
```

```java
// Java
public class Produto {
    private String nome;
    private double preco;

    public Produto(String nome, double preco) {
        this.nome = nome;
        this.preco = preco;
    }

    public String getDescricao() {
        return nome + " - R$ " + preco;
    }
}

// Uso
Produto p = new Produto("Caneta", 2.50);
System.out.println(p.getDescricao());
```

---

### Herança e interfaces

```php
// PHP
interface Pagamento {
    public function processar(float $valor): bool;
}

class PagamentoCartao implements Pagamento {
    public function processar(float $valor): bool {
        // lógica
        return true;
    }
}
```

```java
// Java — equivalente direto
public interface Pagamento {
    boolean processar(double valor);
}

public class PagamentoCartao implements Pagamento {
    @Override
    public boolean processar(double valor) {
        // lógica
        return true;
    }
}
```

---

### Tratamento de erros

```php
// PHP
try {
    if ($divisor === 0) {
        throw new InvalidArgumentException("Divisor não pode ser zero");
    }
    $resultado = $dividendo / $divisor;
} catch (InvalidArgumentException $e) {
    echo "Erro: " . $e->getMessage();
} finally {
    echo "Operação finalizada";
}
```

```java
// Java — checked vs unchecked exceptions
try {
    if (divisor == 0) {
        throw new IllegalArgumentException("Divisor não pode ser zero");
    }
    int resultado = dividendo / divisor;
} catch (IllegalArgumentException e) {
    System.out.println("Erro: " + e.getMessage());
} finally {
    System.out.println("Operação finalizada");
}
```

---

### String interpolation / formatação

```php
// PHP
$nome = "Maria";
$idade = 25;
echo "Olá, $nome! Você tem $idade anos.";
echo "Olá, {$nome}!";
```

```java
// Java
String nome = "Maria";
int idade = 25;
System.out.printf("Olá, %s! Você tem %d anos.%n", nome, idade);
// Ou com String.format
String msg = String.format("Olá, %s!", nome);
// Java 15+ text blocks
String msg2 = """
        Olá, %s!
        Você tem %d anos.
        """.formatted(nome, idade);
```

---

### Banco de dados

```php
// PHP PDO
$pdo = new PDO("mysql:host=localhost;dbname=loja", "user", "pass");
$stmt = $pdo->prepare("SELECT * FROM produtos WHERE ativo = ?");
$stmt->execute([1]);
$produtos = $stmt->fetchAll(PDO::FETCH_ASSOC);

foreach ($produtos as $p) {
    echo $p['nome'];
}
```

```java
// Java — Spring Data JPA (equivalente e mais poderoso)
@Entity
public class Produto {
    @Id @GeneratedValue
    private Long id;
    private String nome;
    private boolean ativo;
    // getters/setters
}

public interface ProdutoRepository extends JpaRepository<Produto, Long> {
    List<Produto> findByAtivo(boolean ativo);
}

// No Service:
List<Produto> produtos = produtoRepository.findByAtivo(true);
produtos.forEach(p -> System.out.println(p.getNome()));
```

---

## 10.2 Laravel vs Spring Boot

| Conceito | Laravel (PHP) | Spring Boot (Java) |
|---|---|---|
| Roteamento | `Route::get('/users', ...)` | `@GetMapping("/users")` |
| Controller | `class UserController extends Controller` | `@RestController class UserController` |
| ORM | Eloquent (`User::find(1)`) | JPA/Hibernate (`userRepository.findById(1)`) |
| Migrations | `php artisan migrate` | Flyway / Liquibase |
| Variáveis de ambiente | `.env` / `config()` | `application.properties` / `@Value` |
| Validação | `$request->validate([...])` | `@Valid` + Bean Validation |
| Middleware | `middleware('auth')` | `Filter` / `HandlerInterceptor` |
| Injeção de dependência | `app()->make(...)` | `@Autowired` / construtor |
| Testes | PHPUnit + Feature tests | JUnit 5 + MockMvc |

---

### Rota + Controller: comparação direta

```php
// Laravel — routes/api.php
Route::get('/produtos/{id}', [ProdutoController::class, 'show']);

// Laravel — ProdutoController.php
public function show(int $id): JsonResponse {
    $produto = Produto::findOrFail($id);
    return response()->json($produto);
}
```

```java
// Spring Boot — ProdutoController.java
@RestController
@RequestMapping("/produtos")
public class ProdutoController {

    @Autowired
    private ProdutoService service;

    @GetMapping("/{id}")
    public ResponseEntity<Produto> buscar(@PathVariable Long id) {
        return ResponseEntity.ok(service.buscarPorId(id));
    }
}
```

---

### Model Eloquent vs JPA Entity

```php
// Laravel Eloquent
class Produto extends Model {
    protected $fillable = ['nome', 'preco', 'categoria_id'];

    public function categoria(): BelongsTo {
        return $this->belongsTo(Categoria::class);
    }
}

// Uso
$produto = Produto::with('categoria')->find(1);
echo $produto->categoria->nome;
```

```java
// JPA Entity
@Entity
public class Produto {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String nome;
    private double preco;

    @ManyToOne
    @JoinColumn(name = "categoria_id")
    private Categoria categoria;
    // getters/setters
}

// Uso via Repository
Produto produto = produtoRepository.findById(1L).orElseThrow();
System.out.println(produto.getCategoria().getNome());
```

---

## 10.3 Estratégia de migração em 4 fases

### Fase 1 — Análise (2–4 semanas)

- Mapear todos os endpoints/rotas da aplicação PHP
- Listar dependências externas (APIs, serviços, banco de dados)
- Identificar lógica de negócio central vs. utilitários
- Documentar contratos de API (request/response)
- Avaliar volume e complexidade do banco de dados

```
Checklist Fase 1:
[ ] Inventário de rotas e controllers
[ ] Mapeamento de tabelas e relacionamentos
[ ] Identificação de jobs/queues/cron
[ ] Dependências de terceiros (pagamento, email, etc.)
[ ] Cobertura de testes atual
```

---

### Fase 2 — Setup Java (1–2 semanas)

```xml
<!-- pom.xml — dependências equivalentes às do Laravel -->
<dependencies>
    <!-- Web (Laravel Routes → Spring MVC) -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <!-- ORM (Eloquent → Spring Data JPA) -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>

    <!-- Validação (Laravel Validator → Bean Validation) -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-validation</artifactId>
    </dependency>

    <!-- Testes (PHPUnit → JUnit 5) -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>
```

```properties
# application.properties
spring.datasource.url=jdbc:mysql://localhost:3306/minha_base
spring.datasource.username=root
spring.datasource.password=senha
spring.jpa.hibernate.ddl-auto=validate
```

---

### Fase 3 — Migração por módulo (incremental)

Migre um módulo de cada vez, mantendo o PHP funcionando em paralelo:

```
Ordem recomendada:
1. Entidades / Models (sem lógica)
2. Repositories (consultas)
3. Services (regras de negócio)
4. Controllers (endpoints)
5. Autenticação/Autorização
6. Jobs e tarefas agendadas
```

**Padrão Strangler Fig** — redirecione rotas gradualmente:

```
Requisição → Gateway/Nginx
    ├── /api/v1/produtos   → Java (migrado)
    ├── /api/v1/pedidos    → Java (migrado)
    └── /api/v1/usuarios   → PHP (ainda em PHP)
```

---

### Fase 4 — Validação e corte

```java
// Teste de contrato: garanta que o Java retorna o mesmo JSON do PHP
@SpringBootTest(webEnvironment = RANDOM_PORT)
class ProdutoControllerTest {

    @Autowired
    private TestRestTemplate restTemplate;

    @Test
    void deveRetornarProdutoNoMesmoFormatoQuePhp() {
        ResponseEntity<String> response = restTemplate.getForEntity(
            "/produtos/1", String.class
        );

        assertThat(response.getStatusCode()).isEqualTo(HttpStatus.OK);

        // Verificar campos obrigatórios que o PHP retornava
        DocumentContext json = JsonPath.parse(response.getBody());
        assertThat(json.read("$.id", Long.class)).isEqualTo(1L);
        assertThat(json.read("$.nome", String.class)).isNotBlank();
        assertThat(json.read("$.preco", Double.class)).isPositive();
    }
}
```

---

## 10.4 Armadilhas comuns na migração

### 1. `null` vs PHP falsy

```php
// PHP — valores "falsy" silenciosos
$valor = null;
if (!$valor) {
    // executa sem erro
}
```

```java
// Java — NullPointerException se não tratar
String valor = null;
if (valor == null || valor.isEmpty()) {
    // verificação explícita obrigatória
}

// Melhor: usar Optional
Optional<String> valor = Optional.ofNullable(buscarDobanco());
valor.ifPresent(v -> System.out.println(v));
```

---

### 2. Arrays associativos → objetos tipados

```php
// PHP — retorno genérico
function getDadosUsuario(): array {
    return ["nome" => "João", "email" => "joao@email.com"];
}
```

```java
// Java — crie um DTO tipado
public record UsuarioDTO(String nome, String email) {}

public UsuarioDTO getDadosUsuario() {
    return new UsuarioDTO("João", "joao@email.com");
}
```

---

### 3. Requisições HTTP (cURL → RestTemplate/WebClient)

```php
// PHP cURL
$ch = curl_init("https://api.externa.com/dados");
curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
$resposta = curl_exec($ch);
$dados = json_decode($resposta, true);
```

```java
// Java — RestTemplate (síncrono)
RestTemplate restTemplate = new RestTemplate();
Map<String, Object> dados = restTemplate.getForObject(
    "https://api.externa.com/dados", Map.class
);

// Ou WebClient (reativo, recomendado no Spring Boot 3+)
WebClient client = WebClient.create("https://api.externa.com");
Mono<String> resposta = client.get()
    .uri("/dados")
    .retrieve()
    .bodyToMono(String.class);
```

---

### 4. Sessions e autenticação

```php
// PHP — session nativa
session_start();
$_SESSION['usuario_id'] = 123;
```

```java
// Java Spring Boot — Spring Security com JWT
@RestController
public class AuthController {

    @PostMapping("/login")
    public ResponseEntity<TokenResponse> login(@RequestBody LoginRequest req) {
        // validar credenciais
        String token = jwtService.gerarToken(usuario);
        return ResponseEntity.ok(new TokenResponse(token));
    }
}

// Proteger rotas com @PreAuthorize
@GetMapping("/perfil")
@PreAuthorize("isAuthenticated()")
public ResponseEntity<UsuarioDTO> perfil(Principal principal) {
    return ResponseEntity.ok(usuarioService.buscarPorEmail(principal.getName()));
}
```

---

## 10.5 Checklist final da migração

```
✅ Entidades mapeadas com @Entity e relacionamentos corretos
✅ Repositories criados com JpaRepository
✅ Services com lógica de negócio e transações (@Transactional)
✅ Controllers com @RestController e validação (@Valid)
✅ Tratamento de erros centralizado (@ControllerAdvice)
✅ Testes unitários (JUnit 5 + Mockito) cobrindo Services
✅ Testes de integração (MockMvc ou TestRestTemplate)
✅ Configuração de ambiente (application.properties / profiles)
✅ Logs configurados (SLF4J + Logback)
✅ Documentação de API (Swagger/OpenAPI)
```

---

---

## ✅ Exercícios de Migração

### Exercício 1 — Converter Script PHP para Java
Migre este script PHP para Java usando boas práticas:

```php
<?php
function calcularDesconto($preco, $tipo) {
    if ($tipo === "vip") return $preco * 0.80;
    if ($tipo === "regular") return $preco * 0.90;
    return $preco;
}

$produtos = [
    ["nome" => "Notebook", "preco" => 3500.00, "clienteTipo" => "vip"],
    ["nome" => "Mouse",    "preco" => 150.00,  "clienteTipo" => "regular"],
    ["nome" => "Monitor",  "preco" => 2000.00, "clienteTipo" => "normal"],
];

foreach ($produtos as $p) {
    $precoFinal = calcularDesconto($p["preco"], $p["clienteTipo"]);
    echo "{$p['nome']}: R$ {$preco_final}\n";
}
?>
```

**Dicas para migrar:**
- Use `record Produto(String nome, double preco, String tipoCliente)`
- Crie enum `TipoCliente { VIP, REGULAR, NORMAL }` com o percentual de desconto
- Use `List<Produto>` em vez do array PHP
- Use `for-each` e `printf`

### Exercício 2 — API PHP → Spring Boot
Você tem esta "API PHP" simples:

```php
<?php
// usuarios.php
header('Content-Type: application/json');
$acao = $_GET['acao'] ?? '';
$usuarios = json_decode(file_get_contents('usuarios.json'), true) ?? [];

if ($acao === 'listar') {
    echo json_encode($usuarios);
} elseif ($acao === 'buscar') {
    $id = (int)($_GET['id'] ?? 0);
    $encontrado = array_filter($usuarios, fn($u) => $u['id'] === $id);
    echo json_encode(array_values($encontrado)[0] ?? null);
} elseif ($acao === 'criar' && $_SERVER['REQUEST_METHOD'] === 'POST') {
    $novo = json_decode(file_get_contents('php://input'), true);
    $novo['id'] = count($usuarios) + 1;
    $usuarios[] = $novo;
    file_put_contents('usuarios.json', json_encode($usuarios));
    echo json_encode($novo);
}
```

Migre para um `@RestController` Spring Boot com:
- `GET /usuarios` → listar todos
- `GET /usuarios/{id}` → buscar por ID
- `POST /usuarios` → criar novo

### Exercício 3 — Banco de Dados: PDO → JPA
Migre esta consulta PHP/PDO para JPA:

```php
<?php
$pdo = new PDO("mysql:host=localhost;dbname=loja", "root", "senha");

// Buscar pedidos do mês atual com total acima de 500
$stmt = $pdo->prepare("
    SELECT p.id, c.nome AS cliente, p.total, p.criado_em
    FROM pedidos p
    JOIN clientes c ON c.id = p.cliente_id
    WHERE p.total > :minimo
    AND MONTH(p.criado_em) = MONTH(CURRENT_DATE())
    ORDER BY p.total DESC
");
$stmt->execute([':minimo' => 500]);
$pedidos = $stmt->fetchAll(PDO::FETCH_ASSOC);

foreach ($pedidos as $p) {
    echo "{$p['cliente']}: R$ {$p['total']}\n";
}
```

Crie as entidades `Pedido` e `Cliente`, um `PedidoRepository` e um `PedidoService.buscarPedidosDoMes(double valorMinimo)`.

### Exercício 4 — Validação PHP → Bean Validation
Migre esta validação manual PHP para Bean Validation Java:

```php
<?php
function validarCadastro($dados) {
    $erros = [];
    if (empty($dados['nome']) || strlen($dados['nome']) < 3) {
        $erros[] = "Nome deve ter pelo menos 3 caracteres";
    }
    if (!filter_var($dados['email'], FILTER_VALIDATE_EMAIL)) {
        $erros[] = "Email inválido";
    }
    if (strlen($dados['senha']) < 8) {
        $erros[] = "Senha deve ter pelo menos 8 caracteres";
    }
    if ($dados['idade'] < 18) {
        $erros[] = "Deve ser maior de 18 anos";
    }
    return $erros;
}
```

Use `@NotBlank`, `@Size`, `@Email`, `@Min` em um DTO Java e processe-o em um endpoint com `@Valid`.

### Exercício 5 — Projeto Completo de Migração
Implemente em Java um mini-sistema de biblioteca que você escreveria em PHP:

**Funcionalidades:**
- Cadastrar livro (título, autor, ISBN, ano, disponível)
- Registrar empréstimo (livro + usuário + data de devolução prevista)
- Devolver livro
- Listar livros disponíveis
- Listar empréstimos em atraso

**Estrutura esperada:**
```
src/
├── domain/
│   ├── Livro.java
│   ├── Usuario.java
│   └── Emprestimo.java
├── repository/
│   ├── LivroRepository.java
│   └── EmprestimoRepository.java
└── service/
    ├── LivroService.java
    └── EmprestimoService.java
```

---

## 🎯 Gabarito Comentado — Exercício 1

```java
// TipoCliente.java
public enum TipoCliente {
    VIP(0.80),
    REGULAR(0.90),
    NORMAL(1.00);

    private final double multiplicador;

    TipoCliente(double multiplicador) {
        this.multiplicador = multiplicador;
    }

    public double aplicarDesconto(double preco) {
        return preco * multiplicador;
    }
}

// Produto.java
public record Produto(String nome, double preco, TipoCliente tipo) {
    public double precoFinal() {
        return tipo.aplicarDesconto(preco);
    }
}

// Main.java
public class Main {
    public static void main(String[] args) {
        List<Produto> produtos = List.of(
            new Produto("Notebook", 3500.00, TipoCliente.VIP),
            new Produto("Mouse",     150.00, TipoCliente.REGULAR),
            new Produto("Monitor",  2000.00, TipoCliente.NORMAL)
        );

        for (Produto p : produtos) {
            System.out.printf("%-10s: R$ %.2f (desconto: R$ %.2f)%n",
                p.nome(),
                p.precoFinal(),
                p.preco() - p.precoFinal()
            );
        }
    }
}
// Notebook  : R$ 2800,00 (desconto: R$ 700,00)
// Mouse     : R$ 135,00  (desconto: R$ 15,00)
// Monitor   : R$ 2000,00 (desconto: R$ 0,00)
```

---

## 🚀 Próximos Passos

Parabéns por concluir o guia! Você agora tem uma base sólida em Java.

**Para aprofundar:**
- 🔐 **Spring Security** — autenticação, JWT, OAuth2
- 📨 **Mensageria** — RabbitMQ, Apache Kafka com Spring
- 🐳 **Docker** — containerizar sua aplicação Spring Boot
- ☁️ **Cloud** — deploy no AWS, GCP ou Azure
- ⚡ **Reactive** — Spring WebFlux para aplicações reativas
- 🏗️ **Design Patterns** — Factory, Strategy, Observer em Java

---

*Guia Java — Do Iniciante ao Intermediário*
*Módulo 10 de 10*
