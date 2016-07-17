### Step 3: Adding django-webpack-loader

First of all you need to run `pip install django-webpack-loader` and add it to `requirements.txt`.

Next you need to add this reusable Django app to the `INSTALLED_APP` setting in the settings.py:

```python
INSTALLED_APPS = [
    ...
    'webpack_loader',
]