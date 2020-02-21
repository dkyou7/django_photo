[toc]

# Dstagram

## django로 구현해보기 3

### 1. 기능

1. 사진 목록 보여주기 기능
   - 각 사진별로 작성자, 텍스트 설명, 댓글달기 버튼이 같이 출력된다.
2. 사진 생성, 사진 업데이트 기능
3. 사진 상세정보 확인 기능
   - 내부에서 댓글 기능
4. 사진 삭제 기능
5. 로그인 로그아웃 기능
6. 회원가입 기능

### 2. Project init script

```bash
pip install django
django-admin startproject config .
python manage.py migrate
python manage.py createsuperuser
python manage.py runserver
```

이제 웹 서버를 구동할 준비가 완료되었다.

#### 2.1 first commit

 - `.gitignore`

   ```pyth
   *.pyc
   *~
   /venv
   __pycache__
   db.sqlite3
   .DS_Store
   ```

### 3. 앱 만들기

#### 3.1 앱 만들기

1. python manage.py startapp photo

2. `config/settings.py` INSTALLED_APPS 변수에 추가

   ```python
   INSTALLED_APPS = [
       'photo',
       'django.contrib.admin',
       'django.contrib.auth',
       'django.contrib.contenttypes',
       'django.contrib.sessions',
       'django.contrib.messages',
       'django.contrib.staticfiles',
   ]
   ```

#### 3.2 모델 만들기

`photo/models/py`

```python
from django.db import models
from django.contrib.auth.models import User
from django.urls import reverse

# Create your models here.
class Photo(models.Model):
    author = models.ForeignKey(User,on_delete=models.CASCADE, related_name='user_photos')
    photo = models.ImageField(upload_to='photos/%Y/%m/%d',default='photos/no_image.png')
    text = models.TextField()
    created = models.DateTimeField(auto_now_add=True)
    updated = models.DateTimeField(auto_now=True)

    class Meta:
        ordering = ['-updated']

    def __str__(self):
        return self.author.username + " " + self.created.strftime("%Y-%m-%d %H:%M:%S")

    def get_absolute_url(self):
        return reverse('photo:photo_detail',args=[str(self.id)])
```

- ImageField 를 사용하기 위해 Pillow 모듈 설치하기

  > pip install pillow

- `makemigrations` 명령어를 사용해 모델의 변경 사항을 기록한다.

  > python manage.py makemigrations photo

- 기록한 변경사항을 DB에 적용하기 위해

  > python manage.py migrate photo 0001

#### 3.3 관리자 사이트에 모델 등록

- 관리자 사이트에 모델 등록 시 모델 관리 뷰를 만들기 전에 모델을 테스트 가능하다.

- `photo/admin.py`

  ```python
  from django.contrib import admin
  
  from .models import Photo
  
  admin.site.register(Photo)
  ```


#### 3.4 업로드 폴더 관리

- 미디어 파일 추가 : 보안

`config/settings.py`

```python
MEDIA_URL = '/media/'

MEDIA_ROOT = os.path.join(BASE_DIR,'media')
```

#### 3.5 관리자 페이지 커스터마이징

![image](https://user-images.githubusercontent.com/26649731/74912111-57070600-5401-11ea-991f-aa928fe88b8e.png)

`photo/admin.py`

```python
from django.contrib import admin

from .models import Photo

class PhotoAdmin(admin.ModelAdmin):
    list_display = ['id','author','created','updated']
    raw_id_fields = ['author']
    list_filter = ['created','updated','author']
    search_fields = ['text','created']
    ordering = ['-updated','-created']

admin.site.register(Photo,PhotoAdmin)
```

![image](https://user-images.githubusercontent.com/26649731/74913445-fdeca180-5403-11ea-8e63-2a4d8156abe7.png)

#### 3.6 뷰 만들기

```python
from django.shortcuts import render
from .models import Photo
from django.views.generic.edit import CreateView,DeleteView,UpdateView
from django.shortcuts import redirect

# Create your views here.
def photo_list(request):
    photos = Photo.objects.all()
    return render(request,'photo/list.html',{'photos':photos})

class PhotoUploadView(CreateView):
    model = Photo
    fields = ['photo','text']
    template_name = 'photo/upload.html'

    def form_valid(self, form):
        form.instance.author_id = self.request.user.id
        if form.is_valid():
            form.instance.save()
            return redirect('/')
        else:
            return self.render_to_response({'form':form})

class PhotoDeleteView(DeleteView):
    model = Photo
    success_url = '/'
    template_name = 'photo/delete.html'
    fields = '__all__'

class PhotoUpdateView(UpdateView):
    model = Photo
    success_url = '/'
    template_name = 'photo/update.html'
    fields = '__all__'
```

#### 3.7 URL 연결

- 뷰 동작을 위한 URL 연결

`photo/urls.py`

```python
from django.urls import path
from django.views.generic.detail import DetailView

from .views import *
from .models import Photo

app_name='photo'

urlpatterns=[
    path('',photo_list,name="photo_list"),
    path('detail/<int:pk>/',DetailView.as_view(model=Photo,template_name='photo/detail.html'),name='photo_detail'),
    path('upload/',PhotoUpdateView.as_view(),app_name='photo_upload'),
    path('delete/<int:pk>/',PhotoDeleteView.as_view(),app_name='photo_delete'),
    path('update/<int:pk>/', PhotoUpdateView.as_view(), app_name='photo_update'),
]
```

`config/urls.py`

```python
from django.contrib import admin
from django.urls import path,include

urlpatterns = [
    path('',include('photo.urls')),
    path('admin/', admin.site.urls),
]

from django.conf.urls.static import static
from django.conf import settings

urlpatterns += static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)

```

#### 3.8 템플릿 분리와 확장

- base template 만들기

  `templates/base.html`

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">

    <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.1.3/css/bootstrap.min.css" integrity="sha384-MCw98/SFnGE8fJT3GXwEOngsV7Zt27NXFoaoApmYm81iuXoPkFOJwJ8ERdknLPMO" crossorigin="anonymous">
    <script src="https://code.jquery.com/jquery-3.3.1.slim.min.js" integrity="sha384-q8i/X+965DzO0rT7abK41JStQIAqVgRVzpbzo5smXKp4YfRvH+8abtTE1Pi6jizo" crossorigin="anonymous"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/popper.js/1.14.3/umd/popper.min.js" integrity="sha384-ZMP7rVo3mIykV+2+9J3UJ46jBk0WLaUAdn689aCwoqbBJiSnjAK/l8WvCWPIPm49" crossorigin="anonymous"></script>
<script src="https://stackpath.bootstrapcdn.com/bootstrap/4.1.3/js/bootstrap.min.js" integrity="sha384-ChfqqxuZUCnJSK3+MXmPNIyE6ZbWh2IMqE241rYiqJxyMiZ6OW/JmZQ5stwEULTy" crossorigin="anonymous"></script>
    <title>Dstagram {% block title %}{% endblock %}</title>
</head>
<body>

<div class="container">
    <header class="header clearfix">
        <nav class="navbar navbar-expand-lg navbar-light bg-light">
            <a class="navbar-brand" href="/">Dstagram</a>
            <ul class="nav">
                <li class="nav-item"><a href="/" class="active nav-link ">Home</a></li>
                {% if user.is_authenticated %}
                <li class="nav-item"><a href="#" class="nav-link">Welcome, {{user.get_username}}</a></li>
                <li class="nav-item"><a href="{% url 'photo:photo_upload' %}" class="nav-link">Upload</a></li>
                <li class="nav-item"><a href="#" class="nav-link">Logout</a></li>
                {% else %}
                <li class="nav-item"><a href="#" class="nav-link">Login</a></li>
                <li class="nav-item"><a href="#" class="nav-link">Signup</a></li>
                {% endif %}
            </ul>
        </nav>
    </header>
    {% block content %}
    {% endblock %}

    <footer class="footer">
        <p>&copy; 2018 Baepeu. Powered By Django 2</p>
    </footer>
</div>


</body>
</html>
```

- `config/settings.py`

```python
TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [os.path.join(BASE_DIR,"templates")],
        ....
```

- `photo/templates/photo/list.html`

```html
{% extends 'base.html' %}

{% block title %}- List{% endblock %}

{% block content %}
    {% for post in photos %}
        <div class="row">
            <div class="col-md-2"></div>
            <div class="col-md-8 panel panel-default">
                <p><img src="{{post.photo.url}}" style="width:100%;"></p>
                <button type="button" class="btn btn-xs btn-info">
                    {{post.author.username}}</button>
                <p>{{post.text|linebreaksbr}}</p>
                <p class="text-right">
                    <a href="{% url 'photo:photo_detail' pk=post.id %}" class="btn btn-xs btn-success">댓글달기</a>
                </p>
            </div>
            <div class="col-md-2"></div>
        </div>

    {% endfor %}
{% endblock %}
```

- `photo/templates/photo/upload.html`

```python
{% extends 'base.html' %}
{% block title %}
- Upload
{% endblock %}

{% block content %}
<div class="row">
    <div class="col-md-2"></div>
    <div class="col-md-8 panel panel-default">
        <form action="" method="post" enctype="multipart/form-data">
            {{form.as_p}}
            {% csrf_token %}
            <input type="submit" class="btn btn-primary" value="Upload">
        </form>
    </div>
    <div class="col-md-2"></div>
</div>
{% endblock %}
```

- `photo/templates/photo/detail.html`

```python
{% extends 'base.html' %}
{% block title %}
    {{object.text|truncatechars:10}}
{% endblock %}

{% block content %}
    <div class="row">
            <div class="col-md-2"></div>
            <div class="col-md-8 panel panel-default">
                <p><img src="{{object.photo.url}}" style="width:100%;"></p>
                <button type="button" class="btn btn-outline-primary btn-sm">
                    {{object.author.username}}</button>
                <p>{{object.text|linebreaksbr}}</p>

                <a href="{% url 'photo:photo_delete' pk=object.id %}" class="btn btn-outline-danger btn-sm float-right">
                    Delete</a>
                <a href="{% url 'photo:photo_update' pk=object.id %}" class="btn btn-outline-success btn-sm float-right">
                    Update</a>
            </div>
            <div class="col-md-2"></div>
    </div>
{% endblock %}
```

- `photo/templates/photo/update.html`

```python
{% extends 'base.html' %}
{% block title %}
- Upate
{% endblock %}

{% block content %}
<div class="row">
    <div class="col-md-2"></div>
    <div class="col-md-8 panel panel-default">
        <form action="" method="post" enctype="multipart/form-data">
            {{form.as_p}}
            {% csrf_token %}
            <input type="submit" class="btn btn-primary" value="Update">
        </form>
    </div>
    <div class="col-md-2"></div>
</div>
{% endblock %}
```

- `photo/templates/photo/delete.html`

```python
{% extends 'base.html' %}
{% block title %}- Delete{% endblock %}

{% block content %}
<div class="row">
    <div class="col-md-2"></div>
    <div class="col-md-8 panel panel-default">
        <div class="alert alert-info">
            Do you want to delete {{object}}?
        </div>
        <form action="" method="post">
            {{form.as_p}}
            {% csrf_token %}
            <input type="submit" class="btn btn-danger" value="Confirm">
        </form>
    </div>
    <div class="col-md-2"></div>
</div>
{% endblock %}
```



#### 3.9 사진 표시하기

