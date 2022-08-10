# 设计模式-组合模式


# 设计模式-组合模式

> 将多个对象组合在一起进行操作，常用于表示树形结构中，例如二叉树

![1354243465_5968](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/1354243465_5968.PNG)

## 代码实现

```java
package com.designPattern.compositePattern;

import java.util.ArrayList;
import java.util.List;

/**
 * 组合模式
 * 类似树结构
 * 将多个对象组合在一起进行操作，常用于表示树形结构中，例如二叉树
 */
public class CompositePattern {
    public static void main(String[] args) {
        TreeNode treeNode = new TreeNode("hhh");
        TreeNode treeNode1 = new TreeNode("hhh");
        treeNode.add(treeNode1);
        System.out.println(treeNode);
    }
}
class TreeNode {

    private String name;
    private TreeNode parent;
    private List<TreeNode> children = new ArrayList<>();

    public TreeNode(String name){
        this.name = name;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public TreeNode getParent() {
        return parent;
    }

    public void setParent(TreeNode parent) {
        this.parent = parent;
    }

    //添加孩子节点
    public void add(TreeNode node){
        children.add(node);
    }

    //删除孩子节点
    public void remove(TreeNode node){
        children.remove(node);
    }

    //获取孩子节点
    public List<TreeNode> getChildren() {
        return children;
    }

    @Override
    public String toString() {
        return "TreeNode{" +
                "name='" + name + '\'' +
                ", parent=" + parent +
                ", children=" + children +
                '}';
    }
}
```

**运行结果**

![11](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/11.png)

