# Josepfu问题

- 约瑟夫(Josepfu)问题

  > n = 5, 即有5个人
  > k = 1，从1开始报数
  > m = 2，数两下出圈

1. 先创建一个辅助指针helper，事先指向环形链表的最后一个结点。
2. 当小孩报数时，先让first和helper移动k-1次，再让helper和first同时移动m-1次。
3. 这时就可以将first指向的小孩结点出圈
   - first = first.next;
   - helper.next = first;
4. 原来的first就没有引用，就会被垃圾回收机制回收





```java
package cn.akangaroo.datastructure.linkedlist;

public class Josepfu {
    public static void main(String[] args) {
        CircleSingleLinkedList clist = new CircleSingleLinkedList();
        clist.addBoy(5);
        clist.showBoy();
        System.out.println("===================");
        clist.countBoy(1, 2, 5);
    }
}

class CircleSingleLinkedList {
    //创建一个first结点
    private Boy first = null;

    //添加结点，构成一个环形链表
    public void addBoy(int nums) {
        //对nums做一个数据校验
        if (nums < 2) {
            System.out.println("nums的值不正确！");
            return;
        }
        Boy curBoy = null;//辅助变量，帮助我们构建环形链表。

        //使用for循环来创建我们的环形链表
        for (int i = 1; i <= nums; i++) {
            //根据编号创建小孩结点
            Boy boy = new Boy(i);
            //如果是第一个小孩
            if (i == 1) {
                first = boy;
                first.setNext(first);
                curBoy = first;
            } else {
                curBoy.setNext(boy);
                boy.setNext(first);
                curBoy = boy;
            }
        }
    }

    //遍历当前环形链表
    public void showBoy() {
        if (first == null) {
            System.out.println("链表为空！");
            return;
        }
        Boy curBoy = first;
        while (true) {
            System.out.println("小孩的编号" + curBoy.getId());
            if (curBoy.getNext() == first) {
                break;
            }
            curBoy = curBoy.getNext();
        }
    }

    /**
     * 根据用户的输入计算小孩出圈的顺序
     *
     * @param startNum 表示聪第几个小孩开始数数
     * @param countNum 表示数几次
     * @param nums     表示最初有几个小孩
     */
    public void countBoy(int startNum, int countNum, int nums) {
        //先对数据进行校验
        if (first == null || startNum < 1 || startNum > nums) {
            System.out.println("参数输入有误，请重新输入！");
            return;
        }
        //创建一个辅助指针
        Boy helper = first;

        while (true) {
            if (helper.getNext() == first) {//说明helper指向了最后一个结点
                break;
            }
            helper = helper.getNext();
        }
        for (int i = 0; i < startNum - 1; i++) {
            first = first.getNext();
            helper = helper.getNext();
        }
        while (true) {
            if (helper == first) {//说明圈中只有一个结点
                break;
            }
            //让first和helper同时移动countNum-1次
            for (int i = 0; i < countNum - 1; i++) {
                first = first.getNext();
                helper = helper.getNext();
            }
            //这时first指向的结点就是小孩要出圈的结点
            System.out.println("小孩" + first.getId() + "出圈");
            //这时将小孩出圈
            first = first.getNext();
            helper.setNext(first);//
        }
        System.out.println("最后留在圈中的小孩编号是" + first.getId());
    }
}

class Boy {
    private int id;
    private Boy next;

    public Boy(int id) {
        this.id = id;
    }

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public Boy getNext() {
        return next;
    }

    public void setNext(Boy next) {
        this.next = next;
    }
}

```

