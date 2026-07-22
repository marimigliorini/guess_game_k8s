
---

# Jogo de Adivinhação com Flask

Este é um simples jogo de adivinhação desenvolvido utilizando o framework Flask. O jogador deve adivinhar uma senha criada aleatoriamente, e o sistema fornecerá feedback sobre o número de letras corretas e suas respectivas posições.

## Funcionalidades

- Criação de um novo jogo com uma senha fornecida pelo usuário.
- Adivinhe a senha e receba feedback se as letras estão corretas e/ou em posições corretas.
- As senhas são armazenadas  utilizando base64.
- As adivinhações incorretas retornam uma mensagem com dicas.
  
## Componentes Instalados

Essa aplicação foi disponibilizada em um cluster Kubernetes utilizando o ambiente k3d. A solução contempla frontend, backend e banco de dados PostgreSQL, além de autoescalonamento (HPA) para o backend.

#### 1. Postgres:

- **Função:** Banco de dados relacional para persistência da aplicação.

- **Deployment:** Cria o pod com a imagem oficial ```postgres:15```.

- **Service:** Exposto internamente como ```postgres-service```, permitindo que o backend se conecte.

#### 2. Backend:

- **Função:** API Flask que implementa a lógica do jogo.

- **Deployment:** Inicialmente com 2 réplicas, usando a imagem ```marimigliorini/guess_game_backend:v3```.

- **Service:** Exposto internamente como ```backend-service```, consumido pelo frontend.

- **AutoScale (HPA):** Escala entre 2 e 10 réplicas conforme uso de CPU (limite de 70%).

#### 3. Frontend:

- **Função:** Interface web para interação com o usuário.

- **Deployment:** Usa a imagem ```marimigliorini/guess_game_frontend:v3```.

- **Service:** Exposto externamente via NodePort na porta 8080, permitindo acesso direto pelo navegador.

- **Configuração:** Consome o backend através do ```backend-service```.

#### 4. Horizontal Pod Autoscaler (HPA):

- **Função:** Monitora o uso de CPU dos pods do backend.

- **Configuração:**

   - Mínimo: 2 réplicas

   - Máximo: 10 réplicas

   - Escala quando a utilização média de CPU ultrapassa 70%.

- **Benefício:** Garante alta disponibilidade e desempenho sob carga.


## Instalação

### 1. Pré-requisitos: 

   **k3d** e **kubectl** instalados.

### 2. Criar o cluster k3d:

   ```bash
   k3d cluster create lab --port 8080:30080@loadbalancer
   ```
   Aqui será criado um cluster chamado ```lab``` com o mapeamento de portas entre o host e o cluster.

### 3. Aplicar os manifestos:

   ```bash
   kubectl apply -f ./k8s/
   ```

### 4. Verificar recursos:

   ```bash
   kubectl get pods
   kubectl get svc
   kubectl get hpa
   ```
   Aguarde até que todos os pods estejam com status **Running** (pode levar alguns minutos).

### 5. Acessar a aplicação:

   ```bash
   http://localhost:8080
   ``` 

## Imagens Docker Hub

- **Backend:** ```marimigliorini/guess_game_backend:v3```

- **Frontend:** ```marimigliorini/guess_game_frontend:v3```

- **Banco:** ```postgres:15``` (imagem oficial)

## Como Jogar

### 1. Criar um novo jogo

Acesse a url do frontend http://localhost:8080

Digite uma frase secreta

Envie

Salve o game-id


### 2. Adivinhar a senha

Acesse a url do frontend http://localhost:8080

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

