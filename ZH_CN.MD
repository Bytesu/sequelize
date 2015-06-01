Sequelize是基于promise(约定)的ORM(对象关系映射).支持PostgresSQL\MySQL\MariaDB\SQLite和MSSQL方言,和一致性功能,关系,read复制或者更多
.

#开始
##安装
Sequelize可用通过NPM使用!

	$ npm install --save sequlize

*安装如下中一个*

	$ npm install --save pg pg-hstore
	$ npm install --save mysql //For both mysql and mariadb dialects
	$ npm install --save sqlite3
	$ npm install --save tedious //MSSQL

##建立连接

Sequelize在初始化时将建立一个连接池,所以每个应用程序应该进创建一次.

	var sequelize = new Sequelize('database','username','password',{
 		host:'localhost',
        dialect:'mysql'|'mariadb'|'sqlite'|'posgres'|'mssql',
        pool:{
            	max:5,
       			min:0,
		        idle:10000
      	},
      //只有SQLite需要
      storage:'path/to/database.sqlte'
      });
     //或者可以简单地使用一个连接url
 	 var sequelize = new Sequelize('postgres://user:pass@example.com:5432/dbname');
 

Sequelize构造器加载大量的可选项,可用的选项[API reference](http://sequelize.readthedocs.org/en/latest/api/sequelize/)

## 第一个Models(模型)
Models通过 *sequelize.define('name',{attributes},{options})* 定义.

	var User = sequelize.define('user',{
       firstName:Sequelize.STRING,
      	field:'first_name' //这种结果是用户看到的是firstName ,但数据库中实际存放的是fristName
	},
	lastName:{
		type:Sequelize.STRING
	}},{
		freezeTableName:true//模型的表名称将会和模型的名称一致
	});
	User.sync({force:true}).then(function(){
		//表的创建
		return User.create({
			firstName:'John',
			lastName:'Hancock'
		});
	});

更多的选项可以在[Model API reference (模型API 参考)](http://sequelize.readthedocs.org/en/latest/api/model/)中找到.


### 应用广泛的模型选项
Sequelize构造器需要定义默认选项, 默认选项将被用作已定义模型的默认选项(天才啊^V^).
	
	var sequelize = new Sequelize('connectionurl',{
		define:{
			timestamps:false //默认为true
		}
	});
	var User = sequelize.define('user',{})//timestamps选项默认为false
	var Post = sequelize.define('user',{},{
		timestamps:true //timestamps从此为true	
	});



# 表模式的使用
## 同步

sequelize.sync() 将会基于模型定义,创建任何missing表[?V?表示不理解,再次证明能力有限],如果选项force:true,将会在重新创建表之前先删除她们

## 迁移/手动模式变化

Sequelize有一个用于处理执行和迁移日志的[姐妹库(表示真的是这样翻译的吗?)](https://github.com/sequelize/umzug), Sequelize 提供了许多函数创建或修改表模式的的方法.

- 创建表
- 增加行
- 修改行
- 删除行
- 增加索引
- 删除索引
- 增加约束
- 删除约束
## 模型定义
定义模型和表之间映射关系使用 *define* 方法.  Sequelize会自动增加 *createdAt* 和 *updatedAt* 方法. 所以你能够获知数据库实体的创建的时间,已经最后一次被更新的时间. 如果你不想让时间戳作用在你的模型傻瓜你,或者尽享.....

	var Project = sequelize.define('Project',{
		title:Sequelize.STRING,
	   description:Sequelize.TEXT
	})
		
	var Task = sequelize.define('Task',{
		title:Sequelize.STRING,
	    description:Sequelize.TEXT,
	    deadline:Sequelize.DATE
	})

也可以为每一列设置选项:

	var Foo = sequelize.define('Foo',{
	//如果在初始化时没有为flag设值,flag将被自动设置为true
	flag:{
		type:Sequelize.BOOLEAN, 
	    allowNull:false,
	    defaultValue:true
	},
	//默認值爲當前時間
	myDate:{type:Sequelize.DATE,defaultVaule:Sequelize.NOW},
    //allowNull设置为false时将不允许此列值为null, 也就是说在查询时如果此列有空值,	将会抛出异常. 如果你想验证一个值在查询前不为空,查看下面的验证
		   title:{type:Sequelize.STRING,allowNull:false},
		  //创建两个相同值的对象将会抛出错误. 唯一属性值要么是布尔型,要么是字符串. 如果你为多列提供了相同的值,他们将形成相反的唯一键
		 someUnique:{type:Sequelize.STRING,unique:true},
		 uniqueOne:{type:Sequelize.STRING,unique:'compositeIndex'},
		 uniqueTwo:{type:Sequelize.INTEGER, unique:'compositeIndex'},
    //深入阅读关于主键的信息
      identifier:{type:Sequelize.STRING,primary:true},
	//autoIncrement选项可用来创建自动增加的整型列
    incrementMe:{type:Sequelize.INTEGER,autoIncrement:true},
	//MySQL和PG可以为每列指定评价
    hasComment:{ type:Sequelize.INTEGER,comment:'I am a comment!'}
    //通过field选项定制字段名称
    fieldWithUnderscores:{type:Sequelize.STRING,field:'field_with_underscore'},
    //可以创建外键
    bar_id:{
		type:Sequelize.INTEGER,
		references:{
			type:Bar,//这是对另一个模型的引用
		},
		key:'id',//引用模型的id列
        deferrable:Sequelize.Deferrable.INITIALLY_IMMEDIATE//	该选项只对PG其作用,声明什么时间检查外键约束
	}
	});

*comment* 选项也可以作用于表,具体参见[模型配置(model configuration)](http://docs.sequelizejs.com/en/latest/docs/models-definition/#configuration)

## 数据类型
下表是Sequelize支持的部分数据类型,对于Sequelize支持的全部和更新的数据类型,参见[数据类型(DataTypes)](http://docs.sequelizejs.com/en/latest/api/datatypes/)

	Sequelize.STRING 			//VARCHAR(255) 长度为255的可变字符
	Sequelize.STRING(1234)			//VARCHAR(1234)长度为1234的可变长字符
	Sequelize.STRING.BINARY	//VARCHAR BINARY
	Sequelize.TEXT                        // TEXT
	
	Sequelize.INTEGER                     // INTEGER
	Sequelize.BIGINT                      // BIGINT
	Sequelize.BIGINT(11)                  // BIGINT(11)
	
	Sequelize.FLOAT                       // FLOAT
	Sequelize.FLOAT(11)                   // FLOAT(11)
	Sequelize.FLOAT(11, 12)               // FLOAT(11,12)
	
	Sequelize.REAL                        // REAL        仅PostgreSQL 支持.
	Sequelize.REAL(11)                    // REAL(11)    PostgreSQL 支持.
	Sequelize.REAL(11, 12)                // REAL(11,12) PostgreSQL 支持.
	
	Sequelize.DOUBLE                      // DOUBLE
	Sequelize.DOUBLE(11)                  // DOUBLE(11)
	Sequelize.DOUBLE(11, 12)              // DOUBLE(11,12)
	
	Sequelize.DECIMAL                     // DECIMAL
	Sequelize.DECIMAL(10, 2)              // DECIMAL(10,2)
	
	Sequelize.DATE                        // DATETIME for mysql / sqlite, TIMESTAMP WITH TIME ZONE for postgres
	Sequelize.BOOLEAN                     // TINYINT(1)
	
	Sequelize.ENUM('value 1', 'value 2')  // An ENUM with allowed values 'value 1' and 'value 2'
	Sequelize.ARRAY(Sequelize.TEXT)       // Defines an array. PostgreSQL only.
	
	Sequelize.JSON                        // JSON column. PostgreSQL only.
	Sequelize.JSONB                       // JSONB column. PostgreSQL only.
	
	Sequelize.BLOB                        // BLOB (bytea for PostgreSQL)
	Sequelize.BLOB('tiny')                // TINYBLOB (bytea for PostgreSQL. Other options are medium and long)
	
	Sequelize.UUID                        // UUID datatype for PostgreSQL and SQLite, CHAR(36) BINARY for MySQL (use defaultValue: Sequelize.UUIDV1 or Sequelize.UUIDV4 to make sequelize generate the ids automatically)


BLOB数据类型允许插入字符串和buffer
