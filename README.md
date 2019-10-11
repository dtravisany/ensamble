# ensamble
Práctico de Ensamble


## Resumen

En este práctico vamos a ensamblar un genoma bacteriano utilizando los programas [Celera/wgs-assembler](http://wgs-assembler.sourceforge.net/wiki/index.php?title=Main_Page) y [Velvet](https://www.ebi.ac.uk/~zerbino/velvet/). 

Luego, realizaremos la predicción de [CDS](https://www.uniprot.org/help/cds_protein_definition) a partir de los ensambles con la herramienta [Glimmer3.02](http://ccb.jhu.edu/software/glimmer/index.shtml) y, finalmente, a los péptidos predichos con Glimmer, se les asignará función putativa con el programa BLAST y la base de datos [Swiss-Prot](https://www.uniprot.org/statistics/Swiss-Prot).

 Cada grupo tendrá dos sets de lecturas de secuenciación, correspondientes a un genoma desconocido, el cual tendrán que inferir con los análisis.

## Objetivos del Práctico: 

- Familiarizarse con los conceptos de ensamble y anotación.
- Conocer el funcionamiento de herramientas bioinformáticas de ensamble y anotación.
- Adquirir práctica en entorno Unix. 




## Trabajo preliminar:

Conectarse al servidor.

Debido a que los cálculos que realizaremos en este práctico requieren un poder de cómputo moderado, nos conectaremos a uno de los servidores de [Mathomics](http://www.mathomics.cl). Si están en `Linux/MacOS`, puede utilizar la terminal (consola) estándar.

Si están en Windows (anterior a Windows 10) deben instalar un programa llamado `Putty`. Una vez abierta la terminal, cada grupo debe escribir lo siguiente:

	 ssh  usuario@servidor



Las credenciales se les entregaran en la pizarra. 
En el nombre de usuario y la contraseña, la `N` debe
ser reemplazada por el número del grupo que fue asignado.

#### Nombre de las carpetas:

Al conectarnos al servidor, entramos directamente al directorio de trabajo.
En este directorio están los archivos de lecturas de secuenciación que se le
asigno a cada grupo. 
Para poder ver estos archivos debemos escribir lo siguiente:

	ls

## Ensamble de Genomas:

### Celera - wgs-assembler - Canu

[Celera](http://wgs-assembler.sourceforge.net/wiki/index.php?title=Main_Page) es un ensamblador que utiliza [OLC](https://www.ncbi.nlm.nih.gov/pubmed/22184334). Consta de una fase de corrección de lecturas, una de sobrelape, de generación de contigs y finalmente de scaffolding. 

Antes de correr `Celera`, necesitamos generar los archivos de entrada y configuración.

Archivos frg:

`Celera` no recibe directamente archivos `FASTQ` como input, necesita un archivo del tipo `fragmentos`, el cual contiene la descripción de las lecturas a ensamblar. 
Existe un comando en `Celera` que nos permite hacer la transformación de FASTQ a fragmentos `frg` denominado `fastqToCA` (fastq to Celera Assembler).

	fastqToCA -insertsize 180 18 -libraryname over -type illumina -technology illumina -mates  gN.over.A.fastq,gN.over.B.fastq > gN.over.frg

Generar uno para la librería Jump:

	fastqToCA -insertsize 3000 300 -libraryname jump -type illumina -technology illumina -mates  gN.jump3kb.A.fastq,gN.jump3kb.B.fastq > gN.jump3kb.frg


Generar archivos de configuración:
	
Además de los archivos de fragmentos, `Celera` necesita un archivo con la
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


#### Ensamblar las lecturas:

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


### Velvet

`Velvet` es un ensamblador de lecturas cortas, por ende, utiliza el [grafo de Bruijn](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC5531759/).

#### Generar Grafo de Bruijn:

El comando para generar el grafo de Bruijn se llama `velveth`. Como input, debemos
entregarle el nombre de la carpeta donde se guardaran los archivos, el largo del 
`k-mer` para la construcción del grafo, el tipo de librería que se esta utilizando 
(en este caso pareadas) y la lista de lecturas a ensamblar.

	velveth velvet_ensamble 31 -shortPaired -fastq -separate gN.jump3kb.A.fastq gN.jump3kb.B.fastq  gN.over.A.fastq gN.over.B.fastq


## Obtener los contigs


Los contigs son obtenidos resolviendo los maximal unary paths o unitigs en el
grafo de Bruijn. El concepto de "resolver" unitigs hacer referencia a la búsqueda de [caminos eulerianos](https://es.wikipedia.org/wiki/Ciclo_euleriano) dentro del grafo. El comando de velvet para construir los
contigs/scaffolds se llama `velvetg`.

	velvetg velvet_ensamble -min_contig_lgth 1000


### Revisar los ensambles:

Los ensambles generan un archivo de estadísticas. En este archivo podremos ver los
resultados cuantitativos del ensamble.

En el caso de Celera el nombre del archivo es `sssss` y se encuentra en la carpeta `` 


En el caso de velvet este documento se encuentra en la carpeta de salida bajo el nombre de `stats.txt`.

Para este práctico he desarrollado un script en `perl` que calcula los stats primarios de un ensamble en base al resultado de los archivos de contigs.
 
Para el ensamble de velvet ejecutar:

	make_stats.pl  -i velvet_ensamble/contigs.fa

Para el ensamble de celera ejecutar:

	make_stats.pl ....




## Predicción y Anotación


### Predicción de CDS

Antes de poder anotar, necesitamos realizar la predicción de nuestros `CDS` para eso utilizaremos `Glimmer3.02`. 

`Glimmer` se ejecuta para un `FASTA` que contiene solo una secuencia, es decir, si tenemos más de una secuencia en un `FASTA`, Glimmer Haría la predicción de la primera secuencia. Para resolver, esto he programado un pipeline en `perl` que puede ejecutar de la siguiente manera:

	pipe2gbk.pl -i velvet_ensamble/contigs.fa -p peptidos_velvet

Para Celera:

	pipe2gbk.pl -i celera_ensamble/contigs.fa -p peptidos_celera

Para cada ensamble obtendrá dos archivos `peptidos_velvet.faa` y `peptidos_velvet.gbk`. Lo mismo para el caso de `celera`. 

Por ahora continuaremos trabajando solo con los péptidos.

### Anotación

Como BLAST demora en anotar sus péptidos, es necesario que ejecute esta instrucción en un `screen`, puede utilizar alguno de los screen anteriores o crear uno nuevo utilizando el comando:

	screen -S blast

En este screen debe ejecutar la instrucción:

	blastp -db /home/dbioA/databases/SWISSPROT/uniprot_sprot.fasta -query peptidos_velvet.faa -out velvet_blast.txt -evalue 1e-5 -num_threads 4

Realizar el mismo comando pero ahora con los resultados de `celera`.



Ahora tenemos dos archivos velvet_blast.txt y celera_blast.txt, ambos contienen los alineamientos en formato raw,
 pero deben ser "parseados" para poder realizar analisis posteriores (Por ejemplo: tener un excel con la anotación),
 para esto he generado un parseador que se llama `blastparser.pl` y esta basado en los módulos [Bio::SearchIO](https://metacpan.org/pod/Bio::SearchIO)
 y [Bio::SeqIO](https://metacpan.org/pod/Bio::SeqIO) de [BioPerl](https://bioperl.org/).





