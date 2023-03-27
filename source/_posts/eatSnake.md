---
title: typeScript贪吃蛇小案例
date: 2022-12-2 10:52
categories: typeScript
---



## 环境搭建

```json
"devDependencies": {
    "@babel/core": "^7.20.5",
    "@babel/preset-env": "^7.20.2",
    "babel-loader": "^9.1.0",
    "clean-webpack-plugin": "^4.0.0",
    "core-js": "^3.26.1",
    "css-loader": "^6.7.2",
    "html-webpack-plugin": "^5.5.0",
    "less": "^4.1.3",
    "less-loader": "^11.1.0",
    "postcss": "^8.4.19",
    "postcss-loader": "^7.0.2",
    "postcss-preset-env": "^7.8.3",
    "style-loader": "^3.3.1",
    "ts-loader": "^9.4.1",
    "typescript": "^4.9.3",
    "webpack": "^5.75.0",
    "webpack-cli": "^5.0.0",
    "webpack-dev-server": "^4.11.1"
  }
```

小案例需要用到的插件。



## 主要代码以及思路

### HTML部分代码

```html
<!-- 创建容器的主容器 -->
    <div id="main">
        <!-- 设置游戏的舞台 -->
        <div id="stage">
            <!-- 设置蛇 -->
            <div id="snake">
                <!-- snake内部的div 表示蛇的各部分 -->
                <div></div>
            </div>

            <!-- 设置食物 -->
            <div id="food">
                <!-- 添加四个小div 来设置事物的样式 -->
                <div></div>
                <div></div>
                <div></div>
                <div></div>
            </div>

        </div>
        <!-- 设置游戏的积分牌 -->
        <div id="score-panel">
            <div>
                SCORE:<span id="score">0</span>
            </div>
            <div>
                <button id="btn" onclick="location.reload()">再来一局</button>
            </div>
            <div>
                LEVEL:<span id="level">1</span>
            </div>
        </div>
    </div>
```

### 样式

本案例用的样式为 **less**

```less
// 设置变量
@bg-color: #b7d4a8;

// 清楚默认样式
*{
    margin: 0;
    padding: 0;
    // 改变盒子模型的计算方式
    box-sizing: border-box;
}

body{
    font: bold 20px "Courier";
}

// 设置主窗口的样式
#main{
    width: 360px;
    height: 420px;
    background-color: @bg-color;
    margin: 100px auto;
    border: 10px solid black;
    border-radius: 40px;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: space-around;
    
    // 游戏舞台
    #stage{
        width: 304px;
        height: 304px;
        border: 2px solid black;
        // 开启相对定位
        position: relative;

        // 设置蛇的样式
            #snake {
                &>div {
                    width: 10px;
                    height: 10px;
                    background-color: #000;
                    border: 1px solid @bg-color;
                    // 开启绝对定位
                    position: absolute;
                }
            }
            // 设置食物
            #food{
                width: 10px;
                height: 10px;
                // 开启绝对定位
                position: absolute;
                left: 40px;
                top: 100px;
                // background-color: red;
                display: flex;
                flex-flow: row wrap;
                justify-content: space-between;
                align-content: space-between;
                &>div{
                    width: 4px;
                    height: 4px;
                    background-color: #000;
                    transform: rotate(45deg);
                }
            }
    }

    // 记分牌
    #score-panel{
        width: 300px;
        display: flex;
        justify-content: space-between;
        #btn{
            height: 30px;
            width: 60px;
            background-color: #b7d4a8;
            border-radius: 5%;
            border: none;
            cursor: pointer;
        }
    }
}
```



### 食物类

```typescript
// 创建food类
class Food {
    element: HTMLElement;
    constructor() {
        // 获取页面中的food元素并且将其赋值给element
        this.element = document.getElementById('food')!;
    }

    // 定义一个获取食物X轴坐标的方法
    get X() {
        return this.element.offsetLeft;
    }

    // 定义一个获取食物Y轴坐标的方法
    get Y() {
        return this.element.offsetTop;
    }

    // 修改食物的位置
    change() {
        // 生成一个随机的位置
        // 食物的位置最小是0 最大是290
        // 蛇移动一次就是一格, 一格的要求就是10，所以就要求事物的坐标是整十

        let top = Math.round(Math.random() * 29) * 10;
        let left = Math.round(Math.random() * 29) * 10;
        this.element.style.left = left + 'px';
        this.element.style.top = top + 'px'
    }
}
export default Food;
```

### 积分板类

```typescript
// 定义表示记分牌的类
class ScorePanel {
    // score和level用来记录分数和等级
    score = 0;
    level = 1;

    // 分数和等级所在的元素，在构造函数中进行初始化
    scoreEle: HTMLElement;
    levelEle: HTMLElement;

    // 设置一个变量限制等级
    maxLevel: number;
    // 设置一个变量表示多少分时升级
    upScore: number;

    constructor(maxLevel: number = 10, upScore: number = 10) {
        this.scoreEle = document.getElementById('score')!;
        this.levelEle = document.getElementById('level')!;
        this.maxLevel = maxLevel;
        this.upScore = upScore;
    }

    // 设置一个加分的方法
    addScore() {
        // 使分数自增
        this.scoreEle.innerHTML = ++this.score + '';
        // 判断分数是多少
        if (this.score % this.upScore === 0) {
            this.levelUp();
        }
    }

    // 提升等级的方法
    levelUp() {
        if (this.level < this.maxLevel) {
            this.levelEle.innerHTML = ++this.level + '';
        }
    }
}
export default ScorePanel;
```

### 蛇身体类

```typescript
class Snake{
    // 表示蛇的元素
    head: HTMLElement;
    // 蛇的身体(包括蛇头)
    bodies: HTMLCollection;
    // 获取蛇的容器
    element: HTMLElement;

    constructor() {
        this.element = document.getElementById('snake')!;
        this.head = document.querySelector('#snake > div')!;
        this.bodies = this.element.getElementsByTagName('div');
    }

    // 获取蛇的坐标(蛇头坐标)

    // 获取蛇的X轴坐标
    get X() {
        return this.head.offsetLeft;
    }

    // 获取蛇的Y轴坐标
    get Y() {
        return this.head.offsetTop;
    }

    // 设置蛇头的坐标
    set X(value: number) {

        // 如果新值和旧值相同，则直接返回不再修改
        if(this.X === value) {
            return;
        }

        // x的值的合法范围0-290之间
        if(value < 0 || value > 290){
            // 进入判断说明蛇撞墙了
            throw new Error('蛇撞墙了');
        }

        // 修改x时，是在修改水平坐标，蛇在左右移动时，不能向右掉头，反之亦然
        if(this.bodies[1] && (this.bodies[1] as HTMLElement).offsetLeft === value){
            // console.log('水平方向发生了掉头');
            if ((this.bodies[1] as HTMLElement).offsetLeft === 280 || (this.bodies[1] as HTMLElement).offsetLeft===10){
                throw new Error('撞墙啦！')
            }      
            // 如果发生了掉头，让蛇反方向继续移动
            if(value > this.X){
                // 如果新值value大于旧值X，则说明蛇再向右走，此时发生掉头，应该使蛇继续向左走
                value = this.X - 10;
            }else{
                value = this.X + 10;
            }
            
        }
        // 移动身体
        this.moveBody();
        this.head.style.left = value + 'px';
        // 检查有没有撞到自己
        this.checkHeadBody()
    }
    set Y(value: number) {

        // 如果新值和旧值相同，则直接返回不再修改
        if (this.Y === value) {
            return;
        }
        // Y的值的合法范围0-290之间
        if (value < 0 || value > 290) {
            // 进入判断说明蛇撞墙了，抛出一个异常
            throw new Error('蛇撞墙了');
        }

        // 修改y时，是在修改垂直坐标，蛇在向上移动时，不能向下掉头，反之亦然
        if (this.bodies[1] && (this.bodies[1] as HTMLElement).offsetTop === value) {
            if ((this.bodies[1] as HTMLElement).offsetTop === 280 || (this.bodies[1] as HTMLElement).offsetTop === 10) {
                throw new Error('撞墙啦！')
            }  
            // 如果发生了掉头，让蛇反方向继续移动
            if (value > this.Y) {
                value = this.Y - 10;
            } else {
                value = this.Y + 10;
            }
        }

        this.moveBody();
        this.head.style.top = value + 'px';
        // 检查有没有撞到自己
        this.checkHeadBody()
    }

    // 蛇增加身体的方法
    addBody() {
        // 向element中添加一个div
        this.element.insertAdjacentHTML("beforeend", "<div></div>")
    }

    // 添加一个蛇身体移动的方法
    moveBody() {
        /**
         *   将后边的身体设置为前边身体的位置
         *   举例子：
         *         第4节 = 第3节的位置
         *         第3节 = 第2节的位置
         *         第2节 = 蛇头的位置
         */
        // 遍历获取所有的身体
        for(let i = this.bodies.length - 1; i > 0; i--){
            // 获取前边身体的位置
            let X = (this.bodies[i-1] as HTMLElement).offsetLeft;
            let Y = (this.bodies[i-1] as HTMLElement).offsetTop;
            
            // 将值设置到当前身体上
            
            (this.bodies[i] as HTMLElement).style.left = X + 'px';
            (this.bodies[i] as HTMLElement).style.top = Y + 'px';
        }
    }
    // 检查蛇头是否撞到身体上
    checkHeadBody() {
        // 获取所有的身体，检查其是否和蛇头的坐标发生重叠
        for(let i = 1; i < this.bodies.length; i++){
            let bd = this.bodies[i] as HTMLElement
            if(this.X === bd.offsetLeft && this.Y === bd.offsetTop){
                // 进入判断说明蛇头撞到了身体，游戏结束
                throw new Error('撞到自己了！');
            }
        }
    }
}

export default Snake;
```

### 游戏控制类

```typescript
// 引入其他的类
import Snake from "./Snake";
import Food from "./Food";
import ScorePanel from "./ScorePanel";

// 游戏控制器， 控制其他所有的类
class GameControl{
    // 定义三个属性
    // 蛇
    snake: Snake;
    // 食物
    food: Food;
    // 记分牌
    scorePanel: ScorePanel;

    // 创建一个属性来存储蛇的移动方向（也就是按键的方向）
    direction: string = '';

    // 创建一个属性用来记录游戏是否结束
    isLive = true;

    constructor() {
        this.snake = new Snake();
        this.food = new Food();
        this.scorePanel = new ScorePanel(10,2);
        this.init();
    }

    // 游戏初始化方法，调用后游戏即开始
    init(){
        // 绑定键盘按键按下的事件
        document.addEventListener('keydown', this.keydownHandler.bind(this));
        // 调用run方法 使蛇移动
        this.run();
    }

    /**
     * ArrowUp
       ArrowDown
       ArrowLeft
       ArrowRight
     * 
     */
    // 创建一个键盘按下的响应函数
    keydownHandler(event: KeyboardEvent) {
        // 需要检查event.key的值是否合法（用户是否按了正确的按键）
        // 修改direction属性
        this.direction = event.key;
    }

    // 创建一个控制蛇移动的方法
    run() {
        /**
         * 根据方向(this.direction)来使蛇的位置改变
         * 向上 top 减少
         * 向下 top 增加
         * 向左 left 减少
         * 向右 left 增加
         */
        // 获取蛇现在的坐标
        let X = this.snake.X;
        let Y = this.snake.Y;

        // 根据按键方向来修改X值和Y值
        switch(this.direction) {
            case "ArrowUp":
            case "Up":
                // 向上移动 top 减少
                Y -= 10;
                break;
            case "ArrowDown":
            case "Down":
                // 向下移动 top 增加
                Y += 10;
                break;
            case "ArrowLeft":
            case "Left":
                // 向左移动 left 减少
                X -= 10;
                break;
            case "ArrowRight":
            case "Right":
                // 向右移动 left 增加
                X += 10;
                break;
        }

        // 检查蛇是否吃到了食物
        this.checkEat(X,Y)

        // 修改蛇的X值和Y值
        try {
            this.snake.X = X;
            this.snake.Y = Y;
        } catch (e) {
            // 进入到catch，说明出现了异常，游戏结束，弹出一个提示信息
            alert('GAME OVER!');
            this.isLive = false;
        }

        // 开启一个定时调用
        this.isLive && setTimeout(this.run.bind(this), 300 - (this.scorePanel.level-1) * 30)
    }

    // 定义一个方法，用来检查蛇是否吃到食物
    checkEat(X: number,Y: number) {
        if (X === this.food.X && Y === this.food.Y){
            console.log('吃到食物了');
            // 食物的位置要进行重置
            this.food.change();
            // 分数增加
            this.scorePanel.addScore();
            // 蛇要增加一节
            this.snake.addBody();
        }
    }
}

export default GameControl;
```

### 引入

最后在主入口引入，项目即可运行

```typescript
// 引入样式
import './style/index.less';

// 引入控制类

import GameControl from './modules/GameControl';

new GameControl();
```

