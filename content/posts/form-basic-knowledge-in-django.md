---
categories:
  - code
date: 2010-05-08 14:17:59
tags:
  - python
  - django
title: Django中表单基本知识
---


##### Form基本物件
```python
from django import forms
class ContactForm(forms.Form):
    subject = forms.CharField(max_lenth=100)
    sender = forms.EmailField()
```
##### 在View中使用Form
```python
from django.shortcuts import render
from django.http import HttpResponseRedirect
from django.core.urlresolvers import reverse

def contact(request):
    if request.method == 'POST':
        from = ContactForm(request.POST)
        if form.is_valid():
            subject = form.cleaned_data['subject']
            sender = form.cleaned_data['sender']
            #according this data to change db or other logic
            url = reverse('thanks')
            return HttpResponseRedirect(url)
    else:
        form = ContactForm()
        return render(request,'template.html',{'form':form,})
```

##### Formsets
有时我们需要在一个页面生成多个FORM，此时就可以使用Formsets了
```python
from djano.forms.formsets import formset_factory
ContactFormSet = formset_factory(ContactForm,extra=2)
for form in ContactFormSet():
    print form.as_p()
```

##### 根据Model生成Form
Django是一个数据驱动的框架，因此大多数时候这个表单需要填写的数据可能跟模型是一样的，为了避免重复，我们可以根据模型来生成一个表单
```python
from django.forms import ModelForm
class ContactFromModeForm(ModelForm):
    class Meta:
        model = ModelName
        fields = ['subject','sender']

form = ContactFromModeForm()
print form.as_table()

onemodel = ModelName.objects.get(pk=1)
form = ContactFromModeForm(instance=onemodel)
```
模型中对应的Field在表单的Field都有对应，如ManyToManyField的对应类型是MultipleChoiceField

##### 表单验证
```python
if form.is_valid():
```
上面是表单验证是否出错的语句，不过我们可以重写表单中的clean_attname系列方法
```python
def clean_subject(self):          
    subject = self.cleaned_data['subject']         
    num_words = len(subject.split())         
    if num_words < 4:             
        raise forms.ValidationError("Not enough words!")         
    return subject
```
需要注意的是我们必须显示返回subject，否则会带来表单数据丢失。