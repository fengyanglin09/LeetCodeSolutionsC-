## Segment Tree

In computer science, a segment tree, also known as a statistic tree, is a tree data structure used for storing information about intervals, or segments. 
It allows querying which of the stored segments contain a given point. 
It is, in principle, a static structure; that is, it's a structure that cannot be modified once it's built. A similar data structure is the interval tree.

A segment tree for a set I of n intervals uses O(n log n) storage and can be built in O(n log n) time.
Segment trees support searching for all the intervals that contain a query point in O(log n + k), k being the number of retrieved intervals or segments.

Applications of the segment tree are in the areas of computational geometry, and geographic information systems.

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
