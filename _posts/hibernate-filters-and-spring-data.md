---
layout: post
title: Using Hibernate filters with Spring Data
---

[Spring Data](http://docs.spring.io/spring-data/jpa/docs/2.0.0.M3/reference/html/) has become my preferred tool when accessing a relational database in Java. It removes 90% of the boiler-plate code (and ~90% of the bugs) typically required in a Java repository. The ease with which new queries can be created reminds me of the [ActiveRecord](http://guides.rubyonrails.org/active_record_basics.html) gem in Ruby, which makes it ridiculously quick and easy to interact with a database. If you are familiar with Spring Data, you know that most of this magic is accomplished at runtime; typically you just declare your repository interface and add any additional queries as needed:

```java
package com.example.repository;

import com.example.domain.Employee;
import org.springframework.data.jpa.repository.*;
import java.util.List;

/**
 * Spring Data JPA repository for the Employee entity.
 */
@SuppressWarnings("unused")
public interface EmployeeRepository extends JpaRepository<Employee,Long> {
	
	Optional<Employee> findFirstByFirstNameAndLastName(String firstName, String lastName);

}
```

All of the standard CRUD methods are defined in the JpaRepository, and I have defined an additional custom query to find the first employee that matches the provided first and latname parameters (which Spring Data will implement at runtime).  All well and good. 

Now, let's imagine that the business requires us to filter inactive Employees from the application if the current User is not a manager. We could implement this business logic in the Employee Service or Resource layer, but new functionality is being added daily and it isn't inconceivable that another service  will make use of our EmployeeRepository and mistakenly bypass our filtering logic. We could also provide a concrete implementation of our EmployeeRepository and re-implement the various query methods, but that would defeat the purpose of using Spring Data in the first place.

A better approach would be to implement a Hibernate filter on the entity. This will ensure that any filtering occurs beneath our repository on any queries to the employee table:

```java
package com.example.domain;

import org.hibernate.annotations.Cache;
import org.hibernate.annotations.CacheConcurrencyStrategy;

import javax.persistence.*;
import java.io.Serializable;
import java.time.LocalDate;
import java.util.Objects;

/**
 * An Employee.
 */
@Entity
@Table(name = "employee")
@Cache(usage = CacheConcurrencyStrategy.NONSTRICT_READ_WRITE)
@FilterDef(name="employeeFilter", parameters={
		@ParamDef(name="userRole", type="string")
})
@Filters( {
    @Filter(name="employeeFilter", 
    		condition="effective_end_date is null OR :userRole = 'MANAGER')")
})
public class Employee implements Serializable {

    private static final long serialVersionUID = 1L;

    @Id
    @GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "sequenceGenerator")
    @SequenceGenerator(name = "sequenceGenerator")
    private Long id;

    @Column(name = "effective_start_date")
    private LocalDate effectiveStartDate;

    @Column(name = "effective_end_date")
    private LocalDate effectiveEndDate;


    @Column(name = "first_name")
    private String firstName;

    @Column(name = "last_name")
    private String lastName;

	...
}

```

So, we have defined `employeeFilter` and it will automatically filter employees, permitting users to view employees only if the record's effective end date is null or the current user's role is 'MANAGER' (the manager is permitted to view all employees). All we have to do now is set the *userRole* parameter any time a repository is accessed. This is easily done using `@AspectJ`:

```java
package com.example.repository;

import javax.persistence.EntityManager;

import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Pointcut;
import org.hibernate.Filter;
import org.hibernate.Session;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.EnableAspectJAutoProxy;
import org.springframework.stereotype.Component;

import com.example.domain.User;
import com.example.domain.UserContext;

@Component
@Aspect
@EnableAspectJAutoProxy
public class RepositoryInterceptor {

	private final Logger logger = LoggerFactory.getLogger(RepositoryInterceptor.class);

	@Autowired
	private EntityManager entityManager;

	public RepositoryInterceptor() {
		logger.info("RepositoryInterceptor Started");
	}

	@Pointcut("this(org.springframework.data.repository.Repository)")
	public void repositoryLayerAccess() {
	}

	@Around("repositoryLayerAccess()")
	public Object proceed(ProceedingJoinPoint pjp) throws Throwable {
		logger.trace("intercepted repository call");
		if (entityManager.isJoinedToTransaction()) {
			Session session = entityManager.unwrap(Session.class);
			// retreive the user data from a ThreadLocal
			User user = UserContext.getUser();
			applyEmployeeFilterParams(session, user);
		}
		return pjp.proceed();
	}

	protected void applyEmployeeFilterParams(Session session, User user) {
		Filter filter = session.enableFilter("employeeFilter");
		filter.setParameter("userRole", user.getRole());
	}
}

```

Our RepositoryInterceptor will intercept any and all repository invocations, allowing us to set the employee filter (and any other filters we define) before the EmployeeRepository is accessed.

