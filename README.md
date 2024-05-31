This site is built by the Python package `mkdocs`.

To publish the site from the master branch, simply
run `mkdocs gh-deploy`. This will build the site and
copy the files to the `gh-pages` branch online. The
new site should be available for access thereafter. 

Note, to use mkdocs with the Anaconda installed version of Python
and the corresponding mkdocs module (for eg python3), use:
`python -m mkdocs gh-deploy`

Note that whenever the docs folder is updated here, it should be 
manually synced with the docs folder in the Yelmo 
repository.


### Installing Python libraries

```
# Install mkdocs library
pip3 install mkdocs

# Install math rendering library
pip3 install https://github.com/mitya57/python-markdown-math/archive/master.zip
```
