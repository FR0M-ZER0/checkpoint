# Documentação Técnica - Projeto Checkpoint 🕒

## 1. Introdução

Este documento fornece uma visão geral técnica do projeto Checkpoint, uma aplicação web para gerenciamento de jornada de trabalho.

## 2. Arquitetura do Sistema

### 2.1. Visão Geral
O Checkpoint é composto por duas aplicações principais:
* **Frontend:** Desenvolvida em React com TypeScript, responsável pela interface do usuário e interação.
* **Backend:** Construída com Spring Boot (Java), responsável pela lógica de negócios, processamento de dados e persistência.
* **Bancos de Dados:**
    * **MySQL (via AWS RDS):** Utilizado para persistência de dados relacionais principais (colaboradores, jornadas, solicitações, etc.).
    * **MongoDB:** Utilizado para armazenar dados de marcações de ponto, logs de marcação e solicitações de ajuste de ponto, aproveitando sua flexibilidade para esses tipos de dados.
* **Comunicação em Tempo Real:** WebSockets (STOMP sobre SockJS) são utilizados para notificações em tempo real aos usuários.
* **Serviços Externos:**
    * **AWS:** Para hospedagem do banco de dados MySQL (RDS)
    * **Serviço de Email (Gmail SMTP):** Para envio de notificações por email


### 2.2. Fluxo de Dados Simplificado

1.  Colaborador acessa o frontend.
2.  Backend valida credenciais no MySQL.
3.  Backend retorna dados do usuário/token.
4.  Frontend armazena token/ID e permite acesso.
5.  Colaborador registra ponto.
6.  Frontend envia dados da marcação para o backend.
7.  Backend salva marcação no MongoDB e atualiza dados relevantes no MySQL(Ex.: banco de horas)
8.  Backend envia notificação via WebSocket se necessário.

## 3. Backend (`checkpoint_back`)

### 3.1. Estrutura do Projeto
O backend segue uma estrutura de projeto Maven padrão para aplicações Spring Boot. Os principais pacotes incluem:
* `com.fromzero.checkpoint.entities`: Contém as classes de entidade para MySQL
* `com.fromzero.checkpoint.repositories`: Interfaces Spring Data JPA e Spring Data MongoDB para acesso aos dados.
* `com.fromzero.checkpoint.services`: Classes de serviço contendo a lógica de negócios.
* `com.fromzero.checkpoint.controllers`: Controladores REST que expõem os endpoints da API.
* `com.fromzero.checkpoint.dtos`: Data Transfer Objects usados para comunicação entre camadas ou com o frontend.
* `com.fromzero.checkpoint.config`: Classes de configuração (ex: segurança, WebSockets).
* `com.fromzero.checkpoint.utils`: Classes utilitárias.

### 3.2. Tecnologias Utilizadas
* **Linguagem:** Java ( projeto usa JDK 23)
* **Framework Principal:** Spring Boot
    * Spring Web (para APIs REST)
    * Spring Data JPA (para MySQL)
    * Spring Data MongoDB (para MongoDB)
    * Spring WebSocket (com STOMP e SockJS)
* **Build e Gerenciamento de Dependências:** Apache Maven
* **Banco de Dados Relacional:** MySQL (acessado via AWS RDS)
* **Banco de Dados NoSQL:** MongoDB
* **Utilitários:** Lombok (para reduzir boilerplate)
* **Email:** Spring Mail com Gmail SMTP

### 3.3. Configuração do Ambiente de Desenvolvimento
1.  **Pré-requisitos:**
    * JDK 17 ou superior (projeto usa JDK 23).
    * Apache Maven 3.6+.
    * Instância do MySQL acessível.
    * Instância do MongoDB  acessível.
2.  **Clonar o Repositório:** `git clone https://github.com/FR0M-ZER0/checkpoint_back.git`
3.  **Configuração do Banco de Dados MySQL:**
    * Crie o schema/database `CheckPointDBV2` (se ainda não existir).
    * Execute os scripts SQL de criação de tabelas. [Link para o arquivo .sql com os CREATE TABLES]
    * Garanta que as colunas de ID e FK correspondentes a campos `Long` nas entidades Java sejam do tipo `BIGINT`.
4.  **Configuração do MongoDB:**
    * Certifique-se de que o MongoDB está rodando e acessível. O nome do database (`CheckPointDB`) é definido no `application.properties`.
5.  **Arquivo `application.properties` (`src/main/resources/application.properties`):**
    * Configure as propriedades do datasource MySQL:
        ```properties
        spring.datasource.url=jdbc:mysql://[HOST_MYSQL]:3306/CheckPointDBV2?serverTimezone=America/Sao_Paulo&zeroDateTimeBehavior=CONVERT_TO_NULL
        spring.datasource.username=[USUARIO_MYSQL]
        spring.datasource.password=[SENHA_MYSQL]
        spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQLDialect
        spring.jpa.hibernate.ddl-auto=validate # ou none para produção
        ```
    * Configure a URI do MongoDB:
        ```properties
        spring.data.mongodb.uri=mongodb://[HOST_MONGO]:27017/CheckPointDB
        # spring.data.mongodb.username=[USUARIO_MONGO]
        # spring.data.mongodb.password=[SENHA_MONGO]
        ```
    * Configure as credenciais de email:
        ```properties
        spring.mail.host=smtp.gmail.com
        spring.mail.port=587
        spring.mail.username=[EMAIL_GMAIL]
        spring.mail.password=[SENHA_DE_APLICATIVO_GMAIL]
        # ... outras propriedades de email ...
        ```
    * Outras configurações (upload de diretório, etc.).
6.  **Build do Projeto:**
    No terminal, na raiz do projeto backend:
    ```bash
    mvn clean install
    ```


### 3.4. Módulos e Componentes Principais
* **Entidades JPA (MySQL):**
    * `Colaborador`: Informações do funcionário, incluindo saldo de férias.
    * `Gestor`: Informações do gestor.
    * `Jornada`: Define os modelos de jornada (escala, carga horária, tipo).
    * `ColaboradorJornada`: Associa um colaborador a uma `Jornada` por um período.
    * `SolicitacaoFerias`: Registra pedidos de férias.
    * `SolicitacaoFolga`: Registra pedidos de folga (usando banco de horas).
    * `SolicitacaoAbonoFerias`: Pedidos de venda de dias de férias.
    * `Falta`: Registros de faltas/atrasos.
    * `SolicitacaoAbonoFalta`: Justificativas para faltas.
    * `HorasExtras`: Saldo de banco de horas do colaborador.
    * `HorasExtrasManual`: Registro de alterações manuais de horas extras pelo gestor.
    * `Notificacao`: Notificações para os usuários (JPA).
    * `Resposta`: Respostas do gestor a solicitações (JPA).
* **Documentos MongoDB:**
    * `Marcacao`: Registros de ponto (entrada, saída, pausa, retomada).
    * `MarcacaoLog`: Logs de criação/deleção de marcações.
    * `SolicitacaoAjustePonto`: Pedidos de ajuste de marcações de ponto.
* **Serviços:**
    * `JornadaService`: Lógica relacionada à jornada do colaborador, cálculo de dias de descanso.
    * `FeriasService`: Lógica para solicitar, aprovar/rejeitar férias, calcular saldo.
    * `FolgaService`: Lógica para solicitar folgas (usando banco de horas), obter saldo de horas.
    * `MarcacaoService`: Lógica para registrar ponto, calcular horas trabalhadas.
    * `NotificacaoService`: Criação e gerenciamento de notificações.
    * `ColaboradorService` (se existir): Lógica de login, cadastro de colaborador.

* **Controladores Principais:**
    * `ColaboradorController`: Endpoints para login, dados do colaborador, notificações, respostas.
    * `DiasTrabalhoController`: Endpoint para o espelho de ponto (`/dias-trabalho/{id}?ano={ano}`), resumo do dia.
    * `FeriasController`: Endpoints para saldo, solicitação, aprovação/rejeição de férias e venda de dias.
    * `FolgaController`: Endpoints para saldo de horas para folga, solicitação de folgas.
    * `MarcacaoController`: Endpoints para registrar ponto, visualizar marcações.
    * `SolicitacaoAjustePontoController` (se existir).
    * `HorasExtrasController` (se existir para gerenciar saldo ou aprovações).


### 3.5. API Endpoints
A API do Checkpoint é construída seguindo os princípios RESTful. Recomenda-se a utilização de uma ferramenta como Swagger/OpenAPI para uma documentação interativa e detalhada dos endpoints.
* **Autenticação:**
    * `POST /login`: Autentica o colaborador.
* **Colaborador:**
    * `GET /colaborador/{id}`: Busca dados de um colaborador.
    * `GET /colaborador/notificacoes/{id}`: Busca notificações não lidas.
    * `GET /colaborador/resposta/{id}`: Busca respostas não lidas.
    * [Outros endpoints de colaborador]
* **Jornada & Espelho de Ponto:**
    * `GET /dias-trabalho/{colaboradorId}?ano={ano}`: Retorna o calendário de status do colaborador para um ano.
    * `GET /dias-trabalho/{colaboradorId}/{data}`: Detalhes de um dia específico.
    * `GET /dias-trabalho/{colaboradorId}/resumo?ano={ano}`: Resumo anual do colaborador.
    * `POST /api/jornada` (ou similar): Para gestor criar/editar modelos de jornada.
    * `POST /api/colaborador-jornada`: Para gestor associar colaborador a uma jornada.
* **Marcações de Ponto (MongoDB):**
    * `POST /marcacoes`: Registra uma nova marcação.
    * `GET /marcacoes/colaborador/{colaboradorId}/hoje`: Marcações do dia atual.
    * [Outros endpoints de marcação]
* **Solicitações:**
    * `POST /api/ferias/agendar`: Colaborador solicita férias.
    * `GET /api/ferias/solicitacoes/colaborador/{id}`: Lista solicitações de férias do colaborador.
    * `PUT /api/ferias/solicitacoes/{id}/aprovar` ou `/rejeitar`: Gestor aprova/rejeita.
    * `POST /api/folga`: Colaborador solicita folga (usando saldo de horas).
    * `GET /api/folga?colaboradorId={id}`: Lista solicitações de folga.
    * `POST /solicitacao-abono-falta`: Colaborador envia abono de falta.
    * `POST /ajuste-ponto/solicitacao`: Colaborador solicita ajuste de ponto.
* **Saldos:**
    * `GET /api/ferias/saldo?colaboradorId={id}`: Saldo de dias de férias do colaborador.
    * `GET /api/folga/saldo?colaboradorId={id}`: Saldo de horas (banco de horas) para folga.



### 3.6. Esquema do Banco de Dados
* **MySQL:**
    * As tabelas principais incluem `colaborador`, `gestor`, `jornada`, `colaborador_jornada`, `solicitacao_ferias`, `solicitacao_folga`, `falta`, `solicitacao_abono_falta`, `horas_extras`, `notificacao`, `resposta`, `folga` (para folgas efetivadas), `ferias` (para férias efetivadas).
    * [Link para o script SQL de criação das tabelas (`schema.sql` ou similar) ou para um Diagrama Entidade-Relacionamento].
* **MongoDB:**
    * Coleção `marcacoes`: Armazena cada batida de ponto individual.
        * Campos: `_id`, `colaboradorId`, `dataHora`, `tipo` (ENTRADA, SAIDA, PAUSA, RETOMADA), `processada`.
    * Coleção `marcacao_log`: Logs de operações nas marcações.
    * Coleção `solicitacao_ajuste_ponto`: Solicitações de ajuste de ponto.


### 3.7. Notificações em Tempo Real (WebSockets)
* O sistema utiliza STOMP sobre SockJS para notificações em tempo real.
* Endpoints do Broker: Configurados em `WebSocketConfig.java`.
* Tópicos principais:
    * `/topic/solicitacoes`: Notificações gerais de solicitações para gestores.
    * `/topic/notificacoes/{userId}`: Notificações específicas para um colaborador.
    * `/topic/respostas/{userId}`: Respostas específicas para um colaborador.


---
## 4. Frontend (`checkpoint_front`)

### 4.1. Estrutura do Projeto
O frontend é uma aplicação Vite + React + TypeScript. A estrutura de pastas principal é:
* `src/`: Contém todo o código fonte.
    * `components/`: Componentes reutilizáveis (ex: `Modal.tsx`, `Button.tsx`, `Calendar.tsx`).
    * `pages/`: Componentes de página (ex: `LoginPage.tsx`, `EspelhoPontoPage.tsx`, `Ferias.tsx`).
    * `services/`: Configuração do Axios (`api.ts`) para chamadas à API backend.
    * `store/`: Configuração do Redux Toolkit.
        * `slices/`: Slices Redux para gerenciamento de estado (ex: `authSlice.ts`, `notificationSlice.ts`).
    * `utils/`: Funções utilitárias (ex: `formatter.ts`, `hooks.ts`).
    * `assets/`: Imagens e outros arquivos estáticos.
    * `App.tsx`: Componente principal com as rotas.
    * `main.tsx`: Ponto de entrada da aplicação React.

### 4.2. Tecnologias Utilizadas
* **Framework/Biblioteca Principal:** React 18+ com TypeScript.
* **Build Tool:** Vite.
* **Roteamento:** React Router DOM v6+.
* **Gerenciamento de Estado Global:** Redux Toolkit.
* **Chamadas API:** Axios.
* **Estilização:** Tailwind CSS.
* **Componentes UI (exemplos):**
    * `react-calendar` (para seleção de datas).
    * `react-datepicker` (se usado).
    * `react-toastify` (para notificações toast).
    * `react-loading-skeleton` (para feedback de carregamento).
    * Componentes Radix UI (ex: `@radix-ui/react-dialog`).
* **Manipulação de Datas:** `date-fns`.
* **WebSockets:** `@stomp/stompjs` e `sockjs-client`.

### 4.3. Configuração do Ambiente de Desenvolvimento
1.  **Pré-requisitos:**
    * Node.js (versão LTS recomendada, ex: 18.x ou 20.x).
    * NPM (geralmente vem com o Node.js) ou Yarn.
2.  **Clonar o Repositório:** `git clone [URL_DO_REPOSITORIO_FRONTEND]`
3.  **Instalar Dependências:**
    Na raiz do projeto frontend, execute:
    ```bash
    npm install
    ```
    ou se usar Yarn:
    ```bash
    yarn install
    ```
4.  **Variáveis de Ambiente (se houver):**
    * Verifique se existe um arquivo `.env` ou `.env.local` para configurar variáveis como a URL base da API (ex: `VITE_API_BASE_URL=http://localhost:8080`). Se não, a URL da API pode estar hardcoded em `src/services/api.ts`.
5.  **Executar a Aplicação em Modo de Desenvolvimento:**
    ```bash
    npm run dev
    ```
    ou
    ```bash
    yarn dev
    ```
    A aplicação geralmente estará disponível em `http://localhost:5173`.

### 4.4. Módulos e Componentes Chave
* **Gerenciamento de Estado (Redux Slices):**
    * `authSlice`: Gerencia o estado de autenticação do usuário (ID, nome, token).
    * `notificationSlice`: Gerencia notificações em tempo real.
    * `responseSlice`: Gerencia respostas em tempo real.
    * [Adicionar outros slices relevantes]
* **Serviço API (`src/services/api.ts`):**
    * Instância configurada do Axios para fazer requisições ao backend.
    * Pode incluir interceptors para adicionar tokens de autenticação ou tratar erros globalmente.
* **Roteamento (`src/App.tsx` ou arquivo de rotas dedicado):**
    * Define as rotas da aplicação usando `react-router-dom` e quais componentes de página são renderizados para cada rota.
    * Rotas protegidas que exigem login.
* **Páginas Principais:**
    * `LoginPage.tsx`: Formulário de login.
    * `EspelhoPontoPage.tsx`: Exibe o calendário de ponto do colaborador.
    * `DayPage.tsx`: Exibe detalhes de um dia específico do espelho de ponto.
    * `MarkingPage.tsx`: Interface para o colaborador registrar o ponto.
    * `FeriasPage.tsx`: Gerenciamento de solicitações de férias.
    * `FolgaPage.tsx`: Gerenciamento de solicitações de folga.
    * [Adicionar outras páginas importantes para Colaborador e Gestor]
* **Componentes Reutilizáveis:**
    * `Modal.tsx`: Componente genérico para modais.
    * `DateFilter.tsx`: Filtro de data.
    * `PointButton.tsx`, `SquareButton.tsx`: Botões customizados.
    * [Adicionar outros componentes importantes]

### 4.5. Estilização
* O projeto utiliza **Tailwind CSS** para estilização utilitária.
* Configurações do Tailwind estão em `tailwind.config.js`.
* Cores customizadas (ex: `light-blue-color`, `main-orange-color`) são definidas no `tailwind.config.js` .

---
## 5. Deploy (Visão Geral)
* **Backend:**
    * O build do Maven (`mvn clean package`) gera um arquivo JAR executável.
    * Este JAR pode ser implantado em um servidor com Java (ex: AWS EC2, Elastic Beanstalk).
    * Configuração do banco de dados (RDS) e outras variáveis de ambiente precisam ser fornecidas ao ambiente de produção.
* **Frontend:**
    * O build do Vite (`npm run build` ou `yarn build`) gera arquivos estáticos (HTML, CSS, JS) na pasta `dist/`.
    * Esses arquivos podem ser hospedados em um serviço de hospedagem de sites estáticos (ex: AWS S3 + CloudFront, Netlify, Vercel).
    * Configuração para apontar para a URL da API de produção é necessária.

