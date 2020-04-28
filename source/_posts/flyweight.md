---
title: 享元模式
date: 2019/10/8
categories:
  - 设计模式
  - java
typora-root-url: ../
typora-copy-images-to: ../static
abbrlink: 29384
---



重点是重复利用对象 



类似安卓 的 List组件 RecyclerView



创建一个对象



```java
class Bullet {
    public UUID id = UUID.randomUUID();
    boolean living = true;

    @Override
    public String toString() {
        return "Bullet{" +
                "id=" + id +
                '}';
    }
}

```



维持一个池子里的key 

for循环查看key的标签是否可以重复利用 

如果可以则返回

不可以说明都在用 这个时候需要new出来了

```java
public class BulletPool {
    List<Bullet> bullets = new ArrayList<>();

    {
        for (int i = 0; i < 5; i++) bullets.add(new Bullet());
    }

    public Bullet getBullet() {
        for (int i = 0; i < bullets.size(); i++) {
            Bullet b = bullets.get(i);
            if (!b.living) return b;
        }

        return new Bullet();
    }

   

}
```





main函数实际使用



```java
 public static void main(String[] args) {
        BulletPool bp = new BulletPool();

        for (int i = 0; i < 10; i++) {
            Bullet b = bp.getBullet();
            System.out.println(b);
        }
    }

```















