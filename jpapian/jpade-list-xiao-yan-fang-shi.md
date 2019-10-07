# Jpa的list校验方式

```
/**
 * 可被校验的List
 *
 * @param <E> 元素类型
 * @author Deolin
 */
 public class ValidableList<E> implements List<E> {
 
    @Valid
    private List<E> list;
     
    public ValidableList() {
        this.list = new ArrayList<>();
    }
    
    public ValidableList(List<E> list) {
        this.list = list;
    }
    
    public List<E> getList() {
        return list;
    }

    public void setList(List<E> list) {
        this.list = list;
    }
    
    @Override
    public int size() {
        return list.size();
    }

    @Override
    public boolean isEmpty() {
        return list.isEmpty();
    }

    // 其他的Override方法省略。它们与size()和isEmpty()类似，直接调用private List list处理。 

 }

```

```
@PostMapping
public String noMore(@RequestBody @Valid ValidableList<User> users) {
    return "SUCCESS";
}

```



