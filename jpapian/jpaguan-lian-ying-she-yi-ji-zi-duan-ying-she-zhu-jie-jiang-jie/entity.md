# @Entity、@Table、@id

**@Entity\(name=”EntityName”\)**

不区分大小写，name为可选，对应实体类的名称

**@Table\(name=””,catalog=””,schema=””\)**

可选,通常和@Entity配合使用，只能标注在实体的class定义处,表示实体对应的数据库表的信息

name:可选,表示表的名称。默认地，表名和实体名称一致,只有在不一致的情况下才需要指定表名

catalog:可选,表示Catalog名称，默认为Catalog\(" "\)。

schema:可选,表示Schema名称，默认为Schema\(" "\)。

**@id** 

必须，@id定义了映射到数据库表的主键的属性,一个实体只能有一个属性被映射为主键.置于getXxxx\(\)前或者属性前.

```
@Table(name = "work_manage")
@Entity
public class WorkManage {

    /**
     * 作品编号/备案编号
     */
    @Id
    @GeneratedValue(generator = "assignedGenerator")
    @GenericGenerator(name = "assignedGenerator", strategy = "assigned")
    @Column(name = "WORKID")
    private String workid;
    /**
     * 作品名称
     */
    @Column(name = "WORKNAME")
    private String workname;
    /**
     * 作品名称(英文/拼音)
     */
    @Column(name = "WORKNAME_EN")
    private String worknameEn;

}
```



