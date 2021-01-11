# 二叉树的遍历

二叉树的遍历是树的算法中最基础的问题。下面是我总结的几种遍历方法

## 递归实现

递归实现的方法比较简单，在不同的位置进行操作即可实现不同的遍历方式

```java
public void recursion(TreeNode root){
    if (root == null) return;
    // 前序遍历
    recursion(root.left);
    // 中序遍历
    recursion(root.right);
    // 后序遍历
}
```

因为每个节点都需要访问一次，且递归一次需要占用一个栈空间，因此时间复杂度为$O(n)$，空间复杂度为$O(n)$

## 非递归实现

在调用函数时，系统会自动地申请一个栈，并在栈中写入调用的位置等信息，以便调用结束后继续执行后面的操作。我们可以使用栈这一数据结构**显式**的存储各节点的信息，来达到与递归调用一样的效果。

### 前序遍历

```java
public void Inorder(Node root){
    if (root==null) return;
    Stack<Node> stack = new Stack<>();
    Node now = root;
    while(now!=null || !stack.isEmpty()){
        while (now!=null){
            System.out.printf("%d ",now.value);
            stack.push(now);
            now = now.left;
        }
        if (!stack.isEmpty()){
            now = stack.pop();
            now = now.right;
        }
    }
}
```

### 中序遍历

中序遍历的实现方式与前序遍历相似，只是把在操作位置从压栈前改到了出栈后。

把显式的压栈与调用函数时的压栈操作关联起来，会发现非递归实现与递归实现的没有太大区别。

```java
public void Preorder(Node root){
    if (root==null) return;
    Stack<Node> stack = new Stack<>();
    Node now = root;
    while(now!=null || !stack.isEmpty()){
        while (now!=null){
            stack.push(now);
            now = now.left;
        }
        if (!stack.isEmpty()){
            now = stack.pop();
            System.out.printf("%d ",now.value);
            now = now.right;
        }
    }
}
```

### 后序遍历

与前序遍历和中序遍历的实现不同，后序遍历的实现这要麻烦许多。因为在递归实现中有两处函数调用，即一个函数里需要进行两次压栈操作。

而在非递归的实现中，只能记录递归实现中的一次函数调用，因此要使用非递归的形式实现后序遍历，需要使用其他方法。

仔细分析后序遍历的操作时间，可以发现后序遍历是在左右节点都已访问的情况下才进行操作，因此只需要记录上一次访问的节点即可完成后序遍历。

```java
public void Postorder(Node root){
    if (root==null) return;
    Stack<Node> stack = new Stack<>();
    Node now = root;
    Node pre = null;// 用于记录上一次访问的节点
    while(now!=null || !stack.isEmpty()){
        while (now!=null){
            stack.push(now);
            now = now.left;
        }
        if (!stack.isEmpty()){
            now = stack.pop();
            if (now.right == null || now.right == pre) {
                System.out.printf("%d ",now.value);
                pre = now;
                now = null;// 为了跳过循环访问左子树的过程
            }
            else {// 还有未遍历的右子树
                stack.push(now);
                now = now.right;
            }
        }
    }
}
```

在非递归遍历的三种方式里，时间和空间复杂度都是$O(n)$，因为非递归遍历是由递归遍历变化而来，所以在复杂度上没有区别。

## Morris算法

Morris算法则是充分的利用了[叶子节点]^(没有子节点的节点)空余的右子树空间，将叶子节点的右子树指向其递归出栈后的节点，来实现不使用额外的空间遍历整个二叉树。

这种方法因为没有使用额外的空间，所以空间复杂度为$O(1)$

```java
public void morris(Node root){
    if (root==null) return;
    Node now = root;
    Node leftMostRight = null;
    while (now!=null){
        if (now.left!=null){
            leftMostRight = now.left;
            while (leftMostRight.right!=null && leftMostRight.right!=now){
                leftMostRight = leftMostRight.right;
            }
            if (leftMostRight.right==null){
                System.out.printf("%d ",now.value);//inorder
                leftMostRight.right = now;
                now = now.left;
            }
            else {
                leftMostRight.right = null;
                //System.out.printf("%d ",now.value);//preorder
                now = now.right;
            }
        }
        else {
            System.out.printf("%d ",now.value);//inorder preorder
            now = now.right;
        }
    }
}
```

## 其他方法

![例图](https://raw.githubusercontent.com/zqrren/img/master/20210111221633.png)

这种方法基于上图所示的遍历方法，绕着二叉树的轮廓一周，每个节点的左边、下边、右边分别是前序遍历，中序遍历和后序遍历。

```java
public void border(Node root){
    Node now = root;
    ArrayList<Integer> inorder = new ArrayList<>();
    ArrayList<Integer> preorder = new ArrayList<>();
    ArrayList<Integer> postorder = new ArrayList<>();
    Stack<Node> stack = new Stack<>();
    stack.push(null);
    while (now != null){
        // 0 父 1 左 2 右
        int from = 0;
        int to = 0;

        Node top = stack.pop();
        // 如果上一个值是左子节点，说明从左子节点返回
        if (top==now.left){ from = 1; }
        // 如果上一个值是右子节点，说明从右子节点返回
        else if(top==now.right){ from = 2; }
        // 如果既不是左子节点也不是右子节点，说明是父节点，压回去
        else { stack.push(top); }

        Node parent = stack.peek();

        // 如果从父节点来，并且有左子节点，往左走
        if(from==0 && now.left!=null){ to = 1; }
        // 如果不是从右节点来，并且有右节点，往右走【从父节点来说明没有左子节点】
        else if(from!=2 && now.right!=null){ to = 2; }
        // 如果既不能往左走也不能往右走，那么只能回父节点，回父节点之前，要记录父节点的位置并且弹出父节点
        else { stack.pop(); }

        // 如果是从父节点来的，添加到前序遍历
        if (from==0){ inorder.add(now.value); }
        // 如果是叶子节点或者从左子节点来，往右子节点去，添加到中序遍历
        if ((from==0 && to==0) || (from==1 && to==2)){ preorder.add(now.value); }
        // 如果返回父节点，添加到后序遍历
        if (to==0){ postorder.add(now.value); }

        // 把当前节点压栈，告诉下一个节点自己从哪来
        stack.push(now);

        if (to==1){ now = now.left; }
        else if (to==2){ now = now.right; }
        else{ now = parent; }
    }
    System.out.println(inorder);
    System.out.println(preorder);
    System.out.println(postorder);
}
```

