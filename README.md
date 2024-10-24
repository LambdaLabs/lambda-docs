# lambda-docs-mkdocs

MkDocs proof of concept. Running [Material for MkDocs theme](https://squidfunk.github.io/mkdocs-material/)

## Setting up a local build environment for MKdocs on MacOS

### Prerequisites

#### Git

Open a terminal and run `git --version`

If git isn't installed already, you will be prompted to install it.

#### Python

Download the latest [python release](https://www.python.org/downloads/macos/)

Open a terminal at the Python application folder.

Run the command `curl https://bootstrap.pypa.io/get-pip.py | python3`

This ensures that you can use pip to install the required MKdocs plugins and serve a live preview.

### Creating a Local Live Preview

Make a local copy of the GitHub repository

1. Create a file path for the repository you'd like to clone, for example: `/Users/yourname/Documents/GitHub/`

2. Clone the repository to your local disk. You should now have a folder with the repository that looks something like this:

    `/Users/yourname/Documents/GitHub/lambda-docs-mkdocs`

3. Now open a new terminal window at the repository main folder. Right click the folder and select 'New Terminal at Folder' from the thumbnail menu.

![Terminal at Folder](https://cdn.mathpix.com/cropped/2024_06_05_2ef73961a67c69be1b52g-2.jpg?height=740&width=1781&top_left_y=234&top_left_x=234)

### Configuring MKdocs

1. Setup MKdocs on your local environment by running the following commands in your new terminal window:

       pip install mkdocs-material=="9.*"
       pip install -r requirements.txt
       pip install mkdocs-techdocs-redirects
       pip install mkdocs-git-revision-date-localized-plugin
       pip install "mkdocs-material[imaging]"

   This installs the MKdocs [material theme](https://squidfunk.github.io/mkdocs-material/), the [redirect plugin](https://pypi.org/project/mkdocs-techdocs-redirects/), the [date and last git modification plugin](https://github.com/timvink/mkdocs-git-revision-date-localized-plugin), and the [image processing plugin](https://github.com/squidfunk/mkdocs-material/blob/master/docs/plugins/requirements/image-processing.md)

   **Note:** If you have homebrew configured on your mac run:

       export DYLD_FALLBACK_LIBRARY_PATH=/opt/homebrew/lib

   This environment variable [fixes potential errors](https://github.com/squidfunk/mkdocs-material/issues/5121) with finding libraries used by the theme's built-in plugins.

2. Now you can run `mkdocs serve` to build the site. You should expect to see the following output:

       INFO    -  Building documentation...
       INFO    -  Cleaning site directory
       INFO    -  The following pages exist in the docs directory, but are not included in the "nav" configuration:
                   - tags.md
                   - includes/hello-world.incl.md
       INFO    -  Documentation built in 2.52 seconds
       INFO    -  [10:19:27] Watching paths for changes: 'docs', 'mkdocs.yml', 'docs/includes/hello-world.incl.md'
       INFO    -  [10:19:27] Serving on http://127.0.0.1:8000/lambda-docs-mkdocs/

3. Once the site has been built, open a browser window and paste `http://127.0.0.1:8000/` into the search bar and press enter.

   Congratulations, your local preview is now live!

### Configure MKdocs in Docker

1. Build the container:

   ```bash
   docker build -t lambda-docs-mkdocs .
   ```

2. Run the container:

   ```bash
   docker run --rm -it -p 8000:8000 -v ${PWD}:/docs lambda-docs-mkdocs serve
   ```

3. Once the container is running, open a browser window and paste `http://127.0.0.1:8000/lambda-docs-mkdocs` into the search bar and press enter.

Congratulation, your local preview is now live inside of a container!
