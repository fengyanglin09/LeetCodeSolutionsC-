## Problem Statement

In 0-1 Knapsack problem, we are given a set of items, each with a weight and a value and we need to determine the number of each item to include in a collection so that the total weight is less than or equal to a given limit and the total value is as large as possible.

Please not the items are indivisible; we can either take an item or not (0-1 property). For example:

<pre>
Input:
value = [20, 5, 10, 40, 15, 25]
weight = [1, 2, 3, 8, 7, 4]
int W = 10

Output: Knapsack value is 60
value = 20 + 40 = 60
weight = 1 + 8 = 9 < W
</pre>

------------------------------------------------------------------------------------------
### My Codes:
```c++
/*
0-1 knapsack problem: variations
for all items, we try all possible knapsack capacities
let dp[i][j] be the max value of first i items that could possibly be put into a knapsack with capacity j
dp[i][j] = dp[i-1][j], if the i th item is to heavy/large to be put to the knapsack, i.e., w[i] > j
dp[i][j] = max(dp[i-1][j], dp[i - 1][j - w[i]] + v[i]), we can either take the item or leave the item, we want which ever the value is highest 
dp[0][j] = 0, value is zero if no item is taken
dp[i][0] = 0, value is zero if capacity of knapsack is zero

*/
class Solution {
public:
    /*
    find1 is exact translation from the dp formula above
    //T(n) = O(N*W)
    //S(n) = O(N*W)
    */
    int find1(int N, int W, vector<int>& w, vector<int>& v) {
        vector<vector<int>> dp(N + 1, vector<int>(W+1, 0));
        for (int i = 0; i <= N; i++) {
            for (int j = 0; j <= W; j++) {
                if (i == 0 || j == 0) {
                    dp[i][j] = 0;
                    continue;
                }
                if (w[i-1] > j) {
                    //dp[i][j] = dp[i - 1][j];
                    continue;
                }
                dp[i][j] = max(dp[i-1][j], dp[i - 1][j - w[i-1]] + v[i-1]);
            }
        }
        return dp[N][W];
    }

    /*
    find2 is the same as find1, but record the only the previous row of the dp array
    ,this is bc the calculation of the current row only depends on the calculation of the previous row
    //T(n) = O(N*W)
    //S(n) = O(2*W)
    */
    int find2(int N, int W, vector<int>& w, vector<int>& v) {
        vector<int> dp(W + 1, 0);
        vector<int> tmp(W + 1, 0);
        for (int i = 0; i <= N; i++) {
            for (int j = 0; j <= W; j++) {
                if (i == 0 || j == 0) {
                    dp[j] = 0;
                    continue;
                }
                if (w[i - 1] > j) {
                    //dp[j] = tmp[j];
                    continue;
                }
                dp[j] = max(tmp[j], tmp[j - w[i - 1]] + v[i - 1]);
            }
            tmp = dp;
        }
        return dp[W];
    }

    /*
    find3: modification from find1 and find2
    in order to further reduce space, we could remove tmp and reverse read from dp[j] as shown below
    this is because:
    1.inorder to calculate the current dp[i][j], we will need the information from dp[i-1][j], i.e., from the last iteration
    2.at the begining of each outer i iteration, dp[i][j] will store the information from the previous iteration, i.e., dp[i-1][j]
    3.from the formula: dp[j] = max(dp[j], dp[j - w[i - 1]] + v[i - 1]), we can see that:
        3.0. to calculate dp[j], we only need dp[j] from previous iteration i.e., dp[i-1][j] and dp[j - w[i - 1]] from previous iteration i.e., dp[i-1][j - w[i - 1]]
        3.1. w[i-1] is always positive, therefore dp[j] >= dp[j - w[i - 1]] 
        3.2. dp[j] will be value of dp[i-1][j] at begining of each iteration i, and dp[j-w[i-1]] be value of dp[i-1][j-w[i-1]]
        3.3. after dp[j] is updated with max(dp[j], dp[j - w[i - 1]] + v[i - 1]), its value will not be needed for calculating dp[j-1]
    4. therefore, we could calculate dp[j] for the i the iteration by reversely reading from dp[j], bc it will store the calculation result of previous iteration

    Question: can we not read reversly?
    ans: No. If we read from left to right instead, to calculate pd[j], we will need dp[j] and dp[j - w[i - 1]] from previous iteration.
         Both dp[j] and dp[j - w[i - 1]] will be available for the first iteration, but dp[j] will be updated after that. 
         That means value at dp[j] for the previous iteration i-1 will be replaced with value for dp[j] for the current iteration i.
         Because j - w[i-1] < j and dp[j-w[i-1]] may be already updated to current iteration i, it will be impossible to calculate for other dp[j] at current iteration.
    //T(n) = O(N*W)
    //S(n) = O(W)
    */
    int find3(int N, int W, vector<int>& w, vector<int>& v) {
        vector<int> dp(W + 1, 0);
        //vector<int> tmp(W + 1, 0);
        for (int i = 0; i <= N; i++) {
            for (int j = W; j >= 0; j--) {
                if (i == 0 || j == 0) {
                    dp[j] = 0;
                    continue;
                }
                if (w[i - 1] > j) {
                    //dp[j] = dp[j];
                    continue;
                }
                dp[j] = max(dp[j], dp[j - w[i - 1]] + v[i - 1]);
            }
            //tmp = dp;
        }
        return dp[W];
    }
private:
};
/*
testing case:
N = 10, W = 50
w[] = {0, 12, 15, 16, 16, 10, 21, 18, 12, 17, 18} // 1 based indexing
v[] = {0, 3, 8, 9, 6, 2, 9, 4, 4, 8, 9}
*/
int main() {
    Solution S;
    int N = 10;
    int W = 50;
    vector<int> w = { 12, 15, 16, 16, 10, 21, 18, 12, 17, 18 };
    vector<int> v = { 3, 8, 9, 6, 2, 9, 4, 4, 8, 9 };
    cout << S.find1(N, W, w, v) << endl;
    cout << S.find2(N, W, w, v) << endl;
    cout << S.find3(N, W, w, v) << endl;
    return 0;
}
```

