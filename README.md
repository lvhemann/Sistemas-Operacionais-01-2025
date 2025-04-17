# Sistemas-Operacionais-01-2025
## Exemplo 1
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

### Exemplo 2

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


