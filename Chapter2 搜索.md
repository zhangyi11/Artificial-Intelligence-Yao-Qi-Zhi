# 图2.2八数字推盘问题
问题描述：在一块3*3的木板上，有编号为1~8的图块和一个空白区域，与空白区域相邻的图块可以被推入空白区域，我们的目标是从初始布局，通过移动图块达到指定的目标对局。

![image](https://github.com/zhangyi11/Artificial-Intelligence-Yao-Qi-Zhi/blob/main/%E5%9B%BE%E7%89%87/Chapter%202/initial_layout.png?raw=true)![image](https://github.com/zhangyi11/Artificial-Intelligence-Yao-Qi-Zhi/blob/main/%E5%9B%BE%E7%89%87/Chapter%202/final_layout.png?raw=true)

生成上述图片所用到的代码：
```
import matplotlib.pyplot as plt
import numpy as np
import matplotlib.patches as patches

plt.rcParams['font.sans-serif'] = ['SimHei']

# 创建一个 3x3 的数字网格
grid_initial = np.array([[8, 0, 6],
                         [5, 4, 7],
                         [2, 3, 1]])
grid_final = np.array([[0, 1, 2], [3, 4, 5], [6, 7, 8]])

# 颠倒数组，使生成的图像与书中初始布局一致
grid_initial = grid_initial[::-1]
grid_final = grid_final[::-1]

# 创建函数来绘制并保存网格
def plot_grid(grid, title, filename):
    fig, ax = plt.subplots()

    # 设置网格的范围
    ax.set_xlim(-0.5, 2.5)  # x轴从 -0.5 到 2.5
    ax.set_ylim(-0.5, 2.5)  # y轴从 -0.5 到 2.5

    # 绘制 3x3 网格
    for i in range(4):  # 4 条线（包括边界）
        ax.axhline(i - 0.5, color='black', linewidth=2)  # 绘制水平线
        ax.axvline(i - 0.5, color='black', linewidth=2)  # 绘制垂直线

    # 在每个网格格子内添加数字，数字0表示空白格，用灰色填充
    for i in range(3):
        for j in range(3):
            if grid[i, j] == 0:
                ax.add_patch(patches.Rectangle((j-0.5, i-0.5), 1, 1, linewidth = 0, facecolor = 'gray'))
            else:
                ax.text(j, i, str(grid[i, j]), ha='center', va='center', fontsize=16, color='black')

    # 去掉坐标轴的刻度和坐标轴上的数字
    ax.set_xticks([])  # 去掉x轴的刻度
    ax.set_yticks([])  # 去掉y轴的刻度

    # 设置标题
    ax.set_title(title)

    # 保存图像
    fig.savefig(filename)
    plt.close(fig)  # 关闭图形，防止显示多次

# 生成并保存初始布局图像
plot_grid(grid_initial, "八数字推盘问题（初始布局）", "initial_layout.png")

# 生成并保存目标布局图像
plot_grid(grid_final, "八数字推盘问题（目标布局）", "final_layout.png")

# 显示提示信息
print("两张图片已保存为 initial_layout.png 和 final_layout.png")

```
## 使用随机策略解决八数字推盘问题
```
import numpy as np
import random

# 创建一个 3x3 的数字网格
grid_initial = np.array([[8, 0, 6],
                         [5, 4, 7],
                         [2, 3, 1]])

grid_final = np.array([[0, 1, 2],
                       [3, 4, 5],
                       [6, 7, 8]])

# 获取0的位置
rows, cols = np.where(grid_initial == 0)
current_position_0 = (rows[0], cols[0])  # 当前0的位置


# 获取当前位置周围可移动的位置
def possible_position(rows, cols):
    # 计算四个可能的移动位置
    n_position = [(rows-1, cols), (rows+1, cols), (rows, cols-1), (rows, cols+1)]
    # 过滤掉超出边界的位置
    n_position = [(r, c) for r, c in n_position if 0 <= r < 3 and 0 <= c < 3]
    return n_position


# 计数器
count = 0

while True:
    # 随机选择一个可移动的位置
    next_position_0 = random.choice(possible_position(current_position_0[0], current_position_0[1]))

    # 交换当前0位置和目标位置的元素
    grid_initial[current_position_0], grid_initial[next_position_0] = grid_initial[next_position_0], grid_initial[
        current_position_0]

    # 更新当前0的位置
    current_position_0 = next_position_0

    # 增加交换次数
    count += 1

    # 判断是否达到目标布局
    if np.array_equal(grid_initial, grid_final):
        print("总共执行次数", count)
        break
```
我大概试了十多次，最好的执行次数为81573，最坏的执行次数为2878599，该随机策略仍有优化的空间，比如第一次推盘灰色空格向下走，第二次推盘灰色空格就不能再向上走，以保证短期内没有相同的状态空间，相应优化代码编写留给读者自己编写。

## 使用宽度优先搜索解决八数字推盘问题
```
import numpy as np
from collections import deque
import sys

# 创建一个 3x3 的数字网格
grid_initial = np.array([[8, 0, 6],
                         [5, 4, 7],
                         [2, 3, 1]])

grid_final = np.array([[0, 1, 2],
                       [3, 4, 5],
                       [6, 7, 8]])

# 获取当前位置周围可移动的位置
def possible_position(rows, cols):
    # 计算四个可能的移动位置
    n_position = [(rows-1, cols), (rows+1, cols), (rows, cols-1), (rows, cols+1)]
    # 过滤掉超出边界的位置
    n_position = [(r, c) for r, c in n_position if 0 <= r < 3 and 0 <= c < 3]
    return n_position


# 队列和已访问状态集合
q = deque()
visited = set()
parent = {}  # 记录每个状态的父状态

# 计数器
count = 0

# 初始化队列和已访问集合
q.append(grid_initial.tobytes())  # 使用tobytes()来表示状态，numpy数组无法hash，转化成字节后可以hash
visited.add(grid_initial.tobytes())

# 记录父状态和移动方向
parent[grid_initial.tobytes()] = None  # 初始状态没有父状态


while q:
    current_state_bytes = q.popleft()
    current_state = np.frombuffer(current_state_bytes, dtype = int).reshape(3, 3)  # 转换回数组


    if np.array_equal(current_state, grid_final):
        # 回溯最优解
        path = []
        state = current_state_bytes
        while state is not None:
            path.append(np.frombuffer(state, dtype = int).reshape(3, 3))
            state = parent[state]

        # 输出最优解路径
        print("最优解路径：")
        for step in path[::-1]:  # 反向输出路径
            print(step)
            count+=1
        break

    # 找到0的位置
    rows, cols = np.where(current_state == 0)
    current_position_0 = (rows[0], cols[0])

    for next_position_0 in possible_position(rows[0], cols[0]):
        # 交换0和相邻位置的数字
        next_state = current_state.copy()
        next_state[current_position_0], next_state[next_position_0] = next_state[next_position_0], next_state[
            current_position_0]

        # 将新状态转为字节格式，避免重复访问
        next_state_bytes = next_state.tobytes()
        if next_state_bytes not in visited:
            visited.add(next_state_bytes)
            q.append(next_state_bytes)
            # 记录父状态和移动方向
            parent[next_state_bytes] = current_state_bytes
print(f"parent 数组占用内存：{sys.getsizeof(parent)/1024/1024:.2f} MB")
print(f"q占用内存：{sys.getsizeof(q)/1024:.2f} KB")
print(f"visited 数组占用内存：{sys.getsizeof(visited)/1024/1024:.2f} MB")
```
总结：宽度优先搜索方法与问题无关，具有通用性。其优点是不存在死循环问题（上述代码中，如果不设置visited集合则会进入死循环），当问题有解时一定能找到解。然而在搜索过程中，宽度优先搜索需要将下一层的节点放到队列中展开，而每层的节点个数随着层数呈指数增长，所以缺点是算法所需的存储量比较大（宽度优先搜索中parent占10MB、q占用0.61KB、visited占用8MB，八数字推盘问题状态空间有限，最多有9!个状态，故内存消耗较小），考虑在一棵宽度为b、深度为d的树上进行搜索，如果维护访问表（对应上parent数组）空间复杂度为$`O(b^d)`$。

对于不维护访问表的深度优先搜索算法，只需存储当前维护路径上的节点，因此空间复杂度为$`O(bd)`$，故深度优先搜索得以在实际问题中广泛应用的原因（实际问题的状态空间往往很大，比如围棋的状态空间，想要详细了解的读者可以去网上搜索相关资料）

将广度优先搜索中current_state_bytes = q.popleft()修改成current_state_bytes = q.pop()，则变成使用深度优先搜索解决八数字推盘问题的代码示例，在搜索结束后，深度优先搜索中q占用内存：114.05 KB远高于广度优先搜索中q占用的内存0.61KB（深度优先搜索中队列占用的内存可能远大于广度优先搜索中队列占用的内存），在不维护访问表的情况下，深度优先搜索所需的内存远低于广度优先搜索（节省了parent数组占用的10MB内存）。

# 启发式搜索
启发式搜索使用了问题定义本身之外的知识来引导搜索算法。启发式搜索算法依赖于启发式代价函数h，其定义为每个节点到达目标节点代价的估计值。本书介绍了两种基本的启发式搜索方法：贪婪搜索和$`A^*`$搜索。
对于八数字推盘问题常见的启发式函数有两种
  1.曼哈顿距离：每个数字与其目标位置的水平和垂直距离的总和（不懂什么是曼哈顿距离的同学可上网查询相关资料）
  2.不在目标位置的数字个数：计算当前状态与目标状态之间不一致的数字个数
曼哈顿距离更具有普适应，如在迷宫问题中可以用曼哈顿距离作为启发函数，故我们使用曼哈顿距离作为八数字推盘问题的启发函数。
## 贪婪搜索
```
import numpy as np
import heapq

count = 0
# 创建一个 3x3 的数字网格
grid_initial = np.array([[8, 0, 6],
                         [5, 4, 7],
                         [2, 3, 1]])

grid_final = np.array([[0, 1, 2],
                       [3, 4, 5],
                       [6, 7, 8]])

# 获取当前位置周围可移动的位置
def possible_position(rows, cols):
    # 计算四个可能的移动位置
    n_position = [(rows-1, cols), (rows+1, cols), (rows, cols-1), (rows, cols+1)]
    # 过滤掉超出边界的位置
    n_position = [(r, c) for r, c in n_position if 0 <= r < 3 and 0 <= c < 3]
    return n_position


def manhattan_distance(current_state,goal_state):
    distance = 0
    for i in range(8):
        current_pos = np.where(current_state==(i+1))[0],np.where(current_state==(i+1))[1]
        goal_pos = np.where(goal_state==(i+1))[0],np.where(goal_state==(i+1))[1]  # 由于目标状态是不变的，故目标状态的索引值只需求一次，相应优化留给读者。
        distance += abs(current_pos[0]-goal_pos[0])+abs(current_pos[1]-goal_pos[1])
    return distance[0]
# 队列和已访问状态集合
priority_queue = []
visited = set()

# 如果不把numpy数组转化成字节模式放入heapq中会报错，有人知道是为啥吗，莫名其妙的bug。
heapq.heappush(priority_queue,(manhattan_distance(grid_initial,grid_final),grid_initial.tobytes()))
visited.add(grid_initial.tobytes())


while priority_queue:
    current_state_bytes = heapq.heappop(priority_queue)[1]
    current_state = np.frombuffer(current_state_bytes, dtype = int).reshape(3, 3)
    if np.array_equal(current_state,grid_final):
        print(f"总共执行次数：{count}")
        break
    row,col = np.where(current_state==0)[0][0],np.where(current_state==0)[1][0]
    for next_position_0 in possible_position(row,col):
        current_state_copy = current_state.copy()
        current_state_copy[(row,col)],current_state_copy[next_position_0] = current_state_copy[next_position_0],current_state_copy[(row,col)]
        if current_state_copy.tobytes() not in visited:
            visited.add(current_state_copy.tobytes())
            heapq.heappush(priority_queue,(manhattan_distance(current_state_copy,grid_final),current_state_copy.tobytes()))

    count+=1
```
贪婪搜索，总共执行次数：83，相比于深度优先搜索执行次数：16672，加入启发式函数后，大大减少了搜索的次数。本质上，启发式搜索就是有选择的深度搜索，即在一个搜索树中，深度搜索总会优先搜索最左侧的分支，对于添加了启发函数的贪婪搜素而言，搜索时总会搜索启发函数值最小的分支，大大提高了搜索效率。

读者可以尝试将启发函数修改为 *不在目标位置的数字个数* ，比较那个启发函数更高效。启发式搜索的重点就在于针对特定的问题如何找到一个好的启发函数。

## $`A^*`$搜索：贪婪搜索的优化
相比贪婪搜索，$`A^*`$算法采用更为精确的评价函数对扩展节点进行评估，即$`A^*`$算法的评价函数步进利用启发函数，而且还包含起始节点到目前节点的真实损耗（贪婪搜索的评价函数就是启发函数）。令$`g(n)`$表示算法所找的从起始节点S到节点n的实际代价，$`h(n)`$表示启发函数，定义从当前节点到目标节点的最佳路径代价估计（启发函数的结果就是估计值）。

为保证$`A^*`$算法找到最优解，启发函数需要满足$`h(n)≤h^*(n)`$

```
import numpy as np
import heapq

count = 0
loss = 0
# 创建一个 3x3 的数字网格
grid_initial = np.array([[8, 0, 6],
                         [5, 4, 7],
                         [2, 3, 1]])

grid_final = np.array([[0, 1, 2],
                       [3, 4, 5],
                       [6, 7, 8]])

# 获取当前位置周围可移动的位置
def possible_position(rows, cols):
    # 计算四个可能的移动位置
    n_position = [(rows-1, cols), (rows+1, cols), (rows, cols-1), (rows, cols+1)]
    # 过滤掉超出边界的位置
    n_position = [(r, c) for r, c in n_position if 0 <= r < 3 and 0 <= c < 3]
    return n_position


def manhattan_distance(current_state,goal_state):
    distance = 0
    for i in range(8):
        current_pos = np.where(current_state==(i+1))[0],np.where(current_state==(i+1))[1]
        goal_pos = np.where(goal_state==(i+1))[0],np.where(goal_state==(i+1))[1]  # 由于目标状态是不变的，故目标状态的索引值只需求一次，相应优化留给读者。
        distance += abs(current_pos[0]-goal_pos[0])+abs(current_pos[1]-goal_pos[1])
    return distance[0]

def find_index(lst, value):
    try:
        return lst.index(value)
    except ValueError:
        return None

# 队列和已访问状态集合
priority_queue = []
parent = {grid_initial.tobytes():(None,loss)}  # parent记录{子状态：（父状态，初始状态到子状态的实际损耗值）}

# 如果不把numpy数组转化成字节模式放入heapq中会报错，有人知道是为啥吗，莫名其妙的bug。

heapq.heappush(priority_queue, (
    (lambda loss, heuristic: loss + heuristic)(loss, manhattan_distance(grid_initial, grid_final)),
    grid_initial.tobytes()
))

while priority_queue:
    current_state_loss,current_state_bytes = heapq.heappop(priority_queue)
    current_state = np.frombuffer(current_state_bytes, dtype = int).reshape(3, 3)
    if np.array_equal(current_state,grid_final):
        print(f"总共执行次数：{count}")
        path = []
        state = current_state_bytes
        while state is not None:
            path.append(np.frombuffer(state, dtype = int).reshape(3, 3))
            state = parent[state][0]

        # 输出最优解路径
        print("最优解路径：")
        for step in path[::-1]:  # 反向输出路径
            print(step)
            count += 1
        print(f"总共需要挪动{len(path)}次")
        break
    row,col = np.where(current_state==0)[0][0],np.where(current_state==0)[1][0]
    for next_position_0 in possible_position(row,col):
        current_state_copy = current_state.copy()
        current_state_copy[(row,col)],current_state_copy[next_position_0] = current_state_copy[next_position_0],current_state_copy[(row,col)]
        if current_state_copy.tobytes() not in parent:
            parent[current_state_copy.tobytes()] = current_state_bytes,current_state_loss+1
            heapq.heappush(priority_queue,((lambda loss, heuristic: loss + heuristic)(current_state_loss+1, manhattan_distance(current_state_copy, grid_final)),current_state_copy.tobytes()))
        else:
            index = find_index(priority_queue,current_state_copy.tobytes())
            if index is not None:
                current_state_copy_envaluation_value = parent[current_state_copy.tobytes()][1]
                current_state_loss +=1
                if manhattan_distance(current_state_copy, grid_final) + current_state_loss < current_state_copy_envaluation_value:
                    heapq.heappush(priority_queue,(current_state_copy.tobytes(),manhattan_distance(current_state_copy, grid_final) + current_state_loss))
    count+=1

```

