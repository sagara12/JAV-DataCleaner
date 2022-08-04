# JAVA-DataCleaner 
## DataCleaner 클래스가 필요한 이유 
* JunitTest 중 테스트 격리가 이루어지지 않음
* DB의 초기화를 위해 해당 클래스가 필요

> DataCleanUp
```java
package com.datePage.service;

import org.springframework.beans.factory.InitializingBean;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import javax.persistence.Entity;
import javax.persistence.EntityManager;
import javax.persistence.PersistenceContext;
import java.util.List;
import java.util.stream.Collectors;

import com.google.common.base.CaseFormat;
//import org.springframework.test.context.ActiveProfiles;


@Service
public class DataCleanUp implements InitializingBean {

    @PersistenceContext
    private EntityManager entityManager;

    private List<String> tableNames;

    @Override
    public void afterPropertiesSet() {
        tableNames = entityManager.getMetamodel().getEntities().stream()
                .filter(e -> e.getJavaType().getAnnotation(Entity.class) !=null)
                .map(e -> CaseFormat.UPPER_CAMEL.to(CaseFormat.LOWER_UNDERSCORE, e.getName()))
                .collect(Collectors.toList());
    }

    @Transactional
    public void execute() {
        entityManager.flush();
        entityManager.createNativeQuery("SET REFERENTIAL_INTEGRITY FALSE").executeUpdate();

        for (String tableName : tableNames) {
            entityManager.createNativeQuery("TRUNCATE TABLE " + "write " + "RESTART IDENTITY").executeUpdate();
            //entityManager.createNativeQuery("ALTER TABLE " + "write" + " ALTER COLUMN ID RESTART WITH 1").executeUpdate();
        }

        entityManager.createNativeQuery("SET REFERENTIAL_INTEGRITY TRUE").executeUpdate();
    }
}

```


```java
   @BeforeEach
    void clean() {
        writeRepository.deleteAll();
        dataCleanUp.execute();
    }
```

