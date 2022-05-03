# Brick Docs

These are developer-focused docs for using Brick to model buildings.

## Getting Started

1. [Install JupyterBook](https://jupyterbook.org/intro.html#install-jupyter-book)
2. Install deps:
    ```
    pip install --user -r requirements.txt
    jupyter sparqlkernel install
    # depending on how you setup your python env, you may need to run this instead
    # jupyter sparqlkernel install --user
    ```
3. Build docs: `jupyter book build .`
4. Visit docs: `cd _build/html && python3 -m http.server`

One other note: if you are using a virtual environment, do not create the virtual environment in the same directory as you have checked out this repository.
JupyterBook may try to interpret your virutalenv directory as part of the book, if you see messages about "data.metadata.kernelspec must contain ['name', 'display_name'] properties" this is likely what's happening.
