## Binary Index Tree - used to compute prefix sum efficiently
----------------------------------------------------------------------------------------
### What is Binary Index Tree (BIT)

----------------------------------------------------------------------------------------
### My Analysis:
<pre>
The Binary Index Tree (BIT) is an interesting DS that can be used to calculate for prefix sum efficently
the idea is based on the fact that any integers or index can be written in binary form:
1 = 000001 = 0 * 2^0 (store sum in [0,0])
2 = 000010 = 0 * 2^1 (store sum in [0,1])
4 = 000100 = 0 * 2^2 (store sum in [0,3])

Structure of the BIT

Assume an integer array A with size n
1. length of BIT is one more than the length of array, i.e., |BIT| = n+1
2. BIT starts with index 0, but the 0th slot does not contain any element. The elements were stored in BIT starting with index 1
3. the root node BIT must be zero, and the second level nodes can be reduced to zero by removing the rightmost significant digit
             i.e., 1, 2, 4 listed above should be the second level nodes
4. to generalize, the ith level nodes can be reduced to i-1 th level nodes by removing the rightmost significant digit
5. let i be an arbitrary node in BIT and i = p * 2^k where k is the position of the rightmost significant digit, then:
              5.1 the parent node of i is p
              5.2 the prefix sum represented by i is from range [i, i+(2^k - 1)], i.e., (node)4 = 0 * 2^2 so its prefix sum is in range [0, 0+2^2-1] = [0,3] 

......
Calculate Prefix Sum:
Then we can view an array in a form of tree using index binary forms,
for example, if root index is 0, then we want all of its children node 
to have index i that can be converted to 0 by applying following steps:
1. compute i's 2's complement
2. let i & it's 2's complement (this is to get the rightmost significant bit, note in binary form 1s are significant, 0s otherwise)
3. let i substract the result from 2

for example: let i = 4, then
1. i = 00000100, in its binary form
2. it's 2's complement is inverse + 1: 11111011 + 1 = 11111110
3. i & 2's complement = 00000100 & 11111110 = 00000100
4. i - (i & 2's complement) = 00000100 - 00000100 = 0

note1: the above steps are also used to to find parent of any given node of index i
note2: the 2's complement is used only to find the rightmost significant bit

To calculate the prefix sum for A[0,i], we sum nodes in the BIT starting at tree[i+1] to tree[0]
//*******************************************************************************************************************
Proof: given an arbitrary index i, the above steps will calculate the prefix sum A[0,...,i]

if i is index for A, then the corresponding index in tree is i + 1, then lets look at the following two cases:

case1: if the parent of i + 1 is 0, then i + 1 = 0 + 2^k, which represents range [0, 0 + 2^k - 1] == [0, i]

case2: if the parent of i + 1 is j, then i + 1 = j + 2^k, which represents range [j, j + 2^k - 1] = [j, i].
       Asssuming parent of j is 0, then it will represent [0, j - 1] for the same reasoning above. Therefore
       sum all the parent node from node i + 1 in the tree will get the range [0,j-1] + [j,i] = [0, i]

//*******************************************************************************************************************
Update:
If in the situation where the tree needs to be updated, only the sibling nodes sharing same parent and 
some nodes at the parent level need to be updated. They need to be updated, because the updated node value
is used to compute their values. How to find nodes that need to be update? We can apply the following steps:

Given an index i, we need to find all the nodes whose prefix sum values was calculated using the i th element:
1. compute the rightmost significent bit of i in its binary form, (using the method above)
2. increase that bit of i to find the index of the next relevent node 

for example: let i = 5, then the next node to update:
1. i = 5 = 00000101
2. 2's complement of i = 11111010+1 = 11111011
3. i & 2's = 00000101 & 11111011 = 00000001
4. i + (i & 2's) = 00000101 + 00000001 = 00000110, which is the next node with same parent as i

//*******************************************************************************************************************
Proof: the above update steps will only update the elements that needs that updated ele for computing prefix sum

let A be the integer array and let i1 be an arbitrary index of A,
if i1 = j1 + 2^k1, where j1 is the index of the parent of i1, then the prefix sum represented by i1 is: 
            [j1, j1+(2^k1 - 1)]
where it can be interpreted as: starting from index i1, we consider the next 2^k1 eles for the range of the prefix sum.
note that j1 is the parent index of i1, and we do (2^k1 - 1) because the ele at j1 is included in the range. 

Assume i2 is computed by applying above computing steps on i1, then there can be only 2 cases:

case1: i2 is the sibling node of i1, so both i1 and i2 share the same parent: i2 = j1 + 2^k2. Then prefix sum range represented by i2:
            [j1, j1+(2^k2 - 1)], where k2 > k1 because the bit were added by one according to the above computing steps
        therefore the range represented by i2 includes the range represented by i1, which indicates that the updates on ele at i1 should
        also be applied to i2, because i1 is used to compute the prefix sum for the range represented by i2.
        
        Example of case1: i1 = 5, i2 = 6, parent of i1 and i2 = 4

case2: i2 is the next right sibling of the parent of i1. Lets denote the direct parent of i1 as p1, and the next right sibling of p1 as p2.
       Then, the range represented by p2 must include range represented by p1 according the above analysis. Similarly, the range of p2 must
       include range of i1, because p2 is got by adding rightmost significant digit to i1. Therefore, the updates on ele at i1 should also 
       be applied to i2.

       Example of case2: i1 = 6, i2 = 8, where parent of 6 is 4, and parent of 4 and 8 is 0
//*******************************************************************************************************************

Analysis:
T(n) = O(lgn), for look up, update
T(n) = O(nlgn), for building the tree
</pre>

-----------------------------------------------------------------------------------------------------------------
### My Implementation of 2-D BIT Version1 - need to understand 1-d BIT to understand 2-D BIT
```c++
/*
2D BIT
Build Tree: O((m*n)lg(m*n))
Update: O(lg(m*n))
QuerySum: O(lg(m*n))
*/
class TwoD_BIT {
public:
    TwoD_BIT() {}
    TwoD_BIT(vector<vector<int>> matrix) {
        int m = matrix.size();
        int n = matrix[0].size();
        tree = vector<vector<int>>(m+1, vector<int>(n+1, 0));
        for (int r = 0; r < m; r++) {
            for (int c = 0; c < n; c++) {
                for (int i = r+1; i <= m; i += (i & -i)) {
                    for (int j = c+1; j <= n; j += (j & -j)) {
                        tree[i][j] += matrix[i - 1][j - 1];
                    }
                }
            }
        }
    }
    /////
    void update(int r, int c, int val, int old) {
        int m = tree.size();
        int n = tree[0].size();
        for (int i = r + 1; i < m; i += (i & -i)) {
            for (int j = c + 1; j < n; j += (j & -j)) {
                tree[i][j] = tree[i][j] - old + val;
            }
        }
    }
    /////
    int prefixSum(int r, int c) {
        int sum = 0;
        for (int i = r + 1; i > 0; i -= (i & -i)) {
            for (int j = c + 1; j > 0; j -= (j & -j)) {
                sum += tree[i][j];
            }
        }
        return sum;
    }
private:
    vector<vector<int>> tree;
};

int main() {

    vector<vector<int>> A = { {1,1,1,1,1,1},
                              {1,1,1,1,1,1},
                              {1,1,1,1,1,1},
                              {1,1,1,1,1,1},
                              {1,1,1,1,1,1},
                              {1,1,1,1,1,1}
                            };
    TwoD_BIT tree(A);
    
    cout << tree.prefixSum(5, 5) << endl;//expect 36

    tree.update(3, 4, 5, 1);

    cout << tree.prefixSum(5, 5) << endl;//expect 40

    return 0;
}

```
-----------------------------------------------------------------------------------------------------------------
### My Implementation of BIT Version2
```c++

/*
Build Tree: O(nlgn)
Update: O(lgn)
QuerySum: O(lgn)
*/
class BIT {
public:
    BIT() {
    }
    BIT(vector<int> nums) {
        int n = nums.size();
        tree = vector<int>(n + 1, 0);
        //build the tree
        for (int i = 0; i < n; i++) {
            int j = i + 1;//j is index of tree node
            while (j <= n) {//propagate to all sibling nodes
                tree[j] += nums[i];
                j = sibling(j);
            }
        }
    }
    /*
    int i: the index for nums (the original data array)
    int val: the value changed at i th postion in nums
    int old: the old value in nums at i th postion that will be replaced by val
    */
    void update(int i, int val, int old) {
        int j = i + 1;
        int n = tree.size();
        while (j < n) {
            tree[j] = tree[j] - old + val;
            j = sibling(j);
        }
    }
    /*
    int i: the index of the range to get the prefix sum starting from begining
    */
    int prefixSum(int i) {
        int sum = 0;
        int j = i + 1;
        while (j > 0) {//j can not be zero, because 0's parent is 0, which is create infinit loop
            sum += tree[j];
            j = parent(j);
        }
        return sum;
    }
    /*
    int i, j: the indices for the range of getting the sum, where j > i
    */
    int sum(int i, int j) {
        return prefixSum(j) - prefixSum(i);
    }
private:
    int parent(int i) {//-i is i's 2's complement
        return i - (i & -i);
    }
    int sibling(int i) {
        return i + (i & -i);
    }
    vector<int> tree;
};

int main() {

    vector<int> A = { 1,2,3,4,5,6 };
    BIT tree(A);

    cout << tree.sum(0, 5) << endl;//expect 20

    tree.update(5, 7, 6);

    cout << tree.sum(0, 5) << endl;//expect 21    

    tree.update(5, 6, 7);

    cout << tree.prefixSum(5) << endl;//expect 21

    return 0;
}

```

-----------------------------------------------------------------------------------------------------------------
### My Implementation of BIT Version1

```c++
class Solution {
public:
    void buildTree(vector<int>& A) {
        int len = A.size();
        /*
        the length of tree is length of A + 1, because the 0 index (root) of tree does not store anything.
        So, if we want the prefix sum in [0, i], we will need to look at i+1 th ele in tree for that reason. 
        */
        tree = vector<int>(len + 1, 0);
        for (int i = 0; i < len; i++) {
            int j = i + 1;
            while (j < len + 1) {
                tree[j] += A[i];
                j = getSib(j);
            }
        }
    }
    int getPrefixSum(int pos) {
        int i = pos + 1;
        int sum = 0;
        while (i != 0) {
            sum += tree[i];
            i = getParent(i);
        }
        return sum;
    }
private:

    int getParent(int i) {
        // get the rightmost bit of i
        int r = rightBit(i);
        // remove that bit
        return i - r;
    }
    int getSib(int i) {
        int r = rightBit(i);
        return i + r;
    }
    int rightBit(int a) {
        int i = 0;
        for (; i < 32; i++) {
            if (a & (1 << i)) {
                break;
            }
        }
        return 1 << i;
    }
    vector<int> tree;
};

/*

testing case 0:
{1, 2, 3, 4, 5, 6}


*/

int main() {
    Solution S;
    vector<int> A = {1,2,3,4,5,6};
    S.buildTree(A);
    cout << S.getPrefixSum(2) << endl; // expect 6
    cout << S.getPrefixSum(4) << endl; // expect 15
    return 0;
}
```

------------------------------------------------------

## Alternative Way For Calculating Parents and Siblings

```c++

/**
     * To get parent
     * 1) 2's complement to get minus of index
     * 2) AND this with index
     * 3) Subtract that from index
     */
    private int getParent(int index){
        return index - (index & -index);
    }
    
    /**
     * To get next
     * 1) 2's complement of get minus of index
     * 2) AND this with index
     * 3) Add it to index
     */
    private int getNext(int index){
        return index + (index & -index);
    }

```

