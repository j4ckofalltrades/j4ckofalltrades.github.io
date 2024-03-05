---
title: "Publishing to PyPI"
description: Guide for publishing packages to PyPI 
date: 2021-06-13T22:08:53+08:00
draft: false
categories:
- dev
tags:
- pypi
- python
series:
- "Package registry publish guides"
featuredImage: "https://res.cloudinary.com/j4ckofalltrades/image/upload/v1633760759/blog/publish-guides/powerline-k8s_lfxh1c.png"
featuredImageAltText: "Sample package in the PyPI registry"
---

This guide walks you through the necessary steps to upload your package to the
:snake: Python Package Index (PyPI) with some recommendations along the way.

## Dry-run (Publishing to TestPyPI)

*This step is optional but recommended.*

[TestPyPI](https://test.pypi.org) is a separate instance of the *real* package
index which is intended for testing and experimentation. This is a good way to
test out your package before uploading to the real index.

1. Register an account for [TestPyPI](https://test.pypi.org/account/register)

2. Create a [TestPyPI API token](https://test.pypi.org/manage/account/#api-tokens)
-- make sure to set the scope to "Entire account"

3. Use [twine](https://packaging.python.org/key_projects/#twine) to upload your
package

   ```sh
   $ python3 -m pip install --upgrade twine
   # replace dist with the directory where your distribution archive is located
   $ python3 -m twine upload --repository testpypi dist/*
   ```

    You will be prompted for your TestPyPI username and password. Use the
    following values.

    | username    | password |
    | ----------- | -------- |
    | `__token__` | API token including the `pypi-` prefix |

    Alternatively you can also use a `.pypirc` file to define your package
    indexes config i.e. TestPyPI, PyPi. If you choose to go this route, create
    a `$HOME/.pypirc` file with the following contents:

    ```shell
    [testpypi]
    username=__token__
    ```

    You can also use the `password` field and paste in your API token but the
    recommended way is to use `keyring` (which is installed by Twine) for saving
    credentials such as API tokens and passwords. You can do so with:

    ```shell
    keyring set https://test.pypi.org/legacy/ <value_of_api_token>
    ```

    Once the upload process has finished, you should be able to view your
    package on TestPyPI at https://test.pypi.org/project/your-package-name.

4. Installing and testing out your package

    ```shell
    $ python3 -m pip install \
    --index-url https://test.pypi.org/simple/ --no-deps your-package
    ```

    Note that the `no-deps` flag is specified as one or more of your package's
    dependencies might not be present in TestPyPI and may cause the installation
    to fail.

    Once the installation has finished you can test it out by importing your
    package.

    ```shell
    $ python3
    >>> from example_package import your_function
    >>> your_function.do_something()
    ```

    Note that the import package will always be `example_package` regardless of
    the `name` you've configured for your package.

    That's it, you can now try and upload your package to the *real* index.

## Publishing to PyPI

The [steps](#dry-run-publishing-to-testpypi) are identical with how you would
upload a package to TestPyPI. The main difference is that you'll have to
register an account and create an API token at [PyPI](https://pypi.org), as the
former is a separate instance of PyPI.

1. Register an account for [PyPI](https://pypi.org/account/register)

2. Create a [PyPI API token](https://pypi.org/manage/account/#api-tokens)
-- make sure to set the scope to "Entire account"

3. Use [twine](https://packaging.python.org/key_projects/#twine) to upload your
package

   ```sh
   $ python3 -m pip install --upgrade twine
   # replace dist with the directory where your distribution archive is located
   $ python3 -m twine upload --repository pypi dist/*
   ```

    You will be prompted for your PyPI username and password. Use the
    following values.

    | username    | password |
    | ----------- | -------- |
    | `__token__` | API token including the `pypi-` prefix |

    Alternatively you can also use a `.pypirc` file to define your package
    indexes config i.e. TestPyPI, PyPi. If you choose to go this route, create
    a `$HOME/.pypirc` file with the following contents:

    ```shell
    [pypi]
    username=__token__
    ```

    You can also use the `password` field and paste in your API token but the
    recommended way is to use `keyring` (which is installed by Twine) for saving
    credentials such as API tokens and passwords. You can do so with:

    ```sh
    keyring set https://upload.pypi.org/legacy/ <value_of_api_token>
    ```

    When the upload process finishes, you should be able to view your package
    on PyPI at https://pypi.org/project/your-package-name

4. Installing and testing out your package

    ```shell
    python3 -m pip install your-package
    ```

    Once the installation has finished you can test it out by importing your
    package.

    ```shell
    $ python3
    >>> from example_package import your_function
    >>> your_function.do_something()
    ```

    Congrats, you've successfully published your pacakge to PyPI. :tada:

## Bonus: Automation with GitHub Packages

You'll probably want to automate this process as part of your CI/CD pipeline.
Here's how to set it up using GitHub Actions.

1. Create a [new PyPI API token](https://pypi.org/manage/account/#api-tokens)

2. Add the API token as a secret to your target GitHub repository. You can find
this under Settings > Secrets for your repo. Give the secret a name e.g.
`PYPI_API_TOKEN`.

3. Create a GitHub action workflow file in your repo at
`.github/workflows/pypi-publish.yml` with the following contents:

    ```yaml
    name: Publish package to PyPI
    
    on:
    release:
        types: [created]
    
    jobs:
    build-n-publish:
        name: Build and publish to PyPI
        runs-on: ubuntu-18.04
        steps:
        - uses: actions/checkout@master
    
        - name: Set up Python 3.7
            uses: actions/setup-python@v1
            with:
            python-version: 3.7
    
        - name: Install pypa/build
            run: >-
            python -m
            pip install
            build
            --user
        - name: Build a binary wheel and a source tarball
            run: >-
            python -m
            build
            --sdist
            --wheel
            --outdir dist/
            .
        - name: Publish package
            uses: pypa/gh-action-pypi-publish@release/v1
            with:
            password: ${{ secrets.PYPI_API_TOKEN }}
    ```

    This workflow is triggered when a new release is created but you can also
    configure it to be triggered when a different event happens e.g. when a
    new tag gets pushed. Refer to the [GitHub Actions documentation](https://docs.github.com/en/actions)
    for more configuration options.

    That's it, time to get publishing.
