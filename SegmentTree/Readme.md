## Segment tree (supporting range operation)
Segment tree stores the interval/segment at each tree node. It supports query of sum/max/min value within a given range of elements.

With segment tree, preprocessing time is `O(n)` and the time to query the range max/min is `O(log n)`. The extra space required is `O(n)` to store the segment tree.

The best implementation in Python:

Without lazy propagation, just increase one number (instead of a range)
```
class SegTree(object):
    def __init__(self, L, R):
        self.v = collections.defaultdict(int) # dynamically allocation space
        self.L, self.R = L, R                 # any range (pre-known), otherwise, just assign the lower/upper bound of int type
        
    def add(self, i):
        self._add(i, self.L, self.R)
    
    def search(self, e):                      # use it as a Fenwick tree (NO lower query bound)
        return self._search(self.L, e, self.L, self.R)
    
    def _add(self, i, l, r, id = 1):
        if l <= i < r:
            if r - l < 2:
                self.v[id] += 1
                return
            m = (l + r) // 2
            self._add(i, l, m, id * 2)
            self._add(i, m, r, id*2+1)
            self.v[id] = self.v[id*2] + self.v[id*2+1]
            
    def _search(self, s, e, l, r, id = 1):
        if r <= s or l >= e: return 0
        if s <= l < r <= e: return self.v[id]
        m = (l + r) // 2
        return self._search(s, e, l, m, id * 2) + self._search(s, e, m, r, id*2+1)
```

As for the array implementation, one important note is that the leaves, which are the array element, are located at i + N. (N is the length and i the index). The index of tree node would be 1, 2\~3, 4\~7, 8\~15 ...
And 0 is a DUMMY node.

Leetcode 307. Range Sum Query - Mutable (Really concise C++ code <http://codeforces.com/blog/entry/18051>
)
```
class NumArray {
public:
    int N, *t;
    NumArray(vector<int> nums) {
        N = nums.size();
        t = new int[2 * N];
        for (int i = 0; i < N; ++i) t[i + N] = nums[i];
        for (int i = N - 1; i > 0; --i) t[i] = t[i << 1] + t[i << 1 | 1];
    }
    void update(int i, int val) {
        for (t[i += N] = val; i > 1; i >>= 1) t[i >> 1] = t[i] + t[i^1];
    }
    int sumRange(int i, int j) {
        int ret = t[j + N];  // Note ret = nums[j] + sum [i,j)
        for (i += N, j += N; i < j; i >>= 1, j >>= 1) {
            if (i&1) ret += t[i++];
            if (j&1) ret += t[--j];
        }
        return ret;
    }
};
```

But one note is that this code does NOT support binary search like classical segment trees (require extra codes). If you need binary search, then a better choice is to construct tree nodes!

Segment Tree also supports range operations (add value to all the elements in a range). But we shouldn't update all the nodes in this interval, just the maximal ones, then pass it to children when we need. This trick is called Lazy Propagation. We go from root to leaves, when one interval is completed covered by the query range, we can stop there and save the updates at the maximal ones. In this way, every level of the tree has only two/four active nodes for recursion. The time complexity for range operation is `O(log n)`.

See how to construct tree dynamically: [Java](https://leetcode.com/problems/my-calendar-iii/discuss/109568/Java-Solution-O(n-log(len))-beats-100-Segment-Tree)

The best introduction in [C++ by (DarthPrince)](http://codeforces.com/blog/entry/15729)

## Range max/min with lazy operation

Storing the max/min element in an interval at each node. Note that `add(l,r,val)` would add `val` to the **ENTIRE** interval `[x,y)` within range `[l,r)`. The lazy operation accumulates the `val` in `interval.lazy` at such maximal intervals. When the recursive query returns from children, we just add current `interval.lazy` to the returned max value.

**LC732. My Calendar III** 
Return an integer K representing the largest integer such that there exists a K-booking in the calendar. (There are K books overlapping at the same period.)

20-lines Python version:
        
	
```
class SegTree(object):
    def __init__(self):
        self.v = collections.defaultdict(int)
        self.lazy = collections.defaultdict(int)
 
    def add(self, s, e, l = 0, r = 10**9+1, id = 1):
        if s >= r or e <= l: return
        if s <= l < r <= e:
            self.lazy[id] += 1
            self.v[id] += 1
            return
        m = (l + r) // 2
        self.add(s, e, l, m, id * 2)
        self.add(s, e, m, r, id*2+1)
	
	# lazy stores the increase hidden to it children
        # val stores the LOCAL max val in interval (ignoring its parent's lazy)
        self.v[id] = self.lazy[id] + max(self.v[id*2], self.v[id*2+1])
            
class MyCalendarThree(object):
    def __init__(self):
        self.st = SegTree()

    def book(self, start, end):
        self.st.add(start, end)
        return self.st.v[1]
```


```
class STNode:
    def __init__(self, l, r):
        self.l, self.r = l, r
        self.val, self.lazy = 0, 0
        self.left, self.right = None, None
        
class SegmentTree:
    def __init__(self):
        self.root = STNode(0, 10**9+1)
    
    def add(self, nd, x, y, val):
        if nd.l >= y or x >= nd.r: return # no intersection
        if x <= nd.l < nd.r <= y: # fully covered
            nd.val += val
            nd.lazy += val
            return
        
        # construct sub-intervals
        m = (nd.l + nd.r) // 2
        if not nd.left: nd.left = STNode(nd.l, m)
        self.add(nd.left, x, y, val)
        if not nd.right: nd.right = STNode(m, nd.r)
        self.add(nd.right, x, y, val)
        
        # np.lazy stores the increase hidden to nd.left and nd.right
        # np.val stores the max val in interval [nd.l, nd.r)
        nd.val = nd.lazy + max(nd.left.val, nd.right.val)
            
    def Max(self):
        return self.root.val
    
    def Add(self, x, y):
        self.add(self.root, x, y, 1)
        
class MyCalendarThree:

    def __init__(self):
        self.seg = SegmentTree()

    def book(self, start, end):
        self.seg.Add(start, end)
        return self.seg.Max()

# Your MyCalendarThree object will be instantiated and called as such:
# obj = MyCalendarThree()
# param_1 = obj.book(start,end)
```

## Range sum with lazy operation

Unlike the range max/min which only pass `interval.lazy` to parent, the range sum segment tree passes the lazy variable to children.

See C++ code from [C++ by (DarthPrince)](http://codeforces.com/blog/entry/15729)
```
// A function to update a node :
void upd(int id,int l,int r,int x){//	increase all members in this interval by x
	lazy[id] += x;
	s[id] += (r - l) * x;
}


// A function to pass the update information to its children :
void shift(int id,int l,int r){//pass update information to the children
	int mid = (l+r)/2;
	upd(id * 2, l, mid, lazy[id]);
	upd(id * 2 + 1, mid, r, lazy[id]);
	lazy[id] = 0;// passing is done
}


// A function to perform increase queries :
void increase(int x,int y,int v,int id = 1,int l = 0,int r = n){
	if(x >= r or l >= y)	return ;
	if(x <= l && r <= y){
		upd(id, l, r, v);
		return ;
	}
	shift(id, l, r);
	int mid = (l+r)/2;
	increase(x, y, v, id * 2, l, mid);
	increase(x, y, v, id*2+1, mid, r);
	s[id] = s[id * 2] + s[id * 2 + 1];
}
```
Note that `shift` should be called at every level **before** the function `increase` traverses to the children!

## Two-dimensional Segment Tree

Easy, now you need to build a segment tree over `x` coordinates. Each node of the x-coordinate (outer) segment tree stores a segment tree on `y` coordinates that corresponds to strip `[xl, xr]`, meaning that it will store all the sums of rectangles `[xl,xr]×[yl,yr]` where `[yl,yr]` is a valid atomic segment tree segment.

While answering queries you basically first traverse by outer tree until you find O(logn) segments that fits your x-part-of-a-query and then for each such segment you are traversing in corresponding segment tree until you find a set of segments that match your y-part-of-a-query.

Before writing a 2D-segment tree you should consider using partial sums, 2D-Fenwick tree or O(1)-static-rmq for 2D since all of them are like 10× shorter and usually faster.

I was thinking about spliting a 2D rectangle into 4 smaller rectangles. But this does not work because every level may contains more than 4 active interval of query. Unlike 1D case where you got at most 4 active interval on each level, this incorrect 2D extension may grow the active rectangles exponentially -- it is not efficient!
