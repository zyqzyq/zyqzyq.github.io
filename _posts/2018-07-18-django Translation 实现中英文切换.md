---
author: zyqzyq
date: 2018-07-18 10:16:32+00:00
layout: post
title: django Translation 实现中英文切换
key: 20180718
tags:
- django
---

## 环境
    django 2.0.6 (之前文章有详细项目新建的描述)

## 配置

### setting.py

###### 配置 MIDDLEWARE
    
```
MIDDLEWARE = [
   'django.contrib.sessions.middleware.SessionMiddleware',
   'django.middleware.locale.LocaleMiddleware',
   'django.middleware.common.CommonMiddleware',
]
```

注意：LocaleMiddleware 需要放在SessionMiddleware之后，CommonMiddleware之前。
[详细介绍](https://docs.djangoproject.com/zh-hans/2.0/topics/i18n/translation/#how-django-discovers-language-preference)
###### 配置template

```
TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [os.path.join(BASE_DIR, 'templates')],
        'APP_DIRS': True,
        'OPTIONS': {
            'context_processors': [
                'django.template.context_processors.debug',
                'django.template.context_processors.request',
                'django.contrib.auth.context_processors.auth',
                'django.contrib.messages.context_processors.messages',
                'django.template.context_processors.i18n',
            ],
        },
    },
]
```

###### 启用i18n


```
USE_I18N = True

USE_L10N = True

USE_TZ = True

# 翻译文件所在目录，需手工创建
LOCALE_PATHS = [
    os.path.join(BASE_DIR, 'locale'),
]

LANGUAGE_CODE = 'zh-hans'

LANGUAGES = {
    ('zh-hans', '中文简体'),
    ('en', 'English'),
}
```
注意：如果local文件夹在app目录下无需创建LOCALE_PATHS变量。[详细介绍](https://docs.djangoproject.com/zh-hans/2.0/topics/i18n/translation/#how-django-discovers-translations)

LANGUAGES 中可以添加多个django支持的语言（这边以中英文为例）

### yourproject/urls.py

```
from django.conf.urls.i18n import i18n_patterns
urlpatterns = [
    path('i18n/', include('django.conf.urls.i18n')),
]

urlpatterns += i18n_patterns(
    path('', include('yourapp.urls')),
)
```



## 声明需要翻译的字符串

##### 准备工作
需要提前安装gettext 
在ubuntu下运行

```
sudo apt-get install gettext
```
即可。

### 在python代码中

在此示例中，文本“欢迎访问我的网站”。 被标记为翻译字符串：
```
from django.http import HttpResponse
from django.utils.translation import gettext as _

def my_view(request):
    output = _("欢迎访问我的网站")
    return HttpResponse(output)
```

[更多介绍](https://docs.djangoproject.com/zh-hans/2.0/topics/i18n/translation/#internationalization-in-python-code)

### 在template代码中

需要将{%load i18n%}
放到模板文件头部（继承的文件也需要假如该声明）

{%trans%}模板标记转换为常量字符串（用单引号或双引号括起来）或变量内容：





```python
<title>{% trans "This is the title." %}</title>
<title>{% trans myvar %}</title>
```

如果您的翻译需要带变量的字符串（占位符），请改用{％blocktrans％}。

```
{% blocktrans %}This string will have {{ value }} inside.{% endblocktrans %}
```
如果您想要检索已翻译的字符串而不显示它，可以使用以下语法：

```
{% trans "This is the title" as the_title %}

<title>{{ the_title }}</title>
<meta name="description" content="{{ the_title }}">
```

[更多介绍](https://docs.djangoproject.com/zh-hans/2.0/topics/i18n/translation/#internationalization-in-template-code)

## 创建语言文件

要创建或更新消息文件，请运行以下命令：

```
django-admin makemessages -l en

```
en 是你想创建的文件的语言。

注意：该脚本应该从两个地方之一运行（在运行前需要新建locale文件夹）：

    Django项目的根目录（包含manage.py的目录）。
    您的一个Django应用程序的根目录。
    
运行完命令后会创建一个.po文件
编辑 yourproject/locale/en/LC_MESSAGES/django.po

```
#: path/to/python/module.py:23
msgid "欢迎访问我的网站。 "
msgstr "Welcome to my site."
```
msgid 是标记的想要翻译的字符串
msgstr 是你想要翻译后显示的字符串，默认为空。

[更多介绍](https://docs.djangoproject.com/zh-hans/2.0/topics/i18n/translation/#localization-how-to-create-language-files)

### 编译语言文件

运行以下代码

```
django-admin compilemessages
```
大功告成。

[详细介绍](https://docs.djangoproject.com/zh-hans/2.0/topics/i18n/translation/#compiling-message-files)

## 设置语言

在模板中添加以下代码

```
{% load i18n %}

<form action="{% url 'set_language' %}" method="post">{% csrf_token %}
    <input name="next" type="hidden" value="{{ redirect_to }}" />
    <select name="language">
        {% get_current_language as LANGUAGE_CODE %}
        {% get_available_languages as LANGUAGES %}
        {% get_language_info_list for LANGUAGES as languages %}
        {% for language in languages %}
            <option value="{{ language.code }}"{% if language.code == LANGUAGE_CODE %} selected{% endif %}>
                {{ language.name_local }} ({{ language.code }})
            </option>
        {% endfor %}
    </select>
    <input type="submit" value="Go" />
</form>
```
实现语言选择功能。
[更多介绍](https://docs.djangoproject.com/zh-hans/2.0/topics/i18n/translation/#the-set-language-redirect-view)



以上是简单的实现中英文切换的步骤，如需了解更多，可以点击更多介绍进入官网进行了解。


