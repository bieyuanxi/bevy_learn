```rust
bevy_ecs::entity
// size = 8, align = 0x8
pub struct Entity {
    index: u32,
    generation: NonZero<u32>,   //该Entities Vec元素当时的代数,用于确定是否合法
}

The identifier is implemented using a generational index: a combination of an index and a generation. This allows fast insertion after data removal in an array while minimizing loss of spatial locality.
```
[关于generational-indices](https://lucassardois.medium.com/generational-indices-guide-8e3c5f7fd594)



```rust
bevy_ecs::entity
// size = 64 (0x40), align = 0x8
pub struct Entities {
    meta: Vec<EntityMeta>,  //Entity元数据数组
    pending: Vec<u32>,      //存储meta索引的数组,储存空闲和预留的meta数组index,即复用之前被free的空间
    free_cursor: AtomicI64, //将pending数组分割,左边表示free,右边表示reserved
    len: u32,   //  实际在用的EntityMeta个数
}
```
当分配Entity时,优先从pending中分配(预留),pending记录了meta内元素的index,其内容是当前尚未真正使用的元素index.
当释放Entity时,将指定的Entity的index放入pending,预留复用.