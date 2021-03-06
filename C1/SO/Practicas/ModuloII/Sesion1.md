## Ejercicio 1
1. Abre un archivo, y comprueba que ha sido correctamente abierto. Si no se ha conseguido abrir, se cierra
2. Escribe el contenido del array buff `(abcdefghij)`. Si no se ha conseguido escribir, se cierra
3. Posiciona el cursor en la parte inicial del archivo
4. Escribe ahora el contenido del segundo array en el archivo. Si no lo ha conseguido, cierra el programa
5. Finaliza la ejecución

## Ejercicio 2
Implementa un programa que realice la siguiente funcionalidad. El programa acepta como argumento el nombre de un archivo (pathname), lo abre y lo lee en bloques detamaño 80 Bytes, y crea un nuevo archivo de salida, salida.txt, en el que debe aparecer la siguiente información:
- Bloque 1:

  // Aquí van los primeros 80 Bytes del archivo pasado como argumento.

- Bloque 2

  // Aquí van los siguientes 80 Bytes del archivo pasado como argumento.

- Bloque n

  // Aquí van los siguientes 80 Bytes del archivo pasado como argumento.

Si no se pasa un argumento al programa se debe utilizar la entrada estándar como archivo de entrada

```c
#include<unistd.h>
#include<stdio.h>
#include<stdlib.h>
#include<sys/types.h>  	
#include<sys/stat.h>
#include<fcntl.h>
#include<errno.h>
#include <string.h>

int main(int argc, char * argv[]){
    if (argc != 2){
        perror("Parámetros equivocados");
        exit(-1);
    }

    int fd_in;
    if ((fd_in=open(argv[1], O_RDONLY,S_IRUSR|S_IWUSR)) < 0){
        perror("Error al abrir el archivo de entrada");
        exit(-1);
    }

    int fd_out;
    if ((fd_out=open("salida.txt", O_CREAT|O_TRUNC|O_WRONLY,S_IRUSR|S_IWUSR)) < 0){
        perror("Error al abrir el archivo de salida");
        exit(-1);
    }

    setvbuf(stdout,NULL,_IONBF,0);
    
    char buffer[80];
    char mensaje[10];
    
    for (int i=1; read(fd_in, buffer, 80) == 80; i++){
        sprintf(mensaje, "\nBloque %d\n", i);
        write(fd_out, mensaje, 10);
        write(fd_out, buffer, 80);
    }

    exit(0);
}
```

## Ejercicio 3
El programa recibe el nombre de un elemento, y analiza de qué tipo es. Para ello, invoca un struct del tipo `stat`, y compara poco a poco qué flag se activa.

- `S_ISREG` => Es regular
- `S_ISCHR` => Especial de caracteres
- `S_ISDIR` => Es un directorio
...

Si alguno de esos bits se ha activado, copia en la cadena `tipoArchivo` el mensaje tras cada `if()`.

## Ejercicio 4
Una macro es una pieza de código que recibe un nombre. Cada vez que su nombre se usa, se reemplaza por lo que sea que venga tras su definición.
Lo que necesitamos construir es una minifunción booleana, que nos devuelva verdadero o falso.

Para saber los permisos de un archivo, podemos crear un `struct stat` para comprobarlos. Sin embargo, existe un problema: los permisos no vienen como strings, sino en binario. Por tanto, debemos hacer comparaciones de lo que queramos.

En este caso, necesitamos hacer una comparación del parámetro `.st_mode` y `0170000`. Al hacer una operación AND, y el resultado es `0100000`, estamos ante un archivo regular.
`0170000` corresponde a `S_IFMT`. Es una máscara de bits para los campos de bit del tipo de archivo
`0100000` implica que el archivo es regular.

Por tanto, reutilizando el código del ejercicio 3, podemos crear lo siguiente:
```c
#include<unistd.h>
#include<stdio.h>
#include<stdlib.h>
#include<sys/types.h>  
#include<sys/stat.h>
#include<stdio.h>
#include<errno.h>
#include<string.h>

#define S_ISREG2(mode) ( (mode & 0170000) == 0100000)

int main(int argc, char const *argv[]) {
    int i;
	struct stat atributos;
	char tipoArchivo[30];

	if(argc<2) {
		printf("\nSintaxis de ejecucion: tarea2 [<nombre_archivo>]+\n\n");
		exit(EXIT_FAILURE);
	}

	for(i=1;i<argc;i++) {
		printf("%s: ", argv[i]);

		if(lstat(argv[i], &atributos) < 0) {
			printf("\nError al intentar acceder a los atributos de %s",argv[i]);
			perror("\nError en lstat");
		}

		else if(S_ISREG2(atributos.st_mode)) printf("Regular");
        else printf("No es un archivo regular");
    }
    exit(EXIT_SUCCESS);
}
```
