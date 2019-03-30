---
big-title: "Forms"
middle-title: "The basics"
small-title: "Overview"
author: 
  - 김선규(Kim,seon-kyu)
field:
  - The basics
relate:
  - The basics
toc: true
toc-head-level-choice: false
#do this if head level choice is true
# toc-head-max:
# toc-head-min:
---

> 원본 링크: [https://docs.djangoproject.com/en/2.1/topics/forms/](https://docs.djangoproject.com/en/2.1/topics/forms/)

# Working with forms

**About this Document**

이 문서는 웹 폼의 기본 사항과 장고에서 그것을 어떻게 핸들링하는지 소개합니다. 폼 API의 특정 영역에 대한 자세한 내용은 `The Forms API`, `Form fields`, and `Form and field validation`을 참고하십시오.

만약 다른 기능 없이 컨텐츠를 출판하거나 혹은 방문자로부터 아무런 입력을 받지 않아도 되는 웹사이트 혹은 애플리케이션만들지 않는 이상, 당신은 폼의 사용에 대해 이해해야 합니다.

장고는 사이트 방문자로부터 입력을 받을 수 있는 폼을 만들고, 그것을 처리하고 응답하는 것을 도와주기 위해 다양한 도구와 라이브러리를 제공합니다.

# HTML forms

HTML에서, 폼은 **<form> ... </form>** 사이의 요소들의 집합입니다. 그리고 폼은 방문자들이 문자를 입력하고, 옵션을 선택하고, 객체나 제어장치 조작 등의 작업을 할 수 있게 해주고, 그 정보들을 서버로 다시 보냅니다.

이러한 텍스트 인풋 또는 체크박스와 같은 폼 인터페이스 요소들 중 일부는 매우 간단하며 HTML 자체에 내장되어 있습니다. 다른 것들은 더 복잡합니다; 날짜 선택 팝업창이 뜨거나 슬라이더를 이동하거나 제어장치를 조절할 수 있는 인터페이스는 일반적으로 HTML 폼 **<input>** 요소뿐만 아니라 JavaScript와 CSS를 이용하여 이러한 효과를 얻습니다.

**<input>** 요소뿐만 아니라 폼은 다음 두 가지를 지정해야 합니다:

- *어디에*: 사용자 입력에 해당하는 데이터를 반환해야 하는 URL
- *어떻게*: 데이터를 반환하는 HTTP 방법

예를 들어, 장고 admin 로그인 폼에는 몇 가지 <input> 요소가 포함되어 있습니다: 사용자 이름은 **type="text"**, 패스워드는 **type="password"**, 로그인 버튼은 **type="submit"**입니다. 그것은 또한 사용자가 보지 못하는 숨겨진 텍스트 필드를 포함하고 있는데, 이 필드들은 다음에 무엇을 해야 할지 결정하기 위해 사용합니다.

또한 폼 데이터는 **<input>**의 **action** attribute에 명시된 URL로 전송되어야 하며 (예를 들어 **/admin/**), 그리고 **method** attribute에 명시된 HTTP 방법으로 전송되어야 한다(예를 들어 **post**)고 브라우저에게 알려주어야 합니다. 

**<input type="submit" value="Log in">** 요소가 발생할 경우, 데이터는 **/admin/**으로 전달됩니다.

## GET and POST

**GET**과 **POST**만이 폼을 다룰 때 사용되는 HTTP method입니다.

장고의 로그인 폼은 POST 방식을 이용해 반환되는데, POST 방식은 브라우저가 폼 데이터를 일괄적으로 암호화하여 전송하고, 서버로 전송한 다음 응답을 수신하는 방식입니다.

GET은 반대로 제출된 데이터를 문자열로 묶고 URL을 구성하는데 사용됩니다. URL에는 데이터 키 값뿐만 아니라 데이터를 전송해야 하는 주소가 포함되어 있습니다. 당신은 이 액션을 장고 문서 검색창에서 확인할 수 있습니다. 이는 **https://docs.djangoproject.com/search/?q=forms&release=1**와 같은 형식의 URL을 반환합니다.

**GET**과 **POST**는 일반적으로 다른 목적으로 사용됩니다.

> 번역 생략

# Django's role in forms

폼을 처리하는 것은 복잡한 일입니다. 여러 가지 다른 유형의 수많은 데이터 항목이 폼에서 표시되어야 하고, HTML로 렌더링되며, 편리한 인터페이스를 사용하여 편집되고, 서버로 반환되고, 유효성 검사 또는 정리를 한 다음, 저장하거나 추가 처리를 할 수 있는 장고 admin을 고려해보십시오.

장고의 폼 기능은 이 작업의 상당 부분을 단순화하고 자동화할 수 있으며, 대부분의 프로그래머들이 스스로 작성한 코드로 할 수 있는 것보다 더 안전하게 할 수 있습니다.

장고는 폼과 관련된 세 개의 뚜렷한 부분을 처리합니다:

- 렌더링 준비를 위해 데이터를 준비하고 reconstruct합니다.
- 데이터에 대한 HTML 폼을 생성합니다.
- 클라이언트로부터 제출된 폼과 데이터를 수신하고 처리합니다.

이 모든 것을 수동으로 하는 코드를 쓰는 것은 가능하지만, 장고는 당신을 위해 모든 것을 처리할 수 있습니다.

> 번역 생략

# Building a form

## The work that needs to be done

유저 이름을 얻기 위해 웹 사이트에 간단한 폼을 만들고자 한다고 가정해봅시다. 당신은 템플릿에 다음과 같은 것들이 필요할 것입니다:

    <form action="/your-name/" method="post">
        <label for="your_name">Your name: </label>
        <input id="your_name" type="text" name="your_name" value="{{ current_name }}">
        <input type="submit" value="OK">
    </form>

이는 브라우저에게 **POST** 방법을 사용하여 폼 데이터를 **/your-name/** URL에 반환하도록 지시합니다. 이것은 "Your name:"의 텍스트 필드를 보여주고, OK라 표시된 버튼을 보여줍니다. 템플릿 context에 **current_name** 변수가 포함되어 있는 경우, 이 변수는 **your_name** 필드를 미리 채우는데 사용됩니다.

HTML 폼을 포함하는 템플릿을 렌더링할 수 있는 뷰가 필요하고, 적절한 경우에 뷰는 **current_name** 필드를 제공할 수 있어야 합니다.

폼이 제출될 때, 서버로 전송되는 **POST** 요청에는 폼 데이터가 포함됩니다. 

이제 요청에서 적절한 키/값 쌍을 찾은 다음 이를 처리하는 **/your-name/** URL에 해당하는 뷰도 필요합니다.

이는 매우 간단한 폼입니다. 실제로 폼에서는 수십 개 또는 수백 개의 필드가 포함될 수 있으며, 이 중 많은 필드를 미리 입력할 수 있으며, 사용자가 작업을 마무리하기 전에 편집-제출 사이클을 여러번 거치게 될 것으로 예상할 수 있습니다.

양식을 제출하기 전이라도 브라우저에서 검증이 필요할 수 있습니다; 사용자가 일정관리에서 날짜 선택 등과 같은 작업을 할 수 있도록 훨씬 더 복잡한 필드를 사용할 수 있습니다.

## Building a form in Django

### The Form class

우리는 이미 우리의 HTML 양식이 어떻게 보이길 원하는지 알고 있습니다. 우리의 장고에서 출발점은 다음과 같습니다:

forms.py

    from django import forms
    
    class NameForm(forms.Form):
        your_name = forms.CharField(label='Your name', max_length=100)

이것은 하나의 필드(**your_name**)을 갖는 **Form** 클래스를 정의합니다. 우리는 필드에 인간 친화적인 레이블을 적용했고, 이것이 렌더링 되면 **<label>**과 같이 나타날 것입니다. (이 경우, 우리가 지정한 **label**은 자동으로 생성될 label과 동일합니다.)

필드의 최대 허용 길이는 **max_length**로 정의됩니다. 이는 두 가지 일을 합니다. HMLT **<input>**에 **maxlength="100"**을 설정합니다. (그래서 브라우저는 유저가 애당초 그 수 이상의 문자를 입력하는 것을 방지해야 합니다). 이것은 또한 장고가 브라우저로부터 폼을 다시 수신할 때, 길이를 검증하는 것을 의미한다.

**Form** 인스턴스는 **is_valid()** 메소드를 갖고 있고, 이는 그것의 모든 필드에 대해 유효성 검사를 실행합니다. 이 메소드가 호출될 때, 만약 모든 필드가 유효한 데이터를 포함하고 있다면,

- **True**를 반환합니다.
- 폼의 데이터를 **cleaned_data** attribute 안에 위치시킵니다.

전체 폼은, 처음에 랜더링 되고 나서 다음과 같이 생겼습니다:

    <label for="your_name">Your name: </label>
    <input id="your_name" type="text" name="your_name" maxlength="100" required>

이것이 **<form>** 태그나 submit 버튼이 없다는 것에 유의하십시오. 우리는 이들은 템플릿에서 추가해야 합니다.

### The view

장고 웹사이트로 다시 전송된 폼 데이터는 일반적으로 폼을 게시한 것과 동일한 뷰에 의해서 처리됩니다. 이것은 우리가 같은 로직의 일부를 재사용 할 수 있게 해줍니다.

폼을 처리하기 위해서는 폼을 게시하고자 하는 URL의 뷰에서 폼을 인스턴스화 할 필요가 있습니다.

**views.py**

    from django.http import HttpResponseRedirect
    from django.shortcuts import render
    
    from .forms import NameForm
    
    def get_name(request):
        # if this is a POST request we need to process the form data
        if request.method == 'POST':
            # create a form instance and populate it with data from the request:
            form = NameForm(request.POST)
            # check whether it's valid:
            if form.is_valid():
                # process the data in form.cleaned_data as required
                # ...
                # redirect to a new URL:
                return HttpResponseRedirect('/thanks/')
    
        # if a GET (or any other method) we'll create a blank form
        else:
            form = NameForm()
    
        return render(request, 'name.html', {'form': form})

만약 우리가 **GET** 요청으로 이 뷰에 도착한다면, 이것은 빈 폼 인스턴스를 생성하고, 그것을 렌더링 할 템플릿 context에 배치할 것입니다.

**POST** 요청을 사용해 폼을 제출할 경우, 뷰는 다시 한 번 폼 인스턴스를 만들어 요청 요청으로부터 데이터를 받습니다: **form = NameForm(request.POST)** 이것은 "binding data to the form" 이라 부르고 이제 그것은 bound된 폼입니다.

우리는 이 폼의 **is_valid()** 메서드를 호출합니다; 만약 **True**가 아니라면, 우리는 폼을 갖고 있는 템플릿으로 되돌아갑니다. 이번에는 폼이 더 이상 비어있지 않으므로 HTML 폼이 이전에 제출한 데이터로 채워져 필요에 따라 편집하고 수정할 수 있습니다.

만약 is_valid() 가 True라면, 이제 clean_data attribute에서 모든 검증된 폼 데이터를 찾을 수 있습니다. 우리는 이 데이터를 사용하여 데이터베이스를 업데이트하거나 다른 처리를 한 후에 브라우저로 다음 위치를 알려주는 HTTP 리디렉션을 전송할 수 있습니다. 

### The template

우리의 name.html 템플릿에서 많은 것을 할 필요 없습니다. 가장 간단한 예시입니다:


    <form action="/your-name/" method="post">
        { csrf_token %}
        {{ form }}
        <input type="submit" value="Submit">
    </form>


모든 폼의 필드와 attributes는 장고 템플릿 언어에 의해 **{{ form }}** 인 HTML 마크업 폼으로 생길 것이다.

> 생략

# More about Django Form classes

모든 폼 클래스는 django.forms.Form 또는 django.forms.ModelForm의 하위 클래스입니다. ModelForm은 Form의 하위클래스라고 생각하시면 됩니다. Form과 ModelForm은 실제로 (private) BaseForm 클래스의 공통 기능을 상속하지만, 이 구현의 세부사항은 거의 중요하지 않습니다.

- **Models and Forms**

    만약 당신의 폼이 장고 모델을 직접 추가하거나 편집하는데 사용될 것이라면, ModelForm은 많은 시간과 노력, 코드를 절약해 줍니다. 왜냐하면 이것은 모델 클래스에서 적절한 필드 및 그 attribute와 함께 폼을 만들 것이기 때문입니다. 

## Bound and unbound form instances

Bound 폼과 unbound 폼을 구분하는 것은 매우 중요합니다:

- unbound 폼은 그것과 관련된 데이터가 없습니다. 유저에게 렌더링 될 때, 이는 비어 있거나 디폴트 값을 갖고 있습니다.
- bound 폼은 제출된 데이터를 갖고 있기 때문에, 그 데이터가 유효한지를 말하는데 사용될 수 있습니다. 유효하지 않은 bound 폼이 렌더링 될 때, 이는 유저에게 수정해야 할 데이터를 알려주는 인라인 에러 메세지를 갖고 있습니다.

폼의 **is_bound** attribute가 폼이 **bound**된 데이터를 갖고 있는지 아닌지를 말해줍니다.

> 생략

# Working with form templates

당신의 폼을 템플릿으로 만들기 위해 해야 할 일은 폼 인스턴스를 템플릿 컨텍스트에 배치하는 것입니다. 따라서 폼이 context 안에 있는 **form**으로부터 호출되었을 경우, **{{ form }}**은 **<label>**과 **<input>** 요소를 적절하게 렌더링 합니다.

## Form rendering options

- **Additional form template furniture**

    폼의 출력은 그것을 감싸고 있는 <form> 태그나, 폼의 submit 제어를 포함하지 않습니다. 이들을 직접 제공해야 합니다.

**<label>/<input>** 쌍에 대한 다른 출력 옵션이 있습니다:

- **{{ form.as_table }}**은 **<tr>** 태그로 감싸면서 그들을 렌더링 할 것입니다.
- **{{ form.as_p }}**는 **<p>** 태그로 감싸면서 그들을 렌더링 할 것입니다.
- **{{ form.as_ul }}**는 **<li>** 태그로 감싸면서 그들을 렌더링 할 것입니다.

감싸주는 **<table>**이나 **<ul>** 요소를 직접 제공해야 한다는 점에 유의하십시오.

ContactForm 인스턴스에 대한 {{ form.as_p }} 출력 입니다:

    <p><label for="id_subject">Subject:</label>
        <input id="id_subject" type="text" name="subject" maxlength="100" required></p>
    <p><label for="id_message">Message:</label>
        <textarea name="message" id="id_message" required></textarea></p>
    <p><label for="id_sender">Sender:</label>
        <input type="email" name="sender" id="id_sender" required></p>
    <p><label for="id_cc_myself">Cc myself:</label>
        <input type="checkbox" name="cc_myself" id="id_cc_myself"></p>

> 생략
