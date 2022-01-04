
## 一般更新

```java
//update
UpdateDefinition update = Update.update("isLike",isLike)  
 								.set("updated", System.currentTimeMillis());  
mongoTemplate.updateFirst(query,update,UserLike.class);
```


## MongoDB更新并获取更新后的数据
##### 自增更新
```java
@Override
    public Integer savePublishComment(CommentDto commentDto) {
        Movement movement = mongoTemplate.findById(commentDto.getMovementId(), Movement.class);
        if (movement == null) {
            throw new RuntimeException();
        }
        Comment comment = Comment.builder()
                .publishId(commentDto.getPublishId())   
                .publishUserId(movement.getUserId())    
                .commentType(commentDto.getCommentType())   
                .content(commentDto.getComment())   
                .created(System.currentTimeMillis())    
                .build();
        mongoTemplate.save(comment);

        //1.定义查询规范
        Query query = Query.query(Criteria.where("id").is(commentDto.getPublishId()));
        Update update = new Update();
        //2.定义更新字段以及更新方法
        if(comment.getCommentType() == CommentType.LIKE.getType()) {
            update.inc("likeCount",1);
        }else if (comment.getCommentType() == CommentType.COMMENT.getType()){
            update.inc("commentCount",1);
        }else {
            update.inc("loveCount",1);
        }
        //3.设置更新选项
        FindAndModifyOptions options = new FindAndModifyOptions();
        options.returnNew(true);//获取更新后的最新数据
        Movement modifyMovement = mongoTemplate.findAndModify(query, update, options, Movement.class);

        //获取最新评论数量/用于日志打印
        return modifyMovement.statisCount(comment.getCommentType());
    }
```




