# reserving-gym
Feature benchmarking for chainladder package. This project attempts to have Claude solve Exam 5 and Exam 7 problems using Python. Preference is given to the chainladder-python package, but if a problem can't be solved with it, Claude will attempt to solve it using other packages or base Python.

Claude uses publicly-available papers listed on present and past exam syllabi as the information source. These are located in the `papers` folder and can also be downloaded from the CAS website.

The problems and solutions are found in the `problems` directory in the form of Jupyter notebooks. At the end of each solution a diagnostic is given which identifies any gaps that the chainladder-package has in trying to solve it. A feature suggestion is given along with a sketch implementation.

This is used to inform chainladder-python maintainers on the problem-solving capabilities of the package and what new features need to be added.