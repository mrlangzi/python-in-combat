# 关于 B+tree (附 python 模拟代码)

前几天我写了点 btree 的东西(<http://thuhak.blog.51cto.com/2891595/1261783>)，今天继续这个思路，继续写 b+tree。

而且 b+tree 才是我的目的，更加深入理解文件和数据库索引的基本原理。

在之前，我一直只把 b+tree 当成是 btree 的一种变形，或者说是在某种情况下的一种优化，另外一些情况可能还是 btree 好些。但是做完之后才发现，b+tree 在各种情况都可以完全取代 btree，并能够让索引性能得到比 btree 更好的优化。因为 b+tree 设计的核心要点，是为了弥补 btree 最大的缺陷。

**btree 最大的缺陷是什么?**

首先,我们知道对于 btree 和 b+tree 这种多路搜索树来说，一个很重要的特点就是树的度数非常大。因为只有这样才能够降低树的深度，减少磁盘读取的次数。而树的度数越大，叶子节点在树中的比例就越大。假设度数为 1000，那么叶子节点比他上一层内部节点的数量至少要多 1000 倍，在上一层就更加可以忽略不计了。可以说树种 99.9% 的节点都是叶子节点。 但是对于 btree 来说，所有节点都是一样的结构，都含有一定数量的数据和指向节点的指针。这两项数据占据 btree 节点的几乎全部的空间。一个节点内的数据的数量比硬盘指针的数量少一，可以说和指针的数量几乎相等。对于 python 这种动态类型语言感觉不出来，但是对于 C 这种固定类型语言来说，即使这个 children list 数组为空，这个数组的空间也都是预留出去的。导致的结果就是占绝大多数的叶子节点的 children list 指针数组所占的磁盘空间完全浪费。

一个数据的大小和硬盘指针的大小取决于 key-value 中 key 和 value 大小的比值。假如说这个比值是 2:1。那么 btree 浪费了几乎 1/3 的空间。

b+tree 针对这个问题的，把叶子节点和内节点的数据结构分开设计，让叶子节点不存放指针。因此同样大小的叶子节点，b+tree 所能包含数据数量要比 btree 大。按照上面的假设就是大 1/2。数的深度很可能比 btree 矮，大范围搜索或遍历所需要的载入磁盘的次数也少。

另外，b+tree 还有一个特点是所有数据都存放在叶子节点，这些叶子节点也可以组成一个链表，并把这个链表的表头拿出来，方便直访问数据。有些文章认为这对于范围搜索来说是个巨大的优化。但是就我的看法，这个特性最大的作用仅仅是让代码更容易一些，性能上，只会比树的遍历差，而不会比树的遍历好。因为不管是用指向叶子节点的指针搜，还是用树的遍历搜，所搜索的节点的数量都是几乎相同的。在相同大小的范围搜索的性能，只取决于访问顺序的连续性。从树根向下遍历，那么一次可以取得大量的子节点的范围，并针对这些节点做访问排序，得到更好的访问连续性。如果是沿着指向兄弟节点的指针搜索，一是兄弟节点也许是后插入的，存放并不一定和自己是连续的，二是只有每次从硬盘中将该节点载入到内存，才知道兄弟节点放在硬盘哪个位置，这又变成了对硬盘的一个随机的同步操作，性能的下降可想而知。

说 b+tree 因为有指向兄弟节点的指针方便数据库扫库这种结论，是不正确的。

还是上代码吧,依旧只是在内存对数据结构插入删除查找的模拟

be

```
#!/usr/bin/env python
from random import randint,choice
from bisect import bisect_right,bisect_left
from collections import deque
class InitError(Exception):
    pass
class ParaError(Exception):
    pass
class KeyValue(object):
    __slots__=('key','value')
    def __init__(self,key,value):
        self.key=key
        self.value=value
    def __str__(self):
        return str((self.key,self.value))
    def __cmp__(self,key):
        if self.key>key:
            return 1
        elif self.key==key:
            return 0
        else:
            return -1
class Bptree_InterNode(object):
    def __init__(self,M):
        if not isinstance(M,int):
            raise InitError,'M must be int'
        if M<=3:
            raise InitError,'M must be greater then 3'
        else:
            self.__M=M
            self.clist=[]
            self.ilist=[]
            self.par=None
    def isleaf(self):
        return False
    def isfull(self):
        return len(self.ilist)>=self.M-1
    def isempty(self):
        return len(self.ilist)<=(self.M+1)/2-1
    @property
    def M(self):
        return self.__M
class Bptree_Leaf(object):
    def __init__(self,L):
        if not isinstance(L,int):
            raise InitError,'L must be int'
        else:
            self.__L=L
            self.vlist=[]
            self.bro=None
            self.par=None
    def isleaf(self):
        return True
    def isfull(self):
        return len(self.vlist)>self.L
    def isempty(self):
        return len(self.vlist)<=(self.L+1)/2
    @property
    def L(self):
        return self.__L
class Bptree(object):
    def __init__(self,M,L):
        if L>M:
            raise InitError,'L must be less or equal then M'
        else:
            self.__M=M
            self.__L=L
            self.__root=Bptree_Leaf(L)
            self.__leaf=self.__root
    @property
    def M(self):
        return self.__M
    @property
    def L(self):
        return self.__L
    def insert(self,key_value):
        node=self.__root
        def split_node(n1):
            mid=self.M/2
            newnode=Bptree_InterNode(self.M)
            newnode.ilist=n1.ilist[mid:]
            newnode.clist=n1.clist[mid:]
            newnode.par=n1.par
            for c in newnode.clist:
                c.par=newnode
            if n1.par is None:
                newroot=Bptree_InterNode(self.M)
                newroot.ilist=[n1.ilist[mid-1]]
                newroot.clist=[n1,newnode]
                n1.par=newnode.par=newroot
                self.__root=newroot
            else:
                i=n1.par.clist.index(n1)
                n1.par.ilist.insert(i,n1.ilist[mid-1])
                n1.par.clist.insert(i+1,newnode)
            n1.ilist=n1.ilist[:mid-1]
            n1.clist=n1.clist[:mid]
            return n1.par
        def split_leaf(n2):
            mid=(self.L+1)/2
            newleaf=Bptree_Leaf(self.L)
            newleaf.vlist=n2.vlist[mid:]
            if n2.par==None:
                newroot=Bptree_InterNode(self.M)
                newroot.ilist=[n2.vlist[mid].key]
                newroot.clist=[n2,newleaf]
                n2.par=newleaf.par=newroot
                self.__root=newroot
            else:
                i=n2.par.clist.index(n2)
                n2.par.ilist.insert(i,n2.vlist[mid].key)
                n2.par.clist.insert(i+1,newleaf)
                newleaf.par=n2.par
            n2.vlist=n2.vlist[:mid]
            n2.bro=newleaf
        def insert_node(n):
            if not n.isleaf():
                if n.isfull():
                    insert_node(split_node(n))
                else:
                    p=bisect_right(n.ilist,key_value)
                    insert_node(n.clist[p])
            else:
                p=bisect_right(n.vlist,key_value)
                n.vlist.insert(p,key_value)
                if n.isfull():
                    split_leaf(n)
                else:
                    return
        insert_node(node)
    def search(self,mi=None,ma=None):
        result=[]
        node=self.__root
        leaf=self.__leaf
        if mi is None and ma is None:
            raise ParaError,'you need to setup searching range'
        elif mi is not None and ma is not None and mi>ma:
            raise ParaError,'upper bound must be greater or equal than lower bound'
        def search_key(n,k):
            if n.isleaf():
                p=bisect_left(n.vlist,k)
                return (p,n)
            else:
                p=bisect_right(n.ilist,k)
                return search_key(n.clist[p],k)
        if mi is None:
            while True:
                for kv in leaf.vlist:
                    if kv<=ma:
                        result.append(kv)
                    else:
                        return result
                if leaf.bro==None:
                    return result
                else:
                    leaf=leaf.bro
        elif ma is None:
            index,leaf=search_key(node,mi)
            result.extend(leaf.vlist[index:])
            while True:
                if leaf.bro==None:
                    return result
                else:
                    leaf=leaf.bro
                    result.extend(leaf.vlist)
        else:
            if mi==ma:
                i,l=search_key(node,mi)
                try:
                    if l.vlist[i]==mi:
                        result.append(l.vlist[i])
                        return result
                    else:
                        return result
                except IndexError:
                    return result
            else:
                i1,l1=search_key(node,mi)
                i2,l2=search_key(node,ma)
                if l1 is l2:
                    if i1==i2:
                        return result
                    else:
                        result.extend(l.vlist[i1:i2])
                        return result
                else:
                    result.extend(l1.vlist[i1:])
                    l=l1
                    while True:
                        if l.bro==l2:
                            result.extend(l2.vlist[:i2+1])
                            return result
                        else:
                            result.extend(l.bro.vlist)
                            l=l.bro
    def traversal(self):
        result=[]
        l=self.__leaf
        while True:
            result.extend(l.vlist)
            if l.bro==None:
                return result
            else:
                l=l.bro
    def show(self):
        print 'this b+tree is:\n'
        q=deque()
        h=0
        q.append([self.__root,h])
        while True:
            try:
                w,hei=q.popleft()
            except IndexError:
                return
            else:
                if not w.isleaf():
                    print w.ilist,'the height is',hei
                    if hei==h:
                        h+=1
                    q.extend([[i,h] for i in w.clist])
                else:
                    print [v.key for v in w.vlist],'the leaf is,',hei
                                                                                                                                                                                                                                                                                                                                   
    def delete(self,key_value):
        def merge(n,i):
            if n.clist[i].isleaf():
                n.clist[i].vlist=n.clist[i].vlist+n.clist[i+1].vlist
                n.clist[i].bro=n.clist[i+1].bro
            else:
                n.clist[i].ilist=n.clist[i].ilist+[n.ilist[i]]+n.clist[i+1].ilist
                n.clist[i].clist=n.clist[i].clist+n.clist[i+1].clist
            n.clist.remove(n.clist[i+1])
            n.ilist.remove(n.ilist[i])
            if n.ilist==[]:
                n.clist[0].par=None
                self.__root=n.clist[0]
                del n
                return self.__root
            else:
                return n
        def tran_l2r(n,i):
            if not n.clist[i].isleaf():
                n.clist[i+1].clist.insert(0,n.clist[i].clist[-1])
                n.clist[i].clist[-1].par=n.clist[i+1]
                n.clist[i+1].ilist.insert(0,n.ilist[i])
                n.ilist[i]=n.clist[i].ilist[-1]
                n.clist[i].clist.pop()
                n.clist[i].ilist.pop()
            else:
                n.clist[i+1].vlist.insert(0,n.clist[i].vlist[-1])
                n.clist[i].vlist.pop()
                n.ilist[i]=n.clist[i+1].vlist[0].key
        def tran_r2l(n,i):
            if not n.clist[i].isleaf():
                n.clist[i].clist.append(n.clist[i+1].clist[0])
                n.clist[i+1].clist[0].par=n.clist[i]
                n.clist[i].ilist.append(n.ilist[i])
                n.ilist[i]=n.clist[i+1].ilist[0]
                n.clist[i+1].clist.remove(n.clist[i+1].clist[0])
                n.clist[i+1].ilist.remove(n.clist[i+1].ilist[0])
            else:
                n.clist[i].vlist.append(n.clist[i+1].vlist[0])
                n.clist[i+1].vlist.remove(n.clist[i+1].vlist[0])
                n.ilist[i]=n.clist[i+1].vlist[0].key
        def del_node(n,kv):
            if not n.isleaf():
                p=bisect_right(n.ilist,kv)
                if p==len(n.ilist):
                    if not n.clist[p].isempty():
                        return del_node(n.clist[p],kv)
                    elif not n.clist[p-1].isempty():
                        tran_l2r(n,p-1)
                        return del_node(n.clist[p],kv)
                    else:
                        return del_node(merge(n,p),kv)
                else:
                    if not n.clist[p].isempty():
                        return del_node(n.clist[p],kv)
                    elif not n.clist[p+1].isempty():
                        tran_r2l(n,p)
                        return del_node(n.clist[p],kv)
                    else:
                        return del_node(merge(n,p),kv)
            else:
                p=bisect_left(n.vlist,kv)
                try:
                    pp=n.vlist[p]
                except IndexError:
                    return -1
                else:
                    if pp!=kv:
                        return -1
                    else:
                        n.vlist.remove(kv)
                        return 0
        del_node(self.__root,key_value)
def test():
    mini=2
    maxi=60
    testlist=[]
    for i in range(1,10):
        key=i
        value=i
        testlist.append(KeyValue(key,value))
    mybptree=Bptree(4,4)
    for kv in testlist:
        mybptree.insert(kv)
    mybptree.delete(testlist[0])
    mybptree.show()
    print '\nkey of this b+tree is \n'
    print [kv.key for kv in mybptree.traversal()]
    #print [kv.key for kv in mybptree.search(mini,maxi)]
if __name__=='__main__':
    test()
```

实现过程和 btree 很像，不过有几点显著不同。

1. 内节点不存储 key-value，只存放 key
2. 沿着内节点搜索的时候，查到索引相等的数要向树的右边走。所以二分查找要选择 bisect_right
3. 在叶子节点满的时候，并不是先分裂再插入而是先插入再分裂。因为 b+tree 无法保证分裂的两个节点的大小都是相等的。在奇数大小的数据分裂的时候右边的子节点会比左边的大。如果先分裂再插入无法保证插入的节点一定会插在数量更少的子节点上，满足节点数量平衡的条件。
4. 在删除数据的时候，b+tree 的左右子节点借数据的方式比 btree 更加简单有效，只把子节点的子树直接剪切过来，再把索引变一下就行了，而且叶子节点的兄弟指针也不用动。

本文出自 “笔记” 博客，请务必保留此出处 <http://thuhak.blog.51cto.com/2891595/1269059>