---
big-title: "Common Web application tools"
middle-title: "Authentication"
small-title: "Using the authentication system"
author: 
  - 김선규(Kim,seon-kyu)
field:
  - Authentication
relate:
  - Authentication
toc: true
toc-head-level-choice: false
#do this if head level choice is true
# toc-head-max:
# toc-head-min:
---

# Using the Django authentication system

> 미완성 상태

이 문서는 기본 configuration으로 Django의 authentication을 사용하는 방법을 설명합니다. 이 configuration은  가장 일반적인 프로젝트 요구를 충족하도록 발전했으며, 상당히 광범위한 작업들을 처리할 수 있습니다. 그리고 password와 permissions를 신중하게 구현했습니다. 기본값과 다른 authentication이 필요한 프로젝트에서, Django는 authentication에 대한 광범위한 `extension and customization`를 지원합니다.

Django authentication은 authentication과 authorization을 동시에 지원하고, 이 기능들이 같이 쌍을 이루기 때문에 일반적으로 authentication system이라 불립니다.

# User objects

User objects는 authentication system의 핵심입니다. 이들은 일반적으로 너의 사이트와 상호작용 하는 사람들을 나타내고 접근 제한, 사용자 프로필 등록, 콘텐츠를 제작자와 연결 등과 같은 작업들을 가능하게 합니다. Django authentication framework에는 단 한 개의 class가 존재합니다. 즉, **`'supersuers'`** 또는 admin **`'staff'`** users는 다른 클래스의 user objects가 아니라 특별한 attributes sets를 갖고 있는 user objects입니다.

default user의 주요 attribute는 다음과 같습니다:

- username
- password
- email
- first_name
- last_name

전체 reference를 위해서 **`full API documentation`**을 살펴보십시오. 이후 문서는 조금 더 작업 지향적입니다.

## Creating users

users를 생성하는 가장 직접적인 방법은 포함된 **`create_user()`** helper function을 사용하는 것입니다:

    >>> from django.contrib.auth.models import User
    >>> user = User.objects.create_user('john', 'lennon@thebeatles.com', 'johnpassword')
    
    # At this point, user is a User object that has already been saved
    # to the database. You can continue to change its attributes
    # if you want to change other fields.
    >>> user.last_name = 'Lennon'
    >>> user.save()

만약 당신이 Django admin을 설치했다면, 당신은 또한 `users를 interactively`하게 생성할 수 있습니다.

---

## Creating superusers

**`createsuperusers`** 명령어를 사용하여 superusers를 생성하십시오:

    $ python manage.py createsuperuser --username=joe --email=joe@example.com

password를 입력하라는 메세지가 나타날 것입니다. 입력을 한 후, user는 즉시 생성될 것입니다. 만약 당신이 **`—username`** 또는 **`—email`** option을 비워둔다면, 그것은 당신에게 그 값들을 요구할 것입니다.

---

## Changing passwords

Django는 raw password(clear text)를 user model에 저장하지 않을 겁니다, 하지만 단지 hash를 저장합니다(자세한 내용을 보기 위해서는 `documentation of how passwords are managed`를 보십시오). 이 때문에, user의 password attribute를 직접적으로 조정하려 하지 마십시오.

사용자의 비밀번호를 변경하기 위해 다음 몇 가지 option을 사용하십시오:

**`manage.py changepassword *username*`**은 명령어로부터 사용자의 비밀번호를 바꿀 수 있는 method를 제공합니다. 이는 당신에게 주어진 사용자의 비밀번호를 바꾸라고 말할 것입니다. 그리고 당신은 두 번 입력해야 합니다. 만약 두 개가 일치한다면, 새로운 비밀번호는 바로 바뀔 것입니다. 만약 당신이 사용자을 입력하지 않는다면, 명령어는 현재 시스템 사용자와 일치하는 사용자 이름의 p비밀번호를 바꾸려 시도할 것입니다.

당신은 또한 프로그램적으로 비밀번호를 바꿀 수 있습니다. `**set_password()**`를 사용하십시오:

    >>> from django.contrib.auth.models import User
    >>> u = User.objects.get(username='john')
    >>> u.set_password('new password')
    >>> u.save()

만약 Django admin을 설치했다면, 당신은 또한 사용자의 비밀번호를 `authentication system's admin pages`에서 바꿀 수 있습니다.

Django는 또한 사용자들에게 그들의 비밀번호를 바꿀 수 있게 `views`와 `forms`를 제공합니다.

사용자의 비밀번호를 바꾸고 나서 모든 세션에서 로그아웃 될 것입니다. 자세한 사항은 `Session invalidation on password change`를 보십시오.

---

## Authenticating users

**authenticate(*request=None, **credentials*)**

credential(자격 증명)을 확인하기 위해 **`authenticate()`**를 사용하십시오. 이것은 keyword arguments로 credentials, 기본 값들로 **username**과 **password**를 받아들이며, 각각의 대응하는 `authentication backend` 확인하고, 만약 credentials가 backend에 유효하면 **`User`** object를 반환합니다. 만약 credentials가 어떠한 backend에도 유효하지 않거나 backend가 **`PermissionDenied`**를 일으키면, 이 함수는 **None**을 반환합니다. 예를 보십시오:

    from django.contrib.auth import authenticate
    user = authenticate(username='john', password='secret')
    if user is not None:
        # A backend authenticated the credentials
    else:
        # No backend authenticated the credentials

**request**는 optional **`HttpRequest`**로, authentication backends의 **`authenticate()`** method에 전달됩니다.

- **Note**

    이는 credentials set을 authenticate하는 low level입니다; 예를 들어, 이는 **`RemoteUserMiddleware`**에 의해 사용됩니다. 당신이 당신 고유의 authentication system을 만들지 않는 이상, 당신은 이를 사용하지 않을 것입니다. 만약 당신이 User login 방법을 찾고 있다면, **`LoginView`**를 사용하십시오.

---

# Permissions and Authorization

Django는 간단한 permissions system을 갖고 있습니다. Django는 특정 users와 groups of users에 permissions를 부여하는 방법을 제공합니다.

이는 Django admin site에서 사용됩니다. 그러나 당신은 이를 당신의 코드에서 사용할 수 있습니다.

Django admin site는 다음과 같은 permissions를 사용합니다:

- objects를 볼 수 있는 access는 해당 object 유형에 대한 "view" 또는 "change" permission을 갖고 있는 users로 제한됩니다.
- "add" form을 보고 object를 추가하는 access는 해당 object 유형에 대한 "add" permission을 갖고 있는 users로 제한됩니다.
- change list를 보고, "change" form을 보고 object를 변경할 수 있는 access는 해당 object 유형에 대한 "change" permission을 갖고 있는 users로 제한됩니다.
- object 삭제에 대한 access는 해당 object 유형에 대한 users로 제합됩니다.

Permissions는 특정 유형의 object에만 적용될 수 있는 것이 아니라 특정한 object instance에도 설정할 수 있습니다. **`ModelAdmin`** class에서 제공되는 **`has_view_permission()`**, **`has_add_permission()`**, **`has_change_permission()`** and **`has_delete_permission()`** methods를 사용함으로써, 같은 유형의 다른 object instances에 대하여 permissions를 customize할 수 있습니다.

**`User`** objects는 두 가지 many-to-many fields를 갖고 있습니다: **groups**와 **user_permissions**입니다. **`User`** objects는 다른 `Django model`과 마찬가지로 related object에 접근할 수 있습니다:

    myuser.groups.set([group_list])
    myuser.groups.add(group, group, ...)
    myuser.groups.remove(group, group, ...)
    myuser.groups.clear()
    myuser.user_permissions.set([permission_list])
    myuser.user_permissions.add(permission, permission, ...)
    myuser.user_permissions.remove(permission, permission, ...)
    myuser.user_permissions.clear()

## Default permissions

**django.contrib.auth**가 당신의 **`INSTALLED_APPS`** setting에 있다면, 이는 네 가지 기본 permissions - add(추가), change(변경), delete(삭제), and view(보기) - 가 installed applications 안에 정의된 각각의 Django model에 대하여 생성될 것입니다.

이와 같은 permissions는 당신이 **`manage.py migrate`**를 실행할 때 생성될 것입니다; **`INSTALLED_APPS`**에 **django.contrib.auth**를 추가한 뒤에 처음으로 **migrate**를 실행시킬 때, 기본 permissions들은 기존에 installed 된 모든 models에 대하여 생성될 것입니다. 이후에 기본 permissions들은 당신이 **`manage.py migrate`**를 실행시킬 때마다 새로운 modles에 생성될 것입니다(permissions를 생성하는 function들은 **`post_migrate`** signal과 연결됩니다).

당신이 사용해야 하는 기본적인 permissions를 테스트하기 위해, **`app_label`**가 **foo**인 application과 이름이 **Bar**인 model을 갖고 있다고 가정합시다:

- add: **user.has_perm('foo.add_bar')**
- change: **user.has_perm('foo.change_bar')**
- delete: **user.has_perm('foo.delete_bar')**
- view: **user.has_perm('foo.view_bar')**

**`Permission`** model은 직접적으로 accessed 되지 않습니다.

---

## Groups

**`django.contrib.auth.models.Group`** models는 users를 categorize하는 generic한 방법으로 당신은 permissions, 또는 다른 label을 그들 users에게 적용할 수 있습니다. 한 user는 많은 수의 groups에도 속할 수 있습니다.

group에 있는 한 user는 자동적으로 해당 group에 주어진 permission을 갖습니다. 예를 들어, 만약 **Site editors** group이 **can_edit_home_page** permission을 갖고 있다면, 그 group에 있는 모든 user는 그 permission을 갖게 될 것입니다.

permissions 이외에도, groups는 users에게 몇몇 label이나 확장된 functionality를 주기 위해 그들을 categorize하는 편리한 방법입니다. 예를 들어, 당신은 'Special users' group을 만들 수 있고, 그들에게 당신의 site에서 members와 관련된 부분의 access를 부여하거나 members 에게 이메일을 보낼 수 있게 코드를 작성할 수 있습니다.

---

## Programmatically creating permissions

`custom permissions`는 model의 **Meta** class 내에서 정의될 수 있지만, 당신은 또한 직접적으로 permissions를 생성할 수 있습니다. 예를 들어, 당신은 **can_publish** permission을 **myapp**에 있는 **BlogPost** model에 줄 수 있습니다:

    from myapp.models import BlogPost
    from django.contrib.auth.models import Permission
    from django.contrib.contenttypes.models import ContentType
    
    content_type = ContentType.objects.get_for_model(BlogPost)
    permission = Permission.objects.create(
        codename='can_publish',
        name='Can Publish Posts',
        content_type=content_type,
    )

permission은 **`User`**의 **user_permissions** attribute을 통해 **`User`**에게 부여할 수 있고, **`Group`**의 **permissions** attribute를 사용하여 **`Group`**에게 부여할 수 있습니다.

---

## Permission caching

**`ModelBackend`**는 permissions check를 위해 permissions가 사용되는 첫 번째 이후로 user object에 대한 permissions를 cache합니다. 이는 일반적으로 request-response cycle에서는 괜찮은데 왜냐하면 permissions는 그들이 추가된 이후로 즉시 check되지 않기 때문입니다(예를 들어 admin에서). 만약 당신이 permissions를 추가하고 그들을 바로 checking한다면, 예를 들어 test나 veiw에서, 가장 쉬운 방법은 database로부터 user를 새로 가져오는 것입니다. 예를 보십시오:

    from django.contrib.auth.models import Permission, User
    from django.contrib.contenttypes.models import ContentType
    from django.shortcuts import get_object_or_404
    
    from myapp.models import BlogPost
    
    def user_gains_perms(request, user_id):
        user = get_object_or_404(User, pk=user_id)
        # any permission check will cache the current set of permissions
        user.has_perm('myapp.change_blogpost')
    
        content_type = ContentType.objects.get_for_model(BlogPost)
        permission = Permission.objects.get(
            codename='change_blogpost',
            content_type=content_type,
        )
        user.user_permissions.add(permission)
    
        # Checking the cached permission set
        user.has_perm('myapp.change_blogpost')  # False
    
        # Request new instance of User
        # Be aware that user.refresh_from_db() won't clear the cache.
        user = get_object_or_404(User, pk=user_id)
    
        # Permission cache is repopulated from the database
        user.has_perm('myapp.change_blogpost')  # True
    
        ...

---

# Authentication in Web requests

Django는 `sessions`와 middleware를 사용하여 authentication system을 **request obejcts**에 연결시킵니다.

이들은 모든 request에 대하여 **`request.user`**를 제공하는데, 현재 user를 나타냅니다. 만약 현재 user가 log in 하지 않았다면, 이 attribute는 **`AnonymousUser`**의 instance에 설정될 것이고 그렇지 않으면 **`User`** instance에 설정될 것입니다.

당신은 그들을 is_authenticated를 다음과 같이 사용하여 분리할 수 있습니다:

    if request.user.is_authenticated:
        # Do something for authenticated users.
        ...
    else:
        # Do something for anonymous users.
        ...

## How to log a user in

현재 session에 연결할 authenticated user를 갖고 있다면, 이는 login() function으로 할 수 있습니다.

**login(*request, user, backend=None*)**

view에서 user를 log in 시키기 위해서는, **`login()`**을 사용합니다. 이는 **`HttpRequest`**와 **`User`** object를 인수로 받아들입니다. **`login()`**은 Django의 session framework를 사용하여 session에 user의 ID를 저장합니다.

anonymous session에 있는 모든 data set은 user가 log in 한 후에도 유지된다는 점을 유의하십시오.

이 예시는 **`authenticate()`**와 **`login()`**을 어떻게 사용하는지 보여줍니다:

    from django.contrib.auth import authenticate, login
    
    def my_view(request):
        username = request.POST['username']
        password = request.POST['password']
        user = authenticate(request, username=username, password=password)
        if user is not None:
            login(request, user)
            # Redirect to a success page.
            ...
        else:
            # Return an 'invalid login' error message.
            ...

**Selecting the authentication backend**

user가 log in 할 때, user의 ID와 authentication을 할 때 사용된 backend는 user의 session에 저장됩니다. 이는 같은 `authentication backend`가 다음 request에 대하여 user의 세부사항을 가져오게 할 수 있습니다. session에 저장할 authentication backend는 다음과 같이 선택됩니다:

1. 제공된다면 optional backend argument를 사용하십시오.
2. 존재한다면 user.backend attribute의 값을 사용하십시오. 이는 authenticate()와 login()을 페어링 하게 해줍니다: authenticate()는 그것이 반환하는 user object에 user.backend attribute를 설정합니다.
3. 만약 한 가지만 있다면, **`AUTHENTICATION_BACKENDS`**에 있는 **backend**를 사용합니다.
4. 그렇지 않으면 exception을 일으킵니다.

1번과 2번의 경우에 backend argument의 값과 user.backend attribute는 실제 backend class가 아니라, dotted import path string이어야 합니다(AUTHENTICATION_BACKENDS에서 발견할 수 있는 것과 같습니다).

---

## How to log a user out

**logout(*request*)**

`**django.contrib.auth.login()**`을 통해 log in 한 user를 log out 시키기 위해, `**django.contrib.auth.logout()**`을 당신의 view에 사용하십시오. 이는 **`HttpRequest`** object를 인수로 쓰고 반환 값은 없습니다. 예를 보십시오:

    from django.contrib.auth import logout
    
    def logout_view(request):
        logout(request)
        # Redirect to a success page.

**`logout()`**은 만약 user가 log in 되지 않았을 경우 아무런 error를 throw하지 않습니다.

당신이 **`logout()`**을 호출할 때, 현재 request에 대한 session data는 완전히 없어질 것입니다. 모든 남아있는 data는 제거될 것입니다. 이는 같은 Web browser를 사용하는 다른 사람이 Log in하고 이전 user의 session data에 access하는 것을 막기 위함입니다. 만약 user가 log out한 뒤에 바로 사용 가능한 어떤 정보를 session에 담고 싶다면, **`django.contrib.auth.logout()`**을 호출한 뒤에 하십시오.

---

## Limiting access to logged-in users

**The raw way**

pages에 대한 access를 제한하는 가장 raw한 방법은 **`request.user.is_authenticated`**를 확인하고 login page로 redirect하는 것입니다:

 

    from django.conf import settings
    from django.shortcuts import redirect
    
    def my_view(request):
        if not request.user.is_authenticated:
            return redirect('%s?next=%s' % (settings.LOGIN_URL, request.path))
        # ...

또는 error message를 보여 주는 것입니다:

    from django.shortcuts import render
    
    def my_view(request):
        if not request.user.is_authenticated:
            return render(request, 'myapp/login_error.html')
        # ...

---

## The login_required decorator

**login_required(*redirect_field_name='next', login_url=None)***

간편하게, 당신은 **`login_required()`** decorator를 사용할 수 있습니다:

    from django.contrib.auth.decorators import login_required
    
    @login_required
    def my_view(request):
        ...

**`login_required()`**는 다음과 같은 것을 합니다:

- 만약 사용자가 로그인하지 않았다면, **`settings.LOGIN_URL`**으로 redirect해주고, query string에 현재 absolute path를 넘겨줍니다. 예를 들어: **/accounts/login/?next=/polls/3/.**
- 만약 로그인 되어 있다면, 평소와 같이 view를 실행합니다. 이 view code는 사용자가 로그인 했다고 가정할 수 있습니다.

기본적으로 성공적인 authentication 일 경우 사용자를 redirect 해주어야 하는 경로는 "next"라 불리는 query string parameter에 저장됩니다. 만약 이 parameter에 대하여 다른 이름을 선호한다면, **`login_required()`**는 optional **redirect_field_name** parameter를 받아들입니다:

    from django.contrib.auth.decorators import login_required
    
    @login_required(redirect_field_name='my_redirect_field')
    def my_view(request):
        ...

어떤 한 값을 **redirect_field_name**로 줄 경우, 대부분 login template을 customize를 할 필요가 있을 것입니다. 왜냐하면 redirect path를 갖고 있는 template context variable은 그것의 key로 초기 값인 "**next**" 대신에 **redirect_field_name**의 값을 사용할 것이기 때문입니다.

**`login_required()`**는 또한 optional **login_url** parameter를 취합니다. 예를 보십시오:

    from django.contrib.auth.decorators import login_required
    
    @login_required(login_url='/accounts/login/')
    def my_view(request):
        ...

만약 **login_url** parameter를 명시하지 않았다면, 당신은 **`settings.LOGIN_URL`**과 login view가 제대로 연결되어 있는지 확인해야 할 것입니다. 예를 들어, 기본 값들을 사용한다면, URLconf에 다음 줄들을 추가하십시오:

    from django.contrib.auth import views as auth_views
    
    path('accounts/login/', auth_views.LoginView.as_view()),

**`settings.LOGIN_URL`**은 또한 view function names와 `named URL patter`를 받아들입니다. 이는 당신의 setting 을 업데이트 하지 않고도 URLconf로 login view로 자유롭게 다시 연결할 수 있습니다.

- **Note**

    **login_required** decorator는 user의 **is_active** flag를 확인하는 것이 아닙니다. 그러나 기본 **`AUTHENTICATION_BACKENDS`**가 inactive users를 거부할 것입니다

- **See also**

    만약 Django의 admin에 대한 custom view를 작성하고 (또는 built-in views가 사용하는 같은 authorization check가 필요하다면), 아마 **login_required()** 대용으로 **`django.contrib.admin.views.decorators.staff_member_required()`**가 더 유용합니다.

## The LoginRequired mixin

`class-based viwes`를 사용할 때, **LoginRequriedMixin**을 사용하여 **login_required**와 같은 동작을 만들 수 있습니다. 이 mixin은 inheritance list에서 가장 마지막에 있어야 합니다.

***class* LoginRequiredMixin**

만약 view가 이 mixin을 사용하고 있다면, 모든 non-authenticated user의 request는 **`raise_exception`** parameter에 따라서 login page로 redirect 되거나 HTTP 403 Forbidden error를 일으킬 것입니다.

unauthorized users를 자유롭게 다루기 위해 **`AccessMixin`**의
 어떠한 parameter도 설정할 수 있습니다.

    from django.contrib.auth.mixins import LoginRequiredMixin
    
    class MyView(LoginRequiredMixin, View):
        login_url = '/login/'
        redirect_field_name = 'redirect_to'

- **Note**

    **login_required** decorator와 마찬가지로, 이 mixin은 user의 **is_active** flag를 확인하지 않습니다. 하지만 기본 **`AUTHENTICATION_BAKCENDS`**가 inactive users를 거부할 것입니다.

---

## Limiting access to logged-in users that pass a test

특정 permissions나 다른 테스트에 따라 access를 제한하기 위해, 앞의 섹션에서 기술된 것과 동일한 작업을 수행하십시오.

간단한 방법은 view에서 **`request.user`**에 대한 테스트를 직접 실행하는 것입니다. 예를 들어, 이 view는 user가 알맞은 도메인에 email을 갖고 있는지를 확인하고, 만약 그렇지 않다면 login page로 redirect합니다.

    from django.shortcuts import redirect
    
    def my_view(request):
        if not request.user.email.endswith('@example.com'):
            return redirect('/login/?next=%s' % request.path)
        # ...

**user_passes_text(*test_func, login_url=None, redirect_field_name='next'*)**

shortcut으로서, callable이 **False**를 반환하면 redirect를 수행하는 편리한 **user_passes_text**를 사용할 수 있습니다.

    from django.contrib.auth.decorators import user_passes_test
    
    def email_check(user):
        return user.email.endswith('@example.com')
    
    @user_passes_test(email_check)
    def my_view(request):
        ...

**`user_passes_test()`**는 필수 argument가 있습니다: 바로 **`User`** object를 받아들이고 만약 user가 page를 볼 수 있다면 **True**를 반환하는 callable 입니다. **`user_passes_test()`**는 **`User`**가 anonymous가 아닌지 자동적으로 확인하지 않다는 점에 유의하십시오.

**`user_passes_test()`**는 두 개의 optional arguments를 취합니다:

**login_url**

테스트를 통과하지 못한 사용자가 redirect 될 URL을 지정할 수 있습니다. 아마 login
 page이거나 만약 아무것도 지정하지 않는다면 **`settings.LOGIN_URL`**에 있는 기본 값이 될 것입니다.

**redirect_field_name**

**`login_required()`**와 같습니다. 이를 **None**으로 설정하는 것은 이를 URL에서 제거하는데, 이는 만약 테스트를 통과하지 않은 사용자들을 "next page"가 없는 non-login page로 redirect할 때 원할 것입니다.

다음은 예시입니다:

    @user_passes_test(email_check, login_url='/login/')
    def my_view(request):
        ...

***class* UserPassesTestMixin**

`class-based views`를 사용할 때, **UserPassesTestMixin**을 사용할 수 있습니다.

**test_func()**

수행될 test를 제공하기 위해서 class의 **test_func()**을 override해야 합니다. 더욱이, unathroized users를 다루는 것을 customize하기 위해 **`AccessMinxin`** parameter를 설정할 수 있습니다.

    from django.contrib.auth.mixins import UserPassesTestMixin
    
    class MyView(UserPassesTestMixin, View):
    
        def test_func(self):
            return self.request.user.email.endswith('@example.com')

**get_test_func()**

mixin이 다른 이름의 function들을 사용할 수 있게 하기 위해 **get_test_func()** method를 override할 수 있습니다. (**`test_func()`** 대신에)

- **Stacking UserPassesTestMixin**

    **UserPassesTestMixin**이 구현된 방법 때문에, 그들을 당신의 inheritance list에 stack할 수 있습니다. 다음은 작동하지 않을 것입니다:

        class TestMixin1(UserPassesTestMixin):
            def test_func(self):
                return self.request.user.email.endswith('@example.com')
        
        class TestMixin2(UserPassesTestMixin):
            def test_func(self):
                return self.request.user.username.startswith('django')
        
        class MyView(TestMixin1, TestMixin2, View):
            ...

    만약 **TestMixin1**이 **super()**을 호출하고 그 결과를 고려한다면, **TestMixin1**은 더 이상 독립적으로 작동하지 않을 것입니다. (원문 : If TestMixin1 would call super() and take that result into account, TestMixin1 wouldn’t work standalone anymore.)

---

**The permission_required decorator**

**permission_required(*perm, login_url=None, raise_exception=False*)**

user가 특정 permission을 가지고 있는지 확인하는 것은 비교적 흔한 작업입니다. 이 이유로, Django는 이 경우에 대한 shortcut을 제공합니다: permission_required() decorator입니다:

    from django.contrib.auth.decorators import permission_required
    
    @permission_required('polls.can_vote')
    def my_view(request):
        ...

has_perm() method와 마찬가지로, permission 이름은 "<app label>.<permission codename>"의 형태를 취합니다(즉, polls.can_vote는 polls application의 model에 대한 permission입니다).

decorator는 permissions의 iterable도 취할 수 있는데, user가 view에 access하기 위해 모든 permissions를 가지고 있어야 하는 경우에 사용합니다.

permission_required()는 또한 optional login_url parameter도 갖고 있다는 점을 알아 두십시오.

    from django.contrib.auth.decorators import permission_required
    
    @permission_required('polls.can_vote', login_url='/loginpage/')
    def my_view(request):
        ...

login_required() decorator와 마찬가지로, login_url의 기본값은 settings.LOGIN_URL로 설정되어 있습니다.

만약 raise_exception parameter가 주어진다면, decorator는 PermissionDenied를 일으킬 것이며, login page로 redirecting 하는 대신에 403(HTTP Forbidden) view를 보여줄 것입니다.

    from django.contrib.auth.decorators import login_required, permission_required
    
    @login_required
    @permission_required('polls.can_vote', raise_exception=True)
    def my_view(request):
        ...

이는 또한 LoginView의 redirect_authenticated_user=True이고 logged-in
 user가 필요한 모든 permissions를 갖고 있지 않을 경우 redirect loop를 회피해줍니다.

---

The PermissionRequiredMixin mixin

class-based views에 permission checks를 적용하기 위해, PermissionRequiredMixin을 사용할 수 있습니다:

class PermissionRequiredMixin

permission_required decorator와 마찬가지로, 이 mixin은 view에 accessing하는 user가 모든 주어진 permissions를 갖고 있는지 확인합니다. 당신은 permission_required parameter를 사용하여 permission (혹은 iterable of permissions)를 명시해주어야 합니다:

    from django.contrib.auth.mixins import PermissionRequiredMixin
    
    class MyView(PermissionRequiredMixin, View):
        permission_required = 'polls.can_vote'
        # Or multiple of permissions:
        permission_required = ('polls.can_open', 'polls.can_edit')

당신은 unauthorized users를 다루는 것을 customize하기 위해 어떠한 AccessMixin의 parameters도 설정할 수 있습니다.

다음과 같은 methods를 override할 수 있습니다:

get_permission_required()

mixin에 사용된 iterable of permission을 반환합니다. 기본값은 permission_requried로 설정되며, 필요할 경우 tuple로 변환됩니다.

has_permission()

현재 user가 decorated view를 실행시킬 수 있는 permission을 갖고 있는지 표시하는 boolean을 반환합니다. 기본적으로, 이는 get_permission_required()에 의해 반환된 list of permissions와 함께 has_perms()를 호출한 결과를 반환합니다.

---

## Redirecting unauthorized requests in class-based views

class-based views에서 접근 제한을 쉽게 다루기 위해, AccessMixin은 access가 거부 될 때, view의 행동을 configure하기 위해 사용될 수 있습니다. Authenticated users는 HTTP 403 Forbidden response로 access가 거부됩니다. 
Anonymous users는 raise_exception attribute에 따라 login page로 redirect 되거나 HTTP 403 Forbidden response를 볼 수 있습니다.

- Changed in django 2.1:

    이전 버전에서, permissions가 없는 authenticated users은 HTTP 403 Forbidden response를 받는 대신에 (결과적으로 loop가 되는) login page로 redirect 되었습니다.

class AccessMixin

login_url

get_login_url()에 대한 default 반환 값입니다. 기본 값은 None 이며, 이 경우 get_login_url()이 settings.LOGIN_URL으로 되돌아갑니다.

permission_denied_message

get_permission_denied_message()에 대한 default 반환 값입니다. 기본 값은 empty string으로 설정되어 있습니다.

redirect_field_name

get_redirect_field_name()에 대한 default 반환 값입니다. 기본 값은 "next"로 설정되어 있습니다.

raise_exception

만약 이 attribute가 True로 설정되어 있다면, conditions가 충족되지 않는다면 PermissionDenied exception가 일어납니다. False일 때, anonymous users가 login page로 redirect 됩니다. 

get_login_url()

test를 통과하지 못한 users가 redirect 될 URL을 반환합니다. 만약 설정되어 있다면, login_url를 반화하거나 그렇지 않으면 settings.LOGIN_URL을 반환합니다.

get_permission_denied_message()

raise_exception가 True라면, 이 method는 user에게 표시하기 위한 error handler에 전달 된 error message를 제어하는데 사용할 수 있습니다. 기본적으로 permission_denied_message attribute를 반환합니다.

get_redirect_field_name()

user가 성공적으로 login한 후에 redirect 될 URL을 갖고 있는 query parameter의 이름을 반환합니다.

handle_no_permission()

raise_exception의 값에 따라, 이 method는 PermissionDenied exception을 일으키거나 user를 login_url로 redirect합니다. 그리고 만약 설정되어 있다면 optional하게 redirect_field_name을 포함합니다.

Session invalidation on password change

만약 AUTH_USER_MODEL이 AbstractBaseUser로 부터 상속 받았거나 그것의 get_session_auth_hash() method를 구현 했다면, authenticated sessions는 이 함수에 의해 반환된 hash를 포함할 것입니다. AbstractBaseUser의 경우에, 이는 password field의 HMAC입니다. Django는 각각의 request의 session에 있는 hash가 request 동안 계산된 것과 match하는지 검증합니다. 이는 user가 그들의 password를 바꾸어 모든 그들의 session에서 log out 할 수 있게 해줍니다.

Django에 포함된 default password change views, PasswordChangeView 그리고 django.contrib.auth admin에 있는 user_change_password view,이들은 session을 새로운 password로 update하여 그들의 password를 바꾼 User가 스스로 log out 되지 않도록 합니다. 만약 custom password change view를 갖고 있고 비슷한 동작을 원한다면, update_session_auth_hash() 함수를 사용하십시오.

update_session_auth_hash(request, user)

이 함수는 현재 request와 새 session hash가 파생될 updated user object를 받아들이고 session hash를 적절하게 update합니다. 이는 또한 session key를 rotate하여 도둑맞은 session cookie는 유효하지 않을 것입니다(invalidate).

사용법 예시 입니다:

    from django.contrib.auth import update_session_auth_hash
    
    def password_change(request):
        if request.method == 'POST':
            form = PasswordChangeForm(user=request.user, data=request.POST)
            if form.is_valid():
                form.save()
                update_session_auth_hash(request, form.user)
        else:
            ...

- Note

    get_session_auth_hash()가 SECRET_KEY 기반이기 때문에, 사이트가 새로운 secret을 사용하도록 update하는 것은 존재하는 모든 session을 invalidate하게 만들 것입니다.

---

## Authentication Views

Django는 login, logout, password management를 다루기 위해 사용할 수 있는 몇 가지 views를 제공합니다. 이들은 stock auth forms를 활용하지만, 당신만의 고유 forms를 넘겨줄 수도 있습니다.

Django는 authentication view를 위한 기본 template를 제공하지 않습니다. 당신은 사용하고 싶은 views를 위한 template를 직접 만드셔야 합니다. template context는 각각의 view에 documented되어 있습니다. All authentication views를 보십시오.

Using the views

당신의 프로젝트에 이 views를 구현할 수 있는 여러 방법들이 있습니다. 가장 쉬운 방법은 URLconf에 있는 django.contrib.auth.urls에 있는 주어진 URLconf를 포함하는 것입니다. 예를 보십시오:

    urlpatterns = [
        path('accounts/', include('django.contrib.auth.urls')),
    ]

이는 다음과 같은 URL patterns를 포함합니다:

    accounts/login/ [name='login']
    accounts/logout/ [name='logout']
    accounts/password_change/ [name='password_change']
    accounts/password_change/done/ [name='password_change_done']
    accounts/password_reset/ [name='password_reset']
    accounts/password_reset/done/ [name='password_reset_done']
    accounts/reset/<uidb64>/<token>/ [name='password_reset_confirm']
    accounts/reset/done/ [name='password_reset_complete']

이 views는 쉬운 참조를 위해 URL name을 제공합니다. 주어진 이름의 URL patterns를 사용하기 위해서는 the URL documentation을 참조하십시오.

만약 당신의 URLs를 더 컨트롤하고 싶다면, URLconf에 특정 view를 참조해주면 됩니다:

    from django.contrib.auth import views as auth_views
    
    urlpatterns = [
        path('change-password/', auth_views.PasswordChangeView.as_view()),
    ]

이 view들은 view의 동작을 바꾸기 위해 사용할 수 있는 optional arguements를 갖고 있습니다. 예를 들어, 만약 view가 사용하는 template name을 바꾸고 싶다면, template_name argument를 제공해주면 됩니다. 이를 하는 방법은 URLconf에 keyword argument를 제공해주면, 이는 view로 전달 될 것입니다. 예를 보십시오:

    urlpatterns = [
        path(
            'change-password/',
            auth_views.PasswordChangeView.as_view(template_name='change-password.html'),
        ),
    ]

모든 views는 상속을 통해 쉽게 customize할 수 있는 class-based입니다.

---

All authentication views

이는 django.contrib.auth가 제공하는 모든 view의 list입니다. implementation details를 보고 싶다면 Using the views를 참조하십시오.

class LoginView

URL name: login

이름이 주어진 URL patterns에 대한 세부사항은 URL documentation을 보십시오.

Attributes:

- template_name: user를 log in 하는데 사용될 view를 보여주는 template 이름입니다. 기본 값으로는 registration/login.html입니다.
- redirect_field_name: login이후에 redirect할 URL을 갖고 있는 GET field입니다. 기본 값으로는 next입니다.
- authentication_form: authentication에 사용할 (전형적으로는 form class) callable입니다. 기본 값으로는 AuthenticationForm입니다.
- extra_context: template에 전달 된 default context data에 추가 될 context data의 dictionary입니다.
- redirect_authenticated_user: login page에 access한 authenticated user가 막 성공적으로 log in 된 것처럼 redirect 되는지 여부를 제어하는 boolean입니다. 기본적으로 False입니다.
    - Warning

        만약 redirect_authenticated_user를 사용한다면, 다른 웹 사이트에서는 해당 웹 사이트의 이미지 파일에 대한 redirect URL을 request하여 해당 방문자가 사용자 사이트에서 authenticated되었는지 여부를 확인할 수 있습니다. 이러한 "social media fingerprinting" 정보 누출을 방지하기 위해서 모든 이미지와 즐겨찾기를 별도의 도메인에 호스팅 하십시오.

        redirect_authenticated_user를 사용하는 것은 raise_exception parameter가 사용되지 않는 이상 permission_required()를 사용할 때 redirect loop의 결과를 초래할 수 있습니다.

- success_url_allowed_hosts: request.get_host()뿐만 아니라 hosts의 set으로 login 이후 redirect를 하는데 안전합니다. (A set of hosts, in addition to request.get_host(), that are safe for redirecting after login. Defaults to an empty set.)

다음은 LoginView가 하는 것들입니다:

- GET을 통해 호출하면 동일한 URL에 POST를 표시하는 login form이 표시됩니다. 이것에 대해서는 잠시 후 더 설명합니다.
- POST를 통해 user가 제출한 credentials와 함께 호출되면, 이는 user를 login하려 합니다. 성공적으로 log in할 경우, view가 next에 지정된 URL로 redirect합니다. next가 제공되지 않으면 이는 settings.LOGIN_REDIRECT_URL로 redirect합니다. 성공적으로 log in 안 될 경우, 이는 login form을 다시 보여줍니다.

기본적으로 registration/login.html인 login template에 html을 제공해야야 합니다. 이 template는 다음 네 가지 template context variables를 갖습니다:

- form: AuthenticationForm을 나타내는 Form object입니다.
- next: login을 성공한 이후 redirect하는 URL입니다. 이는 query string을 포함할 수 있습니다.
- site: SITE_ID setting에 따른 현재 Site입니다. 만약 설치된 site framework를 갖고 있지 않다면, 이는 RequestSite의 instance로 설정될 것입니다. 그리고 이 site 이름과 도메인을 현재 HttpRequest에서 파생합니다.
- site_name: site.name의 alias입니다. 만약 설치된 site framework가 없다면, 이는 request.META['SERVER_NAME']의 값으로 저장됩니다. 더 많은 sites에 대해서는, The "sites" framework를 보십시오.

만약 registration/login.html template을 호출하는 것을 원하지 않는다면, 당신의 URLconf에  있는 as_view method에 추가 argument를 통해 template_name parameter를 넘겨줄 수 있습니다. 예를 들어, 이 URLconf line은 myapp/login.html을 대신 사용합니다:

    path('accounts/login/', auth_views.LoginView.as_view(template_name='myapp/login.html')),

 

또한, redirect_field_name을 사용하여 login한 이후에 redirect할 URL을 포함하고 있는 GET field의 이름을 지정해줄 수 있습니다. 기본적으로 이 field는 next라 불립니다.

다음은 시작으로 사용할 수 있는 샘플 registration/login.html template입니다. 이는 content block을 정해준 base.html template가 있다고 가정합니다:
    {% raw %}
    {% extends "base.html" %}
    
    {% block content %}
    
    {% if form.errors %}
    <p>Your username and password didn't match. Please try again.</p>
    {% endif %}
    
    {% if next %}
        {% if user.is_authenticated %}
        <p>Your account doesn't have access to this page. To proceed,
        please login with an account that has access.</p>
        {% else %}
        <p>Please login to see this page.</p>
        {% endif %}
    {% endif %}
    
    <form method="post" action="{% url 'login' %}">
    {% csrf_token %}
    <table>
    <tr>
        <td>{{ form.username.label_tag }}</td>
        <td>{{ form.username }}</td>
    </tr>
    <tr>
        <td>{{ form.password.label_tag }}</td>
        <td>{{ form.password }}</td>
    </tr>
    </table>
    
    <input type="submit" value="login">
    <input type="hidden" name="next" value="{{ next }}">
    </form>
    
    {# Assumes you setup the password_reset view in your URLconf #}
    <p><a href="{% url 'password_reset' %}">Lost password?</a></p>
    
    {% endblock %}
    {% endraw %}

만약 customized authentication(Customizing Authentication을 보십시오)를 갖고 있다면, authentication_form attribute를 설정함으로서 custom authentication을 사용할 수 있습니다. 이 form은 그것의 __init__() method에 request keyword argument를 받아들여야 하고 authenticated user object를 반환하는 get_user() method를 제공해야 합니다.

class LogoutView

user를 logout 시킵니다.

URL name: logout

Attributes:

- next_page: logout후에 redirect할 URL입니다. 기본적으로 settings.LOGOUT_REDIRECT_URL입니다.
    - template_name: user가 log out한 후에 보여줄 template의 전체 이름입니다. 기본적으로 registration/logged_out.html입니다.
    - redirect_field_name: log out 후에 redirect할 URL을 갖고 있는 GET field의 이름입니다. 기본적으로 next 입니다. 주어진 GET parameter가 주어지면 next_page URL을 override합니다.
    - extra_context: template에 전달되는 default context data에 추가되는 context data의 dictionary입니다.
    - success_url_allowed_hosts: request.get_host()와 더불어 logout 후에 redirecting에 안전한 hosts의 set입니다. 기본적으로는 빈 set입니다.

    Template context:

    - title: localized된 "Logged out" 문자열입니다.
    - site: SITE_ID 설정에 따른 현재 Site입니다. 만약 설치된 site framework가 없다면, 이는 현재 HttpRequest에서 site이름과 도메인을 파생하는 RequestSite의 instance로 설정될 것입니다.
    - site_name: site.name의 alias입니다. 만약 설치된 site framework가 없다면, 이는 request.META['SERVER_NAME']의 값으로 설정될 것입니다. 더 많은 sites에 대해서는 The "sites" framework를 보십시오.

logout_then_login(request, login_url=None)

user를 log out 한 뒤 login page로 redirect합니다.

URL name: 제공된 기본 URL이 없습니다.

Optional arguments:

- login_url: redirect할 login page의 URL입니다. 제공되지 않는다면 settings.LOGIN_URL이 기본값입니다.

class PasswordChangeView

URL name: password_change

user가 그들의 password를 바꾸도록 해줍니다.

Attributes:

- template_name: password change form을 보여주는데 사용할 template name입니다. 제공되지 않는다면 기본적으로 registration/password_change_form.html입니다.
- success_url: 성공적으로 password를 바꾼 후에 redirect될 URL입니다.
- form_class: user keyword argument를 받아야만하는 custom "change password" form입니다. form은 user의 password를 실제로 변경하는 책임이 있다. 기본적으로 PasswordChangeForm입니다.
- extra_context: template에 전달되는 default context data에 추가되는 context data의 dictionary입니다.

Template context:

- form: password change form입니다.(위의 form_class를 보십시오)

class PasswordChangeDoneView

URL name: password_change_done

user가 password를 바꾼 후 보여지는 페이지입니다.

Attributes:

- template_name: 사용할 template의 전체 이름입니다. 기본적으로 registration/password_change_done.html입니다.
- extra_context: template에 전달되는 default context data에 추가될 context data의 dictionary입니다.

class PasswordResetView

URL name: password_reset

password를 reset하는데 사용할 수 있는 일회용 링크를 생성하고, 그 링크를 사용자의 등록된 이메일 주소로 전송하여 user가 password를 재설정 할 수 있도록 한다.

만약 제공된 이메일 주소가 존재하지 않는다면, 이 view는 이메일을 전송하지 않지만, user 역시 아무런 error message를 받을 수 없습니다. 이는 가능한 공격으로부터 정부 누출을 예방하기 위해서 입니다. 만약 이 경우 error message를 제공하기 원한다면, PsswordResetForm을 상속하여 form_class attribute 를 사용할 수 있습니다.

사용할 수 없는 password로 flagged된 users는(set_unusable_password())는 LDAP와 같은 외부 authentication 인증 소스를 사용할 때 오용을 막기 위해 비밀번호 재설정을 요청할 수 없습니다. 이들 역시 이들의 계정이 드러날 수 있기 때문에 아무런 error message를 받지 못하지만, 이메일 역시 보내지지 않을 것입니다.

Attributes:

- template_name: password reset form을 보여주기 위해 사용할 template의 전체 이름입니다. 기본적으로 registration/password_reset_form.html으로 설정되어 있습니다.
- form_class: password를 reset하기 위해 사용자의 이메일을 받기 위한 폼입니다. 기본적으로 PasswordResetForm입니다.
- email_template_name: 비밀번호 재설정 링크와 함께 이메일을 생성하는데 사용할 템플릿의 전체 이름입니다. 기본적으로 registration/password_reset_email.html입니다.
- subject_template_name: 비밀번호 재설정 링크가 있는 이메일 제목에 사용할 템플릿의 전체 이름입니다. 기본적으로 registration/password_reset_subject.txt입니다.
- token_generator: 일회성 링크를 확인하는 클래스의 인스턴스입니다. 이는 기본적으로 default_token_generator이고 django.contrib.auth.tokens.PasswordResetTokenGenerator입니다.
- success_url: 성공적으로 비밀번호 재설정 요청 이후에 리디렉트할 URL입니다.
- from_email: 유효한 이메일 주소입니다. 기본적으로 django는 DEFAULT_FROM_EMAIL을 사용합니다.
- extra_context: 템플릿에 전달되는 기본 context data에 추가될 context data의 딕셔너리입니다.
- html_email_template_name: 비밀번호 재설정 링크를 사용하여 text/html 다중 메일 생성에 사용할 템플릿의 전체 이름입니다. 기본적으로 HTML 이메일은 보내지지 않습니다.
- extra_email_context: 이메일 템플릿에서 사용가능한 context data의 딕셔너리입니다.

Template context:

- form: 사용자의 비밀번호를 재설정 하는데 사용하는 폼입니다(위의 form_class을 보십시오).

Email template context:

- email: user.email의 alias입니다.
- user: 이메일 폼 필드에 따른 현재 User입니다. 활성 사용자만이 비밀번호를 재설정 할 수 있습니다(User.is_active is True).
- site_name: site.name에 대한 alias입니다. 만약 설치된 사이트 프레임워크를 갖고 있지 않다면, 이는 request.META['SERVER_NAME']의 값으로 설정됩니다. 더 많은 사이트에 대해서는 The "sites" framework를 보십시오.
- domain: site.domain에 대한 alias입니다. 만약 설치된 사이트 프레임워크를 갖고 있지 않다면, 이는 request.get_host()의 값으로 설정됩니다.
- protocol: http 또는 https
- uid: 64 기반으로 인코딩된 사용자의 primary key입니다.
- token: 재설정 링크가 유효한지 확인해주는 토큰입니다.

샘플 registration/password_reset_email.html(email body template)입니다.
    {% raw %}
    Someone asked for password reset for email {{ email }}. Follow the link below:
    {{ protocol}}://{{ domain }}{% url 'password_reset_confirm' uidb64=uid token=token %}
    {% endraw %}

같은 template context가 subject template에도 사용됩니다. 제목(Subject)은 반드시 한 줄의 일반 텍스트 문자열이어야 합니다.(single line plain text string)

class PasswordRestDoneView

URL name: password_reset_done

사용자가 암호를 재설정 할 수 있는 링크를 이메일로 받은 후에 보여지는 페이지입니다. 이 view는 PassWordResetView가 명시적 success_url URL set을 갖고 있지 않다면 기본적으로 호출됩니다.

- Note

    만약 제공된 이메일이 존재하지 않거나, 사용자가 비활성 상태(inactive)이거나, 사용할 수 없는 비밀번호를 가진 경우, 사용자는 여전히 이 view로 리디렉트 되지만 이메일은 보내지지 않습니다.

Attributes:

- template_name: 사용될 템플릿의 전체 이름입니다. 기본값은 registration/password_reset_done.html입니다.
- extra_context: 템플릿에 전달되는 default context data에 추가될 context data의 딕셔너리입니다.

class PasswordResetConfrimView

URL name: password_reset_confrim