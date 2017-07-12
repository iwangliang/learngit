## Modelling entities

### Schema
```

	// In the build.gradle file of your app project:
	android {
	...
	}
	 
	greendao {
		// 写在这里的
	    schemaVersion 2
	}

```

- schemaVersion

		数据库的版本号，*OpenHelpers类升级数据库时使用，如果你改变实体类，这个值必须增加。默认是1

- daoPackage

		生成的DAOs, DaoMaster, and DaoSession 的包名，默认是你实体类所在的包名

- targetGenDir

		与上面那个一起，这是是表示上面那个包放在哪个位置，默认是 build/generated/source/greendao 经常写成 src/main/java

- generateTests

		设置为 true 生成单元测试

- targetGenDirTests

		生成单元测试的位置，默认是 src/androidTest/java


### Entities and Annotations

	greendao 使用注解来定义实体，下面官网的例子

```java
	
	@Entity
	public class User {
	    @Id
	    private Long id;
	 
	    private String name;
	 
	    @Transient
	    private int tempUsageCount; // not persisted
	 
	   // getters and setters for id and user ...
	}

```

- @Entity
	
		标有@Entity 的java类将被转换为数据库的表
	>tips:注意，只有java类才能生成数据库的表，如果你使用kotlin 请保证你的bean 是java写的

	当然Entity还能有更多丰富的配置

	
		@Entity(
		        // If you have more than one schema, you can tell greenDAO
		        // to which schema an entity belongs (pick any string as a name).
		        schema = "myschema",
		        
		        // Flag to make an entity "active": Active entities have update,
		        // delete, and refresh methods.
		        active = true,
		        
		        // Specifies the name of the table in the database.
		        // By default, the name is based on the entities class name.
		        nameInDb = "AWESOME_USERS",
		        
		        // Define indexes spanning multiple columns here.
		        indexes = {
		                @Index(value = "name DESC", unique = true)
		        },
		        
		        // Flag if the DAO should create the database table (default is true).
		        // Set this to false, if you have multiple entities mapping to one table,
		        // or the table creation is done outside of greenDAO.
		        createInDb = false,
		 
		        // Whether an all properties constructor should be generated.
		        // A no-args constructor is always required.
		        generateConstructors = true,
		 
		        // Whether getters and setters for properties should be generated if missing.
		        generateGettersSetters = true
		)

	- schema

			不知道干啥的
	- active

			是否活跃，设置为 true 则这个实体类拥有 update delete refresh 方法
	- nameInDb

			设置在数据库中的名字，默认为类名
	- indexes

			设置索引
	- createInDb

			是否创建表，false 不创建，比如你的数据库中已经有这个表了，就不在创建了。默认为true
	- generateConstructors

			是否生成构造方法
	- generateGettersSetters 

			是否生成get/set方法

- 简单实体类


```java

	@Entity
	public class User {
	    @Id(autoincrement = true)
	    private Long id;
	 
	    @Property(nameInDb = "USERNAME")
	    private String name;
	 
	    @NotNull
	    private int repos;
	 
	    @Transient
	    private int tempUsageCount;
	 
	    ...
	}

```

- @Id

		标志为Id的必须是 Long/long 类型，相当于数据库的主键,autoincrement 表示自增

- @Property

		定义改字段在数据的名字，如果没有，默认用字段名，并且大写，用下划线连接，去掉驼峰。比如 customName ->  CUSTOM_NAME

- @NotNull 

		该字段不可为空

- @Transient

		该字段不存入数据库
> tips : Primary key restrictions : 关于主键限制，上面说过 greendao 的主键必须是Long/long 但是如果你的主键不是long/Long,也可以写成下面这样,为他创建一个唯一索引
 
	@Id
	private Long id;
	 
	@Index(unique = true)
	private String key;

- @Index

		为该字段加索引 name 属性可以改名字，unique 是否唯一

- @Unique

		该列值唯一，sqlite 默认会为他隐式加上索引

> tips : 关于 @Generated 和 @Keep ，官方不建议改自动生成的代码，如果有这样的需求，就去看官网吧



## Sessions



