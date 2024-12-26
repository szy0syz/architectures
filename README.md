# archs

## 复盘

### nyt-v1

- sqlc快且极致，但无法构建复杂的domain model (wrap base model)，就算用Embedding structs但灵活度也不够，面对复杂业务对心智负担
- 以后还是换go-jet，同样是编译时，虽然没有sqlc的极致，但留给程序员的空间够大，可以分解复杂业务
