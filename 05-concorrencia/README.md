# Módulo 05 — Concorrência e Threads

## O que você vai aprender

- O que são Threads e por que existem
- Criando e gerenciando Threads
- Problemas de concorrência (race condition)
- Synchronized e locks
- ExecutorService — pool de threads
- CompletableFuture — programação assíncrona moderna

---

## 5.1 O que é Concorrência?

**Thread** = fluxo de execução independente dentro de um programa.

```
Programa sem concorrência (sequencial):
Tarefa A → Tarefa B → Tarefa C
Tempo total: 3 segundos

Programa com concorrência (paralelo):
Tarefa A ──┐
Tarefa B ──┼── executam ao mesmo tempo
Tarefa C ──┘
Tempo total: ~1 segundo
```

**Quando usar:**
- Processar múltiplas requisições simultâneas (servidores web)
- Tarefas demoradas sem bloquear a UI
- Processamento em batch de grandes volumes
- I/O paralelo (consultar 3 sistemas ao mesmo tempo)

---

## 5.2 Criando Threads

```java
// Forma 1: Implementar Runnable (preferida)
public class TarefaProcessamento implements Runnable {
    private final String nome;

    public TarefaProcessamento(String nome) {
        this.nome = nome;
    }

    @Override
    public void run() {
        System.out.println("Iniciando: " + nome + " | Thread: " + Thread.currentThread().getName());
        try {
            Thread.sleep(1000); // simula processamento
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
        System.out.println("Concluído: " + nome);
    }
}

public class Main {
    public static void main(String[] args) throws InterruptedException {
        // Criar e iniciar threads
        Thread t1 = new Thread(new TarefaProcessamento("Relatório"));
        Thread t2 = new Thread(new TarefaProcessamento("Exportação"));
        Thread t3 = new Thread(() -> System.out.println("Lambda thread!")); // Lambda

        t1.start();
        t2.start();
        t3.start();

        // Aguardar todas terminarem antes de continuar
        t1.join();
        t2.join();
        t3.join();

        System.out.println("Todas as tarefas concluídas!");
    }
}
```

---

## 5.3 ExecutorService — Pool de Threads (Recomendado)

Gerenciar threads manualmente é trabalhoso. Use `ExecutorService`:

```java
import java.util.concurrent.*;
import java.util.List;

public class ExemploExecutor {
    public static void main(String[] args) throws Exception {
        // Pool com 4 threads
        ExecutorService executor = Executors.newFixedThreadPool(4);

        // Submeter tarefas
        for (int i = 1; i <= 10; i++) {
            final int tarefa = i;
            executor.submit(() -> {
                System.out.printf("Processando tarefa %d na thread %s%n",
                    tarefa, Thread.currentThread().getName());
                try { Thread.sleep(500); } catch (InterruptedException e) {}
                return "Resultado " + tarefa;
            });
        }

        // Encerrar o pool (aguarda tarefas pendentes)
        executor.shutdown();
        executor.awaitTermination(30, TimeUnit.SECONDS);
        System.out.println("Pool encerrado!");
    }
}
```

---

## 5.4 CompletableFuture — Assíncrono Moderno

```java
import java.util.concurrent.*;

public class ExemploCompletableFuture {
    public static void main(String[] args) throws Exception {

        // Executar assincronamente
        CompletableFuture<String> futuro = CompletableFuture.supplyAsync(() -> {
            // Simula busca demorada (ex: consulta Oracle)
            try { Thread.sleep(1000); } catch (InterruptedException e) {}
            return "Dados do banco";
        });

        // Enquanto o banco processa, faz outra coisa
        System.out.println("Fazendo outras coisas enquanto o banco processa...");

        // Quando terminar, transforma o resultado
        CompletableFuture<String> processado = futuro
            .thenApply(dados -> dados.toUpperCase())
            .thenApply(dados -> "Processado: " + dados);

        System.out.println(processado.get()); // aguarda e pega o resultado

        // Executar múltiplas consultas em paralelo
        CompletableFuture<String> consultaOracle = CompletableFuture.supplyAsync(
            () -> buscarDoBanco("Oracle")
        );
        CompletableFuture<String> consultaSAP = CompletableFuture.supplyAsync(
            () -> buscarDoBanco("SAP")
        );
        CompletableFuture<String> consultaProtheus = CompletableFuture.supplyAsync(
            () -> buscarDoBanco("Protheus")
        );

        // Aguardar TODAS (paralelo!)
        CompletableFuture.allOf(consultaOracle, consultaSAP, consultaProtheus).join();

        System.out.println("Oracle: " + consultaOracle.get());
        System.out.println("SAP: " + consultaSAP.get());
        System.out.println("Protheus: " + consultaProtheus.get());
    }

    static String buscarDoBanco(String sistema) {
        try { Thread.sleep((long)(Math.random() * 2000)); } catch (InterruptedException e) {}
        return "Dados do " + sistema;
    }
}
```

---

## ⚠️ Cuidados com Concorrência

```java
// PROBLEMA: Race condition
public class ContadorInseguro {
    private int contador = 0;

    public void incrementar() {
        contador++;  // NÃO é thread-safe! (read-modify-write)
    }
}

// SOLUÇÃO 1: synchronized
public class ContadorSeguro {
    private int contador = 0;

    public synchronized void incrementar() {
        contador++;  // só uma thread por vez
    }
}

// SOLUÇÃO 2: AtomicInteger (melhor performance)
import java.util.concurrent.atomic.AtomicInteger;

public class ContadorAtomic {
    private AtomicInteger contador = new AtomicInteger(0);

    public void incrementar() {
        contador.incrementAndGet();  // operação atômica
    }

    public int getValor() { return contador.get(); }
}
```

---

## ✅ Pontos-chave

- Use `ExecutorService` em vez de criar `Thread` manualmente
- `CompletableFuture` para operações assíncronas modernas
- `AtomicInteger`, `AtomicLong` para contadores thread-safe
- No Spring Boot, use `@Async` para métodos assíncronos simples

---

## ✅ Exercícios

### Exercício 1 — Contador Thread-Safe
Crie uma classe `ContadorSeguro` com um `AtomicInteger`. Crie 10 threads que cada uma incrementa o contador 1000 vezes. Verifique que o resultado final é sempre 10.000.

```java
// Resultado esperado:
// Total (deve ser sempre 10000): 10000
```

### Exercício 2 — Pool de Tarefas
Use um `ExecutorService` com 4 threads para processar uma lista de 20 "pedidos" (simulados com `Thread.sleep`). Meça o tempo total e compare com execução sequencial.

### Exercício 3 — CompletableFuture com Múltiplas APIs
Simule chamadas a 3 APIs externas que demoram tempos diferentes (200ms, 500ms, 300ms). Use `CompletableFuture.allOf()` para esperar todas terminarem e medir o tempo total.

```java
// Comportamento esperado:
// Sem async: ~1000ms (200 + 500 + 300)
// Com async: ~500ms (o mais lento define o tempo total)
```

### Exercício 4 — Produtor / Consumidor
Implemente o padrão Produtor/Consumidor com `BlockingQueue`:
- Uma thread produtora gera números de 1 a 10 (com delay de 100ms cada)
- Duas threads consumidoras processam os números (com delay de 150ms cada)
- Use `LinkedBlockingQueue` como buffer

---

## ➡️ Próximo módulo: [06 - Arquiteturas](../06-arquiteturas/README.md)
