# entity-manager-configurations-for-each-database

// 1. Primary Database Configuration - PrimaryDataSourceConfig.java
package com.boswin.openKYC.config;

import com.zaxxer.hikari.HikariConfig;
import com.zaxxer.hikari.HikariDataSource;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.boot.orm.jpa.EntityManagerFactoryBuilder;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Primary;
import org.springframework.data.jpa.repository.config.EnableJpaRepositories;
import org.springframework.orm.jpa.JpaTransactionManager;
import org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean;
import org.springframework.transaction.PlatformTransactionManager;
import org.springframework.transaction.annotation.EnableTransactionManagement;

import javax.sql.DataSource;
import jakarta.persistence.EntityManagerFactory;

@Configuration
@EnableTransactionManagement
@EnableJpaRepositories(
        basePackages = "com.boswin.openKYC.repository.primary", // Primary DB repositories
        entityManagerFactoryRef = "primaryEntityManagerFactory",
        transactionManagerRef = "primaryTransactionManager"
)
public class PrimaryDataSourceConfig {

    @Bean
    @Primary
    @ConfigurationProperties("spring.datasource.primary")
    public HikariConfig primaryHikariConfig() {
        return new HikariConfig();
    }

    @Bean
    @Primary
    public DataSource primaryDataSource(@Qualifier("primaryHikariConfig") HikariConfig config) {
        return new HikariDataSource(config);
    }

    @Bean
    @Primary
    public LocalContainerEntityManagerFactoryBean primaryEntityManagerFactory(
            EntityManagerFactoryBuilder builder,
            @Qualifier("primaryDataSource") DataSource dataSource) {
        return builder
                .dataSource(dataSource)
                .packages("com.boswin.openKYC.entity.primary") // Primary entities package
                .persistenceUnit("primary")
                .build();
    }

    @Bean
    @Primary
    public PlatformTransactionManager primaryTransactionManager(
            @Qualifier("primaryEntityManagerFactory") EntityManagerFactory entityManagerFactory) {
        return new JpaTransactionManager(entityManagerFactory);
    }
}

// 2. HR Database Configuration - HrDataSourceConfig.java
package com.boswin.openKYC.config;

import com.zaxxer.hikari.HikariConfig;
import com.zaxxer.hikari.HikariDataSource;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.boot.orm.jpa.EntityManagerFactoryBuilder;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.jpa.repository.config.EnableJpaRepositories;
import org.springframework.orm.jpa.JpaTransactionManager;
import org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean;
import org.springframework.transaction.PlatformTransactionManager;
import org.springframework.transaction.annotation.EnableTransactionManagement;

import javax.sql.DataSource;
import jakarta.persistence.EntityManagerFactory;

@Configuration
@EnableTransactionManagement
@EnableJpaRepositories(
        basePackages = "com.boswin.openKYC.repository.hr", // HR DB repositories
        entityManagerFactoryRef = "hrEntityManagerFactory",
        transactionManagerRef = "hrTransactionManager"
)
public class HrDataSourceConfig {

    @Bean
    @ConfigurationProperties("spring.datasource.hr")
    public HikariConfig hrHikariConfig() {
        return new HikariConfig();
    }

    @Bean
    public DataSource hrDataSource(@Qualifier("hrHikariConfig") HikariConfig config) {
        return new HikariDataSource(config);
    }

    @Bean
    public LocalContainerEntityManagerFactoryBean hrEntityManagerFactory(
            EntityManagerFactoryBuilder builder,
            @Qualifier("hrDataSource") DataSource dataSource) {
        return builder
                .dataSource(dataSource)
                .packages("com.boswin.openKYC.entity.hr") // HR entities package
                .persistenceUnit("hr")
                .build();
    }

    @Bean
    public PlatformTransactionManager hrTransactionManager(
            @Qualifier("hrEntityManagerFactory") EntityManagerFactory entityManagerFactory) {
        return new JpaTransactionManager(entityManagerFactory);
    }
}

// 3. Primary Database Entity - User.java
package com.boswin.openKYC.entity.primary;

import jakarta.persistence.*;
import lombok.Data;
import lombok.NoArgsConstructor;
import lombok.AllArgsConstructor;

import java.time.LocalDateTime;

@Entity
@Table(name = "users")
@Data
@NoArgsConstructor
@AllArgsConstructor
public class User {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(name = "username", unique = true, nullable = false)
    private String username;
    
    @Column(name = "email", unique = true, nullable = false)
    private String email;
    
    @Column(name = "password", nullable = false)
    private String password;
    
    @Column(name = "first_name")
    private String firstName;
    
    @Column(name = "last_name")
    private String lastName;
    
    @Column(name = "phone")
    private String phone;
    
    @Enumerated(EnumType.STRING)
    @Column(name = "status")
    private UserStatus status = UserStatus.ACTIVE;
    
    @Column(name = "created_at")
    private LocalDateTime createdAt;
    
    @Column(name = "updated_at")
    private LocalDateTime updatedAt;
    
    @PrePersist
    protected void onCreate() {
        createdAt = LocalDateTime.now();
        updatedAt = LocalDateTime.now();
    }
    
    @PreUpdate
    protected void onUpdate() {
        updatedAt = LocalDateTime.now();
    }
    
    public enum UserStatus {
        ACTIVE, INACTIVE, SUSPENDED
    }
}

// 4. Primary Database Entity - KycDocument.java
package com.boswin.openKYC.entity.primary;

import jakarta.persistence.*;
import lombok.Data;
import lombok.NoArgsConstructor;
import lombok.AllArgsConstructor;

import java.time.LocalDateTime;

@Entity
@Table(name = "kyc_documents")
@Data
@NoArgsConstructor
@AllArgsConstructor
public class KycDocument {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(name = "user_id", nullable = false)
    private Long userId;
    
    @Column(name = "document_type", nullable = false)
    private String documentType;
    
    @Column(name = "document_number", nullable = false)
    private String documentNumber;
    
    @Column(name = "file_path")
    private String filePath;
    
    @Enumerated(EnumType.STRING)
    @Column(name = "verification_status")
    private VerificationStatus verificationStatus = VerificationStatus.PENDING;
    
    @Column(name = "verified_by")
    private Long verifiedBy;
    
    @Column(name = "verified_at")
    private LocalDateTime verifiedAt;
    
    @Column(name = "created_at")
    private LocalDateTime createdAt;
    
    @Column(name = "updated_at")
    private LocalDateTime updatedAt;
    
    @PrePersist
    protected void onCreate() {
        createdAt = LocalDateTime.now();
        updatedAt = LocalDateTime.now();
    }
    
    @PreUpdate
    protected void onUpdate() {
        updatedAt = LocalDateTime.now();
    }
    
    public enum VerificationStatus {
        PENDING, APPROVED, REJECTED
    }
}

// 5. HR Database Entity - Employee.java
package com.boswin.openKYC.entity.hr;

import jakarta.persistence.*;
import lombok.Data;
import lombok.NoArgsConstructor;
import lombok.AllArgsConstructor;

import java.time.LocalDate;
import java.time.LocalDateTime;

@Entity
@Table(name = "employees")
@Data
@NoArgsConstructor
@AllArgsConstructor
public class Employee {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(name = "employee_code", unique = true, nullable = false)
    private String employeeCode;
    
    @Column(name = "first_name", nullable = false)
    private String firstName;
    
    @Column(name = "last_name", nullable = false)
    private String lastName;
    
    @Column(name = "email", unique = true, nullable = false)
    private String email;
    
    @Column(name = "phone")
    private String phone;
    
    @Column(name = "department")
    private String department;
    
    @Column(name = "position")
    private String position;
    
    @Column(name = "hire_date")
    private LocalDate hireDate;
    
    @Column(name = "salary")
    private Double salary;
    
    @Enumerated(EnumType.STRING)
    @Column(name = "status")
    private EmployeeStatus status = EmployeeStatus.ACTIVE;
    
    @Column(name = "created_at")
    private LocalDateTime createdAt;
    
    @Column(name = "updated_at")
    private LocalDateTime updatedAt;
    
    @PrePersist
    protected void onCreate() {
        createdAt = LocalDateTime.now();
        updatedAt = LocalDateTime.now();
    }
    
    @PreUpdate
    protected void onUpdate() {
        updatedAt = LocalDateTime.now();
    }
    
    public enum EmployeeStatus {
        ACTIVE, INACTIVE, TERMINATED
    }
}

// 6. HR Database Entity - Department.java
package com.boswin.openKYC.entity.hr;

import jakarta.persistence.*;
import lombok.Data;
import lombok.NoArgsConstructor;
import lombok.AllArgsConstructor;

import java.time.LocalDateTime;

@Entity
@Table(name = "departments")
@Data
@NoArgsConstructor
@AllArgsConstructor
public class Department {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(name = "dept_code", unique = true, nullable = false)
    private String deptCode;
    
    @Column(name = "dept_name", nullable = false)
    private String deptName;
    
    @Column(name = "description")
    private String description;
    
    @Column(name = "manager_id")
    private Long managerId;
    
    @Column(name = "budget")
    private Double budget;
    
    @Column(name = "created_at")
    private LocalDateTime createdAt;
    
    @Column(name = "updated_at")
    private LocalDateTime updatedAt;
    
    @PrePersist
    protected void onCreate() {
        createdAt = LocalDateTime.now();
        updatedAt = LocalDateTime.now();
    }
    
    @PreUpdate
    protected void onUpdate() {
        updatedAt = LocalDateTime.now();
    }
}

// 7. Primary Database Repository - UserRepository.java
package com.boswin.openKYC.repository.primary;

import com.boswin.openKYC.entity.primary.User;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

import java.util.Optional;
import java.util.List;

@Repository
public interface UserRepository extends JpaRepository<User, Long> {
    
    Optional<User> findByUsername(String username);
    
    Optional<User> findByEmail(String email);
    
    List<User> findByStatus(User.UserStatus status);
    
    Boolean existsByUsername(String username);
    
    Boolean existsByEmail(String email);
}

// 8. Primary Database Repository - KycDocumentRepository.java
package com.boswin.openKYC.repository.primary;

import com.boswin.openKYC.entity.primary.KycDocument;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

import java.util.List;

@Repository
public interface KycDocumentRepository extends JpaRepository<KycDocument, Long> {
    
    List<KycDocument> findByUserId(Long userId);
    
    List<KycDocument> findByVerificationStatus(KycDocument.VerificationStatus status);
    
    List<KycDocument> findByUserIdAndDocumentType(Long userId, String documentType);
}

// 9. HR Database Repository - EmployeeRepository.java
package com.boswin.openKYC.repository.hr;

import com.boswin.openKYC.entity.hr.Employee;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Query;
import org.springframework.data.repository.query.Param;
import org.springframework.stereotype.Repository;

import java.util.List;
import java.util.Optional;

@Repository
public interface EmployeeRepository extends JpaRepository<Employee, Long> {
    
    Optional<Employee> findByEmployeeCode(String employeeCode);
    
    Optional<Employee> findByEmail(String email);
    
    List<Employee> findByDepartment(String department);
    
    List<Employee> findByStatus(Employee.EmployeeStatus status);
    
    @Query("SELECT e FROM Employee e WHERE e.firstName LIKE %:name% OR e.lastName LIKE %:name%")
    List<Employee> findByNameContaining(@Param("name") String name);
}

// 10. HR Database Repository - DepartmentRepository.java
package com.boswin.openKYC.repository.hr;

import com.boswin.openKYC.entity.hr.Department;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

import java.util.Optional;

@Repository
public interface DepartmentRepository extends JpaRepository<Department, Long> {
    
    Optional<Department> findByDeptCode(String deptCode);
    
    Optional<Department> findByDeptName(String deptName);
}

// 11. Primary Database Service - UserService.java
package com.boswin.openKYC.service;

import com.boswin.openKYC.entity.primary.User;
import com.boswin.openKYC.repository.primary.UserRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import java.util.List;
import java.util.Optional;

@Service
@Transactional("primaryTransactionManager") // Use primary transaction manager
public class UserService {
    
    @Autowired
    private UserRepository userRepository;
    
    @Transactional(readOnly = true)
    public List<User> getAllUsers() {
        return userRepository.findAll();
    }
    
    @Transactional(readOnly = true)
    public Optional<User> getUserById(Long id) {
        return userRepository.findById(id);
    }
    
    @Transactional(readOnly = true)
    public Optional<User> getUserByUsername(String username) {
        return userRepository.findByUsername(username);
    }
    
    @Transactional
    public User saveUser(User user) {
        return userRepository.save(user);
    }
    
    @Transactional
    public void deleteUser(Long id) {
        userRepository.deleteById(id);
    }
}

// 12. HR Database Service - EmployeeService.java
package com.boswin.openKYC.service;

import com.boswin.openKYC.entity.hr.Employee;
import com.boswin.openKYC.repository.hr.EmployeeRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import java.util.List;
import java.util.Optional;

@Service
@Transactional("hrTransactionManager") // Use HR transaction manager
public class EmployeeService {
    
    @Autowired
    private EmployeeRepository employeeRepository;
    
    @Transactional(readOnly = true)
    public List<Employee> getAllEmployees() {
        return employeeRepository.findAll();
    }
    
    @Transactional(readOnly = true)
    public Optional<Employee> getEmployeeById(Long id) {
        return employeeRepository.findById(id);
    }
    
    @Transactional(readOnly = true)
    public Optional<Employee> getEmployeeByCode(String employeeCode) {
        return employeeRepository.findByEmployeeCode(employeeCode);
    }
    
    @Transactional
    public Employee saveEmployee(Employee employee) {
        return employeeRepository.save(employee);
    }
    
    @Transactional
    public void deleteEmployee(Long id) {
        employeeRepository.deleteById(id);
    }
}
