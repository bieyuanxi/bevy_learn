+++
title = "Bevy的ECS系统"
+++
TODO
# Entity

# Component


# System

# Resource


# State
[参考资料](https://bevy-cheatbook.github.io/programming/states.html)

用于控制System是否运行,

In every state, you can have different systems running. You can also add setup and cleanup systems to run when entering or exiting a state.
## 基础用法
```rust
// 定义项目State
#[derive(States, Debug, Clone, PartialEq, Eq, Hash)]
enum MyAppState {
    LoadingScreen,
    MainMenu,
    InGame,
}

// Specify the initial value:
app.insert_state(MyAppState::LoadingScreen);

// Or use the default (if the type impls Default):
app.init_state::<MyGameModeState>();
app.init_state::<MyPausedState>();  //指定其他State
```

### 不同的State运行不同的System
```rust
// 1. on enter state
app.add_systems(OnEnter(MyAppState::LoadingScreen), (
    start_load_assets,
));
// 2. 1. on exit state
app.add_systems(OnExit(MyAppState::LoadingScreen), (
    despawn_loading_screen,
));
// 3. in state
app.add_systems(Update, (
    system_run.run_if(in_state(MyAppState::LoadingScreen)),
));
```

## State在Plugin中的作用
1. 在统一的地方定义所有的state类型,不同的plugins主动将其systems添加到对应状态.
还可以定义在Plugin定义局部State,不暴露给上层或其他Plugin.
```rust
impl Plugin for MyPlugin {
    fn build(&self, app: &mut App) {
        app.add_systems(Update, 
            my_plugin_system1.run_if(in_state(MyAppState::InGame))) //引入全局定义的State
        .add_systems(Update, 
            my_plugin_system1.run_if(in_state(LocalState::Idle)));  //引入局部定义的State
    }
}

// 外部调用时:
app.init_state::<MyAppState>().add_plugins(MyPlugin);
```
2. You can also make plugins that are configurable, so that it is possible to specify what state they should add their systems to:
```rust
pub struct MyPlugin<S: States> {
    pub state: S,   //S是暴露给外部的接口,指定此Plugin运行时App的状态
}

impl<S: States> Plugin for MyPlugin<S> {
    fn build(&self, app: &mut App) {
        app.add_systems(Update, (
            my_plugin_system1,
            my_plugin_system2,
            // ...
        ).run_if(in_state(self.state.clone())));
    }
}

// 外部调用时:
app.add_plugins(MyPlugin {
    state: MyAppState::InGame,  //在这个状态下MyPlugin的systems才运行
});
```
## 控制State

### 获取当前State
```rust
fn debug_current_gamemode_state(state: Res<State<MyGameModeState>>) {
    eprintln!("Current state: {:?}", state.get());
}
```

### 改变State
使用`NextState<T>`.
注意只有调度系统调用`apply_state_transition()`时才应用新的`State`.
```rust
fn toggle_pause_game(
    state: Res<State<MyPausedState>>,
    mut next_state: ResMut<NextState<MyPausedState>>,
) {
    match state.get() {
        MyPausedState::Paused => next_state.set(MyPausedState::Running),
        MyPausedState::Running => next_state.set(MyPausedState::Paused),
    }
}
```

## 使用场景
* A menu screen or a loading screen
* Pausing / unpausing the game
* Different game modes
