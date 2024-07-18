# SDK使用指南
本文档主要介绍SDK通常的结构，以及选手应如何使用主办方提供的SDK

## 标题没想好
回顾人工智能领域中智能体的定义：能够与环境交互并作出决策。如果能够将环境和交互抽象成接口，开发者就仅需要考虑决策过程的实现，这可以大大减轻开发者的负担。智能体大赛主办方提供的SDK就实现了基础的环境和交互过程的抽象，同时可能提供一些可视化和调试工具。

## SDK常见结构
![](imgs/sdk-structure.svg)
### 游戏状态类
游戏状态类是环境的核心类。游戏状态类包含了一个完整的环境。例如，在围棋游戏中，游戏状态类可以使用多个二维数组表示。为了方便维护，游戏状态类可能还会维护重复的信息。游戏状态类通常对选手暴露查询接口，例如选手可以使用查询函数查询棋盘上的某个点的落子情况、落子的步数、剩余的气等信息。

### 状态维护函数
状态维护函数负责在智能体进行操作后维护游戏状态。这保证了游戏状态的实时性和正确性。例如，在围棋游戏中，智能体落子后。状态维护函数将修改二维数组对应位置处的值，保证落子情况、落子步数等信息的变化被更新到环境中。

### 完备的交互环境
状态维护函数和游戏状态类构成了一个完备的交互环境。智能体在作出决策后，可以通过调用状态维护函数使用决策对环境施加影响，并通过游戏状态类的查询接口对环境进行观测，进而做出下一步决策。

### 操作类
操作类是决策的再次封装。智能体所作出的一个决策可能需要大量信息进行描述。操作类提供了决策的封装，选手的决策逻辑可以通过操作类的实例描述作出的操作。例如，一个数组`[a, b]`就可以作为围棋游戏中的一个操作类，它封装了在棋盘的a行b列落子的操作。在更复杂的游戏中，所需要的参数可能更多，此时的封装会显得更加必要，这使得选手可以直接使用操作类与环境进行交互，在参数传递的时候更为方便。

### 游戏控制器
由于在线评测的特性，智能体需要在合适的时间告知评测机当前的决策。为了避免选手进行交互方面的开发，我们引入了游戏控制器。游戏控制器的引入使得选手的智能体可以直接被实现为一个函数，游戏控制器在适当的时间调用该函数，并等待函数返回一个操作类。因此选手无需关心通讯的具体实现和时机，在游戏控制器通知智能体进行决策时直接返回决策即可。通常，游戏局面是游戏控制器类的一个属性，负责维护当前局面，以便于和选手可能在模拟阶段使用的其他游戏局面类做区分。

### 格式化与通信模块
这个模块负责将智能体返回的操作类转化为符合通信协议的字节流，以及将评测机的字节流转化为操作类或更新信息进行局面更新或直接告知智能体。

## 一个典型的使用sdk的智能体实现

- ai.py
```python
from sdk.gamestate import GameState
from sdk.operations import *

def ai(state: Gamestate):
    my_seat = state.seat
    '''
    进行决策
    策略为在从左到右，从上到下的第一个空位落子
    '''
    for a in range(19):
        for b in range(19):
            if state.check_chess_type(row=a, col=b) == -1:
                return Step(row=a, col=b)
```

- main.py
```python
from sdk.controller import GameController
from ai import ai

if __name__ == "__main__":
    controller = GameController()
    controller.init()
    controller.start_game(ai)
```