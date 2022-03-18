# WinterMemories
## 描述
- 2021面计算机设计大赛作品
- 最早接触UE的作品，主要由蓝图实现
## 思路
- 题目有关于冬奥会，如果只涉及竞技部分会落入俗套
- 我们应该将运动员的成长经历融入到游戏中
- 由策划设计故事线，程序实现对应的游戏
- 最终站在奥运舞台比赛，赢得冠军
## 遇到的问题
- 如何推进游戏？
  - 方案一：失败的话进度不往前推，需要反复重复（应用在训练关卡，让玩家体会训练的艰辛）
  - 方案二：脱离玩家控制，程序写好玩家胜利（应用在奥运赛场，让玩家体会功夫不负有心人的快乐）
- 版本迭代：2D转3D
  - 最初的设计为2D模式，但一直达不到想要的效果，而且大量的sprites2D让美工方面难以完成
  - 在中期权衡之后，搜集了一些3D模型，果断推翻原来的工程，转换为3D重新开发
## 设计
- 用到的设计模式
  - 在一个训练关卡需要随机发射炮弹，如果每次spawn destroy会很浪费性能，可以使用**享元模式**创建一个雪球池，存储5个就嘎嘎够用，每次射出没碰到玩家就重回雪球池，节省了一些开销，空间换时间
  - 在过场动画使用了**外观模式**，将特效，镜头序列轨道，音乐包装在一个类里统一播放停止
  - 在角色里有一个用于和屋内物体交互的组件，而不是直接在角色上写逻辑，运用**组件模式**解耦合，且不会影响character在之后关卡中的使用
  - 不同关卡的AI来自同一个父类加上**装饰器模式**，一个可以随机走路，一个可以循迹走路，效果和直接继承差不多
  - 整个游戏都是**外观模式**，玩家只需要做简单操作，不需要了解背后的系统。
  - 使用gameinstance，saveGame来记录玩家玩到了哪一关，是**备忘录模式**
  - event都是**观察者模式**，等待事件发生，给一个通知，执行后面的操作
- 复杂度的考虑
  - 模型有些来自网络，精度太高
    - 更换低模
    - 曲面简化
- 优化的地方
  - 运动员的固定路线移动 由 设置在线上 变成追随 线上的物体（避免了AI不丝滑的运动）
  - 躲避球关卡character的位移长度 ， 一开始是 点击一下 往一个方向走固定长度 后来发现 会走出 躲避球的范围 ， 然后加了 一个 bool  绝对值 <= 2才能操作（越界检查）
## 重难点
- 在level1，玩家需要躲避随机移动的AI，完成滑冰练习。但如果AI仅仅随机运动，玩家可能不需要任何操作就能通过本关卡（地图较大，随机移动可能永远碰不到玩家），需要增大碰撞玩家的概率。
  - 解决方案1：增加AI数量，减小滑冰场面积——新的问题：性能消耗变大，进入关卡卡顿。
  - 解决方法2：Ai每次运动会获取到玩家的location，然后根据location和一个范围，获取其中的随机点，moveto这个随机点。
    - 为了在一个半径为 RR 的圆 CC 中均匀随机生成点，我们可以使用拒绝采样的方法。我们使用一个边长为 2R2R 的正方形覆盖住圆 CC，并在正方形内随机生成点，若该点落在圆内，我们就返回这个点，否则我们拒绝这个点，重新生成知道新的随机点落在圆内。
- 在level2和level4中为了防止玩家不按照正确方向移动，破坏游戏规则，增加了一个vector，玩家需要按照顺序穿过这些碰撞体，保证行动方向的正确
- 物体交互时在场景的物体太小，无法看到细节，以及相关的情节。
  - 解决方法：将控制权转移到物体上，关闭物体的碰撞，将他移动到摄像机的前方，通过上下左右控制物体的旋转查看细节，显示文字和按键开启游戏关卡
