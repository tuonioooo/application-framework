# @Entity、@Table、@id

@Entity\(name=”EntityName”\)不区分大小写，name为可选，对应实体类的名称



@Table\(name=””,catalog=””,schema=””\)

可选,通常和@Entity配合使用，只能标注在实体的class定义处,表示实体对应的数据库表的信息

name:可选,表示表的名称。默认地，表名和实体名称一致,只有在不一致的情况下才需要指定表名

catalog:可选,表示Catalog名称，默认为Catalog\(" "\)。

schema:可选,表示Schema名称，默认为Schema\(" "\)。

  


3.@id

必须

@id定义了映射到数据库表的主键的属性,一个实体只能有一个属性被映射为主键.置于getXxxx\(\)前.

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
    /**
     * 艺术家
     */
    @Column(name = "AUTHOR")
    private String author;
    /**
     * 艺术家(英文)
     */
    @Column(name = "AUTHOR_EN")
    private String authorEn;
    /**
     * 图片URL
     */
    @Column(name = "IMAGEURL")
    private String imageUrl;
    /**
     * 类别    AN101：国画 AN107：油画
     */
    @Column(name = "TYPE")
    private Integer type;
    /**
     * 页面数
     */
    @Column(name = "PAGE_COUNT")
    private Integer pageCount;
    /**
     * 画心宽(mm)
     */
    @Column(name = "SIZE_W")
    private Integer sizeW;
    /**
     * 画心高(mm)
     */
    @Column(name = "SIZE_H")
    private Integer sizeH;
    /**
     * 画面宽(mm)
     */
    @Column(name = "PIC_SIZE_W")
    private Integer picSizeW;
    /**
     * 画面高(mm)
     */
    @Column(name = "PIC_SIZE_H")
    private Integer picSizeH;
    /**
     * 开始创作年份
     */
    @Column(name = "CREATION_YEAR_START")
    private Integer creationYearStart;
    /**
     * 结束创作年份
     */
    @Column(name = "CREATION_YEAR_END")
    private Integer creationYearEnd;
    /**
     * 创作年代
     */
    @Column(name = "WRITTENTIME")
    private String writtentime;
    /**
     * 创作年代(英文)
     */
    @Column(name = "WRITTENTIME_EN")
    private String writtentimeEn;
    /**
     * 材质  0：纸本1：绢本 2：镜心3：其它
     */
    @Column(name = "MATERIAL")
    private Integer material;
    /**
     * 装裱方式 0：立轴1：横轴 2：镜心3：册页
     */
    @Column(name = "MOUNTTYPE")
    private Integer mounttype;
    /**
     * 备注  0：有效 1：失效
     */
    @Column(name = "REMARK")
    private String remark;
}    
```







