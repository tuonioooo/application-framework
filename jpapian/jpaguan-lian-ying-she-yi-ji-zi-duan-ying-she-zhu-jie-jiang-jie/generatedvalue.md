# @GeneratedValue

可选

strategy:表示主键生成策略，有AUTO,INDENTITY,SEQUENCE 和 TABLE 4种,分别表示让ORM框架自动选择,

根据数据库的Identity字段生成,根据数据库表的Sequence字段生成,以有根据一个额外的表生成主键,默认为AUTO

  


\(1\)identity主键自增,这种方式依赖于具体的数据库，如果数据库不支持自增主键，那么这个类型是没法用的

