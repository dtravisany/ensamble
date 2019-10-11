# ensamble
Práctico de Ensamble

Trabajo preliminar:

Conectarse al servidor.

Debido a que los cálculos que realizaremos en este práctico requieren un poder de cómputo moderado, nos conectaremos a uno de los servidores de [Mathomics](http://www.mathomics.cl). Si están en `Linux/MacOS`, puede utilizar la terminal (consola) estándar.

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

	ls

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


Generar archivos de configuración:
	
Además de los archivos de fragmentos, Celera necesita un archivo con la
configuración del ensamble (Parámetros). Para crear este archivo ejecutamos:


		vim celera.specf

Aparecerá un editor de texto (Vim: Vi Improved, Vi: Visual). Presionamos la tecla “i”
(insert text) y escribimos lo siguiente:

		ovlHashBits=23
		ovlHashBlockLength=30000000
		ovlRefBlockSize=7630000
		frgCorrBatchSize= 1000000
		frgCorrThreads= 4
		/home/dbioN/gN.over.frg
		/home/dbioN/gN.jump3kb.frg
	

Para guardar el archivo presionamos la tecla `Esc` (salir del modo insertar texto,
nos pondrá en modo comandos) , luego la tecla `:` (permite escribir comandos) y
finalmente escribimos `wq` (w: write, q:quit ) y presionamos `Enter` (Ejecutar)


Ensamblar las lecturas:

Debido a que el proceso de ensamblar lecturas puede tomar un tiempo prolongado,
ejecutar el comando de ensamble de la manera habitual es inconveniente, ya que si
cerramos la ventana de la consola, el proceso terminará también, por ende,
tendríamos que esperar que el ensamble terminara para cerrar la consola. En el caso
de que nos desconectaramos de internet/red, perderíamos lo que llevamos
ejecutando. Una solución a este problema es la comando screen.


		screen -S gN_celera

Luego, Ejecutamos el comando runCA de Celera para ensamblar:


		runCA -d celera_asm -p primer_ensamble -s celera.specf


Para cerrar la consola sin matar el proceso, tecleamos `Ctrl`+ `a` + `d`. 
Si queremos recuperar la consola donde lanzamos el programa 
escribimos lo siguiente:


		screen -r gN_celera



Velvet

Velvet es un ensamblardor de lecturas cortas, por ende, utiliza el grafo de Bruijn.

Generar Grafo de Bruijn

El comando para generar el grafo de Bruijn se llama `velveth`. Como input, debemos
entregarle el nombre de la carpeta donde se guardaran los archivos, el largo del 
`k-mer` para la construcción del grafo, el tipo de librería que se esta utilizando 
(en este caso pareadas) y la lista de lecturas a ensamblar.


Obtener los contigs


Los contigs son obtenidos resolviendo los >maximal unary paths o unitigs en el
grafo de Bruijn. El concepto de “resolver” unitigs hacer referencia ala búsqueda de
caminos Eulerianos dentro del grafo. El comando de velvet para onstruirs los
contigs/scaffolds se llama velvetg.


Revisar los ensambles:

Los ensambles generar un archivo de estadísticas. En este archivo podremos ver los
resultados cuantitativos del ensamble.
Resultados de Velvet:

Debemos entrar ala carpeta del ensamble. En este caso el archivo de estadísticas se
llama “stats.txt”.


