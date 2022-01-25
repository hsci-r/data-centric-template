# HSCI data-centric repository template and guidelines

## Repository layout

Put all data under `data`, all code under `code`. Track everything under `data` in [git LFS](https://git-lfs.github.com/).

Under `data`, the `data/input` folder is to contain all data that comes to the project from outside. If it comes from a git repository, use git submodules to import it. If the data is over about 500MB in size, do not include it, but instead add a `README.md` that points to it in e.g. [IDA](http://ida.fairdata.fi/).

All data produced by code should go into `data/processed`. Everything under `data/processed` should in principle be automatically regenerateable from what is in `data/input`.

If the project contains manually created data, two choices are available:

1. Put this data in `data/input` and have the `Makefile` or equivalent copy it under `data/processed`.
2. Put the data under `data/manual` instead.

If the purpose of the repo is to produce a unified clean dataset, consider having a `data/processed/final` folder which is its own git repo and a submodule. This way, downstream analysis repos can import only the clean data and not the code, input or other cruft.

If you produce files that are clearly just intermediate/temporary steps in the process, put them under `data/processed/intermediate`. Mostly do not commit these files into git. The exception is if reproducing them takes a long time, or if you think they're otherwise useful (but then maybe they don't belong under `intermediate` in the first place).

If the repository has multiple people experimenting with different things in it that are liable to make the repo a mess (e.g. exploratory analyses), partition both code and data into subdirectories by username, e.g. `code/analysis/jiemakel`, `data/processed/jiemakel` and so on, so that each person is free to make a mess only in their subpart of the project, where they'll hopefully remember what's what.

## Naming conventions

In data file columns as well as code variable and function names, use primarily `_` as a separator. E.g. `document_id`, not `documentId`, `parse_document_id` not `parseDocumentId`. For filenames, either `parse-document-ids.py` or `parse_document_ids.py` is permissible.

## Coding conventions

- When using Python, try to follow [pep8](https://www.python.org/dev/peps/pep-0008/). [autopep8](https://pypi.org/project/autopep8/) and a matching editor plugin/[pre-commit-hook](https://pre-commit.com/) or equivalent support will help in this.
- When using R, prefer [tidyverse](https://www.tidyverse.org/packages/) functions to base R. Also try to follow the [tidyverse style guide](https://style.tidyverse.org/). [styler](https://styler.r-lib.org/) and a matching editor plugin/[pre-commit-hook](https://pre-commit.com/) or equivalent support will help.
- It is desireable to have a `Makefile` or equivalent in the project, through which we can track what code produces what data and through which we can also rerun the pipelines
- Generally, try to make sure there is an easy way by which all the dependencies of a project can be installed. To ensure this, prefer isolated project environments.
  - For managing dependencies in Python projects, prefer using [Poetry](https://python-poetry.org/).
  - For managing dependencies in R, prefer using [renv](https://rstudio.github.io/renv/).
  - For projects combining Python and R, use both.
  - Don't commit the env directories created by the above tools to git, instead just commit the definition/lock files.
- Using notebooks is ok, but:
  - For Python, prefer `#%%` style in plain `.py` files instead of `.ipynb` (see e.g. [Scientific Tools for PyCharm](https://www.jetbrains.com/pycharm/features/scientific_tools.html), [Python Interactive window in VSCode](https://code.visualstudio.com/docs/python/jupyter-support-py) or [Jupytext for Jupyter Notebook](https://github.com/mwouts/jupytext)).
  - For R, prefer `.Rmd` to `.ipynb`.
  - Try to ensure dependency management is still sensible

## Data conventions

- Store data as TSV files with the `.tsv` file extension. Use `"` as the quote character, and quote character doubling as the escape method. Quote output only when necessary. Encode missing values as the empty string (not `NA` from R)
- If the file size of your uncompressed TSV file is over about 150MB, gzip the file into a `.tsv.gz`.
- For naming columns, use English, unless the number of columns is large and comes from an external source, making translating them too costly.
- If you have multiple tables, make sure column names are unique across all of them. So e.g. if both books as well as people have names, use `book_name` in the book table and `person_name` in the person table instead of using just `name` in both.
- Organize data using [tidy data](https://cran.r-project.org/web/packages/tidyr/vignettes/tidy-data.html) principles.
- Unless it is **blatantly** obvious from the file and column names (it never is), include a README file giving an explanation of what each data field contains and how the different files fit together. Also link each file to what creates it, be that a pipeline or an external source (see also above on reproducibility/Makefiles).
- For parsed date and datetime fields, use [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) (`YYYY-MM-DD` and `YYYY-MM-DDThh:mm:ss[Z]`). For datetimes, if possible use Coordinated Universal Time by first giving the local datetime and then the UTC offset, so e.g. midnight Helsinki time should be encoded as `00:00:00+02:00` or `00:00:00+03:00` depending on Daylight Saving Time instead of `22:00:00Z` or `21:00:00Z`. If resolving times to UTC is too hard, just stick to local time and don't give an UTC offset.
- If your data contains one-to-many or many-to-many relations, generally follow relational modeling principles, with one-to-one core attribute tables for the entries, and separate link tables encoding the one-to-many and many-to-many relations. Particularly, no cross-product tables that duplicate attributes. See further below.

### Handling multiple attribute values / one-to-many / many-to-many relations

If an attribute may have multiple values, in general, prefer long format separate tables:

```tsv
person_id   book_id role
person_1    book_1  author
person_1    book_1  publisher
```

In some situations, it is permissible to instead put multiple values in the same field. In these cases, use `|` as the separator.

```tsv
person_id   person_first_name   person_last_name roles
person_1    John                Doe              author|publisher
```

The situations in which this is permissible are when there are many core attributes associated with the entity, of which only one or two may contain multiple values, and it would feel stupid to extract only those into a separate table.

Never duplicate values unnecessarily by e.g. creating cross-product tables like the following:

```tsv
person_id   person_name book_id role
person_1    John Doe    book_1  author
person_1    John Doe    book_1  publisher
```

Instead, have two tables:

```tsv
person_id   person_name
person_1    John Doe
```

and

```tsv
person_id   book_id role
person_1    book_1  author
person_1    book_1  publisher
```
