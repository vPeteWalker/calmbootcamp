Suggestions from [How to contribute to Nutanix Workshops](https://nutanix.handsonworkshops.com/workshops/32805e93-e67f-46b4-9700-a7eb78db4c21/view/)
- git clone https://github.com/nutanixworkshops/workshop-repo-template
  - recommend using this [.gitignore](https://s3.amazonaws.com/handsonworkshops.prod.media/ws/32805e93e67f46b49700a7eb78db4c21/d/file/dd1e954b38474309acca60229d80acb6/.gitignore)
- https://docutils.sourceforge.io/docs/user/rst/quickref.html

## Atom Plugins

- git-plus
- git-log
- tree-view-git-branch
- tree-view-git-status
- language-sphinx
- RESEARCH:
  - Sphinx-Preview
  - https://hub.docker.com/r/dldl/sphinx-server/tags
  - https://github.com/dldl/sphinx-server

## OS Packages

- http://www.sphinx-doc.org/en/master/usage/installation.html
  - `brew` for the win with pipenv
    - See [dotfiles/dmz/python.md#Linux]
  - `apt-get install python3-sphinx`
    - `sphinx-build --help` = Sphinx v1.6.7

        Suggested packages:
          docutils-doc fonts-linuxlibertine | ttf-linux-libertine texlive-lang-french texlive-latex-base
          texlive-latex-recommended python-jinja2-doc ttf-bitstream-vera python3-sphinx-rtd-theme dvipng
          texlive-latex-extra texlive-fonts-recommended texlive-generic-extra latexmk sphinx-doc

        update-alternatives: using /usr/share/docutils/scripts/python3/rst-buildhtml to provide /usr/bin/rst-buildhtml (rst-buildhtml) in auto mode

## Execute

- Setup:

      # git forked upstream https://github.com/jncox/calmbootcamp
      ghq clone mlavi/calmbootcamp
      ghq-look calmbootcamp
      pipenv install sphinx sphinxcontrib-fulltoc sphinx-bootstrap-theme sphinx_fontawesome
      git checkout -B hybridcloudeng

- Author:
    ```
    ghq-look calmbootcamp
    git checkout hybridcloudeng
    pipenv shell

    sphinx-build . _build
    URL=_build/index.html ; firefox $URL || brave-browser $URL ; echo $URL

    sphinx-build . _build && firefox $_/index.html || brave-browser $_/index.html

    ```
- [_build/index.html](./_build/index.html)

- cd calm_marketplace/images; for i in `ls 510*png`; do `git mv $i 5.10/${i:3}` ; done
