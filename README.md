# HNAP Manual

This manual is [mkdocs](https://www.mkdocs.org/) based.
It supports documentation of multiple versions using the
[mike](https://pypi.org/project/mike/) utility.

## Install mkdocs, mkdocs-material and pdf-export-plugin

```bash
python -m pip install -r requirements.txt
```

You might want to work in a Python virtualenv:

```bash
virtualenv --python=/usr/bin/python3 venv/
source venv/bin/activate
python -m pip install -r requirements.txt
```

**Important: Each file name needs to be the same as the first heading in each file.**

## View documentation locally

`mike serve`

This command starts a local webserver at [localhost:8000](http://localhost:8000).

## Deploy to Github Pages

`mike deploy <version> <alias> <-u> <-p>`

Will build the static files and push to the gh-pages branch on Github, so they
will be served automatically at [manual.hnap.de](https://manual.hnap.de)

`<version>` is the version number of the documentation (e.g. 1.0.0).  
`<alias>` is an alias for this version. Currently, we use the alias `latest`
to set the version which is automatically shown when visiting the website.

If the version number (e.g. 1.0.0) already exists, you have to add `-u` (for
update) to the command.

To push the content to Github, add `-p`.

So a complete command might look like this:

`mike deploy 1.0.0 latest -u -p`

## Generate PDF

Currently, the _mkdocs-pdf-export-plugin_ can't handle anchors, therefore it is
necessary to temporarily alter links containing anchors i.e. from
"(../foo#bar)" to "(../foo.md#bar)" to be able to generate
a PDF file.

```bash
export ENABLE_PDF_EXPORT=1
mkdocs build
```
