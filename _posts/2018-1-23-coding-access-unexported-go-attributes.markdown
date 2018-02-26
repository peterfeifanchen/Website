---
layout: post
title:  "Access Unexported Go Attributes"
date:   2018-1-23 15:40:56 -0800
categories: systems
tags: [ Dabbling ]
---

<h1>Overview</h1>

Golang uses capitalized variable names to indicate exported 
variables that can be referenced from other go packages.  

```go
type TestType struct {
	A int
	b int
}
```
For example in the code above, <span class="variable">A</span> can 
be referenced in other <span class="name">Go</span> packages, but 
<span class="variable">b</span> cannot be. However, hope is not lost 
if you are adamant that a specific variable name remain uncapitalized, 
and yet you want to be able to use it outside of its package. For that 
we can turn to the <span class="name">reflect</span> package. 

<h1>Reading unexported attributes</h1>

```go
t := <pkg name>.TestType{}
v := reflect.ValueOf(t)
fmt.Println(v.FieldByName("b").Int())
```
To read unexported attributes, we use reflection and access its value by field name. 

<h1>Writing to unexported attributes</h1>

Writing to unexported variables is a bit more complicated. 
We need to get a pointer to the underlying 
value in order to change it directly. We cannot use the methods 
in <span class="name">reflect.Value</span> 
since the unexported attribute will not have its 
<span class="name">CanSet</span> flag set. 
Therefore, we convert <span class="name">reflect.Value</span> into 
a <span class="name">unsafe.Pointer</span>. However, the actually pointer 
is stored in the <span class="variable">ptr</span> attribute of
<span class="name">unsafe.Pointer</span>, which is unexported. 
In this case, we cannot use our previous 
method <span class="name">(Value) FieldByName</span> as we now have 
<span class="name">unsafe.Pointer</span>, and not a 
<span class="name">reflect.Value</span>. Therefore,
we create a shadow local type that is a copy of 
<span class="name">unsafe.Pointer</span>. This way we 
gain a copy of the pointer to the underlying value and now can use 
it to change the underlying value.
Voila!


```go
t := <pkg name>.TestType{}
v := reflect.ValueOf(t)
value := v.FieldByName("b")
ptr := unsafe.Pointer(&value)
rv := (*struct {
	typ  unsafe.Pointer
	ptr  unsafe.Pointer
	flag uintptr
	})(ptr)
*(*int)(rv.ptr) = <new int value>
```
