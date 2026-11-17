# Módulo 04 — Exceções e I/O

## O que você vai aprender

- O que são exceções e por que existem
- Try/catch/finally
- Checked vs Unchecked Exceptions
- Criando suas próprias exceções
- Leitura e escrita de arquivos

---

## 4.1 O que são Exceções?

Exceção = **evento inesperado** que interrompe o fluxo normal do programa.

```
Hierarquia de Exceções:
Throwable
├── Error (JVM — não capture, não tem o que fazer)
│   ├── OutOfMemoryError
│   └── StackOverflowError
└── Exception
    ├── IOException (checked — você DEVE tratar)
    │   ├── FileNotFoundException
    │   └── SQLException
    └── RuntimeException (unchecked — erro de programação)
        ├── NullPointerException   → usou objeto null
        ├── ArrayIndexOutOfBoundsException → índice inválido
        ├── ClassCastException    → cast inválido
        ├── NumberFormatException → "abc" → int
        └── IllegalArgumentException → argumento inválido
```

---

## 4.2 Try / Catch / Finally

```java
public class ExemploExcecoes {
    public static void main(String[] args) {

        // Captura básica
        try {
            int[] numeros = {1, 2, 3};
            System.out.println(numeros[10]); // índice inválido!
        } catch (ArrayIndexOutOfBoundsException e) {
            System.out.println("Índice inválido: " + e.getMessage());
        }

        // Múltiplos catch
        try {
            String texto = null;
            System.out.println(texto.length()); // NullPointerException
        } catch (NullPointerException e) {
            System.out.println("Objeto nulo: " + e.getMessage());
        } catch (Exception e) {
            System.out.println("Outro erro: " + e.getMessage());
        }

        // Multi-catch (Java 7+)
        try {
            processarInput("abc");
        } catch (NullPointerException | NumberFormatException e) {
            System.out.println("Erro de input: " + e.getMessage());
        }

        // finally — SEMPRE executa (ótimo para fechar recursos)
        try {
            System.out.println("Tentando...");
            if (true) throw new RuntimeException("Simulando erro");
        } catch (RuntimeException e) {
            System.out.println("Capturado: " + e.getMessage());
        } finally {
            System.out.println("Finally: sempre executa!"); // fecharia conexão aqui
        }
    }

    static void processarInput(String valor) {
        int numero = Integer.parseInt(valor); // lança NumberFormatException
        System.out.println(numero * 2);
    }
}
```

---

## 4.3 Checked vs Unchecked

```java
import java.io.*;

public class CheckedUnchecked {

    // Checked: você DEVE declarar throws ou tratar com try/catch
    public static String lerArquivo(String caminho) throws IOException {
        // O compilador OBRIGA você a tratar IOException
        try (BufferedReader reader = new BufferedReader(new FileReader(caminho))) {
            return reader.readLine();
        }
    }

    // Unchecked: RuntimeException, não precisa declarar
    public static int dividir(int a, int b) {
        if (b == 0) throw new ArithmeticException("Não pode dividir por zero");
        return a / b;
    }

    public static void main(String[] args) {
        // Checked — compilador obriga a tratar
        try {
            String linha = lerArquivo("arquivo.txt");
            System.out.println(linha);
        } catch (IOException e) {
            System.out.println("Arquivo não encontrado: " + e.getMessage());
        }

        // Unchecked — tratamento opcional (mas recomendado)
        try {
            System.out.println(dividir(10, 0));
        } catch (ArithmeticException e) {
            System.out.println("Erro: " + e.getMessage());
        }
    }
}
```

---

## 4.4 Criando Suas Próprias Exceções

```java
// Exceção customizada unchecked (boa prática para aplicações)
public class SaldoInsuficienteException extends RuntimeException {
    private final double saldoAtual;
    private final double valorSolicitado;

    public SaldoInsuficienteException(double saldoAtual, double valorSolicitado) {
        super(String.format(
            "Saldo insuficiente. Disponível: R$ %.2f | Solicitado: R$ %.2f",
            saldoAtual, valorSolicitado
        ));
        this.saldoAtual = saldoAtual;
        this.valorSolicitado = valorSolicitado;
    }

    public double getSaldoAtual() { return saldoAtual; }
    public double getValorSolicitado() { return valorSolicitado; }
}

// Uso
public class Conta {
    private double saldo;

    public Conta(double saldo) { this.saldo = saldo; }

    public void sacar(double valor) {
        if (valor > saldo) {
            throw new SaldoInsuficienteException(saldo, valor);
        }
        saldo -= valor;
    }

    public static void main(String[] args) {
        Conta conta = new Conta(500.0);

        try {
            conta.sacar(800.0);
        } catch (SaldoInsuficienteException e) {
            System.out.println(e.getMessage());
            System.out.printf("Faltam R$ %.2f%n",
                e.getValorSolicitado() - e.getSaldoAtual());
        }
    }
}
```

---

## 4.5 Try-with-Resources

**Fecha recursos automaticamente** (arquivos, conexões, etc.):

```java
import java.io.*;
import java.nio.file.*;

public class TryWithResources {
    public static void main(String[] args) {
        // O BufferedReader é fechado automaticamente ao sair do try
        try (BufferedReader reader = new BufferedReader(new FileReader("dados.txt"))) {
            String linha;
            while ((linha = reader.readLine()) != null) {
                System.out.println(linha);
            }
        } catch (IOException e) {
            System.out.println("Erro: " + e.getMessage());
        }

        // Múltiplos recursos
        try (
            FileReader fr = new FileReader("entrada.txt");
            BufferedWriter bw = new BufferedWriter(new FileWriter("saida.txt"))
        ) {
            int c;
            while ((c = fr.read()) != -1) {
                bw.write(c);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

---

## 4.6 I/O com java.nio (Moderno — Java 7+)

```java
import java.io.IOException;
import java.nio.file.*;
import java.util.List;

public class ExemploNIO {
    public static void main(String[] args) throws IOException {
        Path arquivo = Path.of("exemplo.txt");

        // Escrever
        List<String> linhas = List.of(
            "Linha 1: Java é incrível",
            "Linha 2: PHP → Java",
            "Linha 3: Fim do arquivo"
        );
        Files.write(arquivo, linhas);
        System.out.println("Arquivo criado!");

        // Ler tudo de uma vez
        List<String> conteudo = Files.readAllLines(arquivo);
        conteudo.forEach(System.out::println);

        // Ler como texto único
        String texto = Files.readString(arquivo);
        System.out.println("Total: " + texto.length() + " caracteres");

        // Verificar existência
        System.out.println("Existe: " + Files.exists(arquivo));

        // Copiar
        Path destino = Path.of("copia.txt");
        Files.copy(arquivo, destino, StandardCopyOption.REPLACE_EXISTING);

        // Deletar
        Files.deleteIfExists(destino);

        // Listar arquivos de um diretório
        try (var stream = Files.list(Path.of("."))) {
            stream
                .filter(p -> p.toString().endsWith(".txt"))
                .forEach(System.out::println);
        }
    }
}
```

---

## 4.7 Boas Práticas com Exceções

```java
public class BoasPraticas {

    // ❌ ERRADO — capturar e ignorar
    public void errado() {
        try {
            processarDados();
        } catch (Exception e) {
            // silêncio... nunca faça isso!
        }
    }

    // ❌ ERRADO — capturar Exception genérica desnecessariamente
    public void tambemErrado() {
        try {
            int x = Integer.parseInt("123");
        } catch (Exception e) { // muito amplo
            System.out.println(e);
        }
    }

    // ✅ CERTO — capturar o específico, logar corretamente
    public void certo() {
        try {
            int x = Integer.parseInt("abc");
        } catch (NumberFormatException e) {
            System.err.println("Valor não numérico: " + e.getMessage());
            // Em produção: logger.error("Parsing falhou", e);
        }
    }

    // ✅ CERTO — lançar com contexto
    public void carregarConfig(String arquivo) {
        try {
            // ... carrega arquivo
        } catch (IOException e) {
            throw new RuntimeException("Falha ao carregar config: " + arquivo, e);
        }
    }

    private void processarDados() throws Exception {}
}
```

---

## ✅ Exercícios

### Exercício 1 — Calculadora Segura
Crie uma classe `Calculadora` com os 4 operadores. Lance exceções customizadas:
- `DivisaoPorZeroException` para divisão por zero
- `OperacaoInvalidaException` para operador não reconhecido

```java
// Uso esperado:
Calculadora calc = new Calculadora();
System.out.println(calc.calcular(10, "/", 2));  // 5.0
System.out.println(calc.calcular(10, "/", 0));  // lança DivisaoPorZeroException
System.out.println(calc.calcular(10, "$", 2));  // lança OperacaoInvalidaException
```

### Exercício 2 — Leitor de CSV
Crie um arquivo `produtos.csv` com o conteúdo abaixo e leia-o em Java:
```
nome,preco,quantidade
Notebook,3500.00,5
Mouse,89.90,20
Teclado,150.00,15
Monitor,1200.00,8
```
Popule uma `List<Produto>` e imprima o valor total do estoque (preco × quantidade).

### Exercício 3 — Sistema de Log
Implemente uma classe `Logger` que:
- Grava mensagens em arquivo `app.log`
- Adiciona timestamp em cada linha: `[2024-01-15 10:30:45] INFO: mensagem`
- Suporta níveis: INFO, WARN, ERROR
- Usa `try-with-resources`

```java
Logger logger = new Logger("app.log");
logger.info("Aplicação iniciada");
logger.warn("Conexão lenta detectada");
logger.error("Falha ao conectar no banco");
```

### Exercício 4 — Hierarquia de Exceções
Crie um sistema de pedidos com exceções específicas:
- `PedidoException` (base)
  - `EstoqueInsuficienteException`
  - `PagamentoRecusadoException`
  - `PedidoExpiradoException`

Implemente um método `processarPedido()` que pode lançar qualquer uma delas.

### Exercício 5 — Serialização Simples
Crie uma classe `Configuracoes` com campos como `idioma`, `tema` e `volumeMaximo`. Salve e carregue essas configurações em um arquivo `config.txt` no formato `chave=valor`.

---

## 🎯 Gabarito Comentado — Exercício 3 (Logger)

```java
import java.io.*;
import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;

public class Logger {
    private final String arquivo;
    private static final DateTimeFormatter FORMATO =
        DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");

    public Logger(String arquivo) {
        this.arquivo = arquivo;
    }

    private void gravar(String nivel, String mensagem) {
        String linha = String.format("[%s] %s: %s%n",
            LocalDateTime.now().format(FORMATO), nivel, mensagem);

        // try-with-resources fecha o arquivo automaticamente
        try (FileWriter fw = new FileWriter(arquivo, true); // true = append
             BufferedWriter bw = new BufferedWriter(fw)) {
            bw.write(linha);
        } catch (IOException e) {
            System.err.println("Erro ao gravar log: " + e.getMessage());
        }
    }

    public void info(String mensagem)  { gravar("INFO",  mensagem); }
    public void warn(String mensagem)  { gravar("WARN",  mensagem); }
    public void error(String mensagem) { gravar("ERROR", mensagem); }
}
```

---

## ➡️ Próximo módulo: [05 - Concorrência](../05-concorrencia/README.md)
