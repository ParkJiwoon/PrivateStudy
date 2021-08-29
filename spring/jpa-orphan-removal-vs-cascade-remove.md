# JPA Entity 삭제: orphanRemoval vs CascadeType.REMOVE

# 1. Overview

JPA 에서 연관 관계를 설정할 때 여러가지 옵션을 추가할 수 있습니다.

삭제에 관련된 옵션은 `orphanRemoval` 와 `cascade` 가 있는데 둘이 어떤점이 다른지 알아봅니다.

<br>

# 2. orphanRemoval

JPA 2.0 부터 지원하며 부모 엔티티와 관계가 끊어진 자식 엔티티를 자동으로 삭제해줍니다.

`@OneToMany` 와 `@OneToOne` 에서 지원하는 옵션입니다.

아마 `@ManyToOne` 는 보통 연관 관계의 주인인 엔티티가 사용해서 저 옵션이 없는 것 같네요.

<br>

```java
@Entity
public class School {

    @OneToMany(mappedBy = "school", orphanRemoval = true)
    private List<Teacher> teachers = new ArrayList<>();
}

@Entity
public class Teacher {

    @ManyToOne
    @JoinColumn(name = "school_id")
    private School school;
}
```

위 도메인은 `School(1) <-> Teacher(N)` 인 다대일 양방향 관계를 나타냅니다.

원래는 연관 관계의 주인만 외래키의 저장, 조회, 삭제 권한을 갖기 때문에 `School.teachers` Collections 에서 데이터를 지워도 반영되지 않습니다.

그러나 `orphanRemoval = true` 옵션을 추가해주면 부모 엔티티의 컬렉션에서 자식 엔티티를 삭제할 때 참조가 끊어지면서 DB 에서도 삭제됩니다.

<br>

# 3. CascadeType.REMOVE

```java
@Entity
public class School {

    @OneToMany(mappedBy = "school", cascade = CascadeType.REMOVE)
    private List<Teacher> teachers = new ArrayList<>();
}

@Entity
public class Teacher {

    @ManyToOne
    @JoinColumn(name = "school_id")
    private School school;
}
```

`cascade` 는 영속성 전이에 관한 옵션입니다.

설정된 엔티티가 저장/수정/삭제 될 때 연관된 엔티티들도 전부 동일한 액션을 해줍니다.

그래서 연관 관계의 주인이 아니더라도 해당 엔티티를 지우면 관련된 모든 엔티티들이 삭제됩니다.

<br>

# 4. Conclusion

- 부모 엔티티가 삭제되면 자식 엔티티도 전부 삭제되는 것은 동일하지만 원인이 다름
  - `orphanRemoval = true` 옵션은 부모 엔티티가 사라지면서 자식 엔티티와의 참조(연결)가 끊어져서 삭제됨
  - `CascadeType.REMOVE` 옵션은 원래 엔티티가 삭제될 때 연관된 엔티티를 전부 삭제하는 옵션
- `orphanRemoval` 옵션은 Collections 에서 자식 엔티티를 삭제하는 걸로 DB 에서도 삭제 가능하지만 `cascade` 는 불가능


<br>

# Reference

- [Deleting JPA Entity Objects - Orphan Removal](https://www.objectdb.com/java/jpa/persistence/delete#Orphan_Removal_)
- [JPA 고아 객체](https://ultrakain.gitbooks.io/jpa/content/chapter8/chapter8.5.html)