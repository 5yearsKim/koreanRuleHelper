# korean-rule-helper
### 한국어 rule-based 처리할 때 활용할 수 있는 패키지
* 품사 태그를 활용한 문장 구조 및 분석
* 한국어 규칙 적용

## Installation

### dependency
* [python-mecab-ko](https://pypi.org/project/python-mecab-ko/) 패키지 설치(링크 참조)

Using `pip`
```
pip install korean-rule-helper
```

## Usage
<hr/>

### 기본 구성
**KoreanSentence**

*KoreanSentence 는 문장(한국어 string)을 품사태그로 나누어서 저장합니다.*
```python
from korean_rule_helper import KoreanSentence
sentence = KoreanSentence('나는 너를 사랑한다.')
print(sentence.parsed) #[$나/NP, $는/JX, $_, $너/NP, $를/JKO, $_, $사랑/NNG, $한다($하/XSV, $ᆫ다/EF)/XSV+EF, $./SF]
print(sentence.text) # 나는 너를 사랑한다.
```
**Rule**

*rule 은 python dict 형태로 표현되며 형태소의 품사, 표현형을 체크할 수 있습니다. default 값은 아래와 같습니다.*
```python
rule_default = {
    'surface': None, # str | [str]
    'is_first': False, 
    'is_last': False,
    # 'is_concat': False,
    'pos': None, # str | [str]
    '!pos': None, # str | [str]
    'return': False,
}
```
- `suface(str|[str])`: 표현형
- `is_first(boolean)`: 문장의 첫번째로 시작하는지 여부
- `is_last(boolean)`: 문장의 마지막으로 끝나는지 여부
- `pos(str|[str])`: 포함하는 품사
- `!pos(str|[str])`: 포함하지 않는 품사
- `return(boolean)`: argument 로 반납 여부

## KoreanSentence
<hr/>


```python
import KoreanSentence, KoreanRuleHelper
```

### replace
*문장의 형태소를 조건에 맞게 변경합니다.*
```python
# replace
sentence = KoreanSentence('문재인은 19대 대통령이다.')

rule = {'surface': '다', 'pos': 'EF'}
sentence = sentence.replace(rule, '에요')
print(sentence.text) # 문재인은 19대 대통령이에요.


rule = {'pos': 'NNP'}
sentence = sentence.replace(rule, '000')
print(sentence.text) # 000은 19대 대통령이에요.
```

### strip
*문장의 앞 뒤 형태소를 조건에 맞게 잘라냅니다.*
```python
sentence = KoreanSentence('박근혜는 18대 대통령이다.')

rules = [{'surface': '.', 'pos': 'S'}, '박근혜', '는'] # multiple rule
sentence = sentence.strip(rules)
print(sentence.text) #  18대 대통령이다

rule = '다' # single rule
sentence = sentence.strip(rule)
print(sentence.text) #  18대 대통령이 
```


## KoreanRuleHelper
<hr/>

### order_match
*문장이 형태소를 조건에 맞는 순서로 포함하고 있는지 여부를 체크합니다.*

```python
sentence = KoreanSentence('내 이름은 김치야.')

rule = ['내', '이름']
is_match, arg = rh.order_match(sentence, rule)
print(is_match, arg) # True []

rule = ['내', '이름', {'return': True, 'pos': 'N'}] 
is_match, arg = rh.order_match(sentence, rule)
print(is_match, arg) # True ['김치']

rule = ['내', '성별', {'return': True, 'pos': 'N'}]
is_match, arg = rh.order_match(sentence, rule)
print(is_match, arg) # False None 
``` 


### add_josa
*한국어 단어의 종성 여부에 맞추어 조사를 붙여줍니다.*
```
I_GA: 이/가
EUN_NEUN: 은/는
GWA_WA: 과/와
A_YA: 아/야
EUL_REUL: 을/를
RYUL_YUL: 률/율
EURO_RO: 으로/로
I_X: 이/''
```

```python
word = '설빙'
josa_word = rh.add_josa(word, type='EUL_REUL')
print(josa_word) # 설빙을

word = '사과'
josa_word = rh.add_josa(word, type='GWA_WA')
print(josa_word) # 사과와
```



## 품사 태그 비교 표

> [mecab-ko-dict](http://openuiz.blogspot.com/2016/07/mecab-ko-dic.html) 의 태그를 따름(세종 품사 태그)

|대분류|태그|설명|
|---|---|---|
|체언|NNG|일반 명사|
||NNP|고유 명사|
||NNB|의존 명사|
||NR|수사|
||NP|대명사|
|용언|VV|동사|
||VA|형용사|
||VX|보조 용언|
||VCP|긍정 지정사|
||VCN|부정 지정사|
|관형사|MM|관형사|
|부사|MAG|일반 부사|
||MAJ|접속 부사|
|감탄사|IC|감탄사|
|조사|JKS|주격 조사|
||JKC|보격 조사|
||JKG|관형격 조사|
||JKO|목적격 조사|
||JKB|부사격 조사|
||JKV|호격 조사|
||JKQ|인용격 조사|
||JX|보조사|
||JC|접속 조사|
|선어말 어미|EP|선어말 어미|
|어말 어미|EF|종결 어미|
||EC|연결 어미|
||ETN|명사형 전성 어미|
||ETM|관형형 전성 어미|
|접두사|XPN|체언 접두사|
|접미사|XSN|명사 파생 접미사|
||XCV|동사 파생 접미사|
||XSA|형용사 파생 접미사|
|어근|XR|어근|
|부호|SF|마침표, 물음표, 느낌표|
||SE|줄임표 …|
||SSO|여는 괄호 (, [|
||SSC|닫는 괄호 ), ]|
||SC|구분자 , · / :|
||SY|기타 기호|
||SL|외국어|
||SH|한자|
||SN|숫자|