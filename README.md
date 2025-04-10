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
