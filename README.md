# Convenient LaTeX Compiler
This repository represents a convenient LaTeX Compiler, that can be used to compile LaTeX `.tex` files with only one command by specifying the desired engines that should be used -- LaTeX and BibTeX. The so called *auxiliary files* that  are generated by the LaTeX and BibTeX engines but not used by the user are automatically stashed into a specified folder in the same directory as the file that will be compiled. 


## Table Of Contents

[Introduction](#introduction)
* [Why not use bash script instead?](#why-not-use-bash-script-instead)

[Installation](#installation)

[Application](#application)
  * [Conventional use from Terminal](#conventional-use-from-terminal)
  * [Command Line Arguments](#command-line-arguments)
  * [Folder Structure after compilation](#folder-structure-after-compilation)
  * [Advanced use in Python program](#advanced-use-in-python-program)
  * [Consideration of Shebangs](#consideration-of-shebangs)

[License](#license)

## Introduction
The simple fact that LaTeX previewers like [TeXShop](https://pages.uoregon.edu/koch/texshop/) *-- for MacOS Systems at least --* do not provide the function to store the auxiliary files into an extra folder was the originator of this work. The auxiliary files are simply stashed in the working directory. [TeXstudio](https://www.texstudio.org) eg. provides the --`-aux-directory <directory>` command for Windows users to specify the folder in which the auxiliary files will be stored. With this module, it is possible *-- for each system, ie. Linux, MacOS or Windows --* to compile LaTeX files in such a way, that the auxiliary files will be stored in an extra folder, specified by the user. The files are moved into the desired folder, and not deleted, since these files contain important informations about the Bibliography, Table Of Contents (TOC), Table Of Figures (TOF), Acronyms, etc. which are of interest in the output file (.pdf) at the end. Without them, the Bibliography etc. will not be properly displayed. If these files are present, one LaTeX command less needs to be executed than in the scenario where they were not. All in all, there are two Pipelines based on the existence of the auxiliary files:
- Auxiliary files do not exist, ie. they need to be generated first, so that TOC, TOF etc. will be properly displayed in the output file :
```bash
LaTeX engine --> LaTeX engine [--> BibTeX engine --> LaTeX engine]
```

- Auxiliary files do exist, so the changes can be made visible by only executing the LaTeX command once:
```bash
LaTeX engine [--> BibTeX engine --> LaTeX engine]
```

Note that after executing the BibTeX engine, the updates need to be included in the output document by executing the LaTeX engine once again.

### Why not use bash script instead?
One might think that implementing a simple Bash script and executing it by just specifying the path to the script in the corresponding [TeXShop](https://pages.uoregon.edu/koch/texshop/) engines would be sufficient. However, the execution of the file is based to the prior knowledge of which engine to use in the first place. Technically, the easiest way would be to create for each LaTeX and BibTeX engine, a corresponding script, since the user wants to stash the auxiliary files into an extra folder, no matter which engine is used. Then, the user needs to specify the path to the right script that should be executed in the engines settings each time another engine needs to be used. This can be very error prone *-- by specifying the wrong script --* or/and difficult for users with no knowledge in using such scripts or specifying the engines in the previewer. Now, there is for sure a smarter way, like implementing a script that automatically executes the right engine, by scanning the head of the .tex file for a shebang (Magic Line), however such a functionality implemented in Python can also be used and embedded into other projects, where LaTeX files are generated automatically and has thus a broader application than such a script in itself. Further, when using a script, the compilation pipeline needs to be executed by the user as well. Again, such a Pipeline can be easily implemented in the script, but it will then be executed using LaTeX engine settings while also creating a Bibliography using BibTeX or Biber, which has its own engine settings in the previewer. This has no effect on the compilation at all, but it is just misleading for inexperienced user and also not the best way to solve the problem at hand since the BibTeX task is not performed using the provided previewer setting.

As mentioned earlier, this Python module can be used for compiling generated LaTeX files in an autonomous way without compiling the generated file using LaTeX and BibTeX engines by hand.

## Installation
It is expected that the desired LaTeX and BibTeX engines are properly installed and working (before using this module).
To install the module *-- if desired in an Anaconda environment --* simply use PyPi:

`pip install LatexCompiler`

or install directly from the repository:

`pip install git+https://github.com/amrane99/LatexCompiler`.


## Application
The LatexCompiler can be used for just compiling a .tex file or it can be embedded in a system that generates .tex files to compile it after the creation process. 

### Conventional use from Terminal
To compile a .tex file it is very important to understand, that the auxiliary files are generated where the *python command* is executed from, for instance the location from where the *LaTeX and BibTeX engines commands* are launched. This means, LaTeX searches for the files like pictures or other .tex files that are included/referenced in a .tex file in the directory where the LaTeX command is launched, not the directory where the .tex file itself is located. In such cases, LaTeX will throw errors like `file not found`, for reference see [this post](https://tex.stackexchange.com/questions/95617/includegraphics-file-not-found-even-when-in-same-directory). So before using this method, it is important to navigate to the directory where the .tex file lives that needs to be compiled:
```bash
          ~ $ source ~/.bashrc
          ~ $ source activate <your_anaconda_env>
(<your_anaconda_env>) $ cd LaTeX_project_XX
(<your_anaconda_env>) LaTeX_project_XX $ LatexCompiler -file <file_name>.tex
```
Note that the first and second commands are only necessary, if this module is installed in an anaconda environment. If it is installed in the systems environment, these steps can be skipped. The execution of the fourth command will compile the LaTeX file `<file_name>.tex` with the default settings the following way:
```bash
 pdflatex --file-line-error --synctex=1 <full_path_to_file_name>.tex
 pdflatex --file-line-error --synctex=1 <full_path_to_file_name>.tex
 biber <full_path_to_file_name>.tex
 pdflatex --file-line-error --synctex=1 <full_path_to_file_name>.tex
```
Further the auxiliary files will be stashed in the auxiliary `.latex/` directory that is on the same level located as  `<full_path_to_file_name>.tex`.
Note that the algorithm will first check if the auxiliary directory already exists, in which case the auxiliary files can be used in the LaTeX command. That way, `<full_path_to_file_name>.tex` only needs to be compiled once before using BibTeX to include the changes. Further, the full path to `<file_name>.tex` will be extracted in the module, if the command has been executed in the working folder where the file is located. If the command will not be executed from the working directory, it is crucial to provide the full path to the `<file_name>.tex`, so the auxiliary folder is generated at the right location.

### Command Line Arguments
With the following flags and arguments, the used engines and name of auxiliary folder can be specified:

| Tag_name | description | required | choices | default | 
|:-:|-|:-:|:-:|:-:|
| `-file` | Specify *[full] path* to the main .tex file. | yes | -- | -- |
| `-tex_engine` | Specify which LaTeX engine to use. | no | `pdflatex, lualatex, xelatex` | `pdflatex` |
| `-bib_engine` | Specify which BibTeX engine to use. | no | `biber, bibtex` | `biber` |
| `-no_bib_engine` | Use this flag if the BibTeX engine should not be used, ie. .tex file has no Bibliography. | no | -- | `False` |
| `-aux_folder` | Specify the name of the folder in which the auxiliary files will be stashed. | no | -- | `.latex` |
| `-h` or `--help` | Simply shows help on which arguments can and should be used. | -- | -- | -- |

#### Example use cases
1. If the LaTeX `example.tex` file, located at `/LaTeX_project_XX` needs to be executed using LuaLaTeX and BibTeX, by stashing the auxiliary files into `/LaTeX_project_XX/.aux/`, the command would be the following: 
```bash
          ~ $ source ~/.bashrc
          ~ $ source activate <your_anaconda_env>
(<your_anaconda_env>) $ cd LaTeX_project_XX
(<your_anaconda_env>) LaTeX_project_XX $ LatexCompiler -file example.tex
                                                       -tex_engine lualatex -bib_engine bibtex
                                                       -aux_folder .aux
```
2. If the LaTeX `example.tex` file, located at `/LaTeX_project_XX` needs to be executed using XeLaTeX without using a BibTeX engine since it does not have a bibliography, while stashing the auxiliary files into `/LaTeX_project_XX/auxiliary_files/`, the command would be the following: 
```bash
          ~ $ source ~/.bashrc
          ~ $ source activate <your_anaconda_env>
(<your_anaconda_env>) $ cd LaTeX_project_XX
(<your_anaconda_env>) LaTeX_project_XX $ LatexCompiler -file example.tex
                                                       -tex_engine xelatex -no_bib_engine
                                                       -aux_folder auxiliary_files
```
Note that in the second example no BibTeX engine will be used, so the command
```bash
 xelatex /LaTeX_project_XX/example.tex
```

would be executed only once, if the auxiliary files at `/LaTeX_project_XX/auxiliary_files/` exist and twice otherwise. In both examples, the first two steps can be omitted, if this module is installed in the systems environment. Further, if the command will be executed within the folder of the .tex file, only the .tex file itself and not the full path needs to be provided.

### Folder Structure after compilation
After successfully compiling the .tex file, all from the engines generated auxiliary files are stored in the specified folder. First of all, it is important to define which files are all auxiliary files:

---
**Auxiliary Files**

Everyhting that is not a folder, nor a .tex file, nor a .pdf file is considered as an *auxiliary file*.

---
The auxiliary files will be moved to the desired folder.
The structure before compilation looks more or less like the following *-- depending on users style of structuring a LaTeX project --*:

    <directory_to_tex_file>/
        ├── <file_to_compile>.tex
        ├── sections/
        │   ├── section_00.tex
        │   ├── section_01.tex
        │   ├── ...
        ├── images/
        │   ├── image_00.png
        │   ├── image_01.png
        │   ├── ...
        └── docs/
            ├── input_00.pdf
            ├── input_00.pdf
            ├── ...

After normal execution using previewers, all auxiliary files are stored at the same level as `<file_to_compile>.tex`, for instance in `<directory_to_tex_file>/` which makes the working directory pretty messy:

    <directory_to_tex_file>/
        ├── <file_to_compile>.tex
        ├── sections/
        │   ├── section_00.tex
        │   ├── section_01.tex
        │   ├── ...
        ├── images/
        │   ├── image_00.png
        │   ├── image_01.png
        │   ├── ...
        ├── docs/
        │   ├── input_00.pdf
        │   ├── input_00.pdf
        │   ├── ...
        ├── <file_to_compile>.pdf
        ├── <file_to_compile>.aux
        ├── <file_to_compile>.toc
        ├── <file_to_compile>.bcf
        ├── <file_to_compile>.log
        ├── ...

What the user really wants from all generated files is only the `<file_to_compile>.pdf` file, everything else is not of relevance for the conventional user. When compiling the `<file_to_compile>.tex` file using this module and specifying the folder name to be `.aux/`, the working directory structure will look like the following on the compilation is done:

    <directory_to_tex_file>/
        ├── <file_to_compile>.tex
        ├── <file_to_compile>.pdf
        ├── sections/
        │   ├── section_00.tex
        │   ├── section_01.tex
        │   ├── ...
        ├── images/
        │   ├── image_00.png
        │   ├── image_01.png
        │   ├── ...
        ├── docs/
        │   ├── input_00.pdf
        │   ├── input_00.pdf
        │   ├── ...
        └── .aux/
            ├── <file_to_compile>.aux
            ├── <file_to_compile>.toc
            ├── <file_to_compile>.bcf
            ├── <file_to_compile>.log
            ├── ...

### Advanced use in Python program
Let's assume there is a Python algorithm/program that automatically generates a .tex file that embodies data provided from a .csv file (*.csv --> .tex*). The in this module provided function `compile_document(tex_engine, bib_engine, no_bib, path, folder_name)` can then be used in the program after saving the generated (.tex) file to compile the file as well *-- without any human assistance --*:
```python
import os
from <own_module> import some_transformation_to_tex
from LatexCompiler.LatexCompiler import LC

# -- Load .csv -- #
csv = open('<dest_path>', 'r')

# -- Transform into tex format --> provide own function -- #
tex = some_transformation_to_tex(csv)

# -- Save tex file at path -- #
path = '<targ_path>'
tex_name = os.path.join(path, '<file>.tex')
with open(tex_name, "w") as f:
    f.write(tex)

# -- Now compile the file using module -- #
LC.compile_document(tex_engine = 'LuaLaTeX',
                    bib_engine = 'biber', # Value is not necessary
                    no_bib = True, path = tex_name, # Provide the full path to the file!
                    folder_name = '.aux')
```
In this example, a .csv file will be loaded using Python, than transformed into a .tex file using a provided and implemented function *-- some_transformation_to_tex --> Pseudo Function --* and saved at a specified path `<targ_path>/<file>.tex`. *-- In such a scenario it is of crucial importance to provide the full path to the .tex file that needs to be compiled, since the algorithm/program and thus the engines might not be triggered in the same directory as the file is created! --* Then the LatexCompiler module function will be used to compile the just saved .tex file using LuaLateX and **no** BibTeX engine. The auxiliary files will be stored under `<targ_path>/.aux` . It is important to say, that `no_bib = True` indicates that no BibTeX engine needs to be used, ie. `bib_engine` will not be considered in such a scenario.
Further, any error occurring during the compilation using this module is solely caused by the LaTeX files and not from the Python code itself, since the code only executes LaTeX commands.

### Consideration of Shebangs
Since the provided method does not *-- at no point in the program --* open any files, the so called *Shebangs* or *Magic Lines* of LaTeX or BibTeX are not considered. Even if the shebang for using biber is included in the header of the .tex file, it still has no effect on the compilation, ie. the user needs to specify the engines that should be used by using the [Command Line Arguments](#command-line-arguments) or when using the function in form of variables. 
For references: The shebangs for using the LuaLaTeX and Biber engines for compilations would *-- have no effect on the compilation and --* look like the following:
```bash
% !TeX program = lualatex
% !BIB TS-program = biber
```


## License
[MIT](https://choosealicense.com/licenses/mit/)
