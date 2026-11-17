# Módulo 03 — Coleções, Streams e Lambda

## O que você vai aprender

- List, Set, Map — quando usar cada um
- Iterator e for-each
- Expressões Lambda
- Streams API — processamento funcional de dados
- Optional — evitando NullPointerException

---

## 3.1 Collections Framework — Mapa Mental

```
Collection
├── List (ordenada, aceita duplicatas)
│   ├── ArrayList  → acesso rápido por índice, uso geral
│   └── LinkedList → inserção/remoção rápida no início/meio
│
├── Set (sem duplicatas)
│   ├── HashSet    → sem ordem, muito rápido
│   ├── LinkedHashSet → mantém ordem de inserção
│   └── TreeSet    → ordenado naturalmente (A-Z, 0-9)
│
└── Queue/Deque
    └── ArrayDeque → fila e pilha ao mesmo tempo

Map (chave → valor, sem duplicatas de chave)
├── HashMap       → sem ordem, muito rápido
├── LinkedHashMap → mantém ordem de inserção
└── TreeMap       → ordenado por chave
```

---

## 3.2 List

```java
import java.util.*;

public class ExemploList {
    public static void main(String[] args) {
        // ArrayList — uso mais comum
        List<String> nomes = new ArrayList<>();

        // Adicionar
        nomes.add("Carlos");
        nomes.add("Ana");
        nomes.add("Bruno");
        nomes.add("Ana");  // duplicata permitida!

        System.out.println(nomes);        // [Carlos, Ana, Bruno, Ana]
        System.out.println(nomes.size()); // 4
        System.out.println(nomes.get(1)); // Ana (índice 0 = Carlos)

        // Verificar
        System.out.println(nomes.contains("Bruno")); // true
        System.out.println(nomes.indexOf("Ana"));    // 1 (primeira ocorrência)

        // Remover
        nomes.remove("Ana");              // remove a primeira "Ana"
        nomes.remove(0);                  // remove pelo índice

        // Iterar
        for (String nome : nomes) {
            System.out.println(nome);
        }

        // Ordenar
        Collections.sort(nomes);                        // ordem alfabética
        nomes.sort(Comparator.reverseOrder());           // ordem reversa
        nomes.sort(Comparator.comparingInt(String::length)); // por tamanho

        // Lista imutável (Java 9+)
        List<String> fixo = List.of("SP", "RJ", "BH");
        // fixo.add("DF"); // lança UnsupportedOperationException!

        // Converter array → List
        String[] array = {"x", "y", "z"};
        List<String> daArray = new ArrayList<>(Arrays.asList(array));
    }
}
```

---

## 3.3 Map

```java
import java.util.*;

public class ExemploMap {
    public static void main(String[] args) {
        Map<String, Integer> estoque = new HashMap<>();

        // Colocar valores
        estoque.put("Parafuso M6", 500);
        estoque.put("Porca M6", 480);
        estoque.put("Arruela", 1000);

        // Acessar
        System.out.println(estoque.get("Parafuso M6"));     // 500
        System.out.println(estoque.get("Pino"));             // null
        System.out.println(estoque.getOrDefault("Pino", 0)); // 0

        // Verificar
        System.out.println(estoque.containsKey("Arruela")); // true
        System.out.println(estoque.containsValue(480));     // true

        // Atualizar
        estoque.put("Parafuso M6", 450);              // substitui
        estoque.merge("Porca M6", 20, Integer::sum);  // 480 + 20 = 500

        // Iterar chave-valor
        for (Map.Entry<String, Integer> entry : estoque.entrySet()) {
            System.out.printf("%-15s → %d%n", entry.getKey(), entry.getValue());
        }

        // Iterar só chaves ou valores
        for (String chave : estoque.keySet()) {
            System.out.println(chave);
        }

        // Map imutável (Java 9+)
        Map<String, String> config = Map.of(
            "url", "jdbc:oracle:thin:@server:1521/db",
            "user", "apps"
        );
    }
}
```

---

## 3.4 Lambda e Expressões Funcionais

Lambda é uma **função anônima** — uma função sem nome, declarada inline.

```java
// Antes do Java 8 — Classe anônima verbosa
Comparator<String> compAntigo = new Comparator<String>() {
    @Override
    public int compare(String a, String b) {
        return a.compareTo(b);
    }
};

// Com Lambda — compacto
Comparator<String> compNovo = (a, b) -> a.compareTo(b);

// Ainda mais curto com method reference
Comparator<String> compRef = String::compareTo;
```

**Sintaxe Lambda:**
```java
// (parâmetros) -> { corpo }
// (parâmetros) -> expressão (quando corpo tem 1 linha)

Runnable r = () -> System.out.println("Sem parâmetros");
Consumer<String> c = s -> System.out.println(s);       // 1 parâmetro
Function<Integer, Integer> f = x -> x * 2;             // entrada → saída
BiFunction<Integer, Integer, Integer> soma = (a, b) -> a + b;
```

---

## 3.5 Streams API

Stream = pipeline de operações sobre uma coleção. **Não modifica a original.**

```java
import java.util.*;
import java.util.stream.*;

public class ExemploStreams {
    public static void main(String[] args) {
        List<Integer> numeros = List.of(5, 3, 8, 1, 9, 2, 7, 4, 6, 10);

        // --- Operações intermediárias (lazy) ---

        // filter — filtra elementos
        List<Integer> pares = numeros.stream()
            .filter(n -> n % 2 == 0)
            .collect(Collectors.toList());
        System.out.println("Pares: " + pares); // [8, 2, 4, 6, 10]

        // map — transforma cada elemento
        List<Integer> dobrados = numeros.stream()
            .map(n -> n * 2)
            .collect(Collectors.toList());

        // sorted — ordena
        List<Integer> ordenados = numeros.stream()
            .sorted()
            .collect(Collectors.toList());

        // --- Operações terminais ---

        // count
        long qtdMaioresQue5 = numeros.stream()
            .filter(n -> n > 5)
            .count();
        System.out.println("Maiores que 5: " + qtdMaioresQue5); // 5

        // sum, min, max, average
        int soma = numeros.stream()
            .mapToInt(Integer::intValue)
            .sum();
        System.out.println("Soma: " + soma); // 55

        OptionalInt max = numeros.stream()
            .mapToInt(Integer::intValue)
            .max();
        System.out.println("Máximo: " + max.getAsInt()); // 10

        // findFirst
        Optional<Integer> primeiroPar = numeros.stream()
            .filter(n -> n % 2 == 0)
            .findFirst();
        primeiroPar.ifPresent(n -> System.out.println("Primeiro par: " + n));

        // anyMatch, allMatch, noneMatch
        boolean temNegativo = numeros.stream().anyMatch(n -> n < 0);
        boolean todosMaioresZero = numeros.stream().allMatch(n -> n > 0);

        // --- Exemplo com objetos ---
        record Produto(String nome, double preco, String categoria) {}

        List<Produto> produtos = List.of(
            new Produto("Notebook", 3500.0, "Eletrônico"),
            new Produto("Mouse", 150.0, "Eletrônico"),
            new Produto("Cadeira", 800.0, "Móvel"),
            new Produto("Mesa", 1200.0, "Móvel"),
            new Produto("Monitor", 2000.0, "Eletrônico")
        );

        // Produtos eletrônicos acima de R$ 500, ordenados por preço
        produtos.stream()
            .filter(p -> p.categoria().equals("Eletrônico"))
            .filter(p -> p.preco() > 500)
            .sorted(Comparator.comparingDouble(Produto::preco))
            .forEach(p -> System.out.printf("%-10s R$ %.2f%n", p.nome(), p.preco()));

        // Soma total dos eletrônicos
        double totalEletronicos = produtos.stream()
            .filter(p -> p.categoria().equals("Eletrônico"))
            .mapToDouble(Produto::preco)
            .sum();
        System.out.printf("Total eletrônicos: R$ %.2f%n", totalEletronicos);

        // Agrupar por categoria (collectors)
        Map<String, List<Produto>> porCategoria = produtos.stream()
            .collect(Collectors.groupingBy(Produto::categoria));

        porCategoria.forEach((cat, lista) -> {
            System.out.println(cat + ": " + lista.stream()
                .map(Produto::nome)
                .collect(Collectors.joining(", ")));
        });

        // Nomes como String separados por vírgula
        String nomes = produtos.stream()
            .map(Produto::nome)
            .collect(Collectors.joining(", "));
        System.out.println("Produtos: " + nomes);
    }
}
```

---

## 3.6 Optional — Chega de NullPointerException

```java
import java.util.Optional;

public class ExemploOptional {

    public Optional<String> buscarUsuario(int id) {
        if (id == 1) return Optional.of("Ygor");
        return Optional.empty();
    }

    public static void main(String[] args) {
        ExemploOptional ex = new ExemploOptional();

        Optional<String> usuario = ex.buscarUsuario(1);

        // Verificar e pegar
        if (usuario.isPresent()) {
            System.out.println("Encontrado: " + usuario.get());
        }

        // Valor padrão se ausente
        String nome = ex.buscarUsuario(99)
            .orElse("Usuário não encontrado");
        System.out.println(nome);

        // Executar se presente
        ex.buscarUsuario(1).ifPresent(n -> System.out.println("Olá, " + n));

        // Transformar se presente
        int tamanho = ex.buscarUsuario(1)
            .map(String::length)
            .orElse(0);
        System.out.println("Tamanho do nome: " + tamanho);
    }
}
```

---

## ✅ Exercícios

### Exercício 1 — Streams com Funcionários
Dado um `record Funcionario(String nome, String departamento, double salario)`, crie uma lista com pelo menos 8 funcionários de departamentos variados e use Streams para:
- Listar os do departamento "TI"
- Calcular a média salarial geral
- Encontrar o nome do funcionário com maior salário
- Contar quantos ganham acima de 5000
- Agrupar por departamento e imprimir cada grupo
- Listar os nomes em ordem alfabética, separados por vírgula

### Exercício 2 — Estoque com HashMap
Crie um sistema de estoque com `HashMap<String, Integer>` que suporte:
- `adicionar(produto, quantidade)` — adiciona ao estoque
- `retirar(produto, quantidade)` — retira (impede se quantidade insuficiente)
- `consultar(produto)` — retorna a quantidade
- `listarAbaixoDe(minimo)` — lista produtos com estoque crítico
- `relatorio()` — imprime todo o estoque ordenado por nome

### Exercício 3 — Set para Eliminar Duplicatas
Você tem uma lista de e-mails com possíveis duplicatas. Use `Set` para:
- Eliminar duplicatas
- Verificar se um e-mail já está cadastrado
- Encontrar e-mails que estão em duas listas diferentes (interseção)

```java
List<String> lista1 = List.of("a@x.com", "b@x.com", "c@x.com", "a@x.com");
List<String> lista2 = List.of("b@x.com", "d@x.com", "e@x.com");
// Resultado esperado na interseção: [b@x.com]
```

### Exercício 4 — Processamento de Notas
Dada uma lista de alunos com suas notas, use Streams para:
- Calcular a média de cada aluno
- Classificar como Aprovado (>= 7.0), Recuperação (>= 5.0) ou Reprovado
- Listar os 3 melhores alunos
- Calcular a porcentagem de aprovados

```java
record Aluno(String nome, List<Double> notas) {
    double media() {
        return notas.stream().mapToDouble(Double::doubleValue).average().orElse(0);
    }
}
```

### Exercício 5 — Agenda de Contatos
Implemente uma agenda usando `TreeMap<String, String>` (nome → telefone) que:
- Adicione contatos
- Busque por nome (parcial, case-insensitive)
- Liste todos em ordem alfabética
- Remova contatos
- Exporte como String formatada

---

## 🎯 Gabarito Comentado — Exercício 1 (Streams)

```java
import java.util.*;
import java.util.stream.*;

public class ExercicioFuncionarios {

    record Funcionario(String nome, String departamento, double salario) {}

    public static void main(String[] args) {
        List<Funcionario> funcionarios = List.of(
            new Funcionario("Ana",    "TI",      7500.0),
            new Funcionario("Bruno",  "RH",      4200.0),
            new Funcionario("Carlos", "TI",      8900.0),
            new Funcionario("Diana",  "Vendas",  5600.0),
            new Funcionario("Eva",    "TI",      6300.0),
            new Funcionario("Felipe", "RH",      4800.0),
            new Funcionario("Gabi",   "Vendas",  7100.0),
            new Funcionario("Hugo",   "TI",      9200.0)
        );

        // 1. Funcionários de TI
        System.out.println("=== TI ===");
        funcionarios.stream()
            .filter(f -> f.departamento().equals("TI"))
            .forEach(f -> System.out.println(f.nome() + " — R$ " + f.salario()));

        // 2. Média salarial
        double media = funcionarios.stream()
            .mapToDouble(Funcionario::salario)
            .average()
            .orElse(0);
        System.out.printf("Média salarial: R$ %.2f%n", media);

        // 3. Maior salário
        funcionarios.stream()
            .max(Comparator.comparingDouble(Funcionario::salario))
            .ifPresent(f -> System.out.println("Maior salário: " + f.nome()));

        // 4. Acima de 5000
        long qtd = funcionarios.stream()
            .filter(f -> f.salario() > 5000)
            .count();
        System.out.println("Acima de R$ 5.000: " + qtd);

        // 5. Agrupado por departamento
        Map<String, List<Funcionario>> porDepto = funcionarios.stream()
            .collect(Collectors.groupingBy(Funcionario::departamento));
        porDepto.forEach((depto, lista) -> {
            System.out.println(depto + ": " +
                lista.stream().map(Funcionario::nome).collect(Collectors.joining(", ")));
        });

        // 6. Nomes em ordem alfabética
        String nomesOrdenados = funcionarios.stream()
            .map(Funcionario::nome)
            .sorted()
            .collect(Collectors.joining(", "));
        System.out.println("Nomes: " + nomesOrdenados);
    }
}
```

---

## ➡️ Próximo módulo: [04 - Exceções e I/O](../04-excecoes-io/README.md)
