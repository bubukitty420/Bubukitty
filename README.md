# Bubukittyname: build addon

on:
  push:
    tags: ["*"]
    # To build on main/master branch, uncomment the following line:
    branches: [ main , master ]

  pull_request:
    branches: [ main, master ]

  workflow_dispatch:

jobs:
  build:

    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v3

    - run: echo -e "pre-commit\nscons\nmarkdown">requirements.txt

    - name: Set up Python
      uses: actions/setup-python@v4
      with:
        python-version: 3.9
        cache: 'pip'

    - name: Install dependencies
      run: |
        python -m pip install --upgrade pip wheel
        pip install -r requirements.txt
        sudo apt-get update  -y
        sudo apt-get install -y gettext

    - name: Code checks
      run: export SKIP=no-commit-to-branch; pre-commit run --all

    - name: building addon
      run: scons

    - uses: actions/upload-artifact@v3
      with:
        name: packaged_addon
        path: ./*.nvda-addon

  upload_release:
    runs-on: ubuntu-latest
    if: ${{ startsWith(github.ref, 'refs/tags/') }}
    needs: ["build"]
    steps:
    - uses: actions/checkout@v3
    - name: download releases files
      uses: actions/download-artifact@v3
    - name: Display structure of downloaded files
      run: ls -R

    - name: Release
      uses: softprops/action-gh-release@v1
      with:
        files: packaged_addon/*.nvda-addon
        fail_on_unmatched_files: true
        prerelease: ${{ contains(github.ref, '-') }}
