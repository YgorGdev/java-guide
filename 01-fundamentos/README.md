# Módulo 01 — Fundamentos da Linguagem Java

## O que você vai aprender

- Como Java funciona por dentro (JVM, JDK, JRE)
- Tipos de dados e variáveis
- Operadores
- Estruturas de controle (if, switch, loops)
- Métodos e passagem de parâmetros
- Arrays básicos

---

## 1.1 Como Java Funciona

```
Código Java (.java)
       ↓  javac (compilador)
  Bytecode (.class)
       ↓  JVM (Java Virtual Machine)
  Execução em qualquer SO
```

**Por que isso importa?**
Você escreve uma vez, roda em qualquer sistema operacional. É o famoso *"Write Once, Run Anywhere"*.

| Conceito | O que é |
|----------|---------|
| **JDK** | Java Development Kit — para desenvolver (você instala isso) |
| **JRE** | Java Runtime Environment — para só executar programas Java |
| **JVM** | Java Virtual Machine — traduz bytecode para o sistema operacional |

> 💡 **Analogia:** o bytecode é como uma partitura musical. A JVM é o músico que toca a partitura — pode ser um músico no Windows, no Linux ou no Mac, mas todos tocam a mesma partitura.

**Comparação com PHP:**
- PHP é interpretado diretamente pelo servidor (linha por linha)
- Java é compilado para bytecode ANTES de rodar
- Java exige declarar o tipo de cada variável; PHP não exige

---

## 1.2 Primeiro Programa

```java
// HelloWorld.java
public class HelloWorld {
    public static void main(String[] args) {
        System.out.println("Olá, Mundo!");
    }
}
```

**Entendendo linha por linha:**
- `public class HelloWorld` → declara uma classe pública chamada HelloWorld
- `public static void main(String[] args)` → o ponto de entrada do programa (como `index.php` em PHP)
- `System.out.println(...)` → imprime no console com quebra de linha
- `System.out.print(...)` → imprime sem quebra de linha

> ⚠️ **Regra de ouro:** o nome do arquivo `.java` deve ser idêntico ao nome da classe pública. `HelloWorld.java` → `public class HelloWorld`.

**Como executar:**
```bash
# Compilar
javac HelloWorld.java

# Executar
java HelloWorld
```

---

## 1.3 Tipos de Dados Primitivos

> 💡 Em PHP você escrevia `$nome = "Ana"` sem definir o tipo.
> Em Java, **toda variável precisa ter seu tipo declarado**. Isso é chamado de **tipagem estática**.

```java
public class Tipos {
    public static void main(String[] args) {
        // === INTEIROS ===
        byte  b  = 127;            // -128 a 127 (1 byte)
        short s  = 32767;          // -32.768 a 32.767 (2 bytes)
        int   i  = 2_000_000;      // -2 bilhões a 2 bilhões (4 bytes) — mais usado
        long  l  = 9_000_000_000L; // muito grande — note o L no final (8 bytes)

        // === DECIMAIS ===
        float  f = 3.14f;    // precisão simples — note o f no final
        double d = 3.14159;  // precisão dupla — padrão para decimais

        // === OUTROS ===
        boolean ativo = true;   // true ou false
        char    letra = 'A';    // caractere único com aspas simples

        // === STRING — NÃO é primitivo, é uma classe! ===
        String nome = "Maria";

        // Imprimir com concatenação
        System.out.println("Int: " + i);
        System.out.println("Double: " + d);
        System.out.println("Nome: " + nome);
        System.out.println("Ativo: " + ativo);

        // Imprimir formatado (como printf)
        System.out.printf("Olá, %s! Você tem %d anos e %.2f de altura.%n", nome, 25, 1.75);
    }
}
```

**Quando usar cada um?**
| Tipo | Use quando... |
|------|---------------|
| `int` | Números inteiros no geral (quantidade, ID, índice) |
| `long` | Números muito grandes (timestamp, CPF, telefone com código) |
| `double` | Valores decimais (preço, peso, temperatura) |
| `boolean` | Flags verdadeiro/falso (ativo, aprovado, existe) |
| `String` | Textos em geral |

---

## 1.4 Variáveis e Constantes

```java
public class VariaveisConstantes {
    // Constante — final + MAIÚSCULAS é convenção
    public static final double PI = 3.14159;
    public static final int LIMITE_TENTATIVAS = 3;

    public static void main(String[] args) {
        // Variável comum — pode ser reatribuída
        int contador = 0;
        contador = 5;      // OK
        contador++;        // incremento

        // Variável final — não pode ser reatribuída depois de definida
        final String NOME_SISTEMA = "Java Guide";
        // NOME_SISTEMA = "outro"; // ← ERRO de compilação!

        // Inferência de tipo com var (Java 10+)
        var mensagem = "Olá!";       // compilador infere String
        var numero = 42;             // compilador infere int
        var lista = new java.util.ArrayList<String>(); // infere ArrayList

        System.out.println(PI);
        System.out.println(mensagem);
        System.out.println(numero);
    }
}
```

---

## 1.5 Operadores

```java
public class Operadores {
    public static void main(String[] args) {
        int a = 10, b = 3;

        // === ARITMÉTICOS ===
        System.out.println(a + b);   // 13 — soma
        System.out.println(a - b);   // 7  — subtração
        System.out.println(a * b);   // 30 — multiplicação
        System.out.println(a / b);   // 3  — divisão INTEIRA (perde o decimal!)
        System.out.println(a % b);   // 1  — resto da divisão (módulo)

        // Para obter decimal na divisão, use cast
        System.out.println((double) a / b); // 3.3333...

        // === ATRIBUIÇÃO COMPOSTA ===
        int x = 10;
        x += 5;   // x = x + 5 = 15
        x -= 3;   // x = x - 3 = 12
        x *= 2;   // x = x * 2 = 24
        x /= 4;   // x = x / 4 = 6
        x %= 4;   // x = x % 4 = 2
        System.out.println("x final: " + x); // 2

        // === COMPARAÇÃO — retornam boolean ===
        System.out.println(a > b);   // true
        System.out.println(a < b);   // false
        System.out.println(a >= 10); // true
        System.out.println(a <= 10); // true
        System.out.println(a == b);  // false
        System.out.println(a != b);  // true

        // ⚠️ ATENÇÃO: para Strings, use .equals(), NÃO ==
        String s1 = "Java";
        String s2 = "Java";
        System.out.println(s1 == s2);       // true (pode variar!)
        System.out.println(s1.equals(s2));  // true (sempre correto)

        // === LÓGICOS ===
        boolean p = true, q = false;
        System.out.println(p && q);  // false — AND (ambos precisam ser true)
        System.out.println(p || q);  // true  — OR (pelo menos um true)
        System.out.println(!p);      // false — NOT (inverte)

        // === INCREMENTO / DECREMENTO ===
        int contador = 5;
        System.out.println(contador++); // imprime 5, depois vira 6
        System.out.println(++contador); // vira 7, depois imprime 7
        System.out.println(contador--); // imprime 7, depois vira 6
    }
}
```

---

## 1.6 Estruturas de Controle

### if / else if / else

```java
public class ControleFluxo {
    public static void main(String[] args) {
        int nota = 75;

        if (nota >= 90) {
            System.out.println("Conceito A — Excelente!");
        } else if (nota >= 80) {
            System.out.println("Conceito B — Ótimo!");
        } else if (nota >= 70) {
            System.out.println("Conceito C — Aprovado!");  // ← este executa
        } else if (nota >= 50) {
            System.out.println("Conceito D — Recuperação");
        } else {
            System.out.println("Conceito F — Reprovado");
        }

        // Operador ternário — forma compacta para if/else simples
        // condição ? valor_se_verdadeiro : valor_se_falso
        String resultado = nota >= 70 ? "Aprovado" : "Reprovado";
        System.out.println(resultado); // Aprovado

        // Verificar nulo antes de usar
        String nome = null;
        if (nome != null && !nome.isEmpty()) {
            System.out.println("Nome: " + nome);
        } else {
            System.out.println("Nome não informado");
        }
    }
}
```

### switch (clássico e moderno)

```java
public class Switch {
    public static void main(String[] args) {
        int mes = 3;

        // Switch tradicional
        switch (mes) {
            case 1:  System.out.println("Janeiro");   break;
            case 2:  System.out.println("Fevereiro"); break;
            case 3:  System.out.println("Março");     break; // ← executa
            case 12: System.out.println("Dezembro");  break;
            default: System.out.println("Mês inválido");
        }

        // Switch moderno com expressão (Java 14+) — mais limpo e seguro
        String nomeMes = switch (mes) {
            case 1  -> "Janeiro";
            case 2  -> "Fevereiro";
            case 3  -> "Março";
            case 4  -> "Abril";
            case 5  -> "Maio";
            case 6  -> "Junho";
            case 7  -> "Julho";
            case 8  -> "Agosto";
            case 9  -> "Setembro";
            case 10 -> "Outubro";
            case 11 -> "Novembro";
            case 12 -> "Dezembro";
            default -> "Inválido";
        };
        System.out.println("Mês: " + nomeMes);

        // Switch com múltiplos casos na mesma linha
        String diaSemana = "SEGUNDA";
        String tipoDia = switch (diaSemana) {
            case "SEGUNDA", "TERÇA", "QUARTA", "QUINTA", "SEXTA" -> "Dia útil";
            case "SÁBADO", "DOMINGO" -> "Final de semana";
            default -> "Dia inválido";
        };
        System.out.println(tipoDia);
    }
}
```

### Loops (for, while, do-while)

```java
public class Loops {
    public static void main(String[] args) {

        // === FOR clássico ===
        // for (início; condição; passo)
        System.out.println("--- For ---");
        for (int i = 1; i <= 5; i++) {
            System.out.println("Iteração " + i);
        }

        // For decrescente
        for (int i = 10; i >= 1; i--) {
            System.out.print(i + " ");
        }
        System.out.println();

        // === WHILE — quando não sabe quantas vezes vai repetir ===
        System.out.println("--- While ---");
        int tentativas = 0;
        int senhaCorreta = 1234;
        int senhaDigitada = 0;

        while (senhaDigitada != senhaCorreta && tentativas < 3) {
            tentativas++;
            senhaDigitada = 1234; // simulando digitação correta
            System.out.println("Tentativa " + tentativas);
        }

        // === DO-WHILE — executa pelo menos uma vez ===
        System.out.println("--- Do-While ---");
        int numero = 1;
        do {
            System.out.println("Número: " + numero);
            numero *= 2;
        } while (numero <= 16);
        // Imprime: 1, 2, 4, 8, 16

        // === FOR-EACH — para arrays e coleções ===
        System.out.println("--- For-Each ---");
        String[] linguagens = {"Java", "PHP", "Python", "JavaScript"};
        for (String lang : linguagens) {
            System.out.println("Linguagem: " + lang);
        }

        // === break e continue ===
        System.out.println("--- Break e Continue ---");
        for (int i = 1; i <= 10; i++) {
            if (i % 2 == 0) continue;  // pula os pares
            if (i > 7) break;          // para quando passa de 7
            System.out.print(i + " "); // imprime: 1 3 5 7
        }
        System.out.println();
    }
}
```

---

## 1.7 Métodos

> 💡 Em PHP você usava `function minhaFuncao($param)`. Em Java é `tipoRetorno nomeMetodo(TipoParam param)`.

```java
public class Metodos {

    // Método sem retorno (void)
    public static void exibirBoasVindas(String nome) {
        System.out.println("Bem-vindo, " + nome + "!");
        System.out.println("Hora: " + java.time.LocalTime.now());
    }

    // Método com retorno
    public static int somar(int a, int b) {
        return a + b;
    }

    // Sobrecarga — mesmo nome, parâmetros diferentes
    public static double somar(double a, double b) {
        return a + b;
    }

    // Método que retorna boolean
    public static boolean ehPrimo(int n) {
        if (n < 2) return false;
        for (int i = 2; i <= Math.sqrt(n); i++) {
            if (n % i == 0) return false;
        }
        return true;
    }

    // Número variável de argumentos (varargs) — como ... spread em JS
    public static int somarTodos(int... numeros) {
        int total = 0;
        for (int n : numeros) {
            total += n;
        }
        return total;
    }

    // Método recursivo — chama a si mesmo
    public static int fatorial(int n) {
        if (n <= 1) return 1;        // caso base
        return n * fatorial(n - 1);  // chamada recursiva
    }

    public static void main(String[] args) {
        exibirBoasVindas("Ygor");

        System.out.println(somar(3, 4));         // 7 (int)
        System.out.println(somar(3.5, 4.2));     // 7.7 (double)

        System.out.println(ehPrimo(7));           // true
        System.out.println(ehPrimo(10));          // false

        System.out.println(somarTodos(1, 2, 3));  // 6
        System.out.println(somarTodos(10, 20, 30, 40, 50)); // 150

        System.out.println(fatorial(5)); // 120 (5*4*3*2*1)
        System.out.println(fatorial(0)); // 1
    }
}
```

---

## 1.8 Arrays

```java
public class Arrays {
    public static void main(String[] args) {

        // === DECLARAÇÃO E INICIALIZAÇÃO ===
        int[] notas = {8, 7, 9, 6, 10};

        // Declaração com tamanho — valores padrão: 0, false, null
        double[] precos = new double[5];
        precos[0] = 29.90;
        precos[1] = 49.90;

        // === ACESSO — índice começa em 0 ===
        System.out.println(notas[0]);        // 8 (primeiro)
        System.out.println(notas[notas.length - 1]); // 10 (último)
        System.out.println(notas.length);    // 5 (tamanho)

        // ⚠️ Cuidado com ArrayIndexOutOfBoundsException
        // notas[10] → ERRO em tempo de execução

        // === PERCORRER E CALCULAR ===
        int soma = 0;
        int maior = notas[0];
        for (int nota : notas) {
            soma += nota;
            if (nota > maior) maior = nota;
        }
        double media = (double) soma / notas.length;
        System.out.printf("Soma: %d | Maior: %d | Média: %.1f%n", soma, maior, media);

        // === ORDENAR (usa a classe Arrays) ===
        java.util.Arrays.sort(notas);
        System.out.println(java.util.Arrays.toString(notas)); // [6, 7, 8, 9, 10]

        // === ARRAY BIDIMENSIONAL (matriz) ===
        int[][] tabela = {
            {1, 2, 3},
            {4, 5, 6},
            {7, 8, 9}
        };
        System.out.println(tabela[0][0]); // 1 (linha 0, coluna 0)
        System.out.println(tabela[1][2]); // 6 (linha 1, coluna 2)
        System.out.println(tabela[2][2]); // 9 (linha 2, coluna 2)

        // Percorrer matriz
        for (int[] linha : tabela) {
            for (int valor : linha) {
                System.out.printf("%3d", valor);
            }
            System.out.println();
        }
    }
}
```

---

## 1.9 Strings — Operações Essenciais

```java
public class OperacoesString {
    public static void main(String[] args) {
        String texto = "  Olá, Mundo!  ";

        // Tamanho
        System.out.println(texto.length()); // 15 (inclui espaços)

        // Remover espaços das bordas
        System.out.println(texto.trim());   // "Olá, Mundo!"

        // Converter case
        System.out.println("java".toUpperCase()); // JAVA
        System.out.println("JAVA".toLowerCase()); // java

        // Verificar conteúdo
        String email = "usuario@empresa.com";
        System.out.println(email.contains("@"));          // true
        System.out.println(email.startsWith("usuario"));  // true
        System.out.println(email.endsWith(".com"));        // true

        // Buscar e substituir
        String frase = "PHP é legal, PHP é rápido";
        System.out.println(frase.replace("PHP", "Java")); // substitui todas

        // Dividir string (split)
        String csv = "Ana,Bruno,Carlos,Diana";
        String[] nomes = csv.split(",");
        for (String nome : nomes) {
            System.out.println(nome.trim());
        }

        // Pegar parte da string
        String data = "2024-12-25";
        System.out.println(data.substring(0, 4));  // "2024"
        System.out.println(data.substring(5, 7));  // "12"
        System.out.println(data.substring(8));     // "25"

        // Verificar vazio
        String vazia = "";
        System.out.println(vazia.isEmpty());        // true
        System.out.println("  ".isBlank());         // true (Java 11+)

        // StringBuilder — mais eficiente para concatenação em loop
        StringBuilder sb = new StringBuilder();
        for (int i = 1; i <= 5; i++) {
            sb.append("Item ").append(i).append("\n");
        }
        System.out.println(sb.toString());

        // String.format — formatar texto
        String resultado = String.format("Nome: %-10s | Nota: %.2f", "Ana", 9.5);
        System.out.println(resultado); // Nome: Ana        | Nota: 9.50
    }
}
```

---

## ✅ Exercícios

### Exercício 1 — IMC
Crie um programa que calcule o IMC (peso / altura²) e classifique:
- Abaixo de 18.5 → Abaixo do peso
- 18.5 a 24.9 → Peso normal
- 25.0 a 29.9 → Sobrepeso
- 30.0 ou mais → Obesidade

```java
// Dica: use variáveis double para peso e altura
double peso = 70.0;
double altura = 1.75;
double imc = peso / (altura * altura);
```

### Exercício 2 — FizzBuzz
Imprima os números de 1 a 100, mas:
- Múltiplos de 3 → imprima "Fizz"
- Múltiplos de 5 → imprima "Buzz"
- Múltiplos de 3 e 5 → imprima "FizzBuzz"
- Outros → imprima o número

```java
// Dica: verifique o múltiplo de 15 PRIMEIRO
for (int i = 1; i <= 100; i++) {
    if (i % 15 == 0) { ... }
    else if (i % 3 == 0) { ... }
    ...
}
```

### Exercício 3 — Estatísticas de Array
Crie um método `estatisticas(int[] numeros)` que retorne (como print) a soma, média, maior e menor valor.

```java
// Exemplo de saída esperada:
// Soma: 55 | Média: 5.5 | Maior: 10 | Menor: 1
```

### Exercício 4 — Número Primo
Crie um método `listarPrimos(int limite)` que imprima todos os números primos até o limite dado.

```java
// Exemplo: listarPrimos(20)
// Saída: 2 3 5 7 11 13 17 19
```

### Exercício 5 — Tabuada
Crie um programa que imprima a tabuada completa de 1 a 10 usando loops aninhados:
```
1 x 1 = 1   |   2 x 1 = 2   |   ...
1 x 2 = 2   |   2 x 2 = 4   |   ...
```

### Exercício 6 — Palindromo
Crie um método `ehPalindromo(String texto)` que retorne `true` se a palavra for igual de trás para frente (ex: "arara", "racecar").

```java
// Dica: compare o texto com sua versão invertida
// StringBuilder tem um método .reverse()
```

### Exercício 7 — Calculadora
Crie uma calculadora com um método `calcular(double a, String operacao, double b)` que suporte `+`, `-`, `*`, `/` e retorne o resultado. Lance uma exceção para divisão por zero.

---

## 🎯 Gabarito Comentado — Exercício 1 (IMC)

```java
public class IMC {
    public static void main(String[] args) {
        double peso = 70.0;
        double altura = 1.75;

        double imc = peso / (altura * altura);

        String classificacao;
        if (imc < 18.5) {
            classificacao = "Abaixo do peso";
        } else if (imc < 25.0) {
            classificacao = "Peso normal";
        } else if (imc < 30.0) {
            classificacao = "Sobrepeso";
        } else {
            classificacao = "Obesidade";
        }

        System.out.printf("IMC: %.2f — %s%n", imc, classificacao);
        // IMC: 22.86 — Peso normal
    }
}
```

---

## ➡️ Próximo módulo: [02 - Programação Orientada a Objetos](../02-poo/README.md)
