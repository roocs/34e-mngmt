# roocs release procedure


I have been following this to release roocs-utils, dachar, daops and clisops but the general workflow can be applied anywhere.

For the above packages, the work setting up for releases has already been done so you can skip straight to [packaging and releasing](#Packaging-and-releasing)

For other packages go through the steps below to make sure everything is set up correctly.


# Steps to follow before release:


## Are all tests running?

Check the tests are all running locally.

NOTE: some tests use "real" data and are only flagged to run on local installation. So we need to make sure they all run locally.

The roocs repositories use github actions and the readthedocs webhook.


## Are classes/functions documented with adequate doc strings?

At least the external API classes and functions should be well described with proper doc strings.


## Have we updated the Notebooks?

Make sure the Notebooks are working.

The following repos should have a "notebooks" directory at the top-level:



*   rooki
*   dachar
*   daops
*   clisops
*   roocs-utils

Question: 



*   Do we want to render notebooks _with_ outputs? 
*   this requires full installation in readthedocs

To render the notebooks in the docs, update conf.py to:


```
extensions = [
    "sphinx.ext.autodoc",
    "sphinx.ext.viewcode",
    "sphinx.ext.mathjax",
    "sphinx.ext.napoleon",
    "sphinx.ext.coverage",
    "sphinx.ext.todo",
    "sphinx.ext.autosectionlabel",
    "nbsphinx",
    "IPython.sphinxext.ipython_console_highlighting",
]
```


 


```
nbsphinx_execute = "auto"
nbsphinx_timeout = 300
```


 

A binder directory in the top level is required for any extra requirements and to install the package - see roocs-utils for an example: https://github.com/roocs/roocs-utils/tree/master/binder

 


## Have we updated the documentation?

The roocs-utils repository is a good example of what the docs should look like: [https://github.com/roocs/roocs-utils/tree/master/docs](https://github.com/roocs/roocs-utils/tree/master/docs)

Update the documentation under "docs/".

We should have as a minimum: Installation, History, Authors, Contributing, Usage, Notebooks (Examples), Readme (Quick Guide), Modules (api)

Add an api/modules.rst file if missing. Point to this in index.rst. sphinx-autodoc will use this to generate documentation from doc strings. Modules should be added manually e.g.


```
.. automodule:: roocs_utils.utils.time_utils
   :noindex:
   :members:
   :undoc-members:
   :show-inheritance:
```


so ensure any new modules are added.

Add a readthedocs.yml file if missing E.g:


```
# .readthedocs.yml
# Read the Docs configuration file
# See https://docs.readthedocs.io/en/stable/config-file/v2.html for details

# Required
version: 2

# Build documentation in the docs/ directory with Sphinx
sphinx:
 configuration: docs/conf.py

# Build documentation with MkDocs
#mkdocs:
#  configuration: mkdocs.yml

# Optionally build your docs in additional formats such as PDF and ePub
formats: all

# Optionally set the version of Python and requirements required to build your docs
#python:
#  version: 3.6
python:
 version: 3.6
 install:
   - method: pip
     path: .
     extra_requirements:
       - docs

# conda:
#   environment: docs/environment.yml

build:
 image: stable
```


docs requirements should be set in setup.py [https://github.com/roocs/roocs-utils/blob/master/setup.py](https://github.com/roocs/roocs-utils/blob/master/setup.py)

Are notebooks linked in via symlink back to top-level "notebooks" directory? If not use the command: `ln -s ../notebooks docs/` from top level directory

Any markdown files should be changed to rst. 

`pip install m2r` ([https://github.com/miyakogi/m2r](https://github.com/miyakogi/m2r) ) and use that to convert them. 

The command is `m2r your_document.md [your_document2.md ...]`

Delete markdown files. `setup.py `may need to be updated to read from rst instead of md. (Change reading of README and set `long_description_content_type="text/x-rst" and update`


```
import os
here = os.path.abspath(os.path.dirname(__file__))
_long_description= open(os.path.join(here, "README.rst")).read()
```


 

Change MANIFEST.in to rst instead of md files 

To include etc/roocs.ini - update `MANIFEST.in `and add `recursive-include etc * `

To check the docs will build properly run "make html" locally in "./docs/" 

If it fails on `readthedocs`, is it because the `sys.path` change happens _after_ the import of our package? This causes an import error that needs to be ignored - add the error code (E402) from flake8 to the flake8 ignore section in setup.cfg.

You may need to update `html_static_path `in` conf.py`

Update the theme of the documentation following these instructions:[ https://sphinx-rtd-theme.readthedocs.io/en/stable/](https://sphinx-rtd-theme.readthedocs.io/en/stable/)

Add a logo: set `html_logo `in conf.py to the file path to the image. Put it after  `html_static_path. `

Turn the readthedocs webhook on so that the docs build on a pull request. ([https://docs.readthedocs.io/en/latest/pull-requests.html](https://docs.readthedocs.io/en/latest/pull-requests.html))


## Install and configure pre_commit


```
conda install -c conda-forge pre_commit
pre-commit install
pre-commit run --all-files
```


 

You can also pip install: `pip install pre-commit`

 

NOTE: you might need to configure this in:  .pre-commit-config.yaml


```
-   repo: https://github.com/ambv/black
    rev: 20.8b1
    hooks:
    -   id: black
        language_version: python3
        args: ["--line-length", "88", "--target-version", "py37"]
-   repo: https://github.com/pycqa/flake8
    rev: 3.8.3
    hooks:
    -   id: flake8
        language_version: python3
        args: ['--max-line-length', '88', '--config=setup.cfg']
```


 

Make sure the versions are up to date with those that will be used in travis 

Line lengths can also be configured in setup.cfg.

in .pre-commit-config.yaml you may need to exclude setup.cfg from trailing-whitespace and end-of-file-fixer as bumpversion changes these and the bumpversion commit then fails. This is a known problem with open pr on their repo)

e.g.


```
-   id: end-of-file-fixer
    exclude: setup.cfg
```



## Pull request template

Would it be useful to have a pull request template? - to include points such as:



*   has the HISTORY.rst file been updated

e.g. [https://github.com/roocs/clisops/blob/master/.github/PULL_REQUEST_TEMPLATE.md](https://github.com/roocs/clisops/blob/master/.github/PULL_REQUEST_TEMPLATE.md)


## Identify "breaking changes" and update HISTORY.rst

Breaking changes _may_ occur from:

- changes to the public API

- changes to the dependencies (packages and/or versions)

- changes to any build recipes/instructions

- will it affect any of the other packages

GENERAL RULE: 

- If we are not sure if a change is breaking, we should document it as a breaking change to help users/co-developers stay informed.

- we should be guided by whether we have had to make changes to our dev environments

TIP:

It is good practice to keep an unreleased section at the top of this while developing:[ https://keepachangelog.com/en/0.3.0/](https://keepachangelog.com/en/0.3.0/) 

The history should match the logged changes that will be put into release notes. Format of HISTORY.rst should be:


```
Version History
===============

v0.1.2 (2020-10-15)
-------------------

General description

Upgrade Steps
^^^^^^^^^^^^^
* None

Breaking Changes
^^^^^^^^^^^^^^^^
* None

New Features
^^^^^^^^^^^^
* None

Bug Fixes
^^^^^^^^^
* None

Other Changes
^^^^^^^^^^^^^
* Updated doc strings to improve documentation.
* Updated documentation.
```


 

If any sections are None, remove them

Each release should be included at the top of the file.

Review the commits since the last release to identify changes that should be mentioned.


## Configure bumpversion in: setup.cfg

Anywhere the version is mentioned and would need to be updated should be added to setup.cfg for bumpversion to find.

 


```
[bumpversion]
current_version = 0.4.2
commit = True
tag = True

[bumpversion:file:setup.py]
search = __version__ = "{current_version}"
replace = __version__ = "{new_version}"

[bumpversion:file:roocs_utils/__init__.py]
search = __version__ = "{current_version}"
replace = __version__ = "{new_version}"

[bumpversion:file:docs/conf.py]
search = version = "{current_version}"
replace = version = "{new_version}"
```



# Packaging and releasing

If all changes have been merged into master, documentation has been updated and all tests are passing then we can release a new version.




1. Make sure the HISTORY.rst file includes all changes being made in this new release. (It should do, as we have been updating as we make a change). Update the release date to today’s date. Merge this into master.
2. On the master branch, run bumpversion to increase the package version: 

    Do a dry run

    You can run for example:

    ```
    bumpversion --dry-run --verbose patch (or major/minor)
    ```

    to check you are doing what you expect       

    Run bumpversion

    ```
    bumpversion patch (or major/minor)
    git push
    git push --tags
    ```     

    Note: you have to do both push commands


3. Publish on PyPi:

    You can test on test pypi first as detailed here:[ https://packaging.python.org/tutorials/packaging-projects/#uploading-the-distribution-archives](https://packaging.python.org/tutorials/packaging-projects/#uploading-the-distribution-archives) 


    If first publish to PyPI follow these instructions:[ https://medium.com/@joel.barmettler/how-to-upload-your-python-package-to-pypi-65edc5fe9c56](https://medium.com/@joel.barmettler/how-to-upload-your-python-package-to-pypi-65edc5fe9c56)


    To update the version on PyPI run:


    ```
    python setup.py sdist bdist_wheel
    ```



     


    ```
    twine upload dist/*
    ```



    you will be asked for your username and password here. 


     


    Check that it works with e.g.


     


    ```
    pip install roocs-utils --upgrade
    ```


4. Create a release on GitHub

    Create a release from the tag on github.


    We should use copy of the release notes in HISTORY.rst

5. Update the package on conda. Once released on PyPI a PR will automatically be generated on the conda-forge feedstock for the package. This may take several hours. Check that all the dependencies are correct and the tests pass before merging. If the feedstock doesn’t exist yet, follow these instructions to create it: [https://github.com/conda-forge/staged-recipes/blob/master/README.md](https://github.com/conda-forge/staged-recipes/blob/master/README.md) 


# Appendix


## Notes on readthedocs:



*   Stable version is kept up to date with tagged releases (unless you want a custom stable - then call branch/tag stable)
*   Latest version points to master branch
*   Read the Docs supports two workflows for versioning: based on tags or branches. If you have at least one tag, tags will take preference over branches when selecting the stable version.
*   This is useful info about versioning:[ https://docs.readthedocs.io/en/stable/versions.html](https://docs.readthedocs.io/en/stable/versions.html)  

 
