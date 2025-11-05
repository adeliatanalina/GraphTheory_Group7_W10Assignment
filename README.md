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

#define N 4  // Size of the cost matrix

int cost[N][N] = {
    {9, 2, 7, 8},
    {6, 4, 3, 7},
    {5, 8, 1, 8},
    {7, 6, 9, 4}
};

int assignment[N];

void hungarian() {
    int rowCover[N] = {0}, colCover[N] = {0};
    int marked[N][N] = {0};
    int i, j;

    // Step 1: Subtract row minima
    for (i = 0; i < N; i++) {
        int min = cost[i][0];
        for (j = 1; j < N; j++)
            if (cost[i][j] < min)
                min = cost[i][j];
        for (j = 0; j < N; j++)
            cost[i][j] -= min;
    }

    // Step 2: Subtract column minima
    for (j = 0; j < N; j++) {
        int min = cost[0][j];
        for (i = 1; i < N; i++)
            if (cost[i][j] < min)
                min = cost[i][j];
        for (i = 0; i < N; i++)
            cost[i][j] -= min;
    }

    // Step 3: Mark zeros
    for (i = 0; i < N; i++) {
        for (j = 0; j < N; j++) {
            if (cost[i][j] == 0 && !rowCover[i] && !colCover[j]) {
                marked[i][j] = 1;
                rowCover[i] = 1;
                colCover[j] = 1;
            }
        }
    }

    // Step 4: Extract assignment
    for (i = 0; i < N; i++) {
        for (j = 0; j < N; j++) {
            if (marked[i][j]) {
                assignment[i] = j;
                break;
            }
        }
    }
}

int main() {
    hungarian();

    printf("Optimal assignment:\n");
    for (int i = 0; i < N; i++) {
        printf("Worker %d assigned to Job %d (Cost: %d)\n", i, assignment[i], cost[i][assignment[i]]);
    }

    return 0;
}
```

**B. Explanation**


**C. Input-Output Sample**





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


