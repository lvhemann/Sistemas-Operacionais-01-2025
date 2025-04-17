# Sistemas-Operacionais-01-2025
## Exemplo 1 - Produtor vs Consumidor
```
#include <stdio.h>
#include <stdlib.h>
#include <pthread.h>
#include <semaphore.h>
#include <unistd.h>

#define N 5
#define NUM_ITENS 10

int fila[N];
int in = 0, out = 0;

sem_t empty;
sem_t full;
pthread_mutex_t mutex;

void* produtor(void* arg) {
    for (int i = 0; i < NUM_ITENS; i++) {
        int item = rand() % 100;

        sem_wait(&empty);
        pthread_mutex_lock(&mutex);

        fila[in] = item;
        printf("Produtor: inseriu %d na posição %d\n", item, in);
        in = (in + 1) % N;

        // ERRO: esquecido de liberar o mutex
        pthread_mutex_unlock(&mutex);  // INTENCIONALMENTE OMITIDO
        sem_post(&full);

        sleep(1);
    }
    pthread_exit(NULL);
}

void* consumidor(void* arg) {
    for (int i = 0; i < NUM_ITENS; i++) {
        sem_wait(&full);
        pthread_mutex_lock(&mutex);  // Vai travar aqui para sempre após o primeiro loop

        int item = fila[out];
        printf("Consumidor: removeu %d da posição %d\n", item, out);
        out = (out + 1) % N;

        pthread_mutex_unlock(&mutex);
        sem_post(&empty);

        sleep(1);
    }
    pthread_exit(NULL);
}

int main() {
    pthread_t t_produtor, t_consumidor;

    sem_init(&empty, 0, N);
    sem_init(&full, 0, 0);
    pthread_mutex_init(&mutex, NULL);

    pthread_create(&t_produtor, NULL, produtor, NULL);
    pthread_create(&t_consumidor, NULL, consumidor, NULL);

    pthread_join(t_produtor, NULL);
    pthread_join(t_consumidor, NULL);

    sem_destroy(&empty);
    sem_destroy(&full);
    pthread_mutex_destroy(&mutex);

    return 0;
}

```

### Exemplo 2 - Consumidor

```
#include <stdio.h>
#include <stdlib.h>
#include <pthread.h>
#include <semaphore.h>
#include <unistd.h>

#define N 3  // Tamanho da fila (buffer)
#define NUM_PRODUTOS 5
#define NUM_CONSUMOS 10

int fila[N];
int in = 0, out = 0;

sem_t empty;   // Quantos espaços vazios
sem_t full;    // Quantos itens disponíveis
pthread_mutex_t mutex;

void* produtor(void* arg) {
    for (int i = 0; i < NUM_PRODUTOS; i++) {
        int item = rand() % 100;

        sem_wait(&empty);               // Espera espaço livre
        pthread_mutex_lock(&mutex);     // Região crítica

        fila[in] = item;
        printf("Produtor: inseriu %d na posição %d\n", item, in);
        in = (in + 1) % N;

        pthread_mutex_unlock(&mutex);
        sem_post(&full);                // Sinaliza item disponível

        sleep(1);
    }
    printf("Produtor finalizado.\n");
    pthread_exit(NULL);
}

void* consumidor(void* arg) {
    for (int i = 0; i < NUM_CONSUMOS; i++) {
        printf("Consumidor esperando item...\n");
        sem_wait(&full);                // Espera item disponível
        pthread_mutex_lock(&mutex);     // Região crítica

        int item = fila[out];
        printf("Consumidor: removeu %d da posição %d\n", item, out);
        out = (out + 1) % N;

        pthread_mutex_unlock(&mutex);
        sem_post(&empty);               // Sinaliza espaço livre

        sleep(1);
    }
    printf("Consumidor finalizado.\n");
    pthread_exit(NULL);
}

int main() {
    pthread_t t_produtor, t_consumidor;

    sem_init(&empty, 0, N);
    sem_init(&full, 0, 0);
    pthread_mutex_init(&mutex, NULL);

    pthread_create(&t_produtor, NULL, produtor, NULL);
    pthread_create(&t_consumidor, NULL, consumidor, NULL);

    pthread_join(t_produtor, NULL);
    pthread_join(t_consumidor, NULL);  // Vai travar após o 5º item

    sem_destroy(&empty);
    sem_destroy(&full);
    pthread_mutex_destroy(&mutex);

    return 0;
}


```
### Exemplo 3 - Produtor 
```
#include <stdio.h>
#include <stdlib.h>
#include <pthread.h>
#include <semaphore.h>
#include <unistd.h>

#define TAM_FILA 5
#define NUM_PRODUTOS 10
#define NUM_CONSUMOS 5

int fila[TAM_FILA];
int in = 0, out = 0;

sem_t empty;  // indica espaços disponíveis
sem_t full;   // indica itens disponíveis
pthread_mutex_t mutex;

void* produtor(void* arg) {
    for (int i = 0; i < NUM_PRODUTOS; i++) {
        int item = rand() % 100;

        printf("Produtor: tentando inserir item %d\n", i + 1);
        sem_wait(&empty);  // bloqueia se a fila estiver cheia

        pthread_mutex_lock(&mutex);
        fila[in] = item;
        printf("Produtor: inseriu %d na posição %d\n", item, in);
        in = (in + 1) % TAM_FILA;
        pthread_mutex_unlock(&mutex);

        sem_post(&full);   // sinaliza que um item está disponível
        sleep(1);
    }

    printf("Produtor finalizou.\n");
    return NULL;
}

void* consumidor(void* arg) {
    for (int i = 0; i < NUM_CONSUMOS; i++) {
        sem_wait(&full);  // bloqueia se a fila estiver vazia

        pthread_mutex_lock(&mutex);
        int item = fila[out];
        printf("Consumidor: removeu %d da posição %d\n", item, out);
        out = (out + 1) % TAM_FILA;
        pthread_mutex_unlock(&mutex);

        sem_post(&empty);  // sinaliza que um espaço ficou livre
        sleep(2);
    }

    printf("Consumidor finalizou.\n");
    return NULL;
}

int main() {
    pthread_t t_produtor, t_consumidor;

    sem_init(&empty, 0, TAM_FILA);  // fila começa totalmente vazia
    sem_init(&full, 0, 0);
    pthread_mutex_init(&mutex, NULL);

    pthread_create(&t_produtor, NULL, produtor, NULL);
    pthread_create(&t_consumidor, NULL, consumidor, NULL);

    pthread_join(t_produtor, NULL);
    pthread_join(t_consumidor, NULL);

    sem_destroy(&empty);
    sem_destroy(&full);
    pthread_mutex_destroy(&mutex);

    return 0;
}

```


### Exemplo 4 - Deadlocks
```
#include <stdio.h>
#include <stdlib.h>
#include <pthread.h>
#include <unistd.h>

// Inicializa os recursos (mutexes) que simulam R1 e R2
pthread_mutex_t R1 = PTHREAD_MUTEX_INITIALIZER;
pthread_mutex_t R2 = PTHREAD_MUTEX_INITIALIZER;

// Processo 1: sempre adquire R1 primeiro, depois R2
void* processo1(void* arg) {
    pthread_mutex_lock(&R1);
    printf("P1: R1 adquirido\n");
    sleep(1);  // Simula tempo de processamento

    printf("P1: tentando adquirir R2...\n");
    pthread_mutex_lock(&R2);
    printf("P1: R2 adquirido\n");

    // Região crítica (simulada)
    printf("P1: executando com R1 e R2\n");

    pthread_mutex_unlock(&R2);
    pthread_mutex_unlock(&R1);
    printf("P1: finalizou\n");

    return NULL;
}

// Processo 2:
// Para causar deadlock: adquire R2 e tenta pegar R1
// Para evitar deadlock: adquire R1 e depois R2 (mesma ordem do processo1)
void* processo2(void* arg) {

    // COMENTE ESTA LINHA para evitar deadlock:
    pthread_mutex_lock(&R2);  // Deadlock se R1 já estiver com P1

    // DESCOMENTE ESTA LINHA para evitar deadlock:
    // pthread_mutex_lock(&R1);  // Segue mesma ordem de aquisição

    printf("P2: recurso 1 adquirido\n");
    sleep(1);

    printf("P2: tentando adquirir outro recurso...\n");

    // COMENTE ESTA LINHA para evitar deadlock:
    pthread_mutex_lock(&R1);

    // DESCOMENTE ESTA LINHA para evitar deadlock:
    // pthread_mutex_lock(&R2);

    printf("P2: recurso 2 adquirido\n");

    // Região crítica (simulada)
    printf("P2: executando com os dois recursos\n");

    // Libera recursos
    pthread_mutex_unlock(&R1);
    pthread_mutex_unlock(&R2);
    printf("P2: finalizou\n");

    return NULL;
}

int main() {
    pthread_t t1, t2;

    pthread_create(&t1, NULL, processo1, NULL);
    pthread_create(&t2, NULL, processo2, NULL);

    pthread_join(t1, NULL);
    pthread_join(t2, NULL);

    return 0;
}

```

### Exemplo 5 
```
#include <stdio.h>
#include <stdlib.h>
#include <pthread.h>
#include <semaphore.h>
#include <unistd.h>

sem_t R1;
sem_t R2;

void* processo1(void* arg) {
    sem_wait(&R1);
    printf("P1: R1 adquirido\n");
    sleep(1);  // Simula tempo de processamento

    printf("P1: tentando adquirir R2...\n");
    sem_wait(&R2);
    printf("P1: R2 adquirido\n");

    printf("P1: executando com R1 e R2\n");

    sem_post(&R2);
    sem_post(&R1);
    printf("P1: finalizou\n");

    return NULL;
}

void* processo2(void* arg) {
    // COMENTE esta linha e DESCOMENTE a linha abaixo para evitar deadlock:
    sem_wait(&R2);  // Adquire R2 primeiro — mesma ordem invertida = risco de deadlock

    // sem_wait(&R1);  // Ordem segura: mesma de P1

    printf("P2: R2 adquirido\n");
    sleep(1);

    printf("P2: tentando adquirir R1...\n");
    sem_wait(&R1);
    printf("P2: R1 adquirido\n");

    printf("P2: executando com R2 e R1\n");

    sem_post(&R1);
    sem_post(&R2);
    printf("P2: finalizou\n");

    return NULL;
}

int main() {
    pthread_t t1, t2;

    // Inicializa os semáforos com valor 1 (semáforo binário)
    sem_init(&R1, 0, 1);  // Recurso R1 está livre
    sem_init(&R2, 0, 1);  // Recurso R2 está livre

    pthread_create(&t1, NULL, processo1, NULL);
    pthread_create(&t2, NULL, processo2, NULL);

    pthread_join(t1, NULL);
    pthread_join(t2, NULL);

    sem_destroy(&R1);
    sem_destroy(&R2);

    return 0;
}

```

### Exemplo 6
```
#include <stdio.h>
#include <stdlib.h>
#include <pthread.h>
#include <semaphore.h>
#include <unistd.h>

// -------- CONFIGURAÇÃO DE QUANTIDADE DE INSTÂNCIAS --------
#define INSTANCIAS_R1 1
#define INSTANCIAS_R2 1  // <- Pode mudar durante a aula
#define INSTANCIAS_R3 2
#define INSTANCIAS_R4 3
// -----------------------------------------------------------

// Semáforos para os recursos
sem_t R1;
sem_t R2;
sem_t R3;
sem_t R4;

void* P1(void* arg) {
    sleep(1);  // Espera um pouco para mostrar ordem

    printf("P1: tentando adquirir R1...\n");
    sem_wait(&R1);
    printf("P1: R1 adquirido\n");
    sleep(2);

    printf("P1: tentando adquirir R3...\n");
    sem_wait(&R3);
    printf("P1: R3 adquirido\n");

    printf("P1: executando com R1 e R3\n");
    sleep(2);

    sem_post(&R3);
    sem_post(&R1);
    printf("P1: liberou R3 e R1\n");
    return NULL;
}

void* P2(void* arg) {
    printf("P2: tentando adquirir R3...\n");
    sem_wait(&R3);
    printf("P2: R3 adquirido\n");
    sleep(2);

    printf("P2: tentando adquirir R1...\n");
    sem_wait(&R1);
    printf("P2: R1 adquirido\n");

    printf("P2: executando com R3 e R1\n");
    sleep(2);

    sem_post(&R1);
    sem_post(&R3);
    printf("P2: liberou R1 e R3\n");
    return NULL;
}

void* P3(void* arg) {
    sleep(3);  // Começa por último

    printf("P3: tentando adquirir R2...\n");
    sem_wait(&R2);
    printf("P3: R2 adquirido\n");
    sleep(1);

    printf("P3: tentando adquirir R4...\n");
    sem_wait(&R4);
    printf("P3: R4 adquirido\n");

    printf("P3: executando com R2 e R4\n");
    sleep(2);

    sem_post(&R4);
    sem_post(&R2);
    printf("P3: liberou R4 e R2\n");
    return NULL;
}

int main() {
    pthread_t t1, t2, t3;

    // Inicializa semáforos com base nas constantes definidas
    sem_init(&R1, 0, INSTANCIAS_R1);
    sem_init(&R2, 0, INSTANCIAS_R2);
    sem_init(&R3, 0, INSTANCIAS_R3);
    sem_init(&R4, 0, INSTANCIAS_R4);

    // Cria as threads (processos)
    pthread_create(&t1, NULL, P1, NULL);
    pthread_create(&t2, NULL, P2, NULL);
    pthread_create(&t3, NULL, P3, NULL);

    // Aguarda todas terminarem
    pthread_join(t1, NULL);
    pthread_join(t2, NULL);
    pthread_join(t3, NULL);

    // Libera recursos
    sem_destroy(&R1);
    sem_destroy(&R2);
    sem_destroy(&R3);
    sem_destroy(&R4);

    return 0;
}

```
