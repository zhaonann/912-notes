Conclusion on Chapter Two: List
===============================

## List是采取动态的存储策略

这是容易理解的，因为`List`中每个结点都是在需要时才被动态创建的，这和`Vector`预先分配了一个数组具有本质的区别。因此，他们支持的操作也会有一些不同。

一般操作可以分为静态操作和动态操作。静态操作就好像是只读(read only)的一些操作，例如按秩索引某个元素，查找等；而动态操作则是会修改数据成员的一些操作，例如插入，删除等。

一般说来，静态存储策略比较擅长静态操作，而动态存储策略比较擅长动态操作。比如`Vector`的按秩索引以及查找操作分别只需要$O(1)$以及$O(logn)$的时间复杂度，而`List`的按秩索引和查找操作则都需要$O(n)$的时间复杂度。

相应的，动态存储策略也会比较擅长动态操作。比如`List`的插入和删除操作都只需要$O(1)$的时间复杂度，而`Vector`的插入和删除操作则都需要$O(n)$的时间复杂度。

## 列表的头尾结点

> 为什么列表的实现中需要设置头尾结点？

显然是为了简化操作。但我之前对这里[简化操作]并不是很有理解，现在来理解一下。

举两个例子：

+ 插入操作。考虑一种情况，被插入结点是当前的第一个结点，没有头尾结点的版本将会每次都判断当前列表是否非空，如果为空的话，需要建立一个新的结点，然后将其前后位置指针都赋值为`nullptr`。而如果非空的话，则需要在创建结点后修改当前结点与前驱(后继)结点的指针。这两种操作显然是不同的，因此需要分情况讨论。而如果是有头尾结点的版本，无论当前列表是否为空，总是可以通过修改被插入结点的前驱(后继)来完成操作，情况很简单，逻辑也因此变得清晰。
+ 删除操作。考虑无头尾指针的情况。如果被删除结点为一般结点，则需要修改前驱和后继的指针，然后释放被删除结点。而如果被删除结点是首结点，则需要重新指定首节点，尾结点亦然。此外还有一种情况，如果被删除结点是列表中最后一个结点，需要同时将首末结点赋值为`nullptr`。这么多种情况全部需要分别讨论。而如果有头尾结点的话，所有情况都可以通过上面的一般情况进行处理，大大简化了逻辑。

可见，通过引入了头尾哨兵结点，只需要常数的空间开销，却带来思路清晰，逻辑简单的程序，并且可以省去许多判断操作。其成本远远低于由此带来的便利。

## 无序列表的唯一化操作

就算法而言，其实和无序向量的唯一化操作一样，即保证当前元素前面的所有元素都是唯一化的，然后在前面元素中查找当前元素，如果找到了就进行删除操作。

但是这里列表有一个实现细节上的不同，代码如下：

```cpp
template <typename T>
int List<T>::deduplicate(){
	int oldSize = size;
	ListNodePosi(T) p = head->succ;
	ListNodePosi(T) tmp;
	for (; p != tail; p = p->succ){
		for(tmp = head->succ; tmp != p; tmp = tmp->succ){
			if(tmp->val == p->val){
				pop(tmp);
				break;
			}
		}
	}
	return oldSize - size;
}
```

可以看到，在列表的唯一化算法中，找到前面元素中与当前结点p相同的结点tmp时，并不是像向量那样直接删除p，而是选择删除tmp。其实这是非常自然的，因为如果删除了p，还需要慢慢移动到p之后的那个结点，需要额外的开销。究其原因，还是因为列表不能实现随机的存取。

## 有序列表的查找

查找代码如下：

```cpp
template <typename T>
ListNodePosi(T) List<T>::search(T const &val, int n, ListNodePosi(T) p) const{
	for(int ix = 0; ix != n; ++ix){
		p = p->prev;
		if (p->val <= val) return p;
	}
	return p->prev;
}
```

这里主要想指出，查找函数的接口。
+ 一方面它的参数很奇怪。是查找位置`p`前面`n`个结点，不包括`p`。
+ 另一方面，它的接口是沿用了之前`Vector`查找的接口。就是说，列表的`search`，也是返回不大于被查找元素的最后一个。同样地，这样的接口也具有类似`Vector`那样的好处，例如在后面的插入排序中，可以保证排序的稳定性。

## 插入排序

插入排序的算法是简单的，这里主要还是专注于一些细节。其代码如下:

```cpp
template <typename T>
void List<T>::insertion_sort(ListNodePosi(T) p, int n){
	ListNodePosi(T) target;
	for(int ix = 1; ix != n; ix++){
		p = p->succ;
		target = search(p->val, ix, p);
		insert_before(target->succ, p->val);
		pop(p);
	}
}
```

可以看到，每次寻找正确的插入的位置，是通过调用之前说过的`search`来实现的。这样可以保证插入排序是稳定的，这是因为`search`返回的是不大于当前元素的最后一个，即使具有多个相同的匹配项，也只会返回最后一个，这样排序以后重复元素仍将保持原有的次序。

## 选择排序

还是只讲细节。

```cpp
template <typename T>
ListNodePosi(T) List<T>::selectMax(ListNodePosi(T) p, int n){
	ListNodePosi(T) maxPosi = p;
	while(--n){
		p = p->succ;
		if (maxPosi->val <= p->val) maxPosi = p;
	}
	return maxPosi;
}

template <typename T>
void List<T>::selection_sort(ListNodePosi(T) p, int n){
	ListNodePosi(T) currMax;
	ListNodePosi(T) tail = p;
	for (int ix = 0; ix != n; ++ix, tail = tail->succ);

	for (; n != 1; --n) {
		currMax = selectMax(p, n);
		swap(currMax->val, tail->prev->val);
		tail = tail->prev;
	}
}
```

这里要注意的细节是`selectMax`中的比较是`<=`。就是说，`selectMax`返回的是序列中最后面的一个最大元素。这样，在`selection_sort`中，一次交换操作以后，仍然可以保证重复元素继续保持原有的次序。因此保证了这里的选择排序也是一个稳定的排序算法。