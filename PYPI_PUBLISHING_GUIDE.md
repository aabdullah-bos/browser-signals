# PyPI Publishing Guide for browser-signals

## Pre-Publishing Checklist

### ✅ Already Done

- [x] Project structure with `src/` layout
- [x] `pyproject.toml` with proper metadata
- [x] README.md with clear documentation
- [x] LICENSE file (MIT)
- [x] Version number set to 0.1.0
- [x] Author information updated
- [x] GitHub repository URLs configured
- [x] MANIFEST.in created

### 📝 To Do Before First Publish

1. **Update your email in `pyproject.toml`**
   - Currently set to: `aquil@aquilabdullah.com`
   - Change to your actual email address

2. **Verify package name availability**
   - Visit <https://pypi.org/project/browser-signals/>
   - If the name is taken, you'll need to choose a different name

3. **Install build tools**

   ```bash
   pip install --upgrade build twine
   ```

4. **Create PyPI account**
   - Register at <https://pypi.org/account/register/>
   - Register at <https://test.pypi.org/account/register/> (for testing)

5. **Set up API tokens (recommended over passwords)**
   - Go to <https://pypi.org/manage/account/token/>
   - Create a new API token
   - Save it securely (you'll only see it once)

## Publishing Steps

### Test on TestPyPI First (Recommended)

1. **Build the package**

   ```bash
   cd /Users/aquilabdullah/devel/aquil/projects/browser-signals
   python -m build
   ```

   This creates `dist/browser_signals-0.1.0-py3-none-any.whl` and `.tar.gz` files

2. **Upload to TestPyPI**

   ```bash
   python -m twine upload --repository testpypi dist/*
   ```

   - Username: `__token__`
   - Password: your TestPyPI API token (including the `pypi-` prefix)

3. **Test installation from TestPyPI**

   ```bash
   pip install --index-url https://test.pypi.org/simple/ --no-deps browser-signals
   ```

4. **Verify the package works**
   - Create a new virtual environment
   - Install your package
   - Test import and basic functionality

### Publish to PyPI (Production)

1. **Clean previous builds** (if you made changes after testing)

   ```bash
   rm -rf dist/ build/ src/*.egg-info
   python -m build
   ```

2. **Upload to PyPI**

   ```bash
   python -m twine upload dist/*
   ```

   - Username: `__token__`
   - Password: your PyPI API token (including the `pypi-` prefix)

3. **Verify on PyPI**
   - Visit <https://pypi.org/project/browser-signals/>
   - Check that all information displays correctly

4. **Test installation**

   ```bash
   pip install browser-signals
   ```

## Using .pypirc for Easier Authentication

Create `~/.pypirc` to store credentials:

```ini
[distutils]
index-servers =
    pypi
    testpypi

[pypi]
username = __token__
password = pypi-YourActualAPITokenHere

[testpypi]
repository = https://test.pypi.org/legacy/
username = __token__
password = pypi-YourTestPyPIAPITokenHere
```

Then you can upload without entering credentials:

```bash
python -m twine upload --repository testpypi dist/*
python -m twine upload dist/*
```

## Future Releases

For subsequent releases:

1. **Update version number** in two places:
   - `pyproject.toml` → `[project] version = "0.1.1"`
   - `src/browser_signals/__init__.py` → `__version__ = "0.1.1"`

2. **Clean, build, and upload**:

   ```bash
   rm -rf dist/ build/ src/*.egg-info
   python -m build
   python -m twine upload dist/*
   ```

## Troubleshooting

### "File already exists" error

- You cannot re-upload the same version
- Increment the version number and rebuild

### Import errors after installation

- Verify package structure: ensure `src/browser_signals/__init__.py` exports the right symbols
- Check dependencies in `pyproject.toml`

### Missing files in distribution

- Update `MANIFEST.in`
- Run `python -m build` and inspect the `.tar.gz` contents:

  ```bash
  tar -tzf dist/browser_signals-0.1.0.tar.gz
  ```

## Automated Publishing with GitHub Actions

This repository uses GitHub Actions with Trusted Publishing (OpenID Connect) to automatically publish to PyPI when you create a release from the `releases` branch.

### Initial Setup (One-time)

1. **Create the `pypi` environment in GitHub**:
   - Go to <https://github.com/aabdullah-bos/browser-signals/settings/environments>
   - Click "New environment"
   - Name it: `pypi`
   - Click "Configure environment"
   - (Optional) Add protection rules like requiring approval before deploying

2. **Configure Trusted Publishing on PyPI** (after first manual upload):
   - Go to <https://pypi.org/manage/project/browser-signals/settings/publishing/>
   - Add a new pending publisher with:
     - PyPI Owner: `aabdullah-bos`
     - Repository name: `browser-signals`
     - Workflow name: `python-publish.yml`
     - Environment name: `pypi`
   - Save

### Release Workflow

The repository uses two branches:
- **`main`** - Active development branch
- **`releases`** - Stable branch for published versions

**Process for creating a new release:**

1. **Develop on `main` branch**:
   ```bash
   git checkout main
   # ... make changes, test, commit ...
   ```

2. **Prepare for release**:
   - Update version number in two places:
     - `pyproject.toml` → `version = "0.1.1"`
     - `src/browser_signals/__init__.py` → `__version__ = "0.1.1"`
   - Commit the version bump:
     ```bash
     git add pyproject.toml src/browser_signals/__init__.py
     git commit -m "Bump version to 0.1.1"
     git push origin main
     ```

3. **Merge to releases branch**:
   ```bash
   git checkout releases
   git merge main
   git push origin releases
   ```

4. **Create and push a tag**:
   ```bash
   git tag v0.1.1
   git push origin v0.1.1
   ```

5. **Create GitHub Release**:
   - Go to <https://github.com/aabdullah-bos/browser-signals/releases>
   - Click "Draft a new release"
   - Choose the tag you just pushed (`v0.1.1`)
   - **Important**: Ensure the target is the `releases` branch
   - Add release title (e.g., "v0.1.1")
   - Add release notes describing changes
   - Click "Publish release"

6. **Automated publishing**:
   - GitHub Actions automatically builds the package
   - The workflow publishes to PyPI using Trusted Publishing
   - Monitor progress at: <https://github.com/aabdullah-bos/browser-signals/actions>

### Benefits of This Workflow

- **No API tokens needed** - Uses OpenID Connect for secure authentication
- **Branch protection** - Only releases from `releases` branch are published
- **Automatic** - No manual building or uploading required
- **Audit trail** - All releases tied to GitHub releases with notes

### Manual Publishing (Alternative)

If you need to publish manually instead of using GitHub Actions:

```bash
# From the releases branch
git checkout releases
rm -rf dist/ build/ src/*.egg-info
python -m build
python -m twine upload dist/*
```

## Additional Resources

- [Python Packaging User Guide](https://packaging.python.org/)
- [PyPI Help](https://pypi.org/help/)
- [Twine Documentation](https://twine.readthedocs.io/)
