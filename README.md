---
marp: true
title: datarmor_training
theme: theme-den
paginate: true
---

# Datarmor Training

## Nicolas Barrier ([nicolas.barrier@ird.fr](mailto:nicolas.barrier@ird.fr))

---

# What is Datarmor?

Datarmor is a High-Performance Computer (HPC): 
- CPU : 11088 cores (My PC: 12 cores)
- RAM: 128 Go (My PC: 32 Go)
- Storage: 5 Po = 5000 To (My PC: 1 To).
- Hosting of web services (Jupyter Notebooks, visualuzation nodes)

Source: [Ifremer](https://domicile.ifremer.fr/intraric/Mon-IntraRIC/Calcul-et-donnees-scientifiques/,DanaInfo=w3z.ifremer.fr,SSL+Qu-est-ce-que-Datarmor)

---

# Pulse Secure: install

Outside the Ifremer Network, PulseSecure is required. It can be downloaded [here](https://domicile.ifremer.fr/index.php/s/WC9GArY8Eo51yZE/,DanaInfo=cloud.ifremer.fr,SSL+download?path=\%2F&files). 

Install the right version depending on your OS:

- Windows 64: `PulseSecure.x86.msi`
- Mac Os X 64: `PulseSecure.dmg`
- Ubuntu 20.04: `pulsesecure_9.1.R12_amd64.deb`
- Ubuntu 18.04: `pulse-9.1R8.x86_64.deb`

---

# Pulse Secure: set-up

Now set-up a new connection as follows:

<div align="center">
    <img src="figs/screenshot_pulse_secure.png">
</div>

Connect to the Datarmor VPN using your **extranet** logins.

---
# Connection: Terminal (Linux / Mac Os X)

For Linux/Mac Os X users, open a Terminal and types:
 
```
ssh -X nbarrier@datarmor.ifremer.fr
```

replacing `nbarrier` by your **intranet** login. The `-X` option allows display (for use of text editors for instance).


> For Mac Os X users, I recommend to install and use [iTerm2](https://iterm2.com/) Terminal application, which is more user friendly than the default one.

---

# RSA keys (Linux / Mac Os X)

To connect on Datarmor without typing the password, you need to use a RSA key. First, check if one already exists:

``` {.csh language="csh"}
ls $HOME/.ssh/id_rsa.pub
```

If no such file, generate a key using

``` {.csh language="csh"}
ssh-keygen  
```

and follows instructions. Then, send it to Datarmor:

```
ssh-copy-id nbarrier@datarmor.ifremer.fr
```

---


# Connection: Putty (Windows)

For Windows Users, it is recommended to use [Putty](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html) 

<div align="center">
    <img height=500 src="figs/Capture_putty.PNG">
</div>

---

# Navigating in Datarmor

Datarmor is a Unix computer. You need some Linux background.

- Change directory: `cd my/new/directory`
- Go to parent directory: `cd ..`
- Create new folder: `mkdir -p folder_name`
- Copy a file: `cp file.txt save_file.txt`
- Rename/Move a file: `mv file.text my/dest/renamed.txt`

Visit [linux-commands-cheat-sheet](https://linoxide.com/linux-commands-cheat-sheet/) for a summary of essentials Linux commands.


---


# Important folders

Important folders are:

-   `$HOME`: main folder (50 Mo, backed-up). For codes and important things
-   `$DATAWORK`: data folder (1 To, **no back-up**). Contains data files (NetCDF).
-   `$SCRATCH`: temporary folder (10 To, cleaned every 10 days). Used to run the computation.

In general, computation should follow these steps: 
- Copy codes from `HOME` to `SCRATCH`
- Copy data from `DATAWORK` to `SCRATCH`
- Go to `SCRATCH` and run the computatiob
- Copy output files from `SCRATCH` to `DATAWORK`

---

# Modules

To work with external tools, you need to load them into Datarmor's memory. This is done as follows:

``` {.csh language="csh"}
module load R   # load one module
module load java NETCDF  #  load 2 modules
module load vacumm/3.4.0-intel  # load a specific version
```

To list all the available modules:

``` {.csh language="csh"}
module avail
```

To list the modules that you use:

``` {.csh language="csh"}
module list
```


---

# Modules


To deactivate a module:

``` {.csh language="csh"}
module unload R
```

To unload all the modules at once:

``` {.csh language="csh"}
module purge
```

---

# Default settings

To change some default behaviours, you need to create/edit the Linux configuration file.

This can be done as follows:

```
gedit ${HOME}/.cshrc $
```

> The `&` character implies that you will keep access to your terminal. Else, the terminal will be back once the text editor is closed.

---

# Default settings

In the `.cshrc` file, you can define new shortcuts:

```{.csh language="csh"}
# nouveaux raccourcis
alias x 'exit'
alias rm 'rm -i -v'
alias cp 'cp -i -v -p'
alias mv 'mv -i -v'
```

You can also create environment variables:

```{.csh language="csh"}
setenv R_LIBS_USER $HOME/libs/R/lib
```

Now, you can access the above folder using `cd $R_LIBS_USER`

You can also load your favorite modules in here:

```
module load NETCDF
```

---

# Lancement de Job: interactif

Pour se connecter de maniere interactive sur un noeud de calcul, il faut
taper dans le terminal:

``` {.csh language="csh"}
qsub -I -l walltime=30:00:00 -lmem=100g
```

L'argument `-lmem` specifie la memoire demandee, l'argument
`-l walltime` le temps de calcul demande.

---

# Running a job: PBS

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

---

# Running a sequential job

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

---

# Running a parralel job

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

---

# Following your jobs

``` {.csh language="csh"}
# suivi des jobs pour un utilisateur donne
qstat -u nbarrier
    
# supprimer un JOB
qdel JOB_ID   # job ID est fourni par qstat
```

Le premier argument est le nom du dossier Datarmor que vous voulez
monter, le deuxieme est le dossier destination dans lequel ce montage
sera fait.

---

# Echange de donnee avec/depuis datarmor

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

---

# Environnement logiciel
 
Afin de mettre en place des environnements logiciels non disponibles via les modules, il faut utiliser
le gestionnaire de paquets multilangages **conda** (cf. [Conda sur Datarmor](https://domicile.ifremer.fr/intraric/Mon-IntraRIC/Calcul-et-donnees-scientifiques/Datarmor-Calcul-et-Donnees/Datarmor-calcul-et-programmes/Pour-aller-plus-loin/,DanaInfo=w3z.ifremer.fr,SSL+Conda-sur-Datarmor)). 

Une introduction generale a conda est disponible [ici](https://github.com/umr-marbec/python-training/blob/master/introduction/libinstall.ipynb)

Pour l'activer sur Datarmor, ajouter dans son fichier `.cshrc`:

```
source /appli/anaconda/latest/etc/profile.d/conda.csh
```

---

# Environnement logiciel

Afin de recuperer les environnements conda deja installes sur Datarmor, creer le fichier `~/.condarc` et y mettre:

```
envs_dirs:
- /home1/datahome/nbarrier/softwares/anaconda3-envs
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


Les nouveaux environnements vont etre installes dans `/home1/datahome/nbarrier/softwares/anaconda3-envs`.

---


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

---

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
