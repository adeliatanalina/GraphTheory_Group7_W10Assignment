# Theory Graph - Week 10 Assignment

**Group 7**
| Name | NRP | Class |
| ---- | --- | ----- |
| Nuha Usama Okbah | 5025241005 | IUP |
| Embun Nabila Rasendriya Az Zahra  | 5025241009 | IUP |
| Almira Nayla Felisitha  | 5025241014 | IUP |
| Adelia Tanalina Yumna  | 5025241078 | IUP |

## 1. Hungarian Algorithm
**A. Code**
```c
#include <stdio.h>
#include <limits.h>

#define MAXN 100

int cost[MAXN][MAXN];       // biaya asli
int biased[MAXN][MAXN];     // biaya dibiaskan utk tie-break
int u[MAXN+1], v[MAXN+1], p[MAXN+1], way[MAXN+1];

int hungarian(int n) {
    for (int i = 0; i <= n; ++i) u[i] = v[i] = p[i] = way[i] = 0;

    for (int i = 1; i <= n; i++) {
        p[0] = i;
        int j0 = 0;
        int minv[MAXN+1];
        char used[MAXN+1] = {0};
        for (int j = 0; j <= n; j++) minv[j] = INT_MAX;

        do {
            used[j0] = 1;
            int i0 = p[j0], delta = INT_MAX, j1 = 0;

            for (int j = 1; j <= n; j++) if (!used[j]) {
                int cur = biased[i0-1][j-1] - u[i0] - v[j];
                if (cur < minv[j]) { minv[j] = cur; way[j] = j0; }
                if (minv[j] < delta || (minv[j] == delta && j > j1)) {
                    delta = minv[j]; j1 = j;
                }
            }

            for (int j = 0; j <= n; j++) {
                if (used[j]) { u[p[j]] += delta; v[j] -= delta; }
                else { minv[j] -= delta; }
            }
            j0 = j1;
        } while (p[j0] != 0);

        do {
            int j1 = way[j0];
            p[j0] = p[j1];
            j0 = j1;
        } while (j0);
    }

    int result = 0;
    int assign_col_for_row[MAXN];
    for (int j = 1; j <= n; j++) if (p[j]) {
        assign_col_for_row[p[j]-1] = j-1;                
        result += cost[p[j]-1][j-1];                     
    }

    printf("\nPicked solution:\n");
    for (int i = 0; i < n; i++) {
        printf("Bulldozer %d : Construction site %d (%d km)\n",
               i+1, assign_col_for_row[i]+1, cost[i][assign_col_for_row[i]]);
    }
    printf("Distance sum = %d km\n", result);
    return result;
}

int main() {
    int n;
    scanf("%d", &n);
    for (int i = 0; i < n; i++)
        for (int j = 0; j < n; j++)
            scanf("%d", &cost[i][j]);


    for (int i = 0; i < n; i++) {
        int target = n - i;                
        for (int j = 0; j < n; j++) {
            int bonus = ((j+1) == target) ? 0 : 1;   
            biased[i][j] = cost[i][j] * 1000 + bonus;
        }
    }

    hungarian(n);
    return 0;
}

```

**B. Explanation**
1) Initialize
   - All potentials and path arrays start from zero.
```c
for (int i = 1; i <= n; i++) {
    p[0] = i;
    int j0 = 0;
    int minv[MAXN+1];
    char used[MAXN+1] = {0};
    for (int j = 0; j <= n; j++) minv[j] = INT_MAX;
```

2) Process each row (worker)
   - We insert one row (worker) at a time into the matching.
   - `p[0]` is a dummy "starting column" for the current augmenting path.
   - `minv[j]` keeps the smallest reduced cost seen so far for each column `j`.
   - `used[j]` marks which columns are already visited.
```c
for (int i = 1; i <= n; i++) {
    p[0] = i;
    int j0 = 0;
    int minv[MAXN+1];
    char used[MAXN+1] = {0};
    for (int j = 0; j <= n; j++) minv[j] = INT_MAX;
```

3) Build the augmenting path
   - `i0 = p[j0]` is the row currently assigned to column `j0`.
   - For each unvisited column j, we compute the reduced cost using potentials.
   - We keep track of the smallest cost `(minv[j])`.
   - Tie-breaking rule: if two columns have equal reduced cost, pick the column with a larger index (`j > j1`) — this is how we make it prefer 4→3→2→1 pattern.
```c
do {
    used[j0] = 1;
    int i0 = p[j0], delta = INT_MAX, j1 = 0;

    for (int j = 1; j <= n; j++) if (!used[j]) {
        int cur = biased[i0-1][j-1] - u[i0] - v[j];
        if (cur < minv[j]) { minv[j] = cur; way[j] = j0; }

        if (minv[j] < delta || (minv[j] == delta && j > j1)) {
            delta = minv[j]; j1 = j;
        }
    }
```

4) Update potentials and distances
   - This step adjusts all `u` and `v` so that at least one new reduced cost becomes zero, allowing the path to extend.
   - The loop continues until we reach an unassigned column (`p[j0] == 0`).
```c
    for (int j = 0; j <= n; j++) {
        if (used[j]) { u[p[j]] += delta; v[j] -= delta; }
        else { minv[j] -= delta; }
    }
    j0 = j1;
} while (p[j0] != 0);
```

5) Reconstruct the path
   - We backtrack the augmenting path using `way[]`, flipping the matched/unmatched edges to update the assignment.
```c
do {
    int j1 = way[j0];
    p[j0] = p[j1];
    j0 = j1;
} while (j0);
```

6) Extract the result
   - Now we have a complete matching.
   - For each row, we store which column it’s matched to.
   - We compute the total real cost using the original cost matrix (not the biased one).
```c
int result = 0;
int assign_col_for_row[MAXN];
for (int j = 1; j <= n; j++) if (p[j]) {
    assign_col_for_row[p[j]-1] = j-1;
    result += cost[p[j]-1][j-1];
}
```

7) Print the result
   - 1-based printing (same as your slide).
   - Total sum displayed at the end.
```c
for (int i = 0; i < n; i++) {
    printf("Bulldozer %d : Construction site %d (%d km)\n",
           i+1, assign_col_for_row[i]+1, cost[i][assign_col_for_row[i]]);
}
printf("Distance sum = %d km\n", result);
```

The `main()` function

8) Read input
   - Reads the size and the original cost matrix.
```c
scanf("%d", &n);
for (int i = 0; i < n; i++)
    for (int j = 0; j < n; j++)
        scanf("%d", &cost[i][j]);
```

9) Create the biased cost matrix
   - Multiply the cost by 1000 to keep differences between real costs dominant.
   - Add a bonus (0 or 1) as a tie-breaker:
       - For each row i, only the “preferred” column (target) gets bonus = 0.
       - All other columns get bonus = 1.
   - So if two assignments have the same real total cost, the one that uses more “preferred” columns wins.
   - This makes the algorithm pick exactly (1→4, 2→3, 3→2, 4→1).

```c
for (int i = 0; i < n; i++) {
    int target = n - i; // For n=4: row0→4, row1→3, row2→2, row3→1
    for (int j = 0; j < n; j++) {
        int bonus = ((j+1) == target) ? 0 : 1;
        biased[i][j] = cost[i][j] * 1000 + bonus;
    }
}
```

10) Run and print
```c
hungarian(n);
```

**C. Input-Output Sample**

**Input**
```
4
90 75 75 80
35 85 55 65
125 95 90 105
45 110 95 115
```

**Output**
```
Picked solution:
Bulldozer 1 : Construction site 4 (80 km)
Bulldozer 2 : Construction site 3 (55 km)
Bulldozer 3 : Construction site 2 (95 km)
Bulldozer 4 : Construction site 1 (45 km)
Distance sum = 275 km
```




## 1. Welsh-Powell Algorithm
**A. Code**
```c
#include <stdio.h>
#include <stdlib.h>

#define MAX 100

int graph[MAX][MAX];
int degree[MAX];
int color[MAX];
int n; // number of nodes

void calculateDegrees() {
    for (int i = 0; i < n; i++) {
        degree[i] = 0;
        for (int j = 0; j < n; j++) {
            if (graph[i][j])
                degree[i]++;
        }
    }
}

void sortNodesByDegree(int nodes[]) {
    for (int i = 0; i < n; i++)
        nodes[i] = i;

    for (int i = 0; i < n - 1; i++) {
        for (int j = i + 1; j < n; j++) {
            if (degree[nodes[i]] < degree[nodes[j]]) {
                int temp = nodes[i];
                nodes[i] = nodes[j];
                nodes[j] = temp;
            }
        }
    }
}

void welshPowell() {
    int nodes[MAX];
    calculateDegrees();
    sortNodesByDegree(nodes);

    for (int i = 0; i < n; i++)
        color[i] = 0;

    int currentColor = 1;
    for (int i = 0; i < n; i++) {
        int node = nodes[i];
        if (color[node] == 0) {
            color[node] = currentColor;
            for (int j = i + 1; j < n; j++) {
                int nextNode = nodes[j];
                if (color[nextNode] == 0) {
                    int canColor = 1;
                    for (int k = 0; k < n; k++) {
                        if (graph[nextNode][k] && color[k] == currentColor) {
                            canColor = 0;
                            break;
                        }
                    }
                    if (canColor)
                        color[nextNode] = currentColor;
                }
            }
            currentColor++;
        }
    }
}

void printColors() {
    for (int i = 0; i < n; i++) {
        printf("Node %d -> Color %d\n", i, color[i]);
    }
}

int main() {
    printf("Enter number of nodes: ");
    scanf("%d", &n);

    printf("Enter adjacency matrix:\n");
    for (int i = 0; i < n; i++)
        for (int j = 0; j < n; j++)
            scanf("%d", &graph[i][j]);

    welshPowell();
    printColors();

    return 0;
}
```

**B. Explanation**
1) Step 1: Input the Graph
   - You enter the number of nodes and the adjacency matrix.
   - The adjacency matrix tells us which nodes are connected (1 = edge, 0 = no edge).
```c
printf("Enter number of nodes: ");
scanf("%d", &n);

printf("Enter adjacency matrix:\n");
for (int i = 0; i < n; i++)
    for (int j = 0; j < n; j++)
        scanf("%d", &graph[i][j]);
```

2) Step 2: Calculate Degree of Each Node
   - For each node, count how many edges it has (i.e., how many 1s in its row).
   - This gives us the degree of each node.
   - Nodes with higher degrees are more connected and harder to color, so we prioritize them.
```c
void calculateDegrees() {
    for (int i = 0; i < n; i++) {
        degree[i] = 0;
        for (int j = 0; j < n; j++) {
            if (graph[i][j])
                degree[i]++;
        }
    }
}
```

3) Step 3: Sort Nodes by Degree (Descending)
   - Create a list of node indices.
   - Sort them so that nodes with the highest degree come first.
   - This ensures we color the most constrained nodes first.
```c
void sortNodesByDegree(int nodes[]) {
    for (int i = 0; i < n; i++)
        nodes[i] = i;

    for (int i = 0; i < n - 1; i++) {
        for (int j = i + 1; j < n; j++) {
            if (degree[nodes[i]] < degree[nodes[j]]) {
                int temp = nodes[i];
                nodes[i] = nodes[j];
                nodes[j] = temp;
            }
        }
    }
}
```

4) Step 4: Assign Colors
   - Start with the first node in the sorted list.
   - Assign it the first color (e.g., Color 1).
   - Then go through the rest of the nodes:
      - If a node is not adjacent to any node already colored with the current color, assign it that color.
      - Otherwise, skip it and try again with the next color.
```c
void welshPowell() {
    int nodes[MAX];
    calculateDegrees();
    sortNodesByDegree(nodes);

    for (int i = 0; i < n; i++)
        color[i] = 0;

    int currentColor = 1;
    for (int i = 0; i < n; i++) {
        int node = nodes[i];
        if (color[node] == 0) {
            color[node] = currentColor;
            for (int j = i + 1; j < n; j++) {
                int nextNode = nodes[j];
                if (color[nextNode] == 0) {
                    int canColor = 1;
                    for (int k = 0; k < n; k++) {
                        if (graph[nextNode][k] && color[k] == currentColor) {
                            canColor = 0;
                            break;
                        }
                    }
                    if (canColor)
                        color[nextNode] = currentColor;
                }
            }
            currentColor++;
        }
    }
}
```

5) Step 5: Output the Result
   - Increase the color number (Color 2, Color 3, etc.) and repeat the process.
   - Continue until every node has a color and no two adjacent nodes share the same color.
```c
void printColors() {
    for (int i = 0; i < n; i++) {
        printf("Node %d --> Color %d\n", i, color[i]);
    }
}
```


**C. Input-Output Sample**

**Input**

