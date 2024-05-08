# State Introduce
`State` allows you to define your app state. You can access current state, set a new state. Or do sth between state transition. 

Only one instance of each different state type is created in bevy. You will know why after reading the rest.


## How to use
### Define your own state
First on first, you need to define your state and derive necessary traits for it, which are `Clone + PartialEq + Eq + Hash + Debug + Default (or FromWorld)`. Usually an `enum` is used.

Don't worry, after you derive `States`, the compiler will give the notes which are missing:).
```rust
#[derive(Debug, Clone, Copy, Default, Eq, PartialEq, Hash, States)]
enum AppState {
    #[default]
    Menu,
    InGame,
}
```
### Declare it with the App
Then you announce your state when config app. Use either `init_state()` or `insert_state()`.
```rust
// init state with a default value.
app.init_state::<AppState>();
// Alternatively use .insert_state(AppState::Menu) with a specific value.
app.insert_state(AppState::Menu);
```
After that, you can use your state now!

### Just use it!
Here gives some examples about how to integrate your state in app.

#### Get/Read current state in `system`
In systems, you can read your state by using `Res<State<YourOwnState>`. But notice that you can only get the value instead of gaving a new value.
```rust
fn menu(state: Res<State<AppState>>) {
    let curr_state = state.get();    // coerced cast, curr_state: &Appstate
}
```

#### Update state in `system`
In systems you can change your state by using the mutable resource `ResMut<NextState<AppState>>`. Then use the `.set()` function.
```rust
fn menu(mut next_state: ResMut<NextState<AppState>>) {
    next_state.set(AppState::InGame);
}
```

#### Run systems OnEnter/OnExit/OnTransition state
This system runs when we enter `AppState::Menu`, during the `StateTransition` schedule. 

```rust
app.add_systems(OnEnter(AppState::Menu), setup_menu)
    .add_systems(OnExit(AppState::Menu), cleanup_menu)
    .add_systems(OnEnter(AppState::InGame), setup_game)
    .add_systems(OnTransition {
        from: AppState::Menu,
        to: AppState::Ingame
    }, log_system)
```

All systems from the exit schedule of the state we're leaving are run first, and then all systems from the enter schedule of the state we're entering are run second.

*`OnEnter` is called first when app starts up.


#### Run systems only in a specific state
Run the system only if in that state. See bevy `common_conditions` for more info. Examples are `state_changed`, `in_state`, `state_exists`.
```rust
// Check the value of the `State<T>` resource to see if they should run.
app.add_systems(Update, menu.run_if(in_state(AppState::Menu)))
```

#### Use `StateTransitionEvent` to read state change
You can read the events when state change happens.
```rust
/// print when an `AppState` transition happens
fn log_transitions(mut transitions: EventReader<StateTransitionEvent<AppState>>) {
    for transition in transitions.read() {
        info!(
            "transition: {:?} => {:?}",
            transition.before, transition.after
        );
        // Filter what you don't care about.
    }
}
```

Notice that any state change will send a event, you should filter those you don't care about.

`StateTransitionEvent` is sent when a schedule called `StateTransition` runs, which runs right after `PreUpdate`. So `PreUpdate` can only get the event in next frame.


### What's the difference between `OnTransition` and `StateTransitionEvent`?
`OnTransition` is a schedule label with exactly the `from` and `to`. All systems in that schedule run only if both `from` and `to` match.
```rust
#[derive(ScheduleLabel, Clone, Debug, PartialEq, Eq, Hash)]
pub struct OnTransition<S: States> {
    /// The state being exited.
    pub from: S,
    /// The state being entered.
    pub to: S,
}
```

`StateTransitionEvent` is sent when any state change occurs. You should filter those you don't care about.

 Also notice that this event will not work unless state change really happens, that is, `before` != `after`, according to `apply_state_transition`.

```rust
#[derive(Debug, Copy, Clone, PartialEq, Eq, Event)]
pub struct StateTransitionEvent<S: States> {
    /// the state we were in before
    pub before: S,
    /// the state we're in now
    pub after: S,
}
```

In general, use `OnTransition` if you want systems to run when specific state changes. Use `StateTransitionEvent` if you have interests in what value the state is and was.

## Use case



## Source code

> Adds `State<S>` and `NextState<S>` resources, `OnEnter` and `OnExit` schedules for each state variant (if they don't already exist), an instance of `apply_state_transition::<S>` in
`StateTransition` so that transitions happen before `Update` and a instance of `run_enter_schedule::<S>` in `StateTransition` with a `run_once` condition to run the on enter schedule of the initial state.

State in bevy is implemented using `Resource`s: `State<S>` and `NextState<S>`.

Event `StateTransitionEvent<S>`

Macro `States`

Trait `States`.

### State Initialize
`Resource`s: `State<S>` and `NextState<S>` are initialized.

Event `StateTransitionEvent<S>` is added.

`run_enter_schedule::<S>` and `apply_state_transition::<S>` runs in `StateTransition`. `run_enter_schedule::<S>` runs `OnEnter<S>` schedule exactly once. 


### State Transition
State transition schedule runs in the following order:`First`->`PreUpdate`->`StateTransition`->`RunFixedMainLoop`->`Update`->`SpawnScene`->`PostUpdate`->`Last`

`apply_state_transition::<S>` runs `OnExit`->`OnTransition`->`OnEnter` in order.


Because `ScheduleLabel`s should derive `Hash`, any schedule labels with diverse hash values are **totally different labels**. For example, the following two `OnTransition` labels below have nothing in common! That means no time pays to determine which system should run!
```rust
OnTransition {
    from: AppState::Menu,
    to: AppState::Ingame,
}

OnTransition {
    from: AppState::Ingame,
    to: AppState::Menu,
}
```


### Deref & DerefMut in Res
Allow directly access value in `Res` 
#### get() in Res
```rust
change_detection_impl!(ResMut<'w, T>, T, Resource);
```

#### set() in Res
```rust
change_detection_mut_impl!(ResMut<'w, T>, T, Resource);
```

## Ref
* [Official `State` example](https://github.com/bevyengine/bevy/blob/main/examples/ecs/state.rs)
* [Bevy Cheat Book](https://bevy-cheatbook.github.io/programming/states.html#states)