# @GeneratedValue

可选

strategy:表示主键生成策略，有AUTO、INDENTITY、SEQUENCE 和 TABLE 4种，分别表示让ORM框架自动选择,

根据数据库的Identity字段生成，根据数据库表的Sequence字段生成,以有根据一个额外的表生成主键，默认为AUTO

\(1\)identity主键自增，这种方式依赖于具体的数据库，如果数据库不支持自增主键，那么这个类型是没法用的

```
@Table(name = "defacement")
@Entity
@Setter
@Getter
@AllArgsConstructor
@NoArgsConstructor
public class Defacement {

    /**  */
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "ID")
    private Integer id;
}
```

> 说明：
>
> @GeneratedValue\(strategy = GenerationType.IDENTITY\)，如果使用的是MYSQL，在表中设置自增属性，插入数据时，将会自动插入

\(2\)借助一个表来实现主键自增, 通过一个表来实现主键id的自增，这种方式不依赖于具体的数据库，可以解决数据迁移的问题

```
public class Users implements Serializable {

@Id
@GeneratedValue(strategy=GenerationType.TABLE)
@Column(name = "user_code", nullable = false)
private String userCode;
}
```

\(3\)sequence主键自增,通过Sequence来实现表主键自增，这种方式依赖于数据库是否有SEQUENCE，如果没有就不能用

```
public class Users implements Serializable {

@Id
@GeneratedValue(strategy=GenerationType.SEQUENCE)
@SequenceGenerator(name="seq_user")
@Column(name = "user_id", nullable = false)
private int userId;
}
```



