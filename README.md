# ensamble
Práctico de Ensamble

Trabajo preliminar:

Conectarse al servidor.

Debido a que los cálculos que realizaremos en este práctico requieren un poder de cómputo moderado, nos conectaremos a uno de los servidores de [Mathomics](www.mathomics.cl). Si están en `Linux/MacOS`, puede utilizar la terminal (consola) estándar.

Si están en Windows (anterior a Windows 10) deben instalar un programa llamado `Putty`. Una vez abierta la terminal, cada grupo debe escribir lo siguiente:

	 ssh  usuario@servidor



Las credenciales se les entregaran en la pizarra. 
En el nombre de usuario y la contraseña, la `N` debe
ser reemplazada por el número del grupo que fue asignado.

Nombre de las carpetas:

Al conectarnos al servidor, entramos directamente al directorio de trabajo.
En este directorio están los archivos de lecturas de secuenciación que se le
asigno a cada grupo. 
Para poder ver estos archivos debemos escribir lo siguiente:

	`ls`

Ensamble de Genomas:

Celera - wgs-assembler - Canu

[Celera]() es un ensamblador que utiliza [OLC](). Consta de una fase de corrección de lecturas, una de sobrelape, de generación de contigs y finalmente de scaffolding. 

Antes de correr `Celera`, necesitamos generar los archivos de entrada y configuración.

Archivos frg:

Celera no recibe directamente archivos FASTQ como input, necesita un archivo del tipo `fragmentos`, el cual contiene la descripción de las lecturas a ensamblar. 
Existe un comando en `Celera` que nos permite hacer la transformación de FASTQ a fragmentos `frg` denominado `fastqToCA` (fastq to Celera Assembler).

	fastqToCA -insertsize 180 18 -libraryname over -type illumina -technology illumina -mates  gN.over.A.fastq,gN.over.B.fastq > gN.over.frg

Generar uno para la librería Jump:

	fastqToCA -insertsize 3000 300 -libraryname jump -type illumina -technology illumina -mates  gN.jump3kb.A.fastq,gN.jump3kb.B.fastq > gN.jump.frg











