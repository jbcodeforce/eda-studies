# [Sharing EDA stuffs](https://jbcodeforce.github.io/eda-studies)

The goal of this site is to keep references and notes on studies and best practices I have used to implement EDA customer's solutions and to address some common problems I encountered while trying new technologies. 

All the content of this repository is visible as blog pages and website at [eda-studies](https://jbcodeforce.github.io/eda-studies). 



## Building this booklet locally

The content of this repository is written with markdown files, packaged with [MkDocs](https://www.mkdocs.org/) and can be built into a book-readable format by MkDocs build processes.

1. Install MkDocs locally following the [official documentation instructions](https://www.mkdocs.org/#installation).
1. Install Material plugin for mkdocs:  `pip install mkdocs-material` 
2. `git clone https://github.com/jbcodeforce/eda-studies.git` _(or your forked repository if you plan to edit)_
3. `cd eda-studies`
4. `mkdocs serve`
5. Go to `http://127.0.0.1:8000/` in your browser.

### Building this booklet locally but with docker

In some cases you might not want to alter your Python setup and rather go with a docker image instead. This requires docker is running locally on your computer though.

* docker run --rm -it -p 8000:8000 -v ${PWD}:/docs squidfunk/mkdocs-material
* Go to http://127.0.0.1:8000/ in your browser.

### Pushing the book to GitHub Pages

1. Ensure that all your local changes to the `master` branch have been committed and pushed to the remote repository. `git push origin code`
1. Run `mkdocs gh-deploy --remote-branch main` from the root directory.



