---
layout: post
title:  "Django学习笔记"
date:   2014-10-26
categories: jekyll update
---

### 检测上传文件的大小
forms.py

{% highlight python %}
from django.forms import ModelForm, ValidationError
from django.forms.widgets import Input, Select, Textarea, FileInput
from .models import Feedback


class FeedbackForm(ModelForm):
    class Meta:
        model = Feedback
        fields = ['email', 'category', 'message', 'file']
        # 添加属性
        widgets = {
            'email': Input(attrs={'class': 'input-text'}),
            'category': Select(attrs={'class': 'sova-select'}),
            'message': Textarea(attrs={'class': 'form-control', 'rows': 3}),
            'file': FileInput(),
        }

    # 凡是以clean_开头，后接字段名的都会自动检查，记得return xxx
    def clean_file(self):
        file = self.cleaned_data.get('file', False)
        if file:
            if file._size > 2*1024*1024:
                raise ValidationError(u"文件太大（>2MB）")
            return file
        else:
            raise forms.ValidationError(u'请选择需要上传的图片')
{% endhighlight %}

### xadmin中添加action和inline
adminx.py

{% highlight python %}
from django.utils.translation import ugettext as _
from xadmin.plugins.actions import BaseActionView
import xadmin


class PublishMovieAction(BaseActionView):
    '''改为发布'''
    action_name = 'publish_movie'
    description = _(u'改为发布状态')    # 后台显示
    model_perm = 'change'   # 权限

    def do_action(self, queryset):
        # queryset.update(status='published') 等同下面
        for obj in queryset:
            obj.status = 'published'
            obj.save()
        return None


class IntroductionInline(object):
    model = Introduction    # model name, need fk point to Movie model
    max_num = 1
    extra = 1


class MovieAdmin(object):
    list_display = ('name', )
    search_fields = ('name', )
    inlines = [IntroductionInline, ]
    actions = [PublishMovieAction, ]

xadmin.site.register(Movie, MovieAdmin)
{% endhighlight %}


### DetailView, ListView, FormView 示例

views.py

{% highlight python %}
from django.views.generic import DetailView, ListView, FormView
from django.shortcuts import get_object_or_404


class StaticPageView(DetailView):
    '''静态页面'''
    model = StaticPage

    def get_object(self):
        obj = get_object_or_404(StaticPage, url=self.kwargs.get('url'))
        self.template_name = '{0}.html'.format(self.kwargs.get('url'))
        return obj


class TeamMemberView(ListView):
    '''团队成员'''
    model = TeamMember
    template_name = 'team.html'


class FeedbackView(FormView):
    '''意见反馈'''
    model = Feedback
    form_class = FeedbackForm   # from .forms import FeedbackForm
    template_name = 'feedback.html'
    success_url = '/feedback/'

    def form_valid(self, form):
        form.save(commit=True)
        return super(FeedbackView, self).form_valid(form)
{% endhighlight %}


### xadmin一些配置
xadmin.py

{% highlight python %}
import xadmin


# Xadmin settings start
class MainDashboard(object):
    widgets = [
        [
            {
                "type": "html",
                "title": "后台首页",
                "content": "<h3> 欢迎登陆盛万投电影后台。</h3>"
            },
        ]
    ]


class GolbeSetting(object):
    site_title = u"盛万投电影管理后台"

xadmin.site.register(xadmin.views.IndexView, MainDashboard)
xadmin.site.register(xadmin.views.CommAdminView, GolbeSetting)
# Xadmin settings end
{% endhighlight %}


### reverse()，url tag

{% highlight python %}
from django.core.urlresolvers import reverse
from django.http import HttpResponseRedirect
HttpResponseRedirect(reverse('movie:detial', kwargs={'pk': m.pk}))

\{\% url 'movie:detial' pk=m.pk \%\}    #m is a Movie instance
{% endhighlight %}


### siteuser的登录检查，获取微信openid和登录之后跳转回原来的网页

{% highlight python %}
# -*- coding: utf-8 -*-
from functools import wraps

from django.http import HttpResponseRedirect
from django.http import Http404
from django.core.urlresolvers import reverse
import requests
from sova.settings import APPID, APPSECRET


code_url = 'https://api.weixin.qq.com/sns/oauth2/access_token?appid=%s&secret=%s&code=%s&grant_type=authorization_code'    # 获取access_token,与基础支持中的access_token不同

def login_needed(func, login_url=None):
    @wraps(func)
    def wrap(request, *args, **kwargs):
        if not request.siteuser:
            # No login
            if request.GET.get('is_wechat', None) == 'true':    # 微信链接里加入 ?is_wechat=true
                code = request.GET.get('code', None)
                if code:
                    rsp = requests.get(code_url % (APPID, APPSECRET, code), verify=False)
                    rsp_json = rsp.json()
                    openid = rsp_json.get('openid', None)
                    if openid:
                        request.session['openid'] = openid
                if request.path == reverse('investment:list'):
                    return HttpResponseRedirect(reverse('movie:movie_share', kwargs={'pk': 4}))
            login_url = reverse('profiles:siteuser_login')
            if login_url:
                # 如果不是在登录页面进入登录页面，在登录页面url加入 ?next=request.path, 在登录view里判断跳转
                if request.path != reverse('profiles:siteuser_login'):
                    login_url += '?next=%s' % request.path
                return HttpResponseRedirect(login_url)
            raise Http404

        return func(request, *args, **kwargs)
    return wrap
{% endhighlight %}


### 自定义filter
templatetags.divide.py

{% highlight python %}
#!/usr/bin/env python
#coding:utf-8

from django import template

register = template.Library()

@register.filter
def divide(a, b):
	if int(a) % int(b) == 0:
		return True
	else:
		return False
{% endhighlight %}

template/test.html

{% highlight python %}
\{\% load divide \%\}
...
\{\% if forloop.counter|divide:'3' \%\}
...
{% endhighlight %}