---
uuid: 4b225f30-0f68-11e9-87e6-131530682559
author: dujun
email: emolingzhu@126.com
github: https://github.com/jiandandkl
avatar: https://avatars1.githubusercontent.com/u/16009933?v=3
title: sequelize-typescript联表查询
date: 2019-01-03 23:00:14
tags:
---

> 使用sequelize-typescript的联表查询遇到点问题,文档不太清晰,对着那几个api: ```ForeignKey```,```BelongsTo```,```HasOne```一个个调用尝试,想着以后有空写个清晰简单的文章,帮助其他同学节约点时间。(拖了两个月,组内有其他同学也遇到这个问题,立了flag赶紧补上)

### API

##### @ForeignKey
这个还是比较明显的,需要外联的key,用在源模型中。
参数是目标模型。

```javascript
@ForeignKey(() => Team)
```

##### @BelongsTo
创建源模型到目标模型的关系。
可以传1到2个参数,第一个参数是目标模型,第二个参数是源模型的外联key,第二个参数也可以传一个对象,如:

```javascript
// 只传目标模型,需要指定源模型的@ForeignKey
@BelongsTo(() => Team)

// 传入teamName作为源模型的外联key,默认目标模型对应的外联key为目标模型的主键
@BelongsTo(() => Team, 'teamName')
```

##### @HasOne / @hasMany / @BelongsToMany

不太好用就不介绍了-_-


>又研究了下发现@ForeignKey可以不使用,改为放在@BelongsTo里,见如下示例

### 示例:

player.ts

```javascript
export default class Player extends Model<Player> {
  @Column
  teamName: string;

  @Column
  gameName: string;

  @Column
  teamId: number;

  // 传入targetKey指定目标模型的外联key
  @BelongsTo(() => Team, { foreignKey: 'teamName', targetKey: 'tname' })
  team: Team;
  // 关联其他模型
  @BelongsTo(() => Game, { foreignKey: 'gameName', targetKey: 'gname' })
  game: Game;
  
  static async findData() {
    return this.findOne({
      include: [{
        model: Team,
        // 获取目标模型某些key
        attributes: ['type'],
     }],
      raw: true,
    })
  }
}
```

team.ts

```javascript
@Table({
  tableName: 'team'
})

export default class Team extends Model<Team> {
  @Column
  tname: string;
}
```

game.ts

```javascript
@Table({
  tableName: 'game'
})

export default class Game extends Model<Game> {
  @Column
  gname: string;
}
```

### 常见问题

``` 
Error: Game is not associated to Player 
``` 
处理: 目标模型没有与源模型相关联,是```@BelongsTo```下缺少```team: Team```

```
Error: Foreign key for "Game" is missing on "Player"
```
处理: 缺少foreignKey,可以在源模型的key上添加```@ForeignKey```,或者在```@BelongsTo```第二个参数里添加foreignKey对象