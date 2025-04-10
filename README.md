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
