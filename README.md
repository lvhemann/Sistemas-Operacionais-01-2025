# Sistemas-Operacionais-01-2025

## Modos de Usuário 

```bash  
  ps aux
```

```bash 
ps -eo pid,comm,pri,ni,pcpu,state
```


```bash 
sudo apt install htop –y
```

```bash 
htop
```

## Realização de um Read

```bash 
#include <stdio.h>
#include <stdlib.h>
#include <fcntl.h>
#include <unistd.h>
#include <errno.h>

int main(){
	int fd;
	char buffer[100];
	ssize_t contador;

	fd = open("arquivo.txt", O_RDONLY);
	if (fd == -1){
		perror("erro ao abrir o arquivo");
		return 1;
	}

	contador = read(fd, buffer, sizeof(buffer) - 1);
	if(contador == -1){
		perror("erro ao ler o arquivo");
		close(fd);
		return 1;
	}

	buffer[contador] =  '\0';
	
	printf("Conteudo de arquivo:\n%s\n", buffer);
	close(fd);
	return 0;
}
```

```bash 
echo "Este é um teste de leitura." > arquivo.txt
```

```bash 
gcc read_example.c -o read_example 
```

```bash 
./read_example
```

## Outros Testes
# Excluir o arquivo .txt

```bash
rm arquivo.txt
```

# Adicionar mais informação ao arquivo
```bash 
echo "Linha 1" > arquivo.txt
```

## Exemplo de Proteção Usando Anéis
```bash
cat /dev/mem
```

```bash
sudo hexdump -C /dev/mem | head
```

## Exemplo usando chamadas de sistema
```bash
#include <stdio.h>
#include <stdlib.h>
#include <fcntl.h>    // Para open()
#include <unistd.h>   // Para read(), write(), close()
#include <errno.h>    // Para lidar com erros

#define BUFFER_SIZE 1024  // Tamanho do buffer de leitura

int main(int argc, char *argv[]) {
    if (argc != 3) {
        fprintf(stderr, "Uso: %s <arquivo_origem> <arquivo_destino>\n", argv[0]);
        return 1;
    }

    int fd1, fd2;  // File descriptors
    char buffer[BUFFER_SIZE];
    ssize_t bytes_lidos, bytes_escritos;

    // Abrir arquivo de origem (fd1) no modo leitura
    fd1 = open(argv[1], O_RDONLY);
    if (fd1 == -1) {
        perror("Erro ao abrir o arquivo de origem");
        return 1;
    }

    // Criar/abrir arquivo de destino (fd2) no modo escrita
    fd2 = open(argv[2], O_WRONLY | O_CREAT | O_TRUNC, 0644);
    if (fd2 == -1) {
        perror("Erro ao criar o arquivo de destino");
        close(fd1);
        return 1;
    }

    // Loop de leitura e escrita
    while ((bytes_lidos = read(fd1, buffer, BUFFER_SIZE)) > 0) {
        bytes_escritos = write(fd2, buffer, bytes_lidos);
        if (bytes_escritos != bytes_lidos) {
            perror("Erro ao escrever no arquivo de destino");
            close(fd1);
            close(fd2);
            return 1;
        }
    }

    if (bytes_lidos == -1) {
        perror("Erro ao ler o arquivo de origem");
    }

    // Fechar os arquivos
    close(fd1);
    close(fd2);

    printf("Cópia concluída com sucesso!\n");
    return 0;
}
```

```bash
echo "Este é um teste de cópia via syscalls!" > arquivo_origem.txt
```

```bash
gcc cp_syscalls.c -o cp_syscalls
```

```bash
./cp_syscalls arquivo_origem.txt arquivo_destino.txt
```

```bash
cat arquivo_destino.txt
```

## Intercepta e exibe todas as chamadas de sistema feitas pelo cp_syscalls
```bash
strace ./cp_syscalls arquivo_origem.txt arquivo_destino.txt
```

## Segmentação de memória

```bash
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

void funcao_pilha() {
    int var_pilha = 10;
    printf("Endereço da variável na Stack: %p\n", (void*)&var_pilha);
}

int main() {
    void *heap_inicio = sbrk(0);
    printf("Heap inicial: %p\n", heap_inicio);

    int *ptr = malloc(100 * sizeof(int)); // Alocação dinâmica
    printf("Endereço alocado na Heap: %p\n", (void*)ptr);

    funcao_pilha(); // Chamada para ver a pilha crescer

    free(ptr); // Libera memória
    return 0;
}

```

```bash
gcc memoria.c -o memoria
```

```bash
./memoria
```

