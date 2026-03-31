# CVM-FS software

CVMFS package is installed and configured on all nodes.  
See [CVMFS documentation](https://cvmfs.readthedocs.io/en/stable/) for more information

There are a number of repositories with packages prepared for usage under varios platforms but mostly Alma9(EL9)

## SFT
Information taken from `/cvmfs/sft.cern.ch/README.md`

!!! info "PREQUESITES"

    A number of basic packages need to be present on your machine in order for the LCG releases to work.
    More information can be found at [CERN's GitLab](https://gitlab.cern.ch/linuxsupport/rpms/HEP_OSlibs).


!!! info "Usage"

    For most of our distributed software you can find both a Bash script (ending in `.sh`) as well as a
    C Shell script (ending in `.csh`). By using the `source` command in your local shell with these
    scripts you can change your current shell environment to use the software on CVMFS instead of your
    locally installed packages. These changes only last until you close the current terminal session.

!!! example "Example"

    ```
    source /cvmfs/sft.cern.ch/lcg/views/LCG_107a/x86_64-el9-gcc14-opt/setup.sh
    ```

    All packages under the LCG_107a view will become available for usage.



## GEANT4

Information taken from `/cvmfs/geant4.cern.ch/README`

!!! info "Usage"
    Geant 4 setup needs multiple componets, as such a script as the one below should be used  
    on ISSAF one can use:
    ```
    source /software/alice/enable_cvmfs_g4
    ```

    below the content of the script for reference:
    ```
    #!/usr/bin/env bash

    [[ "${BASH_SOURCE[0]}" -ef "$0" ]] && { echo "This script should be sourced not executed"; exit 1; }

    ################################     UTILITY FUNCTIONS
    ITEM_IN_PATH () {
    local LIST_STR ITEM path_arr n
    [[ -z "${1}" ]] && return 1;
    [[ -z "${2}" ]] && return 1;
    LIST_STR="${1}"
    ITEM="${2}"
    IFS=':' read -ra path_arr <<< "${LIST_STR}"
    for (( n=0; n < ${#path_arr[*]}; n++)); do [[ "$(realpath ${ITEM})" ]] || return 1; [[ "$(realpath ${ITEM})" == "$(realpath ${path_arr[n]})" ]] && return 0; done
    return 1;
    }

    __PATH_INS () { [[ -z "${1}" ]] && return 1; ITEM_IN_PATH "${PATH}" ${1} && return 0; export PATH="${1}${PATH:+:}${PATH}"; }
    __LDLIB_INS () { [[ -z "${1}" ]] && return 1; ITEM_IN_PATH "${LD_LIBRARY_PATH}" ${1} && return 0; export LD_LIBRARY_PATH="${1}${LD_LIBRARY_PATH:+:}${LD_LIBRARY_PATH}"; }
    ####################################################################

    export G4REPOSITORY="/cvmfs/geant4.cern.ch"
    export SFTREPOSITORY="/cvmfs/sft.cern.ch"

    ## SETUP GCC
    GCC_VER="15"
    GCC_ARCH="x86_64-el9"
    GCC_SETUP="${SFTREPOSITORY}/lcg/contrib/gcc/${GCC_VER}/${GCC_ARCH}/setup.sh"
    [[ ! -f "${GCC_SETUP}" ]] && { echo "GCC setup script not found! -> ${GCC_SETUP}"; exit 1; }
    source "${GCC_SETUP}"

    ## SETUP CLHEP
    CLHEP_VER="2.4.7.1"
    CLHEP_ARCH="${GCC_ARCH}-gcc${GCC_VER}-opt"
    export CLHEP_ROOT="${G4REPOSITORY}/externals/clhep/${CLHEP_VER}/${CLHEP_ARCH}"
    [[ ! -d "${CLHEP_ROOT}" ]] && { echo "CLHEP dir not found --> ${CLHEP_ROOT}"; exit 1; }
    __PATH_INS "${CLHEP_ROOT}/bin"
    __LDLIB_INS "${CLHEP_ROOT}/lib"

    export C_INCLUDE_PATH=${CLHEP_ROOT}/include${C_INCLUDE_PATH:+:${C_INCLUDE_PATH}}
    export CPLUS_INCLUDE_PATH=${CLHEP_ROOT}/include${CPLUS_INCLUDE_PATH:+:${CPLUS_INCLUDE_PATH}}

    ## SETUP XERCES
    XERCES_VER="3.3.0"
    XERCES_ARCH="${GCC_ARCH}-gcc${GCC_VER}-opt"
    export XercesC_ROOT="${G4REPOSITORY}/externals/XercesC/${XERCES_VER}/${XERCES_ARCH}"
    [[ ! -d "${XercesC_ROOT}" ]] && { echo "XERCES dir not found --> ${XercesC_ROOT}"; exit 1; }
    __PATH_INS "${XercesC_ROOT}/bin"
    __LDLIB_INS "${XercesC_ROOT}/lib64"

    export C_INCLUDE_PATH=${XercesC_ROOT}/include${C_INCLUDE_PATH:+:${C_INCLUDE_PATH}}
    export CPLUS_INCLUDE_PATH=${XercesC_ROOT}/include${CPLUS_INCLUDE_PATH:+:${CPLUS_INCLUDE_PATH}}

    ## SETUP GEANT4
    G4TAG="11.4.ref02"
    # add -MT for multithreading option
    G4ARCH="x86_64-el9-gcc15-optdeb"
    G4SETUP="${G4REPOSITORY}/geant4/${G4TAG}/${G4ARCH}/CMake-setup.sh"
    [[ ! -f "${G4SETUP}" ]] && { echo "Geant4 setup script not found! --> ${G4SETUP}"; exit 1; }
    source "${G4SETUP}"
    ```


## ROOT

!!! info "Usage"

    ```
    source /cvmfs/sft.cern.ch/lcg/app/releases/ROOT/6.34.06/x86_64-almalinux9.5-gcc115-opt/bin/thisroot.sh

    root-config --version
    6.34.06
    ```


## ALICE

!!! info "Usage"

    Find list of packages [here](https://alimonitor.cern.ch/packages/?packagename=O2Physics%3A%3Adaily&packagedeps=&cvmfsstatus=&platforms=&submitBt=%BB)

    Load package environment with:
    ```
    /cvmfs/alice.cern.ch/bin/alienv enter O2Physics::daily-20250401-0000-1
    ```

    Or to load the environment in current shell (or within the analysis scripts)
    ```
    export O2PHYS_TAG=daily-20250401-0000-1 WORK_DIR="/cvmfs/alice.cern.ch" ALIBUILD_ARCH_PREFIX="el9-x86_64/Packages" DISABLE_GPU=1
    NOT_FOUND=""

    O2PHYS_DIR="${WORK_DIR}/${ALIBUILD_ARCH_PREFIX}/O2Physics/${O2PHYS_TAG}"
    [[ ! -d "${O2PHYS_DIR}" ]] && { ALIBUILD_ARCH_PREFIX="el7-x86_64/Packages"; NOT_FOUND="1"; }

    if [[ -n "${NOT_FOUND}" ]]; then
        O2PHYS_DIR="${WORK_DIR}/${ALIBUILD_ARCH_PREFIX}/O2Physics/${O2PHYS_TAG}"
        [[ ! -d "${O2PHYS_DIR}" ]] && { echo "O2Physics was not found in ${O2PHYS_DIR}"; exit 1; }
    fi

    O2PHYSICS_INIT="${O2PHYS_DIR}/etc/profile.d/init.sh"
    [[ ! -f "${O2PHYSICS_INIT}" ]] && { echo "${O2PHYSICS_INIT} not available"; return 1; }

    source "${O2PHYSICS_INIT}" &> /dev/null
    ```

