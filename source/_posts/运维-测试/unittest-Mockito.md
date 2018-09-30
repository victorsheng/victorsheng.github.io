---
title: unittest-Mockito
abbrlink: 1427218878
date: 2018-08-10 20:02:17
tags:
categories:
---

# when then
待测试类
```
public class MyList extends AbstractList<String> {
 
    @Override
    public String get(final int index) {
        return null;
    }
    @Override
    public int size() {
        return 1;
    }
}
```

```
//为Mock对象配置简单的返回值
@Test
public final void whenMockReturnBehaviorIsConfigured_thenBehaviorIsVerified() {
    final MyList listMock = Mockito.mock(MyList.class);
    when(listMock.add(anyString())).thenReturn(false);
    final boolean added = listMock.add(randomAlphabetic(6));
    assertThat(added, is(false));
}


//为mock对象的指定方法配置抛出异常

@Test(expected = IllegalStateException.class)
    public final void givenMethodIsConfiguredToThrowException_whenCallingMethod_thenExceptionIsThrown() {
        final MyList listMock = Mockito.mock(MyList.class);
        when(listMock.add(anyString())).thenThrow(IllegalStateException.class);
        listMock.add(randomAlphabetic(6));
}


//多次调用配置行为，第一次调用不抛出异常,第二次调用时抛出异常

@Test(expected = IllegalStateException.class)
public final void givenBehaviorIsConfiguredToThrowExceptionOnSecondCall_whenCallingTwice_thenExceptionIsThrown() {
    final MyList listMock = Mockito.mock(MyList.class);
    when(listMock.add(anyString())).thenReturn(false).thenThrow(IllegalStateException.class);
    listMock.add(randomAlphabetic(6));
    listMock.add(randomAlphabetic(6));
}

//配置mock对象调用真实的方法
@Test
public final void whenMockMethodCallIsConfiguredToCallTheRealMethod_thenRealMethodIsCalled() {
    final MyList listMock = Mockito.mock(MyList.class);
    when(listMock.size()).thenCallRealMethod();
    assertThat(listMock.size(), equalTo(1));
}



```

# Verify
```
//验证Mock对象的简单调用

List<String> mockedList = mock(MyList.class);
mockedList.size();
verify(mockedList).size();

//验证Mock对象的调用次数
List<String> mockedList = mock(MyList.class);
mockedList.size();
verify(mockedList, times(1)).size();

//验证整个Mock对象的都没有被调用
List<String> mockedList = mock(MyList.class);
verifyZeroInteractions(mockedList);

//验证Mock对象的某个方法没有被调用
List<String> mockedList = mock(MyList.class);
verify(mockedList, times(0)).size();

//验证Mock对象的没有意料之外的调用--此案例会失败
List<String> mockedList = mock(MyList.class);
mockedList.size();
mockedList.clear();
verify(mockedList).size();
verifyNoMoreInteractions(mockedList);


//验证调用顺序
List<String> mockedList = mock(MyList.class);
mockedList.size();
mockedList.add("a parameter");
mockedList.clear();
 
InOrder inOrder = Mockito.inOrder(mockedList);
inOrder.verify(mockedList).size();
inOrder.verify(mockedList).add("a parameter");
inOrder.verify(mockedList).clear();


//验证调用没有发生
List<String> mockedList = mock(MyList.class);
mockedList.size();
verify(mockedList, never()).clear();


//验证某个方法至少或者之多被调用的次数
List<String> mockedList = mock(MyList.class);
mockedList.clear();
mockedList.clear();
mockedList.clear();
 
verify(mockedList, atLeast(1)).clear();
verify(mockedList, atMost(10)).clear();


//准确验证方法调用时的参数
List<String> mockedList = mock(MyList.class);
mockedList.add("test");
verify(mockedList).add("test");


//使用flexible和any参数
List<String> mockedList = mock(MyList.class);
mockedList.add("test");
verify(mockedList).add(anyString());



//使用参数捕获器
List<String> mockedList = mock(MyList.class);
mockedList.addAll(Lists.<String> newArrayList("someElement"));
ArgumentCaptor<List> argumentCaptor = ArgumentCaptor.forClass(List.class);
verify(mockedList).addAll(argumentCaptor.capture());
List<String> capturedArgument = argumentCaptor.<List<String>> getValue();
assertThat(capturedArgument, hasItem("someElement"));


```