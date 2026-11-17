# ☕ Guia Java — Do Iniciante ao Intermediário

> Guia prático e progressivo para quem quer aprender Java do zero ou migrar de outra linguagem (PHP, JavaScript, Python).

---

## 🎯 Para quem é este guia?

- Estudantes de Computação/Sistemas que nunca viram Java
- Desenvolvedores PHP/JavaScript que querem aprender Java
- Programadores de qualquer linguagem que querem entender o ecossistema Java empresarial
- Quem quer se preparar para o mercado de trabalho com Java + Spring Boot

---

## 🗺️ Mapa do Conteúdo

| Módulo | Tema | Nível |
|--------|------|-------|
| [01 - Fundamentos](./01-fundamentos/README.md) | Tipos, variáveis, loops, métodos, arrays | 🟢 Iniciante |
| [02 - POO](./02-poo/README.md) | Classes, herança, polimorfismo, interfaces | 🟢 Iniciante |
| [03 - Coleções e Streams](./03-colecoes-streams/README.md) | List, Map, Set, Lambda, Streams | 🟡 Básico-Intermediário |
| [04 - Exceções e I/O](./04-excecoes-io/README.md) | Try/catch, arquivos, leitura de dados | 🟡 Básico-Intermediário |
| [05 - Concorrência](./05-concorrencia/README.md) | Threads, ExecutorService, CompletableFuture | 🟠 Intermediário |
| [06 - Arquiteturas](./06-arquiteturas/README.md) | Layered, Hexagonal, MVC, boas práticas | 🟠 Intermediário |
| [07 - Spring Boot](./07-spring-boot/README.md) | REST API completa com Spring Boot | 🟠 Intermediário |
| [08 - Banco de Dados](./08-banco-dados/README.md) | JPA, Hibernate, Spring Data | 🟠 Intermediário |
| [09 - Testes](./09-testes/README.md) | JUnit 5, Mockito, TDD | 🟠 Intermediário |
| [10 - Migração PHP → Java](./10-migracao-php/README.md) | Guia prático de migração para devs PHP | 🟠 Intermediário |

---

## 🚀 Por onde começar?

### Sou totalmente iniciante em programação
Leia os módulos **01 → 02 → 03 → 04** em ordem. Faça todos os exercícios antes de avançar.

### Já sei PHP (ou outra linguagem)
Leia o **Módulo 10** primeiro para entender as diferenças principais. Depois siga a ordem normal.

### Já sei Java básico e quero aprender Spring Boot
Pule para o **Módulo 06 → 07 → 08 → 09**.

---

## 🛠️ Pré-requisitos

### Instalar o JDK
```bash
# Verificar se já tem instalado
java -version
javac -version

# Se não tiver, instale o JDK 21 (versão LTS recomendada)
# Windows: https://adoptium.net/
# Linux (Ubuntu/Debian):
sudo apt install openjdk-21-jdk

# Mac:
brew install openjdk@21
```

### IDE Recomendada
- **IntelliJ IDEA Community** (gratuita) — recomendada para Java
- **VS Code** com extensão "Extension Pack for Java"
- **Eclipse** (alternativa gratuita)

### Compilar e rodar sem IDE
```bash
# Compilar
javac HelloWorld.java

# Rodar
java HelloWorld

# Compilar e rodar com Maven
mvn compile
mvn exec:java -Dexec.mainClass="com.exemplo.Main"
```

---

## 📚 Conceitos-chave que você vai dominar

```
Fundamentos
├── Tipagem estática (diferente de PHP/Python/JS)
├── JVM — Write Once, Run Anywhere
├── Compilação → Bytecode → Execução
└── Garbage Collector automático

Orientação a Objetos
├── Classes e Objetos
├── Encapsulamento (private/public/protected)
├── Herança (extends)
├── Polimorfismo (mesmo método, comportamentos diferentes)
└── Interfaces (contratos)

Ecossistema
├── Maven / Gradle — gerenciamento de dependências
├── Spring Boot — framework web mais usado no mercado
├── JPA / Hibernate — ORM para banco de dados
└── JUnit / Mockito — testes automatizados
```

---

## 💡 Dicas para aprender bem

1. **Digite o código** — não copie e cole. Digitar força seu cérebro a processar.
2. **Quebre o código** — mude valores, remova linhas, veja o que acontece.
3. **Faça os exercícios** — são a parte mais importante de cada módulo.
4. **Use o debugger** — coloque breakpoints e observe os valores das variáveis.
5. **Leia os erros** — a mensagem de erro do Java é sua maior aliada.

---

## ➡️ Começar: [Módulo 01 — Fundamentos](./01-fundamentos/README.md)
