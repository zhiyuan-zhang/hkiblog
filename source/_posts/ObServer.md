---
title: 观察者模式
date: 2019/10/8
categories:
  - 设计模式
  - java
typora-root-url: ../
typora-copy-images-to: ../static
abbrlink: 29384
---











简单说下 后续再改



首先这个模式很常见  类似的有 callback  function ,listener , hook 都是这样的, 也可以说是一个东西



窗口添加事件  

js 回调事件

等等类似的其实本质上都是ObServer模式



首先我们看下main函数运行的代码 

```java

public static void main(String[] args) {
    Button b = new Button();
    b.addActionListener(new MyActionListener());
    b.addActionListener(new MyActionListener2());
    b.addActionListener((e)->{
            System.out.println("ppp");
        });
    b.buttonPressed();
}

```



因为你之前new Button() 所以我们会实现一个button类 

里面会有一个for循环来依次实现你之前添加的事件  

就是buttonPressed 里的 for循环

同时还有一个添加事件的方法

```java
class Button {
	
	private List<ActionListener> actionListeners = new ArrayList<ActionListener>();
	
	public void buttonPressed() {
		ActionEvent e = new ActionEvent(System.currentTimeMillis(),this);
		for(int i=0; i<actionListeners.size(); i++) {
			ActionListener l = actionListeners.get(i);
			l.actionPerformed(e);
		}
	}
	
	public void addActionListener(ActionListener l) {
		actionListeners.add(l);
	}
}

```



接下来我们需要写 事件类的接口  参数是你具体的事件本身



```java 
interface ActionListener {
	public void actionPerformed(ActionEvent e);
}

```





添加事件类接口的实现

```java


class MyActionListener implements ActionListener {

	public void actionPerformed(ActionEvent e) {
		System.out.println("button pressed!");
	}
	
}

class MyActionListener2 implements ActionListener {

	public void actionPerformed(ActionEvent e) {
		System.out.println("button pressed 2!");
	}
	
}
```



然后是刚刚传的参数你也需要定义  这里我们定义的是 事件本身需要的属性

这里有两个  你可以根据自己的业务需求来判断



```java
class ActionEvent {
	
	long when;
	Object source;
	
	public ActionEvent(long when, Object source) {
		super();
		this.when = when;
		this.source = source;
	}
	
	
	public long getWhen() {
		return when;
	}

	public Object getSource() {
		return source;
	}
	
}
```









一般会和责任链模式配合使用 在返回false后执行失败监听函数



