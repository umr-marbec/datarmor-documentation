Documentation Datarmor
==========================

# Connection

## Windows

Connection via [Putty](https://www.putty.org/) (login + mdp intranet
ifremer)

<div align="center">
    <img src="figs/Capture_putty.PNG">
</div>

<!-- Echange de donnees via [FileZilla](https://filezilla-project.org/).

![image](figs/filezilla_short.png)
-->

## Linux / Mac Os X

Connection et echange possible via Putty, mais aussi via le terminal:

``` {.csh language="csh"}
# connexion (login + mdp intranet ifremer)
ssh nbarrier@datarmor.ifremer.fr
```

Pour se connecter a la machine sans mot de passe, il faut utiliser un chiffrage RSA:

``` {.csh language="csh"}
# generate a RSA public/private key pair
# if you have one, skip this step
ssh-keygen  

# send the key to your datarmor account
ssh-copy-id nbarrier@datarmor.ifremer.fr
```

# Dossiers importants

Les dossiers importants sont:

-   `$HOME`: dossier de travail (50 Mo, sauvegarde). Pour les codes et
    les choses importantes

-   `$DATAWORK`: espace de travail, plus d'espace, non sauvegarde (pour
    les donnees)

-   `$SCRATCH`: espace temporaire utilise pour faire tourner les
    modeles. Grosse memoire, mais effacee de maniere reguliere (ne rien
    y laisser)

En general, les jobs doivent tourner sur `$SCRACTH`, apres y avoir copie
les codes depuis `$HOME` et les donnees depuis `$SCRATCH`. Mais
possibilite de faire tourner les codes depuis `$DATAWORK` directement.

# Modules

Pour lister les differents modules disponibles:

``` {.csh language="csh"}
module avail
```

Pour lister les differents modules de charges:

``` {.csh language="csh"}
module list
```

Pour charger un module:

``` {.csh language="csh"}
module load R   # charge un module
module load java NETCDF  #  charge 2 modules a la fois
module load vacumm/3.4.0-intel   # charge une version specifique
```

Pour desactiver un module:

``` {.csh language="csh"}
module unload R
```

Pour enlever tous les modules charges:

``` {.csh language="csh"}
module purge
```

# Reglages par default

Pour changer le comportement par default, il faut creer/editer le
fichier `${HOME}/.cshrc`.

On peut soit ajouter de nouveaux raccourcis (alias) soit changer le comportement
par defaut de certaines commandes

```{.csh language="csh"}
# nouveaux raccourcis
alias x 'exit'
alias c 'clear'
        
# remplacer le comportement par defaut
# evite l'ecrasement de fichiers existant lors
# des copies/renommage/deplacements et effacements
alias rm 'rm -i -v'
alias cp 'cp -i -v -p'
alias mv 'mv -i -v'
```

On peut aussi definir des variables d'environnement supplementaires (par
exemple lieu d'installation des librairies R):

```{.csh language="csh"}
setenv R_LIBS_USER $HOME/libs/R/lib
```

# Lancement de Job

Des exemples de scripts de lancement de jobs se trouvent dans le dossier
datarmor `/appli/services/exemples/`

## Interactif

Pour se connecter de maniere interactive sur un noeud de calcul, il faut
taper dans le terminal:

``` {.csh language="csh"}
qsub -I -l walltime=30:00:00 -lmem=100g
```

L'argument `-lmem` specifie la memoire demandee, l'argument
`-l walltime` le temps de calcul demande.

## Bash mode

Pour faire tourner des codes de maniere non interactive, il faut ecrire
un script de lancement de job, qui sera lance avec la commande `qsub`:

``` {.csh language="csh"}
qsub run_script.pbs
```

Les sorties du job seront mises dans le fichier `run_script.pbs.oXXXX`,
avec `XXXX` l'identification du job.

Des exemples sont fournis par datarmor dans le dossier
`/appli/services/exemples/` (dossiers `R` et `pbs`). Deux exemples sont
fournis ci-dessous, un sequentiel et un en parallel.

**Script sequentiel (lancement script R)**

``` {.csh language="csh"}
#!/bin/csh
#PBS -l mem=1g
#PBS -l walltime=00:30:00
# First line defines the memory used
# Second line defines the "walltime" (if time > walltime, job is killed)

# go to the directory where the job has been run
# cd $PBS_O_WORKDIR

# copy the code into the SCRATCH dir
cp -pr $HOME/code $SCRACTH

# copy the input data into the SCRATCH dir
cp -pr $DATAWORK/data $SCRACTH

# move to the SCRATCH directory
cd $SCRATCH

# Load the modules that will be used to do the job
source /usr/share/Modules/3.2.10/init/csh
module load R

# indique ou trouver les librairies R installees a la main.
setenv R_LIBS_USER $HOME/libs/R/lib

# Run R
date
Rscript script.R >& output.log  # redirects outputs into log
date
```

**Script en parralel (lancement programme MPI)**:

``` {.bash language="csh"}
#!/bin/csh
#PBS -q mpi_2
#PBS -l walltime=00:05:00
# example of using 2node, i.e. 2*28 mpi procs
# cd to the directory you submitted your job
cd $PBS_O_WORKDIR

source /usr/share/Modules/3.2.10/init/csh
module load   NETCDF/4.3.3.1-mpt-intel2016
 
setenv mpiproc `cat $PBS_NODEFILE  |wc -l`
echo "submit job with  $NETCDF_MODULE "
echo "job running with  $mpiproc mpi process "

date
time $MPI_LAUNCH exe >& out
date
```

Commandes utiles pour suivre les jobs:

``` {.csh language="csh"}
# suivi des jobs pour un utilisateur donne
qstat -u nbarrier
    
# supprimer un JOB
qdel JOB_ID   # job ID est fourni par qstat
```

Le premier argument est le nom du dossier Datarmor que vous voulez
monter, le deuxieme est le dossier destination dans lequel ce montage
sera fait.

# Echange de donnee avec/depuis datarmoor

Pour echanger des donnees entre Datarmor et une machine en local, il faut
par le dossier `$SCRATCH/eftp` de Datarmor.

En local, naviguer depuis le terminal dans le dossier source/destination en utilisant `cd`

Puis se connecter au FTP en tapant:

```
ftp eftp.ifremer.fr
```

Renseigner ses identifiants **extranet** puis taper:

```
cd scratch   # WARNING, DON'T FORGET!
prompt  # activate/deactivate interactive mode
get file.nc  # send $SCRATCH/eftp/file.nc to the local directory
mget *  # send all files in $SCRATCH/eftp/ to the local directory
put file.nc  # send local file.nc to the $SCRATCH/eftp/
mput *  # send all files in local directory to the $SCRATCH/eftp
```

**Note: `eftp.ifremer.fr` est aussi accessible en passant par FileZilla. **
    
# Environnement logiciel
 
Afin de mettre en place des environnements logiciels non disponibles via les modules, il faut utiliser
le gestionnaire de paquets multilangages **conda** (cf. [Conda sur Datarmor](https://domicile.ifremer.fr/intraric/Mon-IntraRIC/Calcul-et-donnees-scientifiques/Datarmor-Calcul-et-Donnees/Datarmor-calcul-et-programmes/Pour-aller-plus-loin/,DanaInfo=w3z.ifremer.fr,SSL+Conda-sur-Datarmor)). 

Une introduction generale a conda est disponible [ici](https://github.com/umr-marbec/python-training/blob/master/introduction/libinstall.ipynb)

Pour l'activer sur Datarmor, ajouter dans son fichier `.cshrc`:

```
source /appli/anaconda/latest/etc/profile.d/conda.csh
```

Afin de recuperer les environnements conda deja installes sur Datarmor, creer le fichier `~/.condarc` et y mettre:

```
envs_dirs:
- /appli/conda-env
- /appli/conda-env/2.7
- /appli/conda-env/3.6
pkgs_dirs:
- $DATAWORK/conda/pkgs
```

Pour lister les environnements conda disponibles, taper:

```
conda env list
```

Afin de definir un dossier contenant les futurs environnements, ajouter le dossier en question dans le fichier `.condarc` **en haut de la liste**.
Par exemple, pour installer les environnements dans son $HOME:

```
envs_dirs:
- /home1/datahome/nbarrier/softwares/anaconda3-envs
- /appli/conda-env
- /appli/conda-env/2.7
- /appli/conda-env/3.6
pkgs_dirs:
- $DATAWORK/conda/pkgs
```

Les nouveaux environnements vont etre installes dans `/home1/datahome/nbarrier/softwares/anaconda3-envs`.

Pour activer/desactiver un environnement virtuel:

```
conda activate pyngl
conda deactivate pyngl
```

Conda est principalement utilise avec Python, mais peut aussi etre utilise avec R. Pour creer un environnement virtuel pour R:

```
conda create -n r-env r-base 
```

Le nouvel environnement cree s'appelle `r-env` et contient le package `r-base`

Noter que les packages R commencent par le prefixe `r-`.

# Telechargement de donnees depuis Datarmor

Pour recuperer des donnes sur un FTP depuis Datarmor, il faut:
- soit passer par un job soummis sur la queue FTP en specifiant `-q ftp`
- soit se connecter sur la machine `datasession0` en tapant `ssh datasession0`

Ci dessous un exemple de fichier `.pbs` permettant de telecharger des donnees depuis l'exterieur 
(inspire de `/appli/services/exemples/pbs/ftp.pbs`)

```
#!/bin/csh
#PBS -q ftp
#PBS -l walltime=02:15:00

cd $PBS_O_WORKDIR

# time lftp ... >& output
# time rsync -av /chemin/source/ login@server:/autre/chemin/destination >& output
```

