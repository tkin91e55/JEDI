  This is about DevOps, building, managing versions of JEDI and its models.

  When touching a new OSS project, have it built and run normally is the goal. Read the official doc
  is one of the fastest way to get started. Once the thing can run, other difficulties drop.

# Background

  * [JCSDA: Joint Center for Satellite Data Assimilation](https://www.jcsda.org/)
    * [JEDI: Joint Effort for Data assimilation Integration](https://www.jcsda.org/jcsda-project-jedi)
  * [MPAS: Model for Prediction Across Scales](https://mpas-dev.github.io/)
  * [MPAS-A and MPAS-JEDI tutorials](https://www.mmm.ucar.edu/events/133084/agenda)
    * [MPAS internal](https://www2.mmm.ucar.edu/projects/mpas/tutorial/Boulder2023/lectures/downloading_and_compiling.pdf)


  I guess, JCSDA is an organization, JEDI is a project, this project maintains different computation
  models to simulate atmosphere in different dimensions. MPAS is one of the models. That is all.

# Opinions

  Though opinions, but my experience... Most of the problems are resources missing, version
  mismatch, platform mismatch, authorization, authentication...

  DON'T cmake or build while installing/updating a lot unfamiliar **system packages**. Today software
  eco is too broad, no one is able to chain-up the whole layer, get the goal is more practical

  There can be Cmake, Ansible, Chef, TravisCI, Jenkinds, Shellscript and a lot for Continuous
  Integration (CI & DevOps). If possible, shell scripts hides the direction what to do next, just
  use shell (or Windows' .bat) scripts to do CI and DevOps. Of course, it has its own shortcomings.

  __Key points__:

  1. Read the doc, the doc is the best giving from the author, there is someone supports it and take
     the responsibility, why not rely on him? It took years for me to know, the original doc is the
     best as it comes from the original author, but just sometimes it is too elusive for newcomer.

     However, sometimes, if really desperate, the doc is not well-made or outdated, covering. Ready
     to get hands dirty, build directly and understand the sources. This takes really long time and
     most of cases is not necessary. For big and complex projects, this should not happen, there are
     always to people just responsible for doc.

     Unless going deeper and digging out the core structure of the whole project (which though is
     good), beginner and users can skip most of messing up internal dependency by following guide first
     ( this is not ensured to work out-of-the-box depending the user's environment: CPU archi, system
     authorization, hardware resources)

  1. Virtualisation: don't pollute own OS in development or let own OS stage pollute the build.
     Therefore, start with fresh installation of environment. If change own OS packages, it can
     break easily other application dependencies, it is a nightmare and very likely to happen in
     OpenSUSE (someday I need to learn about it, there is something I don't know yet).
     Virtualization provides isolated environment, today RAM and GB are affordable, 1TB's dollars
     are worth our work days.

     Also, vitalization tools are great that, it supports snapshot and is portable, easy for clone,
     duplicate, transfer to friends. Save time, but not space (dynamic programming, huh?!).

     For many times, installing on local system can corrupt local dependencies (system applications,
     other user applications etc). A havoc that need much time to fix, theoretically everything can be
     fixed, and each experience is a lesson.


  Building can be fun, but modern software often come with too many DevOps tools, it is hard
  for one to master all (and no need).

  See JEDI/Spack docs has given many guide about using virtualisation, it encourages that. I think
  this is its best giving to new-comers (like me). Thrive off!

  If need to reuse/rebuild/redeploy/upgrade/update the project, try to keep the workflow organization
  clean, and minimize the work for such, help yourselves and others. For example, separate own
  config for different targets by making clone of repos, keep a work binary copy for own/others,
  resource links can be dead in future.



  The environment has Cmake, Ctest, ecbuild
  * https://jointcenterforsatellitedataassimilation-jedi-docs.readthedocs-hosted.com/en/1.3.0/inside/jedi-components/mpas-jedi/build.html#mpas-bundle

# VM+PackageManager based approach

## References

  1. [Spack-statck for MPAS](https://spack-stack.readthedocs.io/en/1.3.1/)
      * general doc, too broad
      * the latest version is also good too, but stick to 1.3.1 first
      * Spack is package manager for HPC
  2. [Ubuntu20.04 prerequisite](https://spack-stack.readthedocs.io/en/1.3.1/NewSiteConfigs.html#prerequisites-ubuntu-20-04-one-off)
      * the most useful section, mostly follow here.
      * don't read latest version doc, read 1.3.1
  3. [mpas-bundle repo](https://github.com/JCSDA/mpas-bundle/tree/2.0.0)
      * need to have spack-stack installed


## Spec

  How to get the spec? Trial and error, final got it after wrestling with the doc.

  * Hardwares (any)
    * Intel based CPU
      * RAM: 32GB
      * Storage: 1TB
      * OS: Windows 10 Pro 64 bit
    * M2 based CPU
      * RAM: 32GB
      * Storage 512GB
      * macOS
  * OS: Ubuntu 20.04 Desktop
    * don't use version 22, failed to compile some packages
    * as compile need Qt and windows.h, don't use server version
    * download: https://releases.ubuntu.com/focal/
  * Virtalization: [Virtualbox](https://www.virtualbox.org/), QEMU etc
    * RAM: 8GB, Storage 64GB (can be less...check it)
  * [spack-stack version: 1.3.1](https://github.com/JCSDA/spack-stack/tree/release/1.3.1)
    * branch: release/1.3.1
  * mpas-bundle
    * branch: release/2.0.0
    * [mpas-bundle](https://github.com/JCSDA/mpas-bundle/tree/release/2.0.0)
  * openmpi: just use the version the docs mentioned


  __notes__:

  1. Macbook with M-series cores (mackbook M1, M2 etc) is in ARM architecture, need to emulate AMD/X86,
    the virtualization runs really slow. Emulations is hardware simulation like playing PS3, Switch
    on PC, performance cut.<br>
    VirtualBox doesn't support M-series Mac yet... need to use Home Intel workstation,
    Macbook virtualize x86/amd64 need emulation, much slower

  1. Need Ubuntu 20.04, not 22.04, to save trouble, installing wgrib2 has trouble to detect the Ubuntu
     22, later version of spack-stack 1.5.0 seems to have fixed it, but here stick to 1.3.1 first.
     check [here](https://github.com/JCSDA/spack-stack/blob/release/1.5.0-nco/configs/sites/hercules/packages.yaml#L24C32-L24C32)<br>
     spack-stack later version should also support backwards.<br>
      1. Need Desktop but not server version, as spack-stack requires Qt and windows.h
      1. though support 22.04, the doc has some problem with some packages, 20.04 is one-click install


## Steps

  There are two parts: [Installing spack-stack](#installing-spack-stack) then
  [Installing MPAS](#installing-mpas)

### Installing spack-stack

 1. Get VM, VM network (SSH) ready.
 1. Follow [reference](#references) `1.` to install ubuntu 20.04 prerequisites:

    ```bash
    # at root mode
    apt-get update
    apt-get upgrade

    apt install -y gcc g++ gfortran gdb

    apt install -y environment-modules

    # Misc
    apt install -y build-essential libkrb5-dev m4 git git-lfs \
    bzip2 unzip automake xterm texlive libcurl4-openssl-dev \
    libssl-dev mysql-server libmysqlclient-dev

    # Python
    apt install -y python3-dev python3-pip

    # reboot VM
    ```
 1. git clone the spack-stack repo and checkout version 1.3.1 and update the submodules

     ```bash
     git clone https://github.com/JCSDA/spack-stack
     git checkout -b release/1.3.1 origin/release/1.3.1
     git submodule init
     git submodule update #wait to download
     ```
 1. source the setup.sh, and spack the virtual env

     ```bash
     cd spack-stack
     source setup.sh

     #--name is up to you, but keep simple just follow the doc
     spack stack create env --site linux.default --template unified-env --name unified-dev.mylinux
     spack env acticvate -p envs/unified-dev.mylinux #after this, the have a prefix name appears before command prompt, hinting env activated

     pwd # display current path, for exmaple /home/dev/spack-stack
     export SPACK_SYSTEM_CONFIG_PATH="/home/dev/unified-dev.mylinux/site" # hard coded it, $PWD mentioned in doc should work too
     spack external find --scope system # it search a bit while, and added system packages to its spack env packages
     spack external find --scope system perl python wget mysql texlive # no need curl
     spack compiler find --scope system # remember the current version in the Ubuntu22.04, mine was 9.4.0
     unset SPACK_SYSTEM_CONFIG_PATH

     spack config add "packages:all:providers:mpi:[mpich@4.0.2]"
     spack config add "packages:all:compiler:[gcc@9.4.0]" #used own system compiler instead the 10.3.0
     ```

     Note the available site and templates are under: config/sites and config/templates, for different
     deployment env, ubuntu just use linux.default. As for HPC things, I don't know, other codenames
     are unfamiliar for me.

  1. edit envs/unified-dev.mylinux/spack.yml, find the line

     ```
     # definitions:
     # - compilers: ['clang_g++'...]
     replace as
     # - compilers: ['%gcc']
     ```

     For such thing generally, grep and replace throughout the project

  1. As for remove any external cmake@3.2.0, nothing to do after grepping
  1. concretize and install

     ```bash
     spack concretize # wait a while and display a tree of packages to install
     spack install --fail-fast # fail-fast will stop once there is build error, but this time, no error at all
     # just wait wait wait for success, no more desperation
     ```
  1. `spack module tcl refresh `, let it do something, it said success, good
  1. `spack stack setup-meta-modules`
  1.  reboot VM
  1. activate the spack env again, and

     ```bash
     spack env acticvate -p envs/unified-dev.mylinux
     # spack stack setup-meta-modules, need or not needed again?
     ```

     Now, check if `which module` or `module help` show something, then success.
     Make sure `module` is here, to be used next part.

### Installing MPAS

  This is to install MPAS by mpas-bundle with spack-stack. The thing left is simple:

  Refer to [reference](#references) `3.`

  1. make a folder like mpas2 first, go into it. Then git clone and checkout proper version

     ```bash
     mkdir mpas2 && cd mpas2
     git clone https://github.com/JCSDA/mpas-bundle.git && cd mpas-bundle
     git checkout -b release/2.0.0 origin/release/2.0.0
     ```
  1. go upper and create `build` folder for make and install

     ```bash
     cd .. && mkdir build && cd build
     ```
  1. now, the doc is outdated, but refer to the gnu-openmpi-cheyenne.sh<br>
     Actually all four script are mostly same, csh is for C-Shell (nevere used before)

     Inspect this (*DON'T DO ANYTHING*):
     ```bash
     echo "Loading Spack-Stack 1.3.1"
     source /etc/profile.d/modules.sh  # it looks for that cheyenne HPC, not relevant, ignore it
     module purge
     export LMOD_TMOD_FIND_FIRST=yes # just follow suit
     module use /glade/work/jedipara/cheyenne/spack-stack/modulefiles/misc # no such thing, for HPC only, not ubuntu dev
     module load miniconda/3.9.12 # the previous steps did not include this, no such package
     # up to here, can use `module available` to check previously installed build packages
     module load ecflow/5.8.4 # available, use it, that ecflow exists
     module load mysql/8.0.31 # ignore it, not here... I guess mpas not using mysql

     # the same effect as module use `./envs/jedi-ufs.mymacos/install/modulefiles/Core` at https://spack-stack.readthedocs.io/en/1.3.1/NewSiteConfigs.html#prerequisites-ubuntu-20-04-one-off
     module use /glade/work/epicufsrt/contrib/spack-stack/spack-stack-1.3.1/envs/unified-env/install/modulefiles/Core
     module load stack-gcc/10.1.0 # it is here, but different, mine is stack-gcc/9.4.0
     module load stack-openmpi/4.1.1  # it is here, but different
     module load stack-python/3.9.12 # it is here, but different
     module load jedi-mpas-env/unified-dev # it is here
     module list

     ulimit -s unlimited
     export GFORTRAN_CONVERT_UNIT='big_endian:101-200'
     ```

     Now, type manually (TODO, update this script)

     ```bash
     module purge
     export LMOD_TMOD_FIND_FIRST=yes
     module use /home/dev/spack-stack/envs/unified-env/install/modulefiles/Core
     module available # check the really available versions
     module load ecflow/x.x.x # x.x.x is your own version displayed
     module load stack-gcc/x.x.x
     module load stack-openmpi/x.x.x
     module load stack-python/x.x.x
     module load jedi-mpas-env/unified-dev
     module list # check again the packages to build MPAS2.0

     ulimit -s unlimited
     export GFORTRAN_CONVERT_UNIT='big_endian:101-200'
     ```
  1. at build/, after `module load`, double check now you have the `ecbuild`, e.g. `which ecbuild`

      ```bash
      ecbuild ../mpas-bundle
      make -j4

      # then wait wait wait... 100% complete, success
      ```
  1. run tests, I got 96% passed. May hint something wrong, or just the test cases outdated. Depend
     on what to do next, but it is outside my scope now.

       ```bash
       cd mpas-jedi
       ctest
       ```
  1. END HERE

# Cmake approach

  Cmake apporach is not against virtualization, isolated env cannot be less emphasized.

  * [Downloading and compiling MPAS](https://www2.mmm.ucar.edu/projects/mpas/tutorial/Boulder2023/lectures/downloading_and_compiling.pdf)
    * starting with [mpas-dev](https://mpas-dev.github.io), rather than spack mpas bundle (https://github.com/JCSDA/mpas-bundle)
      mpas-bundle wraps mpas-dev
    * some internal introduction, good
    * introduce good way (git clone) and bad way (direct download) to build from source.
      * neither, just follow higher level doc to build, and it is JEDI organization
  - [ ] Not tried yet, more TODO

# Others

  There are/were other options, it seems to have a paid approach to just use HPC or from AWS cloud.
  It looks expensive, not for learning/dev.

  * JEDI stack
    - [x] [JEDI stack](https://github.com/JCSDA/jedi-stack/blob/master/doc/Build.md)
      * maybe the same to spack-stack, this repo was 2 years, maybe deprecated
    * [Build and test MPAS-JEDI](https://jointcenterforsatellitedataassimilation-jedi-docs.readthedocs-hosted.com/en/latest/inside/jedi-components/mpas-jedi/build.html)

  * Singularity: container based, for HPC rather than Docker,container based platform for HPC, need
    to gain access first, proprietary thing, give money to solve problem,
    * [Installing Singularity](https://jointcenterforsatellitedataassimilation-jedi-docs.readthedocs-hosted.com/en/1.3.0/using/jedi_environment/singularity.html#singularity-install)
    * [JEDI Singularity](https://jointcenterforsatellitedataassimilation-jedi-docs.readthedocs-hosted.com/en/1.3.0/using/jedi_environment/singularity.html#singularity-install)
    - [ ] Not used before, what special about it?
      * What HPC CPU archi is? broad question
      * seem need to pay and for corporate...
    * [Singularity to build mpas2](https://jointcenterforsatellitedataassimilation-jedi-docs.readthedocs-hosted.com/en/latest/inside/jedi-components/mpas-jedi/build.html#building-mpas-bundle-in-singularity)
    * [Singluarity intro](https://jointcenterforsatellitedataassimilation-jedi-docs.readthedocs-hosted.com/en/1.3.0/using/jedi_environment/singularity.html)
    * [JEDI Singularity](https://jointcenterforsatellitedataassimilation-jedi-docs.readthedocs-hosted.com/en/1.3.0/using/jedi_environment/singularity.html)
    * https://jointcenterforsatellitedataassimilation-jedi-docs.readthedocs-hosted.com/en/1.3.0/learning/tutorials/level1/index.html
    * https://jointcenterforsatellitedataassimilation-jedi-docs.readthedocs-hosted.com/en/1.3.0/learning/tutorials/index.html
  * Vagrant, mentioned, a on-top layer to virtualization tools, orchestration tool to build a VM,
    not used recently
  * Docker, mentioned

# Ideas

  Some ideas sparked in mind

  * Global warming
  * Doomsday Clock
  * Species Extinction, War, Storm, Desert
  * Unity tutorial Hexgonal grids, how to represent hexagonal grid?
    * how actually to next(grid)? Or acutally no...
