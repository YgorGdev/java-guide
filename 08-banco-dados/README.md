# Módulo 08 — Banco de Dados com JPA/Hibernate

## O que você vai aprender

- O que é JPA e Hibernate
- Mapeamento de entidades
- Relacionamentos (OneToMany, ManyToOne, ManyToMany)
- Spring Data JPA — repositórios inteligentes
- Queries com JPQL e SQL nativo
- Transações
- Conexão com Oracle (produção)

---

## 8.1 O que é JPA?

**JPA** (Java Persistence API) = especificação de como mapear objetos Java para tabelas do banco.  
**Hibernate** = implementação do JPA (o mais popular).  
**Spring Data JPA** = simplifica ainda mais, gerando queries automaticamente.

```
Objeto Java (Produto) ←→ Tabela banco (produto)
  campo: nome          ←→  coluna: nom_produto
  campo: preco         ←→  coluna: vlr_preco
```

**Comparação com PHP:**
- PHP puro / PDO: você escreve SQL na mão em cada arquivo ou classe
- Laravel Eloquent: ORM que mapeia classes para tabelas (similar ao JPA)
- JPA / Hibernate: você define o mapeamento uma vez, o Hibernate gera o SQL automaticamente

> 💡 Se você usou Eloquent no Laravel, o JPA vai parecer familiar. A diferença é que em Java o mapeamento é feito com anotações (`@Entity`, `@Column`) em vez de convenção de nomes.

---

## 8.2 Mapeamento Básico de Entidade

```java
import jakarta.persistence.*;
import java.time.LocalDateTime;

@Entity                           // esta classe é uma tabela do banco
@Table(name = "produto",          // nome da tabela (se diferente do padrão)
       schema = "apps")           // schema Oracle (equivalente ao "owner")
public class Produto {

    @Id                                              // chave primária
    @GeneratedValue(strategy = GenerationType.IDENTITY) // auto-increment
    @Column(name = "cod_produto")
    private Long id;

    @Column(name = "nom_produto",
            nullable = false,
            length = 100)
    private String nome;

    @Column(name = "vlr_preco",
            nullable = false,
            precision = 10, scale = 2)   // para decimais
    private Double preco;

    @Column(name = "qtd_estoque")
    private Integer estoque;

    @Column(name = "dat_criacao",
            updatable = false)           // não atualiza após insert
    private LocalDateTime dataCriacao;

    @Column(name = "dat_atualizacao")
    private LocalDateTime dataAtualizacao;

    @PrePersist  // executado antes de INSERT
    public void prePersist() {
        dataCriacao = LocalDateTime.now();
        dataAtualizacao = LocalDateTime.now();
    }

    @PreUpdate   // executado antes de UPDATE
    public void preUpdate() {
        dataAtualizacao = LocalDateTime.now();
    }

    // Getters e Setters...
}
```

---

## 8.3 Relacionamentos

### @ManyToOne e @OneToMany (o mais comum)

```java
// PEDIDO tem muitos ITENS → cada ITEM pertence a um PEDIDO

@Entity
@Table(name = "pedido")
public class Pedido {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String cliente;

    // Um Pedido tem muitos Itens
    @OneToMany(mappedBy = "pedido",
               cascade = CascadeType.ALL,  // salva os itens junto com o pedido
               fetch = FetchType.LAZY)      // só carrega os itens quando acessar
    private List<ItemPedido> itens = new ArrayList<>();

    // Método utilitário
    public void adicionarItem(ItemPedido item) {
        itens.add(item);
        item.setPedido(this);
    }

    public double getTotal() {
        return itens.stream()
            .mapToDouble(item -> item.getPreco() * item.getQuantidade())
            .sum();
    }
}

@Entity
@Table(name = "item_pedido")
public class ItemPedido {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    // Muitos Itens pertencem a um Pedido
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "cod_pedido", nullable = false)  // coluna FK
    private Pedido pedido;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "cod_produto", nullable = false)
    private Produto produto;

    private int quantidade;
    private double preco;  // preço no momento da compra
}
```

---

## 8.4 Spring Data JPA — Repositórios Mágicos

```java
import org.springframework.data.jpa.repository.*;
import org.springframework.data.domain.*;

public interface PedidoRepository extends JpaRepository<Pedido, Long> {

    // Spring gera o SQL pelo nome do método
    List<Pedido> findByCliente(String cliente);
    List<Pedido> findByClienteContainingIgnoreCase(String nome);
    List<Pedido> findByDataCriacaoAfter(LocalDateTime data);

    // Com ordenação
    List<Pedido> findByClienteOrderByDataCriacaoDesc(String cliente);

    // Com paginação
    Page<Pedido> findAll(Pageable pageable);

    // Contagem
    long countByCliente(String cliente);

    // Verificação
    boolean existsByClienteAndId(String cliente, Long id);

    // JPQL (Java Persistence Query Language) — trabalha com objetos
    @Query("SELECT p FROM Pedido p JOIN FETCH p.itens WHERE p.id = :id")
    Optional<Pedido> findByIdComItens(@Param("id") Long id);

    // SQL nativo — quando o JPQL não atende
    @Query(value = """
        SELECT p.cod_pedido, p.nom_cliente, COUNT(i.cod_item) qtd_itens,
               SUM(i.vlr_preco * i.qtd_item) total
        FROM pedido p
        JOIN item_pedido i ON i.cod_pedido = p.cod_pedido
        WHERE TRUNC(p.dat_criacao) = TRUNC(SYSDATE)
        GROUP BY p.cod_pedido, p.nom_cliente
        ORDER BY total DESC
        """, nativeQuery = true)
    List<Object[]> relatorioDiario();

    // Modificação
    @Modifying
    @Query("UPDATE Pedido p SET p.status = :status WHERE p.id = :id")
    int atualizarStatus(@Param("id") Long id, @Param("status") String status);
}
```

---

## 8.5 Configuração Oracle (Produção)

```xml
<!-- pom.xml — driver Oracle -->
<dependency>
    <groupId>com.oracle.database.jdbc</groupId>
    <artifactId>ojdbc11</artifactId>
    <version>23.3.0.23.09</version>
    <scope>runtime</scope>
</dependency>
```

```properties
# application.properties — Oracle
spring.datasource.url=jdbc:oracle:thin:@//servidor-oracle:1521/ORCL
spring.datasource.username=${DB_USER:apps}
spring.datasource.password=${DB_PASS:senha}
spring.datasource.driver-class-name=oracle.jdbc.OracleDriver

# Pool de conexões (HikariCP — padrão do Spring Boot)
spring.datasource.hikari.maximum-pool-size=10
spring.datasource.hikari.minimum-idle=2
spring.datasource.hikari.connection-timeout=20000
spring.datasource.hikari.idle-timeout=300000

# JPA
spring.jpa.database-platform=org.hibernate.dialect.OracleDialect
spring.jpa.hibernate.ddl-auto=validate   # produção: nunca use create/create-drop!
spring.jpa.show-sql=false                # desligar em produção
```

---

## 8.6 Transactions

```java
@Service
@Transactional(readOnly = true)   // padrão: só leitura (otimização)
public class PedidoService {

    @Transactional   // sobrescreve: leitura e escrita
    public Pedido criar(CriarPedidoRequest request) {
        Pedido pedido = new Pedido(request.getCliente());

        for (ItemRequest itemReq : request.getItens()) {
            Produto produto = produtoRepository.findById(itemReq.getProdutoId())
                .orElseThrow(() -> new RuntimeException("Produto não encontrado"));

            // Verifica e baixa estoque
            produtoService.baixarEstoque(produto.getId(), itemReq.getQuantidade());

            ItemPedido item = new ItemPedido(produto, itemReq.getQuantidade(), produto.getPreco());
            pedido.adicionarItem(item);
        }

        return pedidoRepository.save(pedido);
        // se qualquer exceção for lançada, tudo é revertido (rollback)
    }
}
```

---

## ✅ Boas Práticas JPA

1. **Sempre use `FetchType.LAZY`** — carrega relacionamentos só quando necessário
2. **Nunca use `ddl-auto=create` em produção** — use Flyway ou Liquibase para migrations
3. **Cuidado com N+1** — use `JOIN FETCH` quando precisar carregar relacionamentos junto
4. **Transactions em Service, não em Controller**
5. **`readOnly = true`** em métodos só de leitura — melhora performance

---

## ➡️ Próximo módulo: [09 - Testes](../09-testes/README.md)
