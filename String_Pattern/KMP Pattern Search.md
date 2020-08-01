## KMP Pattern Search
---------------------------------------------------------
### Problem with naive algorithm

Assume there is a text T and pattern P, we want to find if P exists in T, Then in naive algorithm:
1. at any letter with index i in T, we use as starting point for comparison with P.
2. at any point when comparison found a mismatch, we start over from index i+1.
3. continue above steps until: either P is found, or P is not found and all letter in T were exhausted.

Example:
T = "aaaaaaaaaab" of length 11, we use m to denote for it.
P = "aab" of length 3, we use n to denote for it.

starting from begining letter of T to compare with letters in T and P, we will find the mismatch at index 2 (0 based indices).
We will then start with second letter in T and do the same thing with P, utill the mismatch found at index 3.
This process is repeated until either P is found or P is not found.

In the worst case scenario, every letter in P is compared for every letter in T, therefore the Time complexity is: O(mn)

The problem with this algorithm is that the same letter in T were compared repeatedly even it has already been compared.

Looking at the above example: 
In the first round of comparison, when find the first mismatch, we stop the comparison and start with second letter in T for 2nd round of comparison.
During second round of comparison, the second letter in T was already compared during the first round of comparison, which causing the above
mentioned problem.

------------------------------------------------------------

### To address above mentioned problem in KMP

How to avoid the already compared letters in T from being compared again?

we will look at the same example above:
T = "aaaaaaaaaab" of length 11, we use m to denote for it.
P = "aab" of length 3, we use n to denote for it.

The question is how to reduce the # of repeated comparisons?

One idea is to instead of repeatedly compare letters in T, we compare letters in P. This is because T should be longer than P.

let the first mismatch index be i in T and j in P, that indictes that T[0,...,i-1] must be equal to P[0,...,j-1], and T[i] != P[j].

In naive algorithm, we would start at comparison again with T[1,..., n+1-1] and P[0,...,n-1], which will cause the already compared
letters in T[1,...,n-1] being compared again. The reason for that is bc that although the match between T and P failed with T[0] as
the first letter, there may be a successful match with T[1] as the first letter to begin the comparion. 

So, the question here is: is it possible to have any successful match with the already compared in letters in T[1,...,n-1] with P?

Let length of common prefix/suffix of P[0,...,j-1] be k, then P[0,...,k-1] must equal to P[j-1-k+1,..., j-1]. The fact that the mismatch
for the first round of comparison happens at T[i] and P[j], there must be a match between P[j-1-k+1,..., j-1] and T[i-1-k+1, i-1]. Following that,
there must a match between P[0,...,k-1] and T[i-1-k+1, i-1]. Therefore, we could move j back to k-1, and start the new round of the comparison
with T[i] and P[k], because we know that P[0,...k-1] is equal to T[i-1-k+1, i-1]. 

If that still does not work, then it would be no other subset of T[0,...,i-1] and P[0,...,j-1] that would make a successful match. In that case, we
could start the new comparison between T and P, with T[i] and P[0] as the fresh begining.

Example of prefix/suffix of a pattern P:

1.  abcdabeabf
    000120120
    
2.  abcdeabfabc
    00000120123
    
3.  aaaabaacd
    012301200

Algorithm for calculating the prefix/suffix array:

It turns out the algorithm would be the same as the one described above.
In other words, KMP is actually performing the same algorithm/idea twice:
The 1 st is to find the prefix/suffix array
The 2 nd is to find the position of the pattern in the text.

In the process of finding prefix/suffix array array, we could consider the prefix as pattern and the task is to find its position in the text:

let i = 1, j = 0, s be the pattern, and dp[] be the prefix/suffix array we do:
1. if (s[i] == s[j]), i++, j++
2. if (s[i] != s[j]) && j == 0, i++
3. if (s[i] != s[j]) && j > 0, j = dp[j-1]

In the last step (step 3 above), we set j = dp[j-1]. This is because we consider the process as searching common prefix/suffix in s.
The fact that j > 0 indicates that there are some matchs before the mismatch s[i] != s[j]. Therefore instead of searching from begining,
we could search from one position after the already found prefix/suffix, and that position is stored in dp[j-1].

As an example: let s = "aabaaac", the dp = [0101220].
Using above algorithm, when we encounter the mismatch where i = 5, j = 2, and dp = [0101200] at this point. Becuase s[5] != S[j], we will
need to update j. Because there are already matchs s[0,1] and s[3,4]. So, we want to set value of j to the index of the position just after
the position of the last prefix of s[0,1], which is dp[1] or dp[j-1]. We will not change i, because we will ned to compare s[i] and [j] (with
updated j value) again for the prefix.


Time Complexity Analysis (KMP):
T(n) = O(m)

----------------------------------------------------------------------------------------------------------------------------------

## Example: Leetcode 28. Implement strStr()

Implement strStr().

Return the index of the first occurrence of needle in haystack, or -1 if needle is not part of haystack.

<pre>
Example 1:
Input: haystack = "hello", needle = "ll"
Output: 2

Example 2:
Input: haystack = "aaaaa", needle = "bba"
Output: -1
</pre>

------------------------------------------------------------
#### Solution KMP
```c++

class Solution {
public:
    int strStr(string haystack, string needle) {
        if (needle.empty() || haystack == needle) { return 0; }
        vector<int> dp = prefixLengths(needle);
        ///KMP
        int m = haystack.size();
        int n = needle.size();
        for (int i = 0, j = 0; i < m;) {

            if (haystack[i] == needle[j]) {
                i++;
                j++;
                //check if all chars in needle are matched
                if (j >= n) {
                    return i - n;
                }
                continue;
            }

            if (haystack[i] != needle[j]) {
                if (j == 0) {
                    i++;
                }
                else if (j > 0) {
                    j = dp[j - 1];
                }
            }

        }
        return -1;
    }
private:
    vector<int> prefixLengths(string s) {
        int len = s.size();
        vector<int> res(len, 0);
        for (int i = 1, j = 0; i < len;) {
            if(s[i] == s[j]){
                res[i] = j+1;
                i++; j++;
            }
            else if(s[i] != s[j] && j == 0){
                i++;
            }
            else if(s[i] != s[j] && j > 0){
                j = res[j-1];
            } 
        }
        return res;
    }
};


```







