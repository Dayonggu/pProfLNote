# Basic on Python

## packaging
### Installing packages
~~~
pip install -U pip setuptools
~~~

### Packaging your project

* files needed
  * setup.py
  * setup.cfg
  * MANIFEST.in
  * LICENSE.txt
* install in developer mode
  * `sudo -H pop install -e .`  // in your local root

#### commands
~~~
// install twine
sudo -H pip install twine

// install wheels
sudo -H pip install wheel

// make a source Distribution
python setup.py sdist

//install
sudo -H pip install ./pyDataAnalyzer/
check installation at:
/Library/Python/2.7/site-packages/pydataanalyzer/

sudo -H pip install -e ./pyDataAnalyzer/  // in dev mode, can be modified
Will only create an egg-link in the above site-packages dir
point to your package location
e.g.
/Users/dayonggu/Documents/Work/Perf/pyDataAnalyzer




//uninstall
sudo -H pip uninstall pyDataAnalyzer
~~~
