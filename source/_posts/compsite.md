---
title: 组合模式
date: 2019/8/12
categories:
  - 设计模式
  - java
typora-root-url: ../
typora-copy-images-to: ../static
abbrlink: 29384
---



组合模式用来处理树状接口

处理树状结构本质上就是查看当前节点是否包含子节点

所以说只需要判断 **当前节点下是否有子节点** 然后循环遍历

做这个树状结构也只需要 一个节点抽象类  两个节点信息 (一个是下面有节点,一个是没节点的) 





一个抽象类

```java

abstract class Node {
    abstract public void p();
}

```





有两个类继承节点信息  区别就是一个类有子节点 一个没有

  没有子节点的如下图

```java

class LeafNode extends Node {
    String content;
    public LeafNode(String content) {this.content = content;}

    @Override
    public void p() {
        System.out.println(content);
    }
}


```



有子节点的如下如

有子节点 意味着需要维护一个list<Node> 

还需要一个add 方法来添加子节点

```java

class BranchNode extends Node {
    List<Node> nodes = new ArrayList<>();

    String name;
    public BranchNode(String name) {this.name = name;}

    @Override
    public void p() {
        System.out.println(name);
    }

    public void add(Node n) {
        nodes.add(n);
    }
}
```



使用的话就是

添加root节点

然后依次添加 大节点和小节点

``` java

public static void main(String[] args) {

        BranchNode root = new BranchNode("root");
        BranchNode chapter1 = new BranchNode("chapter1");
        BranchNode chapter2 = new BranchNode("chapter2");
        Node r1 = new LeafNode("r1");
        Node c11 = new LeafNode("c11");
        Node c12 = new LeafNode("c12");
        BranchNode b21 = new BranchNode("section21");
        Node c211 = new LeafNode("c211");
        Node c212 = new LeafNode("c212");

        root.add(chapter1);
        root.add(chapter2);
        root.add(r1);
        chapter1.add(c11);
        chapter1.add(c12);
        chapter2.add(b21);
        b21.add(c211);
        b21.add(c212);
  
				// 遍历
        tree(root, 0);

    }


    static void tree(Node b, int depth) {
        for(int i=0; i<depth; i++) System.out.print("--");
        b.p();

        if(b instanceof BranchNode) {
            for (Node n : ((BranchNode)b).nodes) {
                tree(n, depth + 1);
            }
        }
    }

```












