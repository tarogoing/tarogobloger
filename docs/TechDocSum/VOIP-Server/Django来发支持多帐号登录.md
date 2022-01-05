# Django来发支持多帐号登录
之前曾经写过一篇文章，讲在Django开发中如何整合新浪微博API。当时，新浪微博只支持OAuth1.0，现在已经支持2.0版本，OAuth2.0协议进行了简化，且access token将不能永久使用，它存在一个过期时间。本文讲解了如何在你的django站点中支持多帐号登录，主要包括Google、新浪微博、人人和腾讯微博帐号，其实就是这个博客目前所支持的第三方帐号登录。
在这些第三方帐号中，Google、新浪微博以及人人都已经支持了OAuth2.0，而腾讯微博仍然停留在1.0阶段。
对于OAuth2.0，以Google帐号为例（Google也支持OpenID方式，读者可以自己去实现）。
首先，我们在urls中添加登录以及处理回调的url：
```
urlpatterns += patterns('myapp.views',
    url(r'^accounts/google/login/$', 'google_login', name='social_google_login'),
    url(r'^accounts/google/login/done/$', 'google_auth', name='social_google_login_done'),
)
```
登录地址主要重定向到Google的授权页面，这在Django中可以直接使用django.views.generic.base.RedirectView这个通用视图来重定向，不过，由于我们要获取在授权完成后的重定向地址，也就是初始地址（比如说从/contact/点击登录的，最后完成授权后还要重定向到这个url），所以我们在自己定义的views中实现。
获取初始地址的代码在先前文章中也出现过，代码如下：
```
def _get_referer_url(request):
    referer_url = request.META.get('HTTP_REFERER', '/')
    host = request.META['HTTP_HOST']
    if referer_url.startswith('http') and host not in referer_url:
        referer_url = '/'
    return referer_url
接着定义google_login这个视图：
import urllib
 def google_login(request):
    google_auth_url = '%s?%s' % ('https://accounts.google.com/o/oauth2/auth',
                             urllib.urlencode({
                                 'response_type': 'code',
                                 'client_id': 'Your client id',
                                 'redirect_uri': 'Your redirect uri',
                                 'scope': 'https://www.googleapis.com/auth/userinfo.email https://www.googleapis.com/auth/userinfo.profile',
                                 'state': _get_referer_url(request)
                             }))
    return HttpResponseRedirect(google_auth_url)
```
这里的参数包括response_type，这个对于web应用必须是“code”，它在授权后会重定向到回调地址，并作为参数“code=××××”；client_id和redirect_uri是你在申请Google APIs时所得到或设定的（申请地址，需翻墙，这里要注意，在这个view中所设定的回调地址必须和申请API时所设定一致）；scope是指你要获得用户哪方面的权限，这里包括用户的基本信息以及用户email地址；state是个可选参数，如果设定，在重定向到回调地址时会作为参数传递，这里把初始地址作为state传递，在处理回调完成时就可以重定向到这个初始地址。
用户点击登录地址时，就会重定向到Google的授权页面，如果用户点击同意授权，就重定向到你所设定的地址，也就是由google_auth这个视图在处理。这个时候我们通过request.GET就拿到了code（授权码）和state（可选）这两个参数。拿到了授权码，我们就需要去获取用户的
```
access token：
import urllib2
 
def get_access_token(code):
    auth_url = 'https://accounts.google.com/o/oauth2/token'
    body = urllib.urlencode({
                'code': code, # 授权码
                'client_id': 'your client id',
                'client_secret': 'your client_secret',
                'redirect_uri': 'your redirect uri',
                'grant_type': 'authorization_code' # 必须是这个值
                })
    headers = {
        'Content-Type': 'application/x-www-form-urlencoded',
    }
    req = urllib2.Request(auth_url, body, headers)
    resp = urllib2.urlopen(req)
     
    data = json.loads(resp.read())
     
    return data['access_token']
```
这段代码非常简单，通过一系列的参数POST到Google access token endpoint，拿到用户的access token。接着我们就可以去取到用户的数据了：
```
def get_user_info(access_token):
    if access_token:
        userinfo_url = 'https://www.googleapis.com/oauth2/v1/userinfo'
        query_string = urllib.urlencode({'access_token': access_token})
         
        resp = urllib2.urlopen("%s?%s" % (userinfo_url, query_string))
        data = json.loads(resp.read())
         
        return data
```
通过data['name']，data['email']，data['picture']来分别获取用户的用户名、email地址、头像等等，详细的返回值可以参考这里。视图的代码如下：
```
def google_auth(request):
    if 'blog_user' in request.session:
        return HttpResponseRedirect('/')
    if 'error' in request.GET or 'code' not in request.GET:
        return HttpResponseRedirect('/')
    code = request.GET['code']
    access_token = get_access_token(code)
    blog_user = get_blog_user(get_user_info(access_token))
    request.session['blog_user'] = blog_user
    next = '/'
    if 'state' in request.GET:
        next = request.GET['state']
    return HttpResponseRedirect(next)
```
这里，需要说明的是，在Django中，你可以定义自己的Backend（要在settings中注册），这样就可以用通用的方式来进行认证、登录、登出等操作，详细参考Authentication文档。这里没有这么做，原因有二：
一是博客需要保存用户头像，而内置的django.contrib.auth.admin.User并没有这个字段，要么继承要么用一一对应关系的Model来定义，我也不需要这么复杂，只需要几个字段以字典的形式保存到request.session中即可；
二是得到的user我也不需要保存到数据库中，session超时或者用户自己退出删除session中的数据即可。
这样Google帐号的登录就完成了，可以看到用OAuth2.0还是很简单的。同样的，新浪微博和人人都支持2.0方式，他们登录的操作大同小异，不过有以下注意点。
新浪微博在登录的地址参数中没有state这个选项，这就意味着，用户授权回调的地址中只有code一个参数，所以，在weibo_login这个view中，我们需要把初始地址保存在session中：
request.session['redirect_uri'] = _get_referer_url(request)
另外，要注意，新浪微博的申请时设置回调地址的页面很坑爹。一个weibo应用包括两种：“应用”和“网站”。如果你都申请过，你就会发现“网站”中找不到设置回调地址的地方，但是“应用”页面就有，实际上，你需要手动输入地址：http://open.weibo.com/apps/<your app key>/info/advanced。并且回调地址是不能设置成localhost等等的，它始终会提示你地址不匹配。所以如果要Debug，那么改hosts文件吧……
新浪微博详细授权方法在这里。
人人网有个不同之处，就是它的参数除了一些必备参数，还有个是签名，签名就是把所有参数按key从小到大排序，末尾再加上app secret，再对这个字符串求MD5值。获得签名的代码如下：
```
import hashlib
 
def get_signature(params, secret):
    '''
    param params：字典
    param secret：你申请的app secret
    '''
    params_str = ''.join(['%s=%s'%(k, params[k]) for k in sorted(params)])
    return hashlib.md5("%s%s"%(params_str, secret)).hexdigest()
```
其他操作大同小异。
腾讯微博这方面就弱多了，官方sdk中不包含Python版本，甚至连推荐也没有，不过目前做得比较好的是根据Twitter python sdk改的SDK（地址，文档）。你可以从源码，或者用“easy_install -U pyqqweibo”安装。
方法大致和前一篇新浪微博OAuth1.0方法相同。qqweibo_login代码如下：
```
from qqweibo import OAuthHandler, API
def qqweibo_login(request):
    oauth_handler = OAuthHandler('your app key', 'your app_secret', 'redirect uri')
    qqweibo_auth_url = oauth_handler.get_authorization_url() # 得到授权地址
    request.session['redirect_uri'] = _get_referer_url(request) # 保存初始地址
    request.session['request_token'] = (oauth_handler.request_token.key, oauth_handler.request_token.secret)
    return HttpResponseRedirect(qqweibo_auth_url)
```
腾讯微博中不需要设定回调地址。接着是qqweibo_auth：
```
def qqweibo_auth(request):
    if 'oauth_verifier' not in request.GET:
        return HttpResponseRedirect('/')
     
    verifier = request.GET['oauth_verifier']
    request_token = request.session['request_token']
    del request.session['request_token']
     
    oauth_handler = OAuthHandler('your app key', 'your app_secret', 'redirect uri')
    oauth_handler.set_request_token(request_token[0], request_token[1])
    access_token = oauth_handler.get_access_token(verifier) # 得到access token
     
    blog_user = get_blog_user(access_token)
    request.session['blog_user'] = blog_user
     
    next = request.session['redirect_uri']
    del request.session['redirect_uri']
     
    return HttpResponseRedirect(next)
```
最后，如果要登出，直接在相应的view中删除session，并且重定向到来源地址。
好了，关于Django中集成多帐号登录就介绍这么多了，如果你想看完整代码，可以参看我的博客项目中social这个app中的代码。