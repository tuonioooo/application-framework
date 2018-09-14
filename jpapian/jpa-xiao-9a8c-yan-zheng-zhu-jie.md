# Jpa 校验/验证注解

## 注解说明

| 注解 | 说明 |
| :--- | :--- |
| @Null | 被注释的元素必须为 null |
| @NotNull | 被注释的元素必须不为 null |
| @AssertTrue | 被注释的元素必须为 true |
| @AssertFalse | 被注释的元素必须为 false |
| @Min\(value\) | 被注释的元素必须是一个数字，其值必须大于等于指定的最小值 |
| @Max\(value\) | 被注释的元素必须是一个数字，其值必须小于等于指定的最大值 |
| @DecimalMin\(value\) | 被注释的元素必须是一个数字，其值必须大于等于指定的最小值 |
| @DecimalMax\(value\) | 被注释的元素必须是一个数字，其值必须小于等于指定的最大值 |
| @Size\(max=, min=\) | 被注释的元素的大小必须在指定的范围内 |
| @Digits \(integer, fraction\) | 被注释的元素必须是一个数字，其值必须在可接受的范围内 |
| @Past | 被注释的元素必须是一个过去的日期 |
| @Future | 被注释的元素必须是一个将来的日期 |
| @Pattern\(regex=,flag=\) | 被注释的元素必须符合指定的正则表达式 eg:@Pattern\(regexp="^1\[3,4,5,6,7,8,9\]\\d{9}$", message="手机号码格式不正确"\) |
| Hibernate Validator 附加的 constraint |  |
| @NotBlank\(message =\) | 验证字符串非null，且长度必须大于0 |
| @Email | 被注释的元素必须是电子邮箱地址 |
| @Length\(min=,max=\) | 被注释的字符串的大小必须在指定的范围内 |
| @NotEmpty | 被注释的字符串的必须非空 |
| @Range\(min=,max=,message=\) | 被注释的元素必须在合适的范围内 |

## 示例一（单个实体类进行校验）：

WorkForm表单提交封装实体类——校验的bean

```
/**
 * Created by daizhao.
 * User: tony
 * Date: 2018-9-5
 * Time: 9:59
 * info: 作品信息form(15个属性)
 */
@Setter
@Getter
@NoArgsConstructor
@AllArgsConstructor
public class WorkForm {

    /** 作品编号/备案编号 */
    private String workid;

    /**
     * 作品名称
     */
    @NotBlank(message = "必填项")
    @Length(max=100, message="作品名称字符长度不能超过100")
    private String workname;
    /**
     * 作品名称(英文/拼音)
     */
    @Length(max=100, message="作品名称(英文/拼音)字符长度不能超过100")
    private String worknameEn;
    /**
     * 艺术家
     */
    @NotBlank(message = "必填项")
    @Length(max=30, message="艺术家字符长度不能超过30")
    private String author;
    /**
     * 图片URL
     */
    @NotBlank(message = "必填项")
    @Length(max=150, message="图片URL字符长度不能超过150")
    private String imageUrl;
    /**
     * 类别    AN101：国画 AN107：油画
     */
    @DecimalMin(value = "1", message="必须是正整数")
    private String type;
    /**
     * 画心宽(mm)
     */
    @DecimalMin(value = "1", message="必须是正整数")
    private String sizeW;
    /**
     * 画心高(mm)
     */
    @DecimalMin(value = "1", message="必须是正整数")
    private String sizeH;
    /**
     * 画面宽(mm)
     */
    @DecimalMin(value = "1", message="必须是正整数")
    private String picSizeW;
    /**
     * 画面高(mm)
     */
    @DecimalMin(value = "1", message="必须是正整数")
    private String picSizeH;
    /**
     * 开始创作年份
     */
    @DecimalMin(value = "1", message="必须是正整数")
    private String creationYearStart;
    /**
     * 结束创作年份
     */
    @DecimalMin(value = "1", message="必须是正整数")
    private String creationYearEnd;
    /**
     * 创作年代
     */
    @NotBlank(message = "必填项")
    @Length(max=100, message="创作年代字符长度不能超过100")
    private String writtentime;
    /**
     * 材质  0：纸本1：绢本 2：镜心3：其它
     */
    @DecimalMin(value = "1", message="必须是正整数")
    private String material;


    @NotBlank(message = "必填项")
    @Length(max=32, message="项目编号字符长度不能超过32")
    private String code;

    public Map<String, String> getFieldErrors(BindingResult result){
        Map<String, String> map_field_errors = new HashMap<String, String>();
        if(result.hasErrors()){
            if(result.hasFieldErrors("workname")){
                map_field_errors.put("workname", result.getFieldError("workname").getDefaultMessage());
            }
            if(result.hasFieldErrors("worknameEn")){
                map_field_errors.put("worknameEn", result.getFieldError("worknameEn").getDefaultMessage());
            }
            if(result.hasFieldErrors("author")){
                map_field_errors.put("author", result.getFieldError("author").getDefaultMessage());
            }
            if(result.hasFieldErrors("imageUrl")){
                map_field_errors.put("imageUrl", result.getFieldError("imageUrl").getDefaultMessage());
            }
            if(result.hasFieldErrors("type")){
                map_field_errors.put("type", result.getFieldError("type").getDefaultMessage());
            }
            if(result.hasFieldErrors("sizeW")){
                map_field_errors.put("sizeW", result.getFieldError("sizeW").getDefaultMessage());
            }
            if(result.hasFieldErrors("sizeH")){
                map_field_errors.put("sizeH", result.getFieldError("sizeH").getDefaultMessage());
            }
            if(result.hasFieldErrors("picSizeW")){
                map_field_errors.put("picSizeW", result.getFieldError("picSizeW").getDefaultMessage());
            }
            if(result.hasFieldErrors("picSizeH")){
                map_field_errors.put("picSizeH", result.getFieldError("picSizeH").getDefaultMessage());
            }
            if(result.hasFieldErrors("creationYearStart")){
                map_field_errors.put("creationYearStart", result.getFieldError("creationYearStart").getDefaultMessage());
            }
            if(result.hasFieldErrors("creationYearEnd")){
                map_field_errors.put("creationYearEnd", result.getFieldError("creationYearEnd").getDefaultMessage());
            }
            if(result.hasFieldErrors("writtentime")){
                map_field_errors.put("writtentime", result.getFieldError("writtentime").getDefaultMessage());
            }
            if(result.hasFieldErrors("material")){
                map_field_errors.put("material", result.getFieldError("material").getDefaultMessage());
            }
            if(result.hasFieldErrors("code")){
                map_field_errors.put("code", result.getFieldError("code").getDefaultMessage());
            }
        }
        return map_field_errors;

    }
}
```

Controller接口映射类——校验方法

```
   /**
     * 创建作品/修改作品
     * @param form
     * @param result
     * @return
     */
    @ResponseBody
    @PostMapping(value = "/saveOrUpdate")
    public String saveOrUpdate(@RequestBody @Valid WorkForm form, BindingResult result) {
        Map<String, String> map_field_errors = form.getFieldErrors(result);
        if(!map_field_errors.isEmpty()){
            CCResponse ccResponse = new CCResponse("业务异常[WORK表单校验失败]", RetCode.business_work_exception.code, map_field_errors);
            return ccResponse.toJson();
        }
        workManageService.saveOrUpdate(form);
        return new STResponse().toJson();
    }
```

> 说明：
>
> a.在接口参数的@Valid，来自于javax.validation.Valid;主要作用，表示当前的实体类要进行validation，只要当前的实体类，加了@DecimalMin、@NotBlank、@Length等注解以及上表介绍的注解，都会自动校验其属性是否合法
>
> b.@RequestBody 代表接受的是一个Json字符串
>
> c.BindingResult，这里@Valid的参数后必须紧挨着一个BindingResult 参数，否则spring会在校验不通过时直接抛出异常
>
> 绑定校验结果的错误信息，具体参考上面代码示例

## 示例二（嵌套实体类进行校验）：

主实体类——校验的bean

```
/**
 * Created by daizhao.
 * User: tony
 * Date: 2018-9-10
 * Time: 16:43
 * info: 显微检查页面Form
 */
@Setter
@Getter
@AllArgsConstructor
@NoArgsConstructor
public class MicroForm {

    /**
     * 作品编号
     */
    @NotBlank(message = "作品编号不能为空")
    @Length(max=25, message="作品编号字符长度不能超过25")
    private String workNo;
    /**
     * 页码
     */
    @DecimalMax(value="999999", message = "范围越界")
    @DecimalMin(value = "1", message="必须是正整数")
    private String pageNo;
    /**
     * 页面名称
     */
    @Length(max=100, message="页面名称字符长度不能超过25")
    private String pageName;
    /**
     * 标点图URL base64
     */
    @NotBlank(message = "标点图URL不能为空")
    private String imageUrl;
    /**
     * 检测点数量
     */
    @DecimalMax(value="999999", message = "范围越界")
    @DecimalMin(value = "1", message="必须是正整数")
    private String pointSum;
    /**
     * 检测人
     */
    @Length(max=20, message="检测人字符长度不能超过20")
    private String testMan;
    /**
     * 复检人
     */
    @Length(max=20, message="复检人字符长度不能超过20")
    private String recheckMan;
    /**
     * 备注
     */
    @Length(max=255, message="备注字符长度不能超过255")
    private String remark;
    /**
     * 最后修改人
     */
    @Length(max=20, message="最后修改人字符长度不能超过20")
    private String lastModifier;
    /**
     * 显微检测点
     */
    @Valid //关联被校验
    private List<MicroAppraisalForm> microAppraisalForms = new ArrayList<MicroAppraisalForm>();


    public Map<String, String> getFieldErrors(BindingResult result){
        Map<String, String> map_field_errors = new HashMap<String, String>();
        if(result.hasErrors()){
            if(result.hasFieldErrors("workNo")){
                map_field_errors.put("workNo", result.getFieldError("workNo").getDefaultMessage());
            }
            if(result.hasFieldErrors("pageNo")){
                map_field_errors.put("pageNo", result.getFieldError("pageNo").getDefaultMessage());
            }
            if(result.hasFieldErrors("pageName")){
                map_field_errors.put("pageName", result.getFieldError("pageName").getDefaultMessage());
            }
            if(result.hasFieldErrors("imageUrl")){
                map_field_errors.put("imageUrl", result.getFieldError("imageUrl").getDefaultMessage());
            }
            if(result.hasFieldErrors("pointSum")){
                map_field_errors.put("pointSum", result.getFieldError("pointSum").getDefaultMessage());
            }
            if(result.hasFieldErrors("testMan")){
                map_field_errors.put("testMan", result.getFieldError("testMan").getDefaultMessage());
            }
            if(result.hasFieldErrors("recheckMan")){
                map_field_errors.put("recheckMan", result.getFieldError("recheckMan").getDefaultMessage());
            }
            if(result.hasFieldErrors("remark")){
                map_field_errors.put("remark", result.getFieldError("remark").getDefaultMessage());
            }
            if(result.hasFieldErrors("lastModifier")){
                map_field_errors.put("lastModifier", result.getFieldError("lastModifier").getDefaultMessage());
            }
        }

        if(!microAppraisalForms.isEmpty()){
            JSONArray jsonArray = new JSONArray();
            microAppraisalForms.stream().forEach(microAppraisalForm -> {
                Map<String, String> details_field_errors = microAppraisalForm.getFieldErrors(result);
                if(!details_field_errors.isEmpty()){
                    jsonArray.add(details_field_errors);
                }
            });
            if(!jsonArray.isEmpty()){
                map_field_errors.put("microAppraisalForms", new JsonMapper().toJson(jsonArray));
            }
        }


        return map_field_errors;

    }
}
```

明细实体类——校验的bean

```

```



