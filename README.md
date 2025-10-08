# tapr

Análise do Microsserviço de Autenticação
Analisando a estrutura do projeto no GitHub, podemos identificar os seguintes pontos em relação à Arquitetura Hexagonal:

Portas de Entrada (Input/Driving Ports): São as interfaces que definem como o mundo externo pode interagir com a aplicação para executar um caso de uso. No seu projeto, a porta de entrada é representada pela interface AuthService no pacote auth.service. Ela define o contrato para o caso de uso de autenticação.

Portas de Saída (Output/Driven Ports): São as interfaces que a aplicação usa para se comunicar com sistemas externos (banco de dados, outros serviços, etc.). O domínio define a porta, e a infraestrutura a implementa. No seu projeto, a interface UserRepository em user.repository atua como uma porta de saída, definindo como a aplicação busca dados de usuário, sem conhecer os detalhes do banco de dados.

Adaptadores (Adapters):


Adaptador de Entrada (Input/Driving Adapter): É a tecnologia que recebe a requisição externa e a traduz para uma chamada na porta de entrada. O AuthController (auth.controller) é o adaptador de entrada, recebendo requisições HTTP e chamando o AuthService.

Adaptador de Saída (Output/Driven Adapter): É a implementação concreta de uma porta de saída. A classe UserRepositoryImpl (que implementaria a interface UserRepository) seria o adaptador de saída, responsável por interagir com o banco de dados (neste caso, usando JPA/Hibernate) para buscar os dados do usuário.

Domínio (Domain): É o coração da aplicação, contendo a lógica de negócio pura, sem dependências de frameworks ou tecnologias de infraestrutura. As entidades como User e as regras de negócio dentro dos serviços de aplicação (como a lógica de validação de senha no AuthService) formam o domínio.

O que falta para 100% de aderência?
Seu projeto já segue muitos dos conceitos da Arquitetura Hexagonal, o que é um ótimo começo. Para uma aderência mais estrita, sugiro as seguintes melhorias:

Isolar completamente o Domínio:

A entidade User está anotada com @Entity, @Table, etc. Essas anotações são do framework de persistência (JPA) e, a rigor, "contaminam" o modelo de domínio com detalhes de infraestrutura.
Sugestão: Crie um objeto de domínio puro (POJO - Plain Old Java Object) para User no pacote de domínio. O adaptador de persistência (UserRepositoryImpl) seria então responsável por mapear entre a entidade JPA e o objeto de domínio. Isso garante que o núcleo do negócio não tenha nenhuma dependência de frameworks.

Estrutura de Pacotes mais Explícita: A estrutura atual mistura conceitos. Uma organização mais clara ajudaria a reforçar os limites da arquitetura.
Sugestão de nova estrutura de pacotes:

Plain Text
com.example.auth
├── domain
│   ├── model
│   │   └── User.java         // POJO, sem anotações de framework
│   ├── port
│   │   ├── in
│   │   │   └── AuthService.java  // Porta de entrada (interface)
│   │   └── out
│   │       └── UserRepository.java // Porta de saída (interface)
│   └── service
│       └── AuthServiceImpl.java // Implementação da porta de entrada, lógica de aplicação
└── adapter
    ├── in
    │   └── web
    │       └── AuthController.java // Adaptador de entrada (REST)
    └── out
        └── persistence
            ├── UserJpaEntity.java // Entidade JPA com anotações
            └── UserRepositoryImpl.java // Adaptador de saída (implementa UserRepository)
            
Como aplicar:
Mova a lógica de negócio que hoje está no AuthService para o AuthServiceImpl.

O AuthServiceImpl implementaria a porta de entrada AuthService.

Crie a classe UserJpaEntity com as anotações de persistência e use-a no UserRepositoryImpl para as operações de banco de dados.

O UserRepositoryImpl implementaria a porta de saída UserRepository e faria a conversão entre User (domínio) e UserJpaEntity (persistência).

Questões Extras
Aqui estão as respostas para as questões de múltipla escolha e verdadeiro/falso:

C) Manter o domínio independente de infraestrutura. O objetivo principal é isolar a lógica de negócio de detalhes externos como banco de dados, UIs ou frameworks.

B) Interfaces que expõem casos de uso do aplicativo. As portas de entrada são os contratos que o mundo exterior usa para interagir com a aplicação.

A) Interfaces usadas pelo domínio/aplicação para falar com o “mundo externo”. O domínio define uma interface (a porta de saída) e espera que um adaptador da camada de infraestrutura a implemente.

B) Domínio. O domínio deve ser puro e agnóstico a tecnologias. Anotações de frameworks pertencem aos adaptadores.

C) A UI é um adaptador de entrada plugado em uma porta de entrada. Como a lógica de negócio não conhece o protocolo (REST, gRPC), podemos trocar o adaptador sem impactar o domínio.

B) Domínio anêmico e serviços de aplicação gordos com toda a regra. Um anti-padrão comum é ter entidades que são apenas estruturas de dados (anêmicas) e colocar toda a lógica de negócio nos serviços, o que vai contra os princípios do Domain-Driven Design (DDD) que a Arquitetura Hexagonal complementa bem.


B) Substituir infraestrutura com menor impacto no domínio. A separação clara permite trocar um banco de dados, um framework de mensageria ou a tecnologia da UI com impacto mínimo ou nulo no núcleo da aplicação.
Verdadeiro ou Falso:

[ F ] Domínio pode depender de JPA desde que use apenas @Entity. Falso. @Entity é uma dependência do framework de persistência. O domínio deve ser completamente livre de detalhes de infraestrutura.
[ V ] Input Ports expõem casos de uso; Output Ports modelam dependências externas. Verdadeiro. Portas de entrada são a API da aplicação, e portas de saída são os requisitos que a aplicação tem do mundo externo.
[ F ] Adaptadores conhecem detalhes do domínio e podem validar regras complexas. Falso. Adaptadores são tradutores. Eles não devem conter lógica de negócio; essa responsabilidade é do domínio.
[ V ] Controllers devem falar com o caso de uso (input port), não diretamente com repositórios. Verdadeiro. O Controller (adaptador) deve invocar a porta de entrada (caso de uso), que orquestra a lógica e chama as portas de saída (repositórios).
[ F ] Trocar o banco (JPA → JDBC) deve exigir mudanças extensas no domínio. Falso. A troca deve impactar apenas o adaptador de persistência. O domínio, protegido pela porta de saída, não deve sofrer alterações.
Diferença entre Porta e Adaptador:
Porta (Port): É uma interface que define um contrato de comunicação. Ela pertence à camada de aplicação (o hexágono). Uma porta de entrada define o que a aplicação pode fazer (casos de uso), enquanto uma porta de saída define o que a aplicação precisa do mundo externo (ex: "salvar um usuário").
Adaptador (Adapter): É a implementação concreta de uma porta. Ele fica fora do hexágono e lida com a tecnologia específica. Um Controller REST é um adaptador que traduz requisições HTTP para chamadas em uma porta de entrada. Um RepositoryImpl com JPA é um adaptador que implementa uma porta de saída, traduzindo a necessidade de "salvar um usuário" em comandos SQL para um banco de dados específico.
