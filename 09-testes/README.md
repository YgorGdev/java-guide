# Módulo 09 — Testes Automatizados

## O que você vai aprender

- Por que testar
- JUnit 5 — testes unitários
- Mockito — simulando dependências
- Spring Boot Test — testes de integração
- TDD básico

---

## 9.1 Por que Testar?

> "Código sem teste é código que você não entende completamente."

**Problemas sem testes:**
- Medo de alterar código (pode quebrar algo)
- Bugs que chegam em produção
- Regressões — funcionalidade que funcionava para de funcionar
- Código difícil de refatorar

**Com testes:**
- Refatore com confiança
- Detecta bugs cedo (desenvolvimento, não produção!)
- Documenta o comportamento esperado
- Facilita onboarding do time

---

## 9.2 JUnit 5 — Testes Unitários

```java
import org.junit.jupiter.api.*;
import static org.junit.jupiter.api.Assertions.*;

public class CalculadoraTest {

    private Calculadora calculadora;

    @BeforeEach  // executa ANTES de cada teste
    void setUp() {
        calculadora = new Calculadora();
    }

    @Test
    @DisplayName("Deve somar dois números positivos")
    void deveSomarDoisNumerosPositivos() {
        // Arrange (preparar)
        int a = 5, b = 3;

        // Act (executar)
        int resultado = calculadora.somar(a, b);

        // Assert (verificar)
        assertEquals(8, resultado, "5 + 3 deve ser 8");
    }

    @Test
    @DisplayName("Deve lançar exceção ao dividir por zero")
    void deveLancarExcecaoAoDividirPorZero() {
        // Verificar que a exceção é lançada
        ArithmeticException ex = assertThrows(
            ArithmeticException.class,
            () -> calculadora.dividir(10, 0)
        );
        assertTrue(ex.getMessage().contains("zero"));
    }

    @ParameterizedTest
    @DisplayName("Deve calcular IMC corretamente")
    @CsvSource({
        "70, 1.75, NORMAL",
        "50, 1.75, ABAIXO_DO_PESO",
        "100, 1.75, SOBREPESO",
        "120, 1.65, OBESO"
    })
    void deveCalcularIMC(double peso, double altura, String classificacao) {
        assertEquals(classificacao, calculadora.classificarIMC(peso, altura));
    }
}
```

---

## 9.3 Mockito — Simulando Dependências

```java
import org.junit.jupiter.api.*;
import org.mockito.*;
import static org.mockito.Mockito.*;
import static org.junit.jupiter.api.Assertions.*;
import java.util.Optional;

@ExtendWith(MockitoExtension.class)
class ProdutoServiceTest {

    @Mock
    ProdutoRepository repository;    // objeto falso (mock)

    @InjectMocks
    ProdutoService service;          // recebe o mock injetado

    @Test
    @DisplayName("Deve retornar produto quando encontrado")
    void deveBuscarProdutoPorId() {
        // Arrange — configurar comportamento do mock
        Produto produto = new Produto("Parafuso M6", 0.50, 1000);
        when(repository.findById(1L)).thenReturn(Optional.of(produto));

        // Act
        Produto resultado = service.buscarPorId(1L);

        // Assert
        assertNotNull(resultado);
        assertEquals("Parafuso M6", resultado.getNome());
        verify(repository, times(1)).findById(1L);  // verificar que foi chamado
    }

    @Test
    @DisplayName("Deve lançar exceção quando produto não encontrado")
    void deveLancarExcecaoQuandoNaoEncontrado() {
        when(repository.findById(99L)).thenReturn(Optional.empty());

        assertThrows(
            ProdutoNaoEncontradoException.class,
            () -> service.buscarPorId(99L)
        );
    }

    @Test
    @DisplayName("Não deve salvar produto com preço negativo")
    void naoDeveSalvarProdutoComPrecoNegativo() {
        Produto invalido = new Produto("X", -1.0, 0);

        assertThrows(
            IllegalArgumentException.class,
            () -> service.criar(invalido)
        );

        // Verificar que o repository NÃO foi chamado
        verify(repository, never()).save(any());
    }
}
```

---

## 9.4 Spring Boot Test — Integração

```java
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.test.web.client.TestRestTemplate;
import org.springframework.boot.test.web.server.LocalServerPort;
import org.springframework.http.*;

@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
class ProdutoControllerIntegrationTest {

    @LocalServerPort
    private int porta;

    @Autowired
    private TestRestTemplate restTemplate;

    @Test
    @DisplayName("POST /produtos deve criar e retornar 201")
    void deveCriarProduto() {
        Produto produto = new Produto("Monitor 24\"", 1500.0, 50);

        ResponseEntity<Produto> response = restTemplate.postForEntity(
            "http://localhost:" + porta + "/api/v1/produtos",
            produto,
            Produto.class
        );

        assertEquals(HttpStatus.CREATED, response.getStatusCode());
        assertNotNull(response.getBody().getId());
        assertEquals("Monitor 24\"", response.getBody().getNome());
    }

    @Test
    @DisplayName("GET /produtos/{id} deve retornar 404 quando não existe")
    void deveRetornar404QuandoNaoExiste() {
        ResponseEntity<String> response = restTemplate.getForEntity(
            "http://localhost:" + porta + "/api/v1/produtos/9999",
            String.class
        );

        assertEquals(HttpStatus.NOT_FOUND, response.getStatusCode());
    }
}
```

---

## 9.5 TDD — Test Driven Development

**Ciclo TDD:**
```
1. RED   → Escreve o teste que FALHA (a funcionalidade não existe ainda)
2. GREEN → Escreve o mínimo de código para o teste PASSAR
3. REFACTOR → Melhora o código sem quebrar os testes
```

```java
// Passo 1 — RED: escreve o teste primeiro
@Test
void deveCalcularDescontoProgressivo() {
    // até R$100 → 5%, até R$500 → 10%, acima → 15%
    assertEquals(95.0,  service.calcularComDesconto(100.0));  // 5%
    assertEquals(450.0, service.calcularComDesconto(500.0));  // 10%
    assertEquals(850.0, service.calcularComDesconto(1000.0)); // 15%
}

// Passo 2 — GREEN: implementa o mínimo para passar
public double calcularComDesconto(double valor) {
    if (valor <= 100) return valor * 0.95;
    if (valor <= 500) return valor * 0.90;
    return valor * 0.85;
}

// Passo 3 — REFACTOR: melhora se necessário
// (neste caso está ok, mas poderia extrair para constantes, etc.)
```

---

## ✅ Cobertura mínima recomendada

| Tipo | O que testar |
|------|-------------|
| **Service** | Regras de negócio, edge cases, exceções |
| **Controller** | Mapeamento de rotas, códigos HTTP, validação |
| **Repository** | Queries customizadas (opcional, com banco H2) |
| **Utilitários** | Funções puras e cálculos |

**Meta:** 70-80% de cobertura no Service é razoável para começar.

---

## ➡️ Próximo: [10 - Migrando de PHP para Java](../10-migracao-php/README.md)
