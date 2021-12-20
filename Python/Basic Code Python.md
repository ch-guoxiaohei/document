# Basic Code Python

[TOC]

## Data Type

- int
- float
- string

### String Operation

- String Join

```code
'A' + 'B' > 'AB'

```

- String Copy

```code
'AB' * 3 > 'ABABAB' copy AB 3 time
```

### Save Value In Variable

```python
variable_name = 'ab'
```

#### Variable Name Rules

- just a vocabulary
- just contains word,num,underline
- Do not start with num

### Comment

- single line comment
  
```python
# this is a python single line comment
```

- mult line comment
  
```python
''' 
   这是多行注释，用三个单引号
   这是多行注释，用三个单引号
   这是多行注释，用三个单引号
'''
```

### Common Function

- print()
print messages to console
- input()
get user input messages
- len()
get length of input.eg: `len('')=0`,`len('bb')=2`
- str()
convert input to str. eg: `str(29)='29'`
- int()
convert input to int. eg: `int('29')=29`
- float()
convert input to float. eg: `fload('29')=29.0`

## Controll Stream

### Bool

- True
- False

### Diff Operate Sign

- <= | >=
- ==
- !=
- < | >

### Bool Operate Sign

- and
- or
- not

### Controll Stream Element

- Condition
- Code Block

```code
   if conditon:
      print('aaa')
```

### Controller Stream Syntax

- if

```python
  if name == 'Alice':
      print(name)
```

- else

```python
  if name == 'Alice':
      print('Hi , Alice')
  else:
    print('Hi')
```

- elif

```python
  if name == 'A':
      print('A')
  elif name == 'B':
      print('B')
```

### While Loop

```python
  spam = 0
  while spam < 5:
      print(spam)
      spam+=1
```

### Break

The key will finish loop code block.

### Continue

The key will finish current loop and go into next loop code.

### For Loop And Range Function

```python
   for i in range(10):
      print(i)

   for i in range(1,10):
      print(i)
   
   for i in range(1,10,2):
      print(i)
```

### Import And From Key

- import: import a module
- From: when you import some module , you can use from {module name} import {details function}

### Sys.exit

The methond will end the program in a advance

## Function

### - define -> `def`

- void

```python
def printParam(name):
   print(name)
```

- return value

```python
def returnValue(name):
   return 'A'
```

- None Value

Python has a spcial value , it is None. It refers to not value.
None is the unique value of None Type(likes java null value).

### Key Params And Print

The Key Params were identified via keys that function called.

```python
   print('Hello' , end='')
   print('World')

   result --- > HelloWorld
```

`print` Function , end param is the key param.

- print('A') --> A
- print('A','B','C') --> ABC
- print('A','B','C',sep=',') ---> A,B,C

### Local Variables And Global Variables

- Local Variables can not use on Global Scope.
- Local Variables can not use other local Variables.
- Global Variables can be read on local scope.
- Avoid use same name local and global variables.

### Try Except

we can use `try except` to catch exception with python.

```python
   def spam(divideBy):
      try:
         return 42 / divideBy
      except ZeroDivisionError as e:
         print(e.args)
         print(str(e))
      finally:
         print("The 'try except' is finished")
```

## List

### Date Type For List

```python
   data = [0,1,2,3]
```

- get vaule with index

```python
   data_value = data[0]
   # 0
   print(data_value) 
```

- negative number index . eg: -1

```python
   data_value = data[-1]
       # 3
   print(data_value) 
```

- get sublist with slice.

```python
    sub_list = data[0:2]
    [0 , 1]
    print(sub_list)
```

- len()

```python
   # 4
   print(len(data))
```

- change list value with index

```python
   data[0] = 9
   # [9,1,2,3,4]
   print(data)
```

- list join and list cop

1. join list with '+'

```python
   data1 = [1,2]
   data2 = [3,4]
   data3 = data1 + data2
   # [1,2,3,4]
   print(data3)
```

2.copy list with '*'

```python
   data = [5,6]
   data_copy = data * 3
   # [5,6,5,6,5,6]
   print(data_copy)
```

- del value with 'del' key

```python
   data = [1,2]
   del data[0]
   # [2]
   print(data)
```

if you delete the value that was not existed , it will throw below error .
**IndexError: list assignment index out of range**

### Use List
