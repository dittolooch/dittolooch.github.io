---
layout: post
title:
tags: [python, django]
---

## to start

django-admin startproject sitename
cd sitename && python manage.py migrate

# run test server
python manage.py runserver

python manage.py runserver 127.0.0.1:8001 --settings=mysite.settings


# settings.py

https://docs.djangoproject.com/en/2.0/ref/settings/.

## start app
python manage.py startapp appname

## models
- each model inherits from models.Model
- CharField for strings with optional max_length arg
- SlugField for slug (A slug is a short label that contains only letters, numbers, underscores, or hyphens.)

- ForeignKey, when it is a attribute in a model, it refers to a one to many relationship. In this case one foreign key (User) can have multiple Posts, and on_delete tells the database that when the User is deleted, the posts that User writes should be deleted too. The related_name arg defines the name of the reverse relationship i.e. Post to User so that we can refer to the User from Post easily

- Meta class in model defines the metadata of the model. ordering = ('-publish',) tells Django to sort the data by publish variable in descending order, therefore the minus in the front


```
class Post(models.Model):
    def __str__(self):
        return self.title
    STATUS_CHOICES = (
        ('draft','Draft'),
        ('published',"Published")
    )
    title = models.CharField(max_length = 250)
    slug = models.SlugField(max_length= 250, unique_for_date = 'publish')
    author = models.ForeignKey( User,
                                on_delete = models.CASCADE,
                                related_name = 'blog_posts')
    body = models.TextField()
    publish = models.DateTimeField(default = timezone.now)
    created = models.DateTimeField(auto_now_add = True)
    update = models.DateTimeField(auto_now = True)
    status = models.CharField(  choices = STATUS_CHOICES,
                                max_length = 10,
                                defaul = 'draft')
    class Meta:
        ordering = ('-publish',)

```


# adding app to the project
in mysite/settings.py add the app 'blog.apps.BlogConfig' in INSTALLED_APPS
then run
```python manage.py makemigrations blog```
check sql commands
```python manage.py sqlmigrate blog 0001```

then execute
```python manage.py migrate```


# create super user

python manage.py createsuperuser


# add model into admin site

app/admin.py
from .model import Post
admin.site.register(Post)

# customize how the model is shown in the admin sitename
```
@admin.register(Post)
class PostAdmin(admin.ModelAdmin):
    list_display = ('title', 'slug', 'author', 'publish','status')
    list_filter = ('status', 'created', 'publish', 'author')
    search_fields = ('title', 'body')
    prepopulated_fields = {'slug':('title',)}
    raw_id_fields = ('author',)
    data_hierarchy = 'publish'
    ordering = ('status','publish')
```

# interacting with models in django shell

python manage.py shell

# models api - create & save

from blog.models import Post
post = Post(title="a post")
post.save()

OR
Post.objects.create(title='a post')

# models api - get

Post.objects.get(filter) - > only return 1
Post.objects.all()

## models api - filter and exclude

Post.objects.filter(publish__year=2017)

queries with field lookup methods are built using 2 underscores
so publish.year -> publish__year
author.username -> author__username


Post.objects.filter(publish__year=2017).exclude(title___startswith='Why')

## models api - order_by

Post.objects.order_by('title')
Post.objects.order_by('-title') // descedning order

## models api - delete

post = Post.objects.get(id=1)
post.delete()
Note that deleting objects will also delete any dependent relationships for
ForeignKey objects defined with on_delete set to CASCADE.

## define a custom model manager

class PublishedManager(models.Manager):
    def get_queryset(self):
        return super(PublishedManager, self).get_queryset().filter(status='published')

## view

the render function takes the request object, a template path, and the context var to render the given template

def post_list(request):
    posts = Post.published.all()
    return render(request, 'blog/post/list.html', {'posts':posts})


## view or 404

def post_detail(request, year, month, day, post):
    post = get_object_or_404(
                                Post,
                                slug = post,
                                status = 'published',
                                publish__year = year,
                                publish__month = month,
                                publish__day = day
                            )
    return render(request, 'blog/post/detail.html', {'post':post})

## adding url patterns for the app
path function. first arg is the url, second func is the view rendering func, third is a name for the path


from django.urls import path
from .import views
app_name = 'blog'
urlpatterns = [
path('',views.post_list, name='post_list'),
path('<int:year>/<int:month>/<int:day>/<slug:post>/', views.post_detail, name='post_detail')
]

other path converters can be found at https://docs.djangoproject.com/en/2.0/topics/http/urls/#path-converters

## adding app urlpatterns to the main urlpattern

namespace is unique and will be used later to refer to the urlpatterns from the top down


from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('blog/', include('blog.urls',namespace='blog'))
]


## create absolute url for models with django.urls.reverse

reverse takes a urlpattern name as first arg, and if the urlpattern has arguments, use args = [] to fill in the args

class Post(models.Model):
  def get_absolute_url(self):
    return reverse("blog:post_detail",args = [1,2,3,4])


## templates

template structure
appname/templates/appanme/ooo.html


{% load static %} -> allow the app to use static tag and load static file
<!DOCTYPE html>
<html>
<head>
<title>{% block title %}{% endblock %}</title>
<link href="{% static "css/blog.css" %}" rel="stylesheet">
</head>
<body>
<div id="content">
{% block content %}
{% endblock %}
</div>
<div id="sidebar">
<h2>My blog</h2>
<p>This is my blog.</p>
</div>
</body>
</html>


# pagination in view function

def post_list(request):
    all_posts = Post.published.all()
    paginator = Paginator(all_posts, 3)
    page = request.GET.get('page')
    try:
        posts =  pagninator.page(page)
    except PageNotAnInteger:
        posts = paginator.page(1)
    except EmptyPage:
        posts = paginator.get(paginator.num_pages)

    return render(request, 'blog/post/list.html', {'posts':posts, 'page':page})


using the paginator in html

pagination.html

<div class="pagination">
<span class="step-links">
{% if page.has_previous %}
<a href="?page={{ page.previous_page_number }}">Previous</a>
{% endif %}
<span class="current">
Page {{ page.number }} of {{ page.paginator.num_pages }}.
</span>
{% if page.has_next %}
<a href="?page={{ page.next_page_number }}">Next</a>
{% endif %}
</span>
</div>

include pagination.html in list.html
pass the args after the with keyword
{% include "blog/pagination.html" with page=posts %}



# class based view

class PostListView(ListView):
    queryset = Post.published.all()
    context_object_name = 'posts'
    paginate_by = 3
    template_name = "blog/post/list.html"

urlpattern

path('', views.PostListView.as_view(), name='post_list'),

html

{% include "blog/pagination.html" with page=page_obj %}


# forms
https://docs.djangoproject.com/en/2.0/ref/forms/fields/

from django import forms
class EmailPostForm(forms.Form):
    name = forms.CharField(max_length=25)
    email = forms.EmailField()
    to = forms.EmailField()
    comments = forms.CharField(required=False, widget = forms.Textarea)


# sending Email
settings.py

EMAIL_HOST = 'smtp.gmail.com'
EMAIL_HOST_USER = 'warrenycheng@gmail.com'
EMAIL_HOST_PASSWORD = ''
EMAIL_PORT = 587
EMAIL_USE_TLS = True

def post_share(request, post_id):
    post = get_object_or_404(Post, id=post_id, status = 'published')
    sent = False
    if request.method == "POST":
        form = EmailPostForm(request.POST)
        if form.is_valid():
            cleaned_data = form.cleaned_data
            post_url = request.build_absolute_uri(post.get_absolute_url())

            subject = '{} ({}) recommends you reading "{}"'.format(
            cd['name'], cd['email'], post.title
            )
            message = 'Read "{}" at {}\n\n{}\'s comments: {}'.format(
            post.title, post_url, cd['name'], cd['comment']
            )
            send_mail(subject, message, 'warren@warrencheng.dev',[cd['to']])
            sent = True

            #...send email
    else:
        form = EmailPostForm()
    return render(request, 'blog/post/share.html', {'post':post, 'form':form})



## model form

We are using the Django ORM in the template, executing the
QuerySet comments.count(). Note that Django template language doesn't
use parentheses for calling methods. The {% with %} tag allows us to
assign a value to a new variable that will be available to be used
until the {% endwith %} tag.

{% with comments.count as total_comments %}
<h2>
{{ total_comments }} comment{{ total_comments|pluralize }}
</h2>
{% endwith %}
