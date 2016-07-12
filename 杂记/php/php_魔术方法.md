#PHP 魔术方法

---

###魔术方法

* `__construct()`: 构造函数.
* `__destruct()`: 析构函数.
* `__call()`: 在对象中调用一个不可访问方法时.
* `__callStatic()`: 用静态方式中调用一个不可访问方法时.
* `__get()`: 读取不可访问属性的值时被调用.
* `__set()`: 在给不可访问属性赋值时被调用.
* `__isset()`: 当对不可访问属性调用 `isset()` 或 `empty()` 时, `__isset()` 会被调用.
* `__unset()`: 当对不可访问属性调用 `unset()` 时, `__unset()` 会被调用.
* `__sleep()`: serialize时调用,必须返回一个包含对象中所有应被序列化的变量名称的数组.
* `__wakeup()`: unserialize时调用.
* `__toString()`: 一个类被当成字符串时应怎样回应.`echo $obj;`这样的情况.
* `__invoke()`: 以调用函数的方式调用一个对象时
* `__set_state()`: `var_export()`时调用该静态方法
* `__clone()`: 克隆对象时被调用.
* `__debugInfo()`: `var_dump()`时调用该方法,该方法必须返回一个数组.

####`__sleep()`和`__wakeup()`

`serialize()` 函数会检查类中是否存在一个魔术方法 `__sleep()`. 如果存在,该方法会先被调用,然后才执行序列化操作,并返回一个包含对象中所有应被序列化的变量名称的数组.如果该方法未返回任何内容,则 `NULL` 被序列化,并产生一个 E_NOTICE 级别的错误.

`__sleep()`方法必须返回一个数组,包含需要串行化的属性,PHP会抛弃其它属性的值.如果没有`__sleep()`方法,PHP将保存所有属性.

`unserialize()` 会检查是否存在一个 `__wakeup()` 方法.如果存在,则会先调用 `__wakeup` 方法,预先准备对象需要的资源.

**`__sleep()`示例**

		class A 
		{
			private $a = 1;
			private $b = 2;
		}

		$obj = new A();
		
		var_dump(serialize($obj));
		
		输出:string(42) "O:1:"A":2:{s:4:"Aa";i:1;s:4:"Ab";i:2;}"
		
我们使用__sleep,不期望序列化$b的内容
		
		class A 
		{
			private $a = 1;
			private $b = 2;
			
			public function __sleep()
		    {
		        return array('a');
		    }
		}

		$obj = new A();
		
		var_dump(serialize($obj));
		
		输出:string(27) "O:1:"A":1:{s:4:"Aa";i:1;}"
		
我们用`__sleep()`,不返回任何内容.

		
		class A 
		{
			private $a = 1;
			private $b = 2;
			
			public function __sleep()
		    {
		        return ;
		    }
		}

		$obj = new A();
		
		var_dump(serialize($obj));	
		
		输出(如上所述 NULL 被序列化,并产生一个 E_NOTICE 级别的错误):
		PHP Notice:  serialize(): __sleep should return an array only containing the names of instance-variables to serialize in /Users/chloroplast1983/Sites/test/MagicMthods/sleepAndWakeup.php on line 21
		
		Notice: serialize(): __sleep should return an array only containing the names of instance-variables to serialize in /Users/chloroplast1983/Sites/test/MagicMthods/sleepAndWakeup.php on line 21
		string(2) "N;"	
		
**`__wakeup()`示例**

		class A 
		{
			private $a = 1;
			private $b = 2;
		    
		    public function __wakeup()
		    {
		        echo 'wake up', PHP_EOL;
		    }
		}
		
		$obj = new A();
		$serializeObj = serialize($obj);
		var_dump(unserialize($serializeObj));
		
		输出(第一行输出wakeup):
		wake up
		object(A)#2 (2) {
		  ["a":"A":private]=>
		  int(1)
		  ["b":"A":private]=>
		  int(2)
		}
		
####`__toString()`

`__toString()` 方法用于一个类被当成字符串时应怎样回应.例如 `echo $obj;` 应该显示些什么.
此方法必须**返回**一个字符串,否则将发出一条 `E_RECOVERABLE_ERROR` 级别的致命错误.

**没有`__toString()`示例**

		class A 
		{
			private $a = 1;
			private $b = 2;
		}

		$obj = new A();
		
		echo $obj;
		
		输出:
		PHP Catchable fatal error:  Object of class A could not be converted to string in /Users/chloroplast1983/Sites/test/MagicMthods/toString.php on line 16

		Catchable fatal error: Object of class A could not be converted to string in /Users/chloroplast1983/Sites/test/MagicMthods/toString.php on line 16

**`__toString()`示例**

		class A 
		{
			private $a = 1;
			private $b = 2;
			
		    public function __toString()
		    {
		       return "I am Class A".PHP_EOL;
		    }
		}

		$obj = new A();
		
		echo $obj;
		
		输出:
		I am Class A

####`__invoke()`

当尝试**以调用函数的方式调用一个对象时**,`__invoke()` 方法会被自动调用.

**示例**

		class A 
		{
			function __invoke($x) {
		        var_dump($x);
		    }
		}
		
		$obj = new A();
		
		$obj(5);

####`__set_state()`

`PHP 5.1.0` 起当调用 `var_export()` 导出类时,此静态方法会被调用.

本方法的唯一参数是一个数组,其中包含按 array('property' => value, ...)格式排列的类属性.

就是`var_export()`的`回调函数`.

**`var_export`**

输出或返回一个变量的字符串表示.

`mixed var_export ( mixed $expression [, bool $return ] )`

此函数返回关于传递给该函数的变量的结构信息,它和 `var_dump()` 类似, 不同的是其返回的表示是合法的 PHP 代码.

也就是说,`var_export()`返回的代码,可以直接当作php代码赋值个一个变量.而这个变量就会取得和被var_export一样的类型的值.但是,当变量类型为resource的时候,是无法简单copy复制的,所以,当var_export的变量是resource类型时,var_export会返回NULL.
		
示例:

		$a = array (1, 2, array ("a", "b", "c"));
		var_export ($a);
		输出:
		array (
		  0 => 1,
		  1 => 2,
		  2 =>
		  array (
		    0 => 'a',
		    1 => 'b',
		    2 => 'c',
		  ),
		)
		
		$a = array (1, 2, array ("a", "b", "c"));
		$b = var_export ($a,true);
		
		echo $b;
		输出:
		array (
		  0 => 1,
		  1 => 2,
		  2 =>
		  array (
		    0 => 'a',
		    1 => 'b',
		    2 => 'c',
		  ),
		)

**示例`__set_state()`**

我们创建一个类然后执行`var_export()`观察其输出:

		class A
		{
		    public $var1 = 1;
		    public $var2 = 2;
		}

		$a = new A();
		$b = var_export($a,true);
		
		var_dump($b);
		
输出是:

		string(56) "A::__set_state(array(
		   'var1' => 1,
		   'var2' => 2,
		))"
		
这是一段合法的php代码,它代表一个 static __set_state 方法在我们的 Class A.如果我们要执行它的话就要使用`eval`函数.

		class A
		{
		    public $var1 = 1;
		    public $var2 = 2;
		}

		$a = new A();
		eval('$b = '.var_export($a,true).';');
		
		var_dump($b);
		
输出是:

		PHP Fatal error:  Call to undefined method A::__set_state()...
		
我们不能执行这段PHP代码,因为我们的PHP类没有`__set_state()`,我们必须创建一个.

		class A
		{
		    public $var1 = 1;
		    public $var2 = 2;
		
		    public static function __set_state($array) // As of PHP 5.1.0
		    {
		       $tmp = new A();
		       $tmp->var1 = 3;
		       $tmp->var2 = 4;
		       return $tmp;
		    }
		}
		
		$a = new A();
		eval('$b = '.var_export($a,true).';');
		var_dump($b);
		
输出是:

		object(A)#2 (2) {
		  ["var1"]=>
		  int(3)
		  ["var2"]=>
		  int(4)
		}
		
我们可以看见我们`__set_state`内部的代码已经被执行了.	
		
####`__debugInfo()`

在调用`var_dump()`时触发调用该方法.该方法必须返回数组.

**示例**

		class A
		{
		    public $var1 = 1;
		    public $var2 = 2;
		}
		
		$a = new A();
		var_dump($a);
		
输出:

		object(A)#1 (2) {
		  ["var1"]=>
		  int(1)
		  ["var2"]=>
		  int(2)
		}

我们加上`__debugInfo()`方法:

		class A
		{
		    public $var1 = 1;
		    public $var2 = 2;
		
		    public function __debugInfo() {
		        return array('Hello World');
		    }
		}
		
		$a = new A();
		var_dump($a);

输出:

		object(A)#1 (1) {
		  [0]=>
		  string(11) "Hello World"
		}

####`__construct()`和`__destruct()`

`__construct()`构造函数,`__destruct()`析构函数.

**示例**

		class A
		{
			public function __construct()
			{
				print 'In constructor'.PHP_EOL;
			}
		
			public function __destruct()
			{
				print 'In destructor'.PHP_EOL;
			}
		}
		
		$a = new A();
		
输出:

In constructor

In destructor

####`__call()`和`__callStatic()`

`public mixed __call ( string $name , array $arguments )`

在对象中调用一个不可访问方法时,`__call()` 会被调用.

`public static mixed __callStatic ( string $name , array $arguments )`

用静态方式中调用一个不可访问方法时,`__callStatic()` 会被调用. php5.3.0之后的版本.

* `$name`: 参数是要调用的方法名称.
* `$arguments`: 参数是一个枚举数组,包含着要传递给方法`$name`的参数.

**示例**

		class A
		{
			public function __call($name, $arguments) 
		    {
		        echo "Calling object method '$name' "
		             . implode(', ', $arguments). "\n";
		    }
		
		    public static function __callStatic($name, $arguments) 
		    {
		        // 注意: $name 的值区分大小写
		        echo "Calling static method '$name' "
		             . implode(', ', $arguments). "\n";
		    }
		
		    private function test()
		    {
		    	return 100;
		    }
		}
		
		$a = new A;
		
		//存在的私有方法
		$a->test("hello","world");
		//不存在方法
		$a->notExist("hello","world");
		//静态调用
		A::notExist("hello","world");
		
		输出:
		
		Calling object method 'test' hello, world
		Calling object method 'notExist' hello, world
		Calling static method 'notExist' hello, world

####`__set()`和`__get()`

在静态方法中,这些魔术方法将不会被调用

`public void __set ( string $name , mixed $value )`

在给不可访问属性赋值时,`__set()` 会被调用.

`public mixed __get ( string $name )`

读取不可访问属性的值时,`__get()` 会被调用.

**示例**

		class A
		{
			private $var1;
			private $var2;
		
		    public function __set($name, $value) 
		    {
		        echo "Setting '$name' to '$value'\n";
		    }
		
		
			public function __get($name)
			{
				echo "Getting '$name'",PHP_EOL;
				return 'Hello World'.PHP_EOL;
			}
		}
		
		
		$a = new A();
		$a->var1 = 'A';
		echo $a->var1;

输出:
		Setting 'var1' to 'A'
		Getting 'var1'
		Hello World

####`__isset()`和`__unset()`

在静态方法中,这些魔术方法将不会被调用.

当对不可访问属性调用 `isset()` 或 `empty()` 时, `__isset()` 会被调用.

当对不可访问属性调用 `unset()` 时, `__unset()` 会被调用.

		<?php
		
		class A
		{
			private $var1;
			public $var2;
			public $var3 = 1;
		
		    public function __isset($name) 
		    {
		        echo "Is '$name' set?\n";
		    }
		
		
			public function __unset($name)
			{
				echo "Unsetting '$name'\n";
			}
		}
		
		
		$a = new A();
		
		var_dump(isset($a->var2));//false
		var_dump(isset($a->var3));//true
		isset($a->var1);//Is var1 set?
		empty($a->var1);//Is var1 set?
		unset($a->var1);//Unsetting var1 

####`__clone()`

当复制完成时,如果定义了 `__clone()` 方法,则新创建的对象(复制生成的对象)中的 `__clone()` 方法会被调用,可用于修改属性的值(如果有必要的话).

我们先理解`clone`和不克隆的区别:

		class A
		{
			private $var1;
		
			public function __construct()
			{
				$this->var1 = 1;
			}
		
			public function setValue($value)
			{
				$this->var1 = $value;
			}
		
			public function run()
			{
				echo $this->var1, PHP_EOL;
			}
		}
		
		$a = new A();
		$b = clone $a;
		$c = $a;
		$a->run();//1
		$b->run();//1
		$c->run();//1
		$a->setValue(2);
		$b->setValue(3);
		$c->setValue(4);
		$a->run();//4
		$b->run();//3
		$c->run();//4;
		
`$c`是`$a`的一个引用,所以`$c->setValue(4)`时候,`$a`的值也连带修改了.

`$b`是`$a`的克隆,不在是`$a`的引用而是一个副本(`copy`).

**`__clone()`示例**

		class A
		{
			private $var1;
		
			public function __construct()
			{
				$this->var1 = 1;
			}
		
			public function setValue($value)
			{
				$this->var1 = $value;
			}
		
			public function run()
			{
				echo $this->var1, PHP_EOL;
			}
		
			public function __clone()
			{
				$this->var1 = 2;
			}
		}
		
		$a = new A();
		$b = clone $a;
		$a->run();//1
		$b->run();//2

`$b`在克隆`$a`时候`__clone`方法被调用,所以`var1`的值被修改了.