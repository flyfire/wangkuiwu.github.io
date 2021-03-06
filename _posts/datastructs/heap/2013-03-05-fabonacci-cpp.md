---
layout: post
title: "斐波那契堆(二)之 C++语言详解"
description: "heap"
category: datastructure
tags: [heap]
date: 2013-03-05 12:02
---
 
> 上一章介绍了斐波那契堆的基本概念，并通过C语言实现了斐波那契堆。本章是斐波那契堆的C++实现。

> **目录**  
[第1部分 斐波那契堆的介绍](#anchor1)  
[第2部分 斐波那契堆的基本操作](#anchor2)  
[第3部分 斐波那契堆的C++实现(完整源码)](#anchor3)  


 
<a name="anchor1"></a>
# 第1部分 斐波那契堆的介绍

斐波那契堆(Fibonacci heap)是堆中一种，它和二项堆一样，也是一种可合并堆；可用于实现合并优先队列。斐波那契堆比二项堆具有更好的平摊分析性能，它的合并操作的时间复杂度是O(1)。

与二项堆一样，它也是由一组堆最小有序树组成，并且是一种可合并堆。  
与二项堆不同的是，斐波那契堆中的树不一定是二项树；而且二项堆中的树是有序排列的，但是斐波那契堆中的树都是有根而无序的。

![img](/media/pic/datastruct_algrithm/heap/fibonacci/01.jpg)

 
<a name="anchor2"></a>
# 第2部分 斐波那契堆的基本操作

## 1. 基本定义

    template <class T>
    class FibNode {
        public:
            T key;                // 关键字(键值)
            int degree;            // 度数
            FibNode<T> *left;    // 左兄弟
            FibNode<T> *right;    // 右兄弟
            FibNode<T> *child;    // 第一个孩子节点
            FibNode<T> *parent;    // 父节点
            bool marked;        // 是否被删除第一个孩子

            FibNode(T value):key(value), degree(0), marked(false), 
                left(NULL),right(NULL),child(NULL),parent(NULL) {
                key    = value;
                degree = 0;
                marked = false;
                left   = this;
                right  = this;
                parent = NULL;
                child  = NULL;
            }
    };

FibNode是斐波那契堆的节点类，它包含的信息较多。key是用于比较节点大小的，degree是记录节点的度，left和right分别是指向节点的左右兄弟，child是节点的第一个孩子，parent是节点的父节点，marked是记录该节点是否被删除第1个孩子(marked在删除节点时有用)。

    template <class T>
    class FibHeap {
        private:
            int keyNum;         // 堆中节点的总数
            int maxDegree;      // 最大度
            FibNode<T> *min;    // 最小节点(某个最小堆的根节点)
            FibNode<T> **cons;    // 最大度的内存区域

        public:
            FibHeap();
            ~FibHeap();

            // 新建key对应的节点，并将其插入到斐波那契堆中
            void insert(T key);
            // 移除斐波那契堆中的最小节点
            void removeMin();
            // 将other合并到当前堆中
            void combine(FibHeap<T> *other);
            // 获取斐波那契堆中最小键值，并保存到pkey中；成功返回true，否则返回false。
            bool minimum(T *pkey);
            // 将斐波那契堆中键值oldkey更新为newkey
            void update(T oldkey, T newkey);
            // 删除键值为key的节点
            void remove(T key);
            // 斐波那契堆中是否包含键值key
            bool contains(T key);
            // 打印斐波那契堆
            void print();
            // 销毁
            void destroy();

        private:
            // 将node从双链表移除
            void removeNode(FibNode<T> *node);
            // 将node堆结点加入root结点之前(循环链表中)
            void addNode(FibNode<T> *node, FibNode<T> *root);
            // 将双向链表b链接到双向链表a的后面
            void catList(FibNode<T> *a, FibNode<T> *b);
            // 将节点node插入到斐波那契堆中
            void insert(FibNode<T> *node);
            // 将"堆的最小结点"从根链表中移除，
            FibNode<T>* extractMin();
            // 将node链接到root根结点
            void link(FibNode<T>* node, FibNode<T>* root);
            // 创建consolidate所需空间
            void makeCons();
            // 合并斐波那契堆的根链表中左右相同度数的树
            void consolidate();
            // 修改度数
            void renewDegree(FibNode<T> *parent, int degree);
            // 将node从父节点parent的子链接中剥离出来，并使node成为"堆的根链表"中的一员。
            void cut(FibNode<T> *node, FibNode<T> *parent);
            // 对节点node进行"级联剪切"
            void cascadingCut(FibNode<T> *node) ;
            // 将斐波那契堆中节点node的值减少为key
            void decrease(FibNode<T> *node, T key);
            // 将斐波那契堆中节点node的值增加为key
            void increase(FibNode<T> *node, T key);
            // 更新斐波那契堆的节点node的键值为key
            void update(FibNode<T> *node, T key);
            // 在最小堆root中查找键值为key的节点
            FibNode<T>* search(FibNode<T> *root, T key);
            // 在斐波那契堆中查找键值为key的节点
            FibNode<T>* search(T key);
            // 删除结点node
            void remove(FibNode<T> *node);
            // 销毁斐波那契堆
            void destroyNode(FibNode<T> *node);
            // 打印"斐波那契堆"
            void print(FibNode<T> *node, FibNode<T> *prev, int direction);
    };

FibHeap是斐波那契堆对应的类。min是保存当前堆的最小节点，keyNum用于记录堆中节点的总数，maxDegree用于记录堆中最大度，而cons在删除节点时来暂时保存堆数据的临时空间。下面是斐波那契堆的属性结构图和内存结构图的对比示例。

![img](/media/pic/datastruct_algrithm/heap/fibonacci/02.jpg)

从中可以看出，斐波那契堆是由一组最小堆组成，这些最小堆的根节点组成了双向链表(后文称为"根链表")；斐波那契堆中的最小节点就是"根链表中的最小节点"！

PS. 上面这幅图的结构和测试代码中的"基本信息"测试函数的结果是一致的；你可以通过测试程序来亲自验证！

 

## 2. 插入操作

插入操作非常简单：插入一个节点到堆中，直接将该节点插入到"根链表的min节点"之前即可；若被插入节点比"min节点"小，则更新"min节点"为被插入节点。

![img](/media/pic/datastruct_algrithm/heap/fibonacci/03.jpg)

上面是插入操作的示意图。

斐波那契堆的根链表是"双向链表"，这里将min节点看作双向联表的表头(后文也是如此)。在插入节点时，每次都是"将节点插入到min节点之前(即插入到双链表末尾)"。此外，对于根链表中最小堆都只有一个节点的情况，插入操作就很演化成双向链表的插入操作。

此外，插入操作示意图与测试程序中的"插入操作"相对应，感兴趣的可以亲自验证。

 

    插入操作代码

    /*
     * 将node堆结点加入root结点之前(循环链表中)
     *   a …… root
     *   a …… node …… root
    */
    template <class T>
    void FibHeap<T>::addNode(FibNode<T> *node, FibNode<T> *root)
    {
        node->left        = root->left;
        root->left->right = node;
        node->right       = root;
        root->left        = node;
    }
     
    /*
     * 将节点node插入到斐波那契堆中
     */
    template <class T>
    void FibHeap<T>::insert(FibNode<T> *node)
    {
        if (keyNum == 0)
            min = node;
        else
           {
            addNode(node, min);
            if (node->key < min->key)
                min = node;
        }
        keyNum++;
    }

 

## 3. 合并操作

合并操作和插入操作的原理非常类似：将一个堆的根链表插入到另一个堆的根链表上即可。简单来说，就是将两个双链表拼接成一个双向链表。

![img](/media/pic/datastruct_algrithm/heap/fibonacci/04.jpg)

上面是合并操作的示意图。该操作示意图与测试程序中的"合并操作"相对应！

 

合并操作代码

    /*
     * 将双向链表b链接到双向链表a的后面
     *
     * 注意： 此处a和b都是双向链表
     */
    template <class T>
    void FibHeap<T>::catList(FibNode<T> *a, FibNode<T> *b)
    {
        FibNode<T> *tmp;

        tmp            = a->right;
        a->right       = b->right;
        b->right->left = a;
        b->right       = tmp;
        tmp->left      = b;
    }

      
    /*
     * 将other合并到当前堆中
     */
    template <class T>
    void FibHeap<T>::combine(FibHeap<T> *other)
    {
        if (other==NULL)
            return ;

        if(other->maxDegree > this->maxDegree)
            swap(*this, *other);

        if((this->min) == NULL)                // this无"最小节点"
        {
            this->min = other->min;
            this->keyNum = other->keyNum;
            free(other->cons);
            delete other;
        }
        else if((other->min) == NULL)           // this有"最小节点" && other无"最小节点"
        {
            free(other->cons);
            delete other;
        }                                       // this有"最小节点" && other有"最小节点"
        else
        {
            // 将"other中根链表"添加到"this"中
            catList(this->min, other->min);

            if (this->min->key > other->min->key)
                this->min = other->min;
            this->keyNum += other->keyNum;
            free(other->cons);
            delete other;
        }
    }

 

## 4. 取出最小节点

抽取最小结点的操作是斐波那契堆中较复杂的操作。  
(1）将要抽取最小结点的子树都直接串联在根表中；  
(2）合并所有degree相等的树，直到没有相等的degree的树。

![img](/media/pic/datastruct_algrithm/heap/fibonacci/05.jpg)

上面是取出最小节点的示意图。图中应该写的非常明白了，若有疑问，看代码。

此外，该操作示意图与测试程序中的"删除最小节点"相对应！有兴趣的可以亲自验证。

 

取出最小节点代码

    /*
     * 将"堆的最小结点"从根链表中移除，
     * 这意味着"将最小节点所属的树"从堆中移除!
     */
    template <class T>
    FibNode<T>* FibHeap<T>::extractMin()
    {
        FibNode<T> *p = min;

        if (p == p->right)
            min = NULL;
        else
        {
            removeNode(p);
            min = p->right;
        }
        p->left = p->right = p;

        return p;
    }
     
    /*
     * 将node链接到root根结点
     */
    template <class T>
    void FibHeap<T>::link(FibNode<T>* node, FibNode<T>* root)
    {
        // 将node从双链表中移除
        removeNode(node);
        // 将node设为root的孩子
        if (root->child == NULL)
            root->child = node;
        else
            addNode(node, root->child);

        node->parent = root;
        root->degree++;
        node->marked = false;
    }
     
    /* 
     * 创建consolidate所需空间
     */
    template <class T>
    void FibHeap<T>::makeCons()
    {
        int old = maxDegree;

        // 计算log2(keyNum)，"+1"意味着向上取整！
        // ex. log2(13) = 3，向上取整为3+1=4。
        maxDegree = (log(keyNum)/log(2.0)) + 1;
        if (old >= maxDegree)
            return ;

        // 因为度为maxDegree可能被合并，所以要maxDegree+1
        cons = (FibNode<T> **)realloc(cons, 
                sizeof(FibHeap<T> *) * (maxDegree + 1));
    }

    /* 
     * 合并斐波那契堆的根链表中左右相同度数的树
     */
    template <class T>
    void FibHeap<T>::consolidate()
    {
        int i, d, D;
        FibNode<T> *x, *y, *tmp;

        makeCons();//开辟哈希所用空间
        D = maxDegree + 1;

        for (i = 0; i < D; i++)
            cons[i] = NULL;
     
        // 合并相同度的根节点，使每个度数的树唯一
        while (min != NULL)
        {
            x = extractMin();                // 取出堆中的最小树(最小节点所在的树)
            d = x->degree;                    // 获取最小树的度数
            // cons[d] != NULL，意味着有两棵树(x和y)的"度数"相同。
            while (cons[d] != NULL)
            {
                y = cons[d];                // y是"与x的度数相同的树" 
                if (x->key > y->key)        // 保证x的键值比y小
                    swap(x, y);

                link(y, x);    // 将y链接到x中
                cons[d] = NULL;
                d++;
            }
            cons[d] = x;
        }
        min = NULL;
     
        // 将cons中的结点重新加到根表中
        for (i=0; i<D; i++)
        {
            if (cons[i] != NULL)
            {
                if (min == NULL)
                    min = cons[i];
                else
                {
                    addNode(cons[i], min);
                    if ((cons[i])->key < min->key)
                        min = cons[i];
                }
            }
        }
    }
     
    /*
     * 移除最小节点
     */
    template <class T>
    void FibHeap<T>::removeMin()
    {
        if (min==NULL)
            return ;

        FibNode<T> *child = NULL;
        FibNode<T> *m = min;
        // 将min每一个儿子(儿子和儿子的兄弟)都添加到"斐波那契堆的根链表"中
        while (m->child != NULL)
        {
            child = m->child;
            removeNode(child);
            if (child->right == child)
                m->child = NULL;
            else
                m->child = child->right;

            addNode(child, min);
            child->parent = NULL;
        }

        // 将m从根链表中移除
        removeNode(m);
        // 若m是堆中唯一节点，则设置堆的最小节点为NULL；
        // 否则，设置堆的最小节点为一个非空节点(m->right)，然后再进行调节。
        if (m->right == m)
            min = NULL;
        else
        {
            min = m->right;
            consolidate();
        }
        keyNum--;

        delete m;
    }

 

## 5. 减小节点值


减少斐波那契堆中的节点的键值，这个操作的难点是：如果减少节点后破坏了"最小堆"性质，如何去维护呢？下面对一般性情况进行分析。

(1) 首先，将"被减小节点"从"它所在的最小堆"剥离出来；然后将"该节点"关联到"根链表"中。 倘若被减小的节点不是单独一个节点，而是包含子树的树根。则是将以"被减小节点"为根的子树从"最小堆"中剥离出来，然后将该树关联到根链表中。  
(2) 接着，对"被减少节点"的原父节点进行"级联剪切"。所谓"级联剪切"，就是在被减小节点破坏了最小堆性质，并被切下来之后；再从"它的父节点"进行递归级联剪切操作。  
  &nbsp;&nbsp;&nbsp;&nbsp; 而级联操作的具体动作则是：若父节点(被减小节点的父节点)的marked标记为false，则将其设为true，然后退出。  
  &nbsp;&nbsp;&nbsp;&nbsp; 否则，将父节点从最小堆中切下来(方式和"切被减小节点的方式"一样)；然后递归对祖父节点进行"级联剪切"。  
  &nbsp;&nbsp;&nbsp;&nbsp; marked标记的作用就是用来标记"该节点的子节点是否有被删除过"，它的作用是来实现级联剪切。而级联剪切的真正目的是为了防止"最小堆"由二叉树演化成链表。  
(3) 最后，别忘了对根链表的最小节点进行更新。

![img](/media/pic/datastruct_algrithm/heap/fibonacci/06.jpg)

上面是减小节点值的示意图。该操作示意图与测试程序中的"减小节点"相对应！

 

减小节点值的代码

    /* 
     * 修改度数
     */
    template <class T>
    void FibHeap<T>::renewDegree(FibNode<T> *parent, int degree)
    {
        parent->degree -= degree;
        if (parent-> parent != NULL)
            renewDegree(parent->parent, degree);
    }
     
    /* 
     * 将node从父节点parent的子链接中剥离出来，
     * 并使node成为"堆的根链表"中的一员。
     */
    template <class T>
    void FibHeap<T>::cut(FibNode<T> *node, FibNode<T> *parent)
    {
        removeNode(node);
        renewDegree(parent, node->degree);
        // node没有兄弟
        if (node == node->right) 
            parent->child = NULL;
        else 
            parent->child = node->right;

        node->parent = NULL;
        node->left = node->right = node;
        node->marked = false;
        // 将"node所在树"添加到"根链表"中
        addNode(node, min);
    }

    /* 
     * 对节点node进行"级联剪切"
     *
     * 级联剪切：如果减小后的结点破坏了最小堆性质，
     *     则把它切下来(即从所在双向链表中删除，并将
     *     其插入到由最小树根节点形成的双向链表中)，
     *     然后再从"被切节点的父节点"到所在树根节点递归执行级联剪枝
     */
    template <class T>
    void FibHeap<T>::cascadingCut(FibNode<T> *node) 
    {
        FibNode<T> *parent = node->parent;
        if (parent != NULL)
        {
            if (node->marked == false) 
                node->marked = true;
            else
            {
                cut(node, parent);
                cascadingCut(parent);
            }
        }
    }

    /* 
     * 将斐波那契堆中节点node的值减少为key
     */
    template <class T>
    void FibHeap<T>::decrease(FibNode<T> *node, T key)
    {
        FibNode<T> *parent;

        if (min==NULL ||node==NULL) 
            return ;

        if ( key>=node->key)
        {
            cout << "decrease failed: the new key(" << key <<") "
                 << "is no smaller than current key(" << node->key <<")" << endl;
            return ;
        }

        node->key = key;
        parent = node->parent;
        if (parent!=NULL && node->key < parent->key)
        {
            // 将node从父节点parent中剥离出来，并将node添加到根链表中
            cut(node, parent);
            cascadingCut(parent);
        }

        // 更新最小节点
        if (node->key < min->key)
            min = node;
    }

 

## 6. 增加节点值

增加节点值和减少节点值类似，这个操作的难点也是如何维护"最小堆"性质。思路如下：  
(1) 将"被增加节点"的"左孩子和左孩子的所有兄弟"都链接到根链表中。  
(2) 接下来，把"被增加节点"添加到根链表；但是别忘了对其进行级联剪切。

![img](/media/pic/datastruct_algrithm/heap/fibonacci/07.jpg)

上面是增加节点值的示意图。该操作示意图与测试程序中的"增大节点"相对应！

 

增加节点值的代码

    /* 
     * 将斐波那契堆中节点node的值增加为key
     */
    template <class T>
    void FibHeap<T>::increase(FibNode<T> *node, T key)
    {
        FibNode<T> *child, *parent, *right;

        if (min==NULL ||node==NULL) 
            return ;

        if (key <= node->key)
        {
            cout << "increase failed: the new key(" << key <<") " 
                 << "is no greater than current key(" << node->key <<")" << endl;
            return ;
        }

        // 将node每一个儿子(不包括孙子,重孙,...)都添加到"斐波那契堆的根链表"中
        while (node->child != NULL)
        {
            child = node->child;
            removeNode(child);               // 将child从node的子链表中删除
            if (child->right == child)
                node->child = NULL;
            else
                node->child = child->right;

            addNode(child, min);       // 将child添加到根链表中
            child->parent = NULL;
        }
        node->degree = 0;
        node->key = key;

        // 如果node不在根链表中，
        //     则将node从父节点parent的子链接中剥离出来，
        //     并使node成为"堆的根链表"中的一员，
        //     然后进行"级联剪切"
        // 否则，则判断是否需要更新堆的最小节点
        parent = node->parent;
        if(parent != NULL)
        {
            cut(node, parent);
            cascadingCut(parent);
        }
        else if(min == node)
        {
            right = node->right;
            while(right != node)
            {
                if(node->key > right->key)
                    min = right;
                right = right->right;
            }
        }
    }

 

## 7. 删除节点

删除节点，本文采用了操作是："取出最小节点"和"减小节点值"的组合。  
(1) 先将被删除节点的键值减少。减少后的值要比"原最小节点的值"即可。  
(2) 接着，取出最小节点即可。


删除节点值的代码

    /*
     * 删除结点node
     */
    template <class T>
    void FibHeap<T>::remove(FibNode<T> *node)
    {
        T m = min->key-1;
        decrease(node, m-1);
        removeMin();
    }


注意：关于斐波那契堆的"更新"、"打印"、"销毁"等接口就不再单独介绍了。后文的源码中有给出它们的实现代码，Please RTFSC(Read The Fucking Source Code)！

 
<a name="anchor3"></a>
# 第3部分 斐波那契堆的C++实现(完整源码)


点击查看：[源代码][link_source_code]


斐波那契堆的测试程序包括了"插入"、"合并"、"增大"、"减小"、"删除"、"基本信息"等几种功能的测试代码。默认是运行的"基本信息(验证斐波那契堆的结构)"测试代码，你可以根据自己的需要来对相应的功能进行验证！

下面是基本信息测试代码的运行结果：

    == 斐波那契堆(hb)中依次添加: 18 35 20 42 9 31 23 6 48 11 24 52 13 2 
    == 斐波那契堆(hb)删除最小节点
    == 斐波那契堆的详细信息: ==
     1.    6(3) is root
           9(0) is  6's child
          18(1) is  9's next
          35(0) is 18's child
          20(2) is 18's next
          42(0) is 20's child
          23(1) is 42's next
          31(0) is 23's child
     2.   11(2) is root
          48(0) is 11's child
          24(1) is 48's next
          52(0) is 24's child
     3.   13(0) is root

 
[link_source_code]: https://github.com/wangkuiwu/datastructs_and_algorithm/tree/master/source/heap/fibonacci/cplus
