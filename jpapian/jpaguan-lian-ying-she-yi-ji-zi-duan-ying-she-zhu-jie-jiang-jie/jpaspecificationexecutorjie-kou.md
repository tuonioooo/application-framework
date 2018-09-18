# JpaSpecificationExecutor接口

## 概述

封装 JPA Criteria 查询条件。通常使用匿名内部类的方式来创建该接口的对象

## 接口使用方式——Repository

```
public interface TaskPlanRepository extends JpaRepository<TaskPlan, String>, JpaSpecificationExecutor<TaskPlan> {

}
```

官方接口API：[https://docs.spring.io/spring-data/jpa/docs/current/api/org/springframework/data/jpa/repository/JpaSpecificationExecutor.html](https://docs.spring.io/spring-data/jpa/docs/current/api/org/springframework/data/jpa/repository/JpaSpecificationExecutor.html)

## 复杂查询场景

* **分页查询**

```
/***
* 项目分页查询
*/
public Page<TaskPlan> getTPListPage(TpFormVO vo) {
        Sort sort = new Sort(Sort.Direction.DESC, "createTime").and(new Sort(Sort.Direction.DESC, "code"));
        Pageable pageable = new PageRequest(vo.getCurrentPage(), vo.getSize(), sort);
        return taskPlanRepository.findAll(new Specification<TaskPlan>() {

            @Override
            public Predicate toPredicate(Root<TaskPlan> root, CriteriaQuery<?> query, CriteriaBuilder cb) {
                List<Predicate> predicates = new ArrayList<Predicate>();

                if (StringUtils.isNotBlank(vo.getCreateTimeStart())) {//创建开始时间
                    predicates.add(cb.greaterThanOrEqualTo(root.<Date> get("createTime"), DateUtils.parseDate(vo.getCreateTimeStart())));
                }
                if (StringUtils.isNotBlank(vo.getCreateTimeEnd())) {//创建结束时间
                    predicates.add(cb.lessThanOrEqualTo(root.<Date> get("createTime"), DateUtils.parseDate(vo.getCreateTimeEnd())));
                }

                if (StringUtils.isNotBlank(vo.getDeliveryTimeStart())) {//交付开始时间
                    predicates.add(cb.greaterThanOrEqualTo(root.<Date> get("jjqzDate"), DateUtils.parseDate(vo.getDeliveryTimeStart())));
                }
                if (StringUtils.isNotBlank(vo.getDeliveryTimeEnd())) {//交付结束时间
                    predicates.add(cb.lessThanOrEqualTo(root.<Date> get("jjqzDate"), DateUtils.parseDate(vo.getDeliveryTimeEnd())));
                }
                if (StringUtils.isNotBlank(vo.getStatus()) && !vo.isALLStatus()) {//状态
                    predicates.add(cb.equal(root.<String> get("status"), vo.getStatus()));
                }

                //默认数据状态为1
                predicates.add(cb.equal(root.<String> get("state"), TaskPlan.STATE_TP.state_s.key));

                if (!predicates.isEmpty()) {
                    return cb.and(predicates.toArray(new Predicate[predicates.size()]));
                } else {
                    return cb.conjunction();
                }
            }
        }, pageable);
    }
```

* 实现sql条件"&gt;=" 和 "&lt;="的用法

```
predicates.add(cb.greaterThanOrEqualTo(root.<Date> get("createTime"), DateUtils.parseDate(vo.getCreateTimeStart())));

predicates.add(cb.lessThanOrEqualTo(root.<Date> get("createTime"), DateUtils.parseDate(vo.getCreateTimeEnd())));
```

greaterThanOrEqualTo代表 &gt;= 当前条件时间， lessThanOrEqualTo代表&lt;=当前条件时间，具体代码实现，参考上面的项目分页代码

* 实现sql条件“=”的用法

```
Predicate predicate = cb.equal(root.<String> get("state"), TaskPlan.STATE_TP.state_s.key)
```

equal的用法，详细参考上面的项目分页代码

同样的不要忘记notEqual方法

```
predicates.add(cb.notEqual(root.<String> get("status"), ProcurementInformationEnum.INIT.value));
```

* 实现sql条件“in”的用法

```
/**
     * 作品分页查询
     * @param vo
     * @return
     */
    public Page<WorkManage> getWorkManageListPage(WorkFormVO vo) {

        try{
            TaskPlan taskPlan = taskPlanService.getTP(vo.getCode());
            Sort sort = new Sort(Sort.Direction.DESC, "taskPlanCascade");
            Pageable pageable = new PageRequest(vo.getCurrentPage(), vo.getSize(), sort);
            return workManageRepository.findAll(new Specification<WorkManage>() {
                @Override
                public Predicate toPredicate(Root<WorkManage> root, CriteriaQuery<?> query, CriteriaBuilder cb) {
                    List<Predicate> predicates = new ArrayList<Predicate>();
                    if (StringUtils.isNotBlank(vo.getType())) {//类别
                        if(vo.getType().contains(",")){
                            CriteriaBuilder.In<Integer> in = cb.in(root.get("type"));
                            Arrays.stream(vo.getType().split(",")).forEach(type->{
                                in.value(Integer.parseInt(type));
                            });
                            predicates.add(in);
                        }else{
                            predicates.add(cb.equal(root.<Integer> get("type"), Integer.parseInt(vo.getType())));
                        }
                    }
                    if (StringUtils.isNotBlank(vo.getMaterial())) {//材质
                        if(vo.getMaterial().contains(",")){
                            CriteriaBuilder.In<Integer> in = cb.in(root.get("material"));
                            Arrays.stream(vo.getMaterial().split(",")).forEach(material->{
                                in.value(Integer.parseInt(material));
                            });
                            predicates.add(cb.or(in));
                        }else{
                            predicates.add(cb.equal(root.<Integer> get("material"), Integer.parseInt(vo.getMaterial())));
                        }
                    }
                    if (StringUtils.isNotBlank(vo.getStatus())) {//鉴定状态
                        Join<Object, Object> workManage2Join = root.join("workManage2");
                        predicates.add(cb.equal(workManage2Join.<Integer> get("status"), Integer.parseInt(vo.getStatus())));
                    }
                    predicates.add(cb.equal(root.<TaskPlan> get("taskPlanCascade"), taskPlan));//关联项目关联
                    if (!predicates.isEmpty()) {
                        return cb.and(predicates.toArray(new Predicate[predicates.size()]));
                    } else {
                        return cb.conjunction();
                    }
                }
            }, pageable);
        }catch (Exception e){
            logger.error(e.getMessage());
            throw_process_exception(RetCode.business_work_exception.code, "业务异常[WORK分页获取异常]", e);
            return null;
        }

    }
```

也是一整块分页代码，其中，in的使用方式，是我们关注的要点如下：

```
CriteriaBuilder.In<Integer> in = cb.in(root.get("material"));
Arrays.stream(vo.getMaterial().split(",")).forEach(material->{
in.value(Integer.parseInt(material));
});
predicates.add(cb.or(in));
```

通过循环获取值，放到in的属性值里，在最后通过cb.or\(in\) 连接使用，打印的部分SQL如下：

```
where (
            workmanage0_.TYPE in (
                1 , 2
            )
        )
```

* 关联查询的用法

```
Join<Object, Object> workManage2Join = root.join("workManage2");
predicates.add(cb.equal(workManage2Join.<Integer> get("status"), Integer.parseInt(vo.getStatus())));
```

root.join\("workManage2"\) 连接的是当前对象的关联属性对象，创建一个Join对象，就可以通过该join对象，通过查询条件，关联查询数据，具体使用，参考上面的作品分页查询代码

* 实现sql条件“or”的用法、"like"的用法

```
/**
     * 分页查询招标管理信息(用于后台)
     * @param pageable
     * @return
     */
    public Page<InviteBidManage> listForPage(final InviteBidManage inviteBidManage, Pageable pageable) {
        return inviteBidManageDao.findAll(new Specification<InviteBidManage>() {

            @Override
            public Predicate toPredicate(Root<InviteBidManage> root, CriteriaQuery<?> query, CriteriaBuilder cb) {
                List<Predicate> predicates = new ArrayList<Predicate>();
                if (inviteBidManage == null) {
                    return cb.conjunction();
                }
                if (StringUtils.isNotBlank(inviteBidManage.getCondition())) {
                    Predicate projectNamePredicate = cb.like(root.<String> get("projectName"), "%" + inviteBidManage.getCondition() + "%");
                    Predicate projectCodePredicate = cb.like(root.<String> get("projectCode"), "%" + inviteBidManage.getCondition() + "%");
                    predicates.add(cb.or(projectNamePredicate, projectCodePredicate));
                }
                if (StringUtils.isNotBlank(inviteBidManage.getProperty())) {
                    predicates.add(cb.equal(root.<String> get("property"), inviteBidManage.getProperty()));
                }
                if (StringUtils.isNotBlank(inviteBidManage.getSource())) {
                    predicates.add(cb.equal(root.<String> get("source"), inviteBidManage.getSource()));
                }
                if (inviteBidManage.getState() != null && inviteBidManage.getState() != -1) {
                    predicates.add(cb.equal(root.<Integer> get("state"), inviteBidManage.getState()));
                }

                query.orderBy(cb.desc(root.<String> get("createdAt")));

                if (!predicates.isEmpty()) {
                    return cb.and(predicates.toArray(new Predicate[predicates.size()]));
                } else {
                    return cb.conjunction();
                }
            }
        }, pageable);
    }
```

根据上面的招标分页查询，or、like的用法如下：

```
if (StringUtils.isNotBlank(inviteBidManage.getCondition())) {
       Predicate projectNamePredicate = cb.like(root.<String> get("projectName"), "%" + inviteBidManage.getCondition() + "%");
       Predicate projectCodePredicate = cb.like(root.<String> get("projectCode"), "%" + inviteBidManage.getCondition() + "%");
       predicates.add(cb.or(projectNamePredicate, projectCodePredicate));
}
```

具体实现参考招标分页查询实现

* 排序用法

```
query.orderBy(cb.desc(root.<String> get("createdAt")));
```

上面的是通过CriteriaBuilder实现排序，也可以通过Pageable的方式实现

```
Sort sort = new Sort(Sort.Direction.DESC, "createTime").and(new Sort(Sort.Direction.DESC, "code"));
Pageable pageable = new PageRequest(vo.getCurrentPage(), vo.getSize(), sort);
```

具体实现参考招标分页查询实现、项目分页查询实现







