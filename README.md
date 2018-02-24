# django-multiple-files
Simple project to upload multiple files with a Django form, WITHOUT JavaScript or Ajax.
___

Demonstrating how to link a model to a form, and how to return a human-readable filename whenever a model is printed:
```
### models.py
from django.db import models
import os

class fileentry(models.Model):
    file = models.FileField()
    uploaddate = models.DateTimeField(auto_now_add=True)

    def __str__(self):
        return os.path.basename(self.file.name)
```
This code is mostly unused; I haven't found a way to save individual form files from `request.FILES` yet. But having the code here demonstrates how to link a model to a form.
___

Enabling the `multiple` attribute in the form input, without having to manually display the form in the template:
```
### forms.py
from django import forms
from .models import fileentry

class fileform(forms.ModelForm):
    label = ''                      ### This hides the label next to the "Browse..." button
    class Meta:
        model = fileentry           ### Specify the model to connect to
        fields = ('file', )         ### This trailing comma is neccessary
        widgets = {
            'file':                 ### This is what allows you to select multiple files
                forms.ClearableFileInput(attrs={'multiple': True})}     
```
___

The trick here is looping over `request.FILES.getlist('file')`. Trying to loop over `request.FILES` only returns the last file.
```
### views.py
from django.shortcuts import render
from .forms import fileform

def upload(request):

    context = {'form': None, 'last': None}

    if request.method == 'POST':
        form = fileform(request.POST, request.FILES)
        if form.is_valid():                        ### Display a list of the last files uploaded
            context['last'] = '\n'.join([f.name for f in request.FILES.getlist('file')])
            
            for file in request.FILES.getlist('file'):
                print(file.name)
                with open("uploads/{}".format(file.name), 'wb+') as f:
                    for chunk in file.chunks():    ### Write in chunks; large files won't use a lot of RAM
                        f.write(chunk)
                  f.close()
                  
    else:
        form = fileform()

    context['form'] = form
    return render(request, 'upload/index.html', context)
```
___

What is important here is setting `enctype="multipart/form-data"`.
```
### templates/upload/index.html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Upload</title>
</head>
<body>

<p><strong><span>Files:</span></strong></p>
<form method="post" enctype="multipart/form-data">
    {% csrf_token %}
    {{ form.file.errors }}
    {{ form.file }}
    <button type="submit">Upload</button>
</form>

<p><strong><span>Last upload:</span></strong></p>
<pre>{{ last }}</pre>

</body>
</html>
```
