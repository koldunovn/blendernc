==========
Contribute
==========

If you want to contribute to the development of BlenderNC, please submit a pull request to the `master branch or dev branch <https://github.com/josuemtzmo/blendernc/pulls>`_.

Development workflow
####################

|vscode badge|

.. |vscode badge| image:: https://open.vscode.dev/badges/open-in-vscode.svg
   :target: https://open.vscode.dev/blendernc/blendernc


- Download and install `Visual Studio Code <https://code.visualstudio.com/>`_.
- Install Visual Studio Code Add-On ``jacqueslucke.blender-development`` by searching at the `Extensions in Marketplace`.
- Set up your ``Blender`` executable by pressing ``ctrl+shift+P`` -> ``Blender Start`` -> ``Choose new blender executable``.
- Clone BlenderNC source code (`BlenderNC repo <https://github.com/blendernc/blendernc>`_):

    .. code-block:: bash

        git clone https://github.com/blendernc/blendernc.git

- Set up your remotes (i.e. ``origin`` and ``upstream``).
- Initiate a new workspace in the cloned folder.
- Voalà! Now you can develop, debug, and modify **BlenderNC**.

Testing
#######

In order to merge into any branch of the **BlenderNC** repository, all test must pass. Locally testing **BlenderNC** is really simple, as it uses docker containers by `nytimes/rd-blender-docker <https://github.com/nytimes/rd-blender-docker>`_. Execute the following code from the cloned repository root directory:

.. code-block:: bash

    docker pull nytimes/blender:latest
    docker run -w /addon/blendernc -it --rm --mount type=bind,source="$(pwd)",target=/addon/blendernc -t nytimes/blender:latest /bin/sh -c

If you want more control on each test, you can run an interactive docker container:

.. code-block:: bash

    docker pull nytimes/blender:latest
    docker run -w /addon/blendernc --rm -it --mount type=bind,source="$(pwd)",target=/addon/blendernc -t nytimes/blender:latest /bin/bash

and then run the following code in sections:

.. code-block:: bash

    #!/bin/sh

    $BLENDERPY -m ensurepip --default-pip

    $BLENDERPY -m pip install -r requirements.txt --progress-bar off

    $BLENDERPY -m pip install coverage --progress-bar off

    $BLENDERPY -m pip install -e . --progress-bar off

    COVERAGE_PROCESS_START=${PWD}"/.coveragerc"
    export COVERAGE_PROCESS_START=$COVERAGE_PROCESS_START
    export PYTHONPATH=$PYTHONPATH:${PWD}

    cd tests

    echo -e "import coverage \n\ncov=coverage.process_startup()\n"> sitecustomize.py
    echo -e "print('Initiate coverage')" >> sitecustomize.py
    echo -e "print(cov)" >> sitecustomize.py

    export PYTHONPATH=$PYTHONPATH:${PWD}

    $BLENDERPY run_tests.py

    coverage combine
    coverage report
    coverage xml

.. important::
    Note the flag ``--mount type=bind,source="$(pwd)"`` when using the ``docker run`` command. The source path will be the current working directory. Make sure it corresponds to the root of the cloned repository (i.e. ``/parent/path/to/blendernc/blendernc``, where ``/parent/path/to/blendernc`` is the parent directory where ``github clone`` created the repository.

Linting
#######

BlenderNC is coded using the following linters for formatting conformance:

- `Black <https://github.com/psf/black>`_ ,
- `Flake8 <https://flake8.pycqa.org/en/latest/>`_,
- `Isort <https://isort.readthedocs.io/en/latest/>`_,
- `commitlint <https://commitlint.js.org/#/>`_.


These can be applied before committing code on the developer machine using `pre-commit <https://pre-commit.com/>`_. Follow these steps to set up your development environment.

.. code-block:: bash

    pip install pre-commit
    pre-commit install

Make sure to install `commitlint <https://commitlint.js.org/#/>`_ is installed by doing:

.. code-block:: bash

    npm install --save-dev @commitlint/{cli,config-conventional}
    pre-commit install --hook-type commit-msg


otherwise, the ``pre-commit`` will fail with the following error:

.. code-block:: bash

    Error: Cannot find module "@commitlint/config-conventional" from "/blendernc"

Before commiting or creating a PR, it is recommended to execute:

.. code-block:: bash

    pre-commit

or

.. code-block:: bash

    pre-commit run --all-files

to skip skipping of files.

.. important::
    Commits and PR's pushed into branches ``master``, ``dev`` or ``distribution`` trigger `github actions <https://github.com/features/actions>`_ and perform necessary code-quality checks.


Distribution
############

Distribution of the BlenderNC is automatic updated through `github actions <https://github.com/features/actions>`_ when commits and PR's are pushed into the branches ``master``, ``dev`` or ``distribution``.

BlenderNC releases to PyPi are automated by using `poetry <https://python-poetry.org>`_, `semantic-release <https://semantic-release.gitbook.io/semantic-release/>`_, and `python-semantic-release <https://python-semantic-release.readthedocs.io/en/latest/>`_, through github actions that determine the next version number, generate the release notes and publishe the package to `PyPi <https://pypi.org/project/blendernc/>`_. For more information on the CI integration of semantic-release see `./github/workflows/release.yml <https://github.com/blendernc/blendernc/blob/distribution/.github/workflows/release.yml>`_

Aditionally, a compressed BlenderNC release is publised to `blendernc-zip-install <https://github.com/blendernc/blendernc-zip-install>`_
when commits and PR's are pushed into the branches ``master``, ``dev`` or ``distribution``. This compressed release can be used to manually install BlenderNC in Blender. For more information on the CI integration of zip releases see `./github/workflows/ci.yml <https://github.com/blendernc/blendernc/blob/distribution/.github/workflows/ci.yml>`_
