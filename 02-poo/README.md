# Módulo 02 — Programação Orientada a Objetos (POO)

## O que você vai aprender

- O que é POO e por que usar
- Classes e Objetos
- Encapsulamento
- Herança
- Polimorfismo
- Interfaces e Classes Abstratas
- Modificadores de acesso
- Records (Java 16+)

---

## 2.1 O que é POO?

POO é uma forma de organizar o código **modelando o mundo real em objetos** que têm:
- **Atributos** → o que o objeto *tem* (dados/estado)
- **Métodos** → o que o objeto *faz* (comportamentos/ações)

**Por que usar POO?**
- Código mais organizado e reutilizável
- Mais fácil de manter e evoluir
- Modela problemas reais de forma natural
- Base de praticamente todos os frameworks Java

**Analogia do mundo real:**
```
Carro (Classe — o molde/planta)
├── Atributos: marca, modelo, cor, velocidade
└── Métodos: acelerar(), frear(), ligar(), desligar()

Meu Carro (Objeto — a instância concreta)
├── marca = "Toyota"
├── modelo = "Corolla"
└── cor = "Prata"

Seu Carro (Outro objeto — mesma classe, dados diferentes)
├── marca = "Honda"
├── modelo = "Civic"
└── cor = "Preto"
```

> 💡 A classe é como uma forma de bolo. Os objetos são os bolos que você faz com ela — cada um pode ter sabores (dados) diferentes.

**Comparação PHP → Java:**
```php
// PHP — sem classe formal
$carro = ["marca" => "Toyota", "modelo" => "Corolla"];
function acelerar($carro, $incremento) {
    $carro["velocidade"] += $incremento;
    return $carro;
}
```

```java
// Java — com classe formal
Carro carro = new Carro("Toyota", "Corolla", "Prata");
carro.acelerar(60); // o objeto sabe como se comportar
```

---

## 2.2 Classes e Objetos

```java
// Arquivo: Carro.java
public class Carro {

    // Atributos (fields) — privados por padrão
    private String marca;
    private String modelo;
    private String cor;
    private int velocidade;

    // Construtor — executado quando criamos um objeto com "new"
    public Carro(String marca, String modelo, String cor) {
        this.marca = marca;     // "this" distingue atributo de parâmetro
        this.modelo = modelo;
        this.cor = cor;
        this.velocidade = 0;    // começa parado
    }

    // Métodos (comportamentos)
    public void acelerar(int incremento) {
        if (incremento > 0) {
            velocidade += incremento;
            System.out.println(marca + " acelerou para " + velocidade + " km/h");
        }
    }

    public void frear(int decremento) {
        velocidade = Math.max(0, velocidade - decremento); // não fica negativo
        System.out.println(marca + " freou para " + velocidade + " km/h");
    }

    public void ligar() {
        System.out.println(marca + " " + modelo + " ligado! 🚗");
    }

    // Getters — para ler atributos privados
    public String getMarca()     { return marca; }
    public String getModelo()    { return modelo; }
    public String getCor()       { return cor; }
    public int    getVelocidade(){ return velocidade; }

    // Setters — para modificar com validação
    public void setCor(String cor) {
        if (cor != null && !cor.isBlank()) {
            this.cor = cor;
        }
    }

    // toString — representação em texto (chamado automaticamente no println)
    @Override
    public String toString() {
        return String.format("%s %s (%s) — %d km/h", marca, modelo, cor, velocidade);
    }
}
```

```java
// Arquivo: Main.java
public class Main {
    public static void main(String[] args) {
        // Criar objetos (instâncias) com "new"
        Carro meuCarro = new Carro("Toyota", "Corolla", "Prata");
        Carro outroCarro = new Carro("Honda", "Civic", "Preto");

        // Usar os métodos
        meuCarro.ligar();
        meuCarro.acelerar(60);
        meuCarro.acelerar(40);  // agora está a 100 km/h
        meuCarro.frear(30);     // agora está a 70 km/h

        // toString chamado automaticamente
        System.out.println(meuCarro);    // Toyota Corolla (Prata) — 70 km/h
        System.out.println(outroCarro);  // Honda Civic (Preto) — 0 km/h

        // Acessar via getter
        System.out.println("Marca: " + meuCarro.getMarca());
    }
}
```

---

## 2.3 Encapsulamento

**Princípio:** oculte os detalhes internos. Exponha só o que é necessário.

> 💡 É como um carro: você usa o volante e os pedais (interface pública) sem precisar saber como o motor funciona por dentro (implementação privada).

**Por que encapsular?**
- Protege dados de serem corrompidos
- Permite mudar a implementação sem quebrar quem usa
- Facilita validações e regras de negócio

```java
// SEM encapsulamento — perigoso!
public class ContaRuim {
    public double saldo = 1000.0; // qualquer um pode modificar diretamente!
}
// main: conta.saldo = -99999; // válido! Saldo negativo sem nenhuma validação.

// COM encapsulamento — seguro
public class ContaBancaria {
    private String titular;
    private double saldo;        // privado — ninguém acessa direto
    private String numeroConta;

    public ContaBancaria(String titular, double saldoInicial) {
        this.titular = titular;
        this.numeroConta = gerarNumeroConta();
        this.saldo = saldoInicial > 0 ? saldoInicial : 0; // nunca negativo
    }

    private String gerarNumeroConta() {
        return String.format("%08d", (int)(Math.random() * 100000000));
    }

    // Acesso somente leitura ao saldo
    public double getSaldo() {
        return saldo;
    }

    public String getTitular() {
        return titular;
    }

    // Operações controladas com validação
    public void depositar(double valor) {
        if (valor <= 0) {
            throw new IllegalArgumentException("Valor de depósito deve ser positivo");
        }
        saldo += valor;
        System.out.printf("Depósito: R$ %.2f | Saldo atual: R$ %.2f%n", valor, saldo);
    }

    public void sacar(double valor) {
        if (valor <= 0) {
            throw new IllegalArgumentException("Valor de saque deve ser positivo");
        }
        if (valor > saldo) {
            throw new IllegalStateException("Saldo insuficiente. Disponível: R$ " + saldo);
        }
        saldo -= valor;
        System.out.printf("Saque: R$ %.2f | Saldo atual: R$ %.2f%n", valor, saldo);
    }

    public void transferir(ContaBancaria destino, double valor) {
        this.sacar(valor);         // valida e debita desta conta
        destino.depositar(valor);  // credita na conta destino
        System.out.printf("Transferência de %s para %s: R$ %.2f%n",
            this.titular, destino.getTitular(), valor);
    }

    @Override
    public String toString() {
        return String.format("Conta %s | Titular: %s | Saldo: R$ %.2f",
            numeroConta, titular, saldo);
    }
}
```

```java
public class TesteConta {
    public static void main(String[] args) {
        ContaBancaria conta1 = new ContaBancaria("Ana", 1000.0);
        ContaBancaria conta2 = new ContaBancaria("Bruno", 500.0);

        conta1.depositar(500.0);
        conta1.sacar(200.0);
        conta1.transferir(conta2, 300.0);

        System.out.println(conta1);
        System.out.println(conta2);

        // conta1.saldo = -99999; // ← ERRO de compilação! Protegido.
    }
}
```

---

## 2.4 Herança

**Princípio:** uma classe filha **herda** atributos e métodos da classe mãe e pode adicionar os seus próprios.

> 💡 É como herdar características dos pais. Você herda traços deles (herança), mas também tem características próprias.

```java
// Classe mãe (superclasse)
public class Veiculo {
    protected String marca;      // protected = visível para subclasses
    protected String modelo;
    protected int ano;
    protected boolean ligado;

    public Veiculo(String marca, String modelo, int ano) {
        this.marca = marca;
        this.modelo = modelo;
        this.ano = ano;
        this.ligado = false;
    }

    public void ligar() {
        ligado = true;
        System.out.println(marca + " ligado!");
    }

    public void desligar() {
        ligado = false;
        System.out.println(marca + " desligado.");
    }

    public String getInfo() {
        return String.format("%s %s (%d)", marca, modelo, ano);
    }

    @Override
    public String toString() {
        return getInfo() + (ligado ? " [LIGADO]" : " [DESLIGADO]");
    }
}
```

```java
// Classe filha 1 — herda de Veiculo
public class Moto extends Veiculo {
    private boolean temCarenagem;
    private int cilindradas;

    public Moto(String marca, String modelo, int ano, int cilindradas, boolean temCarenagem) {
        super(marca, modelo, ano);  // OBRIGATÓRIO: chamar construtor da mãe primeiro
        this.cilindradas = cilindradas;
        this.temCarenagem = temCarenagem;
    }

    // Método exclusivo da Moto
    public void empinar() {
        if (!ligado) {
            System.out.println("Ligue a moto primeiro!");
            return;
        }
        System.out.println(marca + " empinando! 🏍️");
    }

    // Sobrescrita (@Override) — redefinindo comportamento da mãe
    @Override
    public String getInfo() {
        String tipo = temCarenagem ? "esportiva" : "custom";
        return super.getInfo() + " — " + cilindradas + "cc (" + tipo + ")";
    }
}
```

```java
// Classe filha 2 — herda de Veiculo
public class Caminhao extends Veiculo {
    private double capacidadeToneladas;
    private int numeroEixos;

    public Caminhao(String marca, String modelo, int ano, double capacidade, int eixos) {
        super(marca, modelo, ano);
        this.capacidadeToneladas = capacidade;
        this.numeroEixos = eixos;
    }

    public void carregar(double pesoToneladas) {
        if (pesoToneladas > capacidadeToneladas) {
            System.out.printf("⚠️ Excesso! Capacidade: %.1ft | Tentativa: %.1ft%n",
                capacidadeToneladas, pesoToneladas);
        } else {
            System.out.printf("✅ Carregando %.1ft no %s (%.0f%% da capacidade)%n",
                pesoToneladas, marca, (pesoToneladas / capacidadeToneladas) * 100);
        }
    }

    @Override
    public String getInfo() {
        return super.getInfo() + " — " + numeroEixos + " eixos, " + capacidadeToneladas + "t";
    }
}
```

```java
public class TesteVeiculos {
    public static void main(String[] args) {
        Moto moto = new Moto("Honda", "CBR 600", 2023, 600, true);
        Caminhao caminhao = new Caminhao("Volvo", "FH", 2022, 25.0, 6);

        moto.ligar();             // herdado de Veiculo
        moto.empinar();           // específico de Moto
        System.out.println(moto.getInfo()); // sobrescrito

        System.out.println("---");

        caminhao.ligar();
        caminhao.carregar(20.0);
        caminhao.carregar(30.0);  // excesso!
        System.out.println(caminhao);

        // instanceof — verifica tipo
        System.out.println(moto instanceof Veiculo);    // true (moto É UM veículo)
        System.out.println(moto instanceof Moto);       // true
        System.out.println(moto instanceof Caminhao);   // false
    }
}
```

---

## 2.5 Polimorfismo

**Princípio:** objetos diferentes respondem ao **mesmo método** de formas diferentes.

> 💡 É como pedir "fale!" para diferentes animais — cada um faz o som do jeito dele.

```java
// Classe abstrata — template, não pode ser instanciada diretamente
public abstract class Animal {
    protected String nome;
    protected String especie;

    public Animal(String nome, String especie) {
        this.nome = nome;
        this.especie = especie;
    }

    // Método abstrato — toda subclasse DEVE implementar (como um contrato)
    public abstract void fazerSom();

    // Método concreto — compartilhado por todos
    public void dormir() {
        System.out.println(nome + " (" + especie + ") está dormindo... 💤");
    }

    public void apresentar() {
        System.out.println("Eu sou " + nome + ", um(a) " + especie);
        fazerSom();  // polimorfismo — chama o método correto de cada subclasse
    }
}
```

```java
public class Cachorro extends Animal {
    public Cachorro(String nome) {
        super(nome, "Cachorro");
    }

    @Override
    public void fazerSom() {
        System.out.println(nome + ": Au au! 🐕");
    }

    public void buscarBola() {
        System.out.println(nome + " foi buscar a bola! 🎾");
    }
}

public class Gato extends Animal {
    public Gato(String nome) {
        super(nome, "Gato");
    }

    @Override
    public void fazerSom() {
        System.out.println(nome + ": Miau! 🐱");
    }
}

public class Papagaio extends Animal {
    private String[] palavras;

    public Papagaio(String nome, String... palavras) {
        super(nome, "Papagaio");
        this.palavras = palavras;
    }

    @Override
    public void fazerSom() {
        for (String palavra : palavras) {
            System.out.println(nome + ": " + palavra + "! 🦜");
        }
    }
}
```

```java
import java.util.ArrayList;
import java.util.List;

public class TestePet {
    public static void main(String[] args) {
        // Lista de Animal — mas cada objeto é de um tipo diferente
        List<Animal> animais = new ArrayList<>();
        animais.add(new Cachorro("Rex"));
        animais.add(new Gato("Whiskers"));
        animais.add(new Papagaio("Loro", "Olá", "Quero biscoito", "Polly quer comida"));
        animais.add(new Cachorro("Bolinha"));

        // Polimorfismo em ação — cada objeto chama seu próprio fazerSom()
        System.out.println("=== Todos os animais ===");
        for (Animal animal : animais) {
            animal.apresentar();
            System.out.println();
        }

        // Verificar tipo e fazer cast
        for (Animal animal : animais) {
            if (animal instanceof Cachorro cachorro) { // pattern matching Java 16+
                cachorro.buscarBola();
            }
        }
    }
}
```

---

## 2.6 Interfaces

**Interface** = contrato. Define *o que* uma classe deve fazer, não *como* ela faz.

> 💡 Uma tomada elétrica é uma "interface": ela define o formato dos pinos e a tensão, mas não importa se o aparelho é um liquidificador ou um notebook.

```java
// Interface — apenas assinaturas de métodos (o "contrato")
public interface Pagavel {
    double calcularTotal();
    String gerarRecibo();

    // Método default (Java 8+) — tem implementação padrão
    default String formatarMoeda(double valor) {
        return String.format("R$ %.2f", valor);
    }
}

public interface Notificavel {
    void enviarEmail(String destinatario, String mensagem);
    void enviarSMS(String telefone, String mensagem);
}

public interface Auditavel {
    String getDataCriacao();
    String getUltimaModificacao();
}
```

```java
import java.time.LocalDateTime;

// Uma classe pode implementar MÚLTIPLAS interfaces
public class Pedido implements Pagavel, Notificavel, Auditavel {
    private long id;
    private String cliente;
    private String email;
    private String telefone;
    private double valorBase;
    private double desconto;
    private LocalDateTime dataCriacao;

    public Pedido(long id, String cliente, String email, String telefone,
                  double valorBase, double desconto) {
        this.id = id;
        this.cliente = cliente;
        this.email = email;
        this.telefone = telefone;
        this.valorBase = valorBase;
        this.desconto = desconto;
        this.dataCriacao = LocalDateTime.now();
    }

    // Implementação de Pagavel
    @Override
    public double calcularTotal() {
        return valorBase * (1 - desconto / 100);
    }

    @Override
    public String gerarRecibo() {
        return String.format(
            "Pedido #%d | Cliente: %s | Total: %s",
            id, cliente, formatarMoeda(calcularTotal()) // usa o método default!
        );
    }

    // Implementação de Notificavel
    @Override
    public void enviarEmail(String destinatario, String mensagem) {
        System.out.println("📧 Email para " + destinatario + ": " + mensagem);
    }

    @Override
    public void enviarSMS(String telefone, String mensagem) {
        System.out.println("📱 SMS para " + telefone + ": " + mensagem);
    }

    // Implementação de Auditavel
    @Override
    public String getDataCriacao() {
        return dataCriacao.toString();
    }

    @Override
    public String getUltimaModificacao() {
        return LocalDateTime.now().toString();
    }

    // Método de negócio
    public void confirmar() {
        String recibo = gerarRecibo();
        enviarEmail(email, "Pedido confirmado! " + recibo);
        enviarSMS(telefone, "Pedido #" + id + " confirmado. Total: " + formatarMoeda(calcularTotal()));
    }
}
```

---

## 2.7 Modificadores de Acesso

| Modificador | Mesma Classe | Mesmo Pacote | Subclasse | Qualquer Lugar |
|-------------|:---:|:---:|:---:|:---:|
| `private`   | ✅ | ❌ | ❌ | ❌ |
| *(padrão)*  | ✅ | ✅ | ❌ | ❌ |
| `protected` | ✅ | ✅ | ✅ | ❌ |
| `public`    | ✅ | ✅ | ✅ | ✅ |

**Regra prática:**
- Atributos → sempre `private` (use getters/setters para acesso)
- Métodos internos → `private`
- Métodos herdáveis → `protected`
- API pública → `public`
- Constantes → `public static final`

```java
public class ExemploAcesso {
    private   int atributoPrivado   = 1; // só esta classe
    int           atributoPacote    = 2; // classe + mesmo pacote
    protected int atributoProtegido = 3; // classe + pacote + subclasses
    public    int atributoPublico   = 4; // todos

    private void metodoInterno() { }   // só esta classe usa
    public void metodoPublico()  { }   // qualquer classe usa
}
```

---

## 2.8 Records (Java 16+) — POO Moderna para DTOs

Para classes que **só guardam dados** (DTOs, Value Objects), use `record`:

```java
// Sem record — verboso (precisa de tudo isso!)
public class PessoaOld {
    private final String nome;
    private final String cpf;
    private final int idade;

    public PessoaOld(String nome, String cpf, int idade) {
        this.nome = nome;
        this.cpf = cpf;
        this.idade = idade;
    }

    public String getNome()  { return nome; }
    public String getCpf()   { return cpf; }
    public int    getIdade() { return idade; }

    @Override
    public String toString() {
        return "Pessoa[nome=" + nome + ", cpf=" + cpf + ", idade=" + idade + "]";
    }

    @Override
    public boolean equals(Object o) { /* ... muito código ... */ return false; }

    @Override
    public int hashCode() { /* ... muito código ... */ return 0; }
}

// Com record — compacto e completo!
// Java gera automaticamente: construtor, getters, toString, equals, hashCode
public record Pessoa(String nome, String cpf, int idade) {}

// Uso
Pessoa p = new Pessoa("Ana", "123.456.789-00", 30);
System.out.println(p.nome());   // "Ana" (sem "get")
System.out.println(p.cpf());    // "123.456.789-00"
System.out.println(p);          // Pessoa[nome=Ana, cpf=123.456.789-00, idade=30]

// Records são imutáveis — não dá para modificar os valores depois de criado
// p.nome = "outra coisa"; // ← ERRO!
```

**Quando usar Record vs Classe normal?**
| Use Record | Use Classe |
|------------|------------|
| Dados de resposta de API (DTO) | Entidades do banco com lógica |
| Configurações imutáveis | Objetos que mudam de estado |
| Coordenadas, pontos | Objetos com comportamento complexo |
| Resultados de consultas | Classes com herança |

---

## ✅ Exercícios

### Exercício 1 — Funcionário
Crie uma classe `Funcionario` com atributos `nome`, `cargo`, `salario` e `ativo`. Implemente:
- Método `receberAumento(double percentual)` que aumenta o salário
- Método `demitir()` que muda `ativo` para false
- `toString()` legível

```java
Funcionario f = new Funcionario("Carlos", "Analista", 5000.0);
f.receberAumento(10); // salário vira 5500.0
System.out.println(f);
```

### Exercício 2 — Formas Geométricas
Crie uma classe abstrata `Forma` com métodos abstratos `calcularArea()` e `calcularPerimetro()`. Implemente:
- `Circulo` (raio)
- `Retangulo` (largura, altura)
- `Triangulo` (base, altura, lado1, lado2, lado3)

```java
// Resultado esperado:
Forma[] formas = { new Circulo(5), new Retangulo(4, 6), new Triangulo(3, 4, 3, 4, 5) };
for (Forma f : formas) {
    System.out.printf("Área: %.2f | Perímetro: %.2f%n", f.calcularArea(), f.calcularPerimetro());
}
```

### Exercício 3 — Interface Exportável
Crie uma interface `Exportavel` com métodos `exportarJSON()` e `exportarCSV()`. Implemente em uma classe `Produto(nome, preco, quantidade)`.

### Exercício 4 — Sistema de Notas
Crie uma classe `Aluno` com lista de notas. Adicione métodos para:
- Adicionar nota
- Calcular média
- Verificar se foi aprovado (média >= 7)
- Listar notas com formatação

### Exercício 5 — Conta Bancária com Herança
Estenda a classe `ContaBancaria` do exemplo anterior com:
- `ContaCorrente` (tem limite de cheque especial)
- `ContaPoupanca` (tem método `renderJuros(double taxa)`)

---

## 🎯 Gabarito Comentado — Exercício 2 (Formas)

```java
// Forma.java
public abstract class Forma {
    public abstract double calcularArea();
    public abstract double calcularPerimetro();

    // toString já formatado
    @Override
    public String toString() {
        return String.format("%s | Área: %.2f | Perímetro: %.2f",
            getClass().getSimpleName(), calcularArea(), calcularPerimetro());
    }
}

// Circulo.java
public class Circulo extends Forma {
    private double raio;

    public Circulo(double raio) {
        this.raio = raio;
    }

    @Override
    public double calcularArea() {
        return Math.PI * raio * raio;  // π * r²
    }

    @Override
    public double calcularPerimetro() {
        return 2 * Math.PI * raio;     // 2πr
    }
}

// Retangulo.java
public class Retangulo extends Forma {
    private double largura, altura;

    public Retangulo(double largura, double altura) {
        this.largura = largura;
        this.altura = altura;
    }

    @Override
    public double calcularArea() {
        return largura * altura;
    }

    @Override
    public double calcularPerimetro() {
        return 2 * (largura + altura);
    }
}

// Uso:
// Circulo c = new Circulo(5);
// System.out.println(c);  // Circulo | Área: 78.54 | Perímetro: 31.42
```

---

## ➡️ Próximo módulo: [03 - Coleções e Streams](../03-colecoes-streams/README.md)
