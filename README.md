# Sistemas-Operacionais-01-2025

## Modos de Usuário 

```bash  
  #include <stdio.h>
#include <stdbool.h>

#define N 3 // número de processos

int lock = 0;             // variável de bloqueio (0 = livre, 1 = ocupado)
bool waiting[N] = {0};    // vetor de controle de intenção

// função atômica simulada para test_and_set
bool test_and_set(int *lock) {
    bool old = *lock;
    *lock = 1;
    return old;
}

// simulação do comportamento de um processo
void processo(int i) {
    int j;

    // processo i quer entrar
    waiting[i] = true;

    // espera ocupada até conseguir o lock
    while (waiting[i] && test_and_set(&lock));

    // região crítica
    printf("🟢 Processo P%d entrou na REGIÃO CRÍTICA\n", i);

    // aqui você pode simular um pequeno delay, se quiser
    // ou pedir para o aluno indicar quando o processo sai

    // processo i sai
    printf("🔴 Processo P%d saiu da REGIÃO CRÍTICA\n", i);
    waiting[i] = false;

    // procura o próximo processo que está esperando
    j = (i + 1) % N;
    while (j != i && !waiting[j]) {
        j = (j + 1) % N;
    }

    if (j == i) {
        // ninguém está esperando, libera o lock
        lock = 0;
    } else {
        // libera o próximo processo
        waiting[j] = false;
    }

    // seção restante
    printf("⚪ Processo P%d está na SEÇÃO RESTANTE\n", i);
}

```

```
// Exemplo de Peterson usando pthreads (duas threads)
#include <stdio.h>
#include <pthread.h>
#include <stdbool.h>
#include <unistd.h>

#define NUM_THREADS 2

volatile bool flag[NUM_THREADS] = {false, false};
volatile int turn;

void enter_region(int i) {
    int j = 1 - i;
    flag[i] = true;
    turn = j;
    while (flag[j] && turn == j); // espera ocupada
}

void leave_region(int i) {
    flag[i] = false;
}

void* thread_function(void* arg) {
    int id = *(int*)arg;
    for (int k = 0; k < 5; ++k) {
        enter_region(id);
        printf("🔐 Thread %d entrou na região crítica (iter %d)\n", id, k);
        sleep(1); // simula trabalho na região crítica
        printf("🔓 Thread %d saindo da região crítica (iter %d)\n", id, k);
        leave_region(id);
        sleep(1); // simula trabalho fora da região crítica
    }
    return NULL;
}

int main() {
    pthread_t threads[NUM_THREADS];
    int ids[NUM_THREADS] = {0, 1};

    for (int i = 0; i < NUM_THREADS; ++i) {
        pthread_create(&threads[i], NULL, thread_function, &ids[i]);
    }

    for (int i = 0; i < NUM_THREADS; ++i) {
        pthread_join(threads[i], NULL);
    }

    return 0;
}

```


```
#include <stdio.h>
#include <pthread.h>
#include <semaphore.h>
#include <unistd.h>

sem_t semaforo; // o nosso semáforo

void* tarefa(void* arg) {
    int id = *((int*)arg);
    
    printf("Thread %d tentando entrar na região crítica...\n", id);
    
    sem_wait(&semaforo); // espera pelo semáforo (equivale ao wait/acquire)
    
    printf("Thread %d entrou na região crítica!\n", id);
    sleep(2); // simula trabalho dentro da região crítica
    printf("Thread %d saindo da região crítica.\n", id);
    
    sem_post(&semaforo); // libera o semáforo (equivale ao signal/release)
    
    return NULL;
}

int main() {
    pthread_t threads[2];
    int ids[2] = {1, 2};

    // inicializa o semáforo com valor 1 (semáforo binário = mutex)
    sem_init(&semaforo, 0, 1);

    // cria duas threads
    pthread_create(&threads[0], NULL, tarefa, &ids[0]);
    pthread_create(&threads[1], NULL, tarefa, &ids[1]);

    // espera as threads terminarem
    pthread_join(threads[0], NULL);
    pthread_join(threads[1], NULL);

    // destrói o semáforo
    sem_destroy(&semaforo);

    return 0;
}


```

```
#include <stdio.h>
#include <pthread.h>
#include <semaphore.h>
#include <unistd.h>

sem_t S, Q; // nossos dois recursos

void* P1(void* arg) {
    printf("P1 tentando pegar S...\n");
    sem_wait(&S);
    printf("P1 pegou S\n");

    sleep(1); // simula trabalho, dá tempo de P2 pegar Q

    printf("P1 tentando pegar Q...\n");
    sem_wait(&Q);
    printf("P1 pegou Q\n");

    printf("P1 executando região crítica...\n");
    sleep(1);

    sem_post(&Q);
    sem_post(&S);

    printf("P1 terminou e liberou os recursos.\n");
    return NULL;
}

void* P2(void* arg) {
    printf("P2 tentando pegar Q...\n");
    sem_wait(&Q);
    printf("P2 pegou Q\n");

    sleep(1); // simula trabalho, dá tempo de P1 pegar S

    printf("P2 tentando pegar S...\n");
    sem_wait(&S);
    printf("P2 pegou S\n");

    printf("P2 executando região crítica...\n");
    sleep(1);

    sem_post(&S);
    sem_post(&Q);

    printf("P2 terminou e liberou os recursos.\n");
    return NULL;
}

int main() {
    pthread_t t1, t2;

    sem_init(&S, 0, 1); // semáforo S
    sem_init(&Q, 0, 1); // semáforo Q

    pthread_create(&t1, NULL, P1, NULL);
    pthread_create(&t2, NULL, P2, NULL);

    pthread_join(t1, NULL);
    pthread_join(t2, NULL);

    sem_destroy(&S);
    sem_destroy(&Q);

    return 0;
}


```
