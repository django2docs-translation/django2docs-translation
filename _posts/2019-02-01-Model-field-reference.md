---
big-title: "The model layer"
middle-title: "Models"
small-title: "Field types"
author: 
  - 김선규(Kim,seon-kyu)
field:
  - Models
relate:
  - Models
toc: true
toc-head-level-choice: false
#do this if head level choice is true
# toc-head-max:
# toc-head-min:
---

# Model field referenceh

> 원본 링크: [https://docs.djangoproject.com/en/2.1/ref/models/fields/](https://docs.djangoproject.com/en/2.1/ref/models/fields/)

이 문서는 장고가 제공하는 모든 `field options`과 `field types`을 포함한 **`Field`**에 대해 설명하고 있습니다.


**See also**<br><br>만약 내장된 field가 작동하지 않으면 `django-localflavor`(`documentation`)를 시도해 보십시오. 이는 특별한 국가와 문화에 대해서 갖가지 유용한 code의 조각을 가지고 있습니다.<br>또한 당신은 쉽게 당신만의 `custom model fields`를 만들 수 있습니다.
{: .notice--info}

**Note**<br><br>기술적으로, 이런 모델들은 **django.db.models.fields**에 저장되어 있으나 편리함을 위해 그들은 **django.db.models**안에 imported 되어있습니다. 기본적인 명명 관습은 **from django.go import models**를 사용하는 것이며, fields는 **models.<Foo>Field**로 나타나는 것입니다.
{: .notice--warning}

# Field options

아래의 arguments는 모두 field의 타입들입니다. 모든 것은 선택적입니다.

> Field options는 Field type안에 포함되는 설정입니다. 예시: name = CharField(max_length=150, null=False). null은 field option이고, max_length는 필수 arguement입니다

## null

**Field.null**

만약 **True**라면, 장고는 데이터 베이스에 비어 있는 값으로 **NULL**을 저장할 것입니다. 기본적으로 **False**로 설정 되어있습니다.

**CharField** 와 **TextField**와 같은 문자열 기반 field의 경우는 **null**의 사용을 피하는 것이 좋습니다. 문자열을 저장하는 field에 **null=True** 라면, 데이터가 없을 경우 field에는 **NULL**과 빈 문자열 두 값이 모두 저장될 수 있습니다. 같은 값에 대해 두 가지 가능한 값을 갖는 것은 지양해야 합니다. 장고의 관습은 빈 문자열을 저장하는 것입니다.

한 가지 예외는 **CharField**가 **unique=True**와 **blank=True**를 전부 가지고 있는 때입니다. 이는 복수의 객체에서 blank value를 저장할 때 **unique** 조건에 위배되는 것을 피하기 위함입니다. (NULL은 여러 개 가질 수 있고, 빈 문자열은 하나의 값으로 인식되어 unique 때문에 하나만 가질 수 있는 것 같습니다.)

string-based 와 non-string-based field 에서 forms에 빈값을 넣는 것을 원한다면 **blank=True**로 설정해 주어야 합니다. **null** 값은 단지 database storage에 영향을 미치기 때문입니다(**blank**를 보십시오).

**Note**<br><br>Oracle database backend를 사용할 때, **NULL**은 이 attribute와 상관없이 빈 값을 나타내는 것으로 저장될 것입니다. 
{: .notice--warning}


## blank

**Field.blank**

만약 **True** 라면 field는 blank를 허용합니다. 기본적으로 **Flase**입니다.

`null`은 순수하게 database-related 인 반면에 `blank`는 validation-related 입니다. 만약 field가 **blank=True**를 가진다면 form validation은 empty value의 입력을 허용 해줄 것입니다. 만약 field가 **blank=False** 라면 field는 반드시 채워져야 합니다



## choices

**Field.choices**

field에 choices로 사용하기 위해서는 두 items의 iterables 이거나 이를 포함하고 있는 iterable이어야 합니다(e.g. [(A, B), (A, B), ...]). 만약 choice가 주어진다면, 그들은 [model validation](https://docs.djangoproject.com/en/2.1/ref/models/instances/#validating-objects)에 의해 실행되고, text field가 아닌, select box에 선택지가 담긴 기본 위젯 form이 실행 될 것입니다.

각 튜플에서 첫번째 요소는 model에 저장되는 실질적 값이고, 두 번째 요소(element)는 사람이 읽게 되는 이름입니다. 예시:
```python
YEAR_IN_SCHOOL_CHOICES = (
        ('FR', 'Freshman'),
        ('SO', 'Sophomore'),
        ('JR', 'Junior'),
        ('SR', 'Senior'),
    )
```

일반적으로 model class 안에 choices를 각 값에 대해서 변함 없게 알맞은 이름을 정의하는 가장 좋은 방법은 다음과 같습니다.

```python
from django.db import models
    
class Student(models.Model):
    FRESHMAN = 'FR'
    SOPHOMORE = 'SO'
    JUNIOR = 'JR'
    SENIOR = 'SR'
    YEAR_IN_SCHOOL_CHOICES = (
        (FRESHMAN, 'Freshman'),
        (SOPHOMORE, 'Sophomore'),
        (JUNIOR, 'Junior'),
        (SENIOR, 'Senior'),
    )
    year_in_school = models.CharField(
        max_length=2,
        choices=YEAR_IN_SCHOOL_CHOICES,
        default=FRESHMAN,
    )
    
    def is_upperclass(self):
        return self.year_in_school in (self.JUNIOR, self.SENIOR)
```

choices list를 model class의 외부에 정의하여 참조하게 할 수 있지만, model class 내부에 각 choice를 정의하는 것은 이것을 사용한 class에 모든 정보를 보관할 수 있다는 장점이 있습니다. 그리고 choices를 쉽게 참조 할 수 있게 합니다.(e.g. **Student** 모델이 import 된 어느 곳에서라도 **Student.SOPHOMORE**이 작동할 것입니다)

사용 가능한 choices를 체계화 하기 위한 목적으로 그룹에 이름을 지을 수 있습니다.

```python
MEDIA_CHOICES = (
    ('Audio', (
            ('vinyl', 'Vinyl'),
            ('cd', 'CD'),
        )
    ),
    ('Video', (
            ('vhs', 'VHS Tape'),
            ('dvd', 'DVD'),
        )
    ),
    ('unknown', 'Unknown'),
)
```
각 tuple의 첫 번째 요소는 그룹을 나타낼 이름입니다. 두 번째 요소는 두 값의 iterable로, 저장되는 값과 사람이 읽을 수 있는 값으로 묶여진 tuple입니다. 하나의 list를 사용하여 그룹화 된 선택지들과 그룹화 되지 않은 선택지들을 합칠 수 있습니다.(예를 들면, 위에서의 unknown option들은 그룹화 되지 않은 선택지입니다)

choices 을 가진 model field 별로, field의 현재 값에 대응되는 사람에게 보여지는 이름을 받아오는 method를 추가합니다. 더 자세한 것은 database API documentation 안에 있는 `get_FOO_display()`를 참고하십시오.

choices는 그 어떤 iterable object로도 받을 수 있다는 것을 기억하십시오. 굳이 list나 tuple 로 구성할 필요가 없습니다. 이것은 choices를 dynamically하게 만들 수 있다는 의미입니다. 어쩌면 choices를 dynamic 하게 만들 방법을 찾아낼 수도 있지만, 아마 적절한 데이터베이스 테이블을 foreignkey와 함께 사용하는 것이 더 좋을 것입니다. choices는 어차피 크게 바꾸지 않을 정적 데이터이기 때문입니다.

**default**로 **blank=False**가 field에 설정되지 않는 이상 select box label에는 "——————————"가 담겨 제공될 것이다. 이를 override 하기 위해서는 **None**을 가진 choices tuple을 추가하십시오; e.g. **(None, 'Your String For Dispay').** 대안으로 예를 들어 CharField와 같이 적절한 곳에서는 empty string 대신 **None**을 자주 쓸 것입니다.



## db_colum

**Field.db_column**

field에 사용할 database column의 이름입니다. 만약 주어지지 않았다면, 장고는 field의 이름을 사용할 것이다.

만약 database column name 이 SQL reserved word, 또는 파이썬 변수이름에서는 사용할 수 없는 문자열을 포함하고 있어도 - 예를 들어 하이픈 - 장고에서는 사용할 수 있습니다. Django quotes column and table names behind the scenes.



## db_index

**Field.db_index**

만약 True 이면 이 field를 위해 database index가 생성될 것입니다.



## db_tablespaces

**Field.db_tablespace**

만약 field가 indexed 되어 있다면, 필드의 index에 대해 사용할 database tablespace의 이름입니다. 기본 값은 프로젝트의 **DEFAULT_INDEX_TABLESPACE** 설정, 또는 모델의 **db_tablespace**(설정된 경우)입니다. 만약 backend가 tablespace를 지원하지 않는다면 이 옵션은 무시될 것입니다.



## default

**Field.default**

필드에 대한 기본 값입니다. 이는 한 값이나 callable 객체가 될 수도 있습니다. 만약 callable 객체라면 새로운 객체가 생성될 때마다 호출될 것입니다.

default는 변경 가능한 객체(model instance, list, set 등)일 수 없으며, 이 객체의 동일한 인스턴스에 대한 참조는 모든 새로운 모델 인스턴스에서 기본 값으로 사용될 것입니다. 대신에, callable로 원하는 default를 wrap할 수 있습니다. 예를 들어, **JSONField**에 대해 default **dict**을 명시하고 싶다면, 다음 함수를 사용하십시오
:
```python
def contact_default():
    return {"email": "to1@example.com"}
    
contact_info = FSONField("ContactInfo", default=contact_default)
```

**default**와 같은 field option으로 **lamda**는 사용할 수 없습니다. 왜냐하면 이들은 Migrations로 serialized되지 않기 때문입니다. 해당 설명서를 참고하십시오.

모델 인스턴스로 매핑하는 **ForeignKey**와 같은 필드에 대해서는, default는 모델 인스턴스 대신 그들이 참조하는 필드 값(**to_field**가 설정되지 않은 경우 **pk**)이어야 합니다.

> default 값을 모델을 설정하지 말고, 해당 모델의 primary key인 field로 설정해야 한다는 뜻 같습니다.

default(기본 값)은 새로운 모델 인스턴스가 생기고, 해당 필드에 대해 값이 주어지지 않을 경우 사용됩니다. 필드가 primary key일 경우, 필드가 None으로 설정되어 있어도 default를 사용합니다.



## editable

**Field.editable**

만약 **False**라면, 이 field는 admin이나 **ModelForm**에서 표시되지 않을 것입니다. 또한 `model validation`을 생략할 것입니다. 기본적으로 **True**입니다.



## error_messages

**Fields.error_messages**

**error_messages** arguments는 이 field에서 에러가 발생할 때의 기본 메세지를 override합니다. override하고 싶은 에러에 대한 메세지를 dictionary로 넘겨주십시오.

에러 메세지의 key는 **null, blank, invalid, invalid_choice, unique,** 그리고 **unique_for_date**를 포함합니다. 추가적인 error message의 key들은 Field types 섹션에 각각의 필드에 명시돼 있습니다.

에러 메세지들은 종종 form으로 전달되지 않습니다. 모델의 error_messages에 대해 고려해야 할 사항에 대한 문서를 참조하십시오.



## help_text

**Field.help_text**

폼 위젯(form widget)에 표시될 추가적인 도움말입니다. 만약 폼에서 해당 필드를 사용하지 않는다 하더라도 설명하기에 유용합니다.

이 값은 자동으로 생성된 폼에서 HTML을 **help_text**에 원하는 대로 포함할 수 있게 해줍니다. 다음은 예시입니다:

```html
help_text="Please use the following format: <em>YYYY-MM-DD</em>."
```

일반 텍스트와 **django.utils.html.escape()**를 사용하여 HTML 특수 문자를 사용하지 않을 수 있습니다. cross-site scripting attack를 피하기 위해서 신뢰할 수 없는 사용자로부터 온 help text를 확실히 escape 해야 합니다.



## primary_key

**Field.primary_key**

**True**라면, 이 필드는 해당 모델에 primary key입니다.

어떤 모델에도 **primary_key=Ture**를 설정하지 않았다면, 장고는 자동으로 primary key 역할을 할 **AutoField**를 추가할 것입니다. 따라서 기본 primary-key behavior을 override하지 않는 이상, **primary_key=Ture**을 명시할 필요가 없습니다. 자세한 사항은 `Automatic primary key fields`를 보십시오.

**primary_key=True**는 **null=False**와 **unique=True**를 암시합니다. 한 객체에 대해서는 하나의 Primary key만 허용됩니다.

primary key 필드는 읽을 수만 있습니다. 만약 존재하는 객체의 primary key 값을 변경하고 저장한다면, 새로운 객체가 옛날 것 옆에 만들어질 것입니다.



## unique

**Field.unique**

**True**라면, 이 필드는 테이블 전체에 걸쳐 고유해야 합니다.

이는 데이터베이스 레벨에서, model validation에 의해 시행됩니다. 만약 unique 필드에 중복된 값의 객체를 저장하려 한다면, **django.db.IntegrityError**가 모델의 **save()** method에 의해 발생할 것입니다.

이 옵션은 **ManyToManyField**와 **OneToOneField**를 제외하고 모든 필드에서 사용할 수 있습니다.

**unique**가 **True**일 때, **db_index**를 명시할 필요가 없습니다. 왜냐하면 **unique**는 index의 생성을 의미합니다.



## unique_for_date

**Field.unique_for_date**

이 필드가 date field 값에 대해 unique하게 하기 위해서 **DateField** or **DateTimeField**의 이름으로 이 값을 설정하십시오. 

예를 들어, **unique_for_date="pub_date"**를 갖고 있는 **title** field를 갖고 있다고 가정하면, 장고는 **title**과 **pub_date**를 동시에 갖고 있는 레코드의 입력을 허락하지 않을 것입니다.

> pub_date = models.DateField( ... )

> title = models.CharField( ... , unique_for_date="pub_date")

이 설정을 **DateTimeField**를 가리키도록 설정하면, 오직 필드의 날짜 부분만 고려된다는 점을 주의하십시오. 게다가 **USE_TZ**가 **True**라면, 객체가 저장될 때 `현재 시간대`에서 점검(check)이 수행될 것입니다.

이는 **Model.validate_unique()**에 의해 수행되는데, model validation 동안 시행되지만 데이터베이스 레벨에서는 시행되지 않습니다. 만약 **unique_for_date** 제약 조건이 **ModelForm**의 일부가 아닌 필드(예: 필드 중 하나가 제외되거나 **editable=False**)라면, **Model.validate_unique()**는 해당 제약 조건에 대한 유효성(validation) 검사를 생략합니다.



## unique_for_month

**Field.unique_for_month**

**unique_for_date**와 비슷하지만, 달(month)에 대해서 고유한 필드를 요구합니다.



## unique_for_year

**Field.unique_for_year**

**unique_for_date**와 **unique_for_month**와 비슷합니다.



## verbose_name

**Field.verbose_name**

해당 필드에 대해 사람이 읽기 쉬운 이름입니다. 만약 verbose name이 주어지지 않는다면, 장고는 자동적으로 field의 attribute 이름에서 undersocre을 띄어쓰기로 바꾸어 사용할 것입니다. `Verbose field names`를 보십시오.



## validators

**Field.validators**

해당 필드에 대해서 실행할 validator의 list입니다. 자세한 사항은 validator document를 보십시오.

**Registering and fetching lookups**

**Field**는 `lookup registration API`를 구현했습니다. API를 사용하여 필드 클래스에 사용할 수 있는 lookup과 필드에서 lookup을 가져오는 방법을 customize할 수 있다.

# Field types


## AutoField

***class* AutoField(**options)**

사용 가능한 ID에 따라 자동적으로 증가하는 **IntegerField** 입니다. 이를 직접적으로 사용할 필요는 없습니다; 만약 primary key를 따로 명시하지 않는다면 자동으로 모델에 추가될 것입니다. `Automatic primary key fields`를 참고하십시오.

## BigAutoField

***class* BigAutoField*(**options*)**

64bit의 정수입니다. 숫자가 **1**부터 **9223372036854775807** 까지 증가한다는 점을 제외하고는 **AutoField**와 같습니다.

## BigIntegerField

***class* BigIntegerField(***options*)**

64bit의 정수입니다. 숫자가 **-9223372036854775808**에서 **9223372036854775807**까지 증가한다는 점을 제외하고는 **IntegerField**와 같습니다. 이 필드에 대한 기본 폼 위젯은 **TextInput**입니다.

## BinaryField

***class* BinaryField(*max_length=None, **options*)**

원시 바이너리 데이터를 저장하는 필드입니다. **bytes, bytearray, memoryview**를 할당할 수 있습니다.

기본적으로 **BinaryField**는 **eidtable**을 **False**로 설정합니다. 이 경우 ModelForm에 추가될 수 없습니다.

- Changed in Django 2.1:

    이전 버전에서는 **editable**을 **True**로 설정하는 것이 금지되었습니다.

**BinaryField**는 추가적인 optional argument를 갖고 있습니다:

**BinaryField.max_length**

필드의 최대 (문자열) 길이입니다. 이는 Django의 validation에서 **MaxLengthValidator**를 사용하여 검증됩니다.

- **Abusing BinaryField**

    이를 이용하여 데이터베이스 파일을 저장할 수 있다고 생각할 수 있지만, 99%의 경우 그것은 나쁜 디자인이 될 것입니다. 이 필드는 `static files` 처리를 위한 대체물이 아닙니다.

## BooleanField

***class* BooleanField(***options*)**

true/false 필드입니다.

이 필드에 대한 기본 폼 위젯은 **CheckboxInput** 이거나, **null=True**일 경우 **NullBooleanSelect**입니다.

**Field.default**가 설정되지 않는다면 **BooleanField**의 기본 값은 **None**입니다.

- **Changed in Django 2.1:**

    이전 버전에서는, 이 필드의 **null=True**가 불가능 했기 때문에 **NullBooleanField**를 사용해야 했습니다. 이제는 **NullBooleanField**의 사용을 권하지 않는데, 이 항목은 앞으로의 Django 버전에서 deprecated 될 것이기 때문입니다.

## CharField

***class* CharField(*max_length=None, **options*)**

작은 사이즈에서 큰 사이즈의 문자열 입니다.

많은 양의 텍스트에 대해서는 **TextField**를 사용하십시오.

이 필드에 대한 기본 폼 위젯은 **TextInput** 입니다.

**CharField**는 하나의 추가적인 필수 argument가 있습니다.

**CharField.max_length**

이 필드에 대한 최대 (문자열) 길이입니다. 이는 Django의 validation에서 **MaxLengthValidator**를 사용하여 검증됩니다.

- **Note**

    복수의 데이터베이스 backends 에서 가능한 application을 작성하고 있다면, 몇몇의 backend에서는 max_length에 제한이 있다는 것을 아셔야 합니다. 자세한 사항은 `database backend notes`를 참고하십시오.

## DateField

***class* DateField(*auto_now=False, auto_now_add=False, **options*)**

Python **datetime.date** instance로 표현된 날짜입니다. 몇 가지 추가적인 optional argument가 있습니다:

**DateField.auto_now**

자동적으로 객체가 저장될 때마다 필드를 현재 시간으로 설정합니다. "last-modified" timestamps에 유용합니다. *항상* 현재 시간이 사용된다는 것을 기억하십시오; 이는 override 할 수 있는 기본 값이 아닙니다.

이 필드는 **Model.save()**를 호출할 때만 자동적으로 업데이트 됩니다. **QuerySet.update()**와 같이 다른 방법으로 필드를 업데이트 할 때 이 필드는 업데이트 되지 않습니다. 단, 그러한 업데이트에서도 필드에 사용자 지정 값을 받을 수 있습니다.

**DateField.auto_now_add**

객체가 처음 생성될 때 이 필드를 현재 시간으로 설정합니다. 생성에 대한 timestamps에 유용합니다. *항상* 현재 시간이 사용된다는 것을 기억하십시오; 이는 override할 수 있는 기본 값이 아닙니다. 따라서 객체를 생성할 때 이 필드에 한 값을 설정해도 그것은 무시될 것입니다. 만약 이 필드를 수정하고 싶다면, **auto_now_add=True**라고 하는 것 대신에 다음을 설정하십시오:

- **DateField**: from **datetime.date.today()**, **default=date.today**
- **DateTimeField**: from **django.utils.timezone.now()**, **default=timezone.now**

이 필드에 대한 기본 폼 위젯은 **TextInput**입니다. admin은 Javascript calendar를 추가하고, "Today"에 대한 shortcut을 제공할 것입니다. 추가적인 **invalid_date** 에러 메세지 key를 포함합니다.

**auto_now_add**, **auto_now** 및 **default**는 상호 배타적입니다. 이 옵션들의 어떠한 조합도 오류를 일으킬 것입니다.

- **Note**

    현재 구현된 대로, **auto_now** 또는 **auto_now_add**를 **True**로 설정하면 필드의 **editable=False** 그리고 **blank=True**로 설정됩니다.

- **Note**

    **auto_now**와 **auto_now_add** options는 생성되거나 업데이트 될 때의 default timezone의 date를 항상 사용합니다. 만약 다른 것이 필요하다면, **auto_now** 또는 **auto_now_add** 대신에 callable default를 사용하거나 **save()**를 override하는 것을 고려해 보십시오; 또는 **DateField** 대신에 **DateTimeField**를 사용하고 디스플레이 시간에 Datetime에서 date로 변환을 처리하는 방법을 결정하십시오. ㅋ

## DateTimeField

***class* DateTimeField(*auto_now=False, auto_now_add=False, **options*)**

Python **datetime.datetime** instance로 나타낸 날짜와 시간입니다. **DateField**와 같은 추가적인 arguments를 갖고 있습니다.

기본적인 폼 위젯은 single **TextInput**입니다. admin은 Javascript shortcust와 함께 분리된 두 개의 **TextInput** 위젯을 사용합니다.

## DecimalField

***class* DecimalField(*max_digits=None, deciaml_places=None, **options*)**

Python **Decimal** instance로 나타낸 고정 소수점 숫자입니다. 이는 **DecimalValidator**를 사용하여 입력을 검증합니다.

두 개의 필수 arguments를 갖고 있습니다.

**DecimalField.max_digits**

숫자에 허용되는 최대 자릿수입니다. 이 값은 **decimal_places**보다 크거나 같아야 한다는 점에 유의하십시오.

**DecimalField.decimal_places**

숫자의 소수점 아래 자릿수 입니다.

예를 들어, 999를 소수점 아래 2자리까지 나타내기 위해서 다음과 같이 사용합니다:

    models.DecimalField(..., max_digits=5, decimal_places=2)

약 10억의 수를 소수점 아래 10자리까지 나타내기 위해서 다음과 같이 사용합니다:

    models.DecimalField(..., max_digits=19, decimal_places=10)

이 필드의 기본 폼 위젯은 **localize**가 **False**일 때 **NumberInput**이고 그렇지 않으면 **TextInput**입니다.

- **Note**

    **FloatField**와 **DecimalField** classes 사이의 차이점에 대한 정보는 FloatField vs DecimalField를 참고하십시오.

## Duration Field

***class* DurationField(***options*)**

Python **timedelta**의해 모델링 된 시간의 기간을 저장하는 필드입니다. PostgreSQL에서 사용될 경우, 데이터 타입은 **interval**이며 Oracle에서 사용될 경우 **INTERVAL DAY(9) TO SECOND(6)**입니다. 그렇지 않으면 microseconds의 **bigint**가 사용됩니다.

- **Note**

    대부분의 경우 **DurationField**와 관련된 산술은 작동합니다. 그러나 PostgreSQL이외의 모든 데이터베이스에서 **DurationField**의 값과 **DateTimeField** instance의 산술 값(숫자)을 비교하는 것은 작동하지 않을 것입니다.

## EmailField

***class* EmailField(*max_length=254, **options*)**

**EmailValidator**를 사용하여 유효한 이메일 주소인지 확인하는 **Charfield**입니다.

## FileField

***class* FileField(*upload_to=None, max_length=100, **options*)**

파일을 업로드하는 필드입니다.

- **Note**

    **primary_key** argument를 지원하지 않으며, 사용시 에러가 발생합니다.

두 가지 optional argument가 있습니다:

**FileField.upload_to**

이 attribute는 업로드 디렉토리와 파일 이름을 설정하는 방법을 제공하며, 두 가지 방법으로 설정할 수 있습니다. 두 경우 모두 그 값이 **Storage.save()** method로 전달됩니다.

문자열 값을 명시할 경우, 파일 업로드 할 때의 날짜/시간으로 대체되는 **strftime()** formatting을 포함할 수 있습니다. 예시를 보십시오:

    class MyModel(models.Model):
        # file will be uploaded to MEDIA_ROOT/uploads
        upload = models.FileField(upload_to='uploads/')
        # or...
        # file will be saved to MEDIA_ROOT/uploads/2015/01/30
        upload = models.FileField(upload_to='uploads/%Y/%m/%d/')

기본 **FileSystemStorage**를 사용하는 경우, 문자열 값은 **MEDIA_ROOT** 경로에 추가되어 업로드 된 파일이 저장될 로컬 파일 시스템의 위치를 형성합니다. 다른 저장소를 사용하는 경우, 해당 저장소의 설명서를 확인하여 **upload_to**를 처리하는 방법을 확인하십시오.

**upload_to**는 함수와 같은 callable일 수 있습니다. 이것은 파일 이름을 포함한 업로드 경로를 얻기 위해 호출될 것입니다. 이 callable은 두 개의 arguments를 받아들이며 저장소 시스템에 전달될 (forward slashes를 갖고 있는) 유닉스 스타일의 경로를 반환해야 합니다. 두 개의 arguments는 다음과 같습니다:

[Untitled](./-0243efdd-7bc7-49a1-9ae5-00d8b3fca68f.csv)

예시를 보십시오:

    def user_directory_path(instance, filename):
        # file will be uploaded to MEDIA_ROOT/user_<id>/<filename>
        return 'user_{0}/{1}'.format(instance.user.id, filename)
    
    class MyModel(models.Model):
        upload = models.FileField(upload_to=user_directory_path)

**FileField.storage**

파일 저장 및 검색을 처리하는 저장소 객체입니다. 이 객체 제공하는 방법에 대한 자세한 내용은 `Managing files`를 보십시오.

이 필드에 대한 기본 폼 위젯은 **ClearableFileInput**입니다.

모델에서 **FileField** 또는 **ImageField**(아래를 보십시오)를 사용하기 위해서는 몇 가지 단계가 필요합니다:

1. 당신의 settings file에서, Django가 업로드 된 파일들을 저장하기 원하는 디렉토리의 전체 경로로서 **MEDIA_ROOT**를 정의해야 합니다. (성능을 위해서 이 파일들은 데이터베이스에 저장되지 않습니다.) 해당 디렉토리의 base public URL로서 **MEDIA_URL**을 정의하십시오. 웹 서버의 사용자 계정이 이 디렉토리에 쓰기가 가능한지 확인하십시오.
2. 모델에 **FileField** 또는 **ImageField**를 추가하십시오. 그리고 업로드 된 파일에 사용할 서브 디렉토리를 명시하기 위해 **upload_to** 옵션을 정의하십시오.
3. 데이터 베이스에 저장될 모든 것은 (**MEDIA_ROOT**와 관련된) 파일에 대한 경로입니다. 아마 Django가 제공하는 편리한 **url** attribute를 사용하고 싶을 것입니다. 예를 들어, **mug_shot**이라 불리는 **ImageField**의 경우, 다음과 같은 템플릿을 이용하여 절대 경로를 얻을 수 있습니다: **{{ object.mug_shot.url }}**

예를 들어, 당신의 **MEDIA_ROOT** 가 '**/home/media'**로 설정되어 있고, **upload_to**가 **'photos/%Y/%m/%d'** 로 설정되어 있다고 가정합시다. **upload_to**의 **%Y/%m/%d'** 부분은 **strftime()** 포맷입니다; **'%Y'**는 네 자리의 '연도', **'%m'**은 두 자리의 '월', **'%d'**은 두 자리의 '일'입니다. 2007년 1월 15일에 파일을 업로드 할 경우, **/home/media/photos/2007/01/15**의 경로에 저장됩니다.

업로드한 파일의 디스크에 있는 파일 이름 또는 크기를 얻으려면, 각각의 **name**과 **size** attributes를 사용할 수 있습니다; 사용 가능한 attributes와 methods에 대한 자세한 설명은 **File** class를 참조하시고 `Managing files` 가이드를 살펴보십시오.

- **Note**

    파일을 저장하는 것은 데이터베이스에 모델을 저장하는 것의 한 부분이기 때문에, 디스크에서 사용될 실제 파일 이름은 모델이 저장된 후에야 사용할 수 있습니다.

업로드한 파일의 relative URL은 **url** attirbute를 사용하여 얻을 수 있습니다. 이는 내부적으로 **Storage** class의 **url()** method를 호출합니다.

보안상의 허점을 피하기 위해서는 업로드한 파일들을 처리할 때마다 그들의 업로드 위치와 파일 형식에 세심한 주의를 기울여야 합니다. 업로드 된 모든 파일의 유효성 검사를 진행하여 파일이 당신이 생각하는 그대로 있는지 확인하십시오. 예를 들어, 유효성 검사 없이 다른 사용자들이 당신의 웹 서버 document root안의 디렉토리에 파일을 업로드하게 한다면, 누군가가 당신의 사이트의 URL을 방문해서 CGI나 PHP 스크립트를 업로드하고 그 스크립트를 실행할 수 있게 됩니다. 이를 허용하지 마십시오.

또한 업로드한 HTML 파일들도 (서버를 통해서가 아니라) 브라우저에 의해 실행될 수 있기 때문에, XSS 또는 CSRF 공격에 준하는 보안 위협을 가할 수 있다는 점에 주의하십시오.

**FileField** instance는 기본 최대 길이(default maximum length)가 100자인 **varchar** column으로서 데이터베이스에 생성됩니다. 다른 필드와 마찬가지로 **max_length** argument를 사용하여 이를 바꿀 수 있습니다.

## FileField and FieldFile

***class* FieldFile**

모델의 **FileField**에 엑세스할 때, 기본 파일에 엑세스하기 위한 proxy로서 **FieldFile** instance가 주어집니다.

**FieldFile**의 API는 **File**의 API을 반영하지만, 한 개의 주요 차이가 있습니다; 클래스에 의해 wrap 된 object가 반드시 Python의 built-in file object의 wrapper는 아닙니다. 대신, **Storage.open()** method의 결과로 둘러싸인 wrapper인데, 이는 **File** object일 수 있고 또는 **File** API의 custom storage's implementation일 수 있습니다. 

**File**에서 상속받은 **read(), write()**와 같은 API 이외에도, **FieldFile**은 기본 파일들과 상호작용할 수 있는 몇 가지 methods가 있습니다.

- **Warning**

    이 클래스의 두 methods인 **save()**와 **delete()**는 데이터베이스에 **FieldFile**과 연관된 모델 개체를 저장하는데 기본적으로 사용됩니다.

**FieldFile.name**

관련 FileField의 Storage의 root로부터의 상대 경로를 포함한 파일 이름.

**FieldFile.url**

기본 **Stroage** class의 **url()** method를 호출하여 파일의 상대적 URL에 엑세스하는 읽기 전용 속성.

**FieldFile.open(*mode='rb'*)**

특정 **mode**로 이 instance와 관련된 파일을 열거나 다시 열어줍니다. 표준 Python **open()** method와 달리, 이는 파일 설명자를 반환하지 않습니다.

**FieldFile.close()**

Python **file.close()**와 같이 작동하며 이 instance와 관련된 파일을 닫습니다.

**FieldFile.save(*name, content, save=True*)**

이 method는 파일 이름과 내용을 인자로 받으며 이들을 필드에 대한 storage class로 전달합니다. 그리고 저장된 파일과 모델 필드를 연결 짓습니다. 모델의 **FileField** instance와 직접 파일 데이터를 연결하고 싶은 경우, **save()** method를 사용하여 해당 파일 데이터를 유지하십시오.

....

## FloatField

***class* FloatField(***options*)**

Python **float** instance에 의해 표현된 부동소수점입니다.

이 필드의 기본 폼 위젯은 **localize**가 **False**일 경우 **NumberInput**이고 아닐 경우 **TextInput**입니다.

- **FloatField vs. DecimalField**

    **FloatField** class는 가끔 **DecimalField** class와 혼용됩니다. 이들 모두 실수를 나타내지만, 그들은 다르게 숫자를 표현합니다. **FloatField**는 내부적으로 Python의 **float** type을 사용하는 반면, **DecimalField**는 Python의 **Decimal** type을 사용합니다. 더 자세한 차이를 알고 싶다면, **decimal** module에 대한 Python documentation을 보십시오.

## ImageField

***class* ImageField(*upload_to=None, height_field=None, width_field=None, max_length=100, **options*)**

...

## IntegerField

***class* IntegerField(***options*)**

정수입니다. **-2147483648**부터 **2147483647**까지의 값은 Django가 지원하는 모든 데이터베이스에서 안전합니다.

**MinValueValidator**와 **MaxValueValidator**를 사용하여 기본 데이터베이스가 지원하는 값을 기반으로 입력을 검증한다.

이 필드에 대한 기본 폼 위젯은 **localize**일 때 **False**이고, 그렇지 않으면 **TextInput**입니다.

## GenericIPAddressField

***class* GenericIPAddressField(*protoco='both', unpack_ipv4=False, **options*)**

문자열 형식(e.g. **192.0.2.30** or **2a02:42fe::4**)의 IPv4 또는 IPv6 주소입니다. 이 필드의 기본 폼 위젯은 **TextInput**입니다.

IPv6 주소 normalization은 RFC [4291#section-2.2](https://tools.ietf.org/html/rfc4291.html#section-2.2)를 따릅니다. 여기에는 **::ffff:192.0.2.0** 와 같은 해당 섹션의 제 3문단에서 제시한 IPv4 포맷이 포함됩니다. 예를 들어 **2001:0::0:01**는 **2001::1**으로 normalize되고,  **::ffff:0a0a:0a0a**는 **::ffff:10.10.10.10**으로 됩니다. 모든 문자는 소문자로 변환됩니다.

**GenericIPAddressField.protocol**

특정 프로토콜로 유효한 입력을 제한합니다. 가능한 값은 **'both'**(default)이거나 **'IPv4'** 또는 **'IPv6'**입니다. 매칭은 대소문자를 구분하지 않습니다.

**GenericIPAddressField.unpack_ipv4**

**::ffffff:192.0.2.1**과 같은 매핑된 IPv4 주소를 풉니다. 이 옵션을 활성화하면 해당 주소가 **192.0.2.1**로 풀리게 됩니다.Default는 비활성화됩니다. **protocol**이 **'both'**로 설정되어 있을 때만 사용할 수 있다.

## NullBooleanField

***class* NullBooleanField(***options*)**

**null=True**인 **BooleanField**와 같습니다. Django 미래 버전에서 이 필드는 deprecated 될 것이므로 **BooleanField**를 사용하십시오.

## PositiveIntegerField

***class* PositiveIntegerField(***options*)**

**IntegerField**와 같지만, **0** 이거나 양수여야 합니다. **0**부터 **2147483647**까지의 값은 Django가 지원하는 모든 데이터베이스에서 안전합니다. **0**의 값은 역호환성 이유로 허가됩니다(*The value 0 is accepted for backward compatibility reasons*).

## PositiveSmallIntegerField

***class* PositiveSmallIntegerField(***options*)**

**PositiveIntegerField**와 비슷하지만, 특정 (데이터베이스에 따라 달라지는) 포인트 이하의 값만 허용한다. **0**에서 **32767**까지의 값은 Django가 지원하는 모든 데이터베이스에서 안전하다.

## SlugField

***class* SlugField(*max_length=50, **options*)**

Slug는 신문용어입니다. slug는 글자, 숫자, 밑줄 또는 하이픈만 포함하는 어떤 것의 짧은 꼬리표입니다. 이들은 일반적으로 URL에서 사용됩니다.

**CharField**처럼 **max_length**를 지정할 수 있습니다(**CharField**에 있는 데이터베이스 호환성에 대한 note와 **max_length**에 대하여 읽어보십시오). **max_length**이 명시되지 않은 경우 Django는 기본적으로 50으로 설정합니다.

이는 **Field.db_index**를 **True**로 설정하는 것을 암시합니다. 

몇몇 다른 값에 기초해 자동으로 SlugField를 미리 생성하여 사용하는 것이 종종 유용합니다. 이를 admin에서 **prepopuluate_fields**를 사용하여 자동적으로 할 수 있습니다.

**SlugField.allow_unicode**

**True**일 경우 이 필드는 ASCII 문자 외에 Unicode 문자를 허용합니다. 기본값은 **False**입니다.

## SmallIntegerField

***class* SmallIntegerField(***options*)**

**IntegerField**와 비슷하지만, 특정 (데이터베이스에 따라 달라지는) 포인트 이하의 값만 허용한다. **-32768** ~ **32767**의 값은 장고에서 지원하는 모든 데이터베이스에서 안전하다.

## TextField

***class* TextField(***options*)**

큰 텍스트 필드입니다. 이 필드에 대한 기본 폼 위젯은 **Textarea**입니다.

만약 **max_length** attribute를 지정하면 자동으로 생성되는 폼 필드의 **Textarea** 위젯에 반영됩니다. 그러나 이는 모델이나 데이터베이스 레벨에서 시행되지는 않습니다. 그것을 위해서는 **CharField**를 사용하십시오.

## TimeField

***class* TimeField(*auto_now=False, auto_now_add=False, **options*)**

Python의 **datetime.time** instance로 표현된 시간입니다. **DateField**와 같은 auto와 관련된 옵션이 있습니다.

이 필드에 대한 기본 폼 위젯은 **TextInput**입니다. admin은 몇개의 Javascript shortcut을 추가합니다.

## URLField

***class* URLField(*max_length=200, **options*)**

URL에 대한 **CharField**입니다. **URLValidator**에 의해 검사됩니다.

이 필드에 대한 기본 폼 위젯은 **TextInput**입니다.

**CharField**의 자식클래스와 같이, **URLField** 역시 optional **max_length** argument를 받아들입니다.
이를 명시하지 않을 경우 기본적으로 200이 사용됩니다.

## UUIDField

***class* UUIDField(***options*)**

보편적으로 고유한 식별자(*identifiers*)를 저장하기 위한 필드입니다. Python의 **UUID** class를 사용합니다. PostgreSQL에서 사용할 때, 이 데이터 유형은 **uuid** 데이터 형식으로 저장되며, 그렇지 않을 경우 **char(32)** 형식입니다.

일반적으로 고유한 식별자는 **primary_key의** **AutoField**에 대한 좋은 대안입니다. 데이터베이스가 **UUID**를 자동으로 생성하지 않으므로, **default**을 사용하는 것을 추천합니다:

    import uuid
    from django.db import models
    
    class MyUUIDModel(models.Model):
        id = models.UUIDField(primary_key=True, default=uuid.uuid4, editable=False)
        # other fields

**UUID** instance가 아니라 (괄호가 생략된 상태의) callable이 **default**로 전달되었다는 점에 유의하십시오.
