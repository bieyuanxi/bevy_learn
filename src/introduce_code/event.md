# Event
`Event` 


## How to use
1. Define your event type and derive `Event`.
    ```rust
    #[derive(Event)]
    struct MyEvent {
        pub message: String,
    }
    ```
1. Add event to app.
    ```rust
    app.add_event::<MyEvent>();
    ```
1. Use `EventReader<MyEvent>` & `EventWriter<MyEvent>`.
    ```rust
    fn event_listener(mut events: EventReader<MyEvent>) {
        for my_event in events.read() {
            info!("{}", my_event.message);
        }
    }
    ```
    ```rust
    fn event_trigger(mut my_events: EventWriter<MyEvent>) {
        my_events.send(MyEvent {
            message: "MyEvent just happened!".to_string(),
        });
    }
    ```


## Use case

### Run systems only `on_event`
```rust
app.add_systems(Update, my_sys.run_if(on_event::<MyEvent>()));
```

## Pitfall!
1. Loss of event if system is paused.
1. Maybe what an event is read is from previous frame. 
1. Event accumulation risk.

## Source Code
This is done by adding a `Resource` of type `Events::<T>`, and inserting an `event_update_system` into `First`.

```rust
#[derive(Debug, Resource)]
pub struct Events<E: Event> {
    /// Holds the oldest still active events.
    /// Note that a.start_event_count + a.len() should always === events_b.start_event_count.
    events_a: EventSequence<E>,
    /// Holds the newer events.
    events_b: EventSequence<E>,
    event_count: usize,
}

#[derive(Debug)]
struct EventSequence<E: Event> {
    events: Vec<EventInstance<E>>,
    start_event_count: usize,
}

#[derive(Debug)]
struct EventInstance<E: Event> {
    pub event_id: EventId<E>,
    pub event: E,
}

pub struct EventId<E: Event> {
    /// Uniquely identifies the event associated with this ID.
    // This value corresponds to the order in which each event was added to the world.
    pub id: usize,
    _marker: PhantomData<E>,
}
```

`Events` use two event sequence: `events_a` and `events_b`:
* `events_a` stores events which are active but sent right after last `update()` is called. 
* While `events_b` stores events which are active and newer than `events_a`'s.

After `update()` is called, events in `events_a` will be dropped and events in `events_b` will be sent to `events_a`(actually using `std::mem::swap`).

`start_event_count` and `event_count` serve as counters as well as index.


```
                 a.events      b.events       
+----------+ +-------------+ +----------+
| dropped  | |    vec_a    | |  vec_b   |
v          v v             v v          v
+---+---+---+---+---+---+---+---+---+---+---+---+
| x | x | x | a | a | a | a | b | b | b |   |   |
+---+---+---+---+---+---+---+---+---+---+---+---+
            ^               ^           ^
            |               |           |
            |               |           +----event_count
a.start_event_count  b.start_event_count

```


# Ref
* [Official `Event` example](https://github.com/bevyengine/bevy/blob/main/examples/ecs/event.rs)
[]()