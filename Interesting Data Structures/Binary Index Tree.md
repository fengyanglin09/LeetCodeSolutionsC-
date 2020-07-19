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
......

Then we can view an array in a form of tree using index binary forms,
for example, if root index is 0, then we want all of its children node 
to have index i that can be converted to 0 by applying following steps:
1. compute i's 2's complement
2. let i & it's 2's complement (this is to get the rightmost significant bit, note in binary form 1s are significant, 0s otherwise)
3. let i substract the result from 2

for example: let i = 4, then
1. i = 00000100, in its binary form
2. it's 2's completment is inverse + 1: 11111011 + 1 = 11111110
3. i & 2's complement = 00000100 & 11111110 = 00000100
4. i - (i & 2's complement) = 00000100 - 00000100 = 0
note the above steps are also used to to find parent of any given node of index i

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
note that j1 is the parent index of i1, and we do (2^k1 - 1) because the array index starts at zero. 

Assume i2 is computed by applying above computing steps on i1, then there can be only 2 cases:

case1: i2 is the sibling node of i1, so both i1 and i2 share the same parent: i2 = j1 + 2^k2. Then prefix sum range represented by i2:
            [j1, j1+(2^k2 - 1)], where k2 > k1 because the bit were added by one according to the above computing steps
        therefore the range represented by i2 includes the range represented by i1, which indicates that the updates on ele at i1 should
        also be applied to i2, because i1 is used to compute the prefix sum for the range represented by i2.
        
        Example of case1: i1 = 5, i2 = 6, parent of i1 and i2 = 4

case2: i2 is the next right sibling of the parent of i1. Lets denote the direct parent of i1 as p1, and the next right sibling of p1 as p2.
       Then, the range represented by p2 must include range represented by p1 according the above analysis. Similarly, the range of p1 must
       include range of i1, because p1 is parent of i1. Therefore, the updates on ele at i1 should also be applied to i2.

       Example of case2: i1 = 6, i2 = 8, where parent of 6 is 4, and parent of 4 and 8 is 0
//*******************************************************************************************************************

Analysis:
T(n) = O(lgn), for look up, update
T(n) = O(n), for building the tree
</pre>

-----------------------------------------------------------------------------------------------------------------
### My Implementation of BIT



