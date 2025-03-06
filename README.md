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


