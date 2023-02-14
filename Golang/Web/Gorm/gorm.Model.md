# 定义

- 是一个结构体

- 包含了ID、CreatedAt、UpdateAt、DeletedAt四个字段：

  ```go
  // gorm.Model 定义
  type Model struct {
    ID        uint `gorm:"primary_key"`
    CreatedAt time.Time
    UpdatedAt time.Time
    DeletedAt *time.Time
  }
  
  ```



# 用途

- 可以嵌入自己的模型中：

  ```go
  // 将 `ID`, `CreatedAt`, `UpdatedAt`, `DeletedAt`字段注入到`User`模型中
  type User struct {
    gorm.Model
    Name string
  }
  
  ```

  也可以完全自定义模型：

  ```go
  // 不使用gorm.Model，自行定义模型
  type User struct {
    ID   int
    Name string
  }
  ```

  