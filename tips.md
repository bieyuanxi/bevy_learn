1. 如何判断一个元素应该使用资源还是组件？
```
取决于这个元素是全局存在还是在实体中独立存在，换句话说，是否可重复，是否应该随实体创建而生，随实体销毁而灭。
```
1. 充分利用元组结构体(newtype): Bevy不允许一个实体内存在重复的组件,因此最好使用newtype将数据包裹起来,否侧可能Bevy可能会拒绝工作.将所用数据包裹起来还有一个优点就是可读性,见下面例子:
```rust
#[derive(Component)]
struct Acceleration(Vec2);  //加速度

#[derive(Component)]
struct Velocity(Vec2);  //速度

#[derive(Component)]
struct Location(Vec2);  //位置,坐标
```

1. 好用的Deref和DerefMut:使用bevy提供的宏Deref和DerefMut直接获取元组结构体内部的数据,免去newtype的烦恼
```rust
// 使用Deref和DerefMut可以直接获取元组结构体内部的数据
#[derive(Component, Deref, DerefMut)]
struct AnimationTimer(Timer);

//example:
let mut timer = AnimationTimer(Timer::from_seconds(1.0, TimerMode::Repeating));
timer.tick(Duration::from_secs_f32(1.5));   // 不需要timer.0.tick(...)
```