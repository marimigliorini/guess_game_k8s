
---

# Jogo de Adivinhação com Flask

Este é um simples jogo de adivinhação desenvolvido utilizando o framework Flask. O jogador deve adivinhar uma senha criada aleatoriamente, e o sistema fornecerá feedback sobre o número de letras corretas e suas respectivas posições.

## Funcionalidades

- Criação de um novo jogo com uma senha fornecida pelo usuário.
- Adivinhe a senha e receba feedback se as letras estão corretas e/ou em posições corretas.
- As senhas são armazenadas  utilizando base64.
- As adivinhações incorretas retornam uma mensagem com dicas.
  
## Arquitetura e Design

A solução foi desenvolvida com foco em práticas modernas de DevOps e arquitetura de microsserviços.

#### 1. Escolha de Serviços:

- **Frontend** (NGINX + React): Atua como Proxy Reverso e servidor de ativos estáticos. Centraliza o tráfego na porta 80 e roteia requisições dinamicamente.

- **Backend** (Backend Flask): Responsável pela lógica do jogo e persistência. Desacoplado da interface, permitindo atualizações independentes.

- **DB** (Postgres): Banco de dados relacional para armazenamento seguro e estruturado dos dados do jogo.

#### 2. Redes e Comunicação:

Foi utilizada uma rede interna do Docker Compose para isolar os serviços. O backend e o banco não expõem portas externamente, aumentando a segurança. O NGINX resolve dinamicamente o endereço do serviço web através do DNS interno do Docker, o que permite escalabilidade sem interrupções.

#### 3. Persistência de Dados:

Volumes Nomeados (postgres_data) foram usados para mapear os dados do Postgres para o host. Isso garante que, mesmo se os containers forem destruídos ou recriados, os dados dos jogos permaneçam intactos.

#### 4. Balanceamento de Carga:

O design permite a Escala Horizontal. Através do bloco upstream no NGINX, podemos aumentar o número de instâncias do backend a qualquer momento, e o proxy distribuirá a carga automaticamente entre elas.


## Instalação

### 1. Pré-requisitos: 

   **Docker** e **Docker Compose** instalados.

### 2. Subir o ambiente completo:

   ```bash
   docker-compose up --build -d
   ```

### 3. Escalar o Backend: Para aumentar a capacidade de processamento:

   ```bash
   docker-compose up -d --scale backend=3
   ```

### 4. Verificação de Status:

   ```bash
   docker-compose ps
   ```

## Manutenção e Atualização

### Estratégia de Atualização por Imagem

Cada componente é definido por uma imagem Docker. Para atualizar um componente (ex: subir uma nova versão do Flask ou do Postgres): edite a versão da imagem no docker-compose.yml (ex: postgres:16-alpine) ou faça o build do novo código no Dockerfile.

Aplicar a mudança:

   ```bash
   docker-compose up --build -d
   ```

O Docker identifica qual serviço mudou e recria apenas aquele container, mantendo os volumes de dados e os outros serviços operacionais. Isso minimiza o tempo de inatividade.

### Resiliência

Todos os serviços utilizam a política ```restart: unless-stopped```. Isso garante que:

Caso um container falhe, ele será reiniciado automaticamente.

Se você parar os serviços manualmente com ```docker-compose stop```, eles permanecerão parados, respeitando o ciclo de vida do ambiente.

## Como Jogar

### 1. Criar um novo jogo

Acesse a url do frontend http://localhost:3000

Digite uma frase secreta

Envie

Salve o game-id


### 2. Adivinhar a senha

Acesse a url do frontend http://localhost:3000

Vá para o endponint breaker

entre com o game_id que foi gerado pelo Creator

Tente adivinhar

## Estrutura do Código

### Rotas:

- **`/create`**: Cria um novo jogo. Armazena a senha codificada em base64 e retorna um `game_id`.
- **`/guess/<game_id>`**: Permite ao usuário adivinhar a senha. Compara a adivinhação com a senha armazenada e retorna o resultado.

### Classes Importantes:

- **`Guess`**: Classe responsável por gerenciar a lógica de comparação entre a senha e a tentativa do jogador.
- **`WrongAttempt`**: Exceção personalizada que é levantada quando a tentativa está incorreta.



## Melhorias Futuras

- Implementar autenticação de usuário para salvar e carregar jogos.
- Adicionar limite de tentativas.
- Melhorar a interface de feedback para as tentativas de adivinhação.

## Licença

Este projeto está licenciado sob a [MIT License](LICENSE).

