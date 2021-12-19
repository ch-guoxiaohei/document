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
