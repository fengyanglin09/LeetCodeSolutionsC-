## Segment Tree

In computer science, a segment tree, also known as a statistic tree, is a tree data structure used for storing information about intervals, or segments. 
It allows querying which of the stored segments contain a given point. 
It is, in principle, a static structure; that is, it's a structure that cannot be modified once it's built. A similar data structure is the interval tree.

A segment tree for a set I of n intervals uses O(n log n) storage and can be built in O(n log n) time.
Segment trees support searching for all the intervals that contain a query point in O(log n + k), k being the number of retrieved intervals or segments.

Applications of the segment tree are in the areas of computational geometry, and geographic information systems.

### Things need to know for implementation:
1. let array be A and its size be N
2. the size for tree array and lazy array should be 4*N
3. consider 3 cases when implementing segment tree:
    1. when range is completely not enclosed
    2. when range is completely enclosed
    3. when range is partially enclosed
4. with recursion, the range will reduce to enclosed range
5. T(n) for both lazy update and lazy query is O(lgn)

-------------------------------------------------------------------------------

### Leetcode example

#### 308. Range Sum Query 2D - Mutable,  this problem can be solved using 2D segment tree or Binary Index Tree

-------------------------------------------------------------------------------
### My Segment Tree Implementation Version 2 - Lazy Propagations

```c++
/*
Implementation of Segment Tree With Lazy Updates

The idea is to not update until it become necessary

example:
        1
    2       3
4       5

normally, if we update one node all the nodes in the path from root to leaf will also be updated,
with lazy update, if the update path is 1->2->5, we may stop update at 2 bc not need to update 5 at the moment
the change will be propagated to node 5 if needed.


O(lgn) for update and query
*/

//Segment Tree for calculating sum 
class SegTree {
public:
    SegTree(vector<int>& a) {
        nums = a;
        n = nums.size();
        tree = vector<int>(n * 4, 0);
        lazy = vector<int>(n * 4, 0);

        ToBuild(0, n - 1, 0);
    }
    /*
      lazy query: the idea is the update the tree if lazy is not zero.
      int l, r are the queried range
      int low, high are the current range of nums
      int pos is the start node of tree and always zero
    */
    int lazyFindSum(int l, int r, int low, int high, int pos = 0) {
        //if completely not enclosed, return 0;
        if (r < low || l > high) {
            return 0;
        }
        /*
          if completely enclosed in range (l, r)
               1. check if update tree from lazy is needed and update it if yes
               2. if the step1 is needed then propagate value of lazy node to its direct children if possible, and set the lazy node to zero
               3. return the updated tree node value
        */
        if (l <= low && r >= high) {//step1
            if (lazy[pos] != 0) {
                tree[pos] = lazy[pos];
                if (low != high) {//step2
                    lazy[2 * pos + 1] = lazy[pos];
                    lazy[2 * pos + 2] = lazy[pos];
                }
                lazy[pos] = 0;//part of step2
            }
            return tree[pos];
        }
        /*
            if partially enclosed, continue search until first complete enclosed is reached
        */
        int mid = (low + high) / 2;
        int left = lazyFindSum(l, r, low, mid, 2*pos+1);
        int right = lazyFindSum(l, r, mid+1, high, 2*pos+2);
        return left + right;
    }
    /*
      Lazy update: the idea is to update only the first enclosed range and stop updating further.
                   The rest part will be updated when needed to
      int l, r are the range on leaf nodes to which val is used to replace the old values
      int low, high are current the range of nums
      int pos is starting tree node, and always zero

    */
    void lazyUpdate(int l, int r, int low, int high, int pos, int val) {
        //if completely out of range, do nothing
        if (r < low || l > high) {
            return;
        }
        /*
        if compeletely in the range:
            1. update the tree with lazy if the corresponding lazy node is not zero
            2. propagate the lazy node value to its children nodes if possible (step1 needed) and reset the lazy node to zero
            3. add the val to the tree node
            4. stop the recursion, no need to update further and children nodes will be updated if necessary
        */
        if (l <= low && r >= high) {
            //step1,2
            lazy[pos] = 0;
            if (low != high) {
                lazy[2 * pos + 1] = val;
                lazy[2 * pos + 2] = val;
            }
            tree[pos] = val;//step3
            //step4
            return;
        }

        /*
            if partially in the range:
                1. update tree if lazy node is not zero and propagate update to the children nodes  
                2. recurse until find the first completed enclosed range
                3. back track and update tree nodes
        */
        if (lazy[pos] != 0) {//step1
            tree[pos] = lazy[pos];
            //step2
            if (low != high) {//if there are children nodes
                lazy[2 * pos + 1] = lazy[pos];
                lazy[2 * pos + 2] = lazy[pos];
            }
            lazy[pos] = 0;
        }
        //step2,3
        int mid = (low + high) / 2;
        lazyUpdate(l, r, low, mid, 2 * pos + 1, val);
        lazyUpdate(l, r, mid+1, high, 2 * pos + 2, val);

        tree[pos] = tree[2 * pos + 1] + tree[2 * pos + 2];

    }
private:
    vector<int> nums;
    vector<int> tree;
    vector<int> lazy;
    int n;//size of nums
    /*
    int low: 0, first index of nums
    int high: n-1, last index of nums
    int tpos: starting tree node index - always 0
    */
    int ToBuild(int low, int high, int tpos) {
        if (low == high) {//if leaf node
            tree[tpos] = nums[low];
            return nums[low];
        }
        int mid = (low + high) / 2;
        int lo = ToBuild(low, mid, 2 * tpos + 1);
        int hi = ToBuild(mid + 1, high, 2 * tpos + 2);
        tree[tpos] = lo + hi;
        return tree[tpos];
    }
};

/*

testing case 0:
{1, 2, 3, 4, 5, 6}

testing case 1:
{ 18, 17, 13, 19, 15, 11, 20, 12, 33, 25 };
Tree after construction:
{ 183, 82, 101, 48, 34, 43, 58, 35, 13, 19, 15, 31, 12, 33, 25, 18, 17, 0, 0, 0, 0, 0, 0, 11, 20, 0, 0, 0, 0, 0, 0 };
*/

int main() {
    vector<int> a = { 1,2,3,4,5,6 };
    SegTree S(a);
    cout << S.lazyFindSum(1, 4, 0, 5) << endl; // expect 14
    S.lazyUpdate(2, 2, 0, 5, 0, 6);
    cout << S.lazyFindSum(1, 4, 0, 5) << endl; // expect 17
    S.lazyUpdate(2, 3, 0, 5, 0, 2);
    cout << S.lazyFindSum(1, 4, 0, 5) << endl; // expect 11
    return 0;
}
```

-------------------------------------------------------------------------------
### My Segment Tree Implementation Version 1

```c++

/*
Segement Tree, version1 - calculating sum
some facts about a tree:
given any tree node i, then:
left child index = 2 * i + 1
right child index = 2 * i + 2
parent index = (i - 1)/2, which is calculated by the formula for left child index

segment tree for sum of an integer array
1. let array A be size of n, then segment tree should be size of 4n

Analysis:
T(n) = O(n), for building the tree
T(n) = O(lgn), for update, quary

note: the update is not very efficent, as there will be O(lgn) for every update.
A more efficient update method is the lazy propagation
*/

class Solution {
public:
    Solution() {
        
    }
    void buildTree(vector<int>& A) {
        int len = A.size();
        //pad the tree with zeros
        tree = vector<int>(4*len, 0);
        lower = 0; upper = len - 1;
        toBuild(A, tree, 0, len-1, 0);
    }
    int quarySum(int lo, int hi) {
        return getSum(tree, lo, hi, lower, upper, 0);
    }
    //this function add values to a range of elements of the array
    void addValue(int lo, int hi, int val) {
        toAdd(tree, lo, hi, lower, upper, 0, val);
    }
private:
    /*
    Algorithm
    1. recursively find the leaf nodes with ranges completely covered by the request range
    2. back tracking and update the relevent nodes values
    note: this algorithm can be easily modified to any other operations s.t., subtraction, multiplication, etc.,
    */
    void toAdd(vector<int>& tree, int rlo, int rhi, int lo, int hi, int pos, int val) {
        //if there is no overlap
        if (rhi < lo || rlo > hi) {
            return;
        }
        //if (hi < rlo || lo > rhi) {
        //    return;
        //}
        //if at the corresponding leaf nodes
        if (rlo <= lo && rhi >= hi && lo == hi) {
            tree[pos] += val;
            return;
        }

        int mid = (lo + hi) / 2;
        toAdd(tree, rlo, rhi, lo, mid, 2 * pos + 1, val);
        toAdd(tree, rlo, rhi, mid + 1, hi, 2 * pos + 2, val);
        tree[pos] = tree[2 * pos + 1] + tree[2 * pos + 2];
    }
    /*
    Algorithm:
    there are 3 possible special cases:
        case1: request range completely include the current node range
        case2: request range partially overlap with the current node range
        case3: request range do not overlap with the current node range at all
    actions to take for each case:
        1. return the corresponding value of the current node directly, if case1
        2. look at left and right child of current node recursively, if case2
        3. return 0, if case3
    final action to take for the general case:
        return left + right
    */
    int getSum(vector<int>& tree, int rlo, int rhi, int lo, int hi, int pos) {
        //if no overlap
        if (rhi < lo || rlo > hi) {
            return 0;
        }
        //if complete incude
        if (rlo <= lo && rhi >= hi) {
            return tree[pos];
        }
        int mid = (lo + hi) / 2;
        int left = getSum(tree, rlo, rhi, lo, mid, 2 * pos + 1);
        int right = getSum(tree, rlo, rhi, mid+1, hi, 2* pos+2);
        return left + right;
    }
    /*
    lo: lowerbound of a, starting with 0
    hi: upperhound of a, starting with a.size()
    pos: corresponding position of item in tree, starting with 0 as the root of the tree 
    */
    void toBuild(vector<int>& a, vector<int>& tree, int lo, int hi, int pos) {
        if (lo == hi) {//update the leave nodes
            tree[pos] = a[lo];
            return;
        }
        int mid = (lo + hi) / 2;
        int leftChild = 2 * pos + 1;
        int rightChild = 2 * pos + 2;
        toBuild(a, tree, lo, mid, leftChild);
        toBuild(a, tree, mid + 1, hi, rightChild);
        /// sum left and right subtrees
        tree[pos] = tree[leftChild] + tree[rightChild];
    }
    //--------------------------------
    vector<int> tree;
    int upper;
    int lower;
};

/*

testing case 0:
{1, 2, 3, 4, 5, 6}

testing case 1:
{ 18, 17, 13, 19, 15, 11, 20, 12, 33, 25 };
Tree after construction:
{ 183, 82, 101, 48, 34, 43, 58, 35, 13, 19, 15, 31, 12, 33, 25, 18, 17, 0, 0, 0, 0, 0, 0, 11, 20, 0, 0, 0, 0, 0, 0 };
*/

int main() {
    Solution S;
    vector<int> a =  {1,2,3,4,5,6};
    S.buildTree(a);
    cout << S.quarySum(1, 4) << endl;//expect 14
    S.addValue(2, 2, 3);
    cout << S.quarySum(1, 4) << endl;//expect 17
    S.addValue(2, 3, 2);
    cout << S.quarySum(1, 4) << endl;//expect 21
    return 0;
}
```
