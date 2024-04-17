
+++
title = "Timer in Bevy"
+++
# 概述
Bevy的计时器`Timer`用于倒计时,需要使用者手动调用`tick()`函数更新其状态.
```rust
/// Tracks elapsed time. Enters the finished state once `duration` is reached.
///
/// Non repeating timers will stop tracking and stay in the finished state until reset.
/// Repeating timers will only be in the finished state on each tick `duration` is reached or
/// exceeded, and can still be reset at any given point.
///
/// Paused timers will not have elapsed time increased.
pub struct Timer { /**private fields */ }
```
计时器`Timer`提供了两种模式,单次和重复.
```rust
pub enum TimerMode {
    /// Run once and stop.
    #[default]
    Once,
    /// Reset when finished.
    Repeating,
}
```

可以在创建时设置模式.
```rust
pub fn new(duration: Duration, mode: TimerMode) -> Self;

pub fn from_seconds(duration: f32, mode: TimerMode) -> Self
```

# 使用场景
## 定时切换sprite动画帧(2d)
```rust
#[derive(Component, Deref, DerefMut)]   //派生了Deref,DerefMut
struct AnimationTimer(Timer);           //newtype包装

fn animate_sprite(
    time: Res<Time>,
    mut query: Query<&mut AnimationTimer>,
) {
    for mut timer in &mut query {
        timer.tick(time.delta());
        if timer.just_finished() {
            // change to next texture
        }
    }
}
```

# 函数解析
```rust
// 最重要的函数,传入时间delta,让计时器更新.
// 使用时,首先调用tick(),告诉timer自上次tick已经经历的时间delta,
// 之后再调用其他函数,如finished().
pub fn tick(&mut self, delta: Duration) -> &Self;

// 检查计时器是否完成,
pub fn finished(&self) -> bool;

// 对单次计时器:finished后第二次tick,just_finished会返回false,即"是否是这次tick完成的";
// 对重复计时器:与finished()作用相同
pub fn just_finished(&self) -> bool;

//TODO
```

# 总结
1. Bevy的`Timer`是"懒"的,需要在使用它时手动`tick()`以更新计时器,之后才能获得计时器的结果.一般搭配`Res<Time>`使用.
1. `Timer`每次`tick()`之后,多余的时间会累积,避免了以帧计时的问题(如非整数帧的判定取整情况).