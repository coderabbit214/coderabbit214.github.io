# Golang 分布式事务


# Golang 分布式事务

> 使用两阶段提交（2PC）协议实现分布式事务

## 1. 数据库设置

创建两个数据库

```sql
CREATE DATABASE test1;
CREATE DATABASE test2;
create table user
(
    id         bigint auto_increment primary key,
    username   varchar(255) not null,
    password   varchar(255) not null,
    created_at timestamp null,
    updated_at timestamp null
);
```

## 2. 实现2PC协议

- coordinator

  ```go
  type Coordinator struct{}
  
  // ExecuteDistributedTransaction runs the distributed transaction using 2PC
  func (c *Coordinator) ExecuteDistributedTransaction(ctx context.Context, prepareFun func(db *gorm.DB) error, participants []*Participant) error {
  	// Phase 1: Prepare
  	var preparedTransactions []*gorm.DB
  	for _, p := range participants {
  		tx := p.db.Begin()
  
  		err := p.Prepare(ctx, tx, prepareFun)
  		if err != nil {
  			return err
  		}
  
  		preparedTransactions = append(preparedTransactions, tx)
  	}
  
  	// Phase 2: Commit
  	for i, p := range participants {
  		err := p.Commit(ctx, preparedTransactions[i])
  		if err != nil {
  			// Rollback transactions if commit fails
  			for _, tx := range preparedTransactions {
  				_ = p.Rollback(ctx, tx)
  			}
  			return err
  		}
  	}
  
  	return nil
  }
  
  ```

-  participant

   ```go
   type Participant struct {
   	db *gorm.DB
   }
   
   // Prepare prepares the transaction in the participant
   func (p *Participant) Prepare(ctx context.Context, tx *gorm.DB, prepareFun func(db *gorm.DB) error) error {
   	return prepareFun(tx)
   }
   
   // Commit commits the transaction in the participant
   func (p *Participant) Commit(ctx context.Context, tx *gorm.DB) error {
   	err := tx.Commit().Error
   	if err != nil {
   		return err
   	}
   	return nil
   }
   
   // Rollback rolls back the transaction in the participant
   func (p *Participant) Rollback(ctx context.Context, tx *gorm.DB) error {
   	err := tx.Rollback().Error
   	if err != nil {
   		return err
   	}
   	return nil
   }
   
   ```

## 3. 测试

```go
func TestCoordinator_ExecuteDistributedTransaction(t *testing.T) {
	db1, err := gorm.Open(mysql.Open(fmt.Sprintf(
		"%s:%s@tcp(%s:%v)/%s?charset=utf8mb4&parseTime=True&loc=Local",
		"root",
		"123456",
		"127.0.0.1",
		"3306",
		"test1",
	)))

	db2, err := gorm.Open(mysql.Open(fmt.Sprintf(
		"%s:%s@tcp(%s:%v)/%s?charset=utf8mb4&parseTime=True&loc=Local",
		"root",
		"123456",
		"127.0.0.1",
		"3306",
		"test2",
	)))

	c := &Coordinator{}

	p1 := &Participant{
		db: db1,
	}
	p2 := &Participant{
		db: db2,
	}

	err = c.ExecuteDistributedTransaction(context.Background(), func(db *gorm.DB) error {
		return db.Model(&model.User{}).Create(&model.User{
			Username: "test",
			Password: "123456",
		}).Error
	}, []*Participant{p1, p2})
	if err != nil {
		t.Error(err)
	}
	fmt.Println("success")
}

```


