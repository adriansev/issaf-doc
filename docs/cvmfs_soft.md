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

    ```
    source /cvmfs/sft.cern.ch/lcg/contrib/gcc/14/x86_64-el9/setup.sh
    source /cvmfs/geant4.cern.ch/geant4/11.3.ref03/x86_64-el9-gcc14-optdeb-MT/CMake-setup.sh

    geant4-config --version
    11.3.0
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

