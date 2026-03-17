# WGXC-DC Python Packaging Workshop

This is a tutorial on the basics of Python packaging.

The content of this tutorial is based on the [Python Packaging 101 tutorial](https://www.pyopensci.org/python-package-guide/tutorials/intro.html)
from PyOpenSci.

We will cover only the basics, but if you are interested in going deeper, we highly recommend checking the [PyOpenSci tutorial](https://www.pyopensci.org/python-package-guide/tutorials/intro.html).

## Prerequisites

Before the workshop, please create an account on [TestPyPI](https://test.pypi.org/account/register/). We will use TestPyPI to practice publishing your package without affecting the real Python Package Index (PyPI).

## Step 1: Setup

We will use GitHub Codespaces for this tutorial, since it removes OS-specific installation hiccups.

1. Fork the repository: https://github.com/pyOpenSci/pyopensci-workshop-create-python-package

   This repo has a workspace already set up with VSCode, copier, Hatch, and everything you need to create your first Python package. From this codespace you will also be able to push your work during the workshop to your own fork.

2. **Before creating the codespace**, update the hatch version in your fork. In `.devcontainer/devcontainer.json`, change the hatch version to `1.16`:

   ```json
   "ghcr.io/devcontainers-extra/features/hatch:1": {
       "version": "1.16"
   }
   ```

   This avoids a [known incompatibility](https://github.com/pypa/hatch/issues/2193) with the older version pre-installed in the devcontainer.

3. From your fork, go to the green `< > Code` tab and select "Create codespace on main".

   ![Open code space instructions](https://www.pyopensci.org/images/github/codespaces/create-github-codespace-main.webp)

4. Once in the codespace, create a branch for your work:

   ```bash
   git checkout -b first-python-package
   ```

### Local installation (for home)

If you are comfortable setting this up locally, you will need:

- [Install Hatch](https://www.pyopensci.org/python-package-guide/tutorials/get-to-know-hatch.html#install-hatch)
- [Install Copier](https://copier.readthedocs.io/en/stable/#installation)

## Step 2: Create your Python package

1. First, make a directory where your package will live:

   ```bash
   mkdir wgxc-package
   ```

2. Use the pyOpenSci copier template to scaffold your package. The template uses Hatch as the default packaging tool.

   ```bash
   copier copy gh:pyopensci/pyos-package-template .
   ```

   After running the command, the template will walk you through a series of questions.

   When you reach the prompt **"Do you want to answer one more question, and skip the rest, using the default values?"** choose **Yes, but with a minimal setup**. For this tutorial we want to create the most basic version of your package that contains documentation, tests, and an example module.

   The template will ask you for your GitHub username, your package name, and other basic questions. It will then create a package with basic tests, documentation, and GitHub configuration.

   You will learn about the basics of the Python package directory structure and associated key files (`__init__.py` and `pyproject.toml`).

## Step 3: Install your package

1. Create a virtual environment and activate it:

   ```bash
   uv venv
   source .venv/bin/activate
   ```

2. Install your package in editable mode:

   ```bash
   uv pip install -e .
   ```

   > **Note:** You may see a warning about hardlinks:
   > ```
   > warning: Failed to hardlink files; falling back to full copy.
   > ```
   > This is normal in Codespaces (and Docker containers, mounted volumes, etc.) because the uv cache and your virtual environment are on different filesystems. uv copies the files instead of hardlinking them, but everything installs correctly.

3. Verify the installation:

   ```bash
   uv pip list
   ```

## Step 4: Test your package

1. Try importing your package in a Python shell:

   ```python
   >>> import wgxcpack
   >>> from wgxcpack.example import add_numbers
   >>> help(add_numbers) # or print(add_numbers.__doc__)
   >>> add_numbers(1, 2)
   3
   ```

2. Install your test dependencies. The template defines a `tests` extra (check the `pyproject.toml`), so you can install everything at once:

   ```bash
   uv pip install -e ".[tests]"
   ```

3. Run the test suite:

   ```bash
   pytest
   ```

Let's explore teh test we just run. `/tests/test_examples.py`

### Useful pytest options

The template includes several useful pytest plugins:

- **randomly** — randomizes test order automatically. To reproduce a failure, reuse the seed:
  ```bash
  pytest -p randomly --randomly-seed=49669506
  ```
- **xdist** — run tests in parallel across CPUs:
  ```bash
  pytest -n auto
  ```
- **cov** — measure code coverage:
  ```bash
  pytest --cov=wgxcpack
  pytest --cov=wgxcpack --cov-report=term-missing  # detailed report
  ```

You can combine them:

```bash
pytest -n auto --cov=wgxcpack --cov-report=term-missing
```

## Step 5: Add a dependency

> **Tip:** Before editing `.toml` files, install the **Even Better TOML** extension in VS Code for syntax highlighting and validation. In the codespace, open the Extensions panel (`Ctrl+Shift+X`) and search for "Even Better TOML".

To add an external dependency (e.g., NumPy), edit your `pyproject.toml`:

```toml
dependencies = ["numpy>=2.0"]
```

Then re-install your package so the new dependency is picked up:

```bash
uv pip install -e .
```

Now let's add a function that uses NumPy. Add the following to your existing `example.py` file (e.g., `src/wgxcpack/example.py`):

```python
import numpy as np


def sqrt_array(values):
    """Compute the square root of a list of numbers using NumPy.

    Parameters
    ----------
    values : list or float
        A list of non-negative numbers or a single non-negative float.

    Returns
    -------
    numpy.ndarray
        An array with the square root of each input value.

    Raises
    ------
    ValueError
        If any value in the list is negative.

    Examples
    --------
    >>> sqrt_array([4, 9, 16])
    array([2., 3., 4.])

    >>> sqrt_array(9)
    np.float64(3.0)
    """
    arr = np.array(values)
    if np.any(arr < 0):
        raise ValueError("Input contains negative values.")
    return np.sqrt(arr)
```

Run tests and coverage

```bash
pytest -n auto --cov=wgxcpack --cov-report=term-missing
```

Next, add tests for this function in your test file (e.g., `tests/test_example.py`):

```python
import numpy as np
import pytest
from wgxcpack.example import sqrt_array


def test_sqrt_array_float():
    result = sqrt_array(9)
    assert result == np.float64(3.0)

def test_sqrt_array():
    result = sqrt_array([4, 9, 16])
    expected = np.array([2.0, 3.0, 4.0])
    np.testing.assert_array_equal(result, expected)


def test_sqrt_array_with_zero():
    result = sqrt_array([0, 1])
    expected = np.array([0.0, 1.0])
    np.testing.assert_array_equal(result, expected)


def test_sqrt_array_negative_raises():
    with pytest.raises(ValueError, match="negative"):
        sqrt_array([-1, 4, 9])
```

Run the tests to make sure everything works, and the coverage is 100%:

```bash
pytest
```

For a reference on supported Python versions, see the [Python Developer's Guide](https://devguide.python.org/versions/).

## Step 6: Push to GitHub and open a PR

1. Stage and commit your changes:

   ```bash
   git add .
   git commit -m "add first python package"
   ```

2. Push your branch to your fork:

   ```bash
   git push origin first-python-package
   ```

3. Open a Pull Request against your own fork's `main` branch on GitHub.

## Step 7: Install from GitHub

Once your code is on GitHub, anyone can install your package directly from the repository, let's try that before merging into main:

In our case the package is in a branch, and in a subdirectory, we need to tell `uv pip` how to find the `pyproject.toml`

```bash
uv pip install git+https://github.com/ncclementi/pyopensci-workshop-create-python-package.git@first-python-package#subdirectory=wgxc-package
```

You can check where the package is being installed by doing


```
uv pip freeze
```

**NOTE:** In general if you have a repo dedicated to your package, but want to install from a branch you will just do:

```bash
python -m pip install git+https://github.com/<your-username>/<your-repo>.git@first-python-package
```

Replace `<your-username>` and `<your-repo>` with the appropriate values. You can also point to a tag or specific commit.

For more details, see the [PyOpenSci guide on installing from GitHub](https://www.pyopensci.org/python-package-guide/tutorials/create-python-package.html#step-6-test-out-your-new-package).

## Step 8: Publish to TestPyPI

### Set up TestPyPI

1. Set up 2FA — you will need it to be able to get a token.
2. We will be publishing in the most secure way we can do manually, and for that we need an API token.

**Get an API Token**

1. Go to <https://test.pypi.org/manage/account/>
2. Scroll to the API Token section.
3. The first time you do this, you'll need to scope the token to "Entire Account (all projects)". This is because your package isn't there yet to create a package-specific token. Later you can delete this token and create a project-scoped one.
4. Copy the token somewhere safe — we will need it shortly.

> **Note:** There are better ways of doing this using CI. We are not going to cover that due to time, but there are detailed instructions in the [PyOpenSci tutorial - trusted publishing](https://www.pyopensci.org/python-package-guide/tutorials/trusted-publishing.html).

### Build the package with hatch

The `pyproject.toml` has dependencies and configuration needed to build the package with hatch. We need to build the right distribution files (sdist and wheel).

Hatch lets you define different environments that perform specific tasks. One of them is the `build` environment (this is a custom name defined under `[tool.hatch.envs.build]`).

For extra info, see the [PyOpenSci publishing guide](https://www.pyopensci.org/python-package-guide/tutorials/publish-pypi.html).

In the `wgxc-package` directory, verify hatch is available:

```bash
hatch --version
```

```bash
hatch env show
```

### Fix hatch build compatibility

The version of hatch installed in the devcontainer has a known issue ([hatch#2193](https://github.com/pypa/hatch/issues/2193)) that causes `hatch run build:check` to fail. To fix this, we need to make two changes to the `pyproject.toml`:

1. Add `hatchling` as a dependency in the `build` environment:

   ```toml
   [tool.hatch.envs.build]
   dependencies = [
       "hatchling",
       "pip-audit",
       "twine",
   ]
   ```

2. Update the `check` script to use `hatchling build` instead of `hatch build`:

   ```toml
   [tool.hatch.envs.build.scripts]
   check = [
       "pip check",
       "hatchling build {args:--clean}",
       "twine check dist/*",
   ]
   ```

For more details, see [pyopensci-workshop-create-python-package#22](https://github.com/pyOpenSci/pyopensci-workshop-create-python-package/issues/22).

### Run the build check

Let's run the `check` script:

```bash
hatch run build:check
```

If everything passes, we are ready to publish.

### Publish to TestPyPI with hatch

1. Disable keyring so hatch prompts for the token interactively:

   ```bash
   export PYTHON_KEYRING_BACKEND=keyring.backends.null.Keyring
   ```

   By default, hatch tries to store and retrieve your API token using the system keyring. In Codespaces (and many CI/headless environments), there is no keyring service available, so hatch will fail. Setting `PYTHON_KEYRING_BACKEND` to the null backend tells hatch to skip keyring entirely and prompt you for the token directly in the terminal.

2. Build the package:

   ```bash
   hatch build --clean
   ```

3. Publish to TestPyPI:

   ```bash
   hatch publish -r test
   ```

   It will ask for a username — enter `__token__` — and a password — paste your API token.

   > **Security note:** Never pass the token via `-a` or `export` on the command line — it gets saved in shell history. After publishing, you can clear your history with:
   >
   > ```bash
   > history -c && > ~/.bash_history
   > ```

### Alternative: publish with twine

If `hatch publish` does not work for you, you can use `twine` as an alternative.

1. Install twine:

   ```bash
   uv pip install twine
   ```

2. Upload the distribution files to TestPyPI:

   ```bash
   twine upload --repository testpypi dist/* --verbose
   ```

   When prompted, paste your API token.

### Verify the published package

1. Your package should now be on TestPyPI. Let's verify by installing it in a fresh environment:

   ```bash
   uv venv new-env
   source new-env/bin/activate
   ```

2. Because `numpy` is a dependency, we need to tell `pip` it can grab it from the real PyPI by providing an extra index:

   ```bash
   uv pip install --index-url https://test.pypi.org/simple/ --extra-index-url https://pypi.org/simple/ wgxcpack
   ```

## Conclusion

Congratulations! You have created a Python package from scratch, added tests, built distribution files, and published to TestPyPI. These are the foundational steps for any Python package you create in the future.

To learn more about Python packaging best practices — including documentation, CI/CD, community guidelines, and more — check out the [PyOpenSci Python Package Guide](https://www.pyopensci.org/python-package-guide/index.html).

## Viewing the slides

The workshop slides are in the `slides/` directory and use [Marp](https://marp.app/) for rendering.

### Option 1: VS Code extension (recommended)

1. Install the **Marp for VS Code** extension (`marp-team.marp-vscode`).
2. Open `slides/slides.md` and click the preview icon in the top right corner.

### Option 2: Marp CLI

1. Install Marp CLI. For example, with Homebrew:

   ```bash
   brew install marp-cli
   ```

   For other installation methods, see the [Marp CLI install instructions](https://github.com/marp-team/marp-cli?tab=readme-ov-file#install).

2. Live preview with hot reload (from the `slides/` directory):

   ```bash
   marp -s slides
   ```

   Or preview a single file directly:

   ```bash
   marp slides/slides.md
   ```

3. Export to HTML or PDF (from the `slides/` directory):

   ```bash
   cd slides
   marp slides.md -o slides.html --theme theme.css
   marp slides.md --pdf -o slides.pdf --theme theme.css
   ```

   > **Note:** For PDF export, the Montserrat font must be installed locally on your machine. The HTML export loads it automatically via Google Fonts.
