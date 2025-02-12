(jupyter_style)=
# Jupyter Style Guide

These guidelines should be followed by notebooks in the documentation.
All notebooks in pymc-examples must follow this to the letter, the style
is more permissive for the ones on pymc where not everything is available.

The documentation websites are generated by Sphinx, which uses
{doc}`myst:index` and {doc}`myst-nb:index`
to parse the notebooks.

:::{tip}
There is a webinar available on [contributing to the PyMC example gallery](https://pymc-data-umbrella.xyz/en/latest/webinars/contributing_to_documentation/index.html)
:::

## General guidelines

* Don't use abbreviations or acronyms whenever you can use complete words. For example, write "random variables" instead of "RVs".

* Explain the reasoning behind each step.

* Attribute quoted text or code, and link to relevant references.

* Keep notebooks short: 20/30 cells for content aimed at beginners or intermediate users, longer notebooks are fine at the advanced level.

### MyST guidelines
Using MyST allows taking advantage of all sphinx features from markdown cells in the notebooks.
All markdown should be valid MyST (note that MyST is a superset of recommonmark).
This guide does not teach nor cover MyST extensively, only gives some opinionated guidelines.

* **Never** use url links to refer to other notebooks, PyMC documentation or other python
  libraries documentations. Use [sphinx cross-references](https://docs.readthedocs.io/en/stable/guides/cross-referencing-with-sphinx.html)
  instead.

  :::{caution}
  Using urls links breaks self referencing in versioned docs! And at the same time they are
  less robust than sphinx cross-references.
  :::

  * When linking to other notebooks, always use a `ref` type cross-reference pointing
    to the target in the {ref}`jupyter_style_first_cell`.

* If the output (or even code and output) of a cell is not necessary to follow the
  notebook or it is very long and can break the flow of reading, consider hiding
  it with a {doc}`toggle button <myst-nb:use/hiding>`

* Consider using {ref}`myst:syntax/figures` to add captions to images used in the notebook.

* Use the glossary whenever possible. If you use a term that is defined in the Glossary, link to it the first time that term appears in a significant manner. Use [this syntax](https://jupyterbook.org/content/content-blocks.html?highlight=glossary#glossaries) to add a term reference. [Link to glossary source](https://github.com/pymc-devs/pymc/blob/main/docs/source/glossary.md) where new terms should be added.

### Variable names

* Above all, stay consistent with variable names within the notebook. Notebooks using multiple names for the same variable will not be merged.
* Use meaningful variable names wherever possible. Our users come from different backgrounds and not everyone is familiar with the same naming conventions.
  - Annotate dimensions too. Notebooks are published to be read, so even if the shape is derived
    from the inputs or you don't like to use named dims and don't use them in your presonal
    code, notebooks must use dims, even if annotating and not setting the shape.
    It makes the code easier to follow, especially for newcomers.
* Sometimes it makes sense to use Greek letters to refer to variables, for example when writing equations, as this makes them easier to read. In that case, use LaTeX to insert the Greek letter like this `$\theta$` instead of using Unicode like `θ`.
* If you need to use Greek letter variable names inside the code, please spell them out instead of using unicode. For example, `theta`, not `θ`.
* When using non meaningful names such as single letters, add bullet points with a 1-2 sentence description of each variable below the equation where they are first introduced.

Choosing variable names can sometimes be difficult, tedious or annoying.
In case it helps, the dropdown below has some suggestions so you can focus on writing the actual content

:::::::{dropdown} Variable name suggestions
:icon: light-bulb

**Models and sampling results**
* Use `idata` for sampling results, always containing a variable of type InferenceData.
* Store inferecedata groups as variables to ease writing and reading of code operating on sampling results.
  Use underscore separated 3-5 word abbreviations or the group name. Some examples of `abbrebiation`/`group_name`:
  `post`/`posterior`, `const`/`constant_data`, `post_pred`/`posterior_predictive` or `obs_data`/`observed_data`
* For stats and diagnostics, use the ArviZ function name as variable name: `ess = az.ess(...)`, `loo = az.loo(...)`
* If there are multiple models in a notebook, assign a prefix to each model,
  and use it throughout to identify which variables map to each model.
  Taking the famous eight school as example, with a `centered` and `non_centered` model
  to compare parametrizations, use `centered_model` (pm.Model object), `centered_idata`, `centered_post`, `centered_ess`... and `non_centered_model`, `non_centered_idata`...

**Dimension and random variable names**
* Use singular dimension names, following ArviZ `chain` and `draw`.
  For example `cluster`, `axis`, `component`, `forest`, `time`...
* If you can't think of a meaningful name for the dimension representing the number of observations such as time, fall back to `obs_id`.
* For matrix dimensions, as xarray doesn't allow repeated dimension names, add a `_bis` suffix. i.e. `param, param_bis`.
* For the dimension resulting from stacking `chain` and `draw` use `sample`, that is `.stack(sample=("chain", "draw"))`.
* We often need to encode a categorical variable as integers. add `_idx` to the name of the variable it's encoding.
  i.e. from `floor` and `county` to `floor_idx` and `county_idx`.
* To avoid clashes and overwriting variables when using `pm.Data`, use the following pattern:

  ```
  x = np.array(...)
  with pm.Model():
      x_ = pm.Data("x", x)
      ...
  ```

  This avoids overwriting the original `x` while having `idata.constant_data["x"]`,
  and within the model `x_` is still available to play the role of `x`.
  Otherwise, always try to use the same variable name as the string name given to the PyMC random variable.

**Plotting**
* Matplotlib figures and axes. Use:
  * `fig` for matplotlib figures
  * `ax` for a single matplotib axes object
  * `axs` for arrays of matplotlib axes objects

  When manually working with multiple matplotlib axes, use local `ax` variables:

  ::::{tab-set}
  :::{tab-item} Local `ax` variables
  ```{code-block} python
  :emphasize-lines: 3, 7
  fig, axs = pyplot.subplots()

  ax = axs[0, 1]
  ax.plot(...)
  ax.set(...)

  ax = axs[1, 2]
  ax.scatter(...)
  ```
  :::
  :::{tab-item} Instead of subsetting every time
  ```
  fig, axs = pyplot.subplots()

  axs[0, 1].plot(...)
  axs[0, 1].set(...)

  axs[1. 2].scatter(...)
  ```
  :::
  ::::

  This makes editing the code if restructuring the subplots easier, only one change per subplot
  is needed instead of one change per matplotlib function call.

* It is often useful to make a numpy linspace into an {class}`~xarray.DataArray`
  for xarray to handle aligning and broadcasing automatically and ease computation.
  * If a dimension name is needed, use `x_plot`
  * If a variable name is needed for the original array and DataArray to coexist, add `_da` suffix

  Thus, ending up with code like:

  ```
  x = xr.DataArray(np.linspace(0, 10, 100), dims=["x_plot"])
  # or
  x = np.linspace(0, 10, 100)
  x_da = xr.DataArray(x)
  ```

**Looping**
* When using enumerate, take the first letter of the variable as the count:

  ```
  for p, person in enumerate(persons)
  ```

* When looping, if you need to store a variable after subsetting with the loop index,
  append the index variable used for looping to the original variable name:

  ```{code-block} python
  :emphasize-lines: 4, 6
  variable = np.array(...)
  x = np.array(...)
  for i in range(N):
      variable_i = variable[i]
      for j in range(K):
          x_j = x[j]
          ...
  ```

:::::::

(jupyter_style_first_cell)=
## First cell
The first cell of all example notebooks should have a MyST target, a level 1 markdown title (that is a title with a single `#`) followed by the post directive.
The syntax is as follows:

```markdown
(notebook_name)=
# Notebook Title

:::{post} Aug 31, 2021
:tags: tag1, tag2, tags can have spaces, tag4
:category: level
:author: Alice Abat, Bob Barceló
:::
```

The date should correspond to the latest update/execution date, at least roughly (it's not a problem if the date is a few days off due to the review process before merging the PR). This will allow users to see which notebooks have been updated lately and will help the PyMC team make sure no notebook is left outdated for too long.

:::{important}
The {ref}`MyST target <myst:syntax/targets>` (the `(notebook_name)=` bit)
is used to link notebooks between each other.
It must be notebook specific, for example its file name.
**Do not copy paste this and leave `notebook_name` unmodified**
:::

Tags can be anything, but we ask you to try to use [existing tags](https://github.com/pymc-devs/pymc/wiki/Categories-and-Tags-for-PyMC-Examples)
to avoid the tag list from getting too long.

Each notebook should have a one or two categories indicating:
* the level of the notebook (required):
  - `beginner` (standing crow icon)
  - `intermediate` (flying dove icon)
  - `advanced` (dragon icon)
* the [diataxis](https://diataxis.fr/) type (optional for old notebooks):
  - `tutorial`
  - `how-to`
  - `explanation`
  - `reference`

Authors should list people who authored, adapted or updated the notebook. See {ref}`jupyter_authors`
for more details.

## Extra dependencies
If the notebook uses libraries that are not PyMC dependencies, these extra dependencies should
be indicated together with some advise on how to install them.
This ensures readers know what they'll need to install beforehand and can for example
decide between running it locally or on binder.

To make things easier for notebook writers and maintainers, pymc-examples contains
a template for this that warns about the extra dependencies and provides specific
installation instructions inside a dropdown.

Thus, notebooks with extra dependencies should:

1.  list the extra dependencies as notebook metadata using the `myst_substitutions` category
    and then either the `extra_dependencies` or the `pip_dependencies` and `conda_dependencies`.
    In addition, there is also an `extra_install_notes` to include custom text inside the dropdown.

    * notebook metadata can be edited from the menu with {menuselection}`Edit --> Edit notebook metadata`

      This will open a window with json formatted text that might look a bit like:

      ::::{tab-set}
      :::{tab-item} No myst_substitutions

      ```json
      {
        "kernelspec": {
          "name": "python3",
          "display_name": "Python 3 (ipykernel)",
          "language": "python"
        },
        "language_info": {
          "name": "python",
          "version": "3.9.7",
          "mimetype": "text/x-python",
          "codemirror_mode": {
            "name": "ipython",
            "version": 3
          },
          "pygments_lexer": "ipython3",
          "nbconvert_exporter": "python",
          "file_extension": ".py"
        }
      }
      ```
      :::

      :::{tab-item} extra_dependencies key

      ```{code-block} json
      :emphasize-lines: 19-21
      {
        "kernelspec": {
          "name": "python3",
          "display_name": "Python 3 (ipykernel)",
          "language": "python"
        },
        "language_info": {
          "name": "python",
          "version": "3.9.7",
          "mimetype": "text/x-python",
          "codemirror_mode": {
            "name": "ipython",
            "version": 3
          },
          "pygments_lexer": "ipython3",
          "nbconvert_exporter": "python",
          "file_extension": ".py"
        },
        "substitutions": {
          "extra_dependencies": "bambi seaborn"
        }
      }
      ```
      :::

      :::{tab-item} pip and conda specific keys
      ```{code-block} json
      :emphasize-lines: 19-22
      {
        "kernelspec": {
          "name": "python3",
          "display_name": "Python 3 (ipykernel)",
          "language": "python"
        },
        "language_info": {
          "name": "python",
          "version": "3.9.7",
          "mimetype": "text/x-python",
          "codemirror_mode": {
            "name": "ipython",
            "version": 3
          },
          "pygments_lexer": "ipython3",
          "nbconvert_exporter": "python",
          "file_extension": ".py"
        },
        "substitutions": {
          "pip_dependencies": "graphviz",
          "conda_dependencies": "python-graphviz",
        }
      }
      ```

      The pip and conda spcific keys overwrite the `extra_installs` one, so it doesn't make
      sense to use `extra_installs` if using them. Either both pip and conda substitutions
      are defined or none of them is.
      :::
      ::::

1.  include the warning and installation advise template with the following markdown right before
    the extra dependencies are imported:

    ```markdown
    :::{include} ../extra_installs.md
    :::
    ```

## Code preamble

In a cell just below the cell where you imported matplotlib and/or ArviZ (usually the first one),
set the ArviZ style to darkgrid (this has to be in another cell than the matplotlib import because of the way matplotlib sets its defaults):

```python
RANDOM_SEED = 8927
rng = np.random.default_rng(RANDOM_SEED)
az.style.use("arviz-darkgrid")
```

A good practice _when generating synthetic data_ is also to set a random seed as above, to improve reproducibility. Also, please check convergence (e.g. `assert all(r_hat < 1.03)`) because we sometime re-run notebooks automatically without carefully checking each one.

## Reading from file

Use a `try... except` clause to load the data and use `pm.get_data` in the except path. This will ensure that users who have cloned pymc-examples repo will read their local copy of the data while also downloading the data from github for those who don't have a local copy. Here is one example:

```python
try:
    df_all = pd.read_csv(os.path.join("..", "data", "file.csv"), ...)
except FileNotFoundError:
    df_all = pd.read_csv(pm.get_data("file.csv"), ...)
```

## pre-commit and code formatting
We run some code-quality checks on our notebooks during Continuous Integration. The easiest way to make sure your notebook(s) pass the CI checks is using [pre-commit](https://github.com/pre-commit/pre-commit). You can install it with

```bash
pip install -U pre-commit
```

and then enable it with

```bash
pre-commit install
```

Then, the code-quality checks will run automatically whenever you commit any changes. To run the code-quality checks manually, you can do, e.g.:

```bash
pre-commit run --files notebook1.ipynb notebook2.ipynb
```

replacing `notebook1.ipynb` and `notebook2.ipynb` with any notebook you've modified.

NB: sometimes, [Black will be frustrating](https://stackoverflow.com/questions/58584413/black-formatter-ignore-specific-multi-line-code/58584557#58584557) (well, who isn't?). In these cases, you can disable its magic for specific lines of code: just write `#fmt: on/off` to disable/re-enable it, like this:

```python
# fmt: off
np.array(
    [
        [1, 0, 0, 0],
        [0, -1, 0, 0],
        [0, 0, 1, 0],
        [0, 0, 0, -1],
    ]
)
# fmt: on
```

(jupyter_authors)=
## Authorship and attribution
After the notebook content finishes, there should be an `## Authors` section with bullet points
to provide attribution to the people who contributed to the the general pattern should be:

```markdown
* <verb> by <author> on <date> ([repo#PR](https://link-to.pr))
```

where `<verb>` must be one listed below, `<author>` should be the name (multiple people allowed)
which can be formatted as hyperlink to personal site or GitHub profile of the person,
and `<date>` should preferably be month and year.

authored
: for notebooks created specifically for pymc-examples

adapted
: for notebooks adapted from other sources such as books or blogposts.
  It will therefore follow a different structure than the example above
  in order to include a link or reference to the original source:

  ```markdown
  Adapted from Alice's [blogpost](blog.alice.com) by Bob and Carol on ...
  ```

re-executed
: for notebooks re-executed with a newer PyMC version without significant changes to the code.
  It can also mention the PyMC version used to run the notebook.

updated
: for notebooks that have not only been re-executed but have also had significant updates to
  their content (either code, explanations or both).

some examples:

```markdown
* Authored by Chris Fonnesbeck in May, 2017 ([pymc#2124](https://github.com/pymc-devs/pymc/pull/2124))
* Updated by Colin Carroll in June, 2018 ([pymc#3049](https://github.com/pymc-devs/pymc/pull/3049))
* Updated by Alex Andorra in January, 2020 ([pymc#3765](https://github.com/pymc-devs/pymc/pull/3765))
* Updated by Oriol Abril in June, 2020 ([pymc#3963](https://github.com/pymc-devs/pymc/pull/3963))
* Updated by Farhan Reynaldo in November 2021 ([pymc-examples#246](https://github.com/pymc-devs/pymc-examples/pull/246))
```

and

```markdown
* Adapted from chapter 5 of Bayesian Data Analysis 3rd Edition {cite:p}`gelman2013bayesian`
  by Demetri Pananos and Junpeng Lao on June, 2018 ([pymc#3054](https://github.com/pymc-devs/pymc/pull/3054))
* Reexecuted by Ravin Kumar with PyMC 3.6 on March, 2019 ([pymc#3397](https://github.com/pymc-devs/pymc/pull/3397))
* Reexecuted by Alex Andorra and Michael Osthege with PyMC 3.9 on June, 2020 ([pymc#3955](https://github.com/pymc-devs/pymc/pull/3955))
* Updated by Raúl Maldonado 2021 ([pymc-examples#24](https://github.com/pymc-devs/pymc-examples/pull/24), [pymc-examples#45](https://github.com/pymc-devs/pymc-examples/pull/45) and [pymc-examples#147](https://github.com/pymc-devs/pymc-examples/pull/147))
```

## References
References should be added to the [`references.bib`](https://github.com/pymc-devs/pymc-examples/blob/main/examples/references.bib) file in bibtex format, and cited with [sphinxcontrib-bibtex](https://sphinxcontrib-bibtex.readthedocs.io/en/latest/) within the notebook text wherever they are relevant.

The references in the `.bib` file should have as id something along the lines `authorlastnameYEARkeyword` or `libraryYEARkeyword` for documentation pages, and they should be alphabetically sorted by this id in order to ease finding references within the file and preventing adding duplicate ones.

References can be cited twice within a single notebook. Two common reference formats are:

```
{cite:p}`bibtex_id`  # shows the reference author and year between parenthesis
{cite:t}`bibtex_id`  # textual cite, shows author and year without parenthesis
```

which can be added inline, within the text itself. At the end of the notebook, add the bibliography with the following markdown

```markdown
## References

:::{bibliography}
:filter: docname in docnames
:::
```

or alternatively, if you wanted to add extra references that have not been cited within the text, use:

```markdown
## References

:::{bibliography}
:filter: docname in docnames

extra_bibtex_id_1
extra_bibtex_id_2
:::
```

## Watermark
Once you're finished with your NB, add a very last cell with [the watermark package](https://github.com/rasbt/watermark). This will automatically print the versions of Python and the packages you used to run the NB -- reproducibility rocks! Here is some example code. Note that the `-p` argument may not be necessary (or it may need to have different libraries as input), but all the other arguments must be present.

```python
%load_ext watermark
%watermark -n -u -v -iv -w -p aesara,aeppl,xarray
```

This second to last code cell should be preceded by a markdown cell with the `## Watermark` title only so it appears in the table of contents.

`watermark` should be in your virtual environment if you installed our `requirements-dev.txt`.
Otherwise, just run `pip install watermark`.
The `p` flag is optional but should be added if Aesara or xarray are not imported explicitly.
This will also be checked by `pre-commit` (because we all forget to do things sometimes 😳).

## Epilogue
The last cell in the notebooks should be a markdown cell with exactly the following content:

```
:::{include} ../page_footer.md
:::
```

The only exception being notebooks that are not on the usual place and therefore need to
update the path to page footer for the include to work.

---

You're all set now 🎉 You can push your changes, open a pull request, and, once it's merged, rest with the feeling of a job well done 👏
Thanks a lot for your contribution to open-source, we really appreciate it!
