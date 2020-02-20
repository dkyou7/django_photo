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

#### 3.5 관리자 페이지 커스터마이징

#### 3.6 뷰 만들기

#### 3.7 URL 연결

#### 3.8 템플릿 분리와 확장

#### 3.9 사진 표시하기

